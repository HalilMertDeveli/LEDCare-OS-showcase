# LEDCare OS — ITIL tabanlı teknik servis platformu

> **Vitrin deposu.** Kaynak kod özel (private) bir depoda tutulur; bu depo yalnızca ürünü, mimariyi ve ekran görüntülerini tanıtır.
> Görüntülerdeki şirket ve kişi adları deneme verisidir. Kod incelemesi için iletişime geçebilirsiniz.
>
> *English summary below.*

LEDCare OS, LED ekranlar için merkezden yönetilen bir teknik servis ve servis masası (service desk) platformudur. Müşteri arızayı bildirir;
servis masası kaydı ITIL terimleriyle sınıflandırır, önceliklendirir, SLA'ya bağlar ve gerekirse eskale eder. İş uzaktan bağlantıyla
ya da sahaya gönderilen teknisyenle çözülür. Tekrarlayan arızalar problem kaydında izlenir ve bilinen hata olarak belgelenir.

**Kapsam:** ASP.NET Core 10 API · Next.js web uygulaması (müşteri portalı, yönetim paneli, teknisyen ekranları) · Flutter mobil uygulama
(Android/iOS) · PostgreSQL 17 · SignalR ile anlık güncelleme · Firebase anlık bildirim.

## ITIL / ITSM özellikleri

| ITIL kavramı | Platformdaki karşılığı |
|---|---|
| **Servis talebi ve olay (incident)** | Her kayıt `Servis talebi` (planlı iş, ör. periyodik bakım) ya da `Olay` (beklenmeyen kesinti) olarak sınıflandırılır |
| **Öncelik matrisi** | Öncelik elle seçilmez: **etki × aciliyet** matrisinden P1–P4 hesaplanır |
| **Yanıt ve çözüm SLA'ları** | Önceliğe ve arıza türüne göre eşleşen politika; ilk yanıt ve çözüm saatleri, "riskte" / "aşıldı" durumları, SLA takip listesi |
| **Fonksiyonel eskalasyon** | Kayıt bir üst destek hattına çıkar: 1. seviye servis masası → 2. seviye teknik destek → 3. seviye uzman |
| **Hiyerarşik eskalasyon** | Yönetim uyarılır; karar yetkisi gerektiren durumlar için (kaynak, müşteri iletişimi) |
| **Otomatik eskalasyon kuralları** | "İlk yanıt aşıldı → 2. seviye", "Çözüm aşıldı → 3. seviye", "İlk yanıt riskte → yönetim" gibi kurallar arka plan işinde bir kez tetiklenir |
| **Problem yönetimi** | Tekrarlayan olayların ortak nedeni `PRB-000001` kaydında izlenir; olaydan doğrudan problem açılır |
| **Bilinen hata (known error) ve KEDB** | Kök neden ve geçici çözüm yazılmadan kayıt "bilinen hata" olamaz; geçici çözüm bağlı tüm olaylarda görünür, aranabilir bilinen hata veritabanı |

Öncelik matrisi:

| Etki ↓ / Aciliyet → | Yüksek | Orta | Düşük |
|---|---|---|---|
| **Yüksek** | P1 Acil | P2 Yüksek | P3 Normal |
| **Orta** | P2 Yüksek | P3 Normal | P4 Düşük |
| **Düşük** | P3 Normal | P4 Düşük | P4 Düşük |

```mermaid
flowchart LR
    C[Müşteri talebi] --> SD[1. seviye: servis masası<br/>kayıt ve sınıflandırma]
    SD --> T{Olay mı,<br/>servis talebi mi?}
    T -->|Servis talebi| SR[Planlı iş:<br/>bakım, kurulum]
    T -->|Olay| P[Etki × aciliyet<br/>→ P1–P4]
    P --> SLA[SLA: yanıt ve çözüm hedefleri]
    SLA --> W[Uzaktan veya sahada çözüm]
    W -->|riskte / aşıldı| E{Eskalasyon}
    E -->|fonksiyonel| L2[2. seviye teknik destek<br/>3. seviye uzman]
    E -->|hiyerarşik| M[Yönetim]
    W -->|tekrarlayan arıza| PR[Problem kaydı]
    PR -->|kök neden + geçici çözüm| KE[Bilinen hata]
    KE --> W
```

### Ekran görüntüleri — servis masası

<table>
  <tr>
    <td width="50%"><img src="images/web-itil-classify.png" alt="ITIL sınıflandırma ve öncelik matrisi"><br><sub><b>Sınıflandırma.</b> Etki ve aciliyet seçilince öncelik matristen gelir (P1 · Acil) ve SLA politikası yeniden eşleşir.</sub></td>
    <td width="50%"><img src="images/web-itil-request.png" alt="SLA ve ITIL panelleri"><br><sub><b>SLA ve ITIL panelleri.</b> Yanıt ve çözüm saatleri, destek hattı, bağlı problem, eskalasyon geçmişi.</sub></td>
  </tr>
  <tr>
    <td><img src="images/web-problems-list.png" alt="Problem listesi"><br><sub><b>Problem yönetimi.</b> Açık problemler, bilinen hata veritabanı süzgeci, bağlı olay sayıları.</sub></td>
    <td><img src="images/web-problem-detail.png" alt="Bilinen hata kaydı"><br><sub><b>Bilinen hata.</b> Kök neden, geçici çözüm ve iki farklı müşterideki bağlı olaylar.</sub></td>
  </tr>
  <tr>
    <td><img src="images/web-sla-rules.png" alt="Eskalasyon kuralları"><br><sub><b>Eskalasyon kuralları.</b> Hiyerarşik uyarı ve hedef destek hattıyla fonksiyonel kurallar.</sub></td>
    <td><img src="images/web-admin-requests.png" alt="Gelen talepler"><br><sub><b>Gelen talepler.</b> Durum, öncelik, önem, arıza türü ve tarihe göre süzme.</sub></td>
  </tr>
