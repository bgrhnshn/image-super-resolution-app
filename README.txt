1) "main.m" uygulamanın kodlarını içermektedir.

2) "trainedVDSR-Epoch-100-ScaleFactors-234.mat" eğitilmiş modeldir.

3) "iaprtc12" klasörü tüm verisetini içermektedir.

4) "iaprtc12" klasörü uygulamadaki kod tarafından indirilmektedir. Klasör 1.8gb civarındadır. Bu yüzden indirilmiş veri setiyle birlikte sisteme yükleyemedik. İndirmeyle ilgili herhangi bir sorun olması durumunda;
	'http://www-i6.informatik.rwth-aachen.de/imageclef/resources/iaprtc12.tgz'
bağlantısından veri seti indirilir. 
	imagesDir = tempdir;
	url = 'http://www-i6.informatik.rwth-aachen.de/imageclef/resources/iaprtc12.tgz';
	downloadIAPRTC12Data(url,imagesDir);

(2,3 ve 4. satırlardaki) kod parçası aşağıdaki şekilde indirilen verisetinin bulunduğu dizini içerecek şekilde değiştirilerek sorun giderilebilir.
	"imagesDir='C:\Users\<User>\Desktop\<file_directory>';"