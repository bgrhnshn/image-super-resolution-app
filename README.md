# Image Super-Resolution Application
Model adı: ‘trainedVDSR-Epoch-100-ScaleFactors-234.mat’
Görüntüler, dizindeki MAT dosyaları olarak diskte saklanır: ‘upsampledDirName’. Ağ yanıtlarını temsil eden hesaplanan artık görüntüler, dizindeki MAT dosyaları olarak diskte saklanır: ‘residualDirName’. 
MAT dosyaları double, ağı eğitirken daha fazla hassasiyet için veri türü olarak saklanır. Ortaya çıkan veri deposu, dsTrainçağın her yinelemesinde ağa mini veri grupları sağlar. 



![Resim2](https://user-images.githubusercontent.com/82895857/125135849-5a904f80-e112-11eb-8904-cdd285ea46b4.png)





Bu uygulama, VDSR ağını, Deep Learning Toolbox ™ 'dan 41 ayrı katman kullanarak tanımlar. VDSR'nin 20 evrişimli katmanı vardır, bu nedenle alıcı alan ve görüntü yama boyutu 41'e 41'dir. Görüntü girdi katmanı, tek kanallı görüntüleri kabul eder çünkü VDSR yalnızca parlaklık kanalı kullanılarak eğitilir.
Görüntü giriş katmanını 3'e 3 boyutunda 64 filtre içeren 2 boyutlu bir evrişimli katman izler. Her bir evrişimli katmanı, ağda doğrusal olmama durumu oluşturan bir ReLU katmanı izler. Orta katmanlar, 18 dönüşümlü evrişimli ve rektifiye edilmiş doğrusal birim katman içerir. Her evrişimli katman, 3'e 3'e 64 boyutunda 64 filtre içerir; burada bir filtre, 64 kanal boyunca 3'e 3 uzaysal bir bölgede çalışır.
Test veri seti, testImagesImage Processing Toolbox ™ içinde gönderilen 21 bozulmamış görüntü içerir.









# Veriseti
'http://www-i6.informatik.rwth-aachen.de/imageclef/resources/iaprtc12.tgz'

20.000 hareketsiz doğal görüntüden [ 2 ] oluşan IAPR TC-12 Benchmark'ı indirdik . Veri seti, insanların, hayvanların, şehirlerin ve daha fazlasının fotoğraflarını içeriyor. Veri dosyasının boyutu ~ 1.8 GB'dir. Eğitim veri setini indirmek istemiyorsanız load('trainedVDSR-Epoch-100-ScaleFactors-234.mat');, komut satırına yazarak önceden eğitilmiş VDSR ağını yükleyebilirsiniz . 

# Eğitim
Momentum (SGDM) optimizasyonu ile stokastik gradyan inişini kullanarak ağı eğitiyoruz. (Derin Öğrenme Araç Kutusu) işlevini kullanarak SGDM için hiperparametre ayarlarını belirtiyoruz. Öğrenme oranı başlangıçta her 10 dönemde bir 10 faktör azaltılır. 100 devir için eğitim(trainingOptions0.1).

Derin bir ağı eğitmek zaman alıcıdır. Yüksek bir öğrenme oranı belirleyerek eğitimi hızlandırıyoruz. Ancak bu, ağın gradyanlarının patlamasına veya kontrolsüz bir şekilde büyümesine neden olarak ağın başarılı bir şekilde eğitilmesini engelleyebilir. Degradeleri anlamlı bir aralıkta tutmak için, 'GradientThreshold'olarak belirterek degrade kırpmayı etkinleştiriyoruz ve degradelerin L2 normunu kullanmayı 0.01 olarak belirtiyoruz ('GradientThresholdMethod').
	
    maxEpochs = 100;
    epochIntervals = 1;
    initLearningRate = 0.1;
    learningRateFactor = 0.1;
    l2reg = 0.0001;
    miniBatchSize = 64;
    options = trainingOptions('sgdm', ...
        'Momentum',0.9, ...
        'InitialLearnRate',initLearningRate, ...
        'LearnRateSchedule','piecewise', ...
        'LearnRateDropPeriod',10, ...
        'LearnRateDropFactor',learningRateFactor, ...
        'L2Regularization',l2reg, ...
        'MaxEpochs',maxEpochs, ...
        'MiniBatchSize',miniBatchSize, ...
        'GradientThresholdMethod','l2norm', ...
        'GradientThreshold',0.01, ...
        'Plots','training-progress', ...
        'Verbose',false);

Eğitim seçeneklerini ve rastgele yama çıkarma veri deposunu yapılandırdıktan sonra, (Derin Öğrenme Araç Kutusu) işlevini kullanarak VDSR ağını eğitiyoruz. Ağı eğitmek için aşağıdaki koddaki parametreyi ayarlıyoruz. 

    doTraining = false;
    if doTraining
        modelDateTime = datestr(now,'dd-mmm-yyyy-HH-MM-SS');
        net = trainNetwork(dsTrain,layers,options);
        save(['trainedVDSR-' modelDateTime '-Epoch-' num2str(maxEpochs*epochIntervals) '-ScaleFactors-' num2str(234) '.mat'],'net','options');
    else
        load('trainedVDSR-Epoch-100-ScaleFactors-234.mat');
    end

Derin öğrenme kullanarak süper çözünürlük sonuçlarını bikübik enterpolasyon gibi geleneksel görüntü işleme tekniklerini kullanarak sonuçla karşılaştırmak için kullanılacak düşük çözünürlüklü bir görüntü oluşturuyoruz. Test veri seti, testImagesImage Processing Toolbox ™ içinde gönderilen 21 bozulmamış görüntüyü içerir. Görüntüleri imageDatastore dan yüklüyoruz.
Süper çözünürlük için referans görüntü olarak kullanmak üzere görüntülerden birini seçiyoruz. Referans görüntü olarak yüksek çözünürlüklü görüntümüzü kullanıyoruz.
imresize0,25 ölçekleme faktörünü kullanarak yüksek çözünürlüklü referans görüntünün düşük çözünürlüklü bir versiyonunu oluşturuyoruz. Görüntünün yüksek frekanslı bileşenleri ölçek küçültme sırasında kaybolur.
VDSR yalnızca bir görüntünün parlaklık kanalı kullanılarak eğitiliyor, çünkü insan algısı parlaklıktaki değişikliklere renkteki değişikliklerden daha duyarlıdır.
Fonksiyonu kullanarak düşük çözünürlüklü görüntüyü RGB renk uzayından parlaklık ( Iy) ve renklilik ( Icbve Icr) kanallarına dönüştürün rgb2ycbcr.
Yüksek çözünürlüklü VDSR parlaklık bileşenini elde etmek için görüntüyü yükseltilmiş parlaklık bileşenine ekliyoruz.
Yüksek çözünürlüklü VDSR parlaklık bileşenini ölçeklendirilmiş renk bileşenleriyle birleştiriyoruz. ycbcr2rgbFonksiyonu kullanarak görüntüyü RGB renk uzayına dönüştürüyoruz. Sonuç, VDSR kullanan nihai yüksek çözünürlüklü renkli görüntüdür.
# Tablo ve Grafik (parametre optimizasyonu):

      InputImage      ResponseImage 
    ______________    ______________
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}
    {41×41 double}    {41×41 double}


    Results for Scale factor 2:			Results for Scale factor 4:
    Average PSNR for Bicubic = 31.809683		Average PSNR for Bicubic = 27.010839
    Average PSNR for VDSR = 31.921784		Average PSNR for VDSR = 27.837260
    Average SSIM for Bicubic = 0.938194		Average SSIM for Bicubic = 0.861604
    Average SSIM for VDSR = 0.949404			Average SSIM for VDSR = 0.877132

    Results for Scale factor 3:
    Average PSNR for Bicubic = 28.170441
    Average PSNR for VDSR = 28.563952
    Average SSIM for Bicubic = 0.884381
    Average SSIM for VDSR = 0.895830


# Yorumlar (En başarılı parametre değerleri, nedenleri vs.):
VDSR, her ölçek faktörü için iki kübik enterpolasyondan daha iyi metrik puanlara sahiptir.

![Resim1](https://user-images.githubusercontent.com/82895857/125132545-223a4280-e10d-11eb-8421-79ea1d5363d5.png)




