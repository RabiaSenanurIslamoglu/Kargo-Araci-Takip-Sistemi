# Kavram Notları

Bu dosya, yol haritasının her aşamasında öğrenilen kavramların kısa özetlerini biriktirir.
Amaç mülakat öncesi hızlı tekrar yapabilmek. Her bölüm kendi başına okunabilir.

---

## Aşama 0 — Temeller

### 1) HTTP nasıl çalışır?

Web'de her şey aslında şu basit döngüden ibaret: **istemci bir istek (request) gönderir,
sunucu bir yanıt (response) döner.**

```mermaid
sequenceDiagram
    participant C as İstemci (Tarayıcı/Mobil Uygulama)
    participant S as Sunucu (Backend)
    C->>S: HTTP İstek (GET /araclar/5)
    S-->>C: HTTP Yanıt (200 OK + JSON veri)
```

Bir HTTP isteği üç parçadan oluşur:
- **Method + Path:** `GET /araclar/5` → "5 numaralı aracın bilgisini getir"
- **Header'lar:** Ek bilgi (`Authorization: Bearer <token>`, `Content-Type: application/json`)
- **Body (gövde):** POST/PUT gibi isteklerde gönderilen veri (genelde JSON)

**HTTP metodları — "hangi fiil ne işe yarar":**

| Method | Anlamı | Kargo projesi örneği |
|---|---|---|
| GET | Veri oku | Araç listesini getir |
| POST | Yeni veri oluştur | Yeni araç ekle |
| PUT/PATCH | Var olanı güncelle | Aracın konumunu güncelle |
| DELETE | Sil | Aracı sistemden çıkar |

**Durum kodları — sunucunun cevabı nasıl özetlediği:**

```mermaid
flowchart LR
    A["Durum Kodu"] --> B["2xx: Başarılı"]
    A --> C["4xx: İstemci hatası"]
    A --> D["5xx: Sunucu hatası"]
    B --> B1["200 OK — istek başarıyla işlendi"]
    B --> B2["201 Created — yeni kayıt oluşturuldu"]
    C --> C1["400 Bad Request — istek hatalı/eksik"]
    C --> C2["401 Unauthorized — giriş yapılmamış"]
    C --> C3["403 Forbidden — girişin var ama yetkin yok"]
    C --> C4["404 Not Found — kaynak yok"]
    D --> D1["500 Internal Server Error — sunucuda beklenmeyen hata"]
```

**Aklında kalsın:** 401 vs 403 mülakatlarda sık karıştırılır.
- **401** = "Sen kimsin, giriş yapmamışsın" (kimlik doğrulama eksik)
- **403** = "Seni tanıyorum ama bu işe yetkin yok" (yetkilendirme reddi)

### 2) REST API nedir?

REST, kaynakları (resource) URL'ler üzerinden, standart HTTP metodlarıyla yönetmenin bir
konvansiyonudur. "Uç nokta (endpoint)" dediğimiz şey = **URL + HTTP method** kombinasyonu.

```
GET    /araclar          → tüm araçları listele
GET    /araclar/5        → 5 numaralı aracı getir
POST   /araclar          → yeni araç oluştur
PUT    /araclar/5        → 5 numaralı aracı güncelle
DELETE /araclar/5        → 5 numaralı aracı sil
```

