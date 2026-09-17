# SEO Master Checklist

Teknik SEO denetimi için 18 fazlık uygulama listesi. Gerçek denetim vakalarından çıkan kurallarla yazıldı; her madde ya bir Google zorunluluğu ya da bedeli ödenmiş bir operasyon dersidir.

**Ana dosya:** [SEO-CHECKLIST.md](SEO-CHECKLIST.md)

## Neden bu liste

Denetimlerde asıl kayıp "neye bakacağımızı bilmemek" değil, **yanlış ölçmek**. Liste bu yüzden Faz 0 (ölçüm protokolü) ile başlar: thread sayısı, redirect takibi, JSON-LD parse, GSC pencere seçimi. Bu faz atlanırsa geri kalan 17 faz yanlış bulgu üretir.

## Etiket sistemi

| Etiket | Anlamı |
|---|---|
| `[G]` | Google zorunluluğu — müşteri raporunda "Google kuralı" diye yazılabilecek tek kategori |
| `[B]` | Best practice — "önerilir" diye yazılır, "ihlal" diye değil |
| `[A]` | Ajans standardı — kendi çalışma kuralımız, Google'a mal edilmez |
| `[K]` | Koşullu — sadece o tip sitede uygulanır |

Öncelik: `P0` trafik/indeks riski · `P1` büyüme engeli · `P2` optimizasyon

## Kullanım

1. Site tipini belirle → checklist sonundaki **"Site tipine göre çalışma sırası"** tablosundan faz sırasını seç.
2. Dosyanın kopyasını `denetim-<site>-<YYYYAAGG>.md` olarak aç.
3. Her maddeyi `PASS / FAIL / N/A` işaretle. `N/A` gerekçesiz bırakılmaz.
4. Faz 17 (kabul testi) canlıdan doğrulanmadan iş kapanmaz.
5. Faz 18 baseline tablosu doldurulmadan iş kapanmaz.

---

## Kurulum

Liste araç bağımsızdır, ama Faz 0'daki ölçüm kuralları aşağıdaki ayarlarla uygulanır.

### Gereksinimler

- Python 3.10+
- Google Search Console erişimi (denetlenen property'de en az `Restricted` yetki)
- İsteğe bağlı: Google Analytics 4 (GSC 2–3 gün gecikmeli olduğu için "bugün" verisi GA4'ten alınır)

```bash
pip install requests beautifulsoup4 lxml openpyxl google-api-python-client google-auth-oauthlib
```

### Crawler ayarları (Faz 0.1 – 0.4)

Bu değerler keyfi değil; farklı ayarlarla yanlış bulgu üretildiği doğrulandı.

```python
CRAWL = {
    "threads": 3,                 # 8-12 thread'de sunucu 429 yerine 404 döner
    "timeout": 20,
    "retry_404": 3,               # her 404 en az 2-3 kez tekrar doğrulanır
    "retry_delay": 0.35,
    "allow_redirects": False,     # durum kodu ham URL'den okunur
    "verify_ssl": True,
    "headers": {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 "
            "(KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
        ),
        "Accept-Language": "tr-TR,tr;q=0.9,en;q=0.8",
    },
}

# Kelime sayımı gövde-only: nav/header/footer/head/script/style silindikten sonra sayılır
STRIP_TAGS = ["nav", "header", "footer", "head", "script", "style", "noscript"]
```

JSON-LD **parse edilir**, regex ile okunmaz:

```python
import json
data = json.loads(script_tag.string)
nodes = data.get("@graph", [data])
for node in nodes:
    types = node.get("@type", "")
    types = types if isinstance(types, list) else [types]   # @type dizi de olabilir
```

### Google Search Console API kurulumu

1. Google Cloud Console → yeni proje → **Search Console API**'yi etkinleştir.
2. **OAuth consent screen** → External → test kullanıcısı olarak kendi hesabını ekle.
3. **Credentials** → OAuth client ID → *Desktop app* → `credentials.json` indir.
4. `credentials.json`'ı repo dışında, `.gitignore`'lu bir klasörde tut. İlk çalıştırmada tarayıcı açılır, `token.json` üretilir.

Scope:

```
https://www.googleapis.com/auth/webmasters.readonly
```

GA4 de kullanılacaksa aynı OAuth istemcisine şu scope eklenir:

```
https://www.googleapis.com/auth/analytics.readonly
```

### Ölçüm penceresi ayarları (Faz 0.6 – 0.8)

```python
GSC = {
    "country": "tur",          # ülke filtresi olmadan kıyas yapılmaz
    "window_days": 28,         # iki pencere de eşit uzunlukta
    "lag_days": 3,             # GSC 2-3 gün gecikir, son 3 gün sayılmaz
    "site_total_dimensions": [],   # top_pages toplamı uzun kuyruğu kaçırır
}
```

- Aylık kıyas: iki **tam** ay veya 28 günlük simetrik pencere. Kısmi ay ile tam ay kıyaslanmaz.
- Haftalık kıyas: son 7 gün ↔ önceki 7 gün.
- Yıllık kıyas (YoY) tercih edilir; QoQ baz etkisi ve sezonluk dalgalanma yüzünden yanıltabilir.

### Yerel gizlilik

Denetim çıktıları müşteri trafik verisi ve teknik açık içerir. Repo'ya yalnızca **listenin kendisi** girer; denetim çıktıları, `credentials.json`, `token.json` ve xlsx raporları girmez.

```gitignore
credentials.json
token.json
*.xlsx
ciktilar/
denetim-*.md
.env
```

---

## Katkı

Yeni bir madde eklenirken **hangi vakadan çıktığı** ve **doğru etiketi** (`[G]` / `[B]` / `[A]`) yazılır. Google şartı olmayan bir kural `[G]` etiketlenmez — bu listenin tek sert kuralı budur.

## Lisans

MIT
