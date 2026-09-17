# SEO Master Checklist — v2.0

**Standart:** Her müşteri sitesinde, her denetimde bu liste çalıştırılır. Sıfırdan "neye bakacağız" diye düşünülmez; `PASS / FAIL / N/A` işaretlenerek ilerlenir.

**Kaynak:** 60+ gerçek denetim vakası (Haz–Eyl 2026) + Google Search Central + 2026 Ahrefs/Semrush checklist'leri.

### Etiketler

| Etiket | Anlamı |
|---|---|
| `[G]` | **Google zorunluluğu.** Uymazsa crawl / index / rich-result / policy problemi çıkar. Müşteri raporunda "Google kuralı" diye yazılabilecek tek kategori. |
| `[B]` | **Best practice.** Google şartı değil, yapılması önerilir. Rapora "önerilir" diye yazılır, "ihlal" diye değil. |
| `[A]` | **Ajans standardı.** Geçmiş vakalardan çıkmış kendi çalışma kuralımız. Bize özel, Google'a mal edilmez. |
| `[K]` | **Koşullu.** Sadece o tip sitede uygulanır. |

**Öncelik:** `P0` trafik/indeks riski · `P1` büyüme engeli · `P2` optimizasyon

---

## FAZ 0 — Ölçüm protokolü `P0`

Bu faz atlanırsa denetimin geri kalanı yanlış bulgu üretir. 5 ayrı yanlış rapor bu adımların eksikliğinden çıktı.

| # | Kural | Neden |
|---|---|---|
| 0.1 `[A]` | HTTP taraması `ThreadPoolExecutor(3)`, gerçek Chrome UA. Her 404'ü 0.35s aralıkla 2–3 kez tekrar doğrula | 8–12 thread'de sunucu 429 yerine 404 döner; bir vakada 53 canlı sayfa "silinmiş" göründü |
| 0.2 `[A]` | Durum kodu ölçümü `allow_redirects=False`. Sadece 200 dönenlerde HTML/canonical/hash oku | Redirect takip edilince 9 blog için sahte "kopya içerik" bulgusu üretildi |
| 0.3 `[A]` | JSON-LD'yi **parse et** (`json.loads`), regex atma. `@graph` gez, `@type`'ı hem str hem list işle | `@type:["Organization","Store"]` dizi formatı regex'e takılmıyor |
| 0.4 `[A]` | Kelime sayımı **gövde-only**: `<nav> <header> <footer> <head> <script>` sil, sonra say | Nav+footer dahil edilince 52 geçiş sayıldı, gerçek 4'tü |
| 0.5 `[A]` | Tarih/spec kontrolü **değer** arar, etiket kelimesi değil. Tarihte hem `GG.AA.YYYY` hem `9 Tem, 2026` formatını tara | Tek regex "hiç tarih yok" yanlış hükmü verdi |
| 0.6 `[A]` | GSC kıyası: **Country filtresi + eşit uzunlukta iki 28/30 günlük pencere**. Kısmi ay ile tam ay kıyaslanmaz | Filtresiz çeyrek kıyası −%34 derken doğru pencere −%77 gösterdi |
| 0.7 `[A]` | Site toplamı = `search_analytics(dimensions:[])`. `top_pages` toplamı uzun kuyruğu kaçırır, düşüşü abartır | Bir sitede top-pages ile −%21, gerçek toplam +%3 |
| 0.8 `[A]` | GSC 2–3 gün gecikir. "Son 2 gün sıfır" paniğinde önce 2–3 başka property'yi aynı günde kontrol et | Hepsi aynı günde kesiliyorsa global gecikme, siteye özel değil |
| 0.9 `[A]` | Site yeni deploy edildiyse deploy **öncesi** GSC penceresinden "Google taramamış" sonucu çıkarma | Takvim hatası "en kritik teknik sorun" diye raporlandı |
| 0.10 `[A]` | Yasaklı ifade/iddia taramasında sabit liste "temiz" hükmü vermez. Listeyi her turda genişlet, eşleşmenin **yönünü** oku | 87/87 PASS sonrası 5 gerçek kalıntı çıktı |
| 0.11 `[A]` | Başka bir AI'ın "canlıda gördüm" iddiasına güvenme; koduna güven, canlı iddiasını kendin doğrula | 3 kez bayat cache çıktı |
| 0.12 `[A]` | `site:domain.com` indeks **sayısı** ölçmez — sadece hızlı varlık kontrolüdür | Google tüm indeksli URL'leri bu operatörde göstermiyor |

---

## FAZ 1 — GSC sağlık kontrolü `P0`

Her şeyden önce. Manuel işlem varsa aşağıdaki 17 fazın hiçbiri sonuç vermez.

- [ ] **1.1 `[G]` Manual Actions temiz** — manuel işlem varsa denetim durur, önce o çözülür
- [ ] **1.2 `[G]` Security Issues temiz** — hack/malware/zararlı içerik uyarısı yok
- [ ] **1.3 `[G]` Property doğrulanmış** ve doğru format (yerel site konfigürasyonu; atlamadan önce canlı property listesine bak — config eskiyor)
- [ ] **1.4 `[B]` Page Indexing raporu incelendi** — hangi kategori büyüyor (Crawled-not-indexed / Discovered / Alternate page with canonical / Soft 404). **Arayüzden bakılır, API'si yok**
- [ ] **1.5 `[B]` Crawl Stats incelendi** — 5xx artışı, ortalama yanıt süresi, host durumu
- [ ] **1.6 `[B]` Core Web Vitals raporu incelendi**
- [ ] **1.7 `[B]` Google-selected canonical** birkaç örnek URL'de kontrol edildi

