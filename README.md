# Park Yeri Doluluk Tespiti (OpenCV)

Bir otopark videosundaki park yerlerinin **boş / dolu** durumunu klasik görüntü
işleme teknikleriyle gerçek zamanlı tespit eden bir Python uygulaması. Derin
öğrenme kullanmadan, yalnızca OpenCV ile her park yerindeki piksel yoğunluğuna
bakarak çalışır; bu sayede hafif ve hızlıdır.

Uygulama iki adımdan oluşur:

1. **Koordinat seçimi** (`meltemkoordinat.py`) — video üzerinden park yerlerini
   fareyle işaretleyip koordinatlarını çıkarırsın.
2. **Analiz** (`meltemmain.py`) — bu koordinatları kullanarak videoyu kare kare
   işler ve her park yerinin doluluk durumunu ekranda gösterir.

---

## Özellikler

- Videodaki her park yerini **yeşil (boş)** veya **kırmızı (dolu)** çerçeveyle
  işaretleme.
- Ekranın üst köşesinde anlık **"Empty Spaces: X/Toplam"** özeti.
- Fareyle tıklayarak park yeri koordinatlarını belirleme aracı.
- Kareyi tek tuşla `output.jpg` olarak kaydetme.
- Harici model / eğitim gerektirmez; tek bağımlılığı OpenCV ve NumPy.

---

## Nasıl Çalışır?

Her kare şu görüntü işleme hattından geçirilir (`analyze_parking_space`):

1. **Gri tonlama** — renk bilgisi doluluk tespiti için gereksiz, atılır.
2. **Gaussian Blur** — küçük gürültüler yumuşatılır.
3. **Adaptive Threshold** (`ADAPTIVE_THRESH_GAUSSIAN_C`, ters ikili) — değişen
   ışık koşullarında bile araç kenarları/dokusu beyaz piksele dönüşür.
4. **Median Blur** — kalan tuz-biber gürültüsü temizlenir.

Ardından her park yeri dikdörtgeni (ROI) için beyaz piksel sayısı
(`countNonZero`) hesaplanır. Bu sayı bir eşiğin (`THRESHOLD_LIMIT`) altındaysa
alan **boş**, üstündeyse **dolu** kabul edilir. Mantık şudur: boş bir park yeri
düz asfalttır (az beyaz piksel), araç olan yerde ise kenar ve doku yoğunluğu
yüksektir (çok beyaz piksel).

---

## Gereksinimler

- Python 3.8+
- OpenCV (`opencv-python`)
- NumPy
- Analiz edilecek bir otopark videosu: `carPark.mp4`

```bash
pip install opencv-python numpy
```

> Not: Kod, video dosyasını `carPark.mp4` adıyla ve script ile aynı klasörde
> arar. Farklı bir dosya kullanacaksan `VIDEO_SOURCE` / `VIDEO_FILE`
> değişkenlerini güncelle.

---

## Kullanım

### 1. Adım — Park yeri koordinatlarını belirle

```bash
python meltemkoordinat.py
```

Açılan pencerede boş park yerlerinin **sol üst köşesine sol tıkla**; her tıklama
o noktaya sabit boyutlu yeşil bir çerçeve çizer ve koordinatı konsola yazar.
İşin bitince `q` tuşuna bas — işaretlenen tüm koordinatlar konsola dökülür.

Bu koordinat listesini kopyalayıp `meltemmain.py` içindeki `PARKING_SPOTS`
listesine yapıştır. (Depoda örnek olarak 69 park yeri koordinatı hazır gelir.)

### 2. Adım — Doluluğu analiz et

```bash
python meltemmain.py
```

Video oynatılır ve her park yeri anlık olarak boş/dolu diye renklendirilir; üstte
toplam boş yer sayısı gösterilir.

### Kontroller (analiz ekranı)

| Tuş | İşlev |
|-----|-------|
| `q` | Çıkış |
| `s` | O anki kareyi `output.jpg` olarak kaydet |

---

## Yapılandırma

Script başlarındaki sabitlerle davranış ayarlanır:

| Değişken | Dosya | Açıklama |
|----------|-------|----------|
| `PARKING_SPOT_SIZE` / `RECT_DIMENSIONS` | koordinat / main | Park yeri kutusunun (genişlik, yükseklik) boyutu (~107×48 px) |
| `THRESHOLD_LIMIT` | main | Boş/dolu ayrım eşiği (varsayılan `670`). Yanlış "dolu" tespiti olursa artır, yanlış "boş" olursa azalt |
| `VIDEO_SOURCE` / `VIDEO_FILE` | koordinat / main | Video dosyasının yolu |
| `PARKING_SPOTS` | main | Park yerlerinin (x, y) koordinat listesi |

> İpucu: `THRESHOLD_LIMIT`, kutu boyutuna ve videonun çözünürlüğüne göre
> ayarlanmalıdır. Kendi videonda en iyi sonucu bulmak için bu değeri birkaç kez
> deneyerek kalibre et.

---

## Dosya Yapısı

```
.
├── meltemkoordinat.py   
├── meltemmain.py        
└── carPark.mp4          
```

---

## Bilinen Sınırlamalar / Notlar

- **Sabit koordinatlar:** Kamera açısı sabit olmalıdır; açı değişirse
  koordinatlar yeniden belirlenmelidir.
- **Eşik hassasiyeti:** Yöntem tamamen piksel yoğunluğuna dayandığından; gölge,
  ıslak zemin veya farklı ışık koşulları yanlış tespide yol açabilir.
- **Kutu boyutu tutarsızlığı:** `meltemkoordinat.py` (107×48) ile
  `meltemmain.py` (109×49) kutu boyutları küçük farklıdır; birebir tutarlılık
  için ikisini eşitlemen önerilir.
- **Koordinat aracında sağ tık ile silme:** `meltemkoordinat.py` içindeki sağ
  tıklayarak işareti kaldırma bölümü şu an eksik/hatalıdır (tanımsız değişkenler
  ve girinti hatası içerir) ve çalışmaz. Şimdilik yalnızca sol tıkla ekleme
  aktiftir.

---

## Olası Geliştirmeler

- Seçilen koordinatların bir dosyaya (`.json` / `.pkl`) kaydedilip
  yüklenmesi — her açılışta yeniden işaretleme gerekmez.
- Eşik değerinin arayüzden bir kaydırma çubuğu (trackbar) ile canlı ayarlanması.
- Klasik yöntem yerine/yardımıyla bir nesne tespiti modeli (ör. YOLO) ile
  doğruluğun artırılması.
