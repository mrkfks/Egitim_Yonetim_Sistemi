# Egitim_Yonetim_Sistemi
<div align="center">
	<h1>Eğitim Yönetim Sistemi</h1>
	<p><strong>Angular 20 + SSR + json-server tabanlı eğitim / kurs yönetim platformu</strong></p>
	<p>
		<img src="https://img.shields.io/badge/angular-20.x-dd0031?logo=angular&logoColor=white" />
		<img src="https://img.shields.io/badge/build-ssr%20enabled-blue" />
		<img src="https://img.shields.io/badge/state-signals-green" />
	</p>
</div>

## İçindekiler
- [İçindekiler](#i̇çindekiler)
- [Genel Bakış](#genel-bakış)
- [Öne Çıkan Özellikler](#öne-çıkan-özellikler)
- [Teknoloji Yığını](#teknoloji-yığını)
- [Ekran Görselleri](#ekran-görselleri)
- [Kurulum](#kurulum)
- [Çalıştırma / Komutlar](#çalıştırma--komutlar)
- [Mimari Notlar](#mimari-notlar)
- [Veri Modeli (db.json)](#veri-modeli-dbjson)
- [Guard ve Yetkilendirme](#guard-ve-yetkilendirme)
- [Önbellek ve Performans](#önbellek-ve-performans)
- [Arama Sistemi](#arama-sistemi)
- [Metrikler](#metrikler)
- [SSR \& Prerender](#ssr--prerender)
- [Güvenlik Sertleştirme](#güvenlik-sertleştirme)
- [Geliştirme Yol Haritası](#geliştirme-yol-haritası)
- [Katkı](#katkı)
- [Lisans](#lisans)

---

## Genel Bakış
Öğrenci ve eğitici rollerini destekleyen, kurs oluşturma / listeleme, ders yönetimi, yorum ekleme, profil metrikleri, kategori filtrasyonu ve canlı arama özelliklerine sahip modern bir Angular uygulaması. Backend prototipleme için `json-server` kullanır; gerçek API'ye geçişe hazır gevşek bağlı servis katmanı vardır.

## Öne Çıkan Özellikler
- Rol tabanlı gezinme: Öğrenci / Eğitici görünümü dinamik
- Kurs CRUD (eğitici paneli) + ders ekleme
- Kategori filtreleme (c1–c5: Frontend, Backend, Siber, Veritabanı, Sistem)
- Otomatik kategori → varsayılan resim eşlemesi
- Kurs detay: Ders listesi + yorumlar + yazar eşlemesi cache ile
- Yorum ekleme (öğrenci rolünde)
- Profil metrikleri (öğrenci: toplam kurs / süre / ₺, eğitici: kurs ve öğrenci sayısı)
- Global canlı arama (kullanıcı, kurs, ders, yorum)
- Eğitmenler sayfası (otomatik instructor filtre)
- Signals + computed ile state türetme (NgRx gerektirmeden hafif model)
- Basit TTL cache katmanı (User lookup speed-up)
- Lazy load + bütçe optimizasyonlu build
- Türkçe arayüz, erişilebilir label ve semantic hiyerarşi

## Teknoloji Yığını
- Angular 20 (standalone bileşenler, signals, server output mode)
- RxJS 7.8 (sınırlı: çoğunlukla HttpClient akışları)
- json-server (mock REST)
- Bootstrap 5 (seçici kullanım, özel sadeleştirilmiş CSS + utility)
- Express SSR server (Angular server bundle)

## Ekran Görselleri
> Aşağıdaki görseller placeholder niteliğindedir; `docs/screenshots` dizinine kendi çıktılarını ekleyebilirsiniz.

| Sayfa | Açıklama | Görsel |
|-------|----------|--------|
| Anasayfa | Kategorilere göre filtrelenebilir kurs listesi | ![Home](docs/screenshots/home.png) |
| Kurs Detay | Ders listesi + yorumlar | ![Course Detail](docs/screenshots/course-detail.png) |
| Eğitici Paneli | Kurs oluşturma / düzenleme formu | ![Edit Courses](docs/screenshots/edit-courses.png) |
| Profil | Öğrenci / Eğitici metrikleri | ![Profile](docs/screenshots/profile.png) |
| Arama | Canlı global arama sonuçları | ![Search](docs/screenshots/search.png) |
| Kayıt Ol | Rol seçimi + validasyon | ![Register](docs/screenshots/register.png) |

Hızlı oluşturmak için (Windows PowerShell):
```powershell
New-Item -ItemType Directory -Force docs/screenshots | Out-Null
'home','course-detail','edit-courses','profile','search','register' | ForEach-Object { New-Item -ItemType File -Force "docs/screenshots/$_.png" | Out-Null }
```

## Kurulum
```bash
git clone <repo-url>
cd Egitim_Yonetim_Sistemi
npm install
```

## Çalıştırma / Komutlar
| Komut | Açıklama |
|-------|----------|
| `npm run start` | Angular dev + json-server paralel (concurrently) |
| `npm run json-server` | Sadece mock API |
| `npm run build` | Production build (SSR output) |
| `npm test` | Unit test (Karma + Jasmine) |
| `npm run serve:ssr:Egitim_Yonetim_Sistemi` | Build sonrası SSR server çalıştır |

Geliştirme ortamı:
```bash
npm run start
# Frontend: http://localhost:4200
# API:      http://localhost:3000
```

## Mimari Notlar
- Tüm bileşenler standalone; modül ağaçları yok → daha düşük kavramsal yük.
- State yönetimi: local signal + derived computed.
- Guard'lar yönlendirme katmanında minimal sorumlulukla (auth / role / not-auth).
- API erişimi merkezi `Api` servisi ile; endpoint stringleri sade tutuldu.
- Yorum ve kullanıcı eşlemesi tek seferlik cache (O(1) id → kullanıcı). 
- Performans: Gereksiz değişim tespiti azaltmak için OnPush (standalone default) + signal.

## Veri Modeli (db.json)
| Koleksiyon | Alanlar (özet) | Açıklama |
|------------|----------------|----------|
| users | id, name, surname, role, profilePhoto, bio? | Öğrenci + Eğitici |
| categories | id, name | Kurs kategorileri |
| courses | id, title, categoryId, instructorId, price, durationHours, image | Kurs temel modeli |
| lessons | id, courseId, title, durationMin | Kurs dersleri |
| enrollments | id, userId, courseId | Many-to-many ilişki |
| comments | id, courseId, userId, text, createdAt | Kurs yorumları |

## Guard ve Yetkilendirme
| Guard | Amaç | Not |
|-------|------|-----|
| auth | Oturum gerektiren route koruması | Yoksa login yönlendirme |
| not-auth | Login/Register erişimini engelle | Oturum varsa root |
| role | Rol tabanlı erişim | `data.roles` kontrolü |

## Önbellek ve Performans
- `CacheService`: TTL + key based bellek.
- `UserLookupService`: Tek sefer kullanıcı listesi → map.
- Profil / kurs detayında tekrar eden HTTP çağrıları azaltıldı.

## Arama Sistemi
`SearchService` koleksiyonları ardışık veya paralel sorgulayarak tip etiketli sonuç döner: `user | course | lesson | comment`.

## Metrikler
Öğrenci: toplam kurs, süre (dakika), toplam ₺.
Eğitici: yayınlanan kurs sayısı, benzersiz öğrenci sayısı.
Tümü reactive `computed()` ile.

## SSR & Prerender
- `outputMode: server` + `server.ts` entry.
- Dinamik parametreli route'lar için `getPrerenderParams` (kurs detay) eklenebilir / genişletilebilir.
- SEO için sayfa ilk yanıt süresi iyileştirilir.

## Güvenlik Sertleştirme
- Parolalar şu an mock (hash'e uygun yapı önerildi).
- Sensitive alanlar localStorage'da saklanmıyor (yalnızca kullanıcı özeti).
- Potansiyel ileri adım: JWT + HttpOnly cookie, CSP, Rate limiting.

## Geliştirme Yol Haritası
| Aşama | Öneri |
|-------|-------|
| 1 | Parola hash + gerçek auth servisi |
| 2 | Yorum düzenleme / silme yetkisi |
| 3 | Ders ilerleme takibi (progress) |
| 4 | Bildirim / etkinlik akışı |
| 5 | Çoklu dil (i18n) modulasyonu |
| 6 | Resim upload (Signed URL) |

## Katkı
Fork → Branch → Commit → PR akışı. Issue açarken: kısa başlık + yeniden üretim adımları + beklenen / gerçekleşen davranış.

## Lisans
Bu proje MIT Lisansı ile sunulmaktadır. Ayrıntılar için bkz: [LICENSE](./LICENSE).

Telif Hakkı (c) 2025 Ömer Kafkas

---
<sub>Bu dokümantasyon Angular 20 standalone + signal mimari ilkeleri göz önünde bulundurularak hazırlanmıştır.</sub>
