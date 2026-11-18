# jmeter_performance_security_testing

## what it is

a small example of two kinds of testing beyond functional automation: load testing a REST API with Apache JMeter, and security scanning a web app with OWASP ZAP.

| folder | content |
| --- | --- |
| `jmeter/` | four JMeter test plans for the same API flow: `qa-mini-project-jmeter.jmx` (1 user), `qa-mini-project-jmeter-10users.jmx`, `qa-mini-project-jmeter-50users.jmx`, `qa-mini-project-jmeter-100users.jmx` |
| `ZAP/` | an OWASP ZAP session named `altaShop-Web-ZAP` (session, `.properties` and `.script` files) |

the JMeter plans differ only in the thread group size (1, 10, 50, 100 users, ramp-up 1 second, 1 loop). each plan:

- targets `altashop-api.fly.dev` over HTTPS through the user variable `BASE_URL`.
- has four groups: Authentication, Products, Categories, Orders. 22 requests in total, for example `auth-register`, `auth-login`, `products-create-item`, `products-get-by-id`, `products-set-item-rating`, `categories-get-all-category`, `orders-create-order`.
- contains negative requests (`auth-register-invalid`, `auth-login-invalid`, `products-get-by-id-invalid`, `products-delete-item-invalid`, `products-set-item-rating-invalid`) with assertions on the response code and on `$.error`.
- chains requests with extractors: the registered email and password, and the product ID, are taken from earlier responses and reused (`${demo_products_id}`).
- has a cookie manager and a View Results Tree listener. it has no timers.

the `ZAP/` folder holds only a saved ZAP session (an unnamed, empty one, no alerts or history inside). it is a place to put your own scan, not a scan result.

## when to use it

- you want to see how a functional API flow becomes a JMeter load plan, and how the same flow runs with more users.
- you want a starting point for running JMeter from the command line and producing the HTML dashboard.

when not to use it:

- you want load tests as Python code: use https://github.com/ClasherGeek77/locust_boilerplate
- you want functional API tests: use https://github.com/ClasherGeek77/pytest_api_boilerplate or https://github.com/ClasherGeek77/katalon_api_boilerplate
- you want a local API to hit instead of a hosted one: use https://github.com/ClasherGeek77/reqresin_clone (its routes differ, so the plans need new requests)

## what you need

- Java and Apache JMeter 5.5 or newer, from https://jmeter.apache.org/download_jmeter.cgi . the plans were saved with JMeter 5.5 (recorded in the `.jmx` header); a newer JMeter should open them (not verified here).
- internet access to the target `https://altashop-api.fly.dev`. when this README was checked, the host did not answer (see "if it breaks"), so the plans may need a new target.
- for the security part: OWASP ZAP from https://www.zaproxy.org/download/ , or its Docker image.

## quick start

run one plan headless from the root of the repository, and write the HTML dashboard:

```bash
jmeter -n -t jmeter/qa-mini-project-jmeter-10users.jmx -l results.jtl -e -o Reports/
```

then open `Reports/index.html` in a browser.

you should see a console summary with the number of samples and errors, and a dashboard with response times and throughput. not run here (JMeter was not installed and the target did not answer). `Reports/` must not exist or must be empty before the run; `*.jtl`, `Reports/` and `jmeter-report-*/` are in `.gitignore`.

to look at a plan first, open it in the GUI:

```bash
jmeter -t jmeter/qa-mini-project-jmeter.jmx
```

use the GUI to build and debug, and the `-n` command line to run the 50 and 100 user plans.

for ZAP: start ZAP, `File`, `Open Session`, choose `ZAP/altaShop-Web-ZAP.session`, then run a baseline scan against your own target (a spider, then an active or baseline scan). the session file does not hold a scan configuration.

## how to read the code

- open `jmeter/qa-mini-project-jmeter.jmx` first (1 user). read it in this order: User Defined Variables (`BASE_URL`), HTTP Request Defaults, then the four groups top to bottom.
- find the extractors in `auth-register` and `products-create-item` and follow where the extracted values are used later.
- compare the four `.jmx` files: only `ThreadGroup.num_threads` changes.

## if it breaks

| symptom | cause | fix |
| --- | --- | --- |
| every request is an error, `UnknownHostException` or connection refused | the target `altashop-api.fly.dev` is down or gone | change `BASE_URL` in User Defined Variables to an API you control |
| `jmeter: command not found` | JMeter's `bin/` folder is not on the PATH | use the full path to `bin/jmeter` |
| `-o` fails with "folder is not empty" | `Reports/` exists from an earlier run | delete it, or use a new folder name |
| `FileNotFoundException` for the `.jmx` | you ran the command from another folder, or used a path that does not exist | run from the repository root and use the `jmeter/` path |
| many 400 or 409 responses at 50 or 100 users | the API rejects duplicate or invalid data under load | check the response body in View Results Tree and the extractor values |
| ZAP cannot open the session | the session was saved by a different ZAP version (HSQLDB 2.7.1 inside) | create a new session and save your own |

## next

- load tests as Python code: https://github.com/ClasherGeek77/locust_boilerplate
- functional API tests: https://github.com/ClasherGeek77/pytest_api_boilerplate
- a local API target: https://github.com/ClasherGeek77/reqresin_clone
- all repositories: https://github.com/ClasherGeek77
