# Install Promtail 
Promtail - Loki-ga jurnallarni yig'ish va yuborish uchun mijoz. Promtail odatda jurnallarni kuzatish uchun zarur bo'lgan har bir mashinaga o'rnatiladi.

**Promtail-ni o'rnating**

Promtail o'rnatish bosqichlari Loki-ga juda o'xshash.

Promtail manba kodi Loki bilan bir xil omborda saqlanadi. GitHub'dan Promtail versiyasining so'nggi versiya tegini oling va o'zgaruvchiga versiya tegini belgilang.
```js
PROMTAIL_VERSION=$(curl -s "https://api.github.com/repos/grafana/loki/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
```
Promtail ikkilik va konfiguratsiya faylini saqlash uchun yangi katalog yarating.
```js
sudo mkdir /opt/promtail
```
Loki repozitori relizlar sahifasidan Promtail arxivini yuklab olish uchun quyidagi buyruqni bajaring.
```js
sudo wget -qO /opt/promtail/promtail.gz "https://github.com/grafana/loki/releases/download/v${PROMTAIL_VERSION}/promtail-linux-amd64.zip"
```
Ikkilik faylni arxivdan chiqarib oling:
```js
sudo gunzip /opt/promtail/promtail.gz
```
Chiqarilgan fayl uchun bajarish ruxsatini o'rnating:
```js
sudo chmod a+x /opt/promtail/promtail
```
**promtail** Biz katalogdagi buyruqqa ramziy havola yaratishimiz mumkin **/usr/local/bin**
```js
sudo ln -s /opt/promtail/promtail /usr/local/bin/promtail
```
Endi **promtail** barcha foydalanuvchilar uchun tizim bo'ylab buyruq sifatida foydalanish mumkin.

Promtail uchun konfiguratsiya faylini yuklab oling:
```js
sudo wget -qO /opt/promtail/promtail-local-config.yaml "https://raw.githubusercontent.com/grafana/loki/v${PROMTAIL_VERSION}/clients/cmd/promtail/promtail-local-config.yaml"
```
O'rnatish muvaffaqiyatli yakunlanganligini tekshirish uchun Promtail versiyasini tekshiring:
```js
promtail -version
```
**Promtail-ni xizmat sifatida ishga tushiring**

Biz systemd ni Promtailni xizmat sifatida ishlatish uchun sozlashimiz mumkin. Systemd birligi faylini yarating:
```js
sudo nano /etc/systemd/system/promtail.service
```
Quyidagi tarkibni qo'shing:
```js
[Unit]
Description=Promtail client for sending logs to Loki
After=network.target

[Service]
ExecStart=/opt/promtail/promtail -config.file=/opt/promtail/promtail-local-config.yaml
Restart=always
TimeoutStopSec=3

[Install]
WantedBy=multi-user.target
```
Promtail xizmatini ishga tushiring:
```js
sudo service promtail start
```
Promtail xizmati ishlayotganiga ishonch hosil qilish uchun quyidagi buyruqni bajarishingiz mumkin:

```js
sudo service promtail status
```
Shuningdek, siz xizmatni to'xtatishingiz yoki qayta ishga tushirishingiz mumkin:
```js
sudo service promtail stop
sudo service promtail restart
```
Promtailni yuklashda ishga tushirish uchun quyidagi buyruqni bajaring:
```js
sudo systemctl enable promtail
```
***Promtail-ni o'chirib tashlang***
Agar siz Promtail-ni butunlay olib tashlamoqchi bo'lsangiz, xizmatni to'xtating va tizim birligi faylini olib tashlang.
```js
sudo service promtail stop
sudo systemctl disable promtail
sudo rm -rf /etc/systemd/system/promtail.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
```
O'rnatish katalogini olib tashlang:
```js
sudo rm -rf /opt/promtail
```
Ramziy havolani olib tashlang:
```js
sudo rm -rf /usr/local/bin/promtail
```


