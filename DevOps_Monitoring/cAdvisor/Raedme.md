# Using cAdvisor to Monitor Containers
We’ve taken a look at quite a few common metrics and how to manipulate them in the past series of
lessons. But thus far, we’ve just been monitoring metrics on the server itself — what about the containers
we host on our servers?

For container monitoring, we’re going to bring in yet another tool, although this one will be a quick and
simple deploy: cAdvisor is a tool released by Google that natively monitors Docker containers and can be
used with a number of other container platforms out of the box. We barely even have to do any work to
deploy it! All we have to do is run:
```js
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8000:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```
Now, if we take a peak at our list of containers, we can see that we have a second container up and
running, called cadvisor :
```js
docker ps
```
The above command also automatically mapped the cAdvisor container to port 8000 on our host, so
we can view it by navigating to the lab server URL and accessing the appropriate port. This is also
where Prometheus will check for these container metrics. Let’s actually go ahead and add cAdvisor to
Prometheus now:
```js
$ sudo nano /etc/prometheus/prometheus.yml
```
```js
- job_name: 'cadvisor'
  static_configs:
  - targets: ['localhost:8000']
```
Next, restart Prometheus:
```js
sudo systemctl restart prometheus
```
Now, when we move to our Prometheus web UI, we can search for the prefix container and see a number
of options! Another thing you might notice is that many of these metrics are the same or similar to the
ones we’ve already discussed. We have CPU data, although with cAdvisor the data is paired down further
and can be separated by system and user; file system data that uses the straight-forward inodes name
instead of files to show us how many inodes are used or free; a selection of basic memory information,
mostly related to errors and usage; and inbound and outbound networking data.

We can also make sure we’re just getting data about the containers we want by using the name label. So if
we want to see the metrics specific to the memory usage of our application, we can search the expression
editor for:
```js
container_memory_usage_bytes{name="ft-app"}
```
With the name value simply taken from our container name itself that we provided Docker.