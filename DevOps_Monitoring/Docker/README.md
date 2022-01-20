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
