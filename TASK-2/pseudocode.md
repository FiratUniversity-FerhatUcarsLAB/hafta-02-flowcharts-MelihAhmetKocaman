// ---------------------------------------------------------------------
// Fonksiyon 1: Kullanıcı Oturumunu ve Sepetini Başlatma
// Açıklama: Kullanıcı siteye girdiği anda çalışır ve misafir/üye sepetini hazırlar.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon OturumYonetimi()
    // Kullanıcının tarayıcısından oturum kimliği okunur.
    OKU kullanici_tarayici_cookie("oturum_kimligi") -> mevcut_oturum_kimligi

    EĞER mevcut_oturum_kimligi GEÇERSİZ İSE
        // Kullanıcı siteye ilk kez giriyor veya çerezleri silinmiş.
        OLUŞTUR yeni benzersiz "oturum_kimligi"
        KAYDET kullanici_tarayici_cookie("oturum_kimligi") = yeni "oturum_kimligi"
        ATA aktif_sepet_sahibi = yeni "oturum_kimligi" // Misafir kullanıcı
    DEĞİLSE
        // Mevcut oturumu olan bir kullanıcı.
        // Veritabanından bu oturum kimliğinin bir üye hesabına bağlı olup olmadığı kontrol edilir.
        OKU Veritabanı.Kullanicilar("oturum_kimligi" == mevcut_oturum_kimligi) -> bagli_kullanici_hesabi

        EĞER bagli_kullanici_hesabi VAR İSE
            // Kullanıcı daha önce giriş yapmış.
            ATA aktif_sepet_sahibi = bagli_kullanici_hesabi.kullanici_id
        DEĞİLSE
            // Kullanıcı giriş yapmamış bir misafir.
            ATA aktif_sepet_sahibi = mevcut_oturum_kimligi
        BİTİR EĞER
    BİTİR EĞER

    YAZ "Oturum ve sepet sahibi belirlendi: " + aktif_sepet_sahibi
BİTİR: Fonksiyon OturumYonetimi

// ---------------------------------------------------------------------
// Fonksiyon 2: Sepete Ürün Ekleme (Stok Kontrolü ile)
// Açıklama: Kullanıcı "Sepete Ekle" butonuna tıkladığında tetiklenir.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon SepeteUrunEkle(urun_id, istenen_miktar)
    // Ürünün stok durumu veritabanından okunur.
    OKU Veritabanı.Urunler("urun_id" == urun_id).stok_adeti -> mevcut_stok

    EĞER mevcut_stok >= istenen_miktar İSE
        // STOK YETERLİ
        // Kullanıcının mevcut sepeti veritabanından okunur.
        OKU Veritabanı.Sepetler("sahip_id" == aktif_sepet_sahibi) -> kullanici_sepeti

        // Ürünün sepette olup olmadığı kontrol edilir.
        OKU kullanici_sepeti.urunler("urun_id" == urun_id) -> sepetteki_urun

        EĞER sepetteki_urun VAR İSE
            // Ürün zaten sepette, sadece miktarı artırılır.
            GÜNCELLE sepetteki_urun.miktar = sepetteki_urun.miktar + istenen_miktar
        DEĞİLSE
            // Ürün sepette yok, yeni ürün olarak eklenir.
            EKLE {urun_id, istenen_miktar} TO kullanici_sepeti.urunler
        BİTİR EĞER

        KAYDET kullanici_sepeti TO Veritabanı.Sepetler
        YAZ "Başarılı: Ürün sepetinize eklendi."
    DEĞİLSE
        // STOK YETERSIZ
        YAZ "Hata: Üzgünüz, bu üründen stokta yeterli sayıda bulunmuyor."
    BİTİR EĞER
BİTİR: Fonksiyon SepeteUrunEkle

// ---------------------------------------------------------------------
// Fonksiyon 3: Sepet Görüntüleme ve Toplamları Hesaplama
// Açıklama: Kullanıcı sepet sayfasına gittiğinde çalışır.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon SepetiGoster()
    ATA ara_toplam = 0
    ATA genel_toplam = 0

    // Aktif kullanıcının sepetindeki tüm ürünler okunur.
    OKU Veritabanı.Sepetler("sahip_id" == aktif_sepet_sahibi).urunler -> sepet_urun_listesi

    YAZ "--- SEPETİNİZ ---"
    DÖNGÜ sepet_urun_listesi içindeki her bir 'sepet_urunu' için
        // Her bir ürünün detayları (fiyat, ad vb.) ürünler tablosundan okunur.
        OKU Veritabanı.Urunler("urun_id" == sepet_urunu.urun_id) -> urun_detaylari

        HESAPLA urun_toplam_fiyat = urun_detaylari.fiyat * sepet_urunu.miktar
        YAZ urun_detaylari.ad + " - Miktar: " + sepet_urunu.miktar + " - Toplam: " + urun_toplam_fiyat + " TL"

        HESAPLA ara_toplam = ara_toplam + urun_toplam_fiyat
    BİTİR DÖNGÜ

    YAZ "------------------"
    YAZ "Ara Toplam: " + ara_toplam + " TL"

    OKU Veritabanı.Sepetler("sahip_id" == aktif_sepet_sahibi) -> sepet_bilgisi
    HESAPLA kargo_ucreti = KargoHesapla(sepet_bilgisi.adres) // Fonksiyon 5'i çağır
    HESAPLA indirim_tutari = sepet_bilgisi.uygulanan_indirim

    HESAPLA genel_toplam = ara_toplam - indirim_tutari + kargo_ucreti

    YAZ "İndirim: -" + indirim_tutari + " TL"
    YAZ "Kargo Ücreti: +" + kargo_ucreti + " TL"
    YAZ "Genel Toplam: " + genel_toplam + " TL"
BİTİR: Fonksiyon SepetiGoster

// ---------------------------------------------------------------------
// Fonksiyon 4: İndirim Kodu Uygulama
// Açıklama: Kullanıcının girdiği indirim kodunu kontrol eder ve uygular.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon IndirimKoduUygula(girilen_kod)
    // Girilen kodun veritabanında var olup olmadığı kontrol edilir.
    OKU Veritabanı.IndirimKodlari("kod" == girilen_kod) -> kupon

    EĞER kupon YOK VEYA kupon.aktif_degil İSE
        YAZ "Hata: Geçersiz veya aktif olmayan indirim kodu."
    DEĞİLSE
        EĞER BUGUNUN_TARIHI > kupon.son_kullanma_tarihi İSE
            YAZ "Hata: Bu indirim kodunun süresi dolmuş."
        DEĞİLSE
            OKU sepet_ara_toplami
            EĞER sepet_ara_toplami < kupon.minimum_tutar İSE
                YAZ "Hata: Bu kodu kullanmak için sepet tutarınız en az " + kupon.minimum_tutar + " TL olmalıdır."
            DEĞİLSE
                // Tüm kontroller başarılı. İndirim uygulanır.
                HESAPLA indirim_tutari = (sepet_ara_toplami * kupon.indirim_orani / 100)
                GÜNCELLE Veritabanı.Sepetler("sahip_id" == aktif_sepet_sahibi).uygulanan_indirim = indirim_tutari
                YAZ "Başarılı: İndirim uygulandı."
            BİTİR EĞER
        BİTİR EĞER
    BİTİR EĞER
BİTİR: Fonksiyon IndirimKoduUygula

// ---------------------------------------------------------------------
// Fonksiyon 5: Kargo Hesaplama
// Açıklama: Adres ve sepet tutarına göre kargo ücretini hesaplar.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon KargoHesapla(teslimat_adresi)
    ATA kargo_ucreti = 15.00 // Varsayılan kargo ücreti

    OKU sepet_ara_toplami
    OKU Veritabanı.Ayarlar("ucretsiz_kargo_limiti") -> ucretsiz_kargo_limiti

    EĞER sepet_ara_toplami >= ucretsiz_kargo_limiti İSE
        ATA kargo_ucreti = 0
    DEĞİLSE
        EĞER teslimat_adresi.sehir == "İstanbul" İSE
            ATA kargo_ucreti = 12.00
        BİTİR EĞER
    BİTİR EĞER

    DÖNDÜR kargo_ucreti
BİTİR: Fonksiyon KargoHesapla

// ---------------------------------------------------------------------
// Fonksiyon 6: Ödeme ve Siparişi Tamamlama
// Açıklama: En kritik adımdır. Ödemeyi alır, siparişi oluşturur, stoğu günceller.
// ---------------------------------------------------------------------
BAŞLA: Fonksiyon OdemeYap(kart_bilgileri, adres_bilgileri)
    // 1. ADIM: Son tutarı tekrar hesapla ve doğrula.
    HESAPLA son_tutar = (sepet_ara_toplami - uygulanan_indirim + kargo_ucreti)

    // 2. ADIM: Ödeme hizmet sağlayıcısına istek gönder.
    YAZ "Ödeme işlemi için bankaya yönlendiriliyor..."
    GÖNDER {kart_bilgileri, son_tutar} TO OdemeServisi -> odeme_yaniti

    // 3. ADIM: Ödeme yanıtını kontrol et.
    EĞER odeme_yaniti.durum == "BASARILI" İSE
        YAZ "Ödeme başarılı. Sipariş oluşturuluyor..."
        // -- VERİTABANI İŞLEMİ BAŞLAT (Hepsi başarılı olmalı ya da hiçbiri) --
        
        // a) Siparişi veritabanına kaydet.
        OLUŞTUR yeni_siparis = {kullanici_id, adres_bilgileri, sepet_urun_listesi, son_tutar, tarih}
        KAYDET yeni_siparis TO Veritabanı.Siparisler

        // b) Satılan ürünlerin stoklarını düş.
        DÖNGÜ sepet_urun_listesi içindeki her bir 'satilan_urun' için
            OKU Veritabanı.Urunler("urun_id" == satilan_urun.urun_id).stok_adeti -> guncel_stok
            HESAPLA yeni_stok = guncel_stok - satilan_urun.miktar
            GÜNCELLE Veritabanı.Urunler("urun_id" == satilan_urun.urun_id).stok_adeti = yeni_stok
        BİTİR DÖNGÜ

        // c) Kullanıcının sepetini temizle.
        SİL Veritabanı.Sepetler("sahip_id" == aktif_sepet_sahibi)

        // -- VERİTABANI İŞLEMİ BİTİR --

        YAZ "Siparişiniz başarıyla tamamlandı! Sipariş Numaranız: " + yeni_siparis.id
        // Kullanıcıya onay e-postası gönderilir.
        
    DEĞİLSE
        // Ödeme başarısız oldu.
        YAZ "Hata: Ödeme işlemi başarısız oldu. Hata mesajı: " + odeme_yaniti.mesaj
        // Hiçbir veritabanı işlemi (stok düşme, sipariş kaydı) yapılmaz.
    BİTİR EĞER
BİTİR: Fonksiyon OdemeYap
