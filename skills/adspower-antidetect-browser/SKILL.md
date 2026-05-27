---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I use AdsPower for multi-account management
  - set up AdsPower browser profiles for automation
  - configure AdsPower antidetect browser
  - automate campaigns with AdsPower profiles
  - manage multiple accounts with AdsPower
  - integrate AdsPower API for browser automation
  - create and control AdsPower browser profiles
  - use AdsPower for marketing automation
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that allows marketing teams to manage multiple browser profiles with unique fingerprints for multi-account management, automation, and campaign execution. Each profile appears as a distinct user to websites, preventing account linking and bans.

## Overview

AdsPower provides:
- Multiple isolated browser profiles with unique fingerprints
- API for profile creation and automation
- RPA (Robotic Process Automation) capabilities
- Cloud synchronization for team collaboration
- Integration with automation frameworks

## Installation

AdsPower consists of two components:

1. **Desktop Application**: Download from official AdsPower website
2. **API Access**: Enable local API in AdsPower settings (default port: 50325)

```bash
# Verify AdsPower is running and API is accessible
curl http://localhost:50325/api/v1/browser/active
```

## API Configuration

AdsPower API runs locally when the application is active. Configure the base URL:

```python
# Python example
ADSPOWER_API_BASE = "http://localhost:50325"
API_ENDPOINT = f"{ADSPOWER_API_BASE}/api/v1"
```

```javascript
// JavaScript/Node.js example
const ADSPOWER_API_BASE = "http://localhost:50325";
const API_ENDPOINT = `${ADSPOWER_API_BASE}/api/v1`;
```

## Core API Operations

### 1. List Browser Profiles

```python
import requests

def list_profiles(group_id=None, page=1, page_size=100):
    """Get all browser profiles"""
    url = f"{API_ENDPOINT}/user/list"
    params = {
        "page": page,
        "page_size": page_size
    }
    if group_id:
        params["group_id"] = group_id
    
    response = requests.get(url, params=params)
    return response.json()

# Usage
profiles = list_profiles()
print(f"Total profiles: {profiles['data']['total']}")
```

```javascript
// JavaScript/Node.js
const axios = require('axios');

async function listProfiles(groupId = null, page = 1, pageSize = 100) {
    const params = { page, page_size: pageSize };
    if (groupId) params.group_id = groupId;
    
    const response = await axios.get(`${API_ENDPOINT}/user/list`, { params });
    return response.data;
}
```

### 2. Create Browser Profile

```python
def create_profile(name, group_id=0, fingerprint_config=None):
    """Create a new browser profile with custom fingerprint"""
    url = f"{API_ENDPOINT}/user/create"
    
    payload = {
        "name": name,
        "group_id": group_id,
        "fingerprint_config": fingerprint_config or {
            "automatic_timezone": 1,
            "language": ["en-US"],
            "ua": "random"
        }
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Create profile with custom settings
profile = create_profile(
    name="Marketing Campaign 1",
    fingerprint_config={
        "automatic_timezone": 1,
        "webrtc": "proxy",
        "language": ["en-US", "en"],
        "ua": "random",
        "screen_resolution": "1920_1080"
    }
)
print(f"Profile ID: {profile['data']['id']}")
```

### 3. Start Browser Profile

```python
def start_browser(profile_id, headless=False):
    """Launch a browser profile"""
    url = f"{API_ENDPOINT}/browser/start"
    params = {
        "user_id": profile_id,
        "headless": 1 if headless else 0
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            "selenium_address": data['data']['ws']['selenium'],
            "webdriver_path": data['data']['webdriver'],
            "debug_port": data['data']['debug_port']
        }
    return None

# Start browser and get connection details
connection = start_browser("profile_id_here")
print(f"Selenium address: {connection['selenium_address']}")
```

### 4. Close Browser Profile

```python
def close_browser(profile_id):
    """Close an active browser profile"""
    url = f"{API_ENDPOINT}/browser/stop"
    params = {"user_id": profile_id}
    
    response = requests.get(url, params=params)
    return response.json()
```

## Integration with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_to_adspower_profile(profile_id):
    """Connect Selenium to AdsPower profile"""
    # Start the profile
    connection = start_browser(profile_id)
    
    if not connection:
        raise Exception("Failed to start browser profile")
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress", 
        f"127.0.0.1:{connection['debug_port']}"
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(
        executable_path=connection['webdriver_path'],
        options=chrome_options
    )
    
    return driver

