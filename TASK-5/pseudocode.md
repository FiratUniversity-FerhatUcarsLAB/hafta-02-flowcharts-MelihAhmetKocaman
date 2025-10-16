BAŞLANGIÇ

DEĞİŞKEN sistemAktif = DOĞRU   // Sistemin genel çalışma durumu
DEĞİŞKEN alarmAktif = YANLIŞ   // Alarmın tetiklenmiş olup olmadığı
DEĞİŞKEN alarmSeviyesi = 0     // Alarmın önem derecesi (0: Yok, 1: Düşük, 2: Orta, 3: Yüksek)

// --- Ana Sistem Döngüsü (7/24 Sürekli Çalışma) ---
SÜRECE:

// Ana döngü, sistem sürekli çalıştığı için sonsuzdur.
DÖNGÜ (DOĞRU)

  // 1. Sistem Aktif mi Kontrolü
  EĞER sistemAktif == DOĞRU ISE

    // 2. Sensör Okuma
    hareketAlgilandi = okuHareketSensörü()
    kapiPencereAcik = okuKapiPencereSensörü()

    // 3. Tehdit Algılama ve Alarm Tetikleme
    // Sadece alarm zaten aktif değilken yeni bir tehdit kontrolü yapılır.
    EĞER (hareketAlgilandi == DOĞRU VEYA kapiPencereAcik == DOĞRU) VE alarmAktif == YANLIŞ ISE

      // 4. Yanlış Alarm Kontrolü (Ev Sahibi Evde mi?)
      evSahibiEvde = kontrolEtEvSahibiKonumu() // GPS, Wi-Fi veya Bluetooth ile kontrol

      EĞER evSahibiEvde == YANLIŞ ISE

        // 5. Kamera Aktivasyonu
        aktifEtKameraKaydi()

        // 6. Alarm Seviyesi Belirleme
        EĞER hareketAlgilandi == DOĞRU VE kapiPencereAcik == DOĞRU ISE
          alarmSeviyesi = 3 // Yüksek Öncelik: Hem hareket var hem de kapı/pencere açık
        DEĞİLSE EĞER kapiPencereAcik == DOĞRU ISE
          alarmSeviyesi = 2 // Orta Öncelik: Bir giriş noktası ihlal edilmiş
        DEĞİLSE
          alarmSeviyesi = 1 // Düşük Öncelik: Sadece hareket algılandı
        BİTİR EĞER

        // Alarm sistemini ve bildirimleri başlat
        alarmAktif = DOĞRU
        calSiren(alarmSeviyesi) // Seviyeye göre farklı ses şiddeti veya türü
        
        // 7. Bildirim Gönderme
        mesaj = "Alarm Seviyesi " + alarmSeviyesi + ": Evde tehdit algılandı!"
        gonderBildirim("SMS", mesaj)
        gonderBildirim("Uygulama", mesaj)
        gonderBildirim("E-posta", mesaj)

      BİTİR EĞER // Yanlış alarm kontrolü sonu

    BİTİR EĞER // Tehdit algılama sonu

  BİTİR EĞER // Sistem aktif kontrolü sonu

  // 8. Alarm Sıfırlama Kontrolü
  // Bu kontrol, sistem kapalı olsa bile alarmı sıfırlamak için döngü içinde kalır.
  EĞER alarmAktif == DOĞRU ISE
    sifirlamaKomutuAlindi = kontrolEtSifirlamaKomutu() // Uygulamadan veya panelden gelen komut

    EĞER sifirlamaKomutuAlindi == DOĞRU ISE
      alarmAktif = YANLIŞ
      alarmSeviyesi = 0
      durdurSiren()
      durdurKameraKaydi()
      gonderBildirim("Uygulama", "Sistem sahibi tarafından devre dışı bırakıldı.")
    BİTİR EĞER
  BİTİR EĞER

  // 9. Bekleme ve Tekrar Kontrol
  // Sistemin işlemciyi yormaması için her döngü arasında kısa bir bekleme süresi eklenir.
  BEKLE(1 saniye)

BİTİR DÖNGÜ // Ana döngü sonu (teorik olarak hiç bitmez)


SON
