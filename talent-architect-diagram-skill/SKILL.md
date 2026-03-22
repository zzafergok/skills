---
name: talent-architect-diagram-skill
description: Create professional architecture diagrams and visual documentation for the Talent Architect platform. Covers C4 model diagrams (Context, Container, Component, Deployment, Dynamic), Mermaid flowcharts, sequence diagrams, ERDs, state diagrams, and Gantt charts. Use when documenting system architecture, visualizing data flows, creating technical documentation, or explaining platform components visually.
---

# Talent Architect Diagram Skill

Talent Architect platformunun mimari dokümantasyonu ve teknik görselleştirme için tek kaynak skill. C4 model diyagramları ve tüm Mermaid diagram tipleri dahil.

## When to use this skill

- Sistem mimarisi veya component yapısı dokümante edilirken
- Feature flow'ları veya API sequence'ları görselleştirilirken
- ERD veya data model diyagramı oluştururken
- Proje planı veya sprint timeline'ı için Gantt chart'a ihtiyaç duyulurken
- PR veya teknik dokümana diagram eklenirken
- Yeni geliştirici onboarding'i için mimari açıklanırken

## How to use it

- Her zaman içsel olarak İngilizce düşün
- Her zaman kullanıcıya Türkçe yanıt ver
- Kod veya konfigürasyon dosyalarına comment satırı yazma

---

## 1. Diagram Tipi Seçimi

### C4 Model — Mimari Dokümantasyon

| Level | Tip              | Kitle          | Gösterir                   |
| ----- | ---------------- | -------------- | -------------------------- |
| 1     | **C4Context**    | Herkes         | Sistem + dış aktörler      |
| 2     | **C4Container**  | Teknik ekip    | App'lar, DB'ler, servisler |
| 3     | **C4Component**  | Geliştiriciler | İç component'ler           |
| 4     | **C4Deployment** | DevOps         | Altyapı node'ları          |
| —     | **C4Dynamic**    | Teknik ekip    | Numaralı istek akışları    |

**Kural**: Context + Container diyagramları çoğu ekip için yeterli. Component/Code diyagramlarını yalnızca gerçek değer kattığında oluştur.

### Mermaid — Genel Görselleştirme

| Durum                              | Diagram Tipi        |
| ---------------------------------- | ------------------- |
| Adım adım süreç, karar ağacı       | `graph` (flowchart) |
| API etkileşimleri, servisler arası | `sequenceDiagram`   |
| Veri modeli, ilişkiler             | `erDiagram`         |
| Nesne hiyerarşisi                  | `classDiagram`      |
| State machine, lifecycle           | `stateDiagram-v2`   |
| Sprint/release planı               | `gantt`             |
| Oranlar                            | `pie`               |
| Kullanıcı yolculuğu                | `journey`           |
| Git branch stratejisi              | `gitGraph`          |

---

## 2. C4 Diyagram Örnekleri

### Talent Architect — System Context (Level 1)

```mermaid
C4Context
  title System Context — Talent Architect

  Person(jobSeeker, "İş Arayan", "CV oluşturur, ATS analizi yapar")
  Person(admin, "Admin", "Kullanıcıları ve sistemi yönetir")

  System(ta, "Talent Architect", "Kariyer platformu — CV, ATS, chatbot, hesaplama araçları")

  System_Ext(firebase, "Firebase", "Auth + Firestore")
  System_Ext(anthropic, "Anthropic API", "Claude AI modeli")
  System_Ext(openrouter, "OpenRouter", "Çoklu AI model erişimi")
  System_Ext(resend, "Resend", "Email gönderimi")

  Rel(jobSeeker, ta, "Kullanır", "HTTPS")
  Rel(admin, ta, "Yönetir", "HTTPS")
  Rel(ta, firebase, "Auth + veri okuma/yazma", "Firebase SDK")
  Rel(ta, anthropic, "AI tamamlama", "REST API")
  Rel(ta, openrouter, "AI tamamlama", "REST API")
  Rel(ta, resend, "Email gönderir", "REST API")
```

### Talent Architect — Container Diagram (Level 2)

