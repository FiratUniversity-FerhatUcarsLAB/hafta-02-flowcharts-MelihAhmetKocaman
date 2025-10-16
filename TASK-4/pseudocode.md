BAŞLA

  // --- Veri Yapılarının Örnek Tanımlamaları ---
  TANIMLA aktifOgrenci NESNESİ:
    ogrenciNo = "12345"
    sifre = "sifre123"
    gpa = 2.4
    alinanDersler = ["MATH101", "PHYS101"]
    mevcutKredi = 0

  TANIMLA mevcutDerslerListesi LİSTESİ:
    [
      {dersKodu: "CS101", dersAdi: "Bilgisayar Bilimlerine Giriş", kredi: 5, kontenjan: 50, kayitliOgrenciSayisi: 48, onKosul: "", zaman: {gun: "Pazartesi", baslangic: 10, bitis: 12}},
      {dersKodu: "CS102", dersAdi: "Veri Yapıları", kredi: 6, kontenjan: 40, kayitliOgrenciSayisi: 40, onKosul: "CS101", zaman: {gun: "Salı", baslangic: 13, bitis: 15}},
      {dersKodu: "MATH201", dersAdi: "İleri Matematik", kredi: 6, kontenjan: 30, kayitliOgrenciSayisi: 25, onKosul: "MATH101", zaman: {gun: "Pazartesi", baslangic: 10, bitis: 12}},
      {dersKodu: "HIST101", dersAdi: "Tarih", kredi: 3, kontenjan: 100, kayitliOgrenciSayisi: 80, onKosul: "", zaman: {gun: "Çarşamba", baslangic: 15, bitis: 17}}
    ]

  TANIMLA secilenDersler LİSTESİ = [] // Başlangıçta boş

  // --- ADIM 1: Öğrenci Girişi ---
  YAZDIR "Öğrenci No Giriniz:"
  KULLANICIDAN AL girilenOgrenciNo
  YAZDIR "Şifre Giriniz:"
  KULLANICIDAN AL girilenSifre

  EĞER (girilenOgrenciNo == aktifOgrenci.ogrenciNo VE girilenSifre == aktifOgrenci.sifre) İSE
    YAZDIR "Giriş başarılı. Ders kayıt ekranına yönlendiriliyorsunuz..."
    TANIMLA kayitDevamEdiyor = DOĞRU
  DEĞİLSE
    YAZDIR "Hatalı öğrenci no veya şifre."
    TANIMLA kayitDevamEdiyor = YANLIŞ
    SONLANDIR
  SON-EĞER

  // --- ADIM 2: Ana Kayıt Döngüsü ---
  SÜRDÜKÇE (kayitDevamEdiyor == DOĞRU)
    YAZDIR "--- Ders Kayıt Menüsü ---"
    YAZDIR "1. Açılan Dersleri Görüntüle"
    YAZDIR "2. Ders Ekle"
    YAZDIR "3. Ders Çıkar"
    YAZDIR "4. Seçilen Dersleri ve Kredi Durumunu Göster"
    YAZDIR "5. Kaydı Tamamla ve Onayla"
    KULLANICIDAN AL secim

    // --- Ders Ekleme Bloğu ---
    EĞER (secim == "2") İSE
      YAZDIR "Eklemek istediğiniz dersin kodunu giriniz:"
      KULLANICIDAN AL eklenecekDersKodu
      TANIMLA eklenecekDers = mevcutDerslerListesi içinden eklenecekDersKodu'na sahip olan dersi bul
      TANIMLA hataMesaji = ""

      EĞER (eklenecekDers MEVCUT DEĞİL) İSE
        hataMesaji = "Bu kod ile bir ders bulunamadı."
      DEĞİLSE
        // **KONTROL 1: Kredi Limiti Kontrolü (Toplam ≤ 35)**
        yeniToplamKredi = aktifOgrenci.mevcutKredi + eklenecekDers.kredi
        EĞER (yeniToplamKredi > 35) İSE
          hataMesaji = "Kredi Limiti Aşıldı! Bu dersi ekleyemezsiniz. (Mevcut: " + aktifOgrenci.mevcutKredi + ", Denediğiniz: " + eklenecekDers.kredi + ")"
        DEĞİLSE
          // **KONTROL 2: Kontenjan Kontrolü**
          EĞER (eklenecekDers.kayitliOgrenciSayisi >= eklenecekDers.kontenjan) İSE
            hataMesaji = "Kontenjan Dolu! '" + eklenecekDers.dersAdi + "' dersini ekleyemezsiniz."
          DEĞİLSE
            // **KONTROL 3: Ön Koşul Dersi Kontrolü**
            EĞER (eklenecekDers.onKosul != "" VE aktifOgrenci.alinanDersler listesi eklenecekDers.onKosul'u İÇERMİYOR) İSE
              hataMesaji = "Ön Koşul Hatası! Bu dersi alabilmek için önce '" + eklenecekDers.onKosul + "' dersini geçmiş olmalısınız."
            DEĞİLSE
              // **KONTROL 4: Zaman Çakışması Kontrolü**
              TANIMLA cakismaVar = YANLIŞ
              HER BİR kayitliDers İÇİN secilenDersler LİSTESİNDE
                EĞER (kayitliDers.zaman.gun == eklenecekDers.zaman.gun VE
                     (eklenecekDers.zaman.baslangic < kayitliDers.zaman.bitis VE eklenecekDers.zaman.bitis > kayitliDers.zaman.baslangic)) İSE
                  cakismaVar = DOĞRU
                  hataMesaji = "Zaman Çakışması! '" + eklenecekDers.dersAdi + "' dersi, '" + kayitliDers.dersAdi + "' dersi ile çakışıyor."
                  DÖNGÜDEN ÇIK // İlk çakışmayı bulmak yeterli
                SON-EĞER
              SON-DÖNGÜ

              // Tüm kontrollerden geçtiyse dersi ekle
              EĞER (cakismaVar == YANLIŞ) İSE
                secilenDersler listesine eklenecekDers'i EKLE
                aktifOgrenci.mevcutKredi = aktifOgrenci.mevcutKredi + eklenecekDers.kredi
                YAZDIR "'" + eklenecekDers.dersAdi + "' dersi başarıyla eklendi."
              SON-EĞER
            SON-EĞER
          SON-EĞER
        SON-EĞER
      SON-EĞER

      // Eğer herhangi bir adımda hata oluştuysa, mesajı yazdır
      EĞER (hataMesaji != "") İSE
        YAZDIR "HATA: " + hataMesaji
      SON-EĞER
    
    // --- Diğer Menü Seçenekleri ---
    DEĞİLSE EĞER (secim == "1") İSE
      YAZDIR "--- Sistemde Açılan Dersler ---"
      HER BİR ders İÇİN mevcutDerslerListesi LİSTESİNDE
        YAZDIR ders.dersKodu + " - " + ders.dersAdi + " (Kredi: " + ders.kredi + ", Kontenjan: " + ders.kayitliOgrenciSayisi + "/" + ders.kontenjan + ")"
      SON-DÖNGÜ

    DEĞİLSE EĞER (secim == "3") İSE
      // (Ders Çıkarma mantığı buraya eklenebilir: secilenDersler'den çıkar, krediyi güncelle)
      YAZDIR "Çıkarılacak dersin kodunu girin:"
      // ...
      
    DEĞİLSE EĞER (secim == "4") İSE
      YAZDIR "--- Seçilen Dersler ---"
      EĞER (secilenDersler LİSTESİ BOŞ) İSE
        YAZDIR "Henüz ders seçmediniz."
      DEĞİLSE
        HER BİR ders İÇİN secilenDersler LİSTESİNDE
          YAZDIR ders.dersKodu + " - " + ders.dersAdi + " (Kredi: " + ders.kredi + ")"
        SON-DÖNGÜ
      SON-EĞER
      YAZDIR "Toplam Kredi: " + aktifOgrenci.mevcutKredi + " / 35"

    DEĞİLSE EĞER (secim == "5") İSE
      YAZDIR "--- Kayıt Özeti ---"
      HER BİR ders İÇİN secilenDersler LİSTESİNDE
        YAZDIR ders.dersKodu + " - " + ders.dersAdi
      SON-DÖNGÜ
      YAZDIR "Toplam Alınan Kredi: " + aktifOgrenci.mevcutKredi

      // **SON KONTROL: Danışman Onayı Gerekli mi? (GPA < 2.5)**
      EĞER (aktifOgrenci.gpa < 2.5) İSE
        YAZDIR "UYARI: Not ortalamanız (GPA: " + aktifOgrenci.gpa + ") 2.5'in altında olduğu için kaydınızın danışman tarafından onaylanması gerekmektedir."
        YAZDIR "Kayıt, danışman onayına gönderildi."
      DEĞİLSE
        YAZDIR "Kayıt başarıyla tamamlandı ve onaylandı."
      SON-EĞER
      
      kayitDevamEdiyor = YANLIŞ // Döngüyü sonlandır
    
    DEĞİLSE
      YAZDIR "Geçersiz bir seçim yaptınız."
    SON-EĞER
    
  SON-SÜRDÜKÇE // Ana kayıt döngüsü sonu

  YAZDIR "Sistemden çıkılıyor..."

SON
