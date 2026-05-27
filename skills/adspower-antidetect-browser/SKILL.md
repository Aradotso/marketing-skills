---
name: adspower-antidetect-browser
description: Use AdsPower antidetect browser for multi-account management, automation, and marketing campaigns with fingerprint protection
triggers:
  - how do I use AdsPower for multi-account management
  - set up AdsPower browser profiles for automation
  - create antidetect browser profiles with AdsPower
  - automate marketing campaigns with AdsPower
  - manage multiple accounts safely with AdsPower
  - integrate AdsPower API for browser automation
  - configure AdsPower profiles for RPA tasks
  - use AdsPower local API for profile control
---

# AdsPower Antidetect Browser

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for multi-account management, marketing automation, and RPA tasks. It allows teams to create isolated browser profiles with unique fingerprints, preventing account association and bans when managing multiple social media, e-commerce, or advertising accounts.

## Installation & Setup

AdsPower requires downloading the desktop application from their official website and optionally using the Local API for automation.

### Desktop Application

1. Download AdsPower from the official website
2. Install and launch the application
3. Create an account or log in
4. Configure your first browser profile through the UI

### Local API Access

AdsPower provides a Local API (REST) that runs on `http://localhost:50325` by default when the application is running.

Check API availability:
```bash
curl http://localhost:50325/status
```

## Core Concepts

- **Browser Profile**: An isolated browsing environment with unique fingerprints (canvas, WebGL, fonts, etc.)
- **Group**: Organizational unit for managing multiple profiles
- **Proxy**: IP address configuration for each profile
- **Fingerprint**: Digital identifier elements that can be customized to avoid detection

## Local API Usage

### Starting a Browser Profile

```python
import requests
import json

# AdsPower Local API endpoint
API_BASE = "http://localhost:50325"

def start_profile(profile_id):
    """Start a browser profile and get automation details"""
    url = f"{API_BASE}/api/v1/browser/start"
    params = {
        "user_id": profile_id,
        "open_tabs": 1  # Number of tabs to open
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data["code"] == 0:
        return {
            "selenium_address": data["data"]["ws"]["selenium"],
            "webdriver_path": data["data"]["webdriver"],
            "debug_port": data["data"]["debug_port"]
        }
    else:
        raise Exception(f"Failed to start profile: {data['msg']}")

# Example usage
profile_info = start_profile("abc123xyz")
print(f"Selenium address: {profile_info['selenium_address']}")
```

### Stopping a Browser Profile

```python
def stop_profile(profile_id):
    """Close a running browser profile"""
    url = f"{API_BASE}/api/v1/browser/stop"
    params = {"user_id": profile_id}
    
    response = requests.get(url, params=params)
    data = response.json()
    
    return data["code"] == 0

# Example usage
stop_profile("abc123xyz")
```

### Creating a New Profile

```python
def create_profile(profile_name, group_id=None, proxy_config=None):
    """Create a new browser profile"""
    url = f"{API_BASE}/api/v1/user/create"
    
    payload = {
        "name": profile_name,
        "group_id": group_id or "0",
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
    
    response = requests.post(url, json=payload)
    data = response.json()
    
    if data["code"] == 0:
        return data["data"]["id"]
    else:
        raise Exception(f"Failed to create profile: {data['msg']}")

# Example with proxy
proxy_config = {
    "proxy_soft": "other",
    "proxy_type": "http",
    "proxy_host": "proxy.example.com",
    "proxy_port": "8080",
    "proxy_user": "username",
    "proxy_password": "password"
}

new_profile_id = create_profile("Marketing Account 1", proxy_config=proxy_config)
```

### Listing Profiles

```python
def list_profiles(group_id=None, page=1, page_size=100):
    """Get list of browser profiles"""
    url = f"{API_BASE}/api/v1/user/list"
    
    params = {
        "page": page,
        "page_size": page_size
    }
    
    if group_id:
        params["group_id"] = group_id
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data["code"] == 0:
        return data["data"]["list"]
    else:
        raise Exception(f"Failed to list profiles: {data['msg']}")

# Example usage
profiles = list_profiles(page_size=50)
for profile in profiles:
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

## Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_to_adspower_profile(profile_id):
    """Connect Selenium to an AdsPower profile"""
    # Start the profile
    profile_info = start_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", profile_info["selenium_address"])
    
    # Create driver
    driver = webdriver.Chrome(
        executable_path=profile_info["webdriver_path"],
        options=chrome_options
    )
    
    return driver

# Example automation
driver = connect_to_adspower_profile("abc123xyz")
driver.get("https://example.com")
# Perform automation tasks
driver.quit()
stop_profile("abc123xyz")
```

## Playwright Integration

