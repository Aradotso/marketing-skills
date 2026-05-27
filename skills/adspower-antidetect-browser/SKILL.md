---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I set up AdsPower browser profiles
  - automate AdsPower for marketing campaigns
  - manage multiple accounts with AdsPower
  - create AdsPower browser automation
  - configure AdsPower antidetect profiles
  - use AdsPower API for automation
  - run marketing campaigns with AdsPower
  - integrate AdsPower with RPA tools
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple browser profiles with unique fingerprints. This skill covers using AdsPower for marketing automation, multi-account management, and campaign orchestration through its API and browser profiles.

## What AdsPower Does

AdsPower provides:
- **Antidetect Browser Profiles**: Each profile has a unique browser fingerprint
- **Multi-Account Management**: Safely manage multiple accounts across platforms
- **Team Collaboration**: Cloud-based profile sharing and team workflows
- **Automation Support**: API and RPA integration for automated workflows
- **Profile Isolation**: Complete separation between browser sessions

## Installation & Setup

### Installing AdsPower

1. Download AdsPower from the official website
2. Install the application on your system (Windows/Mac/Linux)
3. Launch AdsPower and create an account
4. Enable API access in Settings → API

### API Configuration

Enable the Local API in AdsPower settings:
- Settings → API → Enable Local API
- Default API endpoint: `http://localhost:50325`
- Note your API port if customized

## API Overview

AdsPower provides a REST API for profile management and automation.

### Base API Endpoint

```
http://localhost:50325/api/v1
```

### Authentication

Most operations require either:
- Running AdsPower application locally
- Using API key from AdsPower cloud dashboard (for remote operations)

## Key API Operations

### Profile Management

#### List All Profiles

```python
import requests
import os

API_BASE = os.getenv('ADSPOWER_API_BASE', 'http://localhost:50325/api/v1')

def list_profiles(group_id=None, page=1, page_size=50):
    """List browser profiles"""
    params = {
        'page': page,
        'page_size': page_size
    }
    if group_id:
        params['group_id'] = group_id
    
    response = requests.get(f'{API_BASE}/user/list', params=params)
    return response.json()

profiles = list_profiles()
print(f"Total profiles: {profiles['data']['total']}")
for profile in profiles['data']['list']:
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

#### Create a New Profile

```python
def create_profile(name, group_id=None, fingerprint_config=None):
    """Create a new browser profile"""
    payload = {
        'name': name,
        'domain_name': '',
        'open_urls': [],
        'repeat_config': [0]
    }
    
    if group_id:
        payload['group_id'] = group_id
    
    if fingerprint_config:
        payload['fingerprint_config'] = fingerprint_config
    
    response = requests.post(f'{API_BASE}/user/create', json=payload)
    return response.json()

# Create a basic profile
new_profile = create_profile(name='Marketing Campaign Profile 1')
profile_id = new_profile['data']['id']
print(f"Created profile: {profile_id}")
```

#### Update Profile Configuration

```python
def update_profile(user_id, name=None, proxy=None):
    """Update profile settings"""
    payload = {'user_id': user_id}
    
    if name:
        payload['name'] = name
    
    if proxy:
        payload['user_proxy_config'] = proxy
    
    response = requests.post(f'{API_BASE}/user/update', json=payload)
    return response.json()

# Update with proxy
proxy_config = {
    'proxy_type': 'http',
    'proxy_host': os.getenv('PROXY_HOST'),
    'proxy_port': os.getenv('PROXY_PORT'),
    'proxy_user': os.getenv('PROXY_USER'),
    'proxy_password': os.getenv('PROXY_PASSWORD')
}

update_profile(profile_id, proxy=proxy_config)
```

#### Delete Profile

```python
def delete_profile(user_ids):
    """Delete one or more profiles"""
    payload = {'user_ids': user_ids if isinstance(user_ids, list) else [user_ids]}
    response = requests.post(f'{API_BASE}/user/delete', json=payload)
    return response.json()

