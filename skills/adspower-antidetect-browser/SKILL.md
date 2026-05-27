---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - create adspower browser profile
  - manage multiple browser accounts with adspower
  - automate adspower profiles for marketing
  - set up antidetect browser fingerprints
  - control adspower api for automation
  - launch adspower browser instances
  - configure adspower profile settings
  - integrate adspower with marketing automation
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple browser profiles with unique fingerprints. It enables marketing teams to run multi-account campaigns, automate social media management, and perform web scraping while avoiding detection and account bans.

## Overview

AdsPower provides:
- Isolated browser profiles with unique fingerprints (user agents, canvas, WebGL, fonts, etc.)
- Cloud-based profile synchronization
- API for automation and RPA workflows
- Multi-account management for marketing campaigns
- Team collaboration features

## Installation

AdsPower is a desktop application with API access:

1. Download and install AdsPower from the official website
2. Launch the application and create an account
3. Obtain your API key from Settings > API

## API Configuration

The AdsPower API runs locally on your machine (default: `http://local.adspower.net:50325`).

### Authentication

Store your API credentials as environment variables:

```bash
export ADSPOWER_API_KEY="your_api_key_here"
export ADSPOWER_API_URL="http://local.adspower.net:50325"
```

## Core API Endpoints

### Profile Management

#### List All Profiles

```python
import requests
import os

API_URL = os.getenv("ADSPOWER_API_URL", "http://local.adspower.net:50325")

def list_profiles(page=1, page_size=100):
    """Retrieve all browser profiles"""
    response = requests.get(
        f"{API_URL}/api/v1/user/list",
        params={
            "page_size": page_size,
            "page": page
        }
    )
    return response.json()

profiles = list_profiles()
print(f"Total profiles: {profiles['data']['total']}")
for profile in profiles['data']['list']:
    print(f"ID: {profile['user_id']}, Name: {profile['name']}")
```

#### Create New Profile

```python
def create_profile(name, group_id=None, fingerprint_config=None):
    """Create a new browser profile with custom fingerprint"""
    payload = {
        "name": name,
        "group_id": group_id or "0",
        "fingerprint_config": fingerprint_config or {}
    }
    
    response = requests.post(
        f"{API_URL}/api/v1/user/create",
        json=payload
    )
    return response.json()

# Create profile with custom settings
new_profile = create_profile(
    name="Marketing Campaign Profile 1",
    fingerprint_config={
        "automatic_timezone": 1,
        "webrtc": "proxy",
        "location": "ask",
        "ua": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    }
)
print(f"Created profile: {new_profile['data']['id']}")
```

#### Start Browser Profile

```python
def start_browser(profile_id, headless=False):
    """Launch a browser profile and get automation endpoint"""
    response = requests.get(
        f"{API_URL}/api/v1/browser/start",
        params={
            "user_id": profile_id,
            "headless": 1 if headless else 0
        }
    )
    data = response.json()
    
    if data['code'] == 0:
        return {
            "selenium": data['data']['ws']['selenium'],
            "puppeteer": data['data']['ws']['puppeteer'],
            "webdriver": data['data']['webdriver']
        }
    return None

connection_info = start_browser("profile_id_here")
print(f"Selenium endpoint: {connection_info['selenium']}")
```

#### Stop Browser Profile

```python
def stop_browser(profile_id):
    """Close a running browser profile"""
    response = requests.get(
        f"{API_URL}/api/v1/browser/stop",
        params={"user_id": profile_id}
    )
    return response.json()

stop_browser("profile_id_here")
```

### Integration with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(profile_id):
    """Connect Selenium to AdsPower profile"""
    # Start the browser
    connection = start_browser(profile_id)
    
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress", 
        connection['selenium'].replace("http://", "")
    )
    
    driver = webdriver.Chrome(options=chrome_options)
    return driver

# Usage
driver = connect_selenium("your_profile_id")
driver.get("https://example.com")
# Perform automation tasks
driver.quit()
```

### Integration with Puppeteer

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const API_URL = process.env.ADSPOWER_API_URL || 'http://local.adspower.net:50325';

async function startBrowser(profileId) {
    const response = await axios.get(`${API_URL}/api/v1/browser/start`, {
        params: { user_id: profileId }
    });
    return response.data.data.ws.puppeteer;
}

async function connectPuppeteer(profileId) {
    const wsEndpoint = await startBrowser(profileId);
    
    const browser = await puppeteer.connect({
        browserWSEndpoint: wsEndpoint,
        defaultViewport: null
    });
    
    return browser;
}

// Usage
(async () => {
    const browser = await connectPuppeteer('your_profile_id');
    const pages = await browser.pages();
    const page = pages[0] || await browser.newPage();
    
    await page.goto('https://example.com');
    // Perform automation
    
    await browser.disconnect();
})();
```

