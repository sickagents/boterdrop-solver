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

### Step 2: Fetch Camoufox Browser

```python
import subprocess, sys

subprocess.run([sys.executable, "-m", "camoufox", "fetch"], check=True)
print("Camoufox fetched")
```

### Step 3: Config

```python
import json

config = {
    "headless": True,
    "thread": 3,
    "page_count": 2,
    "proxy_support": False,
    "proxy_file": "proxies.txt",
    "host": "0.0.0.0",
    "port": 8000,
    "debug": False
}

with open("config.json", "w") as f:
    json.dump(config, f, indent=4)

print(f"Config saved. Port: {config['port']}, Threads: {config['thread']}")
```

### Step 4: Start Server (Foreground)

Jalankan server di terminal terpisah agar bisa dipantau:

```bash
cd /path/to/boterdrop-solver
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
apt update -y && apt install -y xvfb libasound2
pip install fastapi==0.95.2 uvicorn "camoufox[fetch]" loguru psutil playwright
python3 -m camoufox fetch
echo "Y" | python3 api_server.py
```