</table>

### Ekran görüntüleri — operasyon, müşteri ve saha

<table>
  <tr>
    <td width="50%"><img src="images/web-admin-overview.png" alt="Operasyon paneli"><br><sub><b>Operasyon paneli.</b> Her sayı ilgili filtreli listeye götürür ve anlık güncellenir.</sub></td>
    <td width="50%"><img src="images/web-customer-portal.png" alt="Müşteri paneli"><br><sub><b>Müşteri paneli.</b> Açık talepler, randevular, mesajlar ve kayıtlar.</sub></td>
  </tr>
  <tr>
    <td><img src="images/web-home.jpg" alt="Tanıtım sitesi"><br><sub><b>Tanıtım sitesi.</b> Hesap açmadan arıza bildirme ve talep takibi.</sub></td>
    <td><img src="images/web-technician-home.png" alt="Teknisyen paneli"><br><sub><b>Teknisyen paneli.</b> Yalnızca atanan işler.</sub></td>
  </tr>
</table>

<table>
  <tr>
    <td width="33%"><img src="images/mobile-technician-job.jpg" alt="Mobil teknisyen iş ekranı"><br><sub><b>Mobil iş ekranı.</b> Çevrimdışıyken adım kuyrukta bekler; "Merkezden destek iste".</sub></td>
    <td width="33%"><img src="images/mobile-chat-offline.jpg" alt="Çevrimdışı sohbet"><br><sub><b>Servis ekibiyle sohbet.</b> Çevrimdışı mesajlar bağlantı gelince gönderilir.</sub></td>
    <td width="33%"><img src="images/mobile-push.jpg" alt="Anlık bildirim"><br><sub><b>Anlık bildirim.</b> Bildirim içerik taşımaz; uygulama ayrıntıyı API'den çeker.</sub></td>
  </tr>
</table>

## Mimari ve mühendislik

```mermaid
flowchart LR
  web["Web · Next.js<br/>portal, yönetim, teknisyen"] -- "çerez oturumu" --> api
  mobile["Mobil · Flutter<br/>çevrimdışı giden kutusu"] -- "bearer anahtar" --> api
  api["API · ASP.NET Core 10<br/>modüler monolit, REST + SignalR"] --> db[("PostgreSQL 17")]
  jobs["Arka plan işleri<br/>SLA eskalasyonu, hatırlatma,<br/>bildirim teslimi"] --> db
  api -- "değişiklik ipuçları" --> web
  api -- "değişiklik ipuçları" --> mobile
  jobs --> fcm["Firebase"] --> mobile
```

- **Tek doğru kaynak API:** Web ve mobil aynı API'yi kullanır; aralarında ayrı senkronizasyon yoktur. SignalR yalnızca kimlik taşıyan ipuçları gönderir, istemci veriyi REST'ten yeniden okur.
- **Yetkilendirme sunucuda:** Organizasyon kapsamı ve kayıt görünürlüğü her istekte API'de doğrulanır; kapsam dışı kayıtlar 404 döner. SLA, ITIL ve problem verisi yalnızca servis masasına (yönetici ve operasyon) görünür.
- **Tutarlılık:** İyimser eşzamanlılık (sürüm kontrolü, 409), aynı işlemde yazılan geçmiş, denetim ve bildirim kayıtları; eskalasyon her kural, seviye ve SLA revizyonu için yalnızca bir kez tetiklenir (benzersiz indeks).
- **Çevrimdışı mobil:** Kuyruktaki işlemler aynı `Idempotency-Key` ile yeniden gönderilir, kayıt çoğalmaz.
- **Test:** Gerçek PostgreSQL üzerinde (Testcontainers) 465 backend entegrasyon testi, web birim testleri ve Playwright uçtan uca testleri (ITIL ve SLA akışları dahil), Flutter widget testleri. API sözleşmesi OpenAPI'den dışa aktarılır ve CI'da sapma kontrolü yapılır.

| Katman | Teknoloji |
|---|---|
| API | ASP.NET Core 10 minimal API, EF Core 10, ASP.NET Identity, SignalR |
| Web | Next.js, React, TypeScript, Tailwind CSS, Vitest, Playwright |
| Mobil | Flutter, Riverpod, GoRouter, Dio, Firebase Cloud Messaging |
| Veri | PostgreSQL 17 (satır düzeyi güvenlik açık), Supabase Storage |
| Süreç | GitHub Actions, OpenAPI sözleşmesi, özellik dalları ve PR akışı |

---

## English summary

**LEDCare OS** is a service-desk and field-service platform for LED-screen technical support, built around **ITIL** practices.
This is a **showcase repository**: the source code is private; this repo presents the product, architecture and screenshots
(all names in screenshots are demo data).

- **Service requests vs. incidents**, classified by the service desk.
- **Priority matrix:** impact × urgency gives P1–P4; the matching **SLA policy** (first-response and resolution targets) follows automatically.
- **SLA monitoring** with at-risk and breached states and a watch list.
- **Functional escalation** (1st-line service desk → 2nd-line technical support → 3rd-line expert) and **hierarchical escalation** (management), manual with a reason or driven by SLA rules, with full history.
- **Problem management** with **known errors** (root cause + workaround required) and a searchable **known error database**; incidents link to problems and show the workaround.
- Stack: ASP.NET Core 10, EF Core, PostgreSQL 17, SignalR, Next.js + TypeScript, Flutter, Firebase; 465 backend integration tests on a real database, Playwright E2E and Flutter widget tests.

© Halil Mert Develi. Tüm hakları saklıdır / All rights reserved.
