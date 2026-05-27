---
name: adspower-antidetect-browser
description: AdsPower antidetect browser automation for multi-account management and marketing campaigns
triggers:
  - how do I automate AdsPower browser profiles
  - create multiple browser profiles for marketing
  - manage AdsPower profiles via API
  - automate multi-account campaigns with AdsPower
  - set up antidetect browser automation
  - control AdsPower browser fingerprints
  - launch AdsPower profiles programmatically
  - integrate AdsPower with marketing automation
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that allows marketing teams to manage multiple browser profiles with unique fingerprints for multi-account operations, affiliate marketing, e-commerce, and social media management. This skill covers automating AdsPower through its local API for profile management, browser control, and campaign automation.

## What AdsPower Does

- **Multi-Account Management**: Create and manage hundreds of isolated browser profiles
- **Fingerprint Protection**: Each profile has unique browser fingerprints (canvas, WebGL, fonts, etc.)
- **Team Collaboration**: Share profiles and manage permissions across team members
- **Automation Ready**: Local API for programmatic control of browsers and profiles
- **Proxy Integration**: Support for HTTP, SOCKS5, and SSH proxies per profile
- **Cookie/Session Management**: Import/export cookies and maintain sessions

## Installation & Setup

### Prerequisites

1. Download and install AdsPower application from official website
2. Launch AdsPower and ensure it's running (default port: 50325)
3. Enable API access in AdsPower settings

### Verify API Access

```bash
# Check if AdsPower API is accessible
curl http://localhost:50325/api/v1/status
```

Expected response:
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "status": "running"
  }
}
```

## Core API Endpoints

### Base Configuration

```python
import requests
import os

# AdsPower local API base URL
ADSPOWER_API = "http://localhost:50325/api/v1"

class AdsPowerClient:
    def __init__(self, base_url=ADSPOWER_API):
        self.base_url = base_url
        self.session = requests.Session()
    
    def _request(self, method, endpoint, **kwargs):
        url = f"{self.base_url}{endpoint}"
        response = self.session.request(method, url, **kwargs)
        response.raise_for_status()
        return response.json()
    
    def get(self, endpoint, **kwargs):
        return self._request("GET", endpoint, **kwargs)
    
    def post(self, endpoint, **kwargs):
        return self._request("POST", endpoint, **kwargs)
```

## Profile Management

### Create a New Profile

```python
def create_profile(client, name, proxy_config=None):
    """Create a new browser profile with optional proxy"""
    payload = {
        "name": name,
        "group_id": "0",  # Default group
        "domain_name": "",
        "open_urls": [],
        "repeat_config": ["0"],
        "username": "",
        "password": "",
        "fakey": "",
        "cookie": [],
        "ignore_cookie_error": 1,
        "ip": "",
        "country": "",
        "region": "",
        "city": "",
        "remark": ""
    }
    
    if proxy_config:
        payload.update({
            "user_proxy_config": {
                "proxy_soft": "other",
                "proxy_type": proxy_config.get("type", "http"),
                "proxy_host": proxy_config["host"],
                "proxy_port": proxy_config["port"],
                "proxy_user": proxy_config.get("username", ""),
                "proxy_password": proxy_config.get("password", "")
            }
        })
    
    response = client.post("/user/create", json=payload)
    
    if response["code"] == 0:
        return response["data"]["id"]
    else:
        raise Exception(f"Failed to create profile: {response['msg']}")

# Example usage
client = AdsPowerClient()

# Create profile without proxy
profile_id = create_profile(client, "Marketing Campaign 1")

# Create profile with proxy
proxy = {
    "type": "http",
    "host": os.getenv("PROXY_HOST"),
    "port": os.getenv("PROXY_PORT"),
    "username": os.getenv("PROXY_USER"),
    "password": os.getenv("PROXY_PASS")
}
profile_id_with_proxy = create_profile(client, "Marketing Campaign 2", proxy)
```

### List All Profiles

```python
def list_profiles(client, page=1, page_size=100, group_id=None):
    """List all browser profiles"""
    params = {
        "page": page,
        "page_size": page_size
    }
    
    if group_id:
        params["group_id"] = group_id
    
    response = client.get("/user/list", params=params)
    
    if response["code"] == 0:
        return response["data"]["list"]
    else:
        raise Exception(f"Failed to list profiles: {response['msg']}")

