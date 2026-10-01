# Ortak Karar Forumu: Oylamayla Yönetilen, Ontoloji Tabanlı Tartışma Platformu
**Yazılım Mühendisliği Dersi Proje Raporu**

* **GitHub Deposu:** [https://github.com/revanyusifli/yapay_zeka_icin_yazilim_muhendisligi](https://github.com/revanyusifli/yapay_zeka_icin_yazilim_muhendisligi)
* **Geliştirici:** Ravan Yusifli
* **Ders:** Yapay Zeka İçin Yazılım Mühendisliği

---

### 1. Amaç
Platformda konu açma, düzenleme, silme ve kapatma gibi tüm sistemsel değişiklikler oylama süreçleriyle gerçekleştirilir. Temel amaç; kararların yalnızca konudan doğrudan etkilenen paydaşlarca, adil ve şeffaf biçimde verildiği ve geriye dönük kriptografik olarak denetlenebildiği merkeziyetsiz bir müzakere sistemi kurmaktır. Karar alma mekanizmaları doğrudan forumun kendi veri yapısına yerleştirilmiştir.

---

### 2. Gerçekleştirilen İşlevler ve Mimari
* **Kullanıcı Kaydı ve Güvenlik:** Kayıt esnasında ad, soyad, doğum tarihi, adres, takma ad, şifre ve en az 3 adet ilgi alanı toplanır. Kullanıcı şifreleri tek yönlü tuzlama (*salt*) işleminden geçirilerek **SHA-256** algoritmasıyla şifrelenir. Kişisel veriler gizlenerek yalnızca takma adlar genele açılır.
* **Konu Hiyerarşisi ve Dinamik Yönetim:** Konular hiyerarşik alt başlıklar halinde düzenlenir. Sistemde var olan üst konular seçilebildiği gibi, serbest metinle yeni üst kategori açılmasına da izin verilmiştir. Tartışmanın veya karar metninin belirli bölümlerinin gizlenmesi bağımsız oylamalara tabidir.
* **Nitelikli Oylama ve "Silinmeme" İlkesi:** Silme (arşivleme), metin parçası gizleme ve konu kapatma işlemleri için **2/3 (%66)** nitelikli çoğunluk aranırken; yeni konu ve ek metin kabulünde **1/2** basit çoğunluk uygulanır. Tüm oylamalarda **%25 katılım şartı (*quorum*)** aranır. Sistemde fiziksel silme işlemi yoktur; içerikler oylamayla gizlenir ancak defterde değişmez biçimde saklanır.
* **Ontoloji Belleği (`Ontology.memory`):** Konu alanları kalıtımsal bir ağaç yapısındadır. Bir üst alan (örn. *Üniversite*), alt alanın (örn. *Sınıf*) üyelerini kapsar; ancak alt alandaki bir konuya yalnızca o yerel alanın üyeleri oy kullanabilir.
* **Likit Demokrasi ve Otonom YZ:** Kullanıcılar oy güçlerini ilgili alandaki bir bilirkişiye devredebilir. Bilirkişilerin taban oyu **10 puandır**. Spor/Futbol alanında açılan teklifler için **Claude Futbol-YZ** devreye girerek metni yönetmeliğe göre analiz eder ve otonom oy kullanır. Delegasyon ilişkileri yönlü bir graf üzerinde işletilir.
* **Otomatik Yönetmelik ve Benzerlik Kontrolü:** Hakaret içeren ifadeler, 3'ten fazla açık öneri kotası ve **Jaccard Benzerlik Algoritması** ile mevcut başlıklara %60 ve üzeri benzeyen öneriler otomatik olarak engellenir. Değişmez kuralları ihlal eden teklifler sistemden reddedilir.
* **Dağıtık Blok Defteri (Audit Ledger):** Her işlem (`kayıt`, `öneri`, `oy`, `sonuç`, `delegasyon`) bir önceki bloğun SHA-256 özetini içeren zincirleme blok defterine yazılır. Bütünlük arayüzden tek tıkla doğrulanabilir.
* **Doğrulama ve Test:** `Nijat`, `Osman`, `Revan` ve `Maqa` hesaplarıyla Domino senaryosu canlı olarak koşturulmuş; kabul ve ret çıktıları deftere blok olarak kaydedilmiştir.

---

### 3. Soru 1: Çoğunluk Azınlığı Tüketmesin, Bunu Nasıl Koruruz?
Azınlık haklarını korumak ve çoğunluk tahakkümünü engellemek adına 5 katmanlı koruma kalkanı uygulanmıştır:
1. **Nitelikli Eşik (2/3):** Azınlığı doğrudan etkileyecek kapatma, silme ve gizleme işlemlerinde salt çoğunluk (%50) yetersiz kılınmış, %66 barajı getirilmiştir.
2. **%25 Katılım Şartı (*Quorum*):** Düşük katılımlı oturumlarda örgütlü küçük grupların ani kararlar alması engellenmiştir.
3. **%20 Güç Tavanı (*Power Cap*):** Tek bir bilirkişi veya Yapay Zekâ ajanı, arkasına ne kadar oy devri alırsa alsın toplam oy gücünün %20'sinden (en az 10 puan) fazlasını tek başına temsil edemez.
4. **Ontolojik Alan Kısıtı:** Bir konuya yalnızca o alandan doğrudan etkilenen paydaşlar oy verebilir; genel çoğunluğun ilgisiz yerel alanları ezmesi önlenmiştir.
5. **Kriptografik Geri Alınabilirlik:** Fiziksel silme engellendiği için çoğunluğun aldığı her karar defterde kayıtlı kalır ve karşı bir oylama ile iptal edilebilir.

---

### 4. Soru 2: Çok Sayıda Konu Kalabalık Yaratırsa Nasıl Çözülür?
Sistemik kirlilik ve konu kalabalığı şu yöntemlerle kontrol altına alınmıştır:
1. **Jaccard Benzerlik Filtresi ($J \ge 0.60$):** Yeni başlıklar mevcut konularla küme kesişimi yöntemiyle kıyaslanır. %60 ve üzeri benzerlikte yeni başlık açılması engellenir, kullanıcı mevcut konuya ek metin önermeye yönlendirilir.
2. **Kullanıcı Başına Aktif Öneri Sınırı:** Bir kullanıcının aynı anda en fazla 3 adet sonuçlanmamış önerisi bulunabilir.
3. **Kapsam İndirgeme:** Kullanıcılar sadece kayıtlı oldukları ilgi alanlarına ait müzakereleri görür, ilgisiz akış filtrelenir.
4. **Hiyerarşik Ağaçlandırma:** Konular alt başlıklar altında modüler olarak tutulur; hareketsiz konular oylamayla arşivlenerek görünürlükten kaldırılır.

---

### 5. Yazılım Mühendisliği Yaklaşımı
Sistem katmanlı mimari (*Layered Architecture*) prensibiyle ele alınmıştır:
* **İstemci Katmanı:** Mobil uyumlu, erişilebilir web arayüzü.
* **Alan Mantığı (Domain Services):** Oylama motoru, Jaccard benzerlik filtresi, ontoloji kapsam denetleyicisi ve otonom YZ karar kuralı birbirinden bağımsız, test edilebilir modüller olarak kodlanmıştır.
* **Veri ve Güvenlik Katmanı:** Web Crypto API tabanlı `SHA-256` kriptografik blok defteri ve tuzlanmış kimlik doğrulama katmanı.