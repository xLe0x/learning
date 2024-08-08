___
# الخلاصة

الثغرة دي مع الوقت بيقل وجودها علشان ال browsers بتعمل حماية منها لكن ما زالت موجودة!

==وملحوظة في حتة ال SOP ليه مش بتمنع ال CSRF attacks؟ بكل بساطة لأنها بتمنع ان المهاجم يقرأ responses مش بتمنع انك تبعت ريكويستات!==


___



## ما هي؟

ثغرة أمنية على الويب تسمح للمهاجم بحث المستخدمين على تنفيذ إجراءات لا ينوون تنفيذها. فهو يسمح للمهاجم بالتحايل جزئيًا على نفس سياسة الأصل، والتي تم تصميمها لمنع مواقع الويب المختلفة من التدخل مع بعضها البعض.

## ما هو تأثير هجوم CSRF؟

في هجوم CSRF الناجح، يتسبب المهاجم في قيام المستخدم الضحية بتنفيذ إجراء ما عن غير قصد. على سبيل المثال، قد يكون ذلك لتغيير عنوان البريد الإلكتروني الموجود في حسابهم، أو تغيير كلمة المرور الخاصة بهم، أو إجراء تحويل أموال.

## كيف يعمل CSRF؟

علشان يكون فيه attack ناجح لازم يبقا فيه 3 شروط.
1. طبعا لازم يبقا فيه action مش عليه حماية زي مثلا تغيير الايميل
2. الريكويست يطلب الكوكيز واكيد حاجة زي تغيير الايميل بتتطلب كوكيز
3. مفيش اي parameters متوقعة او متغيرة تعطل ال attack (الا لو عملت لها bypass)

```HTTP
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 30 
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE 

email=wiener@normal-user.com
```

POC for that request.
```html
<html> 
	<body> 
	
		<form action="https://vulnerable-website.com/email/change" method="POST"> 
			<input type="hidden" name="email" value="pwned@evil-user.net" /> 
		</form> 
		
	<script>document.forms[0].submit();</script> 
	
	</body> 
</html>
```


لاحظ أن بعض عمليات استغلال CSRF البسيطة تستخدم طريقة GET ويمكن أن تكون مكتفية ذاتيًا بالكامل باستخدام عنوان URL واحد على موقع الويب الضعيف. في هذه الحالة، قد لا يحتاج المهاجم إلى استخدام موقع خارجي، ويمكنه تغذية الضحايا مباشرةً بعنوان URL ضار على النطاق الضعيف. في المثال السابق، إذا كان من الممكن تنفيذ طلب تغيير عنوان البريد الإلكتروني باستخدام طريقة GET، فسيبدو الهجوم المستقل كما يلي:

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```


## بعض الحمايات ضد ال CSRF

- **CSRF tokens**
- **SameSite cookies** 
ال SameSite Cookies بتحد من استخدام الكوكيز خارج الموقع ==يعني اي موقع خارجي بيطلب ريكويست authenticated من موقع عامل الكوكيز SameSite يبقا الكوكيز دي مش هتروح مع الريكويست==


- **Referer-based validation**
تستخدم بعض التطبيقات Header HTTP Referer لمحاولة الدفاع ضد هجمات CSRF، عادةً عن طريق التحقق من أن الطلب نشأ من المجال الخاص بالتطبيق. يعد هذا بشكل عام أقل فعالية من التحقق من صحة  CSRF token.


## Bypasses

### Bypassing CSRF token validation

##### 1. Validation of CSRF token depends on request method

Some applications correctly validate the token when the request uses the POST method but skip the validation when the GET method is used.
In this situation, the attacker can switch to the GET method to bypass the validation and deliver a CSRF attack:

```HTTP
GET /email/change?email=pwned@evil-user.net HTTP/1.1 
Host: vulnerable-website.com 
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

##### 2. Validation of CSRF token depends on token being present

Some applications correctly validate the token when it is present but skip the validation if the token is omitted.
In this situation, the attacker can remove the entire parameter containing the token (not just its value) to bypass the validation and deliver a CSRF attack:

