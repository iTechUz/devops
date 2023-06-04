# Installing the Node Exporter
To install the Node Exporter, we’ll be working the same way we did for Prometheus and Alertmanager: By
downloading the needed files, adding a system user, and creating a systemd service file so we can start
and stop the exporter as needed.

Let’s first create that system user:
```js
sudo useradd --no-create-home --shell /bin/false node_exporter
```
Then download and extract the .tar.gz file from Prometheus’s download page:
```js
cd /tmp/
```
```js
wget https://github.com/prometheus/node_exporter\
/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
```
```js
tar -xvf node_exporter-0.17.0.linux-amd64.tar.gz 
```
Move into the new directory:
```js
cd node_exporter-0.17.0.linux-amd64/
```
Here we have only one file we need to work with: the node_exporter binary. Let’s move it to the
**/usr/local/bin** directory:
```js
sudo mv node_exporter /usr/local/bin/
```
And set the appropriate ownership:
```js
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
Next, we need to create the systemd file for the service:
```js
sudo nano /etc/systemd/system/node_exporter.service
```
```js
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
```
We can now reload systemd and start the **node_exporter** :
```js
sudo systemctl daemon-reload
```
```js
sudo systemctl start node_exporter
```
Finally, we can add the node_exporter to our endpoints so Prometheus can scrape the data:
```js
sudo nano /etc/prometheus/prometheus.yml
```
```js
- job_name: 'node_exporter'
  static_configs:
  - targets: ['localhost:9100']
```
```js
sudo systemctl restart prometheus
```

Now, if we return to Prometheus and refresh, when we try to search for a metric, various system metrics
are now available. To see for yourself, search memory and view the node_memory_MemFree_bytes metric,
which provides data on the amount of free memory we have. If we switch the time series, we can see
some changes, but what we really want to do is watch some bigger spikes in the graph; for this we can
use a tool called **stress** :
### **Install Stress**
```js
sudo apt-get install stress
```
And force some load on our RAM:
```js
stress -m 2
```
Wait for a minute, then refresh the Prometheus graph; there will be a noticeable difference in activity.