```python
from playwright.sync_api import sync_playwright

def connect_playwright_to_adspower(profile_id):
    """Connect Playwright to an AdsPower profile"""
    profile_info = start_profile(profile_id)
    
    # Extract debug port from selenium address
    debug_port = profile_info["debug_port"]
    
    with sync_playwright() as p:
        browser = p.chromium.connect_over_cdp(f"http://localhost:{debug_port}")
        context = browser.contexts[0]
        page = context.pages[0] if context.pages else context.new_page()
        
        # Perform automation
        page.goto("https://example.com")
        # ... automation tasks ...
        
        browser.close()
    
    stop_profile(profile_id)

# Example usage
connect_playwright_to_adspower("abc123xyz")
```

## Profile Management Patterns

### Bulk Profile Creation

```python
def create_bulk_profiles(profile_configs):
    """Create multiple profiles from configuration list"""
    created_profiles = []
    
    for config in profile_configs:
        try:
            profile_id = create_profile(
                profile_name=config["name"],
                group_id=config.get("group_id"),
                proxy_config=config.get("proxy")
            )
            created_profiles.append({
                "id": profile_id,
                "name": config["name"]
            })
            print(f"✓ Created profile: {config['name']}")
        except Exception as e:
            print(f"✗ Failed to create {config['name']}: {str(e)}")
    
    return created_profiles

# Example configuration
configs = [
    {
        "name": "Facebook Account 1",
        "group_id": "social_media_group",
        "proxy": {
            "proxy_soft": "other",
            "proxy_type": "socks5",
            "proxy_host": "proxy1.example.com",
            "proxy_port": "1080"
        }
    },
    {
        "name": "Facebook Account 2",
        "group_id": "social_media_group",
        "proxy": {
            "proxy_soft": "other",
            "proxy_type": "socks5",
            "proxy_host": "proxy2.example.com",
            "proxy_port": "1080"
        }
    }
]

bulk_create_profiles(configs)
```

### Profile Health Check

```python
def check_profile_status(profile_id):
    """Check if a profile is currently running"""
    url = f"{API_BASE}/api/v1/browser/active"
    params = {"user_id": profile_id}
    
    response = requests.get(url, params=params)
    data = response.json()
    
    return data["data"]["status"] == "Active"

def check_all_profiles_health():
    """Check status of all profiles"""
    profiles = list_profiles()
    status_report = []
    
    for profile in profiles:
        is_active = check_profile_status(profile["user_id"])
        status_report.append({
            "name": profile["name"],
            "id": profile["user_id"],
            "active": is_active
        })
    
    return status_report
```

## Configuration & Environment Variables

Store sensitive configuration in environment variables:

```python
import os

# API configuration
ADSPOWER_API_HOST = os.getenv("ADSPOWER_API_HOST", "http://localhost:50325")
ADSPOWER_API_KEY = os.getenv("ADSPOWER_API_KEY")  # If using cloud API

# Proxy configuration
PROXY_HOST = os.getenv("PROXY_HOST")
PROXY_PORT = os.getenv("PROXY_PORT")
PROXY_USER = os.getenv("PROXY_USER")
PROXY_PASSWORD = os.getenv("PROXY_PASSWORD")

def get_proxy_config_from_env():
    """Build proxy configuration from environment variables"""
    return {
        "proxy_soft": "other",
        "proxy_type": os.getenv("PROXY_TYPE", "http"),
        "proxy_host": PROXY_HOST,
        "proxy_port": PROXY_PORT,
        "proxy_user": PROXY_USER,
        "proxy_password": PROXY_PASSWORD
    }
```

## Common Troubleshooting

### Profile Won't Start

```python
import time

def start_profile_with_retry(profile_id, max_retries=3):
    """Start profile with retry logic"""
    for attempt in range(max_retries):
        try:
            return start_profile(profile_id)
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {str(e)}")
            if attempt < max_retries - 1:
                time.sleep(2)
            else:
                raise
```

### Clean Up Stuck Profiles

```python
def force_stop_all_profiles():
    """Force stop all running profiles"""
    profiles = list_profiles()
    
    for profile in profiles:
        if check_profile_status(profile["user_id"]):
            try:
                stop_profile(profile["user_id"])
                print(f"Stopped profile: {profile['name']}")
            except Exception as e:
                print(f"Failed to stop {profile['name']}: {str(e)}")
```

### API Connection Issues

```bash
# Check if AdsPower is running
curl http://localhost:50325/status

# Verify API port in AdsPower settings (default: 50325)
# Settings > General > Local API Port
```

## Best Practices

1. **Always stop profiles after use** to free system resources
2. **Use unique proxies** for each profile to maintain isolation
3. **Implement error handling** for all API calls
4. **Store profile IDs** in a database for automation workflows
5. **Use groups** to organize profiles by campaign or platform
6. **Monitor profile health** regularly in production environments
7. **Rotate fingerprints** periodically to avoid pattern detection
