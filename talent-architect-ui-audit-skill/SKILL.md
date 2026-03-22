---
name: talent-architect-ui-audit-skill
description: Audit Talent Architect UI components and pages for accessibility, design system compliance, responsive design, UX writing quality, and performance. Use when asked to review, audit, or check any UI file, component, or page against platform standards.
---

# Talent Architect UI Audit Skill

Talent Architect'in UI bileşenlerini ve sayfalarını platform standartlarına, accessibility kurallarına ve design system uyumuna göre denetleme skill'i.

## When to use this skill

- "Şu component'i gözden geçir" isteklerinde
- PR review öncesi UI denetimi yapılırken
- Accessibility audit istendiğinde
- Design system uyumu kontrol edilirken
- UX copy kalitesi değerlendirilirken
- Responsive davranış test edilirken

## How to use it

- Her zaman içsel olarak İngilizce düşün
- Her zaman kullanıcıya Türkçe yanıt ver
- Kod veya konfigürasyon dosyalarına comment satırı yazma
- Bulguları `dosya:satır` formatında raporla

---

## Audit Süreci

Belirtilen dosya(lar)ı şu sırayla denetle:

1. **Design Token Uyumu** — Hardcoded değer var mı?
2. **Accessibility** — WCAG AA gereksinimleri karşılanıyor mu?
3. **Dark Mode Parity** — Her token dark modda test edildi mi?
4. **Responsive** — Tüm breakpoint'ler kapsanıyor mu?
5. **UX Writing** — Microcopy kalite standartları karşılanıyor mu?
6. **Component Pattern** — CVA, compound component, asChild doğru kullanılıyor mu?
7. **Performance** — Gereksiz re-render, animasyon sorunu var mı?
8. **TypeScript** — `any` tipi, eksik tip annotation var mı?

---

## Kural Seti

### R01 — Hardcoded Renk Değeri

```
KURAL: Tailwind'de ham renk sınıfı (text-blue-500, bg-gray-900) token yerine kullanılmamalı.
BEKLENEN: text-primary, bg-background, text-muted-foreground
NEDEN: Tema geçişlerinde kırılır.
```

### R02 — Dark Mode Eksik

```
KURAL: CSS variable kullanmayan her renk veya arka plan dark modda görünmez hale gelir.
KONTROL: Renk değeri var(--...) ile mi ifade ediliyor?
BEKLENEN: Tüm renkler CSS variable üzerinden
```

### R03 — Focus State Eksik

```
KURAL: Tüm interactive element'lerde focus-visible ring olmalı.
BEKLENEN: focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2
NEDEN: Klavye kullanıcıları ve accessibility zorunluluğu.
```

### R04 — Aria Eksikliği

```
KURAL: Icon-only butonlarda aria-label olmalı.
KURAL: Error state'lerde aria-invalid + aria-describedby olmalı.
KURAL: Loading state'lerde aria-busy olmalı.
KURAL: Dialog'larda Dialog.Title ve Dialog.Description olmalı.
```

### R05 — Generic UX Copy

```
KURAL: "Tamam", "Gönder", "Tıkla", "İptal" gibi bağlamsız buton metni kullanma.
BEKLENEN: Eylem + nesne — "CV Oluştur", "Analizi Başlat", "Hesabı Sil"
```

### R06 — Error Message Kalitesi

```
KURAL: Error message'lar kullanıcıyı suçlamamalı, recovery path içermeli.
BEKLENEN FORMAT: [Ne başarısız oldu]. [Neden]. [Ne yapılmalı].
KÖTÜ: "Geçersiz giriş.", "Hata oluştu."
İYİ: "CV yüklenemedi. Dosya boyutu 5MB'ı aşıyor. Daha küçük bir dosya seçin."
```

### R07 — Renk Tek Başına Anlam

```
KURAL: Renk tek başına bilgi taşımamalı (renk körü erişim).
BEKLENEN: Renk + ikon + metin birlikte kullanılmalı.
```

### R08 — Touch Target Küçük

```
KURAL: Minimum touch target 44×44px.
KONTROL: min-h-[44px] min-w-[44px] veya h-10+ w-10+ var mı?
```

### R09 — Viewport Height

```
KURAL: 100vh mobil browser UI'ını hesaba katmaz.
BEKLENEN: 100dvh (dynamic viewport height)
```

### R10 — Nested Button/Link

