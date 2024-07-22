____
# الخلاصة
لو لقيت اي parameter بيقبل url او حتي path معين في الموقع جرب SSRF واستخدم interactsh البديل ل burp collab علشانه مدفوع!

ولو لقيت ال host بيستخدم AWS جرب تبعت ريكويست لل url دا http://169.254.169.254 ولو ظبط ممكن يجيبلك ال metadata تبع الموقع كامل

___


![[Pasted image 20240721190227.png]]

#### في بعض التكنيكات البسيطة زي:
1. فيه parameter بيقبل url علشان يجيب منه داتا لكن المبرمج مش عامل filter لل input فهنا ممكن ابعت ريكويست لحاجة internal او حتي external
2. وممكن ال internal يكون localhost or 127.0.0.1 او ممكن يكون حاجة في range ال ip دا مثلا: 192.168.1.0/24 - 10.10.10.0/24
3. وفيه نوع blind ودا مش بيرجع منه ريسبونس مرئي ودا بيحتاج server زي مثلا burp collab or clientsh يستقبل ال request <mark style="background: #FF5582A6;">وشرط يكون الريسكويست HTTP and DNS</mark>
#### في بقا حاجات advanced حبتين ودي في الغالب بيكون عامل filter لكن اي كلام زي:
1. انه يكون عامل filter ل localhost and 127.0.0.1 فممكن نعمل bypass ب 127.1 مثلا او 127.0.1 وهكذا
2. او انه يكون بياخد ال url الي جاي من ال parameter ويضيفه علي url localhost من عنده فممكن نعمل bypass كالتالي:

```go
----
stockApi=@192.168.0.12:8080/admin/delete?username=carlos
----
```
هو في الباك اند اي حاجة تجيله من ال stockApi هيحطه علي http://localhost وبالتالي الشكل النهائي هيكون كالتالي:
```
http://localhost@192.168.0.12:8080/admin/delete?username=carlos
```
==وعلامة ال `@` بتعمل redirect لل url اللي جاي بعدها==

3. وانه بقا يعملك whitelist ل بعض ال domains زي مثلا يقولك المسموح هو دا:
```
http://stock.weliketoshop.net
```
ولو جيت تغيره يعملك بلوك طيب في حل؟ اها فيه حل وهو ال username in url! بالشكل التالي:

```
http://localhost%2523@stock.weliketoshop.net/       => %2523 = # but doubled url encode
```

الي قبل ال @ هو ال user بس احنا مش عاوزين user لكن عاوزين نحط localhost علشان ندخل للوكال فنضيف ال # علشان يعمل ignore للي بعدها (ال # دي بتاعة ال DOM)


ومن هنا ممكن تجرب تعمل bypass لبعض الفلترز  (https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md#bypassing-filters)