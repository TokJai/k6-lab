# K6 Performance Testing Lab (k6-lab)

Course: ITDS362 Software Quality Assurance and Testing  
Doc: Lab, Performance Testing with Grafana K6  

## Project Overview
This repository contains the performance testing scripts and results executed against the QuickPizza demonstration platform using Grafana K6.

### Repository Contents
- `01-first-test.js`: Step 0.1 - 0.3 basic GET test with VU and iteration controls.
- `04-pizza-api.js`: Step 0.4 API request testing with payload, auth token, and deep response assertions.
- `05-no-token.js`: Step 0.5 Security negative test checking HTTP 401 unauthorized status without triggering failure counters.
- `06-thresholds.js`: Step 0.6 Performance threshold validation (SLA criteria) and exit code validation.
- `07-requirement.js`: Step 0.7 Mapping business requirements to K6 thresholds.
- `08-bug-hunt.js`: Step 0.8 Functional defect identification under load and bug fix.
- `explore.js`: Step 0.8 API metadata exploration for valid toppings and tools.
- `journey.js`: Realistic multi-step user journey module (Homepage -> Quotes -> Pizza Recommendation).
- `a1-smoke.js`: Smoke test with 1 VU for 30s.
- `a2-average.js`: Average load test ramping to 3 VUs.
- `a3-stress.js`: Stress test ramping to 5 VUs.
- `a4-spike.js`: Spike test jumping abruptly to 5 VUs to test recovery.
- `a5-soak.js`: Soak test running 2 VUs for 10 minutes to test endurance and stability.
- `reports/`: Exported K6 web dashboard HTML reports (`a2-average.html`, `a3-stress.html`, `a4-spike.html`, `a5-soak.html`).
- `RESULTS.md`: Detailed test metrics, observations, and answers to Part B reflection questions.