```
KURAL: <button> içinde <a> veya <a> içinde <button> olmamalı.
ÇÖZÜM: asChild prop kullan.
KÖTÜ: <button><Link href="...">...</Link></button>
İYİ: <Button asChild><Link href="...">...</Link></Button>
```

### R11 — forwardRef Eksik

```
KURAL: Radix primitive'leri wrap eden component'ler forwardRef kullanmalı.
NEDEN: asChild composition çalışmaz.
```

### R12 — Framer Motion Portal

```
KURAL: Animasyonlu Radix component'leri forceMount + AnimatePresence kullanmalı.
NEDEN: Portal content unmount'u animasyonu keser.
```

### R13 — any Tipi

```
KURAL: TypeScript'te any tipi sadece geçici geliştirme aşamasında kabul edilir.
BEKLENEN: Explicit tip annotation veya unknown + type guard
```

### R14 — Shimmer / Loading State Eksik

```
KURAL: Async data yükleyen her bölümde skeleton veya loading state olmalı.
KONTROL: Suspense boundary veya isLoading state var mı?
```

### R15 — Responsive Breakpoint Eksik

```
KURAL: Sadece bir viewport için optimize edilmiş layout kabul edilmez.
KONTROL: sm:, md:, lg: class'ları mantıklı şekilde kullanılıyor mu?
KONTROL: overflow-x yatay scroll yaratıyor mu?
```

### R16 — prefers-reduced-motion

```
KURAL: Tüm animasyonlar motion-safe: prefix ile korunmalı veya
       @media (prefers-reduced-motion: reduce) ile devre dışı bırakılmalı.
```

### R17 — Component Token Kullanımı

```
KURAL: Semantic token yerine primitive token kullanmak context bağımsızlığı yaratır.
KÖTÜ: var(--color-blue-500) doğrudan component stilinde
İYİ: var(--primary) veya tailwind semantic class
```

### R18 — Image Alt Text

```
KURAL: Bilgi içeren görsellerde açıklayıcı alt text olmalı.
KURAL: Dekoratif görsellerde alt="" (boş string, attribute olmalı).
KONTROL: alt attribute tamamen eksik mi?
```

---

## Output Format

Her bulgu şu formatla raporlanmalı:

```
[R-KURAL-NO] dosya/yolu:satır_no
  SORUN: Kısa açıklama
  MEVCUT: Sorunlu kod snippet'i
  BEKLENEN: Düzeltilmiş hali
  ÖNCELİK: Kritik | Yüksek | Orta | Düşük
```

**Öncelik Tanımları**:

- **Kritik**: Accessibility kırık, dark mode tamamen bozuk, console error
- **Yüksek**: WCAG AA uyumsuzluk, hardcoded renk, generic error mesajı
- **Orta**: Responsive eksiklik, UX copy kalitesi, loading state eksik
- **Düşük**: Minor token tutarsızlığı, stil iyileştirmesi

---

## Özet Rapor Şablonu

```
## UI Audit Sonuçları — [Component/Sayfa Adı]

### Özet
- Toplam bulgu: X
- Kritik: X | Yüksek: X | Orta: X | Düşük: X

### Kritik Bulgular
[Listele]

### Yüksek Öncelikli Bulgular
[Listele]

### Orta Öncelikli Bulgular
[Listele]

### Genel Değerlendirme
[1-3 cümle genel durum]

### Önerilen İlk 3 Aksiyon
1. ...
2. ...
3. ...
```

---

## CodeGuard Güvenlik Kontrol Listesi (TypeScript/Next.js)

UI audit sırasında şu güvenlik kuralları da kontrol edilmeli:

### R19 — Input Validation Eksik

```
KURAL: Kullanıcıdan gelen her veri validate edilmeli (Zod schema zorunlu)
KONTROL: Form submit handler'ında zodResolver var mı?
BEKLENEN: zodResolver(schema) ile useForm
```

### R20 — Hardcoded Secret

```
KURAL: API key, token, password kod içinde olmamalı
KONTROL: String literallerde 'sk_', 'AIza', 'Bearer ' gibi pattern
BEKLENEN: process.env.VARIABLE_NAME
```

### R21 — dangerouslySetInnerHTML

```
KURAL: Kullanıcı içeriği HTML olarak render edilmemeli
KONTROL: dangerouslySetInnerHTML kullanımı
BEKLENEN: DOMPurify sanitize + dangerouslySetInnerHTML, veya react-markdown
```

### R22 — localStorage'da Sensitive Data

