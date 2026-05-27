---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaign management
triggers:
  - "set up adspower browser profiles"
  - "create antidetect browser automation"
  - "manage multiple account profiles"
  - "configure adspower for marketing campaigns"
  - "automate browser fingerprinting with adspower"
  - "use adspower api for profile management"
  - "integrate adspower with automation scripts"
  - "handle multi-account browser sessions"
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

AdsPower is an antidetect browser platform designed for managing multiple browser profiles with unique fingerprints. It's commonly used for marketing automation, social media management, affiliate marketing, e-commerce operations, and any scenario requiring isolated browser sessions with different digital identities.

Key features:
- Create and manage isolated browser profiles with unique fingerprints
- API-driven automation for profile creation and control
- RPA (Robotic Process Automation) capabilities
- Cloud synchronization for team collaboration
- Multi-account management without detection
- Selenium/Puppeteer integration support

## Installation

AdsPower requires both the desktop application and API access for automation:

1. **Download AdsPower Application**
   - Visit official AdsPower website and download the desktop client
   - Install and create an account
   - Launch the application to enable local API server (typically runs on `http://localhost:50325`)

2. **Verify API Access**
   ```bash
   curl http://localhost:50325/api/v1/status
   ```

## API Configuration

AdsPower's local API server runs automatically when the application is open. Default endpoint:

```
http://localhost:50325
```

For cloud deployments, configure the remote API endpoint:

```bash
export ADSPOWER_API_URL="http://localhost:50325"
export ADSPOWER_API_KEY="your_api_key_here"  # If using cloud version
```

## Core API Operations

### Browser Profile Management

#### Create a New Profile

```python
import requests
import json

ADSPOWER_API = "http://localhost:50325"

def create_profile(name, group_id=0):
    """Create a new browser profile"""
    url = f"{ADSPOWER_API}/api/v1/user/create"
    
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
        "country": "us",
        "region": "",
        "city": "",
        "remark": "",
        "ipchecker": "",
        "sys_app_cate_id": 0,
        "user_proxy_config": {
            "proxy_soft": "no_proxy",
            "proxy_type": "noproxy"
        },
        "fingerprint_config": {
            "automatic_timezone": 1,
            "language": ["en-US"],
            "page_language": "en-US",
            "ua": "default",
            "screen_resolution": "default",
            "fonts": ["all"],
            "canvas": "1",
            "webgl_image": "1",
            "webgl": "1",
            "audio": "1",
            "do_not_track": "default",
            "hardware_concurrency": "default",
            "device_memory": "default",
            "flash": "block",
            "scan_port_type": "1",
            "allow_scan_ports": "",
            "media_devices": "1",
            "client_rects": "1",
            "device_name_switch": "1",
            "random_ua": "1"
        }
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Example usage
result = create_profile("Campaign Profile 1", group_id=0)
print(f"Profile created: {result}")
profile_id = result.get("data", {}).get("id")
```

#### List All Profiles

