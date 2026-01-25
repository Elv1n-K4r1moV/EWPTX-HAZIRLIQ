### AUTH Tipləri:
1) *Password-based* – Universitet portalına hər dəfə username və şifrə yazaraq daxil olursan.

2) *MFA* – Bank hesabına girişdə şifrəni yazırsan, sonra telefonuna gələn SMS kodu təsdiqləyirsən.

3) *Token-based* – Mobil tətbiqdə login edirsən, server sənə token verir və tətbiq hər sorğuda onu göndərir.

4) *Session-based* – Onlayn mağazaya login olandan sonra səhifələri dəyişsən də çıxış olmursan.

5) *Certificate-based* – İş yerinin daxili şəbəkəsinə yalnız sənə verilmiş sertifikatla qoşula bilirsən.

6) *Biometric* – Telefon kilidini üz tanıma ilə açırsan.

7) *SSO* – İş sisteminə bir dəfə login olub e-mail, HR və daxili portalı ayrıca girişsiz açırsan.

<img width="798" height="572" alt="image" src="https://github.com/user-attachments/assets/e758417b-6823-42ea-b4e9-6e78bd0c8246" />

#### CVE‑2020‑15906 – Tiki Wiki CMS‑də admin hesabı çox yalnış loginlə kilidləndikdən sonra boş şifrə ilə autentifikasiyanın ötürülməsinə imkan verən auth bypass zəifliyidir.

### https://www.exploit-db.com/exploits/39167 - Bu exploitdə auth bypass ona görə baş verir ki, tətbiq login olub‑olmadığını server‑side yoxlamaq əvəzinə yalnız $_COOKIE['LoggedIn'] dəyişəninə inanır; hücumçu request‑ə Cookie: LoggedIn=yes əlavə etməklə heç bir username/password olmadan admin panelə daxil olur.