```
KURAL: Token, şifre, kişisel veri localStorage'a yazılmamalı
KONTROL: localStorage.setItem çağrıları
BEKLENEN: Sensitive data httpOnly cookie veya memory'de
```

### R23 — console.log ile Sensitive Data

```
KURAL: Kullanıcı verisi, token, hata detayı console'a yazılmamalı
KONTROL: console.log, console.error çağrılarının içeriği
BEKLENEN: Structured logging, PII olmadan
```

### R24 — Missing Error Boundary

```
KURAL: AI veya async content içeren her feature error boundary ile sarılmalı
KONTROL: Suspense + ErrorBoundary birlikte var mı?
BEKLENEN: <ErrorBoundary fallback={...}><Suspense>...</Suspense></ErrorBoundary>
```

---

## CodeGuard Genişletilmiş Güvenlik Denetimi

### R25 — XSS: Kullanıcı HTML'i Sanitize Edilmemiş

```
KURAL: Kullanıcıdan gelen HTML render edilecekse DOMPurify zorunlu
KONTROL: dangerouslySetInnerHTML kullanımı + DOMPurify.sanitize() var mı?
BEKLENEN: DOMPurify.sanitize(userHtml, { ALLOWED_TAGS: [...] })
```

### R26 — Auth: httpOnly Cookie Kontrolü

```
KURAL: Token localStorage'da saklanmamalı
KONTROL: localStorage.setItem('token', ...) veya localStorage.setItem('auth', ...)
BEKLENEN: httpOnly cookie (Firebase session cookie mimarisi)
```

### R27 — Rate Limiting Eksik (AI Endpoint)

```
KURAL: AI çağrısı içeren route'larda rate limiting zorunlu
KONTROL: /api/chat, /api/cv, /api/ats route'larında rate limit middleware var mı?
BEKLENEN: Rate limiting uygulanmış, 429 response var
```

### R28 — SQL/Firestore Injection

```
KURAL: Firestore query'lerine kullanıcı inputu doğrudan geçirilmemeli
KONTROL: .where(userInput, ...) pattern
BEKLENEN: Input Zod ile validate edildikten sonra query'ye geç
```

### R29 — Log'da PII/Sensitive Data

```
KURAL: Email, ad, şifre, token log'a yazılmamalı
KONTROL: console.log(user), console.log(token), console.log(password)
BEKLENEN: Yalnızca userId veya anonim identifier
```

### R30 — CORS: Wildcard Origin

```
KURAL: Production'da Access-Control-Allow-Origin: * yasak
KONTROL: cors({ origin: '*' }) veya header'da wildcard
BEKLENEN: Spesifik domain listesi
```

---

## Vercel Web Interface Guidelines Denetimi

A�ağıdaki kurallar Vercel'in Web Interface Guidelines'ından alınmıştır. UI audit yaparken bu kuralları da uygula:

### Erişilebilirlik (WCAG AA)

- Tüm interactive element'lerde visible focus state
- Icon-only butonlarda `aria-label` zorunlu
- Form input'larında explicit `<label>` (for/id eşleşmesi)
- Renk tek başına anlam taşımamalı (ikon + metin de ekle)
- Minimum touch target: 44×44px

### Performans

- `<img>` yerine `next/image` kullan
- Above-fold görsel için `priority` prop
- Ağır component'ler `dynamic()` ile lazy load edilmeli

### Etkileşim

- Hover-only kritik aksiyonlar yasak (mobilde hover yok)
- Button loading state'i (`disabled` + spinner)
- Form submit sırasında double-submit engelle

### Responsive

- 320px'de yatay scroll olmamalı
- `100vh` yerine `100dvh` (mobil browser toolbar)
- Mobile-first breakpoint yaklaşımı

---

## Frontend Code Review Bulgu Formatı

Audit bulgularını şu formatta raporla:

```markdown
# Frontend Code Review

Found <N> urgent issues:

## 1 <kısa başlık>

FilePath: <dosya:satır>
<ilgili kod snippet>

### Suggested fix

<kısa düzeltme açıklaması>

---

Found <M> suggestions:

## 1 <kısa başlık>

FilePath: <dosya:satır>
<ilgili kod snippet>

### Suggested fix

<kısa düzeltme açıklaması>
```

**Öncelik:**

- **Urgent**: Güvenlik, auth bypass, XSS, KRİTİK fonksiyon bozukluğu
- **Suggestion**: Performans, erişilebilirlik, UX iyileştirme

