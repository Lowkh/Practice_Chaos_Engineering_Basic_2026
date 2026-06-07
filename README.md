# Chaos Engineering Lab for Beginners

A beginner-friendly hands-on lab to learn **chaos engineering** by running a small web app, checking that it works, introducing a controlled failure, observing the impact, and recovering the service.

## Learning objectives

By the end of this lab, participants should be able to:

- explain chaos engineering in simple terms,
- run a small containerized application,
- verify normal system behaviour,
- inject a small controlled fault,
- observe the impact of failure,
- restore the system and confirm recovery.

## What is chaos engineering?

Chaos engineering is the practice of **intentionally introducing controlled failure** into a system so that teams can learn how the system behaves and improve reliability.

For this beginner lab, the cycle is simple:

1. Start with a working system.
2. Confirm that it is healthy.
3. Break one small part on purpose.
4. Observe what changes.
5. Recover the service.

## Lab flow

In this practical, participants will:

1. install the required tools,
2. create the sample files by copy and paste,
3. build and run a simple Flask app with Docker,
4. verify the application is healthy,
5. stop the app container to simulate failure,
6. observe the outage,
7. restart the service and confirm recovery.

## Requirements

Participants need:

- Windows, macOS, or Linux,
- internet access,
- permission to install software,
- about 20 to 30 minutes.

Recommended software:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Python 3](https://www.python.org/downloads/)
- a text editor such as VS Code or Notepad++
- a browser such as Chrome or Edge

## Installation

### 1. Install Docker Desktop

Install Docker Desktop from the official site:

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

After installation:

1. Start Docker Desktop.
2. Wait until Docker reports that it is running.
3. Open a terminal and test:

```bash
docker --version
```

You should see a Docker version number.

### 2. Install Python 3

Install Python 3 from the official site:

[https://www.python.org/downloads/](https://www.python.org/downloads/)

For Windows, make sure **Add Python to PATH** is selected during installation.

Test the installation:

```bash
python --version
```

If needed, try:

```bash
python3 --version
```

## Project structure

Create a folder called `chaos-engineering-lab`.

Inside it, create the following files:

```text
chaos-engineering-lab/
├── app.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── check_app.py
```

## Copy-and-paste files

### `app.py`

```python
from flask import Flask
import socket
import os

app = Flask(__name__)

@app.route('/')
def home():
    hostname = socket.gethostname()
    return {
        "message": "Hello from the chaos engineering lab",
        "status": "ok",
        "hostname": hostname,
        "experiment": os.environ.get("EXPERIMENT_NAME", "baseline")
    }

@app.route('/health')
def health():
    return {"status": "healthy"}, 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### `requirements.txt`

```text
flask==3.0.3
```

### `Dockerfile`

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

ENV EXPERIMENT_NAME=baseline

EXPOSE 5000

CMD ["python", "app.py"]
```

### `docker-compose.yml`

```yaml
services:
  web:
    build: .
    container_name: chaos-web
    ports:
      - "5000:5000"
    environment:
      EXPERIMENT_NAME: baseline
```

### `check_app.py`

```python
import urllib.request
import json
import sys

url = "http://localhost:5000/health"

try:
    with urllib.request.urlopen(url, timeout=5) as response:
        data = json.loads(response.read().decode())
        print("Health check response:", data)
        if data.get("status") == "healthy":
            print("PASS: application is healthy")
            sys.exit(0)
        else:
            print("FAIL: unexpected health response")
            sys.exit(1)
except Exception as e:
    print("FAIL: could not reach application")
    print("Reason:", e)
    sys.exit(1)
```

## Run the lab

Open a terminal in the `chaos-engineering-lab` folder and run:

```bash
docker compose up --build -d
```

This will:

- build the Docker image,
- start the container,
- run the app in the background.

Check that the container is running:

```bash
docker ps
```

You should see a container named `chaos-web`.

## Verify the baseline

### 1. Check in the browser

Open:

- [http://localhost:5000/](http://localhost:5000/)
- [http://localhost:5000/health](http://localhost:5000/health)

Expected health response:

```json
{
  "status": "healthy"
}
```

### 2. Run the health check script

```bash
python check_app.py
```

If needed:

```bash
python3 check_app.py
```

Expected result:

```text
Health check response: {'status': 'healthy'}
PASS: application is healthy
```

This is the **baseline**. Always confirm the system is healthy before running a chaos experiment.

## Chaos experiment 1: Stop the application

Inject a simple controlled failure:

```bash
docker stop chaos-web
```

This simulates an application outage.

## Observe the failure

Repeat the checks.

### Browser check

Open:

- [http://localhost:5000/](http://localhost:5000/)

The page should no longer load.

### Health check script

```bash
python check_app.py
```

Expected result:

```text
FAIL: could not reach application
```

### Optional container check

```bash
docker ps -a
```

You should see `chaos-web` in a stopped or exited state.

## Reflection questions

Ask participants to record simple observations:

- What was working before the failure?
- What changed after the container stopped?
- How did they know the service was unavailable?
- Which check was easiest to understand: browser, script, or `docker ps -a`?

## Recover the service

Restart the container:

```bash
docker start chaos-web
```

Wait a few seconds, then run:

```bash
python check_app.py
```

Expected result:

```text
Health check response: {'status': 'healthy'}
PASS: application is healthy
```

Open the browser again:

- [http://localhost:5000/](http://localhost:5000/)

The service should be working again.

## Success criteria

Participants have completed the lab successfully if they can:

- start the application,
- verify the app in the browser,
- run the health check successfully,
- stop the container,
- observe the failure,
- restart the container,
- confirm recovery.

## Troubleshooting

### Docker command not found

Docker Desktop may not be installed or may not be running.

### Python command not found

Python may not be installed or may not be added to PATH.

### Port 5000 already in use

Change this in `docker-compose.yml`:

```yaml
      - "5000:5000"
```

To:

```yaml
      - "5001:5000"
```

Then update the URLs and `check_app.py` to use port `5001`.

### The app does not start

Check logs:

```bash
docker logs chaos-web
```

## Optional extension activities

After the basic lab, participants can try:

1. changing `EXPERIMENT_NAME` from `baseline` to `chaos-test`,
2. adding an artificial delay to simulate slow response,
3. removing and recreating the container,
4. predicting the outcome before each experiment.

## Cleanup

When finished, stop and remove the lab resources:

```bash
docker compose down
```

## Recap

This beginner lab follows the core chaos engineering loop:

1. build the system,
2. verify normal behaviour,
3. inject a fault,
4. observe the result,
5. recover the service.

That loop is the foundation for more advanced chaos engineering experiments later.
