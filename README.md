# WaterPuddle
Inventuna Games Test Case Raporu

# WaterPuddle

Inventuna Games Test Case Raporu

İçerik
- Test Case özeti
- Sorunu Anlamak
- “Töze İnmek”
- Görsel Efektin Kapsamı ve Özellik Listesi
- Literatür Taraması
- Örnek Yaklaşımlar
- Kullanılabilecek Yöntemler
- Metodoloji
- Teknolojiler
- Matematik Modelleri
- Gelişim
- Aşamalar

Sonuçlar ve Değerlendirme
- Sorunlar ve Çözümleri
- Optimizasyon Yöntemleri
- Çıkarımlar

---

## Test Case Özeti
Suyun üzerine atılan bir cisim, ağırlığına ve hızına göre su üzerinde dalgalanmalar oluşturacak. Bu etkileşimler ve suyun görüntüsü gerçekçi olacak. Challenge: Unreal HLSL kullanılması gerekiyor.

## Sorunu Anlamak
İstenilen simülatif yapıyı anlamak için istenilenleri parçalayalım. Gerçekçi görünen bir su için, burada spesifik girdilere ihtiyacımız var: Ortam, derinlik ve diğer koşullar. Bu inputları varsayılan şekilde tutarak, daha genel bir yapı kurguladık. Burada önemli olan şey suyun davranışı olduğu için burada simülasyonu sağlayabilecek matematik modellerinin arayışına girdim. Bu bağlamda yapının gerekli özellikleri ortaya çıkmış oldu.

### Su Yüzey Görüntüsü için:
- Depth Fade
- Fresnel Opacity
- Water Puddle Equation
- Water Splash FX

## Literatür Taraması
Örnek yaklaşımları incelediğimde birçok kaynak farklı matematik modeli uygulamıştı. Shader Toy'daki kaynaklar “water rain puddle” şeklinde uygulamalar yapmışlardı. Game engine uyarlamaları da mevcut; burada simülasyon yerine onu “fake” ederek performans artışı elde edecek yöntemler kullanılmış daha çok. Burada oyun motoru kullanmak fizik etkileşimli görsel efektler için ideal olacaktır. Unreal Engine Content Example'da Render Target örnek haritasında, dalgalanmaların render targetlar ile nasıl kullanıldığını gösteren bir örnek vardı; o da diğer bir alternatif yöntemdi.

## Metodoloji
Burada görsel efektler için Custom Shader kullanabilmem gerektiği için oyun motoru kullanmaya karar verdim, UE 4.27. Unreal Engine içerisinde Puddle simulation denklemlerini Custom HLSL node ile çalıştırabilirim. Burada “Challenge” kısım birden fazla objenin etkileşime geçmesi. Puddle simulation'ları bir material içerisinde çalıştırmak problem değil. Burada simülasyonun başladığı start position girdisini verdiğim anda zaman bağlı olarak simülasyon başlıyor fakat birden fazla simülasyon çalıştırmak için ya start position array'i vermem gerekir ki şu anda çok imkanlı görünmüyor fakat çeşitli work around'lar mevcut. Render Target kullanabiliriz.

Araştırmanın önemli bir kısmını matematik modellerine ve bunların uygulanabilirliğine ayırdım.

### Height Field Water Models
- Su yüzeyini bir yükseklik haritası olarak temsil eder.
- Gerçek zamanlı uygulamalar için daha uygundur ve daha az hesaplama maliyetlidir.

### Wave Equation
- Dalga hareketlerini simüle etmek için basit bir denklem.
- Hızlı ve kolay uygulanabilir.
- Temel dalga hareketleri için uygun olabilir, ancak su dinamikleri ile ilgili diğer etkileşimler ve detaylar için yeterli olmayabilir.

### “The Puddle Equation”
- Bulduğum en ger

çekçi model buydu.
- Yüksek performans maliyeti olabileceği için üzerine gitmedim. [Puddle Equation](https://people.rit.edu/nsbsma/home/Puddles.html)

Matematik modellerinin araştırılmasında yapay zekadan destek de aldım.

### Height Field Water Modeli tercih ettim, shader kodu (Temel hali):
```hlsl
// float A = 1.0;  // Dalga genliği
// float k = 1.0;  // Dalga sayısı
// float omega = 1.0;  // Açısal frekans
// float phi = 0.0;  // Faz açısı
// float alpha = 0.1;  // Mesafe ile sönümleme katsayısı
float beta = 0.1;  // Zaman ile sönümleme katsayısı
// float2 sourcePos = float2(0.5, 0.5);  // Dalga kaynağının konumu
// float startTime = 0.0;  // Dalga kaynağının başlangıç zamanı

// Dalga denklemi
float dx = UV.x - sourcePos.x;
float dy = UV.y - sourcePos.y;
float r = sqrt(dx*dx + dy*dy);
float elapsedTime = Time - startTime;
float damping = exp(-alpha * r - beta * elapsedTime);
float h = A * damping * sin(k * r - omega * elapsedTime + phi);

return h;
```

## Gelişim
### Aşamalar
- Matematik Modellerinin belirlenmesi ve ön görselleştirmelerinin yapılması: bunun için yapay zeka ile modellerin python ile görselleştirilmesine giriştim. Chat GPT-4'ten yararlandım bu sırada.
- Modellerin UE içerisinde HLSL ile yazılması: matematik modellerine karar verildikten sonra HLSL node hazırladım.
- Render Target uygulaması: Render target'lar kullanarak birden fazla simülasyonu aynı shader üzerinde görselleştirebilmeyi hedefledim.
- Test Sahnesinin uygulanması
- Water Shader Polishing
- Main Logic: Begin Play'de default value set ediyorum, render target temizliyorum ve bir canvas render target oluşturuyorum.
- Etkileşime geçtiğimizde (burada E'ye bastığımızda Add Puddle event'i çağırılıyor) simülasyonun Dynamic Material Instance oluşturuyorum ve (M_WaterMaterial_inst) parametrelerini ayarlıyorum. Burada Dynamic Material Instance için Puddles array tutuyorum.
- Event Tick içerisinde Draw Puddles çalışıyor sürekli (optimizasyonlar gelecektir).
- Draw Puddles içerisinde, her Puddles için; yaratılan canvas’a çiziliyor. Sonrasında bu canvası M_waterSim Materyalinin içerisine render target'a basıyorum. Burada TextureRenderTarget mevcut.

## Sonuçlar ve Değerlendirme
### Çözümler
- Birden fazla simülasyonun oluşması render target ile çözümlendi. Burada zaman içerisinde texture güncelleniyor ve, dalgalar birbirlerini yer yer sönümlendirici şekilde çalışıyorlar.
- M_WaterMaterial içerisindeki denemelerde hız ve kütle gibi değişkenlerin olduğu bir versiyonda mevcut. 
