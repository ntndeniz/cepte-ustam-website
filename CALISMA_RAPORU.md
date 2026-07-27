# Cepte Ustam — Proje Teslim ve Çalışma Raporu

*Tesisat ustaları için teklif, malzeme ve kazanç takip uygulaması — Flutter + Firebase*
Tarih: 24.07.2026

---

## 1. Genel Bilgi

Cepte Ustam; tesisat / sıhhi tesisat ustalarının **sahadan teklif hazırlaması**, malzeme listesi tutması, müşteri onaylarını yönetmesi ve **gelir-gider (muhasebe) takibi** yapması için geliştirilmiş bir mobil uygulamadır. İstemci tarafı Flutter ile, sunucu (backend) tarafı ise Google Firebase (BaaS) ile inşa edilmiştir. Hedef kullanıcılar çoğunlukla bodrum, şantiye gibi internetin zayıf olduğu ortamlarda çalıştığından, uygulama **çevrimdışı öncelikli (offline-first)** tasarlanmış; bağlantı geldiğinde veriler otomatik senkronlanır.

> Önemli iş kuralı: Malzeme fiyatları katalogda saklanmaz; her teklifte o günkü tedarikçi fiyatı elle girilir (fiyatlar piyasada sürekli değiştiği için). Tüm para değerleri hatasız hesap için kuruş bazlı tam sayı (int) tutulur.

## 2. Teknoloji Yığını

| Katman | Teknoloji / Sürüm |
|--------|-------------------|
| İstemci (Mobil & Web) | Flutter 3.44.6 · Dart 3.12.2 |
| Durum Yönetimi | Riverpod (flutter_riverpod 3.3.2) |
| Yönlendirme | go_router 17.3.0 |
| Backend (BaaS) | Firebase: Authentication, Cloud Firestore, Storage, App Check, Hosting |
| PDF Üretimi | pdf 3.13.0 + printing 5.15.0 |
| Diğer Paketler | share_plus, intl, image_picker, shared_preferences, http, path_provider |
| Hedef Platformlar | Android (imzalı APK), iOS (PWA), Web (PWA) |

## 3. Mimari Yapı

Proje, katmanların birbirinden ayrıldığı 4 katmanlı bir mimari izler:

