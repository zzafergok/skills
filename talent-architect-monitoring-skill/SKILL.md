---
name: talent-architect-monitoring-skill
description: Design, implement, and evolve the Talent Architect monitoring dashboard. Covers KPI selection, Firestore-backed telemetry schema, admin dashboard layout, alert design, and observability best practices. Use when working on /admin/monitoring, monitoring API routes, telemetry events, or platform health visualization.
---

# Talent Architect Monitoring Skill

Talent Architect'in Firestore-backed monitoring altyapısını ve `/admin/monitoring` dashboard'unu tasarlama ve geliştirme skill'i.

## When to use this skill

- `/admin/monitoring` sayfası üzerinde çalışılırken
- `src/lib/monitoring/*` altyapısı geliştirilirken
- `src/app/api/admin/monitoring/*` route'ları eklenirken/değiştirilirken
- Yeni telemetry event tipi tanımlanırken
- Monitoring dashboard layout'u gözden geçirilirken
- Platform health KPI'ları belirlenirken

## How to use it

- Her zaman içsel olarak İngilizce düşün
- Her zaman kullanıcıya Türkçe yanıt ver
- Kod veya konfigürasyon dosyalarına comment satırı yazma

---

## 1. Mevcut Monitoring Altyapısı

### Proje Yapısı

```
src/lib/monitoring/
  config.ts           — Monitoring konfigürasyonu
  redaction.ts        — PII temizleme
  request-context.ts  — İstek bağlamı
  persistence.ts      — Firestore + in-memory fallback
  fetch.ts            — Monitored fetch utility
  route.ts            — Route wrapper
  client.ts           — Client-side capture

src/app/api/admin/monitoring/
  overview/route.ts
  services/route.ts
  incidents/route.ts
  events/route.ts
  timeseries/route.ts

src/app/api/monitoring/
  client/route.ts     — Client error + web vitals intake
```

### Event Tipleri

Yeni event tipi eklerken tutarlılık için şu naming convention'ı uygula:

```typescript
type MonitoringEventType =
  | 'route.request' // HTTP istek tamamlandı
  | 'route.error' // Route-level hata
  | 'auth.login' // Kullanıcı giriş
  | 'auth.logout' // Kullanıcı çıkış
  | 'auth.error' // Auth hatası
  | 'ai.completion' // AI tamamlama çağrısı
  | 'ai.error' // AI servis hatası
  | 'subscription.check' // Subscription kontrol
  | 'client.error' // Client-side JS hatası
  | 'client.vitals' // Web vitals (LCP, FID, CLS)
```

---

## 2. KPI Hiyerarşisi

### Talent Architect için Temel KPI'lar

#### Platform Health (Operasyonel — Real-time)

```yaml
Kullanılabilirlik:
  - Route başarı oranı (%)
  - Aktif incident sayısı
  - Son 1 saatte hata sayısı

Performans:
  - Median yanıt süresi (ms)
  - P95 yanıt süresi (ms)
  - Yavaş route'lar (>2s)

AI Servisleri:
  - AI completion başarı oranı (%)
  - Ortalama AI yanıt süresi (ms)
  - Rate limit isabetleri (saatlik)
```

#### Kullanıcı Metrikleri (Taktik — Günlük/Haftalık)

```yaml
Kullanım:
  - Günlük aktif kullanıcı (DAU)
  - Feature başına kullanım (CV, ATS, chat, hesaplama)
  - Oturum başına ortalama işlem sayısı

Dönüşüm:
  - Ücretsiz → Ücretli dönüşüm oranı
  - Feature adoption rate (ATS, cover letter, chat)

Kalite:
  - Kullanıcı başına ortalama ATS skoru
  - CV oluşturma tamamlanma oranı
  - Chat oturumu başarı oranı
```

---

## 3. Dashboard Layout Prensipleri

### Bilgi Hiyerarşisi

```
┌─────────────────────────────────────────┐
│  Kritik Metrikler (Büyük rakamlar)      │  ← Hemen görünen
├─────────────────────────────────────────┤
│  Trend Grafikleri (Zaman serisi)        │  ← Örüntü tespiti
├─────────────────────────────────────────┤
│  Detaylı Tablolar / Heatmap'ler         │  ← Derinlemesine analiz
├─────────────────────────────────────────┤
│  Aktif Incident'lar / Alert'ler         │  ← Aksiyon gerektiren
└─────────────────────────────────────────┘
```

### RED Method (Servisler için)

- **Rate** — Saniyedeki istek sayısı
- **Errors** — Hata oranı
- **Duration** — Yanıt süresi / gecikme

### USE Method (Kaynaklar için)

- **Utilization** — Kaynak meşguliyet yüzdesi
- **Saturation** — Kuyruk uzunluğu / bekleme süresi
- **Errors** — Hata sayısı

---

## 4. Admin Monitoring Dashboard Bileşenleri

### Overview Panel

```typescript
interface OverviewData {
  period: '1h' | '24h' | '7d' | '30d'
  successRate: number // %
  totalRequests: number
  avgResponseTime: number // ms
  p95ResponseTime: number // ms
  activeIncidents: number
  aiSuccessRate: number // %
  topErrorRoutes: Array<{
    route: string
    errorCount: number
    lastOccurrence: Date
  }>
}
```

### Services Panel

```typescript
interface ServiceHealth {
  service: 'auth' | 'ai' | 'firestore' | 'cv' | 'ats' | 'chat'
  status: 'healthy' | 'degraded' | 'down'
  successRate: number
  avgLatency: number
  errorCount: number
  lastCheck: Date
}
```

