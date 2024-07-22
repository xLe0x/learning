
1. https://medium.com/@oXnoOneXo/a-story-of-a-nice-ssrf-vulnerability-51e16ff6a33f

هنا الراجل في البداية جرب يعمل request ل http://169.254.169.254 علشان يرجع ال metadata تبع الموقع لانه بيستخدم AWS لكن مظبطش
وجرب كذا تكنيك زي:
	1. IP obfuscation with [https://github.com/vysecurity/IPFuscator](https://github.com/vysecurity/IPFuscator)
	2. getting a redirect to 169.254.169.254 with ([http://nicob.net/redir6a](http://nicob.net/redir6a))
	3. IPv6
	4. ==DNS Rebinding attack== and its online tool ([https://lock.cmpxchg8b.com/rebinder.html](https://lock.cmpxchg8b.com/rebinder.html)).
هنا استخدم ال DNS Rebinding وعلي ما فهمت من كلامه انه بتديله ip وليكن جوجل وبتديله ip تاني انت عاوزه يعمله resolve وفي الحالة دي هيكون ال 169.254.169.254
ظبطت معاه لكن كان فيه مشكلة وانه بيرجع 401 ==قرأ الدوكس تبع AWS== لقي انه لازم يبعت الريكويست بهيدر `X-aws-ec2-metadata-token-ttl-seconds: 21600` وهيجيله في الريسبونس ``X-aws-ec2-metadata-token: TOKEN`` هياخد التوكن ويحطه في الريكويست تاني وبس كدا ❤️‍🔥
____

2. https://www.blackhillsinfosec.com/hunting-for-ssrf-bugs-in-pdf-generators/


___

3. https://geleta.eu/2019/my-first-ssrf-using-dns-rebinfing/

---

4. https://github.com/jdonsec/AllThingsSSRF