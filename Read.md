### Session Hijacking 

Bir forum saytında şərh hissəsi var və input filtrasiya olunmur. Hücumçu şərhə zərərli JavaScript yerləşdirir; sən səhifəni açanda script brauzerində işləyir və session cookie-ni oğurlayıb hücumçuya göndərir. 

Hücumçu həmin cookie ilə sənin hesabına şifrəsiz daxil olur (session hijacking).

Qarşısı (qısa):

Input validation + output encoding

HttpOnly cookie

CSP (Content Security Policy)
