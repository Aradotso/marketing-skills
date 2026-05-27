---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I create AdsPower browser profiles
  - manage multiple browser accounts with AdsPower
  - automate AdsPower browser sessions
  - configure antidetect browser profiles
  - use AdsPower API for automation
  - set up AdsPower for marketing campaigns
  - control AdsPower profiles programmatically
  - integrate AdsPower with RPA workflows
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple browser profiles with unique fingerprints. It enables marketing teams to run multi-account campaigns, automate browser sessions, and perform tasks requiring account isolation without detection. This skill covers the AdsPower API, profile management, automation workflows, and integration patterns.

## What AdsPower Does

AdsPower creates isolated browser environments with unique digital fingerprints (canvas, WebGL, fonts, timezone, etc.) to prevent platform detection and account linking. Key features:

- **Multi-account management**: Run hundreds of browser profiles simultaneously
- **Fingerprint customization**: Unique browser fingerprints per profile
- **Team collaboration**: Share profiles across marketing teams
- **Automation support**: API and RPA integration for browser automation
- **Cloud sync**: Synchronize profiles across devices
- **Proxy management**: Assign different proxies per profile

## Installation & Setup

AdsPower consists of a desktop application and API for automation:

### Desktop Application

1. Download from official AdsPower website
2. Install and launch the application
3. Create an account and log in
4. Configure your subscription plan

### API Access

The AdsPower Local API runs on `http://localhost:50325` when the application is active.

## Core API Endpoints

### Starting a Browser Profile

```python
import requests
import json

# Start a browser profile
def start_profile(profile_id):
    url = f"http://localhost:50325/api/v1/browser/start?serial_number={profile_id}"
    response = requests.get(url)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'webdriver_path': data['data']['webdriver'],
            'debug_port': data['data']['ws']['selenium'],
            'user_data_dir': data['data']['user_data_dir']
        }
    else:
        raise Exception(f"Failed to start profile: {data['msg']}")

# Example usage
profile_info = start_profile("profile_abc123")
print(f"Debug port: {profile_info['debug_port']}")
```

### Stopping a Browser Profile

```python
def stop_profile(profile_id):
    url = f"http://localhost:50325/api/v1/browser/stop?serial_number={profile_id}"
    response = requests.get(url)
    data = response.json()
    
    if data['code'] == 0:
        print(f"Profile {profile_id} stopped successfully")
    else:
        raise Exception(f"Failed to stop profile: {data['msg']}")

# Example usage
stop_profile("profile_abc123")
```

### Creating a New Profile

```python
def create_profile(profile_name, proxy_config=None):
    url = "http://localhost:50325/api/v1/user/create"
    
    payload = {
        "name": profile_name,
        "group_id": "0",
        "domain_name": "",
        "open_urls": [],
        "repeat_config": [0],
        "username": "",
        "password": "",
        "fakey": "",
        "cookie": [],
        "ignore_cookie_error": 0,
        "ip": "",
        "country": "US",
        "region": "",
        "city": "",
        "remark": "",
        "ipchecker": 0,
        "sys_app_cate_id": "0",
        "user_proxy_config": proxy_config or {}
    }
    
    response = requests.post(url, json=payload)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['id']
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

new_profile_id = create_profile("Marketing Campaign 1", proxy_config)
print(f"Created profile: {new_profile_id}")
```

### Querying Profile List

```python
def get_profiles(page=1, page_size=50, group_id=None):
    url = "http://localhost:50325/api/v1/user/list"
    
    params = {
        "page": page,
        "page_size": page_size
    }
    
    if group_id:
        params['group_id'] = group_id
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['list']
    else:
        raise Exception(f"Failed to get profiles: {data['msg']}")

# Example usage
profiles = get_profiles(page=1, page_size=10)
for profile in profiles:
    print(f"{profile['serial_number']}: {profile['name']}")
```

## Selenium Integration

Connecting Selenium to an AdsPower profile:

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import requests

def launch_with_selenium(profile_id):
    # Start the profile
    start_url = f"http://localhost:50325/api/v1/browser/start?serial_number={profile_id}"
    response = requests.get(start_url).json()
    
    if response['code'] != 0:
        raise Exception(f"Failed to start: {response['msg']}")
    
    chrome_options = Options()
    chrome_options.debugger_address = response['data']['ws']['selenium']
    
    driver = webdriver.Chrome(
        executable_path=response['data']['webdriver'],
        options=chrome_options
    )
    
    return driver

