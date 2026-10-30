# K6 lab results

## Part 0
- Step 0.1, http_req_duration: avg=255.55ms, min=255.55ms, med=255.55ms, max=255.55ms, p(90)=255.55ms, p(95)=255.55ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: Requests = 80, Rate = 3.83 req/s, p(95) = 270.71 ms. The rate stays below 5 requests per second because each virtual user executes an iteration consisting of an HTTP request (~260 ms) plus an explicit `sleep(1)` (1,000 ms), resulting in an average iteration duration of ~1.29 s. Since each VU can only make 1 request every ~1.29 s, 5 VUs can produce at most 5 / 1.29 ≈ 3.88 requests per second.
- Step 0.6, exit code of the normal run and of the run with P95=300: Normal run exit code = 0 (all thresholds passed); Run with P95=300 exit code = 99 (failed because p(95) was 378.51 ms, which exceeded the 300 ms threshold).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  - Rules that passed: `http_req_failed ['rate<0.01']` (0.00% failed), `http_req_duration ['p(95)<1000', 'p(99)<1500']` (both passed).
  - Measured metrics: p(95) = 371.79 ms, p(99) = 375.92 ms.
  - Exit code after change (setting p(95) < 300 ms): Exit code = 99.
- Step 0.8, what was wrong, and how the checks showed it: The API expected ingredient and tool names with Proper Case (e.g. `'Pepperoni'` and `'Knife'`) in `excludedIngredients` and `excludedTools`. Passing lowercase `'pepperoni'` and `'knife'` caused the backend filter to ignore the exclusions, returning pizzas with Pepperoni (check failed 8 times out of 75, 89% pass) and Knife (check failed 28 times out of 75, 62% pass). Exploring the API via `/api/ingredients/topping` and `/api/tools` revealed the capitalized naming. After updating the payload to `['Pepperoni']` and `['Knife']`, checks passed 100% (225/225).

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.50 | 358.98 ms | 369.12 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.34 | 359.11 ms | 379.66 ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.17 | 366.89 ms | 395.72 ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 1.07 | 365.51 ms | 375.83 ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 1.00 | 357.28 ms | 384.42 ms | 0.00% | 100.00% |

## Part B
1. Completed in Part A results table above.
2. In Step 0.2, each iteration performs one HTTP GET request and then sleeps for 1 second (`sleep(1)`). The total iteration time equals network response time (~260 ms) + think time (1000 ms) ≈ 1260 ms (~1.26–1.29 s). Therefore, one virtual user can perform approximately 1 / 1.29 ≈ 0.77 requests per second. With 5 virtual users running in parallel, the theoretical aggregate throughput is 5 × 0.77 ≈ 3.85 req/s (measured at 3.83 req/s), which is mathematically bounded below 5 requests per second.
3. The response time grew slightly between the average-load test (p(95) = 359.11 ms, max = 379.66 ms) and the stress test (p(95) = 366.89 ms, max = 395.72 ms). In terms of the operational envelope, 3 to 5 virtual users represent a very small load compared to QuickPizza's production capacity. As long as the load remains deep inside the operational envelope, the system handles concurrency without queuing bottlenecks, keeping response times relatively flat and stable.
4. During the spike test (ramping abruptly from 1 to 5 users within 5 seconds and holding for 30 seconds), response times briefly saw peak request durations around 375.83 ms. Due to QuickPizza's high underlying capacity, the system absorbed the spike gracefully without queuing, returning immediately to base latency (~255–260 ms for simple endpoints) within 5 to 10 seconds as users ramped back down.
5. In the soak test (running 2 virtual users continuously for 10 minutes), response times and error rates did not degrade (p(95) remained steady at 357.28 ms with 0.00% failures and 100% check success rate across 666 requests). A soak test over several hours or days is designed to reveal slow-accumulating defects such as memory leaks, database connection pool exhaustion, file descriptor leaks, thread pool starvation, or disk space accumulation from log files.
6. Percentiles like p(95) represent the 95th percentile worst-case user experience (only 5% of requests took longer). A standard average (arithmetic mean) is heavily skewed by high-frequency fast responses and hides latency outliers, failing to capture intermittent stalls or degraded performance experienced by long-tail users.
7. Two reasons why testing a shared public site over the internet differs from testing in a dedicated lab environment:
   - Network variability: Internet routing involves varying WAN latency, ISP routing, packet jitter, and TLS handshake fluctuations, whereas a lab environment has near-zero, deterministic local network latency.
   - Uncontrolled background load: A shared public demo site processes traffic from other learners globally at unpredictable times, whereas a lab test environment is isolated and controlled.
8. Stress, spike, or breakpoint tests using hundreds of virtual users generate massive traffic volumes that consume server CPU, memory, socket connections, and network bandwidth. Against a system you do not own or lack written permission to test, this can cause denial of service (DoS) for legitimate users, incur substantial cloud infrastructure costs for the owner, and may violate terms of service and computer crime laws.
9. Example of a vague performance requirement:
   - *Vague requirement:* "The mobile banking transfer page must be fast and responsive when people transfer money."
   - *Testable requirement:* "Under a sustained load of 50 concurrent users over 5 minutes, 95% of money transfer requests (`POST /api/transfer`) must complete within 800 ms, 99% within 1,200 ms, and the HTTP failure rate must not exceed 0.5%."
   - *K6 thresholds:*
   ```javascript
   export const options = {
     vus: 50,
     duration: '5m',
     thresholds: {
       'http_req_failed': ['rate<0.005'], // failure rate < 0.5%
       'http_req_duration{endpoint:transfer}': ['p(95)<800', 'p(99)<1200'],
     },
   };
   ```
