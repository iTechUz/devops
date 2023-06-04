# Ubuntu 20.04 da Grafana qanday o'rnatiladi

' Grafana - bu bir nechta platformalarda ishlaydigan bepul monitoring va ma'lumotlarni vizualizatsiya qilish dasturi. Bu sizning ma'lumotlaringizni grafik shaklda ko'rsatish orqali chuqur tahlillarni taqdim etadi va o'zaro faoliyat platformadir. Qayta foydalanish mumkin bo'lgan dinamik boshqaruv panellarini yaratish, maxsus so'rovlar orqali ko'rsatkichlarni o'rganish, doimiy ravishda baholanadigan muhim ko'rsatkichlar uchun ogohlantirish qoidalarini o'rnatish va tegishli tomonlarni har qanday o'zgarishlar haqida xabardor qilish va jamoa a'zolari bilan hamkorlik qilish mumkin. ichki almashish orqali. Bundan tashqari, u Graphite, Prometheus, Elasticsearch va InfluxDB kabi boshqa tizimlarga kirish imkonini beradi. Ushbu qo'llanma sizga Grafana-ni Ubuntu 18.04 yoki 20.04 bilan ishlaydigan serverga o'rnatish jarayonini o'rgatadi.'

The system packages are required to be updated.

```js 
apt update
````
Tizim uchun kerakli paketlarni o'rnating.
```js
apt-get install -y gnupg2 curl software-properties-common
```
Grafana GPG kalitini qo'shing.
```js
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
```
Grafana omborini kompyuteringizga o'rnating.
```js
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```
Tizim paketlarini yangilang.
```js 
apt update
```
Endi siz o'rnatishni davom ettirishingiz mumkin:
```js
sudo apt install grafana
```
Grafana o'rnatilgandan so'ng, systemctl Grafana serverini ishga tushirish uchun foydalaning
```js
sudo systemctl start grafana-server
```
Keyin xizmat holatini tekshirish orqali Grafana ishlayotganligini tekshiring:
```js 
sudo systemctl status grafana-server
```
Siz shunga o'xshash natijani olasiz:
```js
Output
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-05-21 08:08:10 UTC; 4s ago
     Docs: http://docs.grafana.org
 Main PID: 15982 (grafana-server)
    Tasks: 7 (limit: 1137)
```
Ushbu chiqishda Grafana jarayoni, jumladan uning holati, asosiy jarayon identifikatori (PID) va boshqalar haqida ma'lumotlar mavjud. active (running)jarayonning to‘g‘ri ketayotganini ko‘rsatadi.

Va nihoyat, Grafana-ni yuklashda avtomatik ravishda ishga tushirish uchun xizmatni yoqing:
```js
sudo systemctl enable grafana-server
```
Siz quyidagi chiqishni olasiz:
```js
Output
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /usr/lib/systemd/system/grafana-server.service.
```


