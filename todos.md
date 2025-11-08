# Teknik TODO Listesi: Web Tabanlı İkon Kütüphanesi

## 1. Mevcut Durum Analizi

Proje, şu anda bir Cloudflare Worker olarak çalışan ve URL parametreleri aracılığıyla dinamik olarak SVG sprite'ları oluşturan bir servistir.

-   **`icons/`**: Tüm SVG ikonların ham olarak bulunduğu klasör.
-   **`build.js`**: `icons/` klasöründeki tüm SVG'leri okuyup `dist/icons.json` adında tek bir JSON dosyasına sıkıştıran bir betik.
-   **`index.js`**: Gelen isteklere yanıt veren ana Cloudflare Worker kodu. URL'den aldığı ikon isimlerine göre `dist/icons.json` dosyasından ilgili SVG verilerini çekip birleştirerek tek bir SVG dosyası olarak döndürür.
-   **API**: `/api/icons` ve `/api/svgs` gibi iki adet endpoint ile ikon listesini ve ham SVG verilerini JSON olarak sunar.

## 2. Tespit Edilen Eksiklikler ve Mimari Sorunlar

Mevcut yapı, "web tabanlı bir ikon kütüphanesi" hedefi için yetersizdir.

-   **Kullanıcı Arayüzü (UI) Yok**: Kullanıcıların ikonları görebileceği, arayabileceği veya filtreleyebileceği bir arayüz mevcut değil. Bu, bir ikon kütüphanesinin en temel gereksinimidir.
-   **Sınırlı İşlevsellik**: Sadece SVG sprite oluşturulabiliyor. İkonları tek tek indirme, farklı formatlarda (PNG, vb.) alma, SVG kodunu kopyalama veya React/Vue gibi framework'ler için kullanım örnekleri sunma gibi özellikler yok.
-   **Ölçeklenebilirlik Sorunu**: Tüm ikonların tek bir JSON dosyasına yüklenmesi, ikon sayısı arttıkça Cloudflare Worker'ın bellek kullanımını artıracaktır. Bu, gelecekte performans sorunlarına ve limit aşımlarına yol açabilir.
-   **Frontend Odaklı Değil**: Proje bir backend servisi olarak tasarlanmış. Gerçek bir ikon kütüphanesi, genellikle bir npm paketi olarak dağıtılan ve frontend projelerine kolayca entegre edilebilen bir yapıya sahip olmalıdır.

## 3. Önerilen Mimari ve Çözüm Yolları

Projenin web tabanlı bir ikon kütüphanesine dönüşmesi için aşağıdaki mimari önerilmektedir.

-   **Frontend Uygulaması**:
    -   React, Vue veya Svelte gibi modern bir JavaScript framework'ü ile geliştirilmiş statik bir web sitesi.
    -   Bu site, tüm ikonların listelendiği bir galeri sayfası içermelidir.
    -   Kullanıcıların istedikleri ikonları kolayca bulabilmesi için bir arama ve filtreleme özelliği olmalıdır.
    -   Her bir ikon için, SVG kodunu kopyalama, farklı formatlarda indirme (SVG, PNG) ve framework bileşenleri (React, Vue) için kod örnekleri sunan bir detay sayfası olmalıdır.
-   **İkon İşleme (Pipeline)**:
    -   Mevcut `build.js` betiği geliştirilerek, sadece `icons.json` oluşturmakla kalmamalı, aynı zamanda:
        -   Her bir SVG'yi [SVGO](https://github.com/svg/svgo) gibi araçlarla optimize etmeli.
        -   İkonlar hakkında meta verileri (kategoriler, etiketler vb.) içeren bir JSON dosyası oluşturmalı.
        -   Frontend'in doğrudan kullanabilmesi için her ikonu ayrı bir dosya olarak `dist/icons` klasörüne kopyalamalı.
-   **API (Opsiyonel ama Önerilir)**:
    -   Mevcut Cloudflare Worker, arama ve filtreleme gibi dinamik işlemler için bir API sunucusu olarak kullanılabilir.
    -   Ancak, daha basit bir başlangıç için, tüm ikon meta verileri build aşamasında oluşturulup statik site ile birlikte sunulabilir. Bu, ek bir sunucu ihtiyacını ortadan kaldırır.
