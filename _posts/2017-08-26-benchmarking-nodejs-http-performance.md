---
layout: post
title: Benchmarking Node.js Http Performance Part 1 Express.js
---

## Overview

Intrigue by [this](https://hackernoon.com/go-vs-net-core-in-terms-of-http-performance-7535a61b67b8) blog by Gerasimos about
performance review of .NET core and Iris. I am interested to do the same with Node.js using Express framework.

## Preparation

Since current LTS (Long Term Support) of Node is 6.x.x so I decided to use this version.
I run an 2013 MacBook Pro with OSX Sierra in this spec:

- Processor: Intel Core i7 2.4 GHz
- Memory: 8 GB

## Testing

Since I like Typescript, I created a simple webserver using it than normal plain javascript. Typescript is basically a typed version of Javascript, it has typechecking functionality which Javascript itself doesn’t.

The code looks like this:

```javascript
// index.ts

import * as express from 'express';

const app = express();
const host = 'localhost';
const port = 3000;

app.get('/', (req, res) => {
    res.send('use ' + host + ':' + port + '/test');
})

app.route('/test')
 .get((req, res) => {
    res.send('test');
});

app.listen(port, () => {
    console.log('App is running on port ', port);
})
```

This will create an endpoint `/test` that will send simple string `test`.

To start running it we should install Typescript first by running:

`npm install -g typescript`

And we need to install `express` as well

`npm install express —save`

Then compile the .ts file to .js by running

`tsc index.ts`

the run it by running

`npm start or node index.js`

To test, I use the same command written on the blog.

`bombardier -c 125 -n 5000000 http://localhost:3000/test`

Running 5000000 requests with 125 connections, I got this results:

```shell
Bombarding http://localhost:3000/test with 5000000 requests using 125 connections
5000000 / 5000000 [====================================================] 100.00% 11m51s
Done!
Statistics Avg Stdev Max
Reqs/sec 7023.75 572.58 7833
Latency 17.79ms 1.62ms 219.78ms
HTTP codes:
1xx - 0, 2xx - 5000000, 3xx - 0, 4xx - 0, 5xx - 0
others - 0
Throughput: 1.83MB/s
```

So single Express.js process needs 11 minutes to handle 5000000 requests with average of **7023.75 requests per second**, with latency **17.79 ms**.

It is inferior when compared against Go with Iris with average of **105643.81 requests per second** with average latency of **1.18 ms**.
Even if we compare with .NET core, with **40226.03** requests per second with average latency of **3.09ms** it’s still losing.

## Optimising

According to Express js manual [here](https://expressjs.com/en/advanced/best-practice-performance.html#set-nodeenv-to-production)
we can optimise performance by using several things, like:

- Change NODE_ENV to production
- Use gzip compression
- Run app in a cluster
- Use load balancer
- Use reverse proxy.

To make it fair, since other frameworks run in only one process and not in a cluster, I added gzip compression and set `NODE_ENV` to `production`.

Here’s the result:

```shell
Bombarding http://localhost:3000/test with 5000000 requests using 125 connections
5000000 / 5000000 [====================================================] 100.00% 15m17s
Done!
Statistics Avg Stdev Max
Reqs/sec 5449.45 920.49 6589
Latency 22.94ms 3.91ms 116.15ms
HTTP codes:
1xx - 0, 2xx - 5000000, 3xx - 0, 4xx - 0, 5xx - 0
others - 0
Throughput: 1.54MB/s
```

Well, it turns out it needs more time to handle the request. Reading more, it turn’s out that compressing is more relevant if we the server returns more files like html, css, or images, while we only returning simple text only. And it seems using compression middleware adds more processing time to the server.

Set up the `NODE_ENV` to `prodcution` seems irrelevant, since according to the Express.js guide it only caches view templates and CSS.
But really? Let’s try with setting it up `NODE_ENV` to `production` only.

```shell
Bombarding http://localhost:3000/test with 5000000 requests using 125 connections
5000000 / 5000000 [==============================================================================================================================================] 100.00% 11m22s
Done!
Statistics Avg Stdev Max
Reqs/sec 7323.42 398.13 7974
Latency 17.05ms 1.09ms 236.24ms
HTTP codes:
1xx - 0, 2xx - 5000000, 3xx - 0, 4xx - 0, 5xx - 0
others - 0
Throughput: 1.91MB/s
```

It turns out it can handles more requests than the default one with **7323.42 requests per second** on average, even though it’s not significant.

## Source

Full source code is in Github [here](https://github.com/mamaz/express-benchmark).
