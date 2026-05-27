---
name: adspower-antidetect-browser
description: AdsPower antidetect browser automation for multi-account management and marketing campaigns
triggers:
  - "how do I automate AdsPower browser profiles"
  - "manage multiple AdsPower accounts programmatically"
  - "control AdsPower antidetect browser via API"
  - "create and launch AdsPower profiles"
  - "automate marketing campaigns with AdsPower"
  - "use AdsPower local API for automation"
  - "manage browser fingerprints with AdsPower"
  - "integrate AdsPower with RPA workflows"
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser designed for managing multiple accounts across platforms without triggering security flags. Each browser profile maintains unique fingerprints (canvas, WebGL, fonts, timezone, geolocation, etc.) to prevent account association. This skill covers automation via AdsPower's Local API for profile management, browser control, and marketing campaign automation.

## What AdsPower Does

- **Multi-Account Management**: Create and manage hundreds of isolated browser profiles
- **Fingerprint Protection**: Each profile has unique digital fingerprints (user agent, canvas, WebRTC, etc.)
- **Team Collaboration**: Share profiles and configurations across marketing teams
- **Automation Support**: Local API for programmatic control via Selenium/Puppeteer
- **Proxy Management**: Assign different proxies per profile to simulate geographic diversity

## Prerequisites

1. **AdsPower Application**: Install from official website (requires license)
2. **Local API Enabled**: AdsPower must be running with Local API active (default port: 50325)
3. **API Documentation**: Access via `http://local.adspower.net:50325` when running

## Local API Base URL

```
http://local.adspower.net:50325
```

Alternative localhost access:
```
http://127.0.0.1:50325
```

## Core API Operations

### 1. List All Profiles

**Endpoint**: `GET /api/v1/user/list`

```python
import requests

API_BASE = "http://local.adspower.net:50325"

def list_profiles(page=1, page_size=100):
    """Retrieve all browser profiles"""
    response = requests.get(
        f"{API_BASE}/api/v1/user/list",
        params={
            "page_size": page_size,
            "page": page
        }
    )
    data = response.json()
    if data["code"] == 0:
        return data["data"]["list"]
    else:
        raise Exception(f"Error: {data['msg']}")

# Usage
profiles = list_profiles()
for profile in profiles:
    print(f"ID: {profile['user_id']}, Name: {profile['name']}")
```

### 2. Create New Profile

**Endpoint**: `POST /api/v1/user/create`

```python
def create_profile(name, group_id="0", proxy_config=None):
    """Create a new browser profile with custom fingerprint"""
    payload = {
        "name": name,
        "group_id": group_id,
        "domain_name": "",
        "open_urls": [],
        "repeat_config": [0],
        "username": "",
        "password": "",
        "fakey": "",
        "cookie": [],
        "ignore_cookie_error": 0,
        "ip": "",
        "country": "",
        "region": "",
        "city": "",
        "remark": "",
        "ipchecker": 0,
        "sys_app_cate_id": "0",
        "user_proxy_config": proxy_config or {}
    }
    
    response = requests.post(
        f"{API_BASE}/api/v1/user/create",
        json=payload
    )
    data = response.json()
    if data["code"] == 0:
        return data["data"]["id"]
    else:
        raise Exception(f"Failed to create profile: {data['msg']}")

# Create profile with proxy
new_profile_id = create_profile(
    name="Marketing Campaign - Facebook",
    proxy_config={
        "proxy_soft": "other",
        "proxy_type": "http",
        "proxy_host": "proxy.example.com",
        "proxy_port": "8080",
        "proxy_user": "username",
        "proxy_password": "password"
    }
)
print(f"Created profile: {new_profile_id}")
```

### 3. Launch Browser Profile

**Endpoint**: `GET /api/v1/browser/start`

```python
def start_browser(user_id, headless=False):
    """Start browser profile and get automation endpoint"""
    response = requests.get(
        f"{API_BASE}/api/v1/browser/start",
        params={
            "user_id": user_id,
            "headless": 1 if headless else 0
        }
    )
    data = response.json()
    if data["code"] == 0:
        return {
            "webdriver": data["data"]["webdriver"],
            "ws_endpoint": data["data"]["ws"]["puppeteer"],
            "selenium_endpoint": data["data"]["ws"]["selenium"]
        }
    else:
        raise Exception(f"Failed to start browser: {data['msg']}")

# Launch browser
browser_info = start_browser(new_profile_id)
print(f"WebDriver path: {browser_info['webdriver']}")
print(f"Selenium endpoint: {browser_info['selenium_endpoint']}")
```

