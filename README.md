# Oda doluluğu, CO₂ kestirimi ve kümeleme

Bu çalışmada aynı sensör verileriyle sınıflandırma, regresyon ve kümeleme yaptım. Modelleri baseline sonuçlarıyla karşılaştırdım.

Ana dosya: **[notebooks/oda_dolulugu.ipynb](notebooks/oda_dolulugu.ipynb)**.
Notebook kodları, hesaplanan sonuçları ve yorumlarımı içeriyor.

[Google Colab'da aç](https://colab.research.google.com/drive/1OMV1xoI8iV2qQEW4_22TxXNrTeLSZzuY?usp=sharing)

## Problemler

| Görev         | Hedef                                | Modeller                                                       |
| ------------- | ------------------------------------ | -------------------------------------------------------------- |
| Sınıflandırma | Odada 0, 1, 2 veya 3 kişi            | En sık sınıf baseline, LogisticRegression, MLPClassifier (ANN) |
| Regresyon     | Aynı andaki CO₂ konsantrasyonu (ppm) | Ortalama baseline, Ridge, RandomForestRegressor                |
| Kümeleme      | Hedef kullanılmaz                    | KMeans, k=2..8; silhouette ve dirsek                           |

Ortak 14 girdi: 4 sıcaklık, 4 ışık, 4 ses ölçümü ve 2 ikili kategorik PIR hareket durumu.
Her üç görevde kişi sayısı, CO₂ ve CO₂ eğimi girdilerden çıkarılır. Tarih/saat yalnız bölme içindir.

## Çalıştırma

Çalışma Python 3.14 ile doğrulandı. Kütüphaneler `requirements.txt` içinde listelenmiştir.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m ipykernel install --user --name ml-assignment --display-name "ML Assignment (hazır ortam)"
python -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=900 notebooks/oda_dolulugu.ipynb
```

Notebook'u bir Jupyter uyumlu editörde açıp **ML Assignment (hazır ortam)** kernel'ini seçerek tüm hücreleri çalıştırmak da mümkündür.
Google Colab kullanıyorsanız notebook'u açın ve `data/raw/Occupancy_Estimation.csv` dosyasını
Colab'ın Dosyalar bölümüne yükleyin. Gerekirse `requirements.txt` içindeki kütüphaneleri kurun.
Colab üzerinde ayrıca doğrulama yapılmadı; farklı sürümlerde sonuçlar küçük farklar gösterebilir.

`notebooks/oda_dolulugu.html` hesaplanmış sonuçların önizlemesidir.
Tablolar `outputs/`, grafikler `outputs/figures/` klasörüne kaydedilir.

## Deney tasarımı ve sınırlar

- 2 saatlik gruplar: aynı zaman bloğu farklı bölmelere giremez.
- Tohum 42; 5 katlı StratifiedGroupKFold'un ilk katı test.
- Teste 15 dakika veya daha yakın eğitim kayıtları tampon olarak ayrılır.
- Eğitimde 3 katlı grup CV; her katta aynı tampon uygulanır.
- Sayısal ölçekleme Pipeline içinde, yalnız ilgili eğitim verisinden öğrenilir. Eksik değer yoktur; PIR alanları zaten 0/1 kodludur.
- Model/ayar seçimi sınıflandırmada macro-F1, regresyonda RMSE üzerinden, test görülmeden yapılır.
- Lojistik regresyon C=10 ve Ridge alpha=10, önceki eğitim CV denemelerinden korunmuştur. ANN'nin iki mimarisi aynı bölmelerde karşılaştırılır.
- ANN'nin rastgele iç doğrulama ayıran early_stopping seçeneği kapalıdır. İki mimari aynı blok CV ile karşılaştırılır.
- KMeans ölçekleyicisi ve merkezleri yalnız eğitim X ile öğrenilir; k seçiminde test veya hedef kullanılmaz.

Bu düzen aynı odadaki gözlenmemiş zaman bloklarını değerlendirir; ileriye dönük tahmin testi değildir.
Tampon süre uzun dönemli bağımlılığı tamamen gidermez. Tek oda, az sayıda tarih ve dengesiz sınıflar genellemeyi sınırlar.
“1 kişi” yalnız ilk iki günde vardır. Başarı yeni bina/oda için doğrulanmış sayılmaz.
CO₂ regresyonu bir sanal sensör çalışmasıdır; gelecekteki hava kalitesi tahmini değildir.

## Veri ve atıf

Singh, A. & Chaudhari, S. (2018). *Room Occupancy Estimation*. UCI Machine Learning Repository.
DOI: [10.24432/C5P605](https://doi.org/10.24432/C5P605).

- [Veri açıklaması](https://archive.ics.uci.edu/dataset/864/room+occupancy+estimation)
- [Ham arşiv](https://archive.ics.uci.edu/static/public/864/room+occupancy+estimation.zip)
- Lisans: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). CSV orijinal haliyle tutulur; model girdileri kodda seçilir.
- 10.129 kayıt, 19 ham sütun; 7 farklı takvim tarihi. Kaynak açıklamasındaki “4 gün” ifadesiyle ham tarih sayısı uyuşmaz; ham dosya esas alınmıştır.
- Gözlem sıklığı nedeniyle 10.129 satır, 10.129 bağımsız deney anlamına gelmez.

Yöntem: [scikit-learn Pipeline ve sızıntı](https://scikit-learn.org/stable/common_pitfalls.html),
[gruplu CV](https://scikit-learn.org/stable/modules/cross_validation.html).
