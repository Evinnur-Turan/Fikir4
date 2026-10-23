# 🔄 Modern ETL Pipeline: Belgelerden ve Web'den Veri Çekme

Yerel belgelerden (**Excel, CSV, PDF, TXT**) ve web kaynaklarından (**HTML Web Scraping, JSON/API**) veri çeken; bu veriyi temizleyip standart bir formata oturtan ve **Excel/CSV** olarak kaydeden, nesne yönelimli (OOP) bir **ETL (Extract, Transform, Load)** projesi.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20processing-150458?logo=pandas&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-uyumlu-20BEFF?logo=kaggle&logoColor=white)
![Stil](https://img.shields.io/badge/kod-clean%20code%20%7C%20SOLID-success)

---

## 📌 İçindekiler

- [Özellikler](#-özellikler)
- [Mimari](#-mimari)
- [Kurulum](#-kurulum)
- [Hızlı Başlangıç](#-hızlı-başlangıç)
- [Kullanım Örnekleri](#-kullanım-örnekleri)
- [Kaggle'da Çalıştırma](#-kaggleda-çalıştırma)
- [Örnek Çıktı](#-örnek-çıktı)
- [Hata Yönetimi](#-hata-yönetimi)
- [Proje Yapısı](#-proje-yapısı)
- [Geliştirme Fikirleri](#-geliştirme-fikirleri)
- [Lisans](#-lisans)

---

## ✨ Özellikler

| Alan | Detay |
|------|-------|
| **Yerel belgeler** | CSV, Excel (`.xlsx`/`.xls`), TXT (özel ayraç desteğiyle), PDF (tablo veya düz metin) |
| **Web kaynakları** | JSON/API uç noktaları, HTML tabloları, CSS seçiciyle öğe çekme |
| **Bot engellerine karşı** | Klasik `requests` yerine **`cloudscraper`** kullanılır |
| **Hızlı HTML ayrıştırma** | `BeautifulSoup(html, "lxml")` |
| **Temizleme** | Fazla boşluk silme, boş/tekrarlı satır kaldırma, sütun adı standardizasyonu (`snake_case` veya `UPPERCASE`), otomatik veri tipi düzeltme |
| **Çıktı** | Excel (`.xlsx`) ve CSV (`utf-8-sig`: Excel'de Türkçe karakterler bozulmaz) |
| **Dayanıklılık** | Ayrıntılı `try-except` blokları ve anlaşılır **Türkçe log mesajları** |
| **Hazır test** | Dışarıdan dosya yüklemeden çalışan sahte CSV/TXT (`StringIO`) ve ücretsiz API testi |

---

## 🏗️ Mimari

Proje, **SOLID** prensiplerine uygun olarak, her biri tek bir işten sorumlu sınıflara ayrılmıştır.

```
┌───────────────┐     ┌──────────────────┐     ┌─────────────┐
│ DataExtractor │ ──► │ DataTransformer  │ ──► │  DataSaver  │
│   (Extract)   │     │   (Transform)    │     │    (Load)   │
└───────────────┘     └──────────────────┘     └─────────────┘
        ▲                      ▲                      ▲
        └──────────────────────┴──────────────────────┘
                         ETLPipeline
                       (koordinatör)
```

| Sınıf | Sorumluluk |
|-------|-----------|
| `DataExtractor` | Belge okuma (`read_csv`, `read_excel`, `read_txt`, `read_pdf`) ve web'den veri çekme (`fetch_json`, `scrape_html_table`, `scrape_html_elements`) |
| `DataTransformer` | Boşluk temizleme, sütun adı standardizasyonu, tekrar/boş satır temizliği, tip dönüşümü |
| `DataSaver` | Veriyi `save_csv` / `save_excel` ile diske yazma |
| `ETLPipeline` | Üç sınıfı **bağımlılık enjeksiyonu** ile alıp akışı yönetir (Dependency Inversion) |

---

## ⚙️ Kurulum

```bash
# 1) Depoyu klonlayın (kullanici-adi kısmını kendi GitHub adınızla değiştirin)
git clone https://github.com/kullanici-adi/etl-pipeline.git
cd etl-pipeline

# 2) (Önerilir) Sanal ortam oluşturun
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3) Bağımlılıkları kurun
pip install cloudscraper beautifulsoup4 lxml pandas openpyxl pdfplumber
```

> `pdfplumber` yalnızca PDF okurken gereklidir. Diğer işlevler onsuz da çalışır.

İsterseniz bağımlılıkları `requirements.txt` dosyasına yazabilirsiniz:

```text
cloudscraper
beautifulsoup4
lxml
pandas
openpyxl
pdfplumber
```

---

## 🚀 Hızlı Başlangıç

```bash
python etl_pipeline.py
```

Bu komut, dosyanın sonundaki `__main__` bloğunda bulunan **3 test senaryosunu** çalıştırır:

1. **API testi:** `jsonplaceholder.typicode.com/users` adresinden JSON verisi çekilir.
2. **Sahte CSV testi:** `StringIO` ile kod içinde oluşturulan CSV, gerçek bir dosya gibi okunur.
3. **Sahte TXT testi:** `;` ayraçlı bir TXT metni `StringIO` ile okunur.

Çıktılar `output/` klasörüne yazılır.

---

## 🧪 Kullanım Örnekleri

### Kendi CSV dosyanızı işleme

```python
from etl_pipeline import DataExtractor, DataTransformer, DataSaver, ETLPipeline

extractor = DataExtractor()
transformer = DataTransformer(case_style="snake")   # "upper" -> BÜYÜK HARF sütun adları
saver = DataSaver(output_dir="output")
pipeline = ETLPipeline(extractor, transformer, saver)

ham_veri = extractor.read_csv("veriler.csv")
pipeline.run(ham_veri, name="temiz_veriler")
```

### Excel ve PDF okuma

```python
excel_df = extractor.read_excel("rapor.xlsx", sheet_name=0)
pdf_df = extractor.read_pdf("fatura.pdf")

pipeline.run(excel_df, name="excel_cikti")
pipeline.run(pdf_df, name="pdf_cikti", save_csv=False)   # yalnızca Excel olarak kaydet
```

### Web sitesinden HTML tablosu çekme

```python
tablo = extractor.scrape_html_table("https://ornek-site.com/tablo-sayfasi", table_index=0)
pipeline.run(tablo, name="web_tablosu")
```

### CSS seçicilerle veri çekme

```python
urunler = extractor.scrape_html_elements(
    "https://ornek-site.com/urunler",
    selectors={"baslik": "h2.title", "fiyat": "span.price"},
)
pipeline.run(urunler, name="urunler")
```

### JSON / API

```python
api_verisi = extractor.fetch_json("https://jsonplaceholder.typicode.com/posts")
pipeline.run(api_verisi, name="gonderiler")
```

---

## 📓 Kaggle'da Çalıştırma

1. Yeni bir Kaggle Notebook açın ve `etl_pipeline.py` içeriğini bir hücreye yapıştırın.
2. Sağ paneldeki **Settings → Internet** seçeneğini **On** yapın. Web scraping ve API testleri bu ayar olmadan çalışmaz.
3. Gerekirse ilk hücrede eksik kütüphaneleri kurun:
   ```python
   !pip install cloudscraper pdfplumber
   ```
4. Hücreyi çalıştırın. Sahte CSV/TXT testleri dışarıdan dosya yüklemeden de çalışır.

---

## 📊 Örnek Çıktı

Test 2'deki sahte CSV (boşluklu, tekrarlı ve boş satırlı), dönüşümden sonra şu hale gelir:

| ad_soyad | sehir | yas | maas_tl | kayit_tarihi |
|----------|-------|-----|---------|--------------|
| Ayşe Yılmaz | İzmir | 25.0 | 15000.5 | 2024-01-15 |
| Mehmet Öztürk | Ankara | 31.0 | 18250.0 | 2024-02-20 |
| Zeynep Çelik | İstanbul | NaN | 21000.0 | 2024-03-05 |

Yapılan dönüşümler:
- `"Ad Soyad "` → `ad_soyad`, `"Maaş (TL)"` → `maas_tl` (Türkçe karakter ve boşluk temizliği)
- Satır içi fazla boşluklar silindi (`"  Ayşe   Yılmaz "` → `"Ayşe Yılmaz"`)
- Tekrar eden satır ve tamamen boş satır kaldırıldı (5 → 3 satır)
- `"15000,50"` gibi virgüllü metinler sayıya, `kayit_tarihi` sütunu tarih tipine çevrildi

---

## 🛡️ Hata Yönetimi

Program hata durumunda çökmek yerine terminale açıklayıcı bir Türkçe mesaj yazar ve boş bir `DataFrame` döndürür:

| Durum | Davranış |
|-------|----------|
| İnternet kopması | `İnternet bağlantısı kurulamadı...` |
| Zaman aşımı | `Zaman aşımı: ... yanıt vermedi.` |
| HTTP hataları (404, 500 vb.) | `Sunucu hata döndürdü (HTTP ...)` |
| Geçersiz JSON | `Yanıt geçerli bir JSON değil` |
| Bozuk/boş/bulunamayan dosya | Dosya türüne özel hata mesajı |
| Eksik HTML etiketleri | Eksik hücreler doldurulur, başlık uyuşmazlığında sütunlar otomatik adlandırılır |
| Dosya açık/yazma izni yok | `Yazma izni yok veya dosya açık` |

Örnek log çıktısı:

```text
15:44:42 | INFO     | Extractor | CSV başarıyla okundu: 5 satır, 5 sütun.
15:44:42 | INFO     | Transformer | Sütun adları standartlaştırıldı: ['ad_soyad', 'sehir', 'yas', 'maas_tl', 'kayit_tarihi']
15:44:42 | INFO     | Transformer | Boş/tekrarlı satırlar temizlendi: 5 -> 3 satır.
15:44:42 | INFO     | Saver | CSV kaydedildi: output/yerel_csv_test.csv
```

---

## 📁 Proje Yapısı

```text
etl-pipeline/
├── etl_pipeline.py     # Tüm sınıflar ve test senaryoları
├── requirements.txt    # Bağımlılıklar
├── README.md           # Bu dosya
└── output/             # Çalıştırınca oluşan CSV/Excel çıktıları
```

---

## 💡 Geliştirme Fikirleri

- Sayfalama (pagination) destekli web scraping
- Komut satırı arayüzü (`argparse`) ile dosya yolu ve çıktı türü seçimi
- `unittest` / `pytest` ile otomatik testler
- SQLite veya PostgreSQL'e kaydetme için yeni bir `DatabaseSaver` sınıfı
- Veri doğrulama (şema kontrolü) adımı

---

## 📄 Lisans

Bu proje eğitim amaçlı geliştirilmiştir. İstediğiniz lisansı (örneğin MIT) `LICENSE` dosyası olarak ekleyebilirsiniz.

---

⭐ Projeyi beğendiyseniz GitHub'da yıldız vermeyi unutmayın!
