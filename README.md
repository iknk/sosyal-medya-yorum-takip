# sosyal-medya-yorum-takip

Instagram ve Facebook gönderilerine gelen yorumları otomatik olarak DeepSeek V3 ile yanıtlayan n8n workflow'u.

## Ne Yapar?

Her 15 dakikada bir çalışır. İşletmenin Instagram Business ve Facebook Page hesaplarındaki son gönderileri tarar, yeni yorumları tespit eder ve her yorum için markanın tonuna uygun kısa bir Türkçe yanıt üretip gönderir.

## Workflow

**n8n ID:** `7D32QaDRQyTUBmtC`
**Ad:** Meta Yorum Takip ve Otomatik Yanit DeepSeek

## Gereksinimler

| Gereksinim | Açıklama |
|---|---|
| n8n instance | Self-hosted veya Cloud |
| Meta Developer Hesabı | developers.facebook.com |
| Page Access Token | Uzun ömürlü token (60 gün) |
| DeepSeek API Key | platform.deepseek.com |

## Hızlı Başlangıç

1. [Kurulum Kılavuzu](docs/kurulum.md) — credentials ve config adımları
2. [Workflow Mimarisi](docs/mimari.md) — node'lar ve veri akışı
3. Meta hesabın yoksa `docs/kurulum.md` → "Meta Hesabı Oluşturma" bölümüne bak

## Desteklenen Platformlar

- **Instagram Business** — yorum yanıtları `/{comment_id}/replies` üzerinden
- **Facebook Page** — yorum yanıtları `/{comment_id}/comments` üzerinden

## AI Modeli

DeepSeek V3 (`deepseek-chat`) — Türkçe, marka tonuna uygun, maksimum 2 cümle yanıt üretir.
