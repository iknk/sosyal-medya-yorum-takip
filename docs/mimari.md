# Workflow Mimarisi

## Genel Bakış

```
Schedule Trigger (15 dk)
    │
    ▼
Marka Yapilandirmasi        ← tek yapılandırma noktası
    │
    ▼
Platform Listesi Olustur    ← 2 item üretir: instagram + facebook
    │
    ▼ (her platform için ayrı çalışır)
Gonderileri Al              ← Graph API: /media (IG) veya /posts (FB)
    │
    ▼
Gonderileri Ayir            ← data[] listesini ayrı item'lara böler
    │
    ▼ (her gönderi için çalışır)
Yorumlari Al                ← Graph API: /{postId}/comments
    │
    ▼
Yorumlari Zenginlestir      ← platform/caption context ekler
    │
    ▼ (tüm itemlar bir arada)
Yorumlari Duzlestir         ← flatten + son 15 dk filtresi
ve Filtrele
    │
    ▼ (her yorum tek tek)
Loop: Yorumlari Tek Tek Isle
    │
    ├─▶ DeepSeek ile Yanit Olustur   ← AI Agent (deepseek-chat)
    │       │
    │       ▼
    │   Yanit Baglamini Hazirla      ← replyText + commentId + platform
    │       │
    │       ▼
    │   Platforma Gore Yonlendir (Switch)
    │       │                │
    │       ▼                ▼
    │   Instagram        Facebook
    │   Yanit Gonder     Yanit Gonder
    │   /replies         /comments
    │       │                │
    └───────┴────────────────┘ (nextBatch)
```

---

## Node'lar

### Marka Yapilandirmasi
**Tip:** Set (manual)

Tüm workflow için tek yapılandırma noktası. Workflow'u her işletmeye uyarlamak için yalnızca bu node'u düzenlemek yeterlidir.

| Alan | Kullanım |
|---|---|
| `brandName` | AI system prompt'a eklenir |
| `brandTone` | AI system prompt'a eklenir |
| `instagramAccountId` | `/media` endpoint'i için |
| `facebookPageId` | `/posts` endpoint'i için |

---

### Platform Listesi Olustur
**Tip:** Code (runOnceForAllItems)

Config node'undan 1 item alır, 2 item üretir:
```json
[
  { "platform": "instagram", "accountId": "...", "brandName": "...", "brandTone": "..." },
  { "platform": "facebook",  "accountId": "...", "brandName": "...", "brandTone": "..." }
]
```

Bu 2 item, aşağıdaki node'ların her ikisi için de paralel çalışmasını sağlar.

---

### Gonderileri Al
**Tip:** Facebook Graph API (GET)

Platform değerine göre dinamik endpoint:
```
{{ $json.platform === "instagram" ? "media" : "posts" }}
```

İstenen alanlar: `id`, `caption`, `message`, `timestamp`, `created_time`
Limit: son 10 gönderi

---

### Gonderileri Ayir
**Tip:** Code (runOnceForEachItem)

Graph API'nin döndürdüğü `data[]` listesini ayrı item'lara böler. Her item: `platform`, `brandName`, `brandTone`, `postId`, `postCaption`.

`$("Platform Listesi Olustur").item.json` ile platform context'i üst node'dan alır.

---

### Yorumlari Al
**Tip:** Facebook Graph API (GET)

Her gönderi için `/{postId}/comments` endpoint'ini çağırır.

İstenen alanlar: `id`, `text`, `message`, `username`, `from`, `timestamp`, `created_time`

> Instagram yorumları `text` + `username` döndürür.
> Facebook yorumları `message` + `from.name` döndürür.
> Her iki alan da isteklere dahil edildiğinden tek node her ikisini de kapsıyor.

---

### Yorumlari Zenginlestir
**Tip:** Code (runOnceForEachItem)

