# Akbank_Gen_AI_chatbot
# Akbank GenAI Bootcamp - RAG Tabanlı Yemek Tarifi Chatbot'u

[cite_start]Bu proje, Akbank GenAI Bootcamp kapsamında geliştirilmiş [cite: 1][cite_start], RAG (Retrieval-Augmented Generation) temelli bir yemek tarifi chatbot'udur[cite: 2].

## [cite_start]Projenin Amacı [cite: 9]

Projenin temel amacı, `turkish_food.csv` (veya benzer bir) veri setindeki yemek tariflerini kullanarak kullanıcıların doğal dilde sorduğu sorulara yanıt veren bir web tabanlı chatbot oluşturmaktır. Kullanıcılar, "Bana bir çorba tarifi ver" veya "İçinde et olan hangi yemekler var?" gibi sorular sorarak tarif veritabanını sorgulayabilir.

---

## [cite_start]2. Veri Seti Hazırlama [cite: 16]

Proje, `turkish_food.csv` (veya `create_database.py` betiğine `--csv` argümanı ile sağlanan herhangi bir) CSV dosyasını işleyerek bir vektör veritabanı oluşturur.

[cite_start]**Veri Seti Bilgisi**: [cite: 10]
Bu projede, `turkish_food.csv` adlı bir CSV dosyası kullanılmıştır. (Not: Kod, `indian_food.csv` gibi benzer yapıdaki diğer dosyalarla da çalışabilir).

Veri setinin, `create_database.py` betiğinin beklentilerine göre şu sütunları içermesi gerekmektedir:
* `name`: Yemeğin adı
* `diet`: Diyet türü (örn: vejetaryen, glutensiz)
* `course`: Kategori (örn: ana yemek, çorba, tatlı)
* `flavor_profile`: Lezzet profili (örn: tatlı, tuzlu, baharatlı)
* `prep_time`: Hazırlama süresi (dakika)
* `cook_time`: Pişirme süresi (dakika)
* `state`: (Bölge/Eyalet)
* `region`: (Bölge)
* `ingredients`: Malzemeler (virgülle ayrılmış metin)

[cite_start]**Hazırlanış Metodolojisi**: [cite: 17]
Veri seti, `create_database.py` betiği kullanılarak RAG mimarisine uygun hale getirilmiştir. Bu betik sırasıyla şu işlemleri yapar:

1.  **CSV Okuma:** Pandas kütüphanesi ile `--csv` yolundaki veri seti okunur.
2.  **Sütun Doğrulama:** `name`, `diet`, `course`, `flavor_profile`, `prep_time`, `cook_time`, `state`, `region`, `ingredients` sütunlarının varlığı kontrol edilir.
3.  **Veri Temizleme ve Normalizasyon:**
    * `norm_str` fonksiyonu: Tüm metin alanlarındaki (örn: `name`, `diet`) başındaki/sonundaki boşluklar alınır ve birden fazla boşluk tek boşluğa indirgenir.
    * `to_int` fonksiyonu: `prep_time` ve `cook_time` sütunlarındaki değerler güvenli bir şekilde tamsayıya (integer) dönüştürülür. "-1" veya "N/A" gibi hatalı girdiler "0" olarak ayarlanır.
    * `norm_ingredients` fonksiyonu: `ingredients` sütunundaki virgül (`,`) veya noktalı virgül (`;`) ile ayrılmış malzeme listesi ayrıştırılır, her bir malzeme küçük harfe çevrilir, temizlenir ve yinelenen girdiler kaldırılır.
4.  **Doküman Oluşturma (`build_doc_text`):**
    * Her bir tarif satırı, LLM'in (Büyük Dil Modeli) anlayabileceği yapılandırılmış bir metin dokümanına dönüştürülür. Bu doküman, model için "Bağlam (Context)" görevi görecektir.
5.  **Embedding ve Depolama:**
    * Hazırlanan tüm metin dokümanları, Google'ın `models/text-embedding-004` modeli kullanılarak vektörlere dönüştürülür.
    * Bu vektörler, tariflerin metadata'ları (isim, diyet, süre vb.) ile birlikte `Chroma` vektör veritabanına kaydedilir. Veritabanı, `./chroma_db` dizininde saklanır.

---

## [cite_start]3. Kodunuzun Çalışma Kılavuzu [cite: 19]

Bu proje bir Google Colab ortamında veya lokal bir makinede çalıştırılabilir.