### 4. Control Browser with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(user_id):
    """Connect to AdsPower profile via Selenium"""
    browser_info = start_browser(user_id)
    
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", browser_info["selenium_endpoint"])
    
    driver = webdriver.Chrome(
        executable_path=browser_info["webdriver"],
        options=chrome_options
    )
    return driver

# Automate with Selenium
driver = connect_selenium("profile_id_here")
driver.get("https://www.facebook.com")
# Perform automation tasks
driver.quit()
```

### 5. Control Browser with Puppeteer

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const API_BASE = 'http://local.adspower.net:50325';

async function launchProfile(userId) {
    // Start browser profile
    const response = await axios.get(`${API_BASE}/api/v1/browser/start`, {
        params: { user_id: userId }
    });
    
    if (response.data.code !== 0) {
        throw new Error(`Failed to start: ${response.data.msg}`);
    }
    
    const { ws } = response.data.data;
    
    // Connect with Puppeteer
    const browser = await puppeteer.connect({
        browserWSEndpoint: ws.puppeteer,
        defaultViewport: null
    });
    
    return browser;
}

// Usage
(async () => {
    const browser = await launchProfile('profile_id_here');
    const page = await browser.newPage();
    await page.goto('https://www.instagram.com');
    
    // Automation logic here
    
    await browser.disconnect();
})();
```

### 6. Stop Browser Profile

**Endpoint**: `GET /api/v1/browser/stop`

```python
def stop_browser(user_id):
    """Stop running browser profile"""
    response = requests.get(
        f"{API_BASE}/api/v1/browser/stop",
        params={"user_id": user_id}
    )
    data = response.json()
    return data["code"] == 0

# Stop browser
stop_browser(new_profile_id)
```

### 7. Update Profile Configuration

**Endpoint**: `POST /api/v1/user/update`

```python
def update_profile(user_id, updates):
    """Update profile settings"""
    payload = {
        "user_id": user_id,
        **updates
    }
    
    response = requests.post(
        f"{API_BASE}/api/v1/user/update",
        json=payload
    )
    data = response.json()
    return data["code"] == 0

# Update profile proxy
update_profile("profile_id_here", {
    "user_proxy_config": {
        "proxy_soft": "other",
        "proxy_type": "socks5",
        "proxy_host": "new-proxy.example.com",
        "proxy_port": "1080"
    }
})
```

### 8. Delete Profile

**Endpoint**: `POST /api/v1/user/delete`

```python
def delete_profile(user_ids):
    """Delete one or more profiles"""
    response = requests.post(
        f"{API_BASE}/api/v1/user/delete",
        json={"user_ids": user_ids if isinstance(user_ids, list) else [user_ids]}
    )
    data = response.json()
    return data["code"] == 0

# Delete profile
delete_profile(["profile_id_1", "profile_id_2"])
```

## Common Automation Patterns

### Bulk Profile Creation for Campaign

```python
def create_campaign_profiles(campaign_name, count, proxies):
    """Create multiple profiles for a marketing campaign"""
    profile_ids = []
    
    for i in range(count):
        proxy = proxies[i % len(proxies)]
        
        profile_id = create_profile(
            name=f"{campaign_name} - Account {i+1}",
            proxy_config={
                "proxy_soft": "other",
                "proxy_type": proxy["type"],
                "proxy_host": proxy["host"],
                "proxy_port": proxy["port"],
                "proxy_user": proxy.get("user", ""),
                "proxy_password": proxy.get("password", "")
            }
        )
        profile_ids.append(profile_id)
        print(f"Created profile {i+1}/{count}: {profile_id}")
    
    return profile_ids

# Create 10 profiles with rotating proxies
proxies = [
    {"type": "http", "host": "proxy1.example.com", "port": "8080"},
    {"type": "http", "host": "proxy2.example.com", "port": "8080"},
    {"type": "socks5", "host": "proxy3.example.com", "port": "1080"}
]

campaign_profiles = create_campaign_profiles("Q1 Facebook Ads", 10, proxies)
```

### Parallel Browser Automation

```python
import concurrent.futures

def automate_profile(user_id, target_url, actions):
    """Automate single profile"""
    driver = connect_selenium(user_id)
    try:
        driver.get(target_url)
        for action in actions:
            action(driver)
        return {"user_id": user_id, "status": "success"}
    except Exception as e:
        return {"user_id": user_id, "status": "failed", "error": str(e)}
    finally:
        stop_browser(user_id)

def run_parallel_automation(profile_ids, target_url, actions, max_workers=5):
    """Run automation across multiple profiles simultaneously"""
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [
            executor.submit(automate_profile, pid, target_url, actions)
            for pid in profile_ids
        ]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]
    return results

# Define automation actions
def example_actions(driver):
    from selenium.webdriver.common.by import By
    driver.find_element(By.ID, "email").send_keys("user@example.com")
    driver.find_element(By.ID, "submit").click()

# Run on 10 profiles in parallel
results = run_parallel_automation(
    campaign_profiles[:10],
    "https://example.com/signup",
    [example_actions],
    max_workers=3
)
```

