## Availability SLI
### The percentage of successful requests over the last 5m
(sum (rate(apiserver_request_total{job="apiserver",code=~"2.."}[2d])) / sum (rate(apiserver_request_total{job="apiserver"}[2d]))) * 100
Comment: Other than the [2d] timespan which is not [5m] as the requirement would suggest, is there anything else wrong with this query?
I followed the following logic: (successful_requests/total_requests)*100 where the successful requests have a 2xx response code.
Otherwise, I could have used the example in the course that stated that the query for the succesful request would be rate(apiserver_request_total{job="apiserver",code!~"5.."}[2d]
Corrected formula: (sum (rate(apiserver_request_total{job="apiserver",code!~"5.."}[5m])) / sum (rate(apiserver_request_total{job="apiserver"}[5m]))) * 100

## Latency SLI
### 90% of requests finish in these times
histogram_quantile(0.95, sum(rate(apiserver_request_duration_seconds_bucket{job="apiserver"}[5m])) by (le, verb))
Corrected formula: histogram_quantile(0.90, sum(rate(apiserver_request_duration_seconds_bucket{job="apiserver"}[5m])) by (le, verb))

## Throughput
### Successful requests per second
sum(rate(apiserver_request_total{job="apiserver",code=~"2.."}[5m]))
Comment: Why is this not correct? It is literraly the same query as in the course.

## Error Budget - Remaining Error Budget
### The error budget is 20%
(1 - (sum (rate(apiserver_request_total{job="apiserver",code=~"2.."}[2d])) / sum (rate(apiserver_request_total{job="apiserver"}[2d])))) * 100
Corrected formula:1 - ((1 - (sum(increase(apiserver_request_total{job="apiserver", code="200"}[7d])) by (verb)) / sum(increase(apiserver_request_total{job="apiserver"}[7d])) by (verb)) / (1 - .90))
