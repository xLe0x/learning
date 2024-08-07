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

#### Bypassing CSRF token validation

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


#### Bypassing SameSite cookie restrictions