# Get all profiles
profiles = list_profiles(client)
for profile in profiles:
    print(f"ID: {profile['user_id']}, Name: {profile['name']}")
```

### Update Profile Configuration

```python
def update_profile(client, profile_id, updates):
    """Update profile settings"""
    payload = {
        "user_id": profile_id,
        **updates
    }
    
    response = client.post("/user/update", json=payload)
    
    if response["code"] == 0:
        return True
    else:
        raise Exception(f"Failed to update profile: {response['msg']}")

# Update profile name and remark
update_profile(client, profile_id, {
    "name": "Updated Campaign Name",
    "remark": "Facebook Ads Campaign Q1"
})
```

### Delete Profile

```python
def delete_profile(client, profile_id):
    """Delete a browser profile"""
    response = client.post("/user/delete", json={"user_ids": [profile_id]})
    
    if response["code"] == 0:
        return True
    else:
        raise Exception(f"Failed to delete profile: {response['msg']}")

# Delete a profile
delete_profile(client, profile_id)
```

## Browser Control

### Start Browser Instance

```python
def start_browser(client, profile_id, headless=False):
    """Launch browser for a profile"""
    params = {
        "user_id": profile_id,
        "open_tabs": 1,
        "headless": 1 if headless else 0
    }
    
    response = client.get("/browser/start", params=params)
    
    if response["code"] == 0:
        return {
            "webdriver": response["data"]["webdriver"],
            "debug_port": response["data"]["debug_port"],
            "ws_endpoint": response["data"]["ws"]["selenium"]
        }
    else:
        raise Exception(f"Failed to start browser: {response['msg']}")

# Start browser in normal mode
browser_info = start_browser(client, profile_id)
print(f"WebDriver path: {browser_info['webdriver']}")
print(f"Debug port: {browser_info['debug_port']}")
```

### Stop Browser Instance

```python
def stop_browser(client, profile_id):
    """Close browser for a profile"""
    response = client.get("/browser/stop", params={"user_id": profile_id})
    
    if response["code"] == 0:
        return True
    else:
        raise Exception(f"Failed to stop browser: {response['msg']}")

# Stop browser
stop_browser(client, profile_id)
```

### Check Browser Status

```python
def check_browser_active(client, profile_id):
    """Check if browser is currently active"""
    response = client.get("/browser/active", params={"user_id": profile_id})
    
    if response["code"] == 0:
        return response["data"]["status"] == "Active"
    else:
        return False

# Check if browser is running
is_active = check_browser_active(client, profile_id)
print(f"Browser active: {is_active}")
```

## Selenium Integration

### Launch Browser with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def launch_with_selenium(client, profile_id):
    """Launch AdsPower profile with Selenium control"""
    browser_info = start_browser(client, profile_id)
    
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", f"127.0.0.1:{browser_info['debug_port']}")
    
    driver = webdriver.Chrome(options=chrome_options)
    return driver

# Use with Selenium
driver = launch_with_selenium(client, profile_id)
driver.get("https://www.example.com")

# Perform automation
title = driver.title
print(f"Page title: {title}")

# Close when done
driver.quit()
stop_browser(client, profile_id)
```

## Group Management

### Create Profile Group

```python
def create_group(client, group_name, remark=""):
    """Create a new profile group"""
    response = client.post("/group/create", json={
        "group_name": group_name,
        "remark": remark
    })
    
    if response["code"] == 0:
        return response["data"]["group_id"]
    else:
        raise Exception(f"Failed to create group: {response['msg']}")

# Create group for campaigns
group_id = create_group(client, "Facebook Campaigns", "All FB ad accounts")
```

### List Groups

```python
def list_groups(client):
    """List all profile groups"""
    response = client.get("/group/list")
    
    if response["code"] == 0:
        return response["data"]["list"]
    else:
        raise Exception(f"Failed to list groups: {response['msg']}")

# Get all groups
groups = list_groups(client)
for group in groups:
    print(f"Group: {group['group_name']}, ID: {group['group_id']}")
```

## Common Automation Patterns

### Batch Profile Creation