```HTTP
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 25 
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm 

email=pwned@evil-user.net 
```
بعد م شلنا ال csrf parameter ال validation اتشال!


##### 3. CSRF token is not tied to the user session CSRF token is not tied to the user session

Some applications do not validate that the token belongs to the same session as the user who is making the request. Instead, the application maintains a global pool of tokens that it has issued and accepts any token that appears in this pool.
In this situation, the attacker can log in to the application using their own account, obtain a valid token, and then feed that token to the victim user in their CSRF attack.


##### 4. CSRF token is tied to a non-session cookie

In a variation on the preceding vulnerability, some applications do tie the CSRF token to a cookie, but not to the same cookie that is used to track sessions. This can easily occur when an application employs two different frameworks, one for session handling and one for CSRF protection, which are not integrated together:

```HTTP
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 68 
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv 

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

This situation is harder to exploit but is still vulnerable. If the website contains any behavior that allows an attacker to set a cookie in a victim's browser, then an attack is possible. The attacker can log in to the application using their own account, obtain a valid token and associated cookie, leverage the cookie-setting behavior to place their cookie into the victim's browser, and feed their token to the victim in their CSRF attack.


##### 5. CSRF token is simply duplicated in a cookie

In a further variation on the preceding vulnerability, some applications do not maintain any server-side record of tokens that have been issued, but instead duplicate each token within a cookie and a request parameter. When the subsequent request is validated, the application simply verifies that the token submitted in the request parameter matches the value submitted in the cookie. This is sometimes called the "double submit" defense against CSRF, and is advocated because it is simple to implement and avoids the need for any server-side state:

```HTTP
POST /email/change HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 68 
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa 

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

In this situation, the attacker can again perform a CSRF attack if the website contains any cookie setting functionality. Here, the attacker doesn't need to obtain a valid token of their own. They simply invent a token (perhaps in the required format, if that is being checked), leverage the cookie-setting behavior to place their cookie into the victim's browser, and feed their token to the victim in their CSRF attack.


### Bypassing SameSite cookie restrictions

منذ عام 2021، يطبق Chrome قيود Lax SameSite افتراضيًا إذا لم يقم موقع الويب الذي يصدر ملف تعريف الارتباط بتعيين مستوى التقييد الخاص به بشكل صريح. هذا معيار مقترح، ونتوقع أن تتبنى المتصفحات الرئيسية الأخرى هذا السلوك في المستقبل. ونتيجة لذلك، من الضروري أن يكون لديك فهم قوي لكيفية عمل هذه القيود، بالإضافة إلى كيفية تجاوزها، من أجل إجراء اختبار شامل لناقلات الهجوم عبر المواقع.

###### ما هو الموقع في سياق ملفات تعريف الارتباط SameSite؟  
في سياق قيود ملفات تعريف الارتباط SameSite، يتم تعريف الموقع على أنه نطاق المستوى الأعلى (TLD)، وعادةً ما يكون مثل .com أو .net، بالإضافة إلى مستوى إضافي واحد من اسم النطاق. ويُشار إلى هذا غالبًا باسم TLD+1.  

							 ![[Pasted image 20240808181053.png]]
عند تحديد ما إذا كان الطلب من نفس الموقع أم لا، يتم أيضًا أخذ نظام URL في الاعتبار. وهذا يعني أن الرابط من http://app.example.com إلى https://app.example.com يتم التعامل معه على أنه موقع مشترك في معظم المتصفحات.  
  

> [!ملحوظة]
> قد تصادف مصطلح "نطاق المستوى الأعلى الفعال" (eTLD). وهذه مجرد طريقة لحساب اللواحق متعددة الأجزاء المحجوزة التي يتم التعامل معها على أنها نطاقات المستوى الأعلى عمليًا، مثل .co.uk.

##### What's the difference between a site and an origin?

The difference between a site and an origin is their scope; a site encompasses multiple domain names, whereas an origin only includes one. Although they're closely related, it's important not to use the terms interchangeably as conflating the two can have serious security implications.