**1. [cite_start]Gereksinimler (`requirements.txt`)** [cite: 21]

Aşağıdaki kütüphanelerin kurulu olması gerekmektedir:
flask flask-ngrok google-generativeai langchain-google-genai langchain-community chromadb markdown python-dotenv pandas pyngrok google-api-python-client google-auth
**2. Kurulum ve Çalıştırma (Lokal/Sunucu)** [cite: 21]

1.  **Sanal Ortam (Önerilir):** [cite: 21]
    ```bash
    python -m venv venv
    source venv/bin/activate  # (Windows: venv\Scripts\activate)
    ```

2.  **Kütüphanelerin Kurulumu:** [cite: 21]
    ```bash
    pip install -r requirements.txt
    ```

3.  **Google API Anahtarı:**
    * `GOOGLE_API_KEY` adında bir ortam değişkeni tanımlayın veya Colab kullanıyorsanız "Secrets" bölümüne ekleyin.

4.  **Veri Seti:**
    * `turkish_food.csv` dosyasını ana dizine yerleştirin.

5.  **Veritabanını Oluşturma:**
    * `create_database.py` betiğini çalıştırarak vektör veritabanını (`./chroma_db`) oluşturun:
    ```bash
    python create_database.py --csv turkish_food.csv
    ```

6.  **Sunucuyu Başlatma:** [cite: 21]
    * `app.py` betiğini çalıştırarak Flask sunucusunu başlatın:
    ```bash
    python app.py
    ```
    * Sunucu varsayılan olarak `http://0.0.0.0:5000` adresinde çalışacaktır.

**3. Çalıştırma (Google Colab)** [cite: 14]

1.  **API Anahtarı:** Colab'in sol tarafındaki "Secrets" (Anahtar ikonu 🔑) bölümüne `GOOGLE_API_KEY`'inizi ekleyin.
2.  **Veri Seti:** Colab'in sol tarafındaki "Files" bölümüne `turkish_food.csv` dosyasını yükleyin.
3.  **Kod Hücreleri:** Proje notebook'undaki tüm `!pip install` hücrelerini çalıştırın.
4.  `%%writefile` komutlarını içeren tüm hücreleri çalıştırarak `create_database.py`, `templates/index.html` ve `app.py` dosyalarını oluşturun.
5.  `!python create_database.py ...` hücresini çalıştırarak veritabanını oluşturun.
6.  API anahtarını yükleyen (Cell 11) ve sunucuyu başlatan (Cell 12: `!python app.py &`) hücreleri çalıştırın.
7.  Son hücreyi (`output.eval_js...`) çalıştırarak proxy URL'sini alın ve chatbot arayüzüne erişin.

---

## 4. Çözüm Mimariniz [cite: 22]

**Çözülen Problem**: [cite: 23]
Bu proje, yapılandırılmış bir CSV dosyasında (yemek tarifleri) bulunan bilgilere doğal dil ile erişim problemini çözmektedir. Kullanıcıların, SQL veya filtreleme mantığı bilmeden, "Bana 30 dakikadan az süren bir tarif bul" gibi karmaşık sorgular yapabilmesini sağlar.

**Kullanılan Teknolojiler**: [cite: 23, 42, 43, 44]

* **Generation (Üretken) Model:** `gemini-2.5-flash` (Google Gemini API) [cite: 42]
* **Embedding Modeli:** `models/text-embedding-004` (Google) [cite: 43]
* **Vektör Veritabanı (Vector DB):** `Chroma` (Lokal disk tabanlı) [cite: 43]
* **RAG Pipeline Framework:** `LangChain` [cite: 44]
* **Web Framework & Arayüz:** `Flask` & `HTML/CSS/JavaScript`

**RAG Mimarisi**: [cite: 23]
Mimari iki ana aşamadan oluşur:

**1. İndeksleme (Offline) - `create_database.py`**
Bu aşama, veri hazırlandıktan sonra bir kez çalıştırılır:
1.  **Yükle:** `turkish_food.csv` dosyası Pandas DataFrame'e yüklenir.
2.  **Dönüştür:** Her satır, `build_doc_text` fonksiyonu ile yapılandırılmış bir metin dokümanına çevrilir.
3.  **Göm (Embed):** Her doküman `GoogleGenerativeAIEmbeddings` (`text-embedding-004`) kullanılarak vektörel bir temsile dönüştürülür.
4.  **Depola:** Bu vektörler ve ilişkili meta veriler (isim, diyet, süre vb.) `Chroma` veritabanına kaydedilir.