### Timeseries Panel

```typescript
interface TimeseriesPoint {
  timestamp: Date
  requestCount: number
  errorCount: number
  avgDuration: number
  p95Duration: number
}
```

---

## 5. Firestore Monitoring Schema

### Collection Yapısı

```
monitoring_events/{eventId}
  eventType: string
  route: string
  duration: number (ms)
  statusCode: number
  userId?: string (hashed — PII değil)
  timestamp: Timestamp
  metadata: {
    service?: string
    aiProvider?: string
    errorMessage?: string (sanitized)
  }
  ttl: Timestamp (otomatik temizlik için)
```

### Retention Policy (Önerilir)

```typescript
const RETENTION_POLICY = {
  'route.error': 30 * 24 * 60 * 60, // 30 gün
  'route.request': 7 * 24 * 60 * 60, //  7 gün
  'ai.completion': 14 * 24 * 60 * 60, // 14 gün
  'client.vitals': 7 * 24 * 60 * 60, //  7 gün
  'client.error': 30 * 24 * 60 * 60, // 30 gün
}
```

### PII Redaction Kuralları

Monitoring event'lerine hiçbir zaman şunlar yazılmamalı:

- Email adresi
- Ad soyad
- Firebase UID (hash'lenmiş versiyon kullanılabilir)
- IP adresi (opsiyonel, GDPR uyumu için)
- CV içeriği
- Chat mesajları

---

## 6. Alert Tasarım Prensipleri

### Alert Öncelik Seviyeleri

```typescript
type AlertSeverity = 'critical' | 'warning' | 'info'

const ALERT_THRESHOLDS = {
  successRate: {
    critical: 95, // < %95 hata oranı kritik
    warning: 98, // < %98 uyarı
  },
  p95Latency: {
    critical: 5000, // > 5s kritik
    warning: 2000, // > 2s uyarı
  },
  aiErrorRate: {
    critical: 10, // > %10 kritik
    warning: 5, // > %5 uyarı
  },
  activeIncidents: {
    critical: 3, // > 3 açık incident kritik
    warning: 1,
  },
}
```

### Alert Mesaj Formatı (UX Writing Uyumlu)

```typescript
const alertMessages = {
  highErrorRate: (rate: number) =>
    `Hata oranı %${rate}'e yükseldi. Son 15 dakika içinde ${Math.round(rate * 10)} istek başarısız oldu. Route loglarını kontrol et.`,

  slowRoutes: (route: string, p95: number) =>
    `${route} rotası yavaşladı. P95 yanıt süresi ${p95}ms. AI servis gecikmesi olabilir.`,

  aiProviderError: (provider: string) => `${provider} servisi yanıt vermiyor. Fallback provider'a geçiş aktif edildi.`,
}
```

---

## 7. Grafik Tipi Seçimi

| Durum                                      | Grafik Tipi                     |
| ------------------------------------------ | ------------------------------- |
| Zaman serisi trend (request rate, latency) | Line chart                      |
| Tek anlık değer (success rate, P95)        | Stat card (büyük rakam + delta) |
| Route bazlı karşılaştırma                  | Horizontal bar chart            |
| Latency dağılımı                           | Histogram                       |
| Hata dağılımı (route, service)             | Pie / Donut                     |
| Cohort / period karşılaştırma              | Grouped bar chart               |
| Hotspot tespiti (saat × gün)               | Heatmap                         |

### Renk Kodu (Platform Token Uyumlu)

```typescript
const chartColors = {
  success: 'hsl(var(--success))',
  error: 'hsl(var(--destructive))',
  warning: 'hsl(var(--warning))',
  primary: 'hsl(var(--primary))',
  muted: 'hsl(var(--muted-foreground))',
}
```

---

## 8. Best Practices

### Dashboard Tasarımı

- Maksimum 5–7 KPI tek ekranda — odak yeteneği sınırlı
- Trend olmadan tek değer anlamsız — zaman karşılaştırması ekle
- Metodoloji gizleme — hesaplama açıklaması her metrik yanında
- Mobile dashboard — admin paneli de responsive olmalı
- 3D grafik kullanma — algıyı bozar
- Vanity metric gösterme — "toplam kayıt sayısı" actionable değil

### Veri Güvenliği

- Admin route'ları `lib/admin/auth.ts` ile korunmalı
- Monitoring event'lerinde PII bulunmamalı (redaction zorunlu)
- TTL field'ı her event'e yazılmalı — otomatik temizlik
- Rate limiting monitoring intake endpoint'lerine uygulanmalı

### Performans

- Dashboard sorguları Firestore index'leri kullanmalı
- Aggregation'lar API route'da yapılmalı, client'ta değil
- Büyük event koleksiyonları için Firestore `limit()` zorunlu
- Real-time listener yerine polling tercih edilebilir (maliyet kontrolü)

---

## 9. Mevcut Mimari Borç

README.md'de belgelenmiş açık konular:

1. **Retention policy yok** — Event'ler süresiz birikir, Firestore maliyeti artar. `ttl` field'ı eklenip Firestore TTL policy'si aktif edilmeli.
2. **Alerting mekanizması yok** — Threshold aşılınca bildirim gönderilmiyor. Firestore trigger veya scheduled function eklenebilir.
3. **Arşiv analizi yok** — Geçmiş veriler query edilemiyor. BigQuery export veya aggregated summary document'lar oluşturulabilir.
4. **Test coverage sınırlı** — AI-heavy monitoring flow'ları için integration test eksik.

Bu konular öncelik sırasıyla ele alınmalı.