# Example automation
driver = launch_with_selenium("profile_abc123")
driver.get("https://example.com")
print(driver.title)
driver.quit()

# Stop the profile
stop_profile("profile_abc123")
```

## Playwright Integration

Using Playwright with AdsPower:

```python
from playwright.sync_api import sync_playwright
import requests

def launch_with_playwright(profile_id):
    # Start the profile
    start_url = f"http://localhost:50325/api/v1/browser/start?serial_number={profile_id}"
    response = requests.get(start_url).json()
    
    if response['code'] != 0:
        raise Exception(f"Failed to start: {response['msg']}")
    
    debug_port = response['data']['ws']['selenium'].split(':')[1]
    
    with sync_playwright() as p:
        browser = p.chromium.connect_over_cdp(f"http://localhost:{debug_port}")
        context = browser.contexts[0]
        page = context.pages[0] if context.pages else context.new_page()
        
        # Your automation code
        page.goto("https://example.com")
        print(page.title())
        
        browser.close()
    
    # Stop the profile
    stop_profile(profile_id)

# Example usage
launch_with_playwright("profile_abc123")
```

## Profile Management Patterns

### Bulk Profile Creation

```python
import time

def create_bulk_profiles(count, name_prefix, proxy_list):
    """Create multiple profiles with rotating proxies"""
    created_profiles = []
    
    for i in range(count):
        proxy = proxy_list[i % len(proxy_list)]
        
        proxy_config = {
            "proxy_soft": "other",
            "proxy_type": proxy['type'],
            "proxy_host": proxy['host'],
            "proxy_port": proxy['port'],
            "proxy_user": proxy.get('user', ''),
            "proxy_password": proxy.get('password', '')
        }
        
        profile_name = f"{name_prefix}_{i+1}"
        profile_id = create_profile(profile_name, proxy_config)
        created_profiles.append(profile_id)
        
        time.sleep(1)  # Rate limiting
    
    return created_profiles

# Example usage
proxies = [
    {"type": "http", "host": "proxy1.example.com", "port": "8080"},
    {"type": "http", "host": "proxy2.example.com", "port": "8080"},
]

profile_ids = create_bulk_profiles(10, "Campaign_A", proxies)
```

### Profile Group Management

```python
def create_group(group_name):
    url = "http://localhost:50325/api/v1/group/create"
    payload = {"group_name": group_name}
    
    response = requests.post(url, json=payload)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['group_id']
    else:
        raise Exception(f"Failed to create group: {data['msg']}")

def move_to_group(profile_id, group_id):
    url = "http://localhost:50325/api/v1/user/update"
    payload = {
        "user_id": profile_id,
        "group_id": group_id
    }
    
    response = requests.post(url, json=payload)
    data = response.json()
    
    if data['code'] != 0:
        raise Exception(f"Failed to move profile: {data['msg']}")

# Example: Organize profiles by campaign
campaign_group_id = create_group("Q1 Marketing Campaign")
for profile_id in profile_ids:
    move_to_group(profile_id, campaign_group_id)
```

## Automation Workflows

### Social Media Account Management

```python
def manage_social_accounts(profiles, post_content):
    """Post content across multiple social media accounts"""
    
    for profile in profiles:
        try:
            driver = launch_with_selenium(profile['serial_number'])
            
            # Navigate to social platform
            driver.get("https://platform.example.com/login")
            
            # Login (credentials stored in profile or env)
            # ... login automation code ...
            
            # Create post
            # ... posting automation code ...
            
            driver.quit()
            stop_profile(profile['serial_number'])
            
            time.sleep(5)  # Delay between accounts
            
        except Exception as e:
            print(f"Failed for profile {profile['name']}: {e}")
            continue
```

### E-commerce Multi-Account Operations

```python
def check_inventory_multi_account(profile_ids, product_url):
    """Check product availability across multiple accounts"""
    results = []
    
    for profile_id in profile_ids:
        driver = launch_with_selenium(profile_id)
        
        try:
            driver.get(product_url)
            time.sleep(2)
            
            # Check stock status
            # ... scraping logic ...
            
            results.append({
                'profile': profile_id,
                'in_stock': True,  # parsed result
                'price': '99.99'
            })
            
        finally:
            driver.quit()
            stop_profile(profile_id)
    
    return results
```

## Configuration Best Practices

### Environment Variables

```python
import os

ADSPOWER_API_BASE = os.getenv('ADSPOWER_API_BASE', 'http://localhost:50325')
ADSPOWER_API_KEY = os.getenv('ADSPOWER_API_KEY')  # For cloud API

def api_request(endpoint, method='GET', data=None):
    url = f"{ADSPOWER_API_BASE}{endpoint}"
    headers = {}
    
    if ADSPOWER_API_KEY:
        headers['Authorization'] = f"Bearer {ADSPOWER_API_KEY}"
    
    if method == 'GET':
        response = requests.get(url, headers=headers)
    else:
        response = requests.post(url, json=data, headers=headers)
    
    return response.json()
```

### Profile Configuration Templates

```python
PROFILE_TEMPLATES = {
    "basic": {
        "fingerprint_config": {
            "automatic_timezone": 1,
            "webrtc": "proxy",
            "location": "ask",
            "language": ["en-US", "en"]
        }
    },
    "stealth": {
        "fingerprint_config": {
            "automatic_timezone": 1,
            "webrtc": "disabled",
            "location": "disabled",
            "language": ["en-US"],
            "canvas": 1,  # Noise enabled
            "webgl": 1,
            "audio": 1
        }
    }
}

def create_from_template(name, template_key, proxy_config):
    template = PROFILE_TEMPLATES[template_key]
    payload = {
        "name": name,
        **template,
        "user_proxy_config": proxy_config
    }
    
    return create_profile_advanced(payload)
```

## Troubleshooting

### Common Issues

**Profile won't start:**
- Ensure AdsPower application is running
- Check if profile already running: `GET /api/v1/browser/active`
- Verify sufficient system resources

**Selenium connection fails:**
```python
def check_profile_status(profile_id):
    url = f"http://localhost:50325/api/v1/browser/active?serial_number={profile_id}"
    response = requests.get(url)
    data = response.json()
    
    if data['code'] == 0 and data['data']['status'] == 'Active':
        print(f"Profile {profile_id} is running")
        return True
    else:
        print(f"Profile {profile_id} is not active")
        return False
```

**API returns error code -1:**
- Check API endpoint syntax
- Verify profile_id/serial_number format
- Ensure local API port 50325 is accessible

**Proxy not working:**
```python
def verify_proxy(profile_id):
    driver = launch_with_selenium(profile_id)
    driver.get("https://api.ipify.org?format=json")
    ip_data = driver.find_element_by_tag_name("pre").text
    print(f"Current IP: {ip_data}")
    driver.quit()
    stop_profile(profile_id)
```

### Error Handling

```python
def safe_profile_operation(profile_id, operation_func):
    """Wrapper for safe profile operations with cleanup"""
    try:
        result = operation_func(profile_id)
        return result
    except Exception as e:
        print(f"Error: {e}")
        return None
    finally:
        # Ensure profile is stopped
        try:
            stop_profile(profile_id)
        except:
            pass

# Usage
def my_automation(profile_id):
    driver = launch_with_selenium(profile_id)
    driver.get("https://example.com")
    # ... automation logic ...
    driver.quit()
    return "Success"

result = safe_profile_operation("profile_abc123", my_automation)
```

## Advanced Features

### Cookie Management

```python
def import_cookies(profile_id, cookies):
    """Import cookies to a profile"""
    url = "http://localhost:50325/api/v1/user/update"
    
    payload = {
        "user_id": profile_id,
        "cookie": cookies  # Array of cookie objects
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Example cookie format
cookies = [
    {
        "name": "session_id",
        "value": "abc123",
        "domain": ".example.com",
        "path": "/",
        "secure": True,
        "httpOnly": True
    }
]

import_cookies("profile_abc123", cookies)
```

This skill provides comprehensive coverage of AdsPower's API and automation capabilities for managing multi-account marketing campaigns and browser automation workflows.