**2. Sorgulama (Online) - `app.py` (`get_answer` fonksiyonu)**
Bu aşama, kullanıcı her mesaj gönderdiğinde çalışır:
1.  **Sorgu (Query):** Kullanıcının (örn: "Hafif bir tatlı tarifi") mesajı alınır.
2.  **Sorgu Gömme (Query Embedding):** Kullanıcının sorgusu da aynı `text-embedding-004` modeli ile bir vektöre dönüştürülür.
3.  **Getir (Retrieve):** Bu sorgu vektörü, `Chroma` veritabanında `similarity_search` yapmak için kullanılır. Veritabanı, sorguya anlamsal olarak en yakın `k=5` adet tarifi (dokümanı) döndürür.
4.  **Büyüt (Augment):** Alınan bu 5 tarif dokümanı, bir "Bağlam (Context)" bloğu olarak birleştirilir.
5.  **Üret (Generate):** Bu bağlam, `gemini-2.5-flash` modeline bir sistem talimatı (prompt) ile birlikte gönderilir.
6.  **Yanıt:** Modelin ürettiği yanıt, Flask üzerinden web arayüzüne geri gönderilir.

---

## 5. Web Arayüzü & Product Kılavuzu [cite: 24]

**Deploy Linki**: [cite: 25]
Proje Google Colab üzerinde çalıştırıldığında, proxy URL'si (örn: `https://5000-m-s-2dgccsbfqwft2-a.us-west4-1.prod.colab.dev/`) deploy linki olarak kullanılır. [cite: 13]

**Çalışma Akışı ve Arayüz Testi**: [cite: 25]
Web arayüzü (`index.html`), sol tarafta bir sohbet geçmişi listesi ve sağ tarafta ana sohbet penceresi bulunan klasik bir chatbot tasarımıdır.

1.  **Ana Sayfa:** Kullanıcı linki açtığında, `index.html` yüklenir. Eğer mevcut bir sohbet yoksa, "Yemek Tarifi Chatbot" başlığı ve örnek bir soru (`Örn: “Air fryer’da patates var mı?”`) görünür.
2.  **Yeni Sohbet:** Soldaki "Yeni Sohbet +" butonuna tıklayarak mevcut sohbeti temizleyebilir ve yeni bir oturum (`session_id`) başlatabilirsiniz.
3.  **Mesaj Gönderme:** Alttaki giriş kutusuna sorunuzu yazıp "Gönder" butonuna tıkladığınızda:
    * Mesajınız "user" (mavi) olarak ekrana eklenir.
    * "Yazıyor..." göstergesi belirir.
    * `/send_message` endpoint'ine (uç noktasına) bir `POST` isteği gönderilir.
    * `app.py` içindeki `get_answer` fonksiyonu çalışır (RAG süreci).
    * Modelden gelen yanıt "bot" (gri) olarak ekranda görünür.
4.  **Sohbet Geçmişi:**
    * Gönderdiğiniz ilk mesaj, o sohbetin başlığı olarak (örn: "Bana bir çorba tarifi...") sol taraftaki listeye eklenir.
    * Geçmiş sohbet başlıklarına tıklayarak o konuşmaları geri yükleyebilirsiniz.

**Projenin Kabiliyetlerini Test Etme Senaryoları**: [cite: 25]
Botun kabiliyetlerini test etmek için aşağıdaki gibi sorular sorabilirsiniz:

* **Genel Tarif İsteği:** "Bana bir ana yemek tarifi verebilir misin?"
* **Spesifik Malzeme Sorgusu:** "İçinde patlıcan olan hangi tarifler var?"
* **Diyet/Kategori Filtrelemesi:** "Vejetaryen tatlı tarifi arıyorum."
* **Süre Filtrelemesi:** "Hazırlaması 15 dakikadan az süren bir yemek var mı?"
* **Veri Setinde Olmayan Soru (Negatif Test):** "Bana İtalyan pizzası tarifi ver." (Eğer `turkish_food.csv` içinde yoksa, modelin "Veritabanımda bu tarif yok." demesi beklenir).
* **Konu Dışı Soru (Güvenlik Testi):** "Türkiye'nin başkenti neresidir?" (Modelin, "Sadece tariflerle ilgili soruları yanıtla." kuralına uyması beklenir)