---

## FAZ 2 — Erişim, tarama, render `P0`

- [ ] **2.1 `[G]` robots.txt 200 dönüyor**, yanlışlıkla `Disallow: /` yok
- [ ] **2.2 `[G]` Kritik CSS/JS robots ile engellenmiyor** — render bozulur
- [ ] **2.3 `[G]` CDN / Cloudflare / WAF / bot koruması Googlebot'u engellemiyor** — bot koruması aktif sitelerde ayrıca kontrol edilir
- [ ] **2.4 `[G]` robots.txt indeks engellemek için kullanılmıyor** — indeks dışı için `noindex`, erişim için şifre
- [ ] **2.5 `[G]` JS ile üretilen kritik içerik Google render'ında mevcut** — URL Inspection → rendered HTML ile doğrula
- [ ] **2.6 `[G]` Lazy-load içerik scroll/click zorunluluğu olmadan erişilebilir**
- [ ] **2.7 `[G]` Linkler gerçek `<a href>`** — `javascript:void` ile duran kritik navigasyon yok (bir vakada iç sayfalarda 2 üst kategori ölü linkti, anasayfada çalışıyordu)
- [ ] **2.8 `[A]` Baş-slash'sız göreli link yok** — `tr/...` → `/tr/tr/...` üretir. **Bulgu vermeden önce `<base href>` kontrol et** — base tanımlıysa göreli yol doğru çözülür, bu bir bulgu değildir (yanlış alarm kaynağı)
- [ ] **2.9 `[B]` AI crawler politikası bilinçli** — GPTBot / PerplexityBot / CCBot / Google-Extended. **Not:** Google-Extended engellenmesi normal Google Search sıralamasını etkilemez, bu bir SEO `P0`'ı değildir
- [ ] **2.10 `[B]` 1000+ sayfalı sitede** crawl bütçesi: kırık iç link + parametreli URL çoğalması
- [ ] **2.11 `[A]` Sitemap dışı sistem URL aileleri sentetik test edildi** — **sitemap crawl ≠ site crawl.** Test seti: `/index.php/<yol>` alt ağacı · iç arama (`/s?q=`) · pagination · filtre/query varyantı · uppercase · trailing slash · dil kökleri · `llms.txt`/feed · slider/demo modül yolları · rastgele 404. (Bir vakada 34 URL'lik sitemap taraması temiz görünürken `/index.php/*` paralel ağacı, bir slider test sayfası ve 3 gizli modül indexable çıktı)

---

## FAZ 3 — Sitemap & robots içeriği `P0`

- [ ] **3.1 `[G]` sitemap.xml 200**, index sitemap alt dosyaları 200
- [ ] **3.2 `[G]` Sadece indekslenmesi istenen canonical URL'ler sitemap'te**
- [ ] **3.3 `[G]` Sitemap içinde 301 / 404 / 5xx / `noindex` URL yok** — bir sitenin sitemap'inin %78'i 301'liydi
- [ ] **3.4 `[G]` URL'ler absolute**, host/protokol standardı doğru
- [ ] **3.5 `[A]` `lastmod` gerçek güncelleme tarihi** — 2 sn arayla iki kez çek, değer sabit kalmalı. Her istekte `date('Y-m-d')`/`now()` üreten yapı bug'dır (3 ayrı sitede çıktı)
- [ ] **3.6 `[B]` `priority` ve `changefreq` ile uğraşma** — Google ikisini de kullanmıyor
- [ ] **3.7 `[A]` Sitemap kapsama açığı** = GSC'de gösterim alan sayfa kümesi − sitemap URL kümesi (bir sitede 87 indeksli sayfa sitemap dışıydı)
- [ ] **3.8 `[B]` robots.txt içinde `Sitemap:` satırı** tanımlı, GSC'ye gönderilmiş
- [ ] **3.9 `[A]` Eski/geçersiz sitemap dosyaları GSC'den silinmiş** (WordPress `page-sitemap.xml` kalıntıları)

> Sitemap API çıktısındaki `indexed: "0"` **indeksleme sorunu değil** — API her mülk için 0 döner. İndeks kanıtı olarak kullanma.

---

## FAZ 4 — İndeksleme & Canonical `P0`

- [ ] **4.1 `[G]` Canonical `$_SERVER['REQUEST_URI']` ile üretilmiyor** — query string dahil edilirse her `?utm_source=` / `?fbclid=` yeni canonical yaratır (3 ayrı sitede çıktı)
- [ ] **4.2 `[G]` Sayfa başına tek canonical** — header + view'da çift basım yok. (`rel="index"` canonical değildir, yanlış alarm verme)
- [ ] **4.3 `[G]` Normal sayfalar self-canonical**, alt sayfalar anasayfaya canonical vermiyor (bir vakada tüm site anasayfaya canonical veriyordu)
- [ ] **4.4 `[G]` Canonical hedefi 200** — redirect veren URL'ye canonical verilmiyor
- [ ] **4.5 `[G]` Canonical + 301 + sitemap + hreflang dördü aynı nihai URL'yi gösteriyor** (bir site aylarca bu yüzden geri döndü)
- [ ] **4.6 `[G]` Soft-404 yok** — olmayan sayfa boş gövdeyle 200 dönmüyor. **Asıl kural 200 dönmemesi**; 404 mü 410 mu ikincil, ikisi de geçerli
- [ ] **4.7 `[B]` Etiket / iç arama / filtre sayfaları `noindex,follow`**

### 4.8 İndeks ölçümü — doğru kaynak `[A]`

Üç ayrı şey birbirine karıştırılmaz:

| Ölçmek istediğin | Doğru kaynak | Yanlış kaynak |
|---|---|---|
| **İndeks kapsaması** | GSC Page Indexing raporu (arayüz) + kritik URL'lerde URL Inspection | `sitemaps.indexed` (hep 0), `site:` operatörü |
| **Görünürlük kümesi** | `search_analytics(dimensions:["page"])` — gösterim alan benzersiz URL (`#fragment` normalize et) | Bunu "indeksli URL sayısı" diye raporlama |
| **Kapsama açığı** | sitemap URL kümesi ↔ gösterim alan URL kümesi farkı | — |

İndeks tarama aracının çıktısındaki `⚠️ Belirsiz` **indeks dışı demek değil** (bir sitede 221'in 131'i sıralanıyordu). O dosyada güvenilir olan tek şey **HTTP sütunu** ve **Google Canonical ≠ URL** sütunu.

- [ ] **4.9 `[B]` İndeks gönderimi:** 1–5 kritik URL → URL Inspection / Request Indexing. Çok sayıda URL → düzgün sitemap + doğru `lastmod`. Aynı URL'yi tekrar tekrar göndermek crawl'ı hızlandırmaz. `[A]` operasyon pratiğimiz: öncelikli 10 URL batch

---

## FAZ 5 — URL standardı & Yönlendirme `P0`

- [ ] **5.1 `[G]` Tek host standardı** — HTTP→HTTPS ve www↔non-www tek atlamada 301. Her iki taraf da 200 dönmüyor (bir vakada iki taraf 200, canonical non-www, 301 yok → indeks çöküşü)
- [ ] **5.2 `[G]` Dil kökü / trailing slash tek standart** — `/`, `/tr`, `/tr/`, `/home`, `/anasayfa`, `index.php` hepsi tek adrese iniyor
- [ ] **5.3 `[G]` Büyük/küçük harf varyantları** kontrol edildi
- [ ] **5.4 `[G]` Kalıcı taşıma 301/308, geçici 302/307**
- [ ] **5.5 `[G]` Redirect chain yok** — eski URL doğrudan nihai hedefe gider
- [ ] **5.6 `[A]` Redirect loop yok** — döngü testinde **ham URL** karşılaştır, normalize edilmiş değil
- [ ] **5.7 `[G]` Kırık link silinmez, 301'lenir.** Konu bazlı hedefe — toplu anasayfaya yığma many-to-one soft-404 sayılabilir
- [ ] **5.8 `[A]` 301 hedeflerinin tamamı HTTP 200** — henüz açılmamış hedef ön koşul işaretlenir (bir vakada 72 URL var olmayan bir hizmet sayfasına gidiyordu)
- [ ] **5.9 `[B]` İç linkler redirect kaynağına değil nihai URL'e gidiyor**
- [ ] **5.10 `[A]` GSC'de gösterim alan 404'ler** tespit edilip 301'lendi (bir relaunch sonrası 130 URL 404, tıklamanın %53'ü kayıp)
- [ ] **5.11 `[B]` Domain/URL migration'da eski redirect'ler uzun süre korunur** — en az 1 yıl
- [ ] **5.12 `[A]` Redirect INSERT'leri idempotent** (`WHERE NOT EXISTS`)
- [ ] **5.13 `[A]` Asset yolları dil router'ına takılmıyor** (`/assets/katalog.pdf` → `/tr/assets/...` olmamalı)

