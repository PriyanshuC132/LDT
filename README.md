# Constant RPS Load Test
This directory contains a Python script for conducting load tests with a fixed requests per second (RPS) rate.

## Prerequisites
Before running the load test, ensure you have the following installed:
- Python 3.6+ : You need Python version 3.6 or higher
- Required Python packages:
  - requests
  - statistics
  - zoneinfo (for Python < 3.9, install `backports.zoneinfo`)

## Installation
1. Install required packages:
   ```sh
   pip install requests statistics
   ```
   
2. For Python < 3.9, also install zoneinfo:
   ```sh
   pip install backports.zoneinfo
   ```

## Preparing Test Payloads
1. Create a directory named `payloads` in the same folder as the script
2. Add JSON files in the `payloads` directory representing the request bodies to be sent
   - The script will randomly select from these payloads for each request

## Running the Test
To start the load test, run the following command:
```sh
python3 load_test_rps.py <target_url> <duration_seconds> <requests_per_second>
```

Example:
```sh
python3 load_test_rps.py https://api.example.com/endpoint 600 160
```
This will run a test against the specified URL for 600 seconds (10 minutes) at 160 requests per second.

## Command Line Arguments
The script requires three command line arguments:
- `target_url`: The endpoint URL to send requests to
- `duration_sec`: Duration of the test in seconds
- `rps`: Target requests per second to maintain

## Monitoring the Test
During execution, the script displays progress updates every 500 requests showing:
- Current timestamp
- Number of requests sent (and total planned)
- Current average RPS
- Response status distribution (200, 500, other)
- Current latency metrics (TP90, TP95, TP99)

## Test Summary
After completion, the script outputs a summary including:
- Total requests sent
- Response status counts
- Latency percentiles (TP90, TP95, TP99)
- Median response time
- Average RPS achieved
- Actual test duration

## Key Features
- Maintains constant RPS regardless of response times
- Uses threading for concurrent requests
- Handles timing compensation to ensure steady request rate
- Collects and reports detailed latency metrics
- Randomly selects from multiple payload options
- Provides real-time progress updates
