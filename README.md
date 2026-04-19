# Video Linki: https://youtu.be/zzmsjIas3vY

ÖZET
Modern bilgi güvenliği disiplininde, kurumsal veritabanlarının iç ve dış tehditlere karşı korunması kritik bir zorunluluktur. Bu projenin amacı; Microsoft SQL Server (MSSQL) üzerinde barındırılan hassas sağlık verilerini korumak adına, "Derinlemesine Savunma" (Defense in Depth) stratejisi ve ISO 27001 bilgi güvenliği standartları doğrultusunda çok katmanlı bir veritabanı güvenlik mimarisi tasarlamaktır.
Çalışmanın ilk aşamasında, yetkisiz veri manipülasyonunu engellemek için "En Az Ayrıcalık Prensibi" (Principle of Least Privilege) uygulanmıştır. SQL Server Authentication ile kısıtlı yetkilere sahip kullanıcı profili oluşturularak yalnızca veri okuma izni tanımlanmış; yetki aşımı (veri silme) denemeleri sistem tarafından başarıyla reddedilmiştir. Yasal standartlar (KVKK) çerçevesinde veri erişimlerinin izlenebilirliğini sağlamak amacıyla 'SQL Server Audit' mekanizması T-SQL scriptleri ile otomatize edilmiş ve hassas verilere yönelik okuma eylemleri arka planda sunucu günlüklerine (log) kaydedilmiştir.
Fiziksel sunucu sızıntılarına ve yedek dosyalarının çalınması riskine karşı veritabanı dosyaları, 'At Rest' (bekleme) konumunda AES-256 algoritmalı TDE (Transparent Data Encryption) ile kriptografik olarak şifrelenmiştir. Son aşamada ise uygulama katmanı zafiyetleri ele alınarak bir SQL Injection (SQLi) saldırısı simüle edilmiş; tespit edilen bu güvenlik açığı, "Parametrik Sorgular" mimarisi ile tamamen etkisiz hale getirilerek saldırı vektörü kapatılmıştır.
Sonuç olarak bu proje; kimlik doğrulama, yetkilendirme, sistem izleme, asimetrik şifreleme ve güvenli kodlama disiplinlerini başarıyla entegre ederek, siber saldırılara karşı tam korunaklı bir veritabanı güvenlik ekosistemi ortaya koymuştur.



İçindekiler
ÖZET	2
1. Giriş	4
2. Erişim Yönetimi ve Yetkilendirme	5
3. Denetim Günlükleri (Audit Logs) ile İz Sürme	6
4. Veri Şifreleme (TDE - Transparent Data Encryption)	7
5. SQL Injection Zafiyet Simülasyonu ve Engellenmesi	8
6. Sonuç	9









1. Giriş
Bilgi güvenliği disiplininde veritabanları, kurumların en kritik varlıklarını barındıran ve siber saldırganların birincil hedefi olan sistemlerdir. Bu projenin amacı; Microsoft SQL Server üzerinde barındırılan hassas sağlık verilerini, iç ve dış tehditlere karşı korumak için çok katmanlı (Defense in Depth) bir güvenlik mimarisi tasarlamaktır.
Çalışma kapsamında, endüstri standartlarına (NIST ve ISO 27001 Bilgi Güvenliği Yönetim Sistemi) uygun olarak dört temel güvenlik adımı uygulanmıştır: Kullanıcı erişimlerinin sınırlandırılması (Authentication & Authorization), veri okuma/yazma aktivitelerinin izlenmesi (Audit Logging), fiziksel veri hırsızlığına karşı şifreleme (Transparent Data Encryption - TDE) ve uygulama katmanı zafiyetlerinin (SQL Injection) simüle edilerek engellenmesi. Tüm bu aşamalar başarılı bir şekilde test edilerek raporlanmıştır.

 


