[EN]

# WaterPuddle

**Contents**

* Test Case Summary
* Understanding the Problem
* Scope and Feature List of the Visual Effect
* Literature Review
* Sample Approaches
* Usable Methods
* Methodology
* Technologies
* Mathematical Models
* Development
* Stages

**Results and Evaluation**

* Problems and Solutions
* Optimization Methods
* Conclusions

---

## Test Case Summary

When an object is dropped onto the water surface, it will create ripples based on its weight and speed. These interactions and the appearance of the water must look realistic.
**Challenge:** The implementation must use Unreal HLSL.

## Understanding the Problem

To understand the required simulation, let’s break down the expectations. For water that appears realistic, we need specific inputs: environment, depth, and other conditions. By assuming these inputs in a default manner, a more generalized structure was created. Since the focus is on the water's behavior, I started searching for mathematical models that could enable this simulation. This revealed the core requirements of the system.

### For Water Surface Appearance:

* Depth Fade
* Fresnel Opacity
* Water Puddle Equation
* Water Splash FX

## Literature Review

Upon examining sample approaches, I found that many sources implemented different mathematical models. ShaderToy featured implementations under terms like “water rain puddle.” Game engine versions also exist; these often use “fake” simulations to boost performance. In this context, using a game engine is ideal for physics-interactive visual effects. In Unreal Engine's Content Examples, a Render Target example map demonstrates how ripple effects are visualized using render targets—another viable alternative.

## Methodology

Since I needed to use custom shaders for visual effects, I chose to work in Unreal Engine 4.27. I can execute puddle simulation equations within Unreal via custom HLSL nodes.
**Challenge:** Simultaneous interaction of multiple objects.
Running a puddle simulation in a single material is not a problem. The simulation begins based on a start position input, and evolves over time. However, running multiple simulations would require an array of start positions—which is currently limited. Still, workarounds exist. For example, using a Render Target.

A significant part of the research focused on mathematical models and their feasibility.

### Height Field Water Models

* Represent the water surface as a heightmap.
* More suitable for real-time applications with lower computational costs.

### Wave Equation

* A basic equation for simulating wave movement.
* Fast and easy to implement.
* Suitable for basic wave behavior but may fall short for detailed water dynamics.

### “The Puddle Equation”

* The most realistic model I found.
* Potentially expensive in terms of performance, so I didn’t pursue it further.
  [Read more on the Puddle Equation](https://people.rit.edu/nsbsma/home/Puddles.html)

I also used AI support during the research of mathematical models.

### Chosen Model: Height Field Water Model

Basic shader code (simplified):

```hlsl
// float A = 1.0;  // Wave amplitude
// float k = 1.0;  // Wave number
// float omega = 1.0;  // Angular frequency
// float phi = 0.0;  // Phase angle
// float alpha = 0.1;  // Distance damping coefficient
float beta = 0.1;  // Time damping coefficient
// float2 sourcePos = float2(0.5, 0.5);  // Position of wave origin
// float startTime = 0.0;  // Start time of the wave

// Wave equation
float dx = UV.x - sourcePos.x;
float dy = UV.y - sourcePos.y;
float r = sqrt(dx*dx + dy*dy);
float elapsedTime = Time - startTime;
float damping = exp(-alpha * r - beta * elapsedTime);
float h = A * damping * sin(k * r - omega * elapsedTime + phi);

return h;
```

## Development

### Stages

* **Defining Mathematical Models and Initial Visualizations:** I used AI and Python to visualize models during this stage, with help from ChatGPT-4.
* **Implementing Models in UE using HLSL:** After choosing the math models, I created the necessary HLSL nodes.
* **Render Target Implementation:** I aimed to visualize multiple simulations within the same shader using render targets.
* **Creating the Test Scene**
* **Water Shader Polishing**
* **Main Logic:** On `Begin Play`, I set default values, cleared the render target, and created a canvas render target.
* **Interaction:** When we interact (e.g., pressing 'E'), the "Add Puddle" event is triggered. I create a dynamic material instance (M\_WaterMaterial\_inst) and configure its parameters. I maintain an array of puddles for the dynamic material instance.
* **In the Event Tick:** `Draw Puddles` runs continuously (optimizations will be applied).
* **Within Draw Puddles:** Each puddle is drawn to the canvas. This canvas is then rendered to the target texture in the `M_waterSim` material. A `TextureRenderTarget` is used here.

## Results and Evaluation

### Solutions

* The problem of simulating multiple puddles was solved using a render target. The texture updates over time, and the ripples interact with each other, sometimes cancelling each other out.
* In experiments with `M_WaterMaterial`, a version that includes variables such as speed and mass was also tested.



[TR]


# WaterPuddle

İçerik
- Test Case özeti
- Sorunu Anlamak
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
