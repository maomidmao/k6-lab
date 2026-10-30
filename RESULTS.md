 # K6 lab results
## Part 0
- Step 0.1, http_req_duration:\
http_req_duration: 467.96 ms (all values are the same, one request)

- Step 0.2, requests, rate and p(95), and why the rate stays below 5:\
77 requests, 3.67 requests per second, p(95) = 316.96 ms.
The rate stays below 5 because one virtual user does not send requests non-stop. One iteration is one request (about 285 ms on average) plus sleep(1), the 1 second of think time, so an iteration takes about 1.32 s (iteration_duration avg = 1.32 s). One user can therefore send only about 1 / 1 32 = 0.76 requests per second, and 5 users send about 5 x 0.76 = 3.8 requests per second. The measured 3.67 is a little lower because K6 divides by the whole run time of 21.0 s, not exactly 20 s.

- Step 0.6, exit code of the normal run and of the run with P95=300: \
Normal run: 0 (all thresholds passed). Run with P95=300: 99 (the p(95) threshold failed, so K6 reports the test as failed).

- Step 0.7, rules that passed, p(95), p(99), exit code after your change:\
All 3 rules passed: failed rate 0.00% (<1%), p(95) = 401.21 ms (<1000), p(99) = 456.74 ms (<1500). After changing p(95)<1000 to p(95)<300, the p(95) rule failed (p(95) = 437.26 ms) while p(99) and the failed rate still passed. The exit code was 99.

- Step 0.8, what was wrong, and how the checks showed it:\
The request used 'pepperoni' and 'knife', but the API names are 'Pepperoni' and 'Knife'. The API ignored them and still answered 200. The checks failed 15% (no pepperoni) and 38% (no knife needed) of the time. After fixing the names, all checks passed (225 of 225). Many requests under load show the problem clearly, and checking the answer's content finds it, not just the status code.

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.50 | 376 ms | 377 ms | 0.00% | 100% |
| A2 | Average load | 3 | 1.31 | 376 ms | 1.08 s | 0.00% | 100% |
| A3 | Stress | 5 | 2.06 | 481 ms | 1.72 s | 0.00% | 100% |
| A4 | Spike | 5 | 1.01 | 470 ms | 1.38 s | 0.00% | 100% |
| A5 | Soak |  2 | 0.92  | 794 ms | 2.4 s  | 0.00%  | 100%   |

## Part B
1. The results table is in Part A above. The p(95) and maximum include all HTTP requests.

2. Five virtual users do not always send five requests every second. Each user waits for a response and then sleeps for one second before sending the next request. My test sent 77 requests in about 21 seconds, giving a rate of 3.67 requests/s. Each cycle took about 1.32 seconds on average, so one user sent about 0.76 requests/s.

3. Yes. The overall p(95) increased from 376 ms with up to 3 users in the average-load test to 481 ms with up to 5 users in the stress test. This is an increase of about 28%. The operational envelope is the range of load where a system meets its requirements. Both tests had 0% failed requests and 100% passing checks. In the stress test, the home page and recommendation response times also passed their separate thresholds. Therefore, both tested loads were within the limits for these runs, even though the stress test was slower. These results do not show the system's maximum capacity or prove that the higher load alone caused the difference, because QuickPizza is a shared site.

4. The spike started at about 30 seconds, reached 5 users by 35 seconds, and dropped back to 1 user by about 70 seconds. Before the spike, the p(95) readings were roughly 353-421 ms. They rose to as high as 645 ms around 70 seconds, then fell to about 399 ms at 110 seconds. This means the response time returned to around its earlier level about 40 seconds after the load dropped. This is an estimate because response times changed from one 10-second interval to another.

5. Response time sometimes became slow, but it did not show a clear, steady increase over the test. For example, p(95) was about 422 ms at 60 seconds and 405 ms at 120 seconds, with spikes to 1,081 ms at 50 seconds and 1,159 ms at 100 seconds. Later, it was about 430 ms at 520 seconds and 605 ms at 600 seconds, with a spike to 1,645 ms at 570 seconds. The error rate stayed at 0% throughout the recorded intervals. A test lasting several hours could reveal a memory leak, where an app keeps using more memory until it slows down or fails. This 11-minute test cannot prove that there is no memory leak.

6. The average can hide slow requests because many fast responses bring it down. The p(95) shows the time within which 95% of requests finished. This helps us see delays that the average might hide, although it does not describe the slowest 5% of requests.

7. First, a lab system may have different hardware, settings, data, or software. Second, the network connection may be faster or more stable in a lab. QuickPizza is also shared with other users, whose activity may affect its performance. These differences can change response times and request rates.

8. Hundreds of virtual users could overload a site, prevent real users from using it, and cause extra costs. We should run these tests only with the owner's permission and within agreed load limits.

9. Vague requirement for a messaging app: "Sending a message should be fast and reliable." Testable requirement: "During a test with up to 50 virtual users sending messages, at least 95% of requests to send a message must finish in under 500 ms, and fewer than 1% of those requests may fail." Tag those requests with `name: 'send_message'`, then set these thresholds:

   ```js
   thresholds: {
     'http_req_duration{name:send_message}': ['p(95)<500'],
     'http_req_failed{name:send_message}': ['rate<0.01'],
   }
   ```

   The script must also set up the 50-user test and add the tag to each send-message request. The thresholds alone do not create the load.
