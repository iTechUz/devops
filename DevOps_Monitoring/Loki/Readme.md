# Install Grafana Loki Log aggregation System

We now proceed to installing Loki with the steps below:

1.Go to Loki’s Release Page and choose the latest version of Loki

2.Navigate to Assets and download the Loki binary zip file to your server. During the release of this article, v2.0.0 is the latest.

```js
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url |  cut -d '"' -f 4 | grep loki-linux-amd64.zip | wget -i -
```
Install unzip
```js
# Ubuntu / Debian
sudo apt install unzip

# CentOS / Fedora / RHEL
sudo yum -y install unzip
```
3. Unzip the binary file to /usr/local/bin
```js
unzip loki-linux-amd64.zip
sudo mv loki-linux-amd64 /usr/local/bin/loki
```
Confirm installed version:
```
$ loki --version
loki, version 2.0.0 (branch: HEAD, revision: 6978ee5d)
  build user:       root@2645337e4e98
  build date:       2020-10-26T15:54:56Z
  go version:       go1.14.2
  platform:         linux/amd64
```
4. Create a YAML file for Loki under /usr/local/bin

Create required data directories:
```js 
sudo mkdir -p /data/loki
```

Create new configuration file.
```js
sudo vim /etc/loki-local-config.yaml
```
Add the following configuration to the file:
```
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
    - from: 2018-04-15
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /data/loki/index

  filesystem:
    directory: /data/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```

5. Create Loki service:

Create the following file under **/etc/systemd/**system to daemonize the Loki service:

```js
sudo tee /etc/systemd/system/loki.service<<EOF
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/loki -config.file /etc/loki-local-config.yaml

[Install]
WantedBy=multi-user.target
EOF
```
6. Reload system daemon then start Loki service:

```js
sudo systemctl daemon-reload
```
```js
sudo systemctl start loki.service
```
You can check and see if the service has started successfully:
```
$ sudo systemctl status loki
● loki.service - Loki service
     Loaded: loaded (/etc/systemd/system/loki.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2020-12-21 11:49:49 UTC; 2min 37s ago
   Main PID: 15223 (loki)
      Tasks: 7 (limit: 1137)
     Memory: 13.6M
     CGroup: /system.slice/loki.service
             └─15223 /usr/local/bin/loki -config.file /etc/loki-local-config.yaml

Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.330959628Z caller=table_manager.go:476 msg="creating table" table=index_2658
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.331092225Z caller=table_manager.go:476 msg="creating table" table=index_2549
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.331220486Z caller=table_manager.go:476 msg="creating table" table=index_2562
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.331347316Z caller=table_manager.go:476 msg="creating table" table=index_2615
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.331471475Z caller=table_manager.go:476 msg="creating table" table=index_2643
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.327278535Z caller=module_service.go:58 msg=initialising module=ring
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.331950866Z caller=module_service.go:58 msg=initialising module=distributor
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.332140208Z caller=module_service.go:58 msg=initialising module=ingester-querier
Dec 21 11:49:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:49:49.332342162Z caller=loki.go:227 msg="Loki started"
Dec 21 11:51:49 ubuntu loki[15223]: level=info ts=2020-12-21T11:51:49.311922692Z caller=table_manager.go:324 msg="synching tables" expected_tables=141
```
You can now access Loki metrics via **http://server-IP:3100/metrics***