# Service Discovery

Up until this point in the course, we have been using a lot of words for the same concept: Client, endpoint,
service, target… All of these are just simple words for something that our monitoring system is monitoring.
And we can see what targets we’re monitoring on Prometheus by going to our Prometheus web UI, and
navigating to the Targets page, located under the Status menu option. Right now, we’re just monitoring
Prometheus itself, but we’re obviously going to want to add more.
Here is where service discovery comes in. Service discovery is simply the way Prometheus detects our
metrics pages for any of our desired targets. If you remember a few lessons ago, we actually looked
directly at the metrics page for our Grafana setup. Well, if we wanted Prometheus to discover that page
itself, we would have to add that information to our Prometheus configuration.

Go ahead and open up **/etc/prometheus/prometheus.yml** now:

```js
sudo nano /etc/prometheus/prometheus.yml
```

At the bottom of the configuration file, we have a section for scrape_configs , that currently only has
information about the prometheus monitoring job:
```js
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```

To add a job for Grafana, we only have to mimic this for our Grafana setup:

```js
- job_name: 'grafana'
  static_configs:
  - targets: ['localhost:3000']
```
While we’re here, let’s also add Alertmanager:
```js
- job_name: 'alertmanager'
  static_configs:
  - targets: ['localhost:9093']
```
Save and exit. Now, to get Prometheus to discover these new targets, all we have to do is restart our
Prometheus service:
```js
sudo systemctl restart promtheus
```
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
tar -xvf /releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
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
$ sudo nano /etc/systemd/system/node_exporter.service
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
$ sudo nano /etc/prometheus/prometheus.yml
```
```js
- job_name: 'node_exporter'
  static_configs:
  - targets: ['localhost:9100']
```
```js
$ sudo systemctl restart prometheus
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
# Using the prom-client
Up until this point, we’ve been dealing with metrics that we, ourselves, did not have to write or add. But
when we write custom applications, we want to be able to monitor those, too. And for custom applications, there’s not going to be a Node Exporter or cAdvisor for us to download. Instead, we’re going to
have to write these instrumentations ourselves.

This is where Prometheus client libraries come in. Official libraries exist for Go, Java/Scala, Python, and
Ruby, although third-party clients exist for a number of other languages, including Bash, C++, Lisp, Elixer,
Erlang, Haskell, Lua, .NET, Node.js, Perl, PHP, and Rust. Since our application is written in Node.js, it’s safe
to guess which one we’ll be using.

Before we begin working on our application, however, I do want to note that anything we learn in this
section can be adapted for any other language. The concepts behind using a Prometheus client library are
the same regardless of language. Although our application is written in Node.js, I personally got familiar
with the concepts by looking at Python tutorials, which were better-written at the time. The details of the
language might change, but where we include our Prometheus counters and tracking in our application
won’t change.

So, let’s go ahead and get started. Move into the forethought directory from the cloud_user ’s home
directory:
```js
cd forethought
```
If you remember way back to our first real lesson, this is where our application is stored. Normally, while
we begin to write our instrumentations, we would be working on a development server. But since we’re
limited with our test environment, we’re going to make our changes straight to this directory itself and
testing from here. But before we can do even that, we need to grab the Prometheus client for Node.js:
```js
npm install prom-client --save
```
With that finished, we can start by adding it to our application. We’ll be working exclusively with the
index.js file. Go ahead and open this file:
```js
sudo nano index.js
```
We now want to make sure our application has access to the Prometheus client, so we can begin adding
metrics. To do this, we need to include the client in our code:
```js
const prom = require('prom-client');
```
This should be added after our existing variable list ( ***var app = express();*** ).

The Prometheus client also offers us the option to collect some default metrics — things like active re-
quests, start time, and memory usage by the application itself. To do this, we just have to add the follow-
ing to our code:
```js
const collectDefaultMetrics = prom.collectDefaultMetrics;
collectDefaultMetrics({ prefix: 'forethought' });
```
Where prom references the constant variable we just defined. The prefix setting just defines the prefix
for the metric name.

However, we currently have no way to see these default metrics. After all, up until this point we’ve only
seen our metrics after they’ve been revealed on a /metrics endpoint. But our application doesn’t have
one of these.

Lucky for us, our application already has the ability to create a simple web server. If you look at our
initial set of variables, we include Express, a Node.js/JavaScript framework for deploying simple websites.
It’s how we have the Forethought application up and running despite having neither Apache or Nginx
installed (seriously, go look: There’s no HTTP server on our host).

