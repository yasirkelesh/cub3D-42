# Ray-Casting


Raycasting'i uygulamak için kullandığım formülleri burada açıklayacağım. Bu raycast, Cub3D adlı 3D bir oyun oluşturmak için yapılmıştır. Proje hala devam etmektedir ⚙️🛠.

Not: Henüz tuşlarla ilgili bitmiş değilim .. Ve evet, oyuncu duvarların içinden geçebilir 🧟‍♀️

Eğer raycasting veya oyun geliştirmeyle ilgili herhangi bir konuda spesifik sorularınız varsa, sormaktan çekinmeyin!

## Introduction

Raycasting'in genel fikri basittir ve formüller çok karmaşık değildir. Duvarları oluşturmak, çizmek ve etrafında hareket etmek için kullandığım hesaplamaları açıklayacağım.

Öncelikle, formüllerde kullandığım matematik kavramlarını size anlatayım.. [Right triangle trigonometry (SOHCAHTOA)](https://www.mathsisfun.com/algebra/sohcahtoa.html), and [Vectors: Components(X , Y), Quantity (magnitude and direction)](https://www.varsitytutors.com/hotmath/hotmath_help/topics/components-of-a-vector). 

## Main Idea

Formüllerin ana fikrini açıklamak için, oyuncunun 2D harita üzerindeki konumunu (x, y) koordinatlarıyla varsayalım. "Oyuncu, kendisinin önündeki duvara (KUZEY) bakıyor, diyelim ki bakış açısı 90 ve doğrudan kuzeye doğru bakıyor." Kendi koordinatından duvarın noktasına olan mesafe bir ışını temsil eder. Oyuncunun Görüş Açısı (FOV)'na bağlı olarak ışınların sayısı belirlenecektir. "Eğer FOV 120 ise, bakış genişliği bakış açısından (90) sola 60 derece ve sağa 60 derece olacak şekilde 60 derece olur".

![](https://github.com/Saxsori/ray-cast/blob/main/images/5.png)

#### Vektörleri nerede kullanacağımızı belirtelim.

Oyuncunun koordinatını ve bakış açısını kullanarak, ışının vektörünü (X ve Y bileşenleri) hesaplayabiliriz. Ardından, ışınların dikey ve yatay olarak duvara ulaşabileceği noktaların ofsetini (ızgara çizgisi aykırısı) hesaplayabiliriz

## GridLine Hit Checkers (Izgara Çizgisi Kontrol Edicileri)
Her bir ışın, yatay veya dikey olarak bir ızgara çizgisine çarptığında, aslında duvarın olup olmadığını kontrol etmemiz gereken nokta olmalıdır. Bunun için öncelikle oyuncunun koordinatlarından hareketle ışının vektörünü hesaplamamız gerekiyor, ardından ilk ızgara çizgisine çarpmak için belirli bir değeri (parametreleri) ekliyoruz ve ardından duvara ulaşmak için her bir ızgara çizgisine çarpmak için sürekli bir sabit değeri ekliyoruz. Bu sabit değerlere de (Izgara Ofsetleri) denir.

![alt text](https://github.com/Saxsori/ray-cast/blob/main/images/1.png)

Yatay Izgara Çizgileri, (KUZEY ve GÜNEY) veya 2D haritanın üst ve alt kenarlarıdır. Dikey Izgara Çizgileri ise (BATI ve DOĞU) veya 2D haritanın sol ve sağ kenarlarıdır.
Oyuncunun bakış açısına bağlı olarak, ışının aslında dikey olarak nereye çarptığını (sol veya sağ) ve yatay olarak nereye çarptığını (yukarı veya aşağı) belirleyebiliriz. Bunun öncesinde aslında ışının çizgisi ve ışının vektörünü nasıl hesaplayabileceğimizi anlamamız gerekiyor.

Işının vektörünü (X ve Y bileşenleri) ışın Y ve ışın X elde etmek için bilinmesi gereken ilk şey, sağ üçgenin kurallarını kullanabileceğimizdir.
### Example
Bu, ışının ızgara çizgilerine çarptığında nasıl görüneceğiyle ilgili bir örnektir. Onları çizmek için desmos kullandım. Ve ofsetleri ve parametreyi manuel olarak ekledim, bakış açısı 60 ve pozisyon (77, 77) için. Kırmızı nokta oyuncunun pozisyonunu, turuncu noktalar üst tarafı, yeşil noktalar alt tarafı, siyah noktalar sol tarafı ve mor noktalar sağ tarafı temsil eder.

<img width="635" alt="Screen Shot 2022-08-27 at 7 37 59 AM" src="https://user-images.githubusercontent.com/92129820/187012948-2590fd02-a71b-461b-bd28-6a7101aa5ac4.png">

### Yatay Izgara Çizgisi

Yatay Izgara Çizgisi Kontrol Edici Formülleri hesaplamak için şu adımları izleyebiliriz:

Işın Y noktası, oyuncunun Y koordinatı olmalıdır.

Ardından ışın X noktasını elde etmek için sağ üçgen kurallarını (SOHCAHTOA) kullanabiliriz. Işın hattı Hipotenüs olacak, Karşıt ise oyuncunun Y koordinatı ile Işın Y noktası arasındaki fark olacak ve Yanlış ise aradığımız ışın X bileşeni olacak. Dolayısıyla ışın X, karşıt (ışın Y) / tan(bakış açısı) olarak hesaplanır.
 ![alt text](https://github.com/Saxsori/ray-cast/blob/main/images/4.png) 

Bunları ızgara boyutuna (64) ölçeklendirmek için:

Işın Y: Öncelikle oyuncunun Y noktasını 64 birime ölçeklendirmeniz gerekmektedir. ((pY / 64) * 64);
Işın X: Oyuncunun X koordinatını eklemelisiniz. (+ pX);
Bir sonraki yatay ızgara çizgisine çarpmalarını sağlamak için bir parametre eklememiz gerekmektedir:

Işın Y noktasına, oyuncu pozisyonundan bir sonraki yatay ızgara çizgisine olan mesafeyi eklemeniz gerekmektedir (oyuncu Y koordinatı ile bir sonraki Y çizgisi arasındaki fark), böylece bir sonraki yatay ızgara çizgisine çarpabilir.

#### Oyuncu (Kuzey) yönüne baktığında formül son olarak şu şekilde olmalıdır:

```ruby
rayY = ((pY / 64) * 64) + (Y.line - pY);
rayX = (pY - rayY) / -tan(looking angle) + pX;
```
![](https://github.com/Saxsori/ray-cast/blob/main/images/7.png)

####  Eğer oyuncu (Güney) yönüne bakıyorsa, yön yatayda farklı olduğu için işaretler sadece değişecektir. Aşağıdaki gibi güncellenir:

```ruby
rayY = ((pY / 64) * 64) - (Y.line - pY);
rayX = (pY - rayY) / -tan(looking angle) + pX;
````
![](https://github.com/Saxsori/ray-cast/blob/main/images/8.png)

### Vertical Gridline 

Dikey ızgara çizgisi çarpmasını kontrol etmek için, aynı mantığı kullanırız ancak zıttıdır. Hesaplama sadece ışın Y'ye bağlı olacaktır, çünkü ışın Y noktasına çarpabilecek Y çizgilerine bağlıdır. Daha önce yatay kontrol edicilerde ışın X'ye bağlıydı, çünkü X çizgilerine çarpabiliyordu.

Bu nedenle formül şu şekilde değişir:

#### Eğer oyuncu doğuya bakıyorsa, formül şu şekilde olacaktır:

```ruby
rayX = ((pX / 64) * 64) + (X.line - pX);
rayY = (pX - rayX) / -tan(looking angle) + pY;
```
![](https://github.com/Saxsori/ray-cast/blob/main/images/9.png)

#### Eğer oyuncu batıya bakıyorsa, yön farklı olacağından işaretler yine değişir.
```ruby
rayX = ((pX / 64) * 64) - (X.line - pX);
rayY = (pX - rayX) / -tan(looking angle) + pY;
```
![](https://github.com/Saxsori/ray-cast/blob/main/images/10.png)

### Gridline Offset


Ofset, temel olarak hesaplamanın hatadan veya aykırı noktadan çıktığını veya hangi noktaya çarpabileceğini belirtmek için kullanılan bir miktar veya değeri ifade eder. Burada, bir sonraki ızgara çizgisine çarpmak için her seferinde eklemek istediğimiz değeri ifade eder. Yani, ışınların tam olarak ızgara çizgilerine çarpmasını istiyoruz. Bu nedenle, bu değerleri hesaplamak için tekrar SOHCAHTOA kullanacağız.

![](https://github.com/Saxsori/ray-cast/blob/main/images/6.png)

#### Eğer oyuncu Kuzey'e bakıyorsa
Y ofseti ızgara boyutu (64) olacaktır. Bu şekilde bir sonraki yatay çizgiye çarpabilir. X ofsetini elde etmek için (SOHCAHTOA) -> Y ofseti * Tan formülünü kullanabiliriz.

```ruby
oY = 64;
oX = oY * tan(looking angle);
```

![](https://github.com/Saxsori/ray-cast/blob/main/images/H-U.png)

#### Eğer oyuncu Güney'e bakıyorsa, 
ofsetler aynı olacak, sadece Y'nin yönü değişecektir.

```ruby
oY = -64;
oX = oY * tan(looking angle);
```

![](https://github.com/Saxsori/ray-cast/blob/main/images/H-D.png)

#### Eğer oyuncu doğuya bakıyorsa, 
X ofseti ızgara boyutu (64) olacaktır. Bu şekilde bir sonraki dikey çizgiye çarpabilir. Y ofsetini elde etmek için (SOHCAHTOA) -> X ofseti * Tan formülünü kullanabiliriz.

````ruby
oX = 64;
oY = oX * tan(looking angle);
````

![](https://github.com/Saxsori/ray-cast/blob/main/images/V-R.png)

#### Eğer oyuncu batıya bakıyorsa
 ofsetler aynı olacak, sadece X'in yönü değişecektir.

```ruby
oX = -64;
oY = oX * tan(looking angle);
```

![](https://github.com/Saxsori/ray-cast/blob/main/images/V-L.png)

Sonra, duvar kontrol döngüsünü gerçekleştiririz. Döngüde, ışın değerlerine ofset değerlerini ekleriz, böylece duvara çarpıncaya kadar devam eder.

```ruby
while (!wall)
{
 rayY += oY;
 rayX += oX;
}
```

Örneğin, diyelim ki oyuncunun bakış açısı 60'tır. Kuzey'e doğru Doğu'ya bakıyor. Dikey olarak çizgi sağ tarafa çarpmalı ve yatay olarak üst tarafa çarpmalıdır. Kullanacağımız formüller ise şunlardır:
```ruby
rayY = ((pY / 64) * 64) + (Y.line - pY);
rayX = (pY - rayY) / -tan(looking angle) + pX;
oX = 64;
oY = oX * tan(looking angle);
```

Ve dikey olarak, çevirsek:

```ruby
rayX = ((pX / 64) * 64) + (X.line - pX);
rayY = (pX - rayX) / -tan(looking angle) + pY;
oY = 64;
oX = oY * tan(looking angle);
```
Şimdi duvar kontrol döngüsünü yapabiliriz (ışın noktalarına ofsetleri ekleyerek), duvara çarpıncaya kadar devam ederiz. Ardından dikey kontrol ve yatay kontrol için ışının hattını hesaplayarak en kısa olanı seçeriz (duvara ilk çarpan).
![](https://github.com/Saxsori/ray-cast/blob/main/images/3.png)

Böylece bakış açısına bağlı olarak duvara kadar ulaşan noktaları elde etmek için doğru formülleri seçebiliriz.

## Duvarları çizme

Duvara ilk çarpanın dikey mi yoksa yatay mı ızgara çizgisi kontrol edicisinin olduğunu kontrol etmek için bir döngü oluşturun. Ardından, ışının duvara çarptığı nokta ile oyuncunun konumu arasındaki mesafeyi hesaplayın ve bu ışın hattının uzunluğu olmalıdır. Bu hesaplama için mesafeyi hesaplamak için Pythagoras formülünü kullanabilirsiniz.[pythagorean rule](https://courses.lumenlearning.com/waymakercollegealgebra/chapter/distance-in-the-plane/) to calculate the length of the line between two points. $distance = \sqrt{{(X2 - X1)}^2 + {(Y2 - Y1)}^2}$


Bu değeri (ışının uzunluğunu) çizim için kaydedin, ancak öncelikle ekranın genişliği ve yüksekliğinin nasıl ölçeklendirilmesi gerektiğini açıklayalım. Nasıl görünmesi gerektiği hakkında bilgi verelim!
### Ekranın Genişliği
Ekranın genişliği sabit olacak. Ekran genişliğini istediğiniz şekilde seçebilirsiniz. Ekran genişliği, oyuncunun görebileceği görüntünün genişliğidir. Genişlik, ışınların sayısı olacaktır. Her bir ışın arasındaki açı, FOV / genişlik olmalıdır.
`````ruby
angle = looking_angle - (FOV / 2);
while (--WIDTH)
{
   ray[i].angle = angle;
   angle += (FOV / WIDTH);
}
`````

### Ekranın Yüksekliği
Ekranın yüksekliği de sabit olacak. Ekran yüksekliği, duvarın ulaşabileceği maksimum yüksekliktir ve istediğiniz bir değeri seçebilirsiniz. Her duvarın yüksekliğini elde etmek için ızgara boyutunu (64) ekran yüksekliğiyle çarpıp ışın uzunluğuna bölmelisiniz.
````ruby
wall_height = (64 * HEIGHT) / ray_len;
````
## Çizim
Tamam, hepsini bir araya getirelim. Tüm bu fikirleri kullanarak sonunda şu adımları takip edebiliriz. Işınların sayısı genişlik boyutu olduğundan bu döngüyü oluşturabiliriz, döngü içinde aşağıdaki adımları gerçekleştirmeliyiz.

1- Izgara çizgisi kontrolünü yapın ve son ışın noktalarını elde edin.

2- Duvar yüksekliğini, ekranın YÜKSEKLİĞİ ve ışın uzunluğunu kullanarak hesaplayın.

3- Duvarların başlangıç noktasını ve bitiş noktasını hesaplayın.

4- Çizmeye başlayın.

````ruby
x = -1;
while (++x < WIDTH)
{
   check_gridline(ray[x]);
   ray[x].wall_height = (64 * HEIGHT) / ray[x].ray_len;
   begin = (HEIGHT / 2) - (wall_height / 2);
   end = (HEIGHT / 2) + (wall_height / 2);
   y = begin - 1;
   while (++y < end)
     draw_line(x, y);
}
````

## Finally ! This how it should look like ✌🏼

![](https://github.com/Saxsori/ray-cast/blob/main/images/20.gif)

## Resources
- [3DSage Raycasting](https://www.youtube.com/watch?v=gYRrGTC7GtA)

- [Lode's Tutorial](https://lodev.org/cgtutor/raycasting.html)

- [Gridline checkers](https://www.permadi.com/tutorial/raycast/rayc7.html)
 
