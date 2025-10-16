BAŞLA

  // --- Değişken Tanımlamaları ---
  TANIMLA PIN_SISTEM = "1234"      // Sistemde kayıtlı PIN
  TANIMLA pinDenemeHakki = 3        // PIN deneme hakkı
  TANIMLA bakiye = 5000             // Kullanıcının mevcut bakiyesi (TL)
  TANIMLA gunlukLimit = 3000        // Günlük çekim limiti (TL)
  TANIMLA gunlukCekilen = 0         // Gün içinde çekilmiş olan tutar
  TANIMLA girilenPIN, cekilecekTutar, devamEt

  // --- Ana İşlem Döngüsü ---
  DÖNGÜ (devamEt != "H" VE devamEt != "h")

    // --- PIN Doğrulama ---
    YAZ "Lütfen kartınızı takınız."
    YAZ "4 haneli PIN kodunuzu giriniz:"
    OKU girilenPIN

    DÖNGÜ (girilenPIN != PIN_SISTEM VE pinDenemeHakki > 1)
      pinDenemeHakki = pinDenemeHakki - 1
      YAZ "Hatalı PIN. Kalan deneme hakkınız: ", pinDenemeHakki
      YAZ "Lütfen PIN kodunuzu tekrar giriniz:"
      OKU girilenPIN
    DÖNGÜ SONU

    EĞER girilenPIN == PIN_SISTEM ISE
      YAZ "PIN doğrulandı. Hoş geldiniz."

      // --- Para Çekme İşlemi ---
      YAZ "Çekmek istediğiniz tutarı giriniz (20 TL ve katları olmalıdır):"
      OKU cekilecekTutar

      // 1. Tutar Kontrolü (20 TL'nin katı mı?)
      EĞER (cekilecekTutar % 20 != 0) ISE
        YAZ "Hata: Lütfen sadece 20 TL ve katları bir tutar giriniz."
      DEĞİLSE
        // 2. Bakiye Kontrolü
        EĞER (cekilecekTutar > bakiye) ISE
          YAZ "Hata: Yetersiz bakiye. Mevcut bakiyeniz: ", bakiye, " TL"
        DEĞİLSE
          // 3. Günlük Limit Kontrolü
          EĞER (cekilecekTutar + gunlukCekilen > gunlukLimit) ISE
            YAZ "Hata: Günlük para çekme limitini aşıyorsunuz."
            YAZ "Bu işlemle birlikte toplam çekilen tutar günlük limiti geçecektir."
            YAZ "Bugün çekebileceğiniz maksimum tutar: ", (gunlukLimit - gunlukCekilen), " TL"
          DEĞİLSE
            // --- İşlem Başarılı ---
            bakiye = bakiye - cekilecekTutar
            gunlukCekilen = gunlukCekilen + cekilecekTutar
            YAZ "Lütfen paranızı alınız."
            YAZ "İşlem başarıyla tamamlandı."
            YAZ "Kalan bakiyeniz: ", bakiye, " TL"
          BİTTİ-EĞER
        BİTTİ-EĞER
      BİTTİ-EĞER

    DEĞİLSE
      YAZ "PIN 3 kez hatalı girildi. Kartınız bloke olmuştur. Lütfen şubenizle iletişime geçin."
      BİTİR  // Programı sonlandır
    BİTTİ-EĞER

    // --- İşlem Tekrarı ---
    YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
    OKU devamEt

    EĞER (devamEt == "E" VEYA devamEt == "e") ISE
      // Döngü devam edecek, PIN tekrar istenecek
      pinDenemeHakki = 3 // Yeni işlem için deneme hakkını sıfırla
    DEĞİLSE
      YAZ "İyi günler dileriz. Lütfen kartınızı alınız."
    BİTTİ-EĞER

  DÖNGÜ SONU

BİTİR
