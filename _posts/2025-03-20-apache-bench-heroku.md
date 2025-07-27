---
layout: post
title: "Quick Load Testing for Heroku Apps with Apache Bench"
date: 2025-03-20 11:26:00 +0545
categories: [Heroku, Performance]
tags: [heroku, performance]
---

Apache Bench (ab) is a lightweight command-line tool that allows you to quickly perform load testing on web applications, making it ideal for Heroku-deployed Rails applications. This guide walks through the process of setting up and running basic load tests on your Heroku app.

## Why Apache Bench?

- **Simplicity**: No complex setup required
- **Speed**: Tests can be executed in minutes
- **Built-in**: Comes pre-installed on many systems
- **Lightweight**: Minimal resource requirements
- **Sufficient**: For many basic load testing needs

## Prerequisites

- Apache Bench installed on your system
  - On macOS: Comes pre-installed
  - On Ubuntu/Debian: `sudo apt-get install apache2-utils`
  - On Windows: Install via Apache HTTP Server or use WSL
- A Heroku application you want to test
- Heroku CLI installed and configured

## Basic Apache Bench Command Syntax

The basic syntax for Apache Bench is:

```bash
ab [options] [http[s]://]hostname[:port]/path
```

Common options include:

- `-n`: Number of requests to perform
- `-c`: Number of concurrent requests
- `-t`: Timelimit in seconds
- `-A`: Supply basic authentication credentials
- `-C`: Add cookie line to requests
- `-H`: Add arbitrary header to requests

## Setting Up for Heroku Testing

### 1. Enable Enhanced Logging

For better visibility during testing:

```bash
heroku addons:upgrade logging:expanded --remote [staging/production]
```

### 2. Add Monitoring (Optional but Recommended)

New Relic provides valuable insights during load tests:

```bash
heroku addons:add newrelic:standard --remote [staging/production]
```

### 3. Scale Up Dynos Before Testing

Increase your app's capacity temporarily for testing:

```bash
heroku ps:scale web=4 --remote [staging/production]
```

## Running Basic Load Tests

### Simple Homepage Test

Test your app's homepage with 1,000 requests and 10 concurrent users:

```bash
ab -n 1000 -c 10 https://your-app.herokuapp.com/
```

### Testing with Authentication

If your staging app is password protected:

```bash
ab -n 5000 -c 50 -A username:password https://staging-app.herokuapp.com/
```

### Testing a Specific Endpoint

Test a specific API endpoint or page:

```bash
ab -n 2000 -c 20 https://your-app.herokuapp.com/api/products
```

### Testing with a Session

For testing authenticated user flows, grab a cookie from your browser:

```bash
ab -n 1000 -c 10 -C "_session_id=1234abcd" https://your-app.herokuapp.com/dashboard
```

## Real-World Example

Here's a practical example for a medium-traffic Heroku application:

```bash
# Scale up dynos for testing
heroku ps:scale web=12 --remote staging

# Open logs in another terminal window
heroku logs -t --remote staging

# Run the test (50k requests, 50 concurrent users)
ab -n 50000 -c 50 -A user:password https://staging.your-app.com/

# After testing, scale back down
heroku ps:scale web=1 --remote staging
```

## Interpreting Results

Apache Bench provides detailed statistics after each test:

```
Server Software:        Cowboy
Server Hostname:        myapp.herokuapp.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128

Document Path:          /
Document Length:        11322 bytes

Concurrency Level:      50
Time taken for tests:   30.285 seconds
Complete requests:      5000
Failed requests:        0
Total transferred:      59845000 bytes
HTML transferred:       56610000 bytes
Requests per second:    165.10 [#/sec] (mean)
Time per request:       302.850 [ms] (mean)
Time per request:       6.057 [ms] (mean, across all concurrent requests)
Transfer rate:          1930.36 [Kbytes/sec] received
```

Key metrics to observe:

- **Requests per second**: Higher is better
- **Time per request**: Lower is better
- **Failed requests**: Should be zero or minimal
- **Connect/Processing/Waiting times**: Helps identify bottlenecks

## Monitoring During Tests

### Heroku Logs

While tests are running, watch your logs:

```bash
heroku logs -t --remote staging
```

Look for:
- Error rates
- Request queuing (indicates you need more dynos)
- Slow database queries
- H12 errors (request timeout)

### New Relic

Check your New Relic dashboard during and after tests for:

- Response time breakdown
- Database load
- Error rates
- Apdex score
- Memory usage

## Realistic Load Testing Strategy

1. **Start small**: Begin with low numbers (e.g., -n 500 -c 5)
2. **Gradually increase**: Double numbers until you see degradation
3. **Test multiple endpoints**: Different routes may have different bottlenecks
4. **Mix in complex operations**: Don't just test the homepage
5. **Test at different times**: Performance can vary based on database size/activity

## Limitations of Apache Bench

Apache Bench is useful for quick tests but has limitations:

- Limited to around 50 concurrent users
- Tests single URLs rather than user journeys
- Doesn't simulate browser behavior (JS execution, asset loading)
- Can't simulate gradual traffic ramp-up

For more comprehensive testing, consider tools like:
- Siege
- JMeter
- k6
- Gatling
- Tsung
- Blitz.io (commercial)

## Conclusion

Apache Bench provides a quick, easy way to test your Heroku application's performance under load. While not as comprehensive as dedicated load testing services, it gives you immediate feedback about how your application handles concurrent traffic and can help identify performance bottlenecks before they impact real users.

Remember that the goal is to measure, improve, and measure again. Use the insights gained from load testing to guide your optimization efforts, whether that's adding caching, optimizing database queries, or scaling your Heroku resources.

{% include inarticle-adsense.html %}