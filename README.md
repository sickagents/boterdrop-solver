# Boterdrop Solver

Local CAPTCHA solver (Turnstile, cf_clearance, reCAPTCHA v3, AWS WAF). FastAPI + Camoufox.

## Jupyter Notebook Setup

### Step 1: Install Dependencies

```python
import subprocess, sys

subprocess.run(["apt-get", "update", "-qq"], check=True)
subprocess.run(["apt-get", "install", "-y", "-qq", "xvfb", "libasound2"], check=True)

subprocess.run([sys.executable, "-m", "pip", "install", "-q",
    "fastapi==0.95.2", "uvicorn", "camoufox[fetch]", "loguru", "psutil", "playwright"], check=True)

print("Done")
```

### Step 2: Clone & Fetch

```python
import subprocess, sys, os

REPO_DIR = os.path.expanduser("~/boterdrop-solver")

if not os.path.exists(REPO_DIR):
    subprocess.run(["git", "clone", "https://github.com/sickagents/boterdrop-solver", REPO_DIR], check=True)
    print(f"Cloned to {REPO_DIR}")
else:
    print(f"Already exists: {REPO_DIR}")

subprocess.run([sys.executable, "-m", "camoufox", "fetch"], check=True)
print("Camoufox fetched")
```

### Step 3: Config

```python
import json, os

REPO_DIR = os.path.expanduser("~/boterdrop-solver")

config = {
    "headless": True,
    "thread": 3,
    "page_count": 2,
    "proxy_support": True,
    "proxy_file": "proxies.txt",
    "host": "0.0.0.0",
    "port": 8000,
    "debug": False
}

config_path = os.path.join(REPO_DIR, "config.json")
with open(config_path, "w") as f:
    json.dump(config, f, indent=4)

print(f"Config saved: {config_path}")
```

### Step 4: Start Server

Jalankan di **terminal terpisah** (bukan Jupyter):

```bash
cd ~/boterdrop-solver
echo "Y" | python3 api_server.py
```

Server akan jalan di `http://0.0.0.0:8000`. Biarkan terminal ini terbuka.

### Step 5: Test Turnstile Solve

```python
import requests

r = requests.get("http://localhost:8000/turnstile", params={
    "sitekey": "0x4AAAAAAAJel0iaAR3mgkjp",
    "url": "https://dash.cloudflare.com/sign-up"
}, timeout=120)

data = r.json()
if data.get("token"):
    print(f"Token: {data['token'][:50]}...")
else:
    print(f"Error: {data}")
```

## API Endpoints

| Endpoint | Method | Params | Description |
|---|---|---|---|
| `/turnstile` | GET | sitekey, url | Cloudflare Turnstile token |
| `/clearance` | GET | url | cf_clearance cookie |
| `/aws-token` | GET | url | AWS WAF token cookie |
| `/recaptchaV3` | GET | sitekey, url | reCAPTCHA v3 token |

## Terminal Setup (Non-Jupyter)

```bash
git clone https://github.com/sickagents/boterdrop-solver ~/boterdrop-solver
cd ~/boterdrop-solver
apt update -y && apt install -y xvfb libasound2
pip install fastapi==0.95.2 uvicorn "camoufox[fetch]" loguru psutil playwright
python3 -m camoufox fetch
echo "Y" | python3 api_server.py
```
