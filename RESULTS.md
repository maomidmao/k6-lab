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
| A2 | Average load | 3 | 1.32 | 358 ms | 398 ms | 0.00% | 100% |
| A3 | Stress | 5 | | | | | |
| A4 | Spike | 5 | | | | | |
| A5 | Soak | 2 | | | | | |
## Part B
1.
2.
3.
4.
5.
6.
7.
8.
9.