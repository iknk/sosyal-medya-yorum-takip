# sosyal-medya-yorum-takip

Instagram ve Facebook gönderilerine gelen yorumları otomatik olarak DeepSeek V3 ile yanıtlayan n8n workflow'u.

## Mevcut Durum

| Adım | Durum |
|---|---|
| n8n workflow oluşturma (14 node) | Tamamlandı |
| Meta Developer App (`yorum-takip`) | Tamamlandı |
| Instagram Business Login kurulumu | Tamamlandı |
| Instagram Access Token (IGAA) | Tamamlandı |
| Facebook Page oluşturma | Tamamlandı — Page ID: `1065597503495311` |
| Facebook Page Access Token alma | Bekliyor |
| n8n credentials girişi | Bekliyor |
| Workflow yapılandırması | Bekliyor |
| Canlı test | Bekliyor |

## Ne Yapar?

Her 15 dakikada bir çalışır. İşletmenin Instagram Business ve Facebook Page hesaplarındaki son gönderileri tarar, yeni yorumları tespit eder ve her yorum için markanın tonuna uygun kısa bir Türkçe yanıt üretip gönderir.

## Workflow

**n8n ID:** `7D32QaDRQyTUBmtC`
**Ad:** Meta Yorum Takip ve Otomatik Yanit DeepSeek

## Meta App Bilgileri

| Alan | Değer |
|---|---|
| App adı | `yorum-takip` |
| App ID | `790922707304467` |
| Instagram App adı | `yorum-takip-IG` |
| Instagram App ID | `863352076158067` |
| Facebook Page ID | `1065597503495311` |

## Gereksinimler

| Gereksinim | Açıklama |
|---|---|
| n8n instance | Self-hosted veya Cloud |
| Meta Developer Hesabı | developers.facebook.com |
| Instagram Access Token | `yorum-takip-IG` uygulamasından (IGAA...) |
| Facebook Page Access Token | Graph API Explorer'dan (EAA...) |
| DeepSeek API Key | platform.deepseek.com |

## Hızlı Başlangıç

1. [Kurulum Kılavuzu](docs/kurulum.md) — credentials ve config adımları
2. [Workflow Mimarisi](docs/mimari.md) — node'lar ve veri akışı
3. Meta hesabın yoksa `docs/kurulum.md` → "Meta Hesabı Oluşturma" bölümüne bak

## Desteklenen Platformlar

- **Instagram Business** — yorum yanıtları `/{comment_id}/replies` üzerinden (Instagram Business Login API)
- **Facebook Page** — yorum yanıtları `/{comment_id}/comments` üzerinden

## AI Modeli

DeepSeek V3 (`deepseek-chat`) — Türkçe, marka tonuna uygun, maksimum 2 cümle yanıt üretir.
