# Kurulum Kılavuzu

## 1. Meta Hesabı ve Uygulama Oluşturma

> Henüz Meta hesabın yoksa bu adımları takip et.

### 1.1 Meta Developer Hesabı

1. [developers.facebook.com](https://developers.facebook.com) adresine git
2. Facebook hesabınla giriş yap
3. "My Apps" → "Create App" tıkla
4. Uygulama türü: **Business**
5. Uygulama adı ve e-posta gir, "Create App" tıkla

### 1.2 Facebook Page ve Instagram Business Hesabı Bağlama

Instagram Business Account'un yoksa:

1. Instagram uygulamasında **Hesap** → **Profesyonele Geç** → **İşletme** seç
2. Mevcut bir Facebook Sayfasına bağla (yoksa yeni oluştur)

### 1.3 Uygulamaya İzinler Ekle

Developer Console'da uygulamanın "Dashboard"una git:

1. **Add Product** → **Webhooks** (opsiyonel, şimdilik gerek yok)
2. **Add Product** → **Instagram Graph API**
3. **Permissions & Features** bölümünde şunların onaylı olduğunu doğrula:
   - `instagram_basic`
   - `instagram_manage_comments`
   - `pages_read_engagement`
   - `pages_manage_engagement`

### 1.4 Page Access Token Alma

```
Graph API Explorer → https://developers.facebook.com/tools/explorer/
```

1. Uygulamanı seç (üst dropdown)
2. "Get User Access Token" → gerekli izinleri işaretle → "Generate Access Token"
3. Elde ettiğin kısa ömürlü (1 saatlik) token'ı uzun ömürlüye çevir:

```
GET https://graph.facebook.com/v23.0/oauth/access_token
  ?grant_type=fb_exchange_token
  &client_id={APP_ID}
  &client_secret={APP_SECRET}
  &fb_exchange_token={SHORT_LIVED_TOKEN}
```

4. "Exchange Token" → Page Token al:

```
GET https://graph.facebook.com/v23.0/me/accounts
  ?access_token={LONG_LIVED_USER_TOKEN}
```

Dönen listeden işletmenin Page'ini bul, `access_token` alanı senin **Page Access Token**'ın.

### 1.5 Instagram Business Account ID Bulma

```
GET https://graph.facebook.com/v23.0/{PAGE_ID}
  ?fields=instagram_business_account
  &access_token={PAGE_ACCESS_TOKEN}
```

Dönen `instagram_business_account.id` değeri workflow'da kullanacağın **Instagram Account ID**'dir.

---

## 2. n8n Credentials Ekleme

### 2.1 Meta Graph API Credential

n8n'de:

1. **Settings** → **Credentials** → **Add Credential**
2. **Facebook Graph API** seç
3. İsim: `Meta Graph API`
4. **Access Token** alanına Page Access Token'ını yapıştır
5. Kaydet

### 2.2 DeepSeek API Credential

1. [platform.deepseek.com](https://platform.deepseek.com) → **API Keys** → **Create new secret key**
2. n8n'de **Add Credential** → **HTTP Bearer Auth** seç
3. İsim: `DeepSeek API`
4. **Token** alanına API key'ini yapıştır, kaydet

> Workflow DeepSeek'i LangChain node'u yerine doğrudan HTTP Request ile çağırır. Bu nedenle credential türü "HTTP Bearer Auth" olmalıdır.

---

## 3. Workflow Yapılandırması

n8n'de `7D32QaDRQyTUBmtC` ID'li workflow'u aç, **Marka Yapilandirmasi** node'una tıkla:

| Alan | Açıklama | Örnek |
|---|---|---|
| `brandName` | İşletme adı | `Kafe İstanbul` |
| `brandTone` | Yanıt tonu | `samimi ve yardımsever` |
| `instagramAccountId` | Instagram Business Account ID | `17841400000000001` |
| `facebookPageId` | Facebook Page ID | `100000000001` |

---

## 4. Workflow'u Aktif Etme

1. Workflow'u aç → sağ üstteki toggle'ı **Active** yap
2. İlk çalışmayı tetiklemek için "Execute Workflow" tıkla
3. Her 15 dakikada otomatik çalışacak

---

## Sık Karşılaşılan Sorunlar

| Hata | Çözüm |
|---|---|
| `OAuthException: Invalid OAuth access token` | Token süresi dolmuş. Yeni Page Access Token al. |
| `(#200) Requires extended permission` | Graph API Explorer'dan eksik izni ekle |
| `Instagram account not found` | Instagram hesabının Facebook Page'e bağlı ve Business hesabı olduğunu doğrula |
| DeepSeek API hatası | API key geçerliliğini ve kota limitini kontrol et |

---

## Token Yenileme Hatırlatıcısı

Page Access Token'lar 60 günde bir sona erer. Sona ermeden önce yenilemeyi unutma. Otomatik yenileme için Meta'nın **System User** özelliğini kullanabilirsin (Meta Business Suite → System Users).
