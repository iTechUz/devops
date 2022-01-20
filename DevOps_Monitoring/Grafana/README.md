# Install Grafana
Grafana ko'p platformali ochiq manbali tahlil va interaktiv vizualizatsiya veb-ilovasidir.    

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
