PROGRAM HastaneRandevuSistemi

  // Gerekli değişkenlerin tanımlanması
  DEĞİŞKEN kullaniciTC, kullaniciParola, secilenPoliklinikID, secilenDoktorID, secilenTarih, secilenSaat, onayMetni TÜRÜNDE METİN
  DEĞİŞKEN girisBasarili, randevuOlusturuldu TÜRÜNDE MANTIKSAL
  DEĞİŞKEN mevcutPoliklinikler, mevcutDoktorlar, uygunSaatler TÜRÜNDE LİSTE
  DEĞİŞKEN aktifKullanici, randevuDetaylari TÜRÜNDE NESNE

  // 1. ADIM: KİMLİK DOĞRULAMA
  //---------------------------------------------------------
  BAŞLANGIÇ:
    EKRANA_YAZ("----- HASTANE RANDEVU SİSTEMİ -----")
    EKRANA_YAZ("Lütfen TC Kimlik Numaranızı giriniz:")
    kullaniciTC = KULLANICIDAN_OKU()

    EKRANA_YAZ("Lütfen parolanızı giriniz:")
    kullaniciParola = KULLANICIDAN_OKU()

    // Kimlik doğrulama fonksiyonu çağrılır
    girisBasarili = FONKSİYON_KimlikDogrula(kullaniciTC, kullaniciParola)

    EĞER girisBasarili == YANLIŞ
      EKRANA_YAZ("Hata: Girilen bilgiler yanlış. Lütfen tekrar deneyin.")
      GİT BAŞLANGIÇ // Kimlik doğrulama adımının başına dön
    DEĞİLSE
      aktifKullanici = FONKSİYON_KullaniciBilgileriniGetir(kullaniciTC)
      EKRANA_YAZ("Hoş geldiniz, " + aktifKullanici.AdSoyad)
    SON_EĞER

  // 2. ADIM: POLİKLİNİK SEÇİMİ
  //---------------------------------------------------------
  POLİKLİNİK_SEÇİMİ:
    mevcutPoliklinikler = FONKSİYON_PoliklinikleriListele()
    EKRANA_YAZ("\n----- POLİKLİNİKLER -----")
    DÖNGÜ her poliklinik İÇİN mevcutPoliklinikler
      EKRANA_YAZ(poliklinik.ID + " - " + poliklinik.Ad)
    SON_DÖNGÜ
    
    EKRANA_YAZ("Lütfen randevu almak istediğiniz polikliniğin numarasını giriniz:")
    secilenPoliklinikID = KULLANICIDAN_OKU()
    
    EĞER FONKSİYON_PoliklinikGecerliMi(secilenPoliklinikID) == YANLIŞ
        EKRANA_YAZ("Hata: Geçersiz poliklinik seçimi. Lütfen tekrar deneyin.")
        GİT POLİKLİNİK_SEÇİMİ
    SON_EĞER

  // 3. ADIM: DOKTOR SEÇİMİ VE LİSTELEME
  //---------------------------------------------------------
  DOKTOR_SEÇİMİ:
    mevcutDoktorlar = FONKSİYON_DoktorlariListele(secilenPoliklinikID)
    EKRANA_YAZ("\n----- DOKTORLAR -----")
    DÖNGÜ her doktor İÇİN mevcutDoktorlar
      EKRANA_YAZ(doktor.ID + " - " + doktor.Unvan + " " + doktor.AdSoyad)
    SON_DÖNGÜ

    EKRANA_YAZ("Lütfen doktor numarasını giriniz:")
    secilenDoktorID = KULLANICIDAN_OKU()

    EĞER FONKSİYON_DoktorGecerliMi(secilenDoktorID, secilenPoliklinikID) == YANLIŞ
        EKRANA_YAZ("Hata: Geçersiz doktor seçimi. Lütfen tekrar deneyin.")
        GİT DOKTOR_SEÇİMİ
    SON_EĞER

  // 4. ADIM: TARİH VE UYGUN SAATLERİ GÖSTERME
  //---------------------------------------------------------
  SAAT_SEÇİMİ:
    EKRANA_YAZ("\nLütfen randevu tarihini giriniz (Örn: 28.10.2025):")
    secilenTarih = KULLANICIDAN_OKU()

    uygunSaatler = FONKSİYON_UygunSaatleriGetir(secilenDoktorID, secilenTarih)

    EĞER uygunSaatler LİSTESİ BOŞ
      EKRANA_YAZ("Seçtiğiniz tarihte bu doktor için uygun randevu saati bulunmamaktadır.")
      EKRANA_YAZ("Başka bir tarih denemek için 'T', menüye dönmek için herhangi bir tuşa basınız.")
      kullaniciSecimi = KULLANICIDAN_OKU()
      EĞER kullaniciSecimi == "T" VEYA kullaniciSecimi == "t"
        GİT SAAT_SEÇİMİ
      DEĞİLSE
        EKRANA_YAZ("İşlem iptal edildi.")
        PROGRAMI_BİTİR
      SON_EĞER
    DEĞİLSE
      EKRANA_YAZ("\n----- " + secilenTarih + " TARİHİ İÇİN UYGUN SAATLER -----")
      DÖNGÜ her saat İÇİN uygunSaatler
        EKRANA_YAZ("- " + saat)
      SON_DÖNGÜ
      
      EKRANA_YAZ("Lütfen bir saat seçiniz (Örn: 10:30):")
      secilenSaat = KULLANICIDAN_OKU()
      
      EĞER uygunSaatler LİSTESİNDE secilenSaat YOK
        EKRANA_YAZ("Hata: Geçersiz saat seçimi. Lütfen listeden bir saat giriniz.")
        GİT SAAT_SEÇİMİ
      SON_EĞER
    SON_EĞER

  // 5. ADIM: RANDEVU ONAYLAMA
  //---------------------------------------------------------
  ONAY_ADIMI:
    randevuDetaylari = FONKSİYON_RandevuDetaylariniOlustur(aktifKullanici, secilenPoliklinikID, secilenDoktorID, secilenTarih, secilenSaat)
    EKRANA_YAZ("\n----- RANDEVU ONAY -----")
    EKRANA_YAZ("Hasta: " + randevuDetaylari.HastaAdi)
    EKRANA_YAZ("Poliklinik: " + randevuDetaylari.PoliklinikAdi)
    EKRANA_YAZ("Doktor: " + randevuDetaylari.DoktorAdi)
    EKRANA_YAZ("Tarih: " + randevuDetaylari.Tarih)
    EKRANA_YAZ("Saat: " + randevuDetaylari.Saat)
    EKRANA_YAZ("\nYukarıdaki bilgileri onaylıyor musunuz? (E/H)")
    onayMetni = KULLANICIDAN_OKU()

    EĞER onayMetni == "E" VEYA onayMetni == "e"
      // Randevu oluşturma fonksiyonu çağrılır
      randevuOlusturuldu = FONKSİYON_RandevuKaydet(randevuDetaylari)
      
      EĞER randevuOlusturuldu == DOĞRU
        EKRANA_YAZ("Randevunuz başarıyla oluşturulmuştur.")
        
        // 6. ADIM: SMS GÖNDERME
        //---------------------------------------------------------
        smsMesaji = "Sayin " + randevuDetaylari.HastaAdi + ", " + randevuDetaylari.Tarih + " " + randevuDetaylari.Saat + " tarihli randevunuz " + randevuDetaylari.DoktorAdi + " adina olusturulmustur. Saglikli gunler dileriz."
        FONKSİYON_SmsGonder(aktifKullanici.Telefon, smsMesaji)
        EKRANA_YAZ("Randevu bilgileriniz SMS ile telefonunuza gönderilmiştir.")
        
      DEĞİLSE
        EKRANA_YAZ("Hata: Randevu oluşturulurken bir sorun oluştu. Lütfen tekrar deneyin.")
      SON_EĞER
      
    DEĞİLSE
      EKRANA_YAZ("Randevu işlemi tarafınızca iptal edilmiştir.")
    SON_EĞER

  PROGRAMI_BİTİR

SON PROGRAM