```python
def create_bulk_profiles(client, count, name_prefix, group_id=None, proxies=None):
    """Create multiple profiles at once"""
    profile_ids = []
    
    for i in range(count):
        name = f"{name_prefix} {i+1}"
        proxy = proxies[i] if proxies and i < len(proxies) else None
        
        profile_id = create_profile(client, name, proxy)
        
        if group_id:
            update_profile(client, profile_id, {"group_id": group_id})
        
        profile_ids.append(profile_id)
        print(f"Created profile {i+1}/{count}: {profile_id}")
    
    return profile_ids

# Create 10 profiles for a campaign
profile_ids = create_bulk_profiles(
    client, 
    count=10, 
    name_prefix="Instagram Campaign",
    group_id=group_id
)
```

### Sequential Browser Automation

```python
import time

def automate_profiles_sequentially(client, profile_ids, automation_func):
    """Run automation on multiple profiles one by one"""
    results = []
    
    for profile_id in profile_ids:
        try:
            driver = launch_with_selenium(client, profile_id)
            result = automation_func(driver)
            results.append({"profile_id": profile_id, "success": True, "result": result})
            driver.quit()
            stop_browser(client, profile_id)
            time.sleep(2)  # Delay between profiles
        except Exception as e:
            results.append({"profile_id": profile_id, "success": False, "error": str(e)})
            stop_browser(client, profile_id)
    
    return results

# Example automation function
def check_login_status(driver):
    driver.get("https://www.facebook.com")
    time.sleep(3)
    return "logged_in" if "login" not in driver.current_url.lower() else "logged_out"

# Run on all profiles
results = automate_profiles_sequentially(client, profile_ids, check_login_status)
```

### Cookie Management

```python
import json

def export_cookies(client, profile_id):
    """Export cookies from a profile"""
    response = client.get("/user/detail", params={"user_id": profile_id})
    
    if response["code"] == 0:
        return response["data"]["cookie"]
    else:
        raise Exception(f"Failed to export cookies: {response['msg']}")

def import_cookies(client, profile_id, cookies):
    """Import cookies to a profile"""
    update_profile(client, profile_id, {"cookie": cookies})

# Export cookies from one profile
cookies = export_cookies(client, profile_ids[0])

# Import to another profile
import_cookies(client, profile_ids[1], cookies)
```

## Troubleshooting

### API Connection Issues

```python
def verify_adspower_running():
    """Check if AdsPower is running and API is accessible"""
    try:
        response = requests.get(f"{ADSPOWER_API}/status", timeout=5)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        return False

if not verify_adspower_running():
    print("Error: AdsPower is not running or API is not accessible")
    print("1. Make sure AdsPower application is running")
    print("2. Check if API is enabled in AdsPower settings")
    print("3. Verify port 50325 is not blocked")
```

### Browser Launch Failures

Common issues and solutions:

1. **Profile already has active browser**: Stop existing browser first
2. **Port conflict**: Check if debug port is already in use
3. **Insufficient resources**: Close unused browsers or increase system resources

```python
def safe_start_browser(client, profile_id, max_retries=3):
    """Start browser with retry logic"""
    for attempt in range(max_retries):
        try:
            # Stop any existing browser
            stop_browser(client, profile_id)
            time.sleep(1)
            
            # Start new instance
            return start_browser(client, profile_id)
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2)
```

### Response Code Reference

- `0`: Success
- `-1`: General error
- `10001`: Profile not found
- `10002`: Browser already running
- `10003`: Browser not running

```python
def handle_api_response(response):
    """Parse and handle API response codes"""
    code = response.get("code")
    msg = response.get("msg", "Unknown error")
    
    if code == 0:
        return response["data"]
    elif code == 10001:
        raise ValueError(f"Profile not found: {msg}")
    elif code == 10002:
        raise RuntimeError(f"Browser already running: {msg}")
    elif code == 10003:
        raise RuntimeError(f"Browser not running: {msg}")
    else:
        raise Exception(f"API error (code {code}): {msg}")
```

## Environment Variables

Store sensitive credentials as environment variables:

```bash
# .env file
PROXY_HOST=proxy.example.com
PROXY_PORT=8080
PROXY_USER=username
PROXY_PASS=password
ADSPOWER_API_URL=http://localhost:50325/api/v1
```

Load in Python:

```python
from dotenv import load_dotenv
load_dotenv()

ADSPOWER_API = os.getenv("ADSPOWER_API_URL", "http://localhost:50325/api/v1")
```