---

## FAZ 6 — On-page `P1`

- [ ] **6.1 `[G]` Boş title yok**
- [ ] **6.2 `[B]` Her önemli URL'de ayırt edilebilir title** — şişkin site-geneli suffix / çift marka eki yok, bozuk dil şablonu yok
- [ ] **6.3 `[B]` Title kalıbı:** kelime başta, marka sonda, CTR kancası (yıl / "güncel"). **Google'ın sabit karakter limiti yoktur**, ~60 iyi pratiktir — "zorunlu" diye raporlama
- [ ] **6.4 `[B]` Benzersiz meta description** — boş description'a jenerik site açıklaması otomatik basılmıyor; birebir aynı description grubu yok. Google snippet'i yine sayfadan seçebilir, bu garanti bir alan değil
- [ ] **6.5 `[A]` Öncelik sırası:** tüm description'ları topluca doldurma; **yüksek gösterim + düşük CTR** olan 10–15 sayfaya odaklan
- [ ] **6.6 `[G]` Gizli / manipülatif ana başlık yok** — `position:absolute;left:-9999px` ile gizlenmiş H1 (bir sitede aylarca yerel kelime kaybettirdi). Computed style ile doğrula
- [ ] **6.7b `[B]` Çift H1'de önce renderer'a ve içerik kaynağına bak** — hero component'i içerik alanının tamamını basıyorsa ya da CMS gövdesinde H1 varsa kök neden şablon/veridedir; sayfa sayfa yama yapma (bir vakada 13 sayfa, kök neden DB gövdesindeki H1'lerdi)
- [ ] **6.7 `[B]` Sayfada tercihen tek, açık ve görünür ana H1** — heading sayısı/sırası Google'ın ranking şartı değildir; iki H1 görüldü diye "kritik SEO fail" yazma
- [ ] **6.8 `[B]` Heading hiyerarşisi semantik** — boş `<h2></h2>` yok, sırf font boyutu için H kullanılmıyor, şablondan kalan başıboş H4/H5 temizlendi
- [ ] **6.9 `[A]` Carousel/slider klonu gereksiz başlık çoğaltmıyor** (Swiper, Owl vb.)
- [ ] **6.10 `[G]` Anlam taşıyan görsellerde alt eksik/placeholder değil** — `alt="alt"`, dosya yolu, "Hizmet Başlığı" yer tutucusu yok. **Dekoratif görselde `alt=""` doğru uygulamadır** `[B]`
- [ ] **6.11 `[B]` Dil tutarlılığı** — TR sayfada İngilizce form/breadcrumb kalmamış, sabit metinler çeviri fonksiyonundan geçiyor
- [ ] **6.12 `[B]` `meta_keywords` ile vakit harcanmıyor** — panelden kaldır ama **DB'deki eski değeri silme**
- [ ] **6.13 `[K][B]` İnce sayfa** — kategori/hub sadece başlık + model kodundan ibaret değil (bir vakada 14 kategori → 322–429 kelime + karşılaştırma tablosu + iç link)