```mermaid
C4Container
  title Container Diagram — Talent Architect

  Person(user, "Kullanıcı", "İş arayan veya admin")

  Container_Boundary(ta, "Talent Architect — Next.js 15") {
    Container(web, "Web App", "Next.js App Router", "SSR + CSR kariyer platformu")
    Container(api, "API Routes", "Next.js Route Handlers", "REST endpoint'leri")
    Container(monitoring, "Monitoring", "Custom lib", "Telemetry ve observability")
  }

  Container_Boundary(state, "Client State") {
    Container(zustand, "Auth/App Store", "Zustand", "Session, subscription, config")
    Container(rq, "Server Cache", "React Query", "API response cache")
  }

  ContainerDb(firestore, "Firestore", "Firebase", "Kullanıcı verileri, CV'ler, olaylar")
  System_Ext(aiProviders, "AI Providers", "Anthropic + OpenRouter")

  Rel(user, web, "Kullanır", "HTTPS")
  Rel(web, api, "API çağrısı", "JSON/HTTP")
  Rel(web, zustand, "State okur/yazar")
  Rel(web, rq, "Server state")
  Rel(api, firestore, "Okur/yazar", "Firebase Admin SDK")
  Rel(api, aiProviders, "AI tamamlama", "REST API")
  Rel(monitoring, firestore, "Event yazar", "Firebase Admin SDK")
```

### Feature Akışı — Dynamic Diagram

```mermaid
C4Dynamic
  title CV Yükleme ve ATS Analizi Akışı

  Container(web, "Web App", "Next.js")
  Container(upload, "CV Upload API", "Route Handler")
  Container(ats, "ATS Service", "Feature Server")
  ContainerDb(db, "Firestore", "Firebase")
  System_Ext(ai, "AI Provider", "Anthropic/OpenRouter")

  Rel(web, upload, "1. PDF yükle", "multipart/form-data")
  Rel(upload, db, "2. CV kaydet", "Firebase Admin SDK")
  Rel(web, ats, "3. Analiz başlat", "JSON/HTTPS")
  Rel(ats, db, "4. CV verisi oku")
  Rel(ats, ai, "5. ATS prompt", "REST API")
  Rel(ats, db, "6. Sonuç kaydet")
  Rel(web, ats, "7. Sonuç al", "polling/streaming")
```

---

## 3. Mermaid Flowchart Örnekleri

### Auth Flow

```mermaid
graph TD
  A([Kullanıcı Giriş]) --> B{Session Var mı?}
  B -->|Evet| C[Dashboard'a Yönlendir]
  B -->|Hayır| D[Login Sayfası]
  D --> E[Firebase Auth]
  E -->|Başarılı| F[Session Cookie Oluştur]
  F --> G[DataInitializer]
  G --> H[App Config + Subscription Yükle]
  H --> C
  E -->|Başarısız| I[Hata Göster]
  I --> D
```

### Subscription Check

```mermaid
graph LR
  A[Feature Talebi] --> B{Auth Kontrol}
  B -->|Yetkisiz| C[401 Döndür]
  B -->|Yetkili| D{Subscription Kontrol}
  D -->|Free Limit Aşıldı| E[Upgrade Prompt]
  D -->|Pro/Premium| F[Feature Çalıştır]
  F --> G{AI Kullanıyor mu?}
  G -->|Evet| H[Rate Limit Kontrol]
  H -->|Limit Aşıldı| I[429 Döndür]
  H -->|OK| J[AI Servis Çağır]
  G -->|Hayır| K[Sonuç Döndür]
  J --> K
```

---

## 4. Sequence Diagram Örnekleri

### AI Chat Akışı

```mermaid
sequenceDiagram
  participant U as Kullanıcı
  participant W as Web App
  participant A as Chat API Route
  participant S as Chat Service
  participant AI as Anthropic API
  participant DB as Firestore

  U->>W: Mesaj gönder
  W->>A: POST /api/chat
  A->>A: Auth kontrol
  A->>S: processMessage(userId, message)
  S->>DB: Kullanıcı bağlamını oku
  S->>AI: Claude API stream
  AI-->>S: Token stream
  S-->>A: Stream yönlendir
  A-->>W: SSE stream
  W-->>U: Gerçek zamanlı yanıt
  S->>DB: Konuşma geçmişi kaydet
```

---

## 5. ERD — Firestore Veri Modeli

```mermaid
erDiagram
  USERS {
    string uid PK
    string email
    string displayName
    timestamp createdAt
    string subscriptionTier
  }
  CV_DOCUMENTS {
    string id PK
    string userId FK
    string title
    json content
    timestamp createdAt
    timestamp updatedAt
  }
  ATS_ANALYSES {
    string id PK
    string userId FK
    string cvId FK
    string jobDescription
    json analysisResult
    number score
    timestamp createdAt
  }
  COVER_LETTERS {
    string id PK
    string userId FK
    string cvId FK
    string content
    string jobTitle
    timestamp createdAt
  }
  CHAT_SESSIONS {
    string id PK
    string userId FK
    json messages
    timestamp lastActivity
  }
  MONITORING_EVENTS {
    string id PK
    string eventType
    string route
    number duration
    timestamp timestamp
    json metadata
  }

  USERS ||--o{ CV_DOCUMENTS : "sahip olur"
  USERS ||--o{ ATS_ANALYSES : "yapar"
  USERS ||--o{ COVER_LETTERS : "oluşturur"
  USERS ||--o{ CHAT_SESSIONS : "açar"
  CV_DOCUMENTS ||--o{ ATS_ANALYSES : "analiz edilir"
  CV_DOCUMENTS ||--o{ COVER_LETTERS : "temel alınır"
```