## Common Marketing Automation Patterns

### Multi-Account Campaign Management

```python
import time

def run_campaign_on_profiles(profile_ids, campaign_function):
    """Execute marketing campaign across multiple profiles"""
    results = []
    
    for profile_id in profile_ids:
        try:
            driver = connect_selenium(profile_id)
            result = campaign_function(driver)
            results.append({
                "profile_id": profile_id,
                "status": "success",
                "result": result
            })
            driver.quit()
            stop_browser(profile_id)
            time.sleep(5)  # Rate limiting
        except Exception as e:
            results.append({
                "profile_id": profile_id,
                "status": "error",
                "error": str(e)
            })
    
    return results

def post_to_social_media(driver):
    """Example campaign function"""
    driver.get("https://socialmedia.com/post")
    # Automation logic here
    return "Posted successfully"

# Run campaign
profile_list = ["profile_1", "profile_2", "profile_3"]
campaign_results = run_campaign_on_profiles(profile_list, post_to_social_media)
```

### Profile Group Management

```python
def create_profile_group(group_name):
    """Create a group for organizing profiles"""
    response = requests.post(
        f"{API_URL}/api/v1/group/create",
        json={"group_name": group_name}
    )
    return response.json()

def bulk_create_profiles(count, group_id, base_name="Profile"):
    """Create multiple profiles in a group"""
    profile_ids = []
    
    for i in range(count):
        profile = create_profile(
            name=f"{base_name} {i+1}",
            group_id=group_id
        )
        if profile['code'] == 0:
            profile_ids.append(profile['data']['id'])
    
    return profile_ids

# Create campaign group and profiles
group = create_profile_group("Summer Campaign 2026")
profiles = bulk_create_profiles(10, group['data']['group_id'], "Summer")
```

## Proxy Configuration

```python
def update_profile_proxy(profile_id, proxy_config):
    """Set proxy for a profile"""
    payload = {
        "user_id": profile_id,
        "user_proxy_config": {
            "proxy_type": proxy_config.get("type", "http"),
            "proxy_host": proxy_config["host"],
            "proxy_port": proxy_config["port"],
            "proxy_user": proxy_config.get("username", ""),
            "proxy_password": proxy_config.get("password", "")
        }
    }
    
    response = requests.post(
        f"{API_URL}/api/v1/user/update",
        json=payload
    )
    return response.json()

# Set proxy from environment variables
proxy_settings = {
    "type": "socks5",
    "host": os.getenv("PROXY_HOST"),
    "port": os.getenv("PROXY_PORT"),
    "username": os.getenv("PROXY_USER"),
    "password": os.getenv("PROXY_PASS")
}

update_profile_proxy("profile_id", proxy_settings)
```

## Troubleshooting

### Browser Won't Start

```python
def check_browser_status(profile_id):
    """Check if browser is already running"""
    response = requests.get(
        f"{API_URL}/api/v1/browser/active",
        params={"user_id": profile_id}
    )
    data = response.json()
    
    if data['data']['status'] == 'Active':
        print(f"Browser already running on port {data['data']['debug_port']}")
        return True
    return False

# Force stop if needed
if check_browser_status("profile_id"):
    stop_browser("profile_id")
    time.sleep(2)
    start_browser("profile_id")
```

### Connection Timeout

Ensure AdsPower application is running and API is enabled in settings. Check firewall settings allow localhost connections on port 50325.

### Profile Not Found

```python
def verify_profile_exists(profile_id):
    """Check if profile exists before operations"""
    profiles = list_profiles()
    profile_ids = [p['user_id'] for p in profiles['data']['list']]
    return profile_id in profile_ids
```

## Best Practices

1. **Rate Limiting**: Add delays between profile operations to avoid detection
2. **Error Handling**: Always wrap API calls in try-except blocks
3. **Resource Cleanup**: Always stop browsers and disconnect drivers after use
4. **Proxy Rotation**: Use different proxies for each profile for better anonymity
5. **Fingerprint Variation**: Customize fingerprints to avoid pattern detection
6. **Profile Organization**: Use groups to manage campaigns effectively
