---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaign management
triggers:
  - how do I use AdsPower for multi-account management
  - set up antidetect browser profiles with AdsPower
  - automate browser profiles for marketing campaigns
  - manage multiple accounts with AdsPower
  - create and configure AdsPower browser profiles
  - integrate AdsPower API for automation
  - handle fingerprint management in AdsPower
  - run RPA scripts with AdsPower browsers
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that enables marketing teams to manage multiple browser profiles with unique fingerprints for multi-account operations, ad campaigns, and automation. Each profile simulates a distinct device and user environment to prevent platform detection and account linking.

## Overview

AdsPower provides:
- **Multi-account management**: Run hundreds of isolated browser profiles
- **Fingerprint protection**: Unique canvas, WebGL, fonts, and hardware fingerprints per profile
- **Team collaboration**: Cloud-based profile sharing and access control
- **Automation support**: API and RPA integration for workflow automation
- **Proxy management**: Built-in proxy configuration per profile

## Installation

### Desktop Application

1. Download AdsPower from the official website
2. Install the application for your OS (Windows, macOS, Linux)
3. Launch and create an account
4. Activate your license or use the free tier

### API Access

AdsPower provides a local API that runs when the application is active (default: `http://localhost:50325`).

## Core Concepts

### Browser Profiles

Each profile represents an isolated browser instance with:
- Unique fingerprint (canvas, WebGL, audio, fonts)
- Dedicated cookies and cache
- Individual proxy settings
- Custom user agent and timezone
- Separate extensions and configurations

### Profile Groups

Organize profiles into logical groups for:
- Different clients or campaigns
- Geographic regions
- Account types or platforms
- Team member assignments

## API Usage

### Connection Setup

```python
import requests
import json

# AdsPower local API base URL
ADSPOWER_API = "http://localhost:50325/api/v1"

def check_api_status():
    """Verify AdsPower is running"""
    try:
        response = requests.get(f"{ADSPOWER_API}/status")
        return response.json()
    except requests.exceptions.ConnectionError:
        raise Exception("AdsPower is not running. Please start the application.")
```

### Profile Management

```python
def create_profile(name, group_id=None, proxy_config=None):
    """Create a new browser profile"""
    payload = {
        "name": name,
        "group_id": group_id or "0",
        "fingerprint_config": {
            "automatic_timezone": "1",
            "webrtc": "proxy",
            "location": "prompt",
            "language": ["en-US", "en"]
        }
    }
    
    if proxy_config:
        payload["user_proxy_config"] = proxy_config
    
    response = requests.post(
        f"{ADSPOWER_API}/user/create",
        json=payload
    )
    return response.json()

def list_profiles(group_id=None, page=1, page_size=50):
    """Get list of browser profiles"""
    params = {
        "page": page,
        "page_size": page_size
    }
    if group_id:
        params["group_id"] = group_id
    
    response = requests.get(
        f"{ADSPOWER_API}/user/list",
        params=params
    )
    return response.json()

def update_profile(profile_id, updates):
    """Update profile configuration"""
    payload = {
        "user_id": profile_id,
        **updates
    }
    response = requests.post(
        f"{ADSPOWER_API}/user/update",
        json=payload
    )
    return response.json()

def delete_profile(profile_id):
    """Delete a browser profile"""
    response = requests.post(
        f"{ADSPOWER_API}/user/delete",
        json={"user_ids": [profile_id]}
    )
    return response.json()
```

### Browser Control

```python
def start_profile(profile_id, headless=False):
    """Launch a browser profile"""
    params = {
        "user_id": profile_id,
        "launch_args": "",
        "headless": "1" if headless else "0"
    }
    response = requests.get(
        f"{ADSPOWER_API}/browser/start",
        params=params
    )
    data = response.json()
    
    if data["code"] == 0:
        return {
            "selenium_address": data["data"]["ws"]["selenium"],
            "webdriver_path": data["data"]["webdriver"],
            "debug_port": data["data"]["debug_port"]
        }
    else:
        raise Exception(f"Failed to start profile: {data['msg']}")

def stop_profile(profile_id):
    """Close a browser profile"""
    response = requests.get(
        f"{ADSPOWER_API}/browser/stop",
        params={"user_id": profile_id}
    )
    return response.json()

def check_profile_status(profile_id):
    """Check if profile is active"""
    response = requests.get(
        f"{ADSPOWER_API}/browser/active",
        params={"user_id": profile_id}
    )
    data = response.json()
    return data["data"]["status"] == "Active"
```

### Proxy Configuration

```python
def configure_proxy(proxy_type, host, port, username=None, password=None):
    """Create proxy configuration object"""
    config = {
        "proxy_soft": proxy_type,  # http, https, socks5
        "proxy_type": "custom",
        "proxy_host": host,
        "proxy_port": str(port)
    }
    
    if username and password:
        config["proxy_user"] = username
        config["proxy_password"] = password
    
    return config

# Example usage
proxy = configure_proxy(
    proxy_type="http",
    host="proxy.example.com",
    port=8080,
    username="user",
    password="pass"
)

profile = create_profile(
    name="Marketing Campaign Profile",
    proxy_config=proxy
)
```

## Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(profile_id):
    """Connect Selenium to an AdsPower profile"""
    # Start the profile
    browser_data = start_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress",
        browser_data["selenium_address"].replace("http://", "")
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(
        executable_path=browser_data["webdriver_path"],
        options=chrome_options
    )
    
    return driver

# Usage example
def automate_profile(profile_id, url):
    """Run automation on a profile"""
    driver = connect_selenium(profile_id)
    
    try:
        driver.get(url)
        # Your automation logic here
        print(f"Page title: {driver.title}")
        
    finally:
        driver.quit()
        stop_profile(profile_id)
```

## Puppeteer Integration

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const ADSPOWER_API = 'http://localhost:50325/api/v1';

async function startProfile(profileId) {
  const response = await axios.get(`${ADSPOWER_API}/browser/start`, {
    params: { user_id: profileId }
  });
  
  if (response.data.code !== 0) {
    throw new Error(`Failed to start profile: ${response.data.msg}`);
  }
  
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

// Usage example
async function automateWithPuppeteer(profileId) {
  const browser = await connectPuppeteer(profileId);
  
  try {
    const pages = await browser.pages();
    const page = pages[0] || await browser.newPage();
    
    await page.goto('https://example.com');
    const title = await page.title();
    console.log(`Page title: ${title}`);
    
  } finally {
    await browser.disconnect();
    // Stop profile via API
    await axios.get(`${ADSPOWER_API}/browser/stop`, {
      params: { user_id: profileId }
    });
  }
}
```

## Common Patterns

### Bulk Profile Creation

```python
def create_campaign_profiles(campaign_name, count, proxies):
    """Create multiple profiles for a campaign"""
    profiles = []
    
    for i in range(count):
        proxy = proxies[i % len(proxies)] if proxies else None
        
        profile = create_profile(
            name=f"{campaign_name} - Profile {i+1}",
            proxy_config=proxy
        )
        
        if profile["code"] == 0:
            profiles.append(profile["data"]["id"])
            print(f"Created profile: {profile['data']['id']}")
        else:
            print(f"Failed to create profile {i+1}: {profile['msg']}")
    
    return profiles
```

### Profile Rotation

```python
import time
from typing import Callable

def rotate_profiles(profile_ids, task_function: Callable, delay=5):
    """Execute a task across multiple profiles with rotation"""
    results = []
    
    for profile_id in profile_ids:
        try:
            print(f"Starting task on profile: {profile_id}")
            result = task_function(profile_id)
            results.append({"profile_id": profile_id, "result": result})
            
            # Delay between profiles
            time.sleep(delay)
            
        except Exception as e:
            print(f"Error on profile {profile_id}: {str(e)}")
            results.append({"profile_id": profile_id, "error": str(e)})
    
    return results

# Example task function
def scrape_data(profile_id):
    driver = connect_selenium(profile_id)
    try:
        driver.get("https://example.com/data")
        data = driver.find_element_by_id("content").text
        return data
    finally:
        driver.quit()
        stop_profile(profile_id)
```

### Group Management

```python
def organize_profiles_by_region(profiles_by_region):
    """Create groups and assign profiles"""
    for region, profile_ids in profiles_by_region.items():
        # Create group
        group_response = requests.post(
            f"{ADSPOWER_API}/group/create",
            json={"group_name": region}
        )
        
        if group_response.json()["code"] == 0:
            group_id = group_response.json()["data"]["group_id"]
            
            # Assign profiles to group
            for profile_id in profile_ids:
                update_profile(profile_id, {"group_id": group_id})
```

## Configuration Best Practices

### Fingerprint Settings

```python
def get_optimized_fingerprint_config(platform="general"):
    """Get recommended fingerprint configuration"""
    configs = {
        "general": {
            "automatic_timezone": "1",
            "webrtc": "proxy",
            "location": "prompt",
            "language": ["en-US", "en"],
            "page_language": ["en-US", "en"],
            "canvas": "1",
            "webgl_image": "1",
            "audio": "1",
            "font": "1"
        },
        "social_media": {
            "automatic_timezone": "1",
            "webrtc": "forward",
            "location": "allow",
            "language": ["en-US", "en"],
            "media_devices": "1"
        },
        "e_commerce": {
            "automatic_timezone": "1",
            "webrtc": "disabled",
            "location": "prompt",
            "do_not_track": "0"
        }
    }
    return configs.get(platform, configs["general"])
```

## Troubleshooting

### Profile Won't Start

```python
def diagnose_profile_start_issue(profile_id):
    """Debug profile startup problems"""
    # Check if profile exists
    profiles = list_profiles()
    profile_exists = any(
        p["user_id"] == profile_id 
        for p in profiles.get("data", {}).get("list", [])
    )
    
    if not profile_exists:
        return "Profile does not exist"
    
    # Check if already running
    if check_profile_status(profile_id):
        stop_profile(profile_id)
        time.sleep(2)
    
    # Try starting with detailed error
    try:
        start_profile(profile_id)
        return "Profile started successfully"
    except Exception as e:
        return f"Startup error: {str(e)}"
```

### Connection Issues

```python
def verify_adspower_connection():
    """Check AdsPower API connectivity"""
    try:
        response = requests.get(f"{ADSPOWER_API}/status", timeout=5)
        if response.status_code == 200:
            return True, "Connected"
    except requests.exceptions.Timeout:
        return False, "Connection timeout - AdsPower may be starting up"
    except requests.exceptions.ConnectionError:
        return False, "Cannot connect - ensure AdsPower application is running"
    except Exception as e:
        return False, f"Unexpected error: {str(e)}"
```

### Memory Management

```python
def cleanup_inactive_profiles():
    """Close inactive profiles to free resources"""
    profiles = list_profiles()
    closed_count = 0
    
    for profile in profiles.get("data", {}).get("list", []):
        profile_id = profile["user_id"]
        
        if check_profile_status(profile_id):
            stop_profile(profile_id)
            closed_count += 1
            time.sleep(1)
    
    return f"Closed {closed_count} profiles"
```

## Environment Variables

Use environment variables for sensitive configuration:

```python
import os

ADSPOWER_API = os.getenv("ADSPOWER_API_URL", "http://localhost:50325/api/v1")
PROXY_USERNAME = os.getenv("PROXY_USERNAME")
PROXY_PASSWORD = os.getenv("PROXY_PASSWORD")
```

## API Reference Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/user/create` | POST | Create new profile |
| `/user/list` | GET | List profiles |
| `/user/update` | POST | Update profile |
| `/user/delete` | POST | Delete profile |
| `/browser/start` | GET | Launch profile |
| `/browser/stop` | GET | Close profile |
| `/browser/active` | GET | Check profile status |
| `/group/create` | POST | Create profile group |
| `/group/list` | GET | List groups |
