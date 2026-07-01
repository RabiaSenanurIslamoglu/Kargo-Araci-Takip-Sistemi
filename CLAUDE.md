# Kargo Aracı Takip Sistemi

## Projenin amacı

Bu proje bir ürün olmaktan önce bir **öğrenme projesi**: kullanıcı (bilgisayar mühendisliği öğrencisi/adayı)
uçtan uca bir sistemi (backend + web + mobil + veritabanı) sıfırdan kurarak "büyük sistem nasıl kurulur"
zihniyetini kazanmak ve mülakatlara deneyime dayalı cevap verebilmek istiyor. Bu nedenle:

- Hız değil **anlayış** önceliklidir. Bir aşamayı atlamadan önce kullanıcı o aşamanın kontrol noktasına
  gelmiş olmalı.
- Kullanıcı bir kavrama takıldığında (JWT, WebSocket, ORM, normalizasyon vb.) kod yazmadan önce durup
  o kavramı gerçek problem üzerinden açıkla.
- Kod yazarken bile "neden böyle yaptık" sorusuna cevap verilebilir olsun — trade-off'ları kısaca belirt.
- Aşamaları atlayıp ileri gitme; kullanıcı onay vermeden bir sonraki aşamaya geçme.

## Yol haritası

Toplam tahmini süre ~10-14 hafta (yarı zamanlı). Her aşama bir öncekinin üzerine kurulu, sırayla ilerlenir.

### Aşama 0 — Temeller
HTTP, REST, istemci-sunucu ayrımı, temel SQL, Git temelleri, geliştirme ortamı kurulumu
(Python, Node.js, PostgreSQL, VS Code).
**Kontrol noktası:** GitHub deposu açık, PostgreSQL kurulu, `git commit` yapılabiliyor.

### Aşama 1 — Gereksinimler ve Mimari
Fonksiyonel/fonksiyonel olmayan gereksinimler, mimari diyagram, teknoloji seçimi (nedenleriyle),
STRIDE ile basit tehdit modeli.
**Kontrol noktası:** Tek sayfalık gereksinim + mimari doküman hazır.

### Aşama 2 — Veritabanı ve Backend Temeli
Veritabanı modeli (kullanıcılar, araçlar, görevler, konumlar), FastAPI kurulumu, SQLAlchemy ORM,
Alembic migrasyonları, CRUD uç noktaları, Pydantic ile girdi doğrulama.
**Kontrol noktası:** Postman/tarayıcı ile araç ekleyip listeleyebiliyor; veri PostgreSQL'e yazılıyor.

### Aşama 3 — Kimlik Doğrulama ve Yetkilendirme
Kayıt/giriş uç noktaları, bcrypt ile şifre hash'leme, JWT + refresh token, RBAC (Yönetici/Dispeçer/Sürücü),
her uç noktada yetki kontrolü.
**Kontrol noktası:** Girişsiz korunan uç noktalara erişilemiyor; roller yalnızca yetkili oldukları işi yapabiliyor.

### Aşama 4 — Gerçek Zamanlı Veri ve İş Mantığı
Konum güncelleme uç noktası, WebSocket ile canlı konum akışı, Redis (önbellek/anlık veri),
"bağlantı koptu" uyarı mantığı, görev atama.
**Kontrol noktası:** Aracın konumu güncellenince bağlı istemcilere anında yansıyor.

### Aşama 5 — Web Paneli
React + TypeScript + Vite, özellik bazlı klasör yapısı, TanStack Query, giriş ekranı + token yönetimi
(access token bellekte, refresh cookie'de), haritada canlı konumlar (WebSocket), yükleniyor/hata/başarılı
durumları, role göre arayüz.
**Kontrol noktası:** Tarayıcıdan giriş yapıp haritada araçları canlı izleyebiliyor.

### Aşama 6 — Mobil Uygulama
React Native, aynı API ile giriş akışı, sürücü görünümü (kendi görevi/konumu), cihaz konumunu backend'e
gönderme.
**Kontrol noktası:** Telefondan/emülatörden giriş yapıp sürücü ekranı görülebiliyor.

### Aşama 7 — Test, Güvenlik ve Dağıtım
pytest ile backend testleri, güvenlik gözden geçirmesi (girdi doğrulama, hata mesajı sızıntısı, secrets),
`.env` ile sır yönetimi, Docker + docker-compose (backend + PostgreSQL + Redis), loglama/izleme.
**Kontrol noktası:** `docker compose up` ile tüm sistem tek komutla çalışıyor; testler geçiyor.

### Aşama 8 — Cilalama ve Portföy
README (mimari diyagram, kurulum, ekran görüntüleri), temiz commit geçmişi, demo video/GIF,
"karşılaştığım zorluklar" notu, CV/LinkedIn, algoritma pratiğine giriş.
**Kontrol noktası:** Proje bir yabancıya 2 dakikada anlatılabiliyor.

### Paralel yürüyen işler
- Haftada 2-3 algoritma/veri yapısı sorusu (kolay-orta seviye).
- Her aşamada öğrenilen kavram için 3-4 satırlık öz not (mülakat öncesi tekrar için) —
  `docs/kavram-notlari.md` içinde birikecek.

## Şu anki durum

- **Aktif aşama:** Aşama 0 tamamlandı → Aşama 1'e geçiliyor.
- Geliştirme ortamı hazır: Node.js v24, PostgreSQL 17 (localhost:5432, superuser `postgres` / şifre `postgres`,
  yalnızca yerel geliştirme için), GitHub CLI, Git.
- Uzak depo: https://github.com/RabiaSenanurIslamoglu/Kargo-Araci-Takip-Sistemi (public)
- Henüz uygulama kodu yok; sıradaki adım gereksinim ve mimari dokümanı (Aşama 1).

## Teknoloji yığını (Aşama 1'de kesinleşecek, şimdilik roadmap varsayımı)

- **Backend:** Python, FastAPI, SQLAlchemy, Alembic, Pydantic
- **Veritabanı:** PostgreSQL, Redis (önbellek/gerçek zamanlı)
- **Web:** React, TypeScript, Vite, TanStack Query
- **Mobil:** React Native
- **Altyapı:** Docker, docker-compose

## Depo yapısı (kurulacak)

```
backend/    FastAPI servisi
web/        React web paneli
mobile/     React Native uygulaması
docs/       Gereksinim/mimari dokümanları, kavram notları
```

## Kurallar

- Her aşama sonunda kontrol noktasını birlikte doğrula, sonraki aşamaya kullanıcı onayı olmadan geçme.
- Sırlar (.env, API anahtarları vb.) asla commit edilmez.
- Yeni bir kavram/kütüphane kullanılmadan önce kullanıcıya neden seçildiği kısaca anlatılır.
