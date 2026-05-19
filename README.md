readme_content = """
# 📰 Mynet Akıllı Haber Merkezi - Otonom RAG Sistemi

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688) ![Gradio](https://img.shields.io/badge/Gradio-UI-FF7C00) ![Gemini](https://img.shields.io/badge/Gemini-2.5_Flash-8E75B2) ![ChromaDB](https://img.shields.io/badge/ChromaDB-Vector_DB-FF4F8B)

Bu proje, internetteki haber kaynaklarından (RSS) otonom olarak veri toplayan, metinleri işleyen, kalıcı bir vektör veritabanına kaydeden ve kullanıcı sorularına **Gelişmiş Hibrit Arama (BM25 + Vektör + Cross-Encoder)** algoritmaları ve **Gemini 2.5 Flash** büyük dil modeli (LLM) ile yanıt veren, uçtan uca mimariye sahip bir **RAG (Retrieval-Augmented Generation)** asistanıdır.

---

## 🚀 Projenin Temel Özellikleri

- **Otonom Veri Hasadı (Daemon Engine):** Arka planda çalışan bir iş parçacığı (thread), her 25 dakikada bir RSS kaynaklarını tarayarak yeni haberleri sisteme dahil eder.
- **Çoklu Arama Stratejisi (Hybrid Search):** Geleneksel anahtar kelime araması (BM25) ile anlamsal vektör araması harmanlanarak **RRF (Reciprocal Rank Fusion)** yöntemiyle birleştirilir.
- **Cross-Encoder ile Re-Ranking:** Bulunan aday metinler, üst düzey bir yeniden sıralayıcı (`cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`) kullanılarak soruya olan gerçek uyumlarına göre yeniden puanlanır ve filtrelenir.
- **Dinamik Hafıza Yönetimi:** Uzayan sohbet geçmişleri sistem hafızasını şişirmemesi için LLM tarafından anlık olarak özetlenir (Memory Summarization).
- **Modüler ve Ölçeklenebilir Mimari:** İstemci-Sunucu mimarisi gözetilerek **FastAPI** ile bir backend oluşturulmuş, kullanıcı arayüzü ise **Gradio** ile ayrıştırılmıştır.

---

## 🧠 Sistem Mimarisi ve Modüller

Proje, iş mantığını birbirinden bağımsız ancak tam entegre modüllere ayırmıştır:

### 1. Veri Hazırlama ve Embedding (Katman 1 & 2)
* **Modül A (Ham Veri Hasadı):** Mynet ve çeşitli haber RSS'lerinden başlık, link, özet gibi ham verileri toplar. Çökme durumlarına karşı *Semantik Çapalar (Fallback)* içerir.
* **Modül B (İçerik Temizleme):** `BeautifulSoup` ve `requests` ile haber sitelerine giderek reklam ve gereksiz HTML etiketlerini ayıklar, saf metni çıkarır.
* **Modül C (Chunking):** `RecursiveCharacterTextSplitter` kullanarak uzun haber metinlerini LLM limitlerine uygun ve anlam bütünlüğünü bozmayacak şekilde (450 karakterlik) parçalara ayırır. Benzersiz ID'ler atayarak (Deduplication) kopyaları engeller.
* **Modül D (DNA Vektörleştirme):** `intfloat/multilingual-e5-large` modeli kullanılarak metinler 1024 boyutlu anlamsal vektörlere dönüştürülür.
* **Modül E (Veritabanı Senkronizasyonu):** Vektörler, meta verileriyle birlikte kalıcı **ChromaDB** sunucusuna işlenir.

### 2. İleri Düzey Geri Çağırma (Modül F)
* Sistem sadece vektör aramasına güvenmez. İçerisinde özel bir **BM25 Anahtar Kelime Motoru** barındırır.
* Kullanıcı sorusu hem BM25 hem de Vektör uzayında aranır. Sonuçlar RRF ile birleştirildikten sonra **Cross-Encoder** modeline sokularak hassas bir şekilde sıralanır (Re-Ranking). Sonuçlar içerisinden kopya başlıklar tekilleştirilerek LLM'e en taze ve zengin bağlam (context) sunulur.

### 3. API ve Sentez (Modül G)
* **FastAPI** üzerinde çalışan uç nokta (`/ask`), kullanıcıdan gelen soruyu alır, arama motoruna yönlendirir ve elde edilen bağlamı sohbet geçmişi ile harmanlayarak **Gemini 2.5 Flash** modeline iletir.
* Yanıtlar markdown formatında, kaynak linkleriyle birlikte profesyonel bir şekilde oluşturulur.

### 4. Kullanıcı Arayüzü (Gradio)
* Kullanıcının sistemle etkileşime girmesi için estetik, akıcı ve API'ye HTTP istekleri atan bir sohbet arayüzü sunar.

---

## 🛠️ Kurulum ve Çalıştırma (Google Colab Uyumlu)

Bu proje Google Colab ortamı için optimize edilmiştir. Kendi ortamınızda çalıştırmak için aşağıdaki adımları izleyin:

### Gereksinimler
Proje ortam değişkenlerinde (Colab Secrets) aşağıdaki anahtarların tanımlı olması gerekmektedir:
- `GOOGLE_API_KEY`: Gemini API kullanımı için.
- `NGROK_TOKEN`: FastAPI sunucusunu dışarıya açmak (tünellemek) için.
- *(Opsiyonel)* `HF_TOKEN`: HuggingFace modellerine erişim kotalarına takılmamak için.

### Adımlar
1. Depoyu klonlayın veya `.ipynb` dosyasını Google Colab'e yükleyin.
2. Gizli anahtarlarınızı (Secrets) ayarlayın.
3. Hücreleri sırasıyla (Yukarıdan Aşağıya) çalıştırın.
4. Arka plan motoru (Daemon) başlatıldığında sistem otonom olarak haberleri çekip veritabanını oluşturmaya başlayacaktır.
5. En alttaki **Gradio** hücresini çalıştırdığınızda oluşturulan public URL üzerinden sisteme erişebilir ve haber asistanıyla konuşmaya başlayabilirsiniz.

---

## 📚 Kullanılan Teknolojiler
- **Dil:** Python
- **LLM:** Google Gemini 2.5 Flash (`google-generativeai`)
- **Vektör Veritabanı:** ChromaDB
- **NLP & Arama:** Sentence-Transformers, Rank-BM25, LangChain Text Splitters
- **Web Kazıma:** BeautifulSoup4, Feedparser, Requests
- **Backend:** FastAPI, Uvicorn
- **Arayüz:** Gradio
- **Tünelleme:** Pyngrok
"""

with open("README.md", "w", encoding="utf-8") as f:
    f.write(readme_content)

print("✅ README.md dosyası başarıyla oluşturuldu!")
print("Sol taraftaki 'Dosyalar' (Klasör ikonu) menüsünü yenileyerek README.md dosyasını bilgisayarınıza indirebilir ve GitHub deponuza yükleyebilirsiniz.")