**Sunum (UI / Ekranlar) → Durum (Riverpod Provider'lar) → Repository (arayüz + implementasyon) → Veri Kaynağı (Firestore / bellek-içi)**

- **Sunum:** Ekranlar ve widget'lar; her ekran için dolu / boş / yükleniyor / hata olmak üzere 4 durum.
- **Durum:** Riverpod provider'ları iş akışını ve UI durumunu yönetir.
- **Repository:** Her veri türü için bir arayüz (interface) + Firestore implementasyonu + geliştirme/test için bellek-içi (memory) implementasyonu.
- **Veri Kaynağı:** Cloud Firestore (çevrimdışı kalıcılık açık).

Repository deseni sayesinde uygulama, gerçek Firebase bağlantısı olmadan da (bellek-içi implementasyonla) çalıştırılıp test edilebilir.

| Metrik | Değer |
|--------|-------|
| Toplam Dart dosyası | 116 |
| Ekran (screen) | 27 |
| Veri modeli | 10 |
| Provider | 15 |
| Repository dosyası | 30 |
| Test dosyası | 5 |

## 4. Uygulama Kapsamı ve Özellikler

- **Kimlik doğrulama:** E-posta + şifre ve Google ile giriş; e-posta doğrulaması zorunlu.
- **Teklif sihirbazı:** Müşteri seç → malzeme ekle → (opsiyonel) fiyat gir → işçilik & kâr oranı → PDF.
- **Teklif durum makinesi:** Taslak → Fiyat Bekliyor → Fiyatlandı → Onay Bekliyor → Onaylandı → İş Tamamlandı → Tahsil Edildi (ayrıca Reddedildi / İptal).
- **Malzeme kataloğu:** Grup/kategori bazlı ürün yönetimi (fiyat tutmadan).
- **Müşteri & tedarikçi** yönetimi.
- **Muhasebe:** Gelir-gider, tahsilat ve net kâr; teklif tahsilatları otomatik gelire işlenir.
- **PDF teklif** oluşturma ve WhatsApp vb. ile paylaşma.
- **Destek sohbeti:** Kural tabanlı asistan; çözülemeyen talep e-posta ile ekibe iletilir (eskalasyon).
- **Çevrimdışı çalışma** ve bağlantı gelince otomatik senkronizasyon.

Navigasyon: alt sekmeler (Ana Sayfa · İşler · Malzemeler · Muhasebe), ortada çentikli (+) hızlı teklif butonu ve hamburger menü (Müşteriler, Tedarikçiler, Profilim, Yardım & Yasal). Animasyonlu açılış (splash) ve ilk kullanım tanıtımı (onboarding) bulunur.

## 5. Uygulanan Teknik Kurallar

| Kural | Uygulanışı |
|-------|------------|
| Çevrimdışı öncelikli | Firestore offline persistence; internetsiz kayıt, bağlantıda senkron. |
| Para birimi | Tüm tutarlar kuruş bazlı tam sayı (int) — kayan nokta hatası önlenir. |
| Katalog fiyatı | Katalogda fiyat saklanmaz; fiyat teklife özel girilir. |
| Veri izolasyonu | Firestore kuralları her kullanıcıyı yalnızca kendi ağacına (users/{uid}/**) kısıtlar. |
| Kimlik güvenliği | E-posta doğrulaması zorunlu; doğrulanmamış kullanıcı oturum açamaz. |
| Erişilebilirlik | Responsive düzen; ~48dp dokunma hedefleri. |
| Tutarlılık | Ana Sayfa ve Muhasebe toplamları aynı kaynaktan, tutarlı. |

## 6. Veri Modeli (Firestore)

Veriler kullanıcı bazlı ağaçta tutulur: **users/{uid}/** altında profil, teklifler, katalog, müşteriler, tedarikçiler, giderler, tahsilatlar, muhasebe defteri ve destek mesajları koleksiyonları. Her belge ilgili kullanıcıya bağlıdır; başka kullanıcıların verisine erişim güvenlik kurallarıyla engellenir.

## 7. Güvenlik

- **Firestore Security Rules:** Varsayılan her şey kapalı; yalnızca users/{uid}/** sahibi erişebilir.
- **Firebase App Check** ile arka uç isteklerinin doğrulanması.
- **Android imzalama:** APK, projeye özel release keystore ile imzalanır.
- **Sır yönetimi:** keystore, key.properties, servis hesabı anahtarları vb. sürüm kontrolüne (Git) hiç girmez.

## 8. Test ve Kalite

| Test / Kontrol | Sonuç |
|----------------|-------|
| TestSprite uçtan uca (E2E) test | 30 / 30 senaryo başarılı (Flutter web build üzerinde) |
| Birim & widget testleri (flutter test) | 22 test yeşil |
| Statik analiz (flutter analyze) | Temiz — yalnızca 4 bilgilendirme (info) uyarısı |

## 9. Yayın ve Teslim Durumu

| Platform / Öğe | Durum |
|----------------|-------|
| Android | İmzalı release APK (~63 MB), yeni logolu launcher ikonu |
| iOS | App Store yerine PWA — Safari "Ana Ekrana Ekle" (Apple sideload'a izin vermiyor) |
| Web / PWA | Firebase Hosting — https://cepte-ustam.web.app |
| Tanıtım Sitesi | Netlify — https://cepte-ustam.netlify.app (APK indirme + iOS kurulum) |
| Kaynak Kod | GitHub — uygulama deposu PRIVATE, tanıtım sitesi deposu PUBLIC |

## 10. Karşılaşılan Sorunlar ve Çözümleri

| Sorun | Çözüm |
|-------|-------|
| APK derlemede aapt2 "Link timed out" (bellek baskısı) | Release'te PNG crunch kapatıldı (isCrunchPngs=false), flutter clean yapıldı, Gradle heap 8G→2G düşürülerek swap baskısı azaltıldı. |
| git-lfs kurulu değilken global config'in onu zorunlu tutması git işlemlerini kilitliyordu | Depo bazında LFS filtreleri nötrlendi (filter.lfs.process boş, required=false). |
| Destek e-postalarının kullanıcıya ulaşmaması | (1) FormSubmit formu aktive edilmemişti; (2) mobilde Origin/Referer başlığı gitmediği için istek reddediliyordu → koda Referer başlığı eklendi; (3) profil boşken kimlik gitmiyordu → giriş e-postası fallback'i (currentEmail) eklendi. |
| Teklif detayından fiyat düzenleme yolu yoktu (TestSprite TC014) | Duruma duyarlı "Fiyatlandır / Tedarikçi Fiyatlarını Düzenle" butonu eklendi ve yeniden test edildi. |
| Varsayılan / eski logo kullanımı | Özgün çapraz-alet (anahtar + tornavida) logosu tasarlandı; app_logo.dart yeniden yazıldı, tüm PWA / Android / favicon ikonları üretildi. |

## 11. Mimari Kararlar ve Nedenleri

| Karar | Neden |
|-------|-------|
| Backend olarak Firebase (BaaS) | Tek geliştirici için sunucu yönetimi gerektirmeyen, hızlı ve ölçeklenebilir çözüm. |
| Çevrimdışı öncelikli tasarım | Hedef kullanıcılar internetsiz sahada (bodrum/şantiye) çalışıyor. |
| Repository deseni + bellek-içi impl. | Gerçek backend olmadan geliştirme ve test yapabilmek. |
| Riverpod ile durum yönetimi | Derleme-zamanı güvenli, test edilebilir ve sürdürülebilir durum yönetimi. |
| Para için kuruş bazlı tam sayı | Finansal hesaplarda kayan nokta (float) yuvarlama hatalarını önlemek. |
| iOS'ta PWA dağıtımı | Apple mağaza dışı IPA kurulumuna izin vermez; PWA ile mağazasız dağıtım sağlanır. |

## 12. Yapay Zeka Kullanımı

Proje sürecinde Anthropic Claude (Claude Code) bir hızlandırıcı/asistan olarak kullanıldı. Kullanıldığı başlıca aşamalar:

- Tıklanabilir prototip ve UX tasarımı,
- Mimari kararlar ve kod üretimi (Flutter/Dart, Firebase entegrasyonu),
- Logo ve ikon tasarımı ile üretimi,
- Test senaryoları (TestSprite) ve hata ayıklama,
- Yayınlama (Firebase Hosting, Netlify, GitHub, APK imzalama) ve teknik sorun çözümü (aapt2, git, e-posta).

Kullanılan tüm teknoloji, framework ve yöntemler geliştirici tarafından anlaşılmakta olup; yapay zeka, karar verici değil üretkenliği artıran bir araç olarak konumlandırılmıştır.

## 13. Sonuç

Cepte Ustam uygulaması tasarlanmış, geliştirilmiş, test edilmiş ve üç platformda (Android APK, iOS PWA ve Web) yayına alınmıştır. Uygulama, tanıtım sitesi ve kaynak kod depoları hazır olup proje **teslim edilebilir** durumdadır.