Neden bu kadar popüler:
- **Stateless (durumsuz):** Sunucu iki istek arasında istemciyle ilgili bir şey "hatırlamaz",
  her istek kendi başına yeterli bilgiyi taşır (örn. token header'da gelir).
- **Kaynak odaklı:** URL bir *isim* olmalı (`/araclar`), bir *fiil* değil (`/araclarigetir` gibi değil).
- **JSON ile konuşma:** İstek/yanıt gövdesi genelde JSON — hem web hem mobil hem başka bir backend
  aynı formatı okuyabilir.

Örnek JSON yanıtı:
```json
{
  "id": 5,
  "plaka": "34 ABC 123",
  "surucu_id": 12,
  "son_konum": { "lat": 41.015, "lon": 28.979 }
}
```

### 3) İstemci-sunucu ayrımı

```mermaid
flowchart LR
    W["Web Paneli (React)"] -- HTTP/WebSocket --> B["Backend (FastAPI)"]
    M["Mobil Uygulama (React Native)"] -- HTTP/WebSocket --> B
    B -- SQL --> DB[("PostgreSQL")]
    B -- önbellek --> R[("Redis")]
```

- **İstemcide (tarayıcı/telefon) çalışan:** Arayüz, kullanıcı etkileşimi, "görünüm" mantığı.
  İstemci kodunu herkes tarayıcının geliştirici araçlarından görebilir — **asla gizli/güvenilir
  bilgi tutmaz.**
- **Sunucuda çalışan:** Asıl iş mantığı, veritabanı erişimi, yetki kontrolü, şifre doğrulama.
  Güvenlik kararları **her zaman** sunucuda verilir; istemci sadece "iyi niyetli" arayüz sağlar.

**Neden önemli:** Bir sürücü uygulamasında "Sil" butonunu arayüzden gizlemek yetmez — sunucu
tarafında da "bu kullanıcı silme yetkisine sahip mi?" kontrolü olmalı. Yoksa biri tarayıcıdan
doğrudan `DELETE /araclar/5` isteği atarak butonu bypass edebilir.

**Aynı backend, çok istemci:** Kargo projesinde hem web paneli hem mobil uygulama **aynı**
FastAPI backend'ine konuşacak. Bu yüzden iş mantığını backend'de tek yerde tutmak, istemcilerde
tekrar etmemek kritik.

### 4) Temel SQL

İlişkisel veritabanı = tablolardan oluşan, aralarında **ilişki** kurulabilen bir yapı.

```mermaid
erDiagram
    KULLANICILAR ||--o{ ARACLAR : "surer"
    ARACLAR ||--o{ KONUMLAR : "gonderir"
    KULLANICILAR ||--o{ GOREVLER : "atanir"
    ARACLAR ||--o{ GOREVLER : "kullanilir"

    KULLANICILAR {
        int id PK
        string ad
        string email
        string rol
    }
    ARACLAR {
        int id PK
        string plaka
        int surucu_id FK
    }
    KONUMLAR {
        int id PK
        int arac_id FK
        float lat
        float lon
        datetime zaman
    }
    GOREVLER {
        int id PK
        int arac_id FK
        int surucu_id FK
        string durum
    }
```

- **Primary key (birincil anahtar):** Bir satırı tekil olarak tanımlayan sütun (`id`).
- **Foreign key (yabancı anahtar):** Başka bir tablonun primary key'ine referans veren sütun.
  `araclar.surucu_id` → `kullanicilar.id` demek "bu araç şu kullanıcıya ait".
- Bu referans sayesinde aynı veriyi (kullanıcı bilgisi) her satırda tekrar etmeyiz —
  buna kabaca **normalizasyon** denir.

**Temel sorgular:**
```sql
-- Tüm araçları listele
SELECT * FROM araclar;

-- Sadece belirli sürücünün araçlarını getir
SELECT * FROM araclar WHERE surucu_id = 12;

-- Yeni araç ekle
INSERT INTO araclar (plaka, surucu_id) VALUES ('34 ABC 123', 12);

-- Aracın son konumunu güncelle
UPDATE araclar SET son_konum_id = 88 WHERE id = 5;

-- Bir aracı sürücü bilgisiyle birlikte getir (JOIN)
SELECT araclar.plaka, kullanicilar.ad
FROM araclar
JOIN kullanicilar ON araclar.surucu_id = kullanicilar.id;
```

**JOIN mantığı:** İki tabloyu ortak bir sütun (genelde FK ↔ PK) üzerinden "yan yana" getirir.
Yukarıdaki örnek: "araçları, sürücülerinin adıyla birlikte listele."

---

*(Sonraki aşamalarda buraya JWT, WebSocket, ORM, RBAC gibi kavramlar eklenecek.)*
