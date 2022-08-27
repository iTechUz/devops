# Install Alertmanager
Alertmanager faqat bildirishnomalarni belgilangan so'nggi nuqtalarga yubormaydi. Shuningdek, u guruhlash, taqiqlash, jim turish va yuqori darajadagi mavjudlikni, shuningdek, Prometey bo'lmaganlar uchun ba'zi ilg'or mijoz xatti-harakatlarini qo'llab-quvvatlaydi.

Uni o'rnatishni ko'rib chiqamiz.
Prometey sozlamalarida bo'lgani kabi, biz tizimni ishga tushirish uchun alertmanager nomli yangi foydalanuvchi yaratmoqchimiz:
birinchi navbatda: 
```js
sudo useradd --no-create-home --shell /bin/false alertmanager
```
Fayllarni saqlash uchun katalog yaratamiz:
```js
sudo mkdir /etc/alertmanager
```
va alertmanager ni yuklab olamiz:

```js
cd /tmp/
```
```js
wget https://github.com/prometheus/alertmanager/\
releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz
```
va faylni chiqarib olamiz:
```js
tar -xvf alertmanager-0.16.1.linux-amd64.tar.gz
```
```js
cd alertmanager-0.16.1.linux-amd64/
```
```js
ls
```
Huddi prometheus da bo'lgani kabi binar fayllarni tayinlab olib ularga egalik huquqini beramiz:
```js
sudo mv alertmanager /usr/local/bin/
```
```js
sudo mv amtool /usr/local/bin/
```
```js
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
```
```js
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```
Jarayonni takrorlang, faqat bu safar standart alertmanager.yml faylini /etc/alertmanager ichiga o'tkazing.
```js
sudo mv alertmanager.yml /etc/alertmanager/
```
```js
sudo chown -R alertmanager:alertmanager /etc/alertmanager/
```
Keling, /etc/systemd/system katalogimizda alertmanager.service faylini yaratamiz:

```js
sudo nano /etc/systemd/system/alertmanager.service
```
```js
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target
[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager \
--config.file=/etc/alertmanager/alertmanager.yml
[Install]
WantedBy=multi-user.target
```
Saqlaymiz va chiqib ketamiz.

Alertmanagerni ishga tushirishdan oldin biz Prometey konfiguratsiyasini ham yangilashimiz kerak.
```js
sudo systemctl stop prometheus
```
Va /etc/prometheus/prometheus.yml faylini yangilang.

```js
sudo nano /etc/prometheus/prometheus.yml
```
```js
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
```
Systemd ni qayta yuklang va prometey va alertmanager xizmatlarini ishga tushiring:
```js
sudo systemctl daemon-reload
```
```js
sudo systemctl start prometheus
```
```js
sudo systemctl start alertmanager
```
va uni yangidan yuklanganiga ishonch hosil qiling.
```js
sudo systemctl enable alertmanager
```
Va buni ham huddi prometheus kabi serverni 9093 portini to'g'irlab uni brauzerda tekshirib ko`rishingiz mumkin.

*Masalan*  
```js
 http:// *serverIpManzili* :9090 
```