# Tugatilmagan ma`lumot
# Project Clone on github
# Install Prom-Client
## Change index file
```js
//Collect default
const collectDefaultMetrics = prom.collectDefaultMetrics;
collectDefaultMetrics({ prefix: 'forethought' });
```

versiyaga qarab o`zgaradi
```js
//add metrics endpoint 
app.get('/metrics', async function(req, res){
        res.set('Content-Type', prom.register.contentType);
        res.end(await prom.register.metrics());
});
```

## Histograms and Summaries
var responseTime = require('response-time');

const tasksumm = new prom.Summary ({
name: 'forethought_requests_summ',
help: 'Latency in percentiles',
});
const taskhisto = new prom.Histogram ({
name: 'forethought_requests_hist',
help: 'Latency in history form',
});


//track response time
app.use(responseTime(function (req, res, time) {
        tasksumm.observe(time);
        taskhisto.observe(time);
}));