Two URLs are considered to have the same origin if they share the exact same scheme, domain name, and port. Although note that the port is often inferred from the scheme.
![[Pasted image 20240808181823.png]]
As you can see from this example, the term "site" is much less specific as it only accounts for the scheme and last part of the domain name. Crucially, this means that a cross-origin request can still be same-site, but not the other way around.

|   |   |   |   |
|---|---|---|---|
|**Request from**|**Request to**|**Same-site?**|**Same-origin?**|
|`https://example.com`|`https://example.com`|Yes|Yes|
|`https://app.example.com`|`https://intranet.example.com`|Yes|No: mismatched domain name|
|`https://example.com`|`https://example.com:8080`|Yes|No: mismatched port|
|`https://example.com`|`https://example.co.uk`|No: mismatched eTLD|No: mismatched domain name|
|`https://example.com`|`http://example.com`|No: mismatched scheme|No: mismatched scheme|

This is an important distinction as it means that any vulnerability enabling arbitrary JavaScript execution can be abused to bypass site-based defenses on other domains belonging to the same site. We'll see an example of this in one of the labs later.


#### How does SameSite work?

قبل تقديم آلية SameSite، كانت المتصفحات ترسل ملفات تعريف الارتباط في كل طلب إلى النطاق الذي أصدرها، حتى لو تم تشغيل الطلب بواسطة موقع ويب تابع لجهة خارجية غير ذي صلة. يعمل SameSite من خلال تمكين المتصفحات ومالكي مواقع الويب من تحديد الطلبات عبر المواقع، إن وجدت، التي يجب أن تتضمن ملفات تعريف ارتباط محددة. يمكن أن يساعد هذا في تقليل تعرض المستخدمين لهجمات CSRF، والتي تحفز متصفح الضحية على إصدار طلب يؤدي إلى اتخاذ إجراء ضار على موقع الويب الضعيف. وبما أن هذه الطلبات تتطلب عادةً ملف تعريف ارتباط مرتبطًا بجلسة المصادقة الخاصة بالضحية، فسوف يفشل الهجوم إذا لم يتضمن المتصفح هذا الملف.

All major browsers currently support the following SameSite restriction levels:

- [`Strict`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#strict)
- [`Lax`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#lax)
- [`None`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#none)

Developers can manually configure a restriction level for each cookie they set, giving them more control over when these cookies are used. To do this, they just have to include the `SameSite` attribute in the `Set-Cookie` response header, along with their preferred value:

`Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict`

Although this offers some protection against CSRF attacks, none of these restrictions provide guaranteed immunity, as we'll demonstrate using deliberately vulnerable, interactive labs later in this section.

> [!ملحوظة]
> لو الموقع مش عامل configure لل SameSite Cookies ف by default كروم بيعمل ال restriction level ل Lax بدون اي تدخل من الموقع
> 
##### Strict
لو الكوكيز فيها `SameSite=Strict`  المتصفحات مش هتبعت اي cross-site request بعبارات بسيطة، هذا يعني أنه إذا كان الموقع المستهدف للطلب لا يتطابق مع الموقع الموضح حاليًا في شريط عنوان المتصفح، فلن يتضمن ملف تعريف الارتباط.

يوصى بهذا عند تعيين ملفات تعريف الارتباط التي تمكن حاملها من تعديل البيانات أو تنفيذ إجراءات حساسة أخرى، مثل الوصول إلى صفحات محددة متاحة فقط للمستخدمين المصادق عليهم. على الرغم من أن هذا هو الخيار الأكثر أمانًا، إلا أنه يمكن أن يؤثر سلبًا على تجربة المستخدم في الحالات التي تكون فيها الوظيفة عبر المواقع مرغوبة. (ارسال واستقبال الريكويستات من مواقع اخري)


##### Lax
ومعناها انه هيبعت الكوكيز في ال cross-site requests ولكن لو توافق شرطين وهم:

- The request uses the `GET` method.
- The request resulted from a top-level navigation by the user, such as clicking on a link.

وهذا يعني أن ملف تعريف الارتباط غير مضمن في طلبات POST عبر المواقع، على سبيل المثال. نظرًا لأن طلبات POST تُستخدم عمومًا لتنفيذ إجراءات تعدل البيانات أو الحالة (على الأقل وفقًا لأفضل الممارسات)، فمن المرجح أن تكون هدفًا لهجمات CSRF.  
  
وبالمثل، لا يتم تضمين ملف تعريف الارتباط في طلبات الخلفية، مثل تلك التي تبدأ بواسطة البرامج النصية أو إطارات iframe أو المراجع إلى الصور والموارد الأخرى



##### None

إذا تم تعيين ملف تعريف الارتباط باستخدام السمة `SameSite=None` ، فسيؤدي ذلك إلى تعطيل قيود SameSite بشكل فعال، بغض النظر عن المتصفح. ونتيجة لذلك، سترسل المتصفحات ملف تعريف الارتباط هذا في جميع الطلبات إلى الموقع الذي أصدره، حتى تلك التي تم تشغيلها بواسطة مواقع خارجية غير ذات صلة تمامًا.  
  
باستثناء Chrome، هذا هو السلوك الافتراضي الذي تستخدمه المتصفحات الرئيسية إذا لم يتم توفير سمة SameSite عند تعيين ملف تعريف الارتباط.  
  
هناك أسباب مشروعة لتعطيل SameSite، كما هو الحال عندما يكون ملف تعريف الارتباط مخصصًا للاستخدام من سياق جهة خارجية ولا يمنح حامله إمكانية الوصول إلى أي بيانات أو وظائف حساسة. يعد تتبع ملفات تعريف الارتباط مثالًا نموذجيًا.  
  
إذا واجهت ملف تعريف ارتباط تم تعيينه باستخدام `SameSite=None` أو بدون قيود صريحة، فمن المفيد التحقق مما إذا كان له أي فائدة. عندما تم اعتماد سلوك "Lax-by-default" لأول مرة بواسطة Chrome، كان لذلك أثر جانبي يتمثل في تعطيل الكثير من وظائف الويب الحالية. كحل سريع، اختارت بعض مواقع الويب ببساطة تعطيل قيود SameSite على جميع ملفات تعريف الارتباط، بما في ذلك الملفات الحساسة.  
  
عند تعيين ملف تعريف ارتباط باستخدام `SameSite=None` ، يجب أن يتضمن موقع الويب أيضًا السمة الآمنة، والتي تضمن إرسال ملف تعريف الارتباط فقط في رسائل مشفرة عبر HTTPS. وإلا فإن المتصفحات سترفض ملف تعريف الارتباط ولن يتم تعيينه.

```HTTP
Set-Cookie: trackingId=0F8tgdOhi9ynR1M9wa3ODa; SameSite=None; Secure
```


#### طرق تخطي هذه الحمايات

##### Bypassing Lax using GET requests

من الناحية العملية، لا تهتم الخوادم دائمًا بما إذا كانت ستتلقى طلب GET أو POST إلى نقطة نهاية معينة، حتى تلك التي تتوقع إرسال نموذج. إذا كانوا يستخدمون أيضًا قيود Lax لملفات تعريف الارتباط الخاصة بالجلسة، إما بشكل صريح أو بسبب الإعداد الافتراضي للمتصفح، فقد تظل قادرًا على تنفيذ هجوم CSRF عن طريق الحصول على طلب GET من متصفح الضحية.  
  
وطالما أن الطلب يتضمن تنقلًا عالي المستوى، فسيظل المتصفح يتضمن ملف تعريف ارتباط جلسة الضحية. فيما يلي أحد أبسط الطرق لشن مثل هذا الهجوم:

```HTML
<script> 
	document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000'; 
</script>
```

Even if an ordinary `GET` request isn't allowed, some frameworks provide ways of overriding the method specified in the request line. For example, Symfony supports the `_method` parameter in forms, which takes precedence over the normal method for routing purposes:

```HTML
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST"> 
	<input type="hidden" name="_method" value="GET"> 
	<input type="hidden" name="recipient" value="hacker"> 
	<input type="hidden" name="amount" value="1000000"> 
</form>
```
Other frameworks support a variety of similar parameters.

##### Bypassing SameSite restrictions using on-site gadgets

If a cookie is set with the `SameSite=Strict` attribute, browsers won't include it in any cross-site requests. You may be able to get around this limitation if you can find a gadget that results in a secondary request within the same site.

One possible gadget is a client-side redirect that dynamically constructs the redirection target using attacker-controllable input like URL parameters. For some examples, see our materials on [DOM-based open redirection](https://portswigger.net/web-security/dom-based/open-redirection).

As far as browsers are concerned, these client-side redirects aren't really redirects at all; the resulting request is just treated as an ordinary, standalone request. Most importantly, this is a same-site request and, as such, will include all cookies related to the site, regardless of any restrictions that are in place.

If you can manipulate this gadget to elicit a malicious secondary request, this can enable you to bypass any SameSite cookie restrictions completely.

*Note that the equivalent attack is not possible with server-side redirects. In this case, browsers recognize that the request to follow the redirect resulted from a cross-site request initially, so they still apply the appropriate cookie restrictions.*


##### Bypassing SameSite restrictions via vulnerable sibling domains

سواء كنت تختبر موقع ويب خاصًا بشخص آخر أو تحاول تأمين موقعك الإلكتروني، فمن الضروري أن تضع في اعتبارك أن الطلب قد يظل من نفس الموقع حتى لو تم إصداره عبر مصدر مشترك.

تأكد من إجراء تدقيق شامل لجميع مناطق الهجوم المتاحة، بما في ذلك أي نطاقات شقيقة. على وجه الخصوص، يمكن لنقاط الضعف التي تمكنك من الحصول على طلب ثانوي عشوائي، مثل XSS، أن تعرض الدفاعات القائمة على الموقع للخطر تمامًا، مما يعرض جميع نطاقات الموقع للهجمات عبر المواقع.

بالإضافة إلى CSRF الكلاسيكي، لا تنس أنه إذا كان موقع الويب المستهدف يدعم WebSockets، فقد تكون هذه الوظيفة عرضة ل [cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking) ([CSWSH](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking)) ، والذي يعد في الأساس مجرد هجوم CSRF يستهدف مصافحة WebSocket. لمزيد من التفاصيل، راجع موضوعنا حول [WebSocket vulnerabilities](https://portswigger.net/web-security/websockets).

##### Bypassing SameSite Lax restrictions with newly issued cookies

لا يتم عادةً إرسال ملفات تعريف الارتباط ذات قيود Lax SameSite في أي طلبات POST عبر المواقع، ولكن هناك بعض الاستثناءات.  
  
كما ذكرنا سابقًا، إذا لم يتضمن موقع الويب سمة SameSite عند تعيين ملف تعريف الارتباط، فسيطبق Chrome تلقائيًا قيود Lax افتراضيًا. ومع ذلك، لتجنب كسر آليات تسجيل الدخول الموحد (SSO)، فإنه لا يفرض هذه القيود فعليًا خلال أول 120 ثانية على طلبات POST ذات المستوى الأعلى. ونتيجة لذلك، هناك نافذة مدتها دقيقتين قد يكون فيها المستخدمون عرضة للهجمات عبر المواقع.


من غير العملي إلى حد ما محاولة توقيت الهجوم ليقع خلال هذه النافذة القصيرة. من ناحية أخرى، إذا تمكنت من العثور على أداة على الموقع تمكنك من إجبار الضحية على إصدار ملف تعريف ارتباط جلسة جديد، فيمكنك تحديث ملف تعريف الارتباط الخاص به بشكل استباقي قبل متابعة الهجوم الرئيسي. على سبيل المثال، قد يؤدي إكمال تدفق تسجيل الدخول المستند إلى OAuth إلى جلسة جديدة في كل مرة لأن خدمة OAuth لا تعرف بالضرورة ما إذا كان المستخدم لا يزال مسجلاً الدخول إلى الموقع المستهدف.

لتشغيل تحديث ملفات تعريف الارتباط دون أن يضطر الضحية إلى تسجيل الدخول يدويًا مرة أخرى، تحتاج إلى استخدام Top-level navigation، والذي يضمن تضمين ملفات تعريف الارتباط المرتبطة بجلسة OAuth الحالية. يشكل هذا تحديًا إضافيًا لأنك ستحتاج بعد ذلك إلى إعادة توجيه المستخدم مرة أخرى إلى موقعك حتى تتمكن من شن هجوم CSRF.

وبدلاً من ذلك، يمكنك تشغيل تحديث ملف تعريف الارتباط من علامة تبويب جديدة حتى لا يغادر المتصفح الصفحة قبل أن تتمكن من تنفيذ الهجوم النهائي. هناك عقبة بسيطة في هذا الأسلوب وهي أن المتصفحات تحظر علامات التبويب المنبثقة ما لم يتم فتحها عبر تفاعل يدوي. على سبيل المثال، سيتم حظر النافذة المنبثقة التالية بواسطة المتصفح افتراضيًا:

```javascript
window.open('https://vulnerable-website.com/login/sso');
```

To get around this, you can wrap the statement in an `onclick` event handler as follows:

```javascript
window.onclick = () => { window.open('https://vulnerable-website.com/login/sso'); }
```

This way, the `window.open()` method is only invoked when the user clicks somewhere on the page.


### Bypassing Referer-based CSRF defenses

بصرف النظر عن الدفاعات التي تستخدم رموز CSRF، تستخدم بعض التطبيقات Header HTTP Referer لمحاولة الدفاع ضد هجمات CSRF، عادةً عن طريق التحقق من أن الطلب نشأ من المجال الخاص بالتطبيق. هذا النهج بشكل عام أقل فعالية وغالبًا ما يخضع للتجاوزات.

##### Referer header

The HTTP Referer header (which is inadvertently misspelled in the HTTP specification) is an optional request header that contains the URL of the web page that linked to the resource that is being requested. It is generally added automatically by browsers when a user triggers an HTTP request, including by clicking a link or submitting a form. Various methods exist that allow the linking page to withhold or modify the value of the `Referer` header. This is often done for privacy reasons.

تتحقق بعض التطبيقات من صحة ال header `Referer` عندما يكون موجودًا في الطلبات ولكنها تتخطى التحقق من الصحة إذا تم حذف ال header.

في هذه الحالة، يمكن للمهاجم صياغة استغلال CSRF الخاص به بطريقة تتسبب في قيام متصفح المستخدم الضحية بإسقاط ال header `Referer`  في الطلب الناتج. هناك طرق مختلفة لتحقيق ذلك، ولكن أسهلها هو استخدام علامة META داخل صفحة HTML التي تستضيف هجوم CSRF:

```HTML
<meta name="referrer" content="never">
```

##### Validation of Referer can be circumvented

Some applications validate the `Referer` header in a naive way that can be bypassed. For example, if the application validates that the domain in the `Referer` starts with the expected value, then the attacker can place this as a subdomain of their own domain: ```http://vulnerable-website.com.attacker-website.com/csrf-attack```

Likewise, if the application simply validates that the `Referer` contains its own domain name, then the attacker can place the required value elsewhere in the URL: `http://attacker-website.com/csrf-attack?vulnerable-website.com`

> [!NOTE]
> Although you may be able to identify this behavior using Burp, you will often find that this approach no longer works when you go to test your proof-of-concept in a browser. In an attempt to reduce the risk of sensitive data being leaked in this way, many browsers now strip the query string from the `Referer` header by default.
> 
> You can override this behavior by making sure that the response containing your exploit has the `Referrer-Policy: unsafe-url` header set (note that `Referrer` is spelled correctly in this case, just to make sure you're paying attention!). This ensures that the full URL will be sent, including the query string.

