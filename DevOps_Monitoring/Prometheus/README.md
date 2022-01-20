# Install Prometheus

Biz endi monitoring qilishimiz uchun Prometheus ni tanladik. Prometheus - bu hodisalarni kuzatish va ogohlantirish uchun ishlatiladigan bepul dastur . U moslashuvchan soʻrovlar va real vaqtda ogohlantirish bilan HTTP tortish modelidan foydalangan holda qurilgan vaqt seriyasidagi maʼlumotlar bazasida (yuqori oʻlchamlilikka imkon beruvchi) real vaqt koʻrsatkichlarini qayd qiladi.

Avvaliga biz yangi foydalanuvchi yaratib olamiz
```js
sudo useradd --no-create-home --shell /bin/false prometheus
```
Keyin yangi kataloglar
```js
sudo mkdir /etc/prometheus
```
```js
sudo mkdir /var/lib/prometheus
```

endi prometheus foydalanuvchisini o'rnatamiz.
```js
sudo chown prometheus:prometheus /var/lib/prometheus
```
Endi prometheus yuklab olamiz.
```js
cd /tmp/
```
```js
wget https://github.com/prometheus/prometheus/\
releases/download/v2.7.1/prometheus-2.7.1.linux-amd64.tar.gz
```

Fayllarni chiqarib olamiz.
```js
tar -xvf prometheus-2.7.1.linux-amd64.tar.gz
```

Va chiqarilgan faylga o'tib uni ichini ko'ramiz.
```js
cd prometheus-2.7.1.linux-amd64/
```
```js
ls
```
configuration va directories fayllarini */etc/* jildiga ko'chiring.
```js
sudo mv console* /etc/prometheus
```
```js
sudo mv prometheus.yml /etc/prometheus
```
Endi ushbu fayllar prometheus niki ekanligiga ishonch hozil qilishimiz kerak.

```js
sudo chown -R prometheus:prometheus /etc/prometheus
```
Jarayonni takrorlaymiz, lekin bu safar ikkilik fayllarimizni **/usr/local/bin/** katalogiga o'tkazamiz:
```js
sudo mv prometheus /usr/local/bin/
```
```js
sudo mv promtool /usr/local/bin/
```
```js
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```
```js
sudo chown prometheus:prometheus /usr/local/bin/promtool
```
Ishga tushirish, to'xtatish-ni boshqarish uchun systemctl-dan foydalanishga ruxsat beruvchi systemd xizmat faylini o'rnatishimiz kerak.
```js
sudo nano /etc/systemd/system/prometheus.service
```
Endi quyidagini faylga nusxalashimiz kerak:
```js
[Unit]
Description=Prometheus
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
Endi Faylni saqlab chiqib ketasiz va systemd ni qayta yuklaymiz:
```js
sudo systemctl daemon-reload
```
Endi esa prometheusni ishga tushirishimiz kerak.
```js
sudo systemctl start prometheus
```
va uni yangidan yuklanganiga ishonch hosil qiling.
```js
sudo systemctl enable prometheus
```
Va nihoyat prometheus tayyor uni ishlatishimiz uchun serverimizni 9090 portini to'g'irlab, brauzer orqali ko'rishimiz mumkin.

  *Masalan*
  ```js
http:// *serverIpManzili* :9090
  ```