```python
def list_profiles(page=1, page_size=100):
    """Retrieve all browser profiles"""
    url = f"{ADSPOWER_API}/api/v1/user/list"
    
    params = {
        "page": page,
        "page_size": page_size
    }
    
    response = requests.get(url, params=params)
    return response.json()

# Get all profiles
profiles = list_profiles()
for profile in profiles.get("data", {}).get("list", []):
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

#### Start a Browser Profile

```python
def start_profile(profile_id, headless=False):
    """Launch a browser profile and get automation details"""
    url = f"{ADSPOWER_API}/api/v1/browser/start"
    
    params = {
        "user_id": profile_id,
        "open_tabs": 1,
        "headless": 1 if headless else 0
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data.get("code") == 0:
        return {
            "selenium_address": data["data"]["ws"]["selenium"],
            "webdriver_path": data["data"]["webdriver"],
            "debug_port": data["data"].get("debug_port")
        }
    return None

# Start browser
browser_info = start_profile(profile_id)
print(f"Selenium address: {browser_info['selenium_address']}")
```

#### Stop a Browser Profile

```python
def stop_profile(profile_id):
    """Close a running browser profile"""
    url = f"{ADSPOWER_API}/api/v1/browser/stop"
    
    params = {
        "user_id": profile_id
    }
    
    response = requests.get(url, params=params)
    return response.json()

# Stop browser
stop_profile(profile_id)
```

### Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium_to_profile(profile_id):
    """Connect Selenium to an AdsPower profile"""
    # Start the profile
    browser_info = start_profile(profile_id)
    
    if not browser_info:
        raise Exception("Failed to start profile")
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", 
                                          browser_info["selenium_address"].replace("http://", ""))
    
    # Connect to the browser
    driver = webdriver.Chrome(
        executable_path=browser_info["webdriver_path"],
        options=chrome_options
    )
    
    return driver

# Example automation
driver = connect_selenium_to_profile(profile_id)
driver.get("https://example.com")
print(driver.title)
driver.quit()

# Don't forget to stop the profile
stop_profile(profile_id)
```

### Puppeteer Integration (Node.js)

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const ADSPOWER_API = 'http://localhost:50325';

async function startProfile(profileId) {
  const response = await axios.get(`${ADSPOWER_API}/api/v1/browser/start`, {
    params: {
      user_id: profileId,
      open_tabs: 1
    }
  });
  
  return response.data.data;
}

async function connectPuppeteer(profileId) {
  const browserData = await startProfile(profileId);
  
  const browser = await puppeteer.connect({
    browserWSEndpoint: browserData.ws.puppeteer,
    defaultViewport: null
  });
  
  return browser;
}

// Usage
(async () => {
  const browser = await connectPuppeteer('your_profile_id');
  const page = await browser.newPage();
  await page.goto('https://example.com');
  console.log(await page.title());
  await browser.disconnect();
  
  // Stop the profile
  await axios.get(`${ADSPOWER_API}/api/v1/browser/stop`, {
    params: { user_id: 'your_profile_id' }
  });
})();
```

## Profile Configuration

### Configure Proxy Settings

```python
def update_profile_proxy(profile_id, proxy_config):
    """Update proxy settings for a profile"""
    url = f"{ADSPOWER_API}/api/v1/user/update"
    
    payload = {
        "user_id": profile_id,
        "user_proxy_config": proxy_config
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Example: HTTP proxy
http_proxy_config = {
    "proxy_soft": "other",
    "proxy_type": "http",
    "proxy_host": "proxy.example.com",
    "proxy_port": "8080",
    "proxy_user": "username",
    "proxy_password": "password"
}

# Example: SOCKS5 proxy
socks5_proxy_config = {
    "proxy_soft": "other",
    "proxy_type": "socks5",
    "proxy_host": "socks.example.com",
    "proxy_port": "1080"
}

update_profile_proxy(profile_id, http_proxy_config)
```

### Update Fingerprint Settings

```python
def update_fingerprint(profile_id, fingerprint_config):
    """Update fingerprint configuration"""
    url = f"{ADSPOWER_API}/api/v1/user/update"
    
    payload = {
        "user_id": profile_id,
        "fingerprint_config": fingerprint_config
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Customize fingerprint
custom_fingerprint = {
    "automatic_timezone": 1,
    "language": ["en-US", "en"],
    "page_language": "en-US",
    "ua": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
    "screen_resolution": "1920x1080",
    "webgl": "1",
    "canvas": "1",
    "audio": "1"
}

update_fingerprint(profile_id, custom_fingerprint)
```

## Common Automation Patterns

### Bulk Profile Creation

```python
def create_bulk_profiles(count, name_prefix, group_id=0):
    """Create multiple profiles for campaigns"""
    profiles = []
    
    for i in range(count):
        name = f"{name_prefix}_{i+1}"
        result = create_profile(name, group_id)
        
        if result.get("code") == 0:
            profile_id = result["data"]["id"]
            profiles.append({
                "id": profile_id,
                "name": name
            })
            print(f"Created profile: {name}")
        else:
            print(f"Failed to create profile {name}: {result.get('msg')}")
    
    return profiles

# Create 10 profiles for a campaign
campaign_profiles = create_bulk_profiles(10, "Campaign_A", group_id=5)
```

### Profile Rotation for Automation

```python
import time

def rotate_profiles(profile_ids, task_function):
    """Rotate through profiles and execute tasks"""
    for profile_id in profile_ids:
        print(f"Starting profile: {profile_id}")
        
        # Start browser
        browser_info = start_profile(profile_id)
        
        if browser_info:
            try:
                # Execute task
                task_function(browser_info, profile_id)
            except Exception as e:
                print(f"Task failed for profile {profile_id}: {e}")
            finally:
                # Stop browser
                stop_profile(profile_id)
                time.sleep(5)  # Delay between profiles

def my_automation_task(browser_info, profile_id):
    """Example task to run on each profile"""
    driver = webdriver.Chrome(
        executable_path=browser_info["webdriver_path"],
        options=get_chrome_options(browser_info["selenium_address"])
    )
    
    driver.get("https://example.com")
    # Perform actions...
    driver.quit()

# Run tasks on multiple profiles
profile_ids = ["id1", "id2", "id3"]
rotate_profiles(profile_ids, my_automation_task)
```

### Profile Group Management

```python
def create_group(group_name, remark=""):
    """Create a profile group for organization"""
    url = f"{ADSPOWER_API}/api/v1/group/create"
    
    payload = {
        "group_name": group_name,
        "remark": remark
    }
    
    response = requests.post(url, json=payload)
    return response.json()

def list_groups():
    """List all profile groups"""
    url = f"{ADSPOWER_API}/api/v1/group/list"
    response = requests.get(url)
    return response.json()

# Create groups for campaigns
facebook_group = create_group("Facebook Campaigns", "FB marketing profiles")
instagram_group = create_group("Instagram Campaigns", "IG automation")
```

## Troubleshooting

### API Connection Issues

```python
def check_api_status():
    """Verify AdsPower API is running"""
    try:
        response = requests.get(f"{ADSPOWER_API}/api/v1/status", timeout=5)
        if response.status_code == 200:
            print("API is running")
            return True
    except requests.exceptions.RequestException as e:
        print(f"API connection failed: {e}")
        print("Make sure AdsPower application is running")
        return False

check_api_status()
```

### Profile Startup Failures

- **Profile already running**: Stop the profile first before starting again
- **Port conflicts**: Check if debug ports are already in use
- **License issues**: Verify your AdsPower subscription supports the number of profiles

```python
def safe_start_profile(profile_id):
    """Start profile with error handling"""
    # Try to stop first in case it's running
    stop_profile(profile_id)
    time.sleep(2)
    
    # Now start
    browser_info = start_profile(profile_id)
    if not browser_info:
        print(f"Failed to start profile {profile_id}")
        return None
    return browser_info
```

### Resource Cleanup

```python
import atexit

active_profiles = []

def cleanup_profiles():
    """Stop all active profiles on exit"""
    for profile_id in active_profiles:
        try:
            stop_profile(profile_id)
            print(f"Stopped profile: {profile_id}")
        except Exception as e:
            print(f"Failed to stop {profile_id}: {e}")

# Register cleanup
atexit.register(cleanup_profiles)
```

## Best Practices

1. **Always stop profiles after use** to free resources
2. **Use groups** to organize profiles by campaign or platform
3. **Rotate user agents and fingerprints** for better anonymity
4. **Implement delays** between profile switches to avoid detection
5. **Monitor API rate limits** if using cloud version
6. **Store profile IDs** in a database for campaign tracking
7. **Use environment variables** for sensitive configuration
8. **Test profiles individually** before bulk operations

## Environment Variables

```bash
# API configuration
export ADSPOWER_API_URL="http://localhost:50325"
export ADSPOWER_API_KEY=""  # For cloud version

# Proxy configuration (if using centralized proxy)
export PROXY_HOST="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USER="username"
export PROXY_PASS="password"
```