2. Erişim Yönetimi ve Yetkilendirme
Veritabanı güvenliğinin ilk katmanı olan kimlik doğrulama ve yetkilendirme süreçleri, "En Az Ayrıcalık Prensibi" (Principle of Least Privilege) temel alınarak kurgulanmıştır. Sistemde yönetici (sa) yetkilerine sahip hesapların günlük işlemlerde kullanılması büyük bir güvenlik riski oluşturduğundan, sadece veri okuma işlemi yapması gereken kısıtlı bir kullanıcı profili oluşturulmuştur.
•	Kimlik Doğrulama: SQL Server Authentication yöntemi kullanılarak Doktor1 adında yeni bir Login hesabı tanımlanmıştır.
•	Yetkilendirme: İlgili kullanıcı, KalpHastalik veritabanına eşlenmiş (User Mapping) ve kendisine yalnızca db_datareader rolü atanmıştır.
•	Sızma ve Yetki Testi: Doktor1 kimliği ile sisteme giriş yapılmış ve DELETE sorgusu çalıştırılarak sistemin tepkisi ölçülmüştür. Veritabanı motoru, kullanıcının yetkisini sınırlandırdığı için işlemi reddetmiş ve veri bütünlüğü (Integrity) başarıyla korunmuştur.
 






3. Denetim Günlükleri (Audit Logs) ile İz Sürme
Kısıtlı yetkilere sahip kullanıcıların dahi hassas hasta verilerine  ne zaman ve hangi sorgularla eriştiğini takip etmek yasal zorunluluklar (KVKK/GDPR) gereği kritik bir adımdır. Bu izlenebilirliği (Traceability) sağlamak amacıyla SQL Server Audit altyapısı kullanılmıştır.
Arayüz kaynaklı olası senkronizasyon hatalarını (Grid Binding) aşmak ve işlemi tamamen otomatize etmek için T-SQL scriptleri ile bir Server Audit ve Database Audit Specification oluşturulmuştur. Log dosyaları doğrudan sunucunun işletim sistemi dizinlerine yazdırılmıştır. Test aşamasında Doktor1 kılığına girilerek okuma işlemi yapılmış, ardından sys.fn_get_audit_file sistem fonksiyonu kullanılarak arka planda tutulan güvenlik logları başarıyla ekrana yansıtılmıştır.
 




4. Veri Şifreleme (TDE - Transparent Data Encryption)
Veritabanına ağ üzerinden yapılan yetkisiz girişler engellenmiş ve loglanmış olsa da, sunucuya fiziksel olarak erişebilecek bir saldırganın .mdf (Veri) veya .bak dosyalarını USB bellek ile çalma riski bulunmaktadır. Bu "At Rest" tehdidini ortadan kaldırmak için TDE mimarisi aktif edilmiştir.
•	Kriptografik Hiyerarşi: İlk olarak master veritabanında güçlü bir parolaya sahip Ana Şifreleme Anahtarı (Master Key) ve bu anahtara bağlı bir Sertifika oluşturulmuştur.
•	Algoritma ve Aktivasyon: KalpHastalik veritabanında, askeri düzeyde şifreleme sunan AES_256 algoritması kullanılarak TDE aktifleştirilmiştir.
•	Felaket Senaryosu Önlemi: Sunucunun çökmesi durumunda verilerin sonsuza dek kilitli kalmasını önlemek amacıyla, oluşturulan güvenlik sertifikası ve Private Key güvenli bir dizine yedeklenmiştir.

 
5. SQL Injection Zafiyet Simülasyonu ve Engellenmesi
Uygulama sunucuları ile veritabanı arasındaki veri akışında ortaya çıkan zafiyetleri test etmek için bir saldırı simülasyonu gerçekleştirilmiştir.
•	Zafiyet (Vulnerability): Dinamik sorgular kullanan zayıf bir giriş ekranı simüle edilmiş, saldırganın şifre alanına ' OR 1=1 -- mantıksal operatörünü yazarak yetkisiz giriş yapabildiği (Bypass Authentication) kanıtlanmıştır.
•	Zafiyetin Kapatılması (Mitigation): Bu kritik güvenlik açığı, SQL motoruna dışarıdan gelen metinlerin "çalıştırılabilir kod" olarak değil, "sabit parametre" olarak algılatılmasını sağlayan Parametrik Sorgu mimarisi ve sp_executesql prosedürü kullanılarak tamamen yamanmıştır. Yapılan ikinci testte saldırının boşa düştüğü görülmüştür.
 
6. Sonuç
Bu projede, MSSQL ortamında yalnızca teorik değil, pratik ve T-SQL tabanlı bir siber güvenlik inşası gerçekleştirilmiştir. Kimlik doğrulama, iz sürme (Audit), asimetrik/simetrik şifreleme (TDE) ve güvenli kodlama (SQLi Defans) konseptleri uçtan uca uygulanarak, modern bir veri merkezinin ihtiyaç duyduğu güvenlik standartları başarıyla karşılanmıştır.