---

## FAZ 7 — Yapısal veri & Sosyal `P1`

- [ ] **7.1 `[G]` UYDURMA SCHEMA YOK** — `AggregateRating`, `Review`, olmayan kişi adı, olmayan fiyat/stok. Google yapısal veri spam politikası ihlali. Bir sitede 18 sayfa / 3.173 uydurma yorum → Ağustos 2026 Spam Update'te çöktü
- [ ] **7.2 `[G]` Schema görünür sayfa içeriğiyle çelişmiyor** — firma adı/adres/telefon gerçek
- [ ] **7.3 `[G]` JSON syntax geçerli**, Rich Results Test yapıldı
- [ ] **7.4 `[G]` `datePublished` / `dateModified` gerçek** — her istekte `now()` üretilmiyor, ikisi aynı tarihi basmıyor
- [ ] **7.5 `[G]` Schema `author` = "admin" olmaz**
- [ ] **7.6 `[A]` Çift schema yok** — BreadcrumbList hem kütüphaneden hem elle basılmıyor
- [ ] **7.7 `[A]` Tek `@graph` bloğu** — collector pattern: sayfa boyunca `schema_add()` ile topla, sonda tek seferde render et. Google şartı değil, bakım/duplicate/`@id` bağlantısı için bizim standardımız
- [ ] **7.8 `[B]` Product kullanımı:** rich result uygunluğu için `offers`, `review`, `aggregateRating` üçünden **en az biri** gerekir. Üçü de gerçekte yoksa **uydurma**; `ItemPage` kullan veya Product'ı rich-result beklentisi olmadan bırak. Sadece GSC uyarısını susturmak için yapay veri eklenmez ("Missing field offers")
- [ ] **7.9 `[B]` Sayfa tipine göre schema:** Organization + sektörel LocalBusiness alt tipi (RecyclingCenter, RoofingContractor, AutoRepair, Store…) + WebSite · ürün `ItemPage` · hizmet `Service` · blog `BlogPosting` · liste `CollectionPage + ItemList` · galeri `ImageGallery` · iletişim `ContactPage + ContactPoint` · hakkımızda `AboutPage` · iç sayfalar `BreadcrumbList`
- [ ] **7.10 `[B]` Çok şubeli firmada** her şube ayrı LocalBusiness, merkeze `branchOf` / `parentOrganization`
- [ ] **7.11 `[B]` `hasOfferCatalog`** dinamik bağlanır — panelden hizmet eklenince schema otomatik güncellenir
- [ ] **7.12 `[B]` Firma bilgileri schema'da:** kuruluş yılı, açık adres, telefon, e-posta, çalışma saatleri, hizmet bölgesi, sosyal profiller. WhatsApp `sameAs` değildir
- [ ] **7.13 `[A]` Schema `@id` ve OG URL'leri query string'siz** — `?utm_*` kimliği bölmesin
- [ ] **7.14 `[B]` Open Graph + Twitter Card** var, sayfa bazlı, görsel ilgili içerikten
- [ ] **7.15 `[B]` `FAQPage` ekleme** — Google FAQ rich result'ı kaldırdı, SEO getirisi yok

---

## FAZ 8 — Çok dilli `[K]` `P0`

