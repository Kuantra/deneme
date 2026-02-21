# Akıllı Mobil Doküman Asistanı

Bu doküman, şu ürün fikrini **uygulanabilir bir MVP yol haritasına** çevirir:

> Kameradan fotoğraf çekerek veya dosya yükleyerek metin içeriğini okuyabilen, içerikleri gruplandırabilen, benzer konuları tek klasörde toplayabilen ve gerektiğinde içeriğe göre cevap üretip ilgili yere gönderebilen mobil uygulama.

## 1) Hedef (MVP)
8 haftada çalışan bir sürüm:
- Görsel/PDF yükleme + kamera çekimi
- OCR ile metin çıkarma
- Konu bazlı klasörleme
- Doküman içeriğine dayalı soru-cevap
- E-posta taslağı üretip gönderime hazırlama

## 2) Teknoloji Seçimi (Önerilen)
- **Mobil:** Flutter
- **Backend:** FastAPI (Python)
- **DB:** PostgreSQL + pgvector
- **Dosya Depolama:** S3 uyumlu storage
- **OCR:** Google Vision (başlangıç), Tesseract (offline alternatif)
- **LLM:** özet, etiketleme, soru-cevap, taslak cevap

> Not: İsterseniz aynı mimariyi React Native + NestJS ile de uygulayabilirsiniz.

## 3) Önerilen Klasör Yapısı
```text
repo_adi/
  mobile/                 # Flutter uygulaması
  backend/
    app/
      api/
      services/
      models/
      workers/
    tests/
  infra/
    docker-compose.yml
  docs/
    prd.md
    api-spec.md
```

## 4) Kurulum (İstenen adımlar dahil)
```bash
git clone REPO_LINK
cd repo_adi
```

Ardından (öneri):
```bash
# backend
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# mobile
cd ../mobile
flutter pub get
```

## 5) Modül Bazlı İş Akışı
1. **Ingestion**
   - Kamera çekimi / dosya yükleme
   - Dosyayı obje depoya yaz, metadata’yı DB’ye kaydet
2. **OCR Pipeline**
   - Görsel ön işleme (crop/deskew/contrast)
   - OCR çalıştır, ham metni sakla
3. **Sınıflandırma & Gruplama**
   - Etiket üret (fatura, sözleşme, not...)
   - Embedding üret, benzer dokümanları aynı klasöre bağla
4. **Q&A / Yanıt Üretimi**
   - Kullanıcı sorusunu doküman context’i ile yanıtla
   - E-posta/mesaj taslağı oluştur
5. **Human-in-the-loop**
   - Kullanıcı onayı olmadan otomatik gönderim yapma

## 6) Örnek API Taslağı
- `POST /v1/documents/upload` → dosya yükleme
- `POST /v1/documents/{id}/ocr` → OCR tetikleme
- `POST /v1/documents/{id}/classify` → etiketleme
- `GET /v1/folders` → konu klasörleri
- `POST /v1/qa/ask` → doküman içeriğine göre cevap
- `POST /v1/messages/draft` → e-posta/mesaj taslağı

## 7) Veritabanı Çekirdeği (MVP)
- `users(id, email, role, created_at)`
- `documents(id, user_id, file_url, raw_text, created_at)`
- `document_embeddings(id, document_id, vector)`
- `folders(id, user_id, name)`
- `document_folder_map(document_id, folder_id)`
- `message_drafts(id, document_id, channel, content, status)`

## 8) 8 Haftalık Plan
- **Hafta 1-2:** Proje iskeleti, auth, upload
- **Hafta 3-4:** OCR ve metin saklama
- **Hafta 5:** Etiketleme + klasörleme
- **Hafta 6:** Q&A ve özet üretimi
- **Hafta 7:** E-posta taslağı + onay akışı
- **Hafta 8:** Test, güvenlik kontrolleri, pilot

## 9) Güvenlik ve Uyum
- KVKK/GDPR uyumlu veri işleme
- PII maskeleme ve log sanitization
- At-rest + in-transit şifreleme
- RBAC (rol bazlı erişim)
- Audit log ve aksiyon izleme

## 10) Copilot’a Verilebilecek Net Prompt Örneği
Aşağıdaki prompt’u doğrudan kullanabilirsiniz:

```text
FastAPI ile bir backend başlat.
- /v1/documents/upload endpoint'i multipart file kabul etsin.
- Dosyayı local storage'a kaydetsin, PostgreSQL'e metadata yazsın.
- Ardından OCR job'u için bir queue mesajı oluştursun.
- Pydantic modelleri, pytest testleri ve OpenAPI açıklamaları eklensin.
```

---

İstersen bir sonraki adımda bu repo için:
1) `backend/` başlangıç kodunu,
2) örnek `docker-compose.yml` dosyasını,
3) ve ilk 3 endpoint'in çalışan implementasyonunu
hazırlayabilirim.