If we look further into our code, we can see that we render our website in only a few simple lines, here:
```js
app.get("/", function (req, res) {
res.render("index", { task: task, complete: complete });
});
```
We can do this same thing for our /metrics endpoint. So we can establish that we want a page called
*/metrics* by creating a callback function for any related requests and responses. This is our ***route definition***:
```js
app.get('/metrics', function (req, res) {
});
```
And then within that definition, we can provide the code related to *prom-client* that will provide the
response objects that will generate our metrics:
```js
app.get('/metrics', function (req, res) {
  res.set('Content-Type', prom.register.contentType);
  res.end(prom.register.metrics());
});
```
Save and exit the file. Now we can go ahead and test our work by running:
```js
node index.js
```
This brings up our application on port 8080. To see that our */metrics* page is available and we’ve started
generating the default metrics, navigate to *YOURLABSERVER:8080/metrics* .

# Counters
In our “Infrastructure Monitoring” section, we frequently used the terms “counters” and “gauges” to dis-
cern how the metrics are displaying our data and what that data meant. But now that we’re the ones
writing these metrics, we want to be sure we really understand what we’re having our metrics track —
and how.

Counters are generally considered the most commonly used instrumentation method. We saw a counter
used within every one of our infrastructure monitoring domains (except load), and that’s because when
paired with a time series, we can calculate a number of trends and additional metrics based on how often
or how little the number is increasing.

And think about how many things within an application we can use a counter for: Connections, any
requests made that are specific to the application, errors thrown, and more. In our case, we’re going
to add a counter to our application that tracks how many total to-do list items have been logged in our
application — so every time we click “Add”, the number of this metric would increase by one, allowing us
to track how active our application is at any given time.

Let’s go ahead and open up our index.js file again:
```js
cd forethought
```
```js
sudo nano index.js
```
When we define a new metric with our Prometheus client, we need to first import the function and provide
it with the metric name and the “help” line explaining that metric’s purpose. For our Node.js application,
this means that every time we add a metric, we’ll be following this structure:
```js
const METRICNAME = new prom.METRICTYPE
  name: 'NAME_OF_METRIC',
  help: 'HELP INFORMATION'
});
```
To import a counter for our “amount of to-dos” metric, we’d add the following:
```js
// Prometheus metric definitions
const todocounter = new prom.Counter({
  name: 'forethought_number_of_todos_total',
  help: 'The number of items added to the to-do list, total'
});
```
Add this near the top of the file, after our variable definitions where we import in the various libraries we
use. I’ll be storing all my metrics in this one section, so I added a comment at the top to clarify.

But the Prometheus client isn’t going to inherently know what is in our application we’re counting — this
is only where we define our counter. To actually have our app start counting up with each new to-do, we
need to locate the part of our app that lets us add these to-do tasks:
```js
// add a task
app.post("/addtask", function(req, res) {
  var newTask = req.body.newtask;
  task.push(newTask);
  res.redirect("/");
});
```
And call our counter in the middle of the function:
```js
// add a task
app.post("/addtask", function(req, res) {
  var newTask = req.body.newtask;
  task.push(newTask);
  res.redirect("/");
  todocounter.inc();
});
```
We use todocounter to call the prom.Counter function we just created, with .inc() signaling that we’re
increasing the count. By default, this increases by one, but we can change this by adding the number we
want the count to increase in by the parenthesis (i.e., .inc(2) would increase by two).

Now let’s see this in action by saving and exiting the file, and running:
```js
node index.js
```
Access the application (make sure you’re on port 8080 and not the live application!) and add some to-dos
to increase the counter, then check the /metrics endpoint. Our new task counter has been keeping track
of how many to-dos we’ve added!
# Gauges
The other metric we’ve been exposed to up until this point has been gauges — these keep track of the
state of our system the moment the metric is taken, meaning it isn’t a consistent count up like our counters, but a number that can be increased or decreased as the state of our metric changes. The number the
gauge tracks can be anything measurable — from how much memory is being used, as we saw before,
to how many concurrent connections there are, to how long a request takes to process.

For our demo application, we want to track just how many unfinished to-dos we have: So it keeps track
of any tasks added.