10+ bulgu varsa "10+ urgent issues" olarak özetle, ilk 10'u göster.

---

## AI/LLM Özgün Güvenlik Kuralları

### R31 — Prompt Injection Koruması

```
KURAL: Kullanıcı inputu AI prompt'a doğrudan eklenmemeli
KONTROL: `${userInput}` şeklinde prompt string interpolation
BEKLENEN: Input validate + sanitize edilmeli, sistem prompt'u ayrı tutulmalı

// ❌ Tehlikeli
const prompt = `Kullanıcı CV'si: ${userProvidedText}. Analiz et.`

// ✅ Güvenli
const prompt = `CV Analizi:\n\n${sanitizeCvText(validatedInput)}`
```

### R32 — AI Çıktısı Sanitize

```
KURAL: AI'dan dönen içerik doğrudan HTML'e render edilmemeli
KONTROL: <div dangerouslySetInnerHTML={{ __html: aiResponse }} />
BEKLENEN: DOMPurify.sanitize() veya react-markdown ile render
```

### R33 — Structured Prompt Pattern

```
KURAL: AI çağrıları sistem prompt + kullanıcı input ayrımını korumalı
KONTROL: AI servis fonksiyonlarında messages array yapısı
BEKLENEN:
  { role: 'system', content: SYSTEM_PROMPT }
  { role: 'user', content: sanitizedUserInput }
```

---

## Review Comment Formatı (Code Review İçin)

Blocking ve suggestion'ları renk kodlamasıyla ayır:

```markdown
🔴 BLOCKING: Auth bypass açığı — route handler'da session kontrolü yok
🟡 SUGGESTION: useCallback eklenebilir — her render'da yeni referans oluşuyor
🟢 NIT: const tercih et — let değişken yeniden atanmıyor
❓ SORU: userId null ise burada ne oluyor?
```

| İşaret | Anlam            | Aksiyon                        |
| ------ | ---------------- | ------------------------------ |
| 🔴     | Merge blocker    | Düzeltilmeden geçilemez        |
| 🟡     | Önemli öneri     | Düzeltilmesi şiddetle önerilir |
| 🟢     | Küçük tercih     | Sonraki PR'a bırakılabilir     |
| ❓     | Açıklama gerekli | Yanıtlanmalı                   |

---

### R34 — SVG Animasyon Wrapper Eksik

```
KURAL: Animasyonlu SVG elementler doğrudan animate edilmemeli — div wrapper gerekli
NEDEN: Çoğu tarayıcı SVG elementleri için hardware acceleration desteği sınırlıdır
KONTROL: <svg className="animate-spin"> veya <svg className="transition-transform">
BEKLENEN: <div className="animate-spin"><svg>...</svg></div>

// ❌ Doğrudan SVG animasyonu
<svg className="animate-spin" viewBox="0 0 24 24">
  <path d="..." />
</svg>

// ✅ Wrapper div ile GPU acceleration
<div className="animate-spin">
  <svg viewBox="0 0 24 24">
    <path d="..." />
  </svg>
</div>
```

Bu kural tüm CSS transform ve transition'lar için geçerlidir: `animate-*`, `transition-*`, `hover:scale-*`, `hover:rotate-*`.

---

### R35 — shadcn/ui Component Conventions

```
KURAL: shadcn/ui bileşenlerinde import sırası ve tip tanımı konvansiyonuna uyulmalı
KONTROL: components/ui/ altındaki dosyalarda import organizasyonu ve ref handling

Import sırası (zorunlu):
  1. External kütüphaneler (class-variance-authority, @radix-ui)
  2. Internal utils (@/lib/utils)
  3. Base component alias (@/components/ui/component)

// ✅ Doğru tip tanımı — named export interface
export interface CvCardProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof cardVariants> {
  asChild?: boolean
}

// ✅ React 19 ref pattern (forwardRef kaldırıldı)
function CvCard({ ref, className, ...props }: CvCardProps & { ref?: React.Ref<HTMLDivElement> }) {
  return <div ref={ref} className={cn(cardVariants({ className }))} {...props} />
}

// ❌ Eski forwardRef pattern (yeni bileşenlerde kullanma)
const CvCard = React.forwardRef<HTMLDivElement, CvCardProps>(...)
```

```
KURAL: CVA variants'ı olan bileşenlerde variant override sadece h-* ve w-* için kabul edilir
BEKLENEN: className prop, token dışı renk veya spacing override içermemeli
```
