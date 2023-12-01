# API Service

| Category     | SLI                                                        |  SLO                                                                                                        |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Availability | total no. of sucessful requests / total no. of requests    | 99%                                                                                                         |
| Latency      | 90th percentile latency over a 5 min period                | 90% of requests below 100ms                                                                                 |
| Error Budget | 1 - availability                                           | Error budget is defined at 20%. This means that 20% of the requests can fail and still be within the budget |
| Throughput   | total number of requests over a 5 min period               | 5 RPS indicates the application is functioning                                                              |