- [ ] **8.1 `[G]` `<html lang>` doğru**
- [ ] **8.2 `[G]` Sabit tek hreflang etiketi yok** — her sayfaya `hreflang="en" → anasayfa` basmak hiç hreflang olmamaktan zararlı
- [ ] **8.3 `[G]` Gerçek karşılığı olmayan sayfada yapay alternate üretilmiyor** — `/en` anasayfasına fallback yasak
- [ ] **8.4 `[G]` Karşılıklı eşleşme** — her hreflang URL kendisini de içerir, hedefler 200
- [ ] **8.5 `[G]` Tek dilli sitede sahte hreflang kümesi basılmıyor**
- [ ] **8.6 `[B]` `x-default` uygun durumlarda tanımlı** — önerilir, zorunlu değil
- [ ] **8.7 `[A]` Dil switcher mutlak URL üretiyor** — `href="en/"` göreli yapısı alt sayfalarda 404 / `/tr/tr/` üretir
- [ ] **8.8 `[G]` canonical + hreflang + schema `@id` + sitemap aynı nihai URL'yi gösteriyor**

---

## FAZ 9 — Mimari, iç link, kanibalizasyon `P0`

- [ ] **9.1 `[G]` Önemli her sayfanın en az bir crawlable iç linki var** — orphan page kontrolü
- [ ] **9.2 `[G]` 404 veren iç link yok**
- [ ] **9.3 `[B]` Anchor text hedef sayfayı anlatıyor**, breadcrumb mantıklı
- [ ] **9.4 `[B]` En önemli ticari sayfalar fazla derinde değil**

### Kanibalizasyon

- [ ] **9.5 `[A]` Kelime bazlı URL sahipliği çıkar** — `search_analytics(dimensions:["query","page"])`. **2+ URL görünmesi tek başına kanibalizasyon değildir.** Kanibalizasyon adayı = aynı niyette **istikrarsız URL değişimi / yanlış URL'nin sıralanması / sinyal bölünmesi / performans kaybı**
- [ ] **9.6 `[A]` HUB ↔ alt kategori ayrıştırması** — uzun ifade HUB'a, kısa ticari ifade kategoriye
- [ ] **9.7 `[A]` Konsolidasyon öncesi GSC trafik verisi çekilir.** 301/silme listesi trafik verisi görülmeden verilmez — iki kez gerçek trafikli sayfa listede yakalandı
- [ ] **9.8 `[A]` Temiz sürümün indekse yansıdığı doğrulanmadan sayfa silinmez/301'lenmez**

**Konsolidasyon formülü `[A]`:** 90 günde tık getiren sayfa korunur, 0 tık 301'lenir. **İkiz-sayfa kuralı tık kuralını ezer.**
Muafiyet — bu 5 durumdan biri varsa 0 tık tek başına yeterli değil, elle karar ver:
`① 90 günden yeni sayfa` · `② gösterim > 0 ama CTR düşük (title/snippet sorunu olabilir)` · `③ backlink taşıyor` · `④ dönüşüm/landing sayfası` · `⑤ sezonluk iş`