Yorumlar API yanıtına, hangi gönderiden geldiğini belirten `platform`, `postId`, `postCaption` bilgilerini ekler. Bir sonraki adımda birleşik filtreleme için bu context gereklidir.

---

### Yorumlari Duzlestir ve Filtrele
**Tip:** Code (runOnceForAllItems)

Tüm post'lardan gelen yorum listelerini (`data[]`) tek düz listeye dönüştürür. Eş zamanlı olarak:

- 15 dakikadan eski yorumları eler
- `text`/`message` ve `username`/`from.name` alanlarını normalize eder
- Her yorum için standart yapı üretir:

```json
{
  "platform": "instagram",
  "brandName": "Kafe Istanbul",
  "brandTone": "samimi",
  "commentId": "...",
  "commentText": "...",
  "authorName": "...",
  "postCaption": "..."
}
```

---

### Yorumlari Tek Tek Isle (Loop)
**Tip:** Split in Batches (batchSize: 1)

Her yorumu ayrı ayrı işler. Boş giriş geldiğinde (son 15 dk yorum yok) loop çalışmaz — bu beklenen davranıştır.

---

### DeepSeek ile Yanit Olustur
**Tip:** AI Agent + DeepSeek Chat Model (deepseek-chat)

**System message:**
```
Sen {brandName} markasinin sosyal medya yoneticisisin.
Ton: {brandTone}.
En fazla 2 cumle, Turkce, sadece yanit metnini yaz,
kullaniciyi adi ile hitap et.
```

**User prompt:**
```
Platform: {platform}
Gonderi: {postCaption}
Kullanici: {authorName}
Yorum: {commentText}
```

**Çıktı:** `{ "output": "yanit metni" }`

---

### Yanit Baglamini Hazirla
**Tip:** Set (manual)

AI Agent çıktısı yalnızca `output` alanını içerir; orijinal `commentId` ve `platform` kaybolur. Bu node:
- `replyText` = `$json.output`
- `commentId` = upstream loop item'dan (`Yorumlari Duzlestir ve Filtrele` node'undan)
- `platform` = upstream loop item'dan

---

### Platforma Gore Yonlendir (Switch)
**Tip:** Switch (v3.2)

`platform` alanına göre 2 çıkış:
- Case 0 → `instagram` → `/{commentId}/replies`
- Case 1 → `facebook`  → `/{commentId}/comments`

---

### Instagrama / Facebooka Yanit Gonder
**Tip:** Facebook Graph API (POST)

Instagram için `/{commentId}/replies?message={replyText}`
Facebook için `/{commentId}/comments?message={replyText}`

Her ikisi de aynı `Meta Graph API` credential'ını kullanır (Page Access Token her iki platformu da kapsar).

---

## Veri Akışı Özeti

```
Config (1 item)
  → Platform split (2 item)
  → Posts API × 2 (2 item)
  → Post split × N (N gönderi)
  → Comments API × N (N item, her biri comments listesi)
  → Enrich × N (N item, her biri comments dizisi ile)
  → Flatten+Filter (M yorum item — M = son 15 dk yorumları)
  → Loop × M (her yorum için AI + post)
```

---

## Değişiklik Yapma Rehberi

| İstenen değişiklik | Düzenlenecek node |
|---|---|
| Marka adı / tonu değiştirme | Marka Yapilandirmasi |
| Kontrol sıklığını değiştirme | Her 15 Dakikada Bir |
| Kaç gönderi kontrol edileceği | Gonderileri Al → `limit` parametresi |
| Kaç yorum kontrol edileceği | Yorumlari Al → `limit` parametresi |
| Zaman penceresi (15 dk) | Yorumlari Duzlestir ve Filtrele → `cutoff` hesabı |
| AI yanıt stili / uzunluğu | DeepSeek ile Yanit Olustur → system message |
| AI modeli | DeepSeek V3 node → `model` alanı (`deepseek-reasoner` = V3 thinking) |