delete_profile([profile_id])
```

### Browser Control

#### Start Browser Session

```python
def start_browser(user_id, open_tabs=True):
    """Launch browser profile"""
    params = {
        'user_id': user_id,
        'open_tabs': 1 if open_tabs else 0
    }
    
    response = requests.get(f'{API_BASE}/browser/start', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'selenium_address': data['data']['ws']['selenium'],
            'webdriver_path': data['data']['webdriver'],
            'debug_port': data['data']['debug_port']
        }
    return None

# Start browser and get connection info
browser_info = start_browser(profile_id)
print(f"Browser started at: {browser_info['selenium_address']}")
```

#### Stop Browser Session

```python
def stop_browser(user_id):
    """Close browser profile"""
    params = {'user_id': user_id}
    response = requests.get(f'{API_BASE}/browser/stop', params=params)
    return response.json()

stop_browser(profile_id)
```

#### Check Browser Status

```python
def check_browser_status(user_id):
    """Check if browser is active"""
    params = {'user_id': user_id}
    response = requests.get(f'{API_BASE}/browser/active', params=params)
    data = response.json()
    return data['data']['status'] == 'Active'

is_running = check_browser_status(profile_id)
print(f"Browser running: {is_running}")
```

## Automation with Selenium

### Connect to AdsPower Browser via Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(user_id):
    """Connect Selenium to AdsPower browser"""
    # Start the browser
    browser_info = start_browser(user_id)
    
    if not browser_info:
        raise Exception("Failed to start browser")
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress", 
        f"127.0.0.1:{browser_info['debug_port']}"
    )
    
    # Connect driver
    driver = webdriver.Chrome(options=chrome_options)
    return driver

# Use with Selenium
driver = connect_selenium(profile_id)
driver.get('https://example.com')
print(driver.title)
driver.quit()
stop_browser(profile_id)
```

### Full Automation Example

```python
import time
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def run_marketing_automation(profile_id, target_url):
    """Run automated marketing task"""
    driver = None
    try:
        # Start browser
        driver = connect_selenium(profile_id)
        
        # Navigate to target
        driver.get(target_url)
        
        # Wait for page load
        wait = WebDriverWait(driver, 10)
        
        # Perform automation tasks
        # Example: collect data, post content, etc.
        
        # Take screenshot
        driver.save_screenshot(f'profile_{profile_id}_screenshot.png')
        
        return {'status': 'success'}
        
    except Exception as e:
        return {'status': 'error', 'message': str(e)}
        
    finally:
        if driver:
            driver.quit()
        stop_browser(profile_id)
```

## Common Patterns

### Bulk Profile Creation

```python
def create_bulk_profiles(count, name_prefix, group_id=None):
    """Create multiple profiles at once"""
    created_profiles = []
    
    for i in range(count):
        profile_name = f"{name_prefix}_{i+1}"
        result = create_profile(name=profile_name, group_id=group_id)
        
        if result['code'] == 0:
            created_profiles.append(result['data']['id'])
            print(f"Created: {profile_name}")
        else:
            print(f"Failed to create {profile_name}: {result['msg']}")
    
    return created_profiles

# Create 10 profiles for campaign
campaign_profiles = create_bulk_profiles(10, 'Campaign_Q1', group_id='group_123')
```

### Profile Rotation for Tasks

```python
def rotate_profiles_for_tasks(profile_ids, tasks):
    """Distribute tasks across multiple profiles"""
    results = []
    
    for idx, task in enumerate(tasks):
        profile_id = profile_ids[idx % len(profile_ids)]
        
        print(f"Running task {idx+1} on profile {profile_id}")
        result = run_marketing_automation(profile_id, task['url'])
        results.append({
            'task': task,
            'profile': profile_id,
            'result': result
        })
        
        # Delay between tasks
        time.sleep(5)
    
    return results
```

### Group Management

```python
def get_groups():
    """List all profile groups"""
    response = requests.get(f'{API_BASE}/group/list')
    return response.json()

def create_group(group_name, remark=''):
    """Create a new profile group"""
    payload = {
        'group_name': group_name,
        'remark': remark
    }
    response = requests.post(f'{API_BASE}/group/create', json=payload)
    return response.json()

# Organize profiles by campaign
campaign_group = create_group('Q1_Social_Campaign', 'Profiles for Q1 social media')
group_id = campaign_group['data']['group_id']
```