- [ ] **9.9 `[A]` Doorway / ince sayfa çoğaltma yok** — yeni ticari blog yazmak parçalanmayı düzeltmez, büyütür (bir sitede 2 kez kanıtlandı)
- [ ] **9.10 `[A]` Ticari niyetli dedike landing var** — ticari kelime anasayfada sıkışmıyor (bir sitede −%40'ın kök nedeni)
- [ ] **9.11 `[A]` Portföy içi kanibalizasyon** — aynı ajansın iki müşterisi aynı SERP'te çakışıyorsa kelime kümesi tek müşteriye atanır. Aynı sektörde birden fazla müşteri varsa çakışma haritası çıkarılır ve kayıt altına alınır
- [ ] **9.12 `[B]` "Benzer ürünler" aynı kategoriden** — statik liste değil
- [ ] **9.13 `[B]` İç link bağlamsal:** blog → ürün, ürün → rehber/fiyat sayfası

---

## FAZ 10 — İçerik riski & doğruluk `[K]` `P0`

Özellikle YMYL: sağlık, finans, hukuk.

- [ ] **10.1 `[A]` Doğrulanmamış teknik değer uydurulmaz** — kapasite, motor gücü, ağırlık, saflık, CAS, sekans. Model numarasından kapasite türetme
- [ ] **10.2 `[G]` Kesin sonuç / tedavi / performans iddiası yok** — "en güçlü", "mucize", "hiçbir yan etkisi yok", "%100", "garantili", "kanıtlanmış fayda"
- [ ] **10.3 `[G]` Sahte onay / sertifika / yetki ifadesi yok** — "yetkili test merkezi", "FDA onaylı üretici" tipi doğrulanmamış ifadeler
- [ ] **10.4 `[G]` Gerçek kişi adı üzerinden sonuç anlatımı yok**
- [ ] **10.5 `[G]` Gerçek olmayan uzman/yazar üretilmiyor**
- [ ] **10.6 `[B]` Ticari içerik ile bilimsel iddia ayrı**, kaynak ve araştırma bağlamı net
- [ ] **10.7 `[B]` Boilerplate uyarı metni `data-nosnippet`** ile snippet'ten ayrılır
- [ ] **10.8 `[K][A]` +18 hassasiyetli sektörde** müstehcen kelime hariç tutulur

---

## FAZ 11 — Tazelik sinyalleri `P1`

- [ ] **11.1 `[A]` Görünür tarih** fiyat/güncel içerikli sayfalarda var ve bayat değil — rakipler bugünün tarihini basıyorsa bu bir fark
- [ ] **11.2 `[G]` Gelecek tarih basılmıyor** — sabah 09:20'de "11:50 güncellendi" gösterilmiyor
- [ ] **11.3 `[A]` Tek kaynak** — görünür tarih + `dateModified` + sitemap `lastmod` + HTTP `Last-Modified` aynı yerden beslenir, çelişmez
- [ ] **11.4 `[B]` `lastmod` sadece gerçek içerik değişikliğinde güncellenir.** Eski lastmod "Google taramıyor" demek **değildir** — kesin kural diye yazma
- [ ] **11.5 `[G]` Sırf taze görünsün diye tarih değiştirilmiyor** — bir sitedeki Ağustos çöküşünün kök nedeni buydu

---

## FAZ 12 — Performans & mobil `P1`

- [ ] **12.1 `[G]` Mobilde ana içerik masaüstüyle eşdeğer**, mobil navigasyon kritik linkleri kaybetmiyor
- [ ] **12.2 `[G]` Intrusive interstitial / araya giren popup yok**
- [ ] **12.3 `[G]` Viewport'ta `maximum-scale=1` yok** (mobilde zoom engelliyor)
- [ ] **12.4 `[B]` LCP < 2,5s · INP < 200ms · CLS < 0,1**
- [ ] **12.5 `[B]` Below-the-fold görsellerde `loading="lazy"`. Hero/LCP görsel lazy DEĞİL** — kritik above-the-fold görsel öncelikli yüklenir
- [ ] **12.5b `[B]` LCP adayı hero medyası kaynak HTML'de keşfedilebilir** — gerçek `<img>`/`<video>` olarak var mı, yoksa yalnız `data-*` + JS `background-image` ile mi kuruluyor? İkincisinde tarayıcının preload scanner'ı göremez. İlk slide `fetchpriority="high"`, sonrakiler lazy (bir vakada hero ilk slide 622 KB mp4, HTML'de hiç medya elementi yoktu)
- [ ] **12.6 `[B]` Aynı kütüphane iki kez yüklenmiyor** (ör. çift jQuery), kritik olmayan script `defer`
- [ ] **12.7 `[B]` Gereksiz üçüncü taraf script azaltıldı**
- [ ] **12.8 `[A]` Mobil iletişim** — "Hemen Ara" + WhatsApp butonu içeriği/CTA'yı kapatmıyor, çerez kutusu butonları engellemiyor
- [ ] **12.9 `[A]` `wa.me` numarası uluslararası formatta** (`wa.me/90...`, baştaki 0 ile çalışmaz), `tel:` boşluksuz E.164

---

## FAZ 13 — Yerel SEO `[K]` `P1`

Yerel hizmet işletmelerinde uygulanır (showroom, servis, tamir, bölgesel hizmet sağlayıcıları).

- [ ] **13.1 `[B]` NAP (isim/adres/telefon) site genelinde tutarlı** — footer, iletişim sayfası, schema aynı değeri veriyor
- [ ] **13.2 `[B]` Google Business Profile bilgileri güncel**, kategoriler doğru
- [ ] **13.3 `[B]` Schema adres/telefon = görünür site bilgisi**
- [ ] **13.4 `[B]` Gerçek lokasyonlar ayrı tanımlı**, hizmet bölgeleri gerçek
- [ ] **13.5 `[G]` Sahte lokasyon / doorway şehir sayfası üretilmiyor**
- [ ] **13.6 `[A]` local_pack kontrolü** — poz 1-3'te olup CTR %0-1 ise sorun SEO değil, local_pack sonucu yiyor demektir. Bu durumda kaldıraç GBP'dir, organik değil
- [ ] **13.7 `[A]` Yerel ticari kelimeler (şehir + ürün) ayrı segment olarak izlenir** — ortalama pozisyon bu segmentteki çöküşü gizler

---

## FAZ 14 — AI / GEO görünürlük `P2`

Ayrı bir "mucize SEO katmanı" değil, normal SEO'nun uzantısı.

- [ ] **14.1 `[B]` Önemli bilgi HTML/metin olarak erişilebilir** (Faz 2.5 ile aynı temel)
- [ ] **14.2 `[B]` Structured data görünür içerikle birebir aynı** — entity ve rich-result anlayışını destekler; **AI görünürlüğü için özel veya birincil bir schema sinyali olduğu varsayılmaz**. Google AI Overviews / AI Mode için ayrı SEO, ayrı schema veya `llms.txt` gerekmiyor
- [ ] **14.3 `[B]` Marka/entity bilgileri farklı kaynaklarda tutarlı**
- [ ] **14.4 `[A]` Önceliklendirme ticari niyet ağırlıklı** — bilgi kelimelerini AI kapıyor

---

## FAZ 15 — Off-page `P1`

> **Politika notu `[G]`:** Google, sıralama amaçlı link satın almayı link spam sayar; ücretli/sponsorlu yerleşimde `rel="sponsored"` veya `nofollow` bekler. Aşağıdaki maddeler bu riski ortadan kaldırmaz — **satın alınan yerleşimin parasal karşılığını aldık mı** sorusunun operasyonel kontrolüdür. Risk müşteriye açık söylenir.

- [ ] **15.1 `[A]` DA/PA ile karar verme** — gerçek organik trafik verisine bak. **Eşik: aylık < 1.000 = alma**
- [ ] **15.2 `[A]` `keywords_count` yüksek + `traffic_sum` düşük** = 2–3. sayfa sıralamaları, değersiz. `price_sum` yüksekse ticari kelimede sıralanıyor, iyi sinyal
- [ ] **15.3 `[A]` Ağ deseni ele** — yaş bilgisi boş + aynı DA/PA + aynı fiyat kümesi, aynı markanın çoklu TLD'si, `*habermerkezi.com` tipi kümeler
- [ ] **15.4 `[A]` Yayın öncesi `rel` kontrolü** — satıcının paralı yazısındaki `<a rel>` değerini kendi gözünle gör (bir yayıncıda 3/3 nofollow çıktı, bütçe boşa gitti)
- [ ] **15.5 `[A]` Aynı kaynağı birden fazla müşteride kullanma** — footprint
- [ ] **15.6 `[A]` Mevcut backlink'leri kontrol et**, mükerrer önerme
- [ ] **15.7 `[B]` GSC Links raporu:** yeni/kaybedilen önemli backlink, değerli link alan eski URL'lerin redirect hedefi kontrol edildi
- [ ] **15.8 `[G]` Disavow rutin değildir** — spam link var ama site üstteyse müşteriye bildirilir, o kadar. Sadece **manuel işlem var veya ciddi doğrulanmış risk varsa** kullanılır. Niş çapında yayılan spam ağı ne kaldıraç ne disavow konusudur
- [ ] **15.10 `[G][A]` Tema/footer'daki ajans kredi linki** `rel="nofollow noopener"`; ticari/sponsorlu ilişkiyse `sponsored`. **`rel="dofollow"` geçerli bir değer değildir**, yok sayılır ve link normal takip edilir. Aynı şablon çok sayıda müşteride çalıştığı için bu **tema deposunda tek seferde** düzeltilir — tek site bazında değil
- [ ] **15.9 `[A]` Disavow dosya formatı:** UTF-8/ASCII `.txt`, satır başına bir URL veya `domain:` girdisi. **Google `#` yorum satırına izin verir; bizim teslim formatımızda yorum satırı kullanılmaz**

---

## FAZ 16 — Uygulama güvenliği (kod & DB) `[A]` `P0`

Bu faz SEO değil, **deployment SOP'udur**. Denetim sonucu canlıya alınırken uygulanır.

- [ ] **16.1** DB + dosya yedeği alındı; kaynak/yedek satır sayısı birebir doğrulandı
- [ ] **16.2** Değişiklik öncesi crawl + mevcut sitemap + mevcut redirect listesi + GSC baseline kaydedildi
- [ ] **16.3** Kritik URL'lerin mevcut title/description/canonical/H1/schema değerleri export edildi
- [ ] **16.4** `DROP` / `TRUNCATE` / `WHERE`'siz `UPDATE` / kontrolsüz `DELETE` yok — hedefli UPDATE + idempotent INSERT
- [ ] **16.5** Büyük içerik alanlarında SHA-256 kontrolü — yedek alındıktan sonra CMS'te değişmiş kaydı eski veriyle ezme
- [ ] **16.6** MyISAM'da rollback'e güvenme — hata noktasında dur, hangi satırın uygulandığını hash ile tespit et, resume patch çalıştır
- [ ] **16.7** Değiştirilen her PHP dosyası `php -l`'den geçti
- [ ] **16.8** Kod ve DB için ayrı rollback paketi hazır
- [ ] **16.9** Ekrana basan (echo eden) çeviri/helper fonksiyonlarının çıktısı değişkene alınmaz — output-buffering sarmalayıcı kullan
- [ ] **16.10** Public kodun içine yorum satırı koyma
- [ ] **16.11** Şablon sayfalarındaki dönüşüm (CTA) blokları kaldırılmaz, buton hedefleri değiştirilmez
- [ ] **16.12** Proje yönetim aracında (task tracker) yalnızca okuma yapılır; yazma işlemi onayla

---

## FAZ 17 — Kabul testi (canlı) `P0`

İş, "yapıldı" demekle değil **canlıdan teyitle** biter. DB çıktısına, rapora, başka bir AI'ın "canlıda gördüm"üne güvenme.

**Site geneli**
- [ ] 17.1 Tüm sitemap URL'leri yeniden tarandı, beklenen indeks URL'leri 200
- [ ] 17.2 Beklenmeyen 3xx/4xx/5xx yok
- [ ] 17.3 Tek canonical + canonical hedefi 200 + duplicate yok
- [ ] 17.4 Benzersiz description / ana başlık sayısı **aktif sayfa sayısıyla eşitlendi** (ör. 108/108)
- [ ] 17.5 Hatalı göreli link 0, kırık iç link 0
- [ ] 17.6 Sitemap'te redirect / 404 / noindex 0
- [ ] 17.7 Schema parse hatası 0, eski/riskli boilerplate 0, yasaklı ifade taraması 0 hit

**Redirect**
- [ ] 17.8 Her yeni 301 tek tek test edildi: doğru status + doğru `Location` + hedef 200 + chain yok + loop yok

**Çok dilli `[K]`**
- [ ] 17.9 Hreflang karşılıklı, dil switcher doğru, canonical/hreflang/sitemap aynı standartta

**Görsel / mobil**
- [ ] 17.10 Header, footer, menü, form, WhatsApp/telefon, galeri, slider, çerez popup, CTA — mobil dahil

- [ ] **17.11 Sonuç `X PASS / Y FAIL` olarak kaydedildi**
- [ ] **17.12 Rapor üç kovaya ayrıldı:** ✅ uygulandı / 🟡 yarım (hangi yarısı) / ❌ yapılmadı. **"Uygulanmadı" cümlesi ancak sitemap + HTTP + `meta robots` + kelime sayısı dördü birden eskiyi gösterdiğinde yazılır**

---

## FAZ 18 — Baseline & takip `P0`

### 18.1 Deploy günü baseline (kaydedilmeden iş kapanmaz)

| Alan | Değer |
|---|---|
| Deploy tarihi | |
| Aktif indekslenebilir URL | |
| Sitemap URL sayısı | |
| 200 / 301 / 404 sayıları | |
| Marka dışı tıklama (28g) | |
| Marka dışı gösterim (28g) | |
| Top query'ler | |
| Top landing page'ler | |
| Ortalama pozisyon | |
| Hedef kelime URL sahiplik oranı | |
| CWV durumu | |
| Kritik schema hatası | |

> **Toplam tıklama ana başarı kriteri değildir** — marka sorgusu ağırlıklı sitelerde yanıltır. KPI = marka dışı tık/gösterim, sorgu çeşitliliği, hedef kelime pozisyonu, URL sahiplik oranı.
> Aynı gün birden fazla değişiklik canlıya alındıysa **cohort kur**: Grup A (title+gövde) / Grup B (sadece title) / Grup C (kontrol).

### 18.2 Takip ritmi

| Zaman | Yapılacak |
|---|---|
| **0. gün** | Kabul testi + baseline |
| **7. gün** | Crawl/index anomalisi, yeni 404, 5xx |
| **28. gün** | GSC eşit pencere karşılaştırması (aynı ülke filtresi) |
| **56. gün** | İkinci performans karşılaştırması |
| **Haftalık** | Trafik anomalisi, 5xx, Manual Action/Security, ranking kaybı, yeni 404 |
| **Aylık** | Page Indexing, CWV, sitemap, kanibalizasyon, keyword ownership, backlink profili |
| **3–6 aylık** | İçerik konsolidasyonu, thin content, eski bilgi, rakip gap, mimari revizyonu |

---

## ASLA

1. Uydurma schema (rating / review / kişi / fiyat)
2. `now()` ile üretilen tarih — sahte tazelik
3. Trafik verisi görülmeden 301/silme listesi
4. Toplu 301'i tek hedefe (anasayfa) yığma
5. Sitemap'te 301'li URL
6. Canonical + 301 + sitemap + hreflang çelişkisi
7. Tek taramaya dayanan "sayfa silinmiş" hükmü
8. Kısmi ay ↔ tam ay kıyası
9. Düşüşü hemen "sorun" etiketleme — önce dönemsellik, baz etkisi, AI Overviews, başka firmanın marka sorgusunun sönmesi
10. Onaylı listeye kendi inisiyatifinle satır ekleme/çıkarma
11. `[B]` veya `[A]` bir maddeyi müşteri raporunda "Google kuralı" diye sunma

---

## Site tipine göre çalışma sırası

| Durum | Fazlar |
|---|---|
| **Yeni müşteri, ilk denetim** | Tamamı |
| **Relaunch / tasarım değişimi sonrası** | 0 → 1 → 2 → 3 → 4 → 5 → 17 → 18 (301 haritası kritik) |
| **"Düştü" şikayeti** | 0 → 1 → 9 → 4 → 11 → 6 (+13 yerelse) |
| **Aylık rutin** | 0 → 1.1 · 1.2 · 1.4 → 3.3 · 3.7 → 4.8 → 9.5 → 11 |
| **Haftalık** | 0 → 1.1 · 1.2 → trafik/5xx/404 anomalisi |
| **Çok dilli** | + Faz 8 |
| **YMYL (sağlık, finans)** | + Faz 10 tamamı |
| **Yerel hizmet işletmesi** | + Faz 13 |
| **Canlıya alma** | + Faz 16 → 17 → 18 |

---

**Dış kaynaklar:** [Search Essentials](https://developers.google.com/search/docs/essentials) · [Crawling & Indexing](https://developers.google.com/search/docs/crawling-indexing) · [Canonical](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls) · [Redirects](https://developers.google.com/search/docs/crawling-indexing/301-redirects) · [Sitemap](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap) · [Hreflang](https://developers.google.com/search/docs/specialty/international/localized-versions) · [Product structured data](https://developers.google.com/search/docs/appearance/structured-data/product-snippet) · [Core Web Vitals](https://developers.google.com/search/docs/appearance/core-web-vitals) · [AI features](https://developers.google.com/search/docs/appearance/ai-features) · [Spam policies](https://developers.google.com/search/docs/essentials/spam-policies) · [Disavow](https://support.google.com/webmasters/answer/2648487) · [Recrawl](https://developers.google.com/search/docs/crawling-indexing/ask-google-to-recrawl)
