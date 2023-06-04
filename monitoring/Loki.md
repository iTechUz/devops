# Loki-ni Ubuntu 20.04-ga o'rnating
**Loki**- bu Grafana Labs tomonidan ishlab chiqilgan ochiq manbali jurnallarni yig'ish tizimi. Loki - bu jurnal ma'lumotlarini saqlash uchun optimallashtirilgan ma'lumotlar do'koni. Uni Grafana bilan jurnallarni ko'rish uchun ma'lumotlar manbai sifatida osongina birlashtirish mumkin.
# Loki-ni o'rnating
GitHub'dan Loki versiyasining so'nggi versiyasi tegini oling. Keyin o'zgaruvchiga versiya tegini tayinlang.
```js
LOKI_VERSION=$(curl -s "https://api.github.com/repos/grafana/loki/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')
```
Loki ikkilik va konfiguratsiya faylini saqlash uchun yangi katalog yarating.
```js
sudo mkdir /opt/loki
```
Arxivni Loki omborining nashrlar sahifasidan yuklab oling.
```js
sudo wget -qO /opt/loki/loki.gz "https://github.com/grafana/loki/releases/download/v${LOKI_VERSION}/loki-linux-amd64.zip"
```
Ikkilik faylni arxivdan chiqarib oling:
```js
sudo gunzip /opt/loki/loki.gz
```
Fayl uchun bajarish ruxsatini o'rnating:
```js
sudo chmod a+x /opt/loki/loki
```
Katalogda /usr/local/binbiz buyruqqa ramziy havola yaratishimiz mumkin loki:
```js
sudo ln -s /opt/loki/loki /usr/local/bin/loki
```
Endi loki buyruq barcha foydalanuvchilar uchun butun tizim buyrug'i sifatida mavjud

**Loki** uchun konfiguratsiya faylini yuklab oling:
```js
sudo wget -qO /opt/loki/loki-local-config.yaml "https://raw.githubusercontent.com/grafana/loki/v${LOKI_VERSION}/cmd/loki/loki-local-config.yaml"
```
O'rnatishni tekshirish uchun biz Loki versiyasini tekshirishimiz mumkin:
```js
loki -version
```

**Loki-ni xizmat sifatida ishga tushiring**

Biz Loki-ni xizmat sifatida ishlatish uchun systemd-ni sozlashimiz mumkin. Systemd birligi faylini yarating:
```js 
sudo nano /etc/systemd/system/loki.service
```
Faylga quyidagi tarkibni qo'shing:
```js
[Unit]
Description=Loki log aggregation system
After=network.target

[Service]
ExecStart=/opt/loki/loki -config.file=/opt/loki/loki-local-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```
Loki xizmatini ishga tushiring:
```js
sudo service loki start
```
Loki xizmati ishlayotganiga ishonch hosil qilish uchun siz quyidagi buyruqdan foydalanishingiz mumkin:
```js
sudo service loki status
```
Shuningdek, siz xizmatni to'xtatishingiz yoki qayta ishga tushirishingiz mumkin:
```js
sudo service loki stop
sudo service loki restart
```
Loki-ni ishga tushirishni yoqish uchun quyidagi buyruqni bajaring:
```js
sudo systemctl enable loki
```
**Loki-ni o'chirib tashlang**

Agar siz Loki-ni butunlay olib tashlashga qaror qilsangiz, xizmatni to'xtating va tizim birligi faylini olib tashlang.
```js
sudo service loki stop
sudo systemctl disable loki
sudo rm -rf /etc/systemd/system/loki.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
```
O'rnatish katalogini o'chiring:
```js
sudo rm -rf /opt/loki
```
Ramziy havolani olib tashlang:
```js
sudo rm -rf /usr/local/bin/loki
```