## Configuration Examples

### Fingerprint Configuration

```python
fingerprint_config = {
    'automatic_timezone': 1,  # Auto timezone from IP
    'webrtc': 'proxy',  # WebRTC mode
    'location': 'proxy',  # Geolocation from proxy
    'ua': '',  # Custom user agent (empty for auto)
    'screen_resolution': '1920_1080',
    'fonts': ['all'],
    'canvas': 1,  # Canvas noise
    'webgl_image': 1,  # WebGL noise
    'audio': 1,  # Audio noise
    'timezone': '',  # Auto from IP
    'language': ['en-US', 'en'],
    'port_scan_protection': 1,
    'do_not_track': 0,
    'hardware_concurrency': 4
}

profile = create_profile(
    name='Custom Fingerprint Profile',
    fingerprint_config=fingerprint_config
)
```

### Proxy Configurations

```python
# HTTP Proxy
http_proxy = {
    'proxy_type': 'http',
    'proxy_host': os.getenv('PROXY_HOST'),
    'proxy_port': os.getenv('PROXY_PORT'),
    'proxy_user': os.getenv('PROXY_USER'),
    'proxy_password': os.getenv('PROXY_PASSWORD')
}

# SOCKS5 Proxy
socks_proxy = {
    'proxy_type': 'socks5',
    'proxy_host': os.getenv('SOCKS_HOST'),
    'proxy_port': os.getenv('SOCKS_PORT'),
    'proxy_user': os.getenv('SOCKS_USER'),
    'proxy_password': os.getenv('SOCKS_PASSWORD')
}

# No proxy
no_proxy = {
    'proxy_type': 'noproxy'
}
```

## Troubleshooting

### Browser Fails to Start

```python
def safe_start_browser(user_id, max_retries=3):
    """Start browser with retry logic"""
    for attempt in range(max_retries):
        try:
            # Check if already running
            if check_browser_status(user_id):
                stop_browser(user_id)
                time.sleep(2)
            
            browser_info = start_browser(user_id)
            if browser_info:
                return browser_info
                
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            time.sleep(5)
    
    raise Exception(f"Failed to start browser after {max_retries} attempts")
```

### Check API Connectivity

```python
def check_api_health():
    """Verify API is accessible"""
    try:
        response = requests.get(f'{API_BASE}/status/check', timeout=5)
        return response.status_code == 200
    except:
        return False

if not check_api_health():
    print("AdsPower API not accessible. Ensure application is running.")
```

### Handle Profile Limits

```python
def check_profile_limit():
    """Check current profile usage"""
    profiles = list_profiles(page_size=1)
    if 'data' in profiles:
        total = profiles['data']['total']
        print(f"Current profiles: {total}")
        return total
    return 0
```

### Clean Up Inactive Profiles

```python
def cleanup_inactive_profiles(days_inactive=30):
    """Remove profiles not used recently"""
    profiles = list_profiles(page_size=100)
    to_delete = []
    
    for profile in profiles['data']['list']:
        # Check last modified date
        # Add logic to identify inactive profiles
        pass
    
    if to_delete:
        delete_profile(to_delete)
        print(f"Deleted {len(to_delete)} inactive profiles")
```

## Environment Variables

Store sensitive configuration in environment variables:

```bash
# API Configuration
ADSPOWER_API_BASE=http://localhost:50325/api/v1

# Proxy Settings
PROXY_HOST=your-proxy-host.com
PROXY_PORT=8080
PROXY_USER=your_username
PROXY_PASSWORD=your_password

# SOCKS Proxy
SOCKS_HOST=socks-proxy.com
SOCKS_PORT=1080
SOCKS_USER=socks_user
SOCKS_PASSWORD=socks_pass
```

Load in Python:

```python
import os
from dotenv import load_dotenv

load_dotenv()

API_BASE = os.getenv('ADSPOWER_API_BASE', 'http://localhost:50325/api/v1')
```