-   **NPM Paketi**:
    -   İkonların bir npm paketi olarak yayınlanması, geliştiricilerin projelerinde ikonları kolayca kullanmasını sağlar. Bu paket, her bir ikon için ayrı React/Vue bileşenleri içerebilir.

## 4. Teknik TODO Listesi

### Aşama 1: Temel Frontend Kurulumu

1.  **Frontend Framework'ü Seçimi ve Kurulumu**:
    -   [ ] Proje için bir frontend framework seçin (Örn: Next.js, Nuxt.js, SvelteKit).
    -   [ ] Seçilen framework ile yeni bir proje oluşturun ve mevcut repoya entegre edin (örneğin, `packages/web` veya `apps/web` gibi bir klasör altında).
2.  **İkon Verilerini Hazırlama**:
    -   [ ] `build.js` betiğini güncelleyerek `dist/metadata.json` adında bir dosya oluşturun. Bu dosya, her ikonun adını, kategorisini ve etiketlerini içeren bir dizi olmalıdır.
    -   [ ] `build.js` betiğinin, optimize edilmiş SVG'leri `dist/icons/` klasörüne ayrı ayrı kaydetmesini sağlayın.
3.  **Ana İkon Galerisi Sayfası**:
    -   [ ] `metadata.json` dosyasını okuyarak tüm ikonları listeleyen bir galeri sayfası oluşturun.
    -   [ ] Her bir ikon için bir kart bileşeni oluşturun. Bu kart, ikonun kendisini ve adını göstermelidir.

### Aşama 2: Gelişmiş Frontend Özellikleri

4.  **Arama ve Filtreleme**:
    -   [ ] Galeri sayfasına bir arama çubuğu ekleyin. Kullanıcıların ikon adlarına göre arama yapmasını sağlayın.
    -   [ ] İkonları kategorilere veya etiketlere göre filtreleme özelliği ekleyin.
5.  **İkon Detay Sayfası (veya Modalı)**:
    -   [ ] Bir ikona tıklandığında, o ikona ait detayların gösterildiği bir modal veya ayrı bir sayfa açın.
    -   [ ] Bu ekranda:
        -   [ ] İkonun daha büyük bir önizlemesini gösterin.
        -   [ ] "SVG Kopyala" butonu ekleyin.
        -   [ ] "SVG İndir" ve "PNG İndir" butonları ekleyin.
        -   [ ] React ve Vue için kullanım örneklerini gösteren kod blokları ekleyin.

### Aşama 3: İkon İşleme ve Optimizasyon

6.  **SVG Optimizasyonu**:
    -   [ ] `build.js` betiğine [SVGO](https://github.com/svg/svgo) entegrasyonu yapın. `icons/` klasöründeki tüm SVG'ler build sırasında otomatik olarak optimize edilmelidir.
7.  **PNG Üretimi**:
    -   [ ] Build sürecine, her bir SVG'den farklı boyutlarda (örn: 64x64, 128x128) PNG'ler üreten bir adım ekleyin. Bunun için [sharp](https://sharp.pixelplumbing.com/) gibi bir kütüphane kullanılabilir.

### Aşama 4: Dağıtım ve NPM Paketi

8.  **Frontend Dağıtımı**:
    -   [ ] Frontend uygulamasının build komutunu `package.json` dosyasına ekleyin.
    -   [ ] Vercel, Netlify veya Cloudflare Pages gibi bir platformda statik sitenin otomatik olarak dağıtılması için CI/CD pipeline'ı kurun.
9.  **NPM Paketi Oluşturma (İsteğe Bağlı)**:
    -   [ ] İkonları React/Vue bileşenleri olarak sarmalayan yeni bir paket oluşturun (örneğin, `packages/react-icons`).
    -   [ ] Bu paketi npm'e yayınlamak için gerekli yapılandırmayı yapın.

### Aşama 5: Mevcut Servisin Güncellenmesi

10. **Cloudflare Worker'ın Geleceği**:
    -   [ ] Frontend uygulaması hayata geçtikten sonra, mevcut Cloudflare Worker'ın amacını yeniden değerlendirin.
    -   [ ] Worker, dinamik SVG sprite oluşturma servisi olarak kalabilir veya sadece gelişmiş arama/filtreleme işlemleri için bir API sunucusu olarak hizmet verebilir. Ya da tamamen kullanımdan kaldırılabilir.
