## Availability SLI
### The percentage of successful requests over the last 5m
(sum (rate(apiserver_request_total{job="apiserver",code=~"2.."}[2d])) / sum (rate(apiserver_request_total{job="apiserver"}[2d]))) * 100

## Latency SLI
### 90% of requests finish in these times
histogram_quantile(0.95, sum(rate(apiserver_request_duration_seconds_bucket{job="apiserver"}[5m])) by (le, verb))


## Throughput
### Successful requests per second
sum(rate(apiserver_request_total{job="apiserver",code=~"2.."}[5m]))

## Error Budget - Remaining Error Budget
### The error budget is 20%
(1 - (sum (rate(apiserver_request_total{job="apiserver",code=~"2.."}[2d])) / sum (rate(apiserver_request_total{job="apiserver"}[2d])))) * 100