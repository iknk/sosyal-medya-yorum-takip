# Kurulum Kılavuzu

## Kurulum Durumu

| Adım | Durum |
|---|---|
| Meta Developer hesabı | ✅ Tamamlandı |
| Facebook Page oluşturma | ✅ Tamamlandı |
| Instagram Business hesabı + Facebook Page bağlama | ✅ Tamamlandı |
| Meta App (`yorum-takip`) oluşturma | ✅ Tamamlandı |
| Instagram izinleri + Access Token (IGAA) | ✅ Tamamlandı |
| Facebook Page Access Token alma | ⏳ Bekliyor (kaldığımız yer) |
| n8n credentials girişi | ⏳ Bekliyor |
| Workflow yapılandırması | ⏳ Bekliyor |

---

## 1. Meta Hesabı ve Uygulama — TAMAMLANDI

### Oluşturulan Kaynaklar

| Kaynak | Değer |
|---|---|
| Meta App adı | `yorum-takip` |
| Meta App ID | `790922707304467` |
| Instagram App adı | `yorum-takip-IG` |
| Instagram App ID | `863352076158067` |
| Facebook Page ID | `1065597503495311` |

### 1.1 Meta Developer Hesabı

1. [developers.facebook.com](https://developers.facebook.com) adresine git
2. Facebook hesabınla giriş yap → **Get Started**
3. Use case seç: **Other** → **Business** türü uygulama oluştur
4. Uygulama adı ve e-posta gir, **Create App** tıkla

### 1.2 Facebook Page Oluşturma

1. [facebook.com/pages/create](https://facebook.com/pages/create) adresine git
2. **Herkese Açık Sayfa** seç → işletme adı ve kategori gir → oluştur

### 1.3 Instagram Business Hesabı + Facebook Page Bağlama

1. Instagram uygulamasında **Hesap** → **Profesyonele Geç** → **İşletme** seç
2. Facebook Sayfanı seç veya yeni oluştur → bağla
3. Sonradan bağlamak için: Instagram **Ayarlar** → **Hesap** → **Sosyal ağlar ve üçüncü taraf uygulamalar** → Facebook

### 1.4 Instagram İzinleri ve Access Token — TAMAMLANDI

Developer Console → **Kullanım durumları** → **API setup with Instagram login**:

1. **"Add all required permissions"** tıkla — şu izinler eklendi:
   - `instagram_business_basic`
   - `instagram_manage_comments`
   - `instagram_business_manage_messages`

2. **App Roles** → **Roles** → **Instagram Testers** → kendi Instagram kullanıcı adını ekle

3. Instagram uygulamasında daveti kabul et: **Ayarlar** → **Uygulamalar ve web siteleri**

4. Developer Console'a geri dön → **"Add account"** → Instagram'la giriş yap → **"İzin ver"**

5. Üretilen token formatı: `IGAA...` — güvenli bir yerde sakla

> Token yenileme: Developer Console → `yorum-takip-IG` → API setup with Instagram login → Refresh token

---

## 2. Facebook Page Access Token Alma — BEKLIYOR

Buradan devam edilecek.

### Graph API Explorer Adımları

1. [developers.facebook.com/tools/explorer](https://developers.facebook.com/tools/explorer) adresine git
2. **Meta App:** `yorum-takip` seçili olsun
3. **"Generate Access Token"** → şu izinleri işaretle → üret:
   - `pages_show_list`
   - `business_management`
   - `instagram_manage_comments`
4. Token üretilince **"User or Page"** dropdown'ından **Facebook Sayfanı** seç (User Token → Page Token'a geçer)
5. URL alanına yaz ve Submit:
   ```
   1065597503495311?fields=instagram_business_account
   ```
6. Dönen `instagram_business_account.id` değerini not al — bu **Instagram Account ID**'dir

---

## 3. n8n Credentials Ekleme

### 3.1 Meta Graph API Credential (Facebook Page Token)

n8n'de:

1. **Settings** → **Credentials** → **Add Credential**
2. **Facebook Graph API** seç
3. İsim: `Meta Graph API`
4. **Access Token** alanına Facebook Page Access Token'ını (EAA...) yapıştır
5. Kaydet

### 3.2 DeepSeek API Credential

1. [platform.deepseek.com](https://platform.deepseek.com) → **API Keys** → **Create new secret key**
2. n8n'de **Add Credential** → **HTTP Bearer Auth** seç
3. İsim: `DeepSeek API`
4. **Token** alanına DeepSeek API key'ini yapıştır, kaydet

> Workflow DeepSeek'i doğrudan HTTP Request ile çağırır. Bu nedenle credential türü "HTTP Bearer Auth" olmalıdır.

---

## 4. Workflow Yapılandırması

n8n'de `7D32QaDRQyTUBmtC` ID'li workflow'u aç, **Marka Yapilandirmasi** node'una tıkla:

| Alan | Açıklama | Örnek |
|---|---|---|
| `brandName` | İşletme adı | `Kafe İstanbul` |
| `brandTone` | Yanıt tonu | `samimi ve yardımsever` |
| `instagramAccountId` | Instagram Business Account ID | Graph API Explorer'dan alınacak |
| `facebookPageId` | Facebook Page ID | `1065597503495311` |

---

## 5. Workflow'u Aktif Etme

1. Workflow'u aç → sağ üstteki toggle'ı **Active** yap
2. İlk çalışmayı tetiklemek için "Execute Workflow" tıkla
3. Her 15 dakikada otomatik çalışacak

---

## Sık Karşılaşılan Sorunlar

| Hata | Çözüm |
|---|---|
| `OAuthException: Invalid OAuth access token` | Token süresi dolmuş. Yeni token al. |
| `(#200) Requires extended permission` | Graph API Explorer'dan eksik izni ekle |
| `Instagram account not found` | Instagram hesabının Facebook Page'e bağlı ve Business hesabı olduğunu doğrula |
| `Yetersiz Geliştirici Görevi` | Instagram hesabına Tester rolü verilmemiş — App Roles → Roles → Instagram Testers → hesabı ekle ve daveti Instagram'dan kabul et |
| DeepSeek API hatası | API key geçerliliğini ve kota limitini kontrol et |

---

## Token Yenileme

- **Instagram Token (IGAA):** Developer Console → `yorum-takip-IG` → API setup with Instagram login → Refresh token
- **Facebook Page Token:** 60 günde bir Graph API Explorer'dan yenile. Otomatik yenileme için Meta Business Suite → System Users.