# Usage
driver = connect_to_adspower_profile("your_profile_id")
driver.get("https://example.com")
# Perform automation tasks
driver.quit()
close_browser("your_profile_id")
```

## Integration with Playwright

```javascript
const { chromium } = require('playwright');

async function connectToAdsPowerProfile(profileId) {
    // Start the profile
    const startUrl = `${API_ENDPOINT}/browser/start?user_id=${profileId}`;
    const response = await axios.get(startUrl);
    
    if (response.data.code !== 0) {
        throw new Error('Failed to start browser profile');
    }
    
    const { ws } = response.data.data;
    
    // Connect Playwright to the browser
    const browser = await chromium.connectOverCDP(ws.puppeteer);
    const context = browser.contexts()[0];
    const page = context.pages()[0] || await context.newPage();
    
    return { browser, page };
}

// Usage
(async () => {
    const { browser, page } = await connectToAdsPowerProfile('profile_id');
    await page.goto('https://example.com');
    // Perform automation
    await browser.close();
    await axios.get(`${API_ENDPOINT}/browser/stop?user_id=profile_id`);
})();
```

## Common Patterns

### Multi-Account Campaign Management

```python
import concurrent.futures

def run_campaign_on_profile(profile_id, campaign_url):
    """Execute campaign on a single profile"""
    driver = connect_to_adspower_profile(profile_id)
    try:
        driver.get(campaign_url)
        # Perform campaign actions
        # e.g., click ads, fill forms, etc.
        return {"profile": profile_id, "status": "success"}
    except Exception as e:
        return {"profile": profile_id, "status": "error", "message": str(e)}
    finally:
        driver.quit()
        close_browser(profile_id)

def run_parallel_campaigns(profile_ids, campaign_url, max_workers=5):
    """Run campaigns across multiple profiles in parallel"""
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [
            executor.submit(run_campaign_on_profile, pid, campaign_url)
            for pid in profile_ids
        ]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]
    return results

# Execute campaign on 10 profiles
profile_ids = ["profile1", "profile2", "profile3"]  # Get from list_profiles()
results = run_parallel_campaigns(profile_ids, "https://campaign-url.com")
```

### Profile Management with Proxy

```python
def create_profile_with_proxy(name, proxy_config):
    """Create profile with specific proxy settings"""
    url = f"{API_ENDPOINT}/user/create"
    
    payload = {
        "name": name,
        "group_id": 0,
        "fingerprint_config": {
            "automatic_timezone": 1,
            "webrtc": "proxy"
        },
        "user_proxy_config": {
            "proxy_soft": "other",
            "proxy_type": proxy_config["type"],  # http, https, socks5
            "proxy_host": proxy_config["host"],
            "proxy_port": proxy_config["port"],
            "proxy_user": proxy_config.get("username", ""),
            "proxy_password": proxy_config.get("password", "")
        }
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Create profile with proxy
proxy = {
    "type": "socks5",
    "host": "proxy.example.com",
    "port": 1080,
    "username": "user",
    "password": "pass"
}
profile = create_profile_with_proxy("Profile with Proxy", proxy)
```

## Troubleshooting

### API Not Responding

```python
def check_adspower_status():
    """Verify AdsPower is running"""
    try:
        response = requests.get(f"{ADSPOWER_API_BASE}/status", timeout=5)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        return False

if not check_adspower_status():
    print("AdsPower application is not running or API is disabled")
```

### Profile Won't Start

- Ensure profile exists: Use `list_profiles()` to verify
- Check if profile is already open: Close it first with `close_browser()`
- Verify sufficient system resources (RAM, CPU)

### Connection Issues

```python
def safe_start_browser(profile_id, max_retries=3):
    """Start browser with retry logic"""
    for attempt in range(max_retries):
        try:
            connection = start_browser(profile_id)
            if connection:
                return connection
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2)
    return None
```

## Environment Variables

```bash
# .env file
ADSPOWER_API_BASE=http://localhost:50325
ADSPOWER_GROUP_ID=0
MAX_CONCURRENT_PROFILES=5
```

```python
import os
from dotenv import load_dotenv

load_dotenv()

ADSPOWER_API_BASE = os.getenv("ADSPOWER_API_BASE", "http://localhost:50325")
API_ENDPOINT = f"{ADSPOWER_API_BASE}/api/v1"
```