### Profile Group Management

```python
def create_profile_group(group_name):
    """Create a new profile group"""
    response = requests.post(
        f"{API_BASE}/api/v1/group/create",
        json={"group_name": group_name}
    )
    data = response.json()
    if data["code"] == 0:
        return data["data"]["group_id"]
    else:
        raise Exception(f"Failed to create group: {data['msg']}")

def assign_to_group(user_id, group_id):
    """Assign profile to a group"""
    return update_profile(user_id, {"group_id": group_id})

# Organize profiles
fb_group_id = create_profile_group("Facebook Campaigns")
for profile_id in campaign_profiles:
    assign_to_group(profile_id, fb_group_id)
```

## Environment Variables for Sensitive Data

Always use environment variables for credentials and API keys:

```python
import os

# Proxy credentials
PROXY_USER = os.getenv("ADSPOWER_PROXY_USER")
PROXY_PASSWORD = os.getenv("ADSPOWER_PROXY_PASSWORD")

# Account credentials for automation
FB_EMAIL = os.getenv("FB_ACCOUNT_EMAIL")
FB_PASSWORD = os.getenv("FB_ACCOUNT_PASSWORD")

proxy_config = {
    "proxy_soft": "other",
    "proxy_type": "http",
    "proxy_host": "proxy.example.com",
    "proxy_port": "8080",
    "proxy_user": PROXY_USER,
    "proxy_password": PROXY_PASSWORD
}
```

## Troubleshooting

### Browser Won't Start

**Issue**: API returns error when starting browser

**Solution**:
```python
# Check if browser is already running
response = requests.get(f"{API_BASE}/api/v1/browser/active", params={"user_id": user_id})
if response.json()["data"]["status"] == "Active":
    stop_browser(user_id)
    time.sleep(2)  # Wait for cleanup
    start_browser(user_id)
```

### Connection Timeout

**Issue**: Cannot connect to Local API

**Solution**:
- Verify AdsPower application is running
- Check API port in AdsPower settings (default: 50325)
- Test connectivity: `curl http://local.adspower.net:50325/status`

### Selenium Connection Failed

**Issue**: Cannot connect Selenium to browser

**Solution**:
```python
import time

def robust_selenium_connect(user_id, retries=3):
    """Connect with retry logic"""
    for attempt in range(retries):
        try:
            browser_info = start_browser(user_id)
            time.sleep(2)  # Wait for browser initialization
            
            chrome_options = Options()
            chrome_options.add_experimental_option(
                "debuggerAddress", 
                browser_info["selenium_endpoint"]
            )
            
            driver = webdriver.Chrome(
                executable_path=browser_info["webdriver"],
                options=chrome_options
            )
            return driver
        except Exception as e:
            if attempt == retries - 1:
                raise
            time.sleep(3)
```

### Fingerprint Detection

**Issue**: Accounts flagged despite using antidetect

**Solution**:
- Use unique proxies per profile
- Vary user behavior patterns (timing, mouse movements)
- Avoid launching too many profiles from same IP
- Randomize browser canvas and WebGL settings
- Use residential proxies instead of datacenter

### Memory Issues with Many Profiles

**Issue**: System slowdown with multiple browsers

**Solution**:
```python
# Close browsers after automation
def cleanup_profiles(profile_ids):
    """Stop all running browsers"""
    for pid in profile_ids:
        stop_browser(pid)
    time.sleep(1)

# Run in smaller batches
batch_size = 3
for i in range(0, len(profile_ids), batch_size):
    batch = profile_ids[i:i+batch_size]
    run_parallel_automation(batch, url, actions, max_workers=batch_size)
    cleanup_profiles(batch)
```

## Best Practices

1. **Profile Isolation**: Never reuse profiles across different platforms
2. **Proxy Rotation**: Use unique proxies per profile, prefer residential
3. **Human Behavior**: Add random delays, mouse movements, scroll patterns
4. **Cookie Management**: Import/export cookies for session persistence
5. **Batch Operations**: Limit concurrent browsers to prevent detection
6. **Error Handling**: Always stop browsers in finally blocks to prevent resource leaks