We can add a gauge-based metric to our application in a way that is similar to how we added a counter.
We first want to define the metric:
```js
const todogauge = new prom.Gauge ({
  name: 'forethought_current_todos',
  help: 'Amount of incomplete tasks'
});
```
Then we can call the metric in our code. However, unlike in our previous lesson, we have to make sure
we’re including both a decrease and an increase:
```js
// add a task
app.post("/addtask", function(req, res) {
  var newTask = req.body.newtask;
  task.push(newTask);
  res.redirect("/");
  todocounter.inc();
  todogauge.inc();
});
// remove a task
app.post("/removetask", function(req, res) {
  var completeTask = req.body.check;
  if (typeof completeTask === "string") {
    complete.push(completeTask);
    task.splice(task.indexOf(completeTask), 1);
  }
else if (typeof completeTask === "object") {
  for (var i = 0; i < completeTask.length; i++) {
    complete.push(completeTask[i]);
    task.splice(task.indexOf(completeTask[i]), 1);
    todogauge.dec();
  }
}
```
In this change, the todogauge.inc() functions work the same as the todocounter function.
todogauge.dec() just decreases the count instead of increases.

Save and exit, then run node index.js . Review the application in your web browser.

But gauges also offer some more advanced utilities, as well. For example, beyond just measuring the
amount of tasks we’re adding and completing, we can keep track of how long the requests take with the
use of the .startTimer() function for gauges. However, we’re not going to be using a gauge to measure
our response time. Instead, we want to use a histogram.
# Histograms and Summaries
Both histograms and summaries are more complex metric types. While all our metrics up until this point
have been storing data across a single time series, summaries and histograms work across multiple time
series, keeping track of both sample observations — like how long it took to make a request — and the
sum of these values, essentially working like a combination of a gauge and a counter.

Summaries, specifically, work by observing and recording an event, such as latency or size, and work in
percentiles. When we define a summary, we can define which percentiles we want to use, and the data
we take will be placed in the appropriate “bucket” for the percentile.

Histograms work similarly only its buckets aren’t restricted to percentages. Histograms allow us to see
how a metric happens, proportionally, in buckets.

For example, here’s a snapshot of a histogram I have running for a demo version of this course:

*forethought_requests_bucket{le="0.005"} 0*
*forethought_requests_bucket{le="0.01"} 0*
*forethought_requests_bucket{le="0.025"} 0*
*forethought_requests_bucket{le="0.05"} 0*
*forethought_requests_bucket{le="0.1"} 13*
*forethought_requests_bucket{le="0.25"} 31826*
*forethought_requests_bucket{le="0.5"} 31982*
*forethought_requests_bucket{le="1"} 34817*
*forethought_requests_bucket{le="2.5"} 34857*
*forethought_requests_bucket{le="5"} 34882*
*forethought_requests_bucket{le="10"} 34903*
*forethought_requests_bucket{le="+Inf"} 34910*
*forethought_requests_sum 8397.841003000045*
*forethought_requests_count 34910*

We can see that 13 request took between 0 and .1 milliseconds, 31826 between 0 and .25, and so on.
Notice a start with 0 got both of these listed buckets: This is because histograms are cumulative in Prometheus, so that .25 millisecond bucket includes everything from the .1 bucket and below, as well.
Also notice that _sum and _count metrics are included — this is what we mean by saying two time series,
with _sum being the sum of all values passed to Prometheus, and _count being the amount of calls made
for the metric (these are done with observe , as we’ll see shortly).

But before we can see this in action, let’s go ahead and write the code to create our summary and his-
togram. We’re going to be measuring the time it takes for each request made to our application, or the
latency.

Before we begin, move into the forethought directory and use npm to install the request-time client –
we’ll use this to measure our request times for us:
```js
cd forethought
```
```js
npm install response-time --save
```
Next, open the index.js file:
```js
sudo nano index.js
```
We can now define our summary and our histogram just as we have in the previous lessons:
```js
const tasksumm = new prom.Summary ({
  name: 'forethought_requests_summ',
  help: 'Latency in percentiles',
});
const taskhisto = new prom.Histogram ({
  name: 'forethought_requests_hist',
  help: 'Latency in history form',
});
```
Now comes the part that begins to differ from previous lessons — we’re not just simply increasing or
decreasing a value here, we’re passing in a number that is returned via the response-time middleware
we just installed. To call our response-time function, we’ll first create a variable:
```js
var responseTime = require('response-time');
```
Then we’ll write the function itself, where time is the response time in miliseconds:
```js
app.use(responseTime(function (req, res, time) {
  res.set('Content-Type', prom.register.contentType);
  res.end(await prom.register.metrics());
}));