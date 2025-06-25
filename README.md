# Vue.js ile Özel Video Oynatıcı

Bu proje, Vue 3 ve Vite kullanılarak oluşturulmuş özel bir HTML5 video oynatıcıdır. Standart video oynatıcının yerine geçen, modern ve özelleştirilebilir bir arayüz sunar.

## Özellikler

- **Oynat/Durdur:** Videoyu oynatma ve durdurma kontrolü.
- **İlerleme Çubuğu:** Videonun mevcut ilerlemesini gösterir ve tıklanarak videoda istenen zamana gidilmesini sağlar.
- **Zaman Göstergesi:** Videonun mevcut süresini ve toplam süresini `HH:MM:SS` formatında gösterir.
- **Ses Kontrolü:** Ses seviyesini ayarlamak için bir kaydırıcı (slider) ve sesi tamamen kapatmak için bir sessize alma düğmesi.
- **Tam Ekran:** Videoyu tam ekran moduna alma ve çıkma.
- **Duyarlı Tasarım:** Kontrollerin görünümü, fare imlecinin oynatıcı üzerinde olup olmamasına göre değişir.

## Dosya Yapısı ve Mimarisi

Video oynatıcı, `src/components/video-player/` klasörü altında üç ana bileşenden oluşur:

### 1. `Player.vue`

Bu, video oynatıcının ana bileşenidir.

- **Sorumlulukları:**
  - HTML `<video>` elementini barındırır.
  - Video oynatıcının genel durumunu (state) yönetir. Bu durumlar arasında `isPlaying` (oynatılıyor mu), `currentTime` (mevcut zaman), `duration` (toplam süre), `volume` (ses seviyesi) ve `isMuted` (sessiz mi) bulunur.
  - `<video>` elementinin olaylarını (`play`, `pause`, `timeupdate`, `volumechange` vb.) dinler ve bu olaylara göre durumu günceller.
  - Video kontrol fonksiyonlarını (`togglePlayPause`, `seek`, `updateVolume` vb.) içerir.
  - `Controls.vue` bileşenini render eder ve ona gerekli `props`'ları (durumları) ve olayları (`emit`'leri) bağlar.

### 2. `Controls.vue`

Bu bileşen, video kontrollerinin kullanıcı arayüzünü (UI) oluşturur.

- **Sorumlulukları:**
  - Oynat/Durdur düğmesi, ilerleme çubuğu, zaman göstergesi, ses kontrolü ve tam ekran düğmesini içerir.
  - Gerekli tüm durumları `Player.vue` bileşeninden `props` olarak alır.
  - Kullanıcı bir kontrolle etkileşime girdiğinde (örneğin, oynat düğmesine tıkladığında), `Player.vue`'daki ilgili fonksiyonu tetiklemek için bir olay (`emit`) yayınlar.
  - Zamanı okunabilir bir formata (`HH:MM:SS`) dönüştürmek için `formatTime` adında bir yardımcı fonksiyon kullanır.
  - Ses kontrolü için `VolumeSlider.vue` bileşenini içerir.

### 3. `VolumeSlider.vue`

Bu, ses seviyesi kaydırıcısı ve sessize alma düğmesi için ayrılmış küçük bir bileşendir.

- **Sorumlulukları:**
  - `Controls.vue`'dan `volume` ve `isMuted` durumlarını `props` olarak alır.
  - Ses seviyesine ve sessize alınma durumuna göre farklı ses ikonları gösterir.
  - Kullanıcı ses seviyesini değiştirdiğinde veya sessize alma düğmesine tıkladığında `Controls.vue`'a olaylar (`emit`) yayınlar.

## Nasıl Çalışır?

Bileşenler arasındaki iletişim, `props` ve `emit` mekanizması üzerine kuruludur:

1.  **State Yönetimi:** `Player.vue`, videonun tüm durumunu kendi içinde tutar.
2.  **Props Down:** `Player.vue`, bu durumu `props` aracılığıyla alt bileşen olan `Controls.vue`'a aktarır. `Controls.vue` da `VolumeSlider.vue`'a gerekli `props`'ları geçirir. Bu sayede alt bileşenler, her zaman güncel durumu görüntüleyebilir.
3.  **Events Up:** Kullanıcı `Controls.vue` veya `VolumeSlider.vue` üzerindeki bir düğmeye tıkladığında, bu bileşenler bir `emit` ile olayı üst bileşene bildirir. Örneğin, oynat düğmesine tıklandığında `Controls.vue`, `togglePlayPause` adında bir olay yayınlar.
4.  **Olayların Yakalanması:** `Player.vue`, bu `emit`'leri dinler ve yakaladığı olaya karşılık gelen fonksiyonu çalıştırır (örneğin, `togglePlayPause()` fonksiyonu ile videoyu oynatır/durdurur).

Bu tek yönlü veri akışı (one-way data flow), uygulamanın daha öngörülebilir ve yönetilebilir olmasını sağlar.

---

## Geliştirme Ortamı (Vue 3 + Vite)

Bu şablon, Vite ile Vue 3 geliştirmeye başlamanıza yardımcı olacaktır. Şablon, Vue 3 `<script setup>` SFC'lerini kullanır, daha fazla bilgi için [script setup belgelerine](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) göz atın.

### Kurulum

Projeyi klonladıktan sonra bağımlılıkları yükleyin:

```bash
npm install
```

### Geliştirme Sunucusunu Başlatma

```bash
npm run dev
```

### Derleme (Build)

```bash
npm run build
```