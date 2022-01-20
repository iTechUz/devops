**Projectni manitoring qilishdan oldin bizga birinchi navbatda muhit kerak bo'ladi va buni biz yaratib olamiz.**

# Install Docker

1.Birinchi navbatda serverimizga dockerni o'rnatib olamiz va buyruqlarni ketma -ket tarzda bajaring
```js
sudo apt-get install apt-transport-https ca-certificates \
```
```js
curl gnupg2 software-properties-common`

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository "deb [arch=amd64] \

https://download.docker.com/linux/ubuntu bionic stable"`
```
```js
sudo apt-get install docker-ce
```
2.Docker sudo -less qo'shing:
```js
sudo usermod -aG docker USERNAME
```
Keyin seansingizni yangilang.

3.Install Node.js and npm
```js
curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh
```
```js
sudo apt-get install nodejs
```
```js
sudo apt-get install build-essential
```
endi biz muhitga ega bo'ldik. Bizning dasturimiz container sifatida ishlaydi.
# Install Prometheus

Biz endi monitoring qilishimiz uchun Prometheus ni tanladik chunki.............................................................................................................................................................................

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
Endi Faylni saqlab chiqib ketasiz va systtemd ni qayta yuklaymiz:
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
```
```js
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
# Install Grafana
Grafana ko'p platformali ochiq manbali tahlil va interaktiv vizualizatsiya veb-ilovasidir.
..................
.
.
.
.


.
.

.
.

.

.
.

.

.
.

Grafana sozlamalari ham Prometheus va Alertmanager ga qaraganda ancha sodda.

Grafana ni o'rnatish uchun biz qilishimiz kerak bo'lgan zarur shartlar to'plamini qo'shishimiz kerak:
```js
sudo apt-get install libfontconfig
```
Grafana faylini yuklab olamiz:
```js
wget https://dl.grafana.com/oss/release/grafana_5.4.3_amd64.deb
```
```js
sudo dpkg -i grafana_5.4.3_amd64.deb
```
Grafana serveri avtomatik ravishda ishga tushiriladi, shuning uchun biz qilishimiz kerak bo'lgan narsa uning qayta ishga tushirilganiga ishonch hosil qilishdir:
```js
sudo systemctl enable grafana-server
```
Endi esa serverimizni 3000 portini sozlab uni tekshirib ko'ramiz brauzerimizda: 
*Masalan*
```js
 http:// *serverIpManzili* :3000 
```
va unga kirayotganimizda bizdan username va parol so`raladi ularni ikkalasi ham admin, parolni o'zgartirishni so'raydi va parolni o'zingizga mos qilib o'zgartirib oling.


# Installing the Node Exporter
# Install Stress
#
#
