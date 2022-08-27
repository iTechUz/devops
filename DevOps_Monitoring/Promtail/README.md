```js
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url |  cut -d '"' -f 4 | grep promtail-linux-amd64.zip | wget -i -
```
```js
unzip promtail-linux-amd64.zip
```
```js
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```
```js
promtail --version
```
```js
sudo vim /etc/promtail-local-config.yaml
```
```js
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /data/loki/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
```
```js
sudo tee /etc/systemd/system/promtail.service<<EOF
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail-local-config.yaml

[Install]
WantedBy=multi-user.target
EOF
```
```js
sudo systemctl daemon-reload
```
```js
sudo systemctl start promtail.service
```
```js 
sudo systemctl status promtail.service
```

- promtail.service - Promtail service
     Loaded: loaded (/etc/systemd/system/promtail.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2020-12-21 11:57:41 UTC; 3s ago
   Main PID: 15381 (promtail)
      Tasks: 6 (limit: 1137)
     Memory: 8.8M
     CGroup: /system.slice/promtail.service
             └─15381 /usr/local/bin/promtail -config.file /etc/promtail-local-config.yaml

Dec 21 11:57:41 ubuntu systemd[1]: Started Promtail service.
Dec 21 11:57:41 ubuntu promtail[15381]: level=info ts=2020-12-21T11:57:41.911186079Z caller=server.go:225 http=[::]:9080 grpc=[::]:35499 msg="server listening on>
Dec 21 11:57:41 ubuntu promtail[15381]: level=info ts=2020-12-21T11:57:41.911859429Z caller=main.go:108 msg="Starting Promtail" version="(version=2.0.0, branch=H>

Agar kerak bo'lsa ***prometheus*** ga qo'shib qo'ysak bo'ladi

```js
sudo nano /etc/prometheus/prometheus.yml
```
```js
  - job_name: 'promtail'
    static_configs:
    - targets: ['localhost:9080']
```
```js
sudo systemctl restart prometheus
```
```js
sudo systemctl daemon-reload
```