---

## 6. Deployment Diagram

```mermaid
C4Deployment
  title Deployment — Talent Architect (Vercel)

  Deployment_Node(browser, "Kullanıcı Tarayıcısı", "Chrome/Firefox/Safari") {
    Container(spa, "Next.js Client", "React 18", "Hydrated SPA")
  }

  Deployment_Node(vercel, "Vercel Edge Network", "Serverless") {
    Container(ssr, "Next.js SSR", "Node 22.x", "Server components + Route handlers")
  }

  Deployment_Node(firebase, "Google Firebase", "GCP") {
    ContainerDb(auth, "Firebase Auth", "OAuth/Email", "Kimlik doğrulama")
    ContainerDb(firestore, "Firestore", "NoSQL", "Uygulama verisi")
  }

  Deployment_Node(external, "Dış Servisler", "Cloud") {
    Container(anthropic, "Anthropic API", "REST", "Claude modeli")
    Container(openrouter, "OpenRouter", "REST", "Çoklu AI")
    Container(resend, "Resend", "REST", "Email")
  }

  Rel(spa, ssr, "SSR + API", "HTTPS")
  Rel(ssr, auth, "Session doğrula", "Firebase Admin SDK")
  Rel(ssr, firestore, "CRUD", "Firebase Admin SDK")
  Rel(ssr, anthropic, "AI tamamlama", "REST API")
  Rel(ssr, openrouter, "AI tamamlama", "REST API")
  Rel(ssr, resend, "Email gönder", "REST API")
```

---

## 7. Sözdizimi Referansı

### Temel Elementler

```
Person(alias, "Etiket", "Açıklama")
System(alias, "Etiket", "Açıklama")
System_Ext(alias, "Etiket", "Açıklama")
Container(alias, "Etiket", "Teknoloji", "Açıklama")
ContainerDb(alias, "Etiket", "Teknoloji", "Açıklama")
Component(alias, "Etiket", "Teknoloji", "Açıklama")

Container_Boundary(alias, "Etiket") { ... }
System_Boundary(alias, "Etiket") { ... }
Deployment_Node(alias, "Etiket", "Tip") { ... }
```

### İlişkiler

```
Rel(from, to, "Etiket")
Rel(from, to, "Etiket", "Teknoloji")
BiRel(from, to, "Etiket")
Rel_U / Rel_D / Rel_L / Rel_R (yön kontrolü)
```

### Layout Kontrolü

```
UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
UpdateRelStyle(from, to, $textColor="blue", $offsetY="-20")
UpdateElementStyle(alias, $bgColor="grey", $borderColor="red")
```

---

## 8. Best Practices

### C4 İçin

- Her element için: Ad + Tip + Teknoloji + Açıklama (kısa, ≤50 karakter)
- Tek yönlü oklar — çift yönlü oklar belirsizlik yaratır
- Ok etiketi eylem fiiliyle başlamalı: "Okur", "Yazar", "Gönderir"
- Teknoloji etiketi ekle: "JSON/HTTPS", "Firebase Admin SDK"
- Diyagram başına ≤20 element
- Her diyagram tek bir dosya

### Mermaid İçin

- Diyagram tipi veriyle eşleşmeli — süreç için graph, etkileşim için sequence
- Okunabilirliği koru — aşırı kalabalık diyagramdan kaçın
- Tutarlı stil ve renkler kullan
- Karmaşık sözdizimini açıklayan yorum ekle (sadece `.md` dosyasında, kod içinde değil)
- Render etmeden önce test et

### Tenant Architectural Rules

- Multi-team ownership varsa → microservice'i System'e terfi ettir
- Event-driven mimaride → tek "Kafka" kutusu değil, individual topic/queue container'ları
- Shared library'leri Container olarak modelleme — Component'tir

---

## 9. Output Locations

Mimari dokümantasyonu şuraya yaz:

```
docs/architecture/
  c4-context.md        — System context diyagramı
  c4-containers.md     — Container diyagramı
  c4-deployment.md     — Deployment diyagramı
  c4-dynamic-{flow}.md — Özel akış diyagramları
  c4-components-{feature}.md — Feature bazlı component diyagramları
```
