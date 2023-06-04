Prometey - bu samarali va ochiq manbali kompyuter monitoringi tizimi. U kompyuter hodisalari, dasturiy ta'minot va xizmatlarni kuzatadi. Prometey ilova ko'rsatkichlarini vaqt seriyali ma'lumotlar bazasida saqlashga imkon beradi, chunki u o'zining saqlash tizimiga ega. Ushbu ilova Soundcloud-da ishlab chiqilgan va ishlatilgan. Biroq, keyinchalik turli tashkilotlar undan monitoring maqsadida foydalanganlar. U Go dasturlash tilida yozilgan.

Prometeyda "qirqish" deb nomlangan ma'lumot yig'ish usuli mavjud. Biroq, "eksportchi" ma'lumotlarni yig'ish vositasidir. Prometey o'z foydalanuvchilarini qiziqtirish uchun ma'lumotlarni qayta ishlash, veb-ga asoslangan foydalanuvchi interfeysi va ogohlantirishlarni boshqarish tizimi kabi turli afzalliklarni taqdim etadi.

Tizimning yangilangan paketlarini olish uchun tizimning paketlar omborini yangilashingiz kerak. Tizim paketlarini yangilash uchun terminalda quyidagi buyruqni bajaring:
```js
$ sudo apt update
```
**2-qadam**: Prometey tizimi guruhini yarating
Birinchidan, prometeyni Ubuntu 22.04 da o'rnatishdan oldin foydalanuvchi va Prometey tizimi guruhini yarating. Prometey tizim guruhini yaratish uchun terminalda berilgan buyruqni bajaring:
```js
$ sudo groupadd --system prometheus
```
3-qadam: Guruhni tayinlash uchun foydalanuvchi qo'shing
Guruh yaratilganda, Prometey foydalanuvchisini qo'shing va unga yaratilgan guruhni tayinlang. Foydalanuvchi qo'shish va yaratilgan guruhni tayinlash uchun Linux terminalida quyidagi buyruqni kiriting va ishga tushiring
```js
$ sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
4-qadam: katalog yarating
Prometey o'zining saqlash tizimiga ega. Biroq, u katalog yordamida ma'lumotlarni saqlaydi.

Shuning uchun, katalog yaratish uchun quyidagi buyruqni bajaring:
```js
$ sudo mkdir /etc/prometheus
```
Bu Prometeyning asosiy katalogidir. Biroq, u ba'zi pastki katalogga ega bo'ladi. Pastki katalog yaratish uchun quyidagi buyruqni bajaring:
```js
$ sudo mkdir /var/lib/prometheus
```
Qadam: 5 Prometeyni Ubuntu 22.04 da yuklab oling
Prometeyning so'nggi versiyasini tekshirish va yuklab olish uchun quyidagi havola orqali github omboriga o'ting.
```js
https://prometheus.io/download/
```
image.png
6-qadam: Faylni chiqarib oling
Fayl yuklab olinganda, uni quyidagi buyruq yordamida chiqarib oling:
```js
$ tar xvf prometheus*.tar.gz
```
7-qadam: katalogga o'ting
Fayl chiqarilganda, quyidagi buyruq yordamida katalogga o'ting:
```js
$ cd prometheus*/
```
Va katalogni o'zgartirish uchun ikkilik fayllarni mahalliy papkaga ko'chiring. Fayllarni /usr/local/bin ga ko'chirish uchun quyidagi buyruqdan foydalaning:
```js
$ sudo mv prometheus promtool /usr/local/bin/```
```
Ubuntu 22.04 da Prometey o'rnatilgan versiyasini tasdiqlash uchun quyidagi buyruqni bajaring:
```js
$ prometheus --version
```
8-qadam: Prometey konfiguratsiya shablonini ko'chiring
O'rnatish muvaffaqiyatli tugallangach, konfiguratsiya shablonini Prometey konfiguratsiyasi uchun /etc katalogiga o'tkazing. Shablonni ko'chirish uchun quyidagi buyruqni bajaring:
```js
$ sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```
Bundan tashqari, uy katalogiga qaytish uchun konsol kutubxonalarini /etc katalogiga ko'chirishingiz kerak. Konsol kutubxonalarini ko'chirish uchun quyidagi buyruqni kiriting va ishga tushiring:
```js
$ sudo mv consoles/ console_libraries/ /etc/prometheus/
$ cd $HOME
```
Ubuntu-da Prometeyni qanday sozlash mumkin
Faylni yaratish va tahrirlash uchun Ubuntu 22.04 da Prometey konfiguratsiyasi uchun terminalda quyidagi buyruqni bajaring:
```js
$ sudo vim /etc/prometheus/prometheus.yml
```
Agar siz prometeyni avtomatik ravishda ishga tushirishni istasangiz, prometheus systemd xizmat faylini yaratishingiz kerak. Terminalda quyidagi buyruqni kiriting va bajaring:
```js
$ sudo nano /etc/systemd/system/prometheus.service
```
Va quyidagilarni joylashtiring:
```js
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
**Prometeyni qanday qayta yuklash va yoqish mumkin**

Tizim qayta ishga tushirilgandan so'ng prometeyni avtomatik ravishda ishga tushirishni istasangiz, prometeyni yoqishingiz kerak. Ubuntu 22.04 da prometeyni qayta yuklash va yoqish uchun terminalda quyidagi buyruqlarni kiriting va ishga tushiring:
```js
$ sudo systemctl daemon-reload 
```
Prometey xizmati muvaffaqiyatli qayta yuklandi.
```js
$ sudo systemctl enable --now prometheus
```
**Prometey portiga qanday kirish mumkin**
9090 porti Prometeyning standart ishlaydigan portidir. Xavfsizlik devorida 9090 portiga ruxsat berish uchun quyidagi buyruqni bajaring:
```js
$ sudo ufw allow 9090/tcp
```
**Xulosa**

Prometeyni Ubuntu 22.04 da rasmiy tizim omboridan foydalanib o'rnatish mumkin. Prometey - bu samarali va ochiq manbali kompyuter monitoringi tizimi. U kompyuter hodisalari, dasturiy ta'minot va xizmatlarni kuzatadi. U ilovalarning ko'rsatkichlarini vaqt seriyali ma'lumotlar bazasida saqlaydi, chunki u o'zining ma'lumotlarni saqlash tizimiga ega. Ushbu maqola Prometeyni Ubuntu 22.04 da o'rnatish bo'yicha to'liq qo'llanmani taqdim etdi. Biz Ubuntu 22.04 da Prometey konfiguratsiyasini ham tasvirlab berdik.