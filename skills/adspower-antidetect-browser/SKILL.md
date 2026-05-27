---
name: adspower-antidetect-browser
description: Use AdsPower antidetect browser for multi-account management, browser automation, and fingerprint protection in marketing campaigns
triggers:
  - how do I use AdsPower for browser automation
  - set up multiple browser profiles with AdsPower
  - automate marketing campaigns with AdsPower
  - manage multiple accounts without detection
  - use AdsPower API for profile management
  - create and control AdsPower browser profiles
  - integrate AdsPower with automation scripts
  - handle browser fingerprinting with AdsPower
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that enables marketing teams to manage multiple browser profiles with unique fingerprints for multi-account operations, ad verification, social media management, and e-commerce automation. Each profile simulates a real device with distinct browser fingerprints to avoid detection and account bans.

## Overview

AdsPower provides:
- **Multi-account management**: Run dozens or hundreds of isolated browser profiles
- **Fingerprint protection**: Each profile has unique canvas, WebGL, fonts, timezone, and other fingerprints
- **Automation support**: Browser automation via Selenium, Puppeteer, or Playwright
- **Team collaboration**: Cloud-based profile sharing across team members
- **RPA capabilities**: No-code automation builder for repetitive tasks

## Installation

### Desktop Application

1. Download AdsPower from the official website
2. Install the application for your OS (Windows, macOS, Linux)
3. Launch AdsPower and create an account
4. Log in to access the dashboard

### API Access

AdsPower provides a local API server that runs when the application is active (default: `http://localhost:50325`).

No additional installation required for API usage - it's built into the desktop app.

## Core Concepts

### Browser Profiles

Each profile represents an isolated browser instance with:
- Unique fingerprint configuration
- Separate cookies, cache, and local storage
- Custom proxy settings
- Specific user agent and device emulation

### Profile Groups

Organize profiles into logical groups for different campaigns, clients, or purposes.

## API Configuration

Set the API base URL as an environment variable:

```bash
export ADSPOWER_API_URL="http://localhost:50325"
```

For remote AdsPower instances:

```bash
export ADSPOWER_API_URL="http://your-server-ip:50325"
export ADSPOWER_API_KEY="your_api_key_here"  # If API key authentication is enabled
```

## Key API Endpoints

### Profile Management

#### List All Profiles

```python
import requests
import os

API_URL = os.getenv('ADSPOWER_API_URL', 'http://localhost:50325')

def list_profiles(page=1, page_size=100):
    """Get list of all browser profiles"""
    response = requests.get(f"{API_URL}/api/v1/user/list", params={
        'page': page,
        'page_size': page_size
    })
    return response.json()

profiles = list_profiles()
print(f"Total profiles: {profiles['data']['total']}")
for profile in profiles['data']['list']:
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

#### Create New Profile

```python
def create_profile(name, group_id=None, proxy_config=None):
    """Create a new browser profile"""
    payload = {
        'name': name,
        'group_id': group_id or '0',
        'fingerprint_config': {
            'automatic_timezone': '1',
            'webrtc': 'proxy',
            'location': 'ask',
            'language': ['en-US', 'en']
        }
    }
    
    if proxy_config:
        payload['user_proxy_config'] = proxy_config
    
    response = requests.post(f"{API_URL}/api/v1/user/create", json=payload)
    return response.json()

# Create profile with proxy
new_profile = create_profile(
    name='Marketing Campaign Profile 1',
    proxy_config={
        'proxy_soft': 'other',
        'proxy_type': 'http',
        'proxy_host': '192.168.1.100',
        'proxy_port': '8080',
        'proxy_user': os.getenv('PROXY_USER'),
        'proxy_password': os.getenv('PROXY_PASSWORD')
    }
)
print(f"Created profile: {new_profile['data']['id']}")
```

#### Update Profile

```python
def update_profile(profile_id, updates):
    """Update profile configuration"""
    payload = {
        'user_id': profile_id,
        **updates
    }
    response = requests.post(f"{API_URL}/api/v1/user/update", json=payload)
    return response.json()

# Update profile name and proxy
update_profile('profile_id_here', {
    'name': 'Updated Profile Name',
    'user_proxy_config': {
        'proxy_soft': 'other',
        'proxy_type': 'socks5',
        'proxy_host': '10.0.0.50',
        'proxy_port': '1080'
    }
})
```

#### Delete Profile

```python
def delete_profile(profile_id):
    """Delete a browser profile"""
    response = requests.post(f"{API_URL}/api/v1/user/delete", json={
        'user_ids': [profile_id]
    })
    return response.json()
```

### Browser Control

#### Start Browser

```python
def start_browser(profile_id, headless=False):
    """Launch browser for a specific profile"""
    params = {
        'user_id': profile_id,
        'open_tabs': '1'
    }
    if headless:
        params['headless'] = '1'
    
    response = requests.get(f"{API_URL}/api/v1/browser/start", params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'selenium': data['data']['ws']['selenium'],
            'webdriver': data['data']['webdriver'],
            'debug_port': data['data']['debug_port']
        }
    return None

connection_info = start_browser('profile_id_here')
print(f"Selenium endpoint: {connection_info['selenium']}")
```

#### Stop Browser

```python
def stop_browser(profile_id):
    """Close browser for a specific profile"""
    response = requests.get(f"{API_URL}/api/v1/browser/stop", params={
        'user_id': profile_id
    })
    return response.json()

stop_browser('profile_id_here')
```

#### Check Browser Status

```python
def check_browser_status(profile_id):
    """Check if browser is currently running"""
    response = requests.get(f"{API_URL}/api/v1/browser/active", params={
        'user_id': profile_id
    })
    data = response.json()
    return data['data']['status'] == 'Active'
```

## Automation Integration

### Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(profile_id):
    """Connect Selenium to AdsPower browser"""
    browser_info = start_browser(profile_id)
    
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", browser_info['selenium'])
    
    driver = webdriver.Chrome(
        executable_path=browser_info['webdriver'],
        options=chrome_options
    )
    return driver

# Usage
driver = connect_selenium('profile_id_here')
driver.get('https://example.com')
# Perform automation tasks
driver.quit()
stop_browser('profile_id_here')
```

### Puppeteer Integration

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const API_URL = process.env.ADSPOWER_API_URL || 'http://localhost:50325';

async function connectPuppeteer(profileId) {
    // Start browser
    const response = await axios.get(`${API_URL}/api/v1/browser/start`, {
        params: { user_id: profileId }
    });
    
    const { ws } = response.data.data;
    
    // Connect Puppeteer
    const browser = await puppeteer.connect({
        browserWSEndpoint: ws.puppeteer,
        defaultViewport: null
    });
    
    return browser;
}

// Usage
(async () => {
    const browser = await connectPuppeteer('profile_id_here');
    const pages = await browser.pages();
    const page = pages[0] || await browser.newPage();
    
    await page.goto('https://example.com');
    // Perform automation tasks
    
    await browser.disconnect();
    
    // Stop browser
    await axios.get(`${API_URL}/api/v1/browser/stop`, {
        params: { user_id: 'profile_id_here' }
    });
})();
```

### Playwright Integration

```python
from playwright.sync_api import sync_playwright
import requests

def connect_playwright(profile_id):
    """Connect Playwright to AdsPower browser"""
    browser_info = start_browser(profile_id)
    
    with sync_playwright() as p:
        browser = p.chromium.connect_over_cdp(
            f"http://127.0.0.1:{browser_info['debug_port']}"
        )
        return browser

# Usage
browser = connect_playwright('profile_id_here')
context = browser.contexts[0]
page = context.pages[0] if context.pages else context.new_page()

page.goto('https://example.com')
# Perform automation tasks

browser.close()
stop_browser('profile_id_here')
```

## Common Patterns

### Bulk Profile Creation

```python
def create_bulk_profiles(count, name_prefix, group_id=None):
    """Create multiple profiles at once"""
    profile_ids = []
    
    for i in range(count):
        profile = create_profile(
            name=f"{name_prefix} {i+1}",
            group_id=group_id
        )
        if profile['code'] == 0:
            profile_ids.append(profile['data']['id'])
    
    return profile_ids

# Create 10 profiles for a campaign
campaign_profiles = create_bulk_profiles(10, 'Campaign A Profile', group_id='group_123')
```

### Profile Rotation

```python
import time

def rotate_profiles(profile_ids, action_func, delay=5):
    """Execute action across multiple profiles with rotation"""
    for profile_id in profile_ids:
        try:
            driver = connect_selenium(profile_id)
            action_func(driver)
            driver.quit()
            stop_browser(profile_id)
            time.sleep(delay)
        except Exception as e:
            print(f"Error with profile {profile_id}: {e}")
            stop_browser(profile_id)

def perform_social_media_task(driver):
    """Example task to perform"""
    driver.get('https://twitter.com')
    # Perform actions
    time.sleep(10)

# Run task across all profiles
rotate_profiles(campaign_profiles, perform_social_media_task)
```

### Group Management

```python
def create_group(group_name, parent_id='0'):
    """Create a profile group"""
    response = requests.post(f"{API_URL}/api/v1/group/create", json={
        'group_name': group_name,
        'parent_id': parent_id
    })
    return response.json()

def list_groups():
    """List all profile groups"""
    response = requests.get(f"{API_URL}/api/v1/group/list")
    return response.json()

# Create campaign groups
campaigns = ['Facebook Ads', 'Google Ads', 'TikTok Ads']
group_ids = {}

for campaign in campaigns:
    group = create_group(campaign)
    if group['code'] == 0:
        group_ids[campaign] = group['data']['group_id']
```

### Proxy Configuration Templates

```python
def create_proxy_config(proxy_type, host, port, username=None, password=None):
    """Create proxy configuration object"""
    config = {
        'proxy_soft': 'other',
        'proxy_type': proxy_type,  # 'http', 'https', 'socks5'
        'proxy_host': host,
        'proxy_port': str(port)
    }
    
    if username:
        config['proxy_user'] = username
    if password:
        config['proxy_password'] = password
    
    return config

# Usage with environment variables
proxy_config = create_proxy_config(
    proxy_type='socks5',
    host=os.getenv('PROXY_HOST'),
    port=os.getenv('PROXY_PORT'),
    username=os.getenv('PROXY_USER'),
    password=os.getenv('PROXY_PASSWORD')
)
```

## Advanced Features

### Fingerprint Customization

```python
def create_custom_fingerprint_profile(name, fingerprint_settings):
    """Create profile with custom fingerprint settings"""
    payload = {
        'name': name,
        'fingerprint_config': {
            'automatic_timezone': '0',
            'timezone': fingerprint_settings.get('timezone', 'America/New_York'),
            'webrtc': fingerprint_settings.get('webrtc', 'proxy'),
            'location': fingerprint_settings.get('location', 'ask'),
            'language': fingerprint_settings.get('language', ['en-US', 'en']),
            'page_language': fingerprint_settings.get('page_language', 'en-US'),
            'ua': fingerprint_settings.get('user_agent'),
            'screen_resolution': fingerprint_settings.get('resolution', '1920x1080'),
            'fonts': fingerprint_settings.get('fonts', ['all']),
            'canvas': fingerprint_settings.get('canvas', '1'),  # 1=noise, 0=off
            'webgl_image': fingerprint_settings.get('webgl', '1'),
            'audio': fingerprint_settings.get('audio', '1'),
            'do_not_track': fingerprint_settings.get('dnt', '0'),
            'hardware_concurrency': fingerprint_settings.get('cpu_cores', '4'),
            'device_memory': fingerprint_settings.get('memory', '8')
        }
    }
    
    response = requests.post(f"{API_URL}/api/v1/user/create", json=payload)
    return response.json()

# Create profile with specific fingerprint
custom_profile = create_custom_fingerprint_profile(
    name='High-Security Profile',
    fingerprint_settings={
        'timezone': 'Europe/London',
        'webrtc': 'disabled',
        'resolution': '1366x768',
        'cpu_cores': '2',
        'memory': '4'
    }
)
```

### Cookie Management

```python
def import_cookies(profile_id, cookies):
    """Import cookies into a profile"""
    # Cookies should be in Netscape format or JSON
    response = requests.post(f"{API_URL}/api/v1/user/cookie/import", json={
        'user_id': profile_id,
        'cookies': cookies
    })
    return response.json()

def export_cookies(profile_id):
    """Export cookies from a profile"""
    response = requests.get(f"{API_URL}/api/v1/user/cookie/export", params={
        'user_id': profile_id
    })
    return response.json()
```

## Troubleshooting

### Browser Won't Start

**Issue**: API returns error when starting browser

**Solutions**:
- Ensure AdsPower desktop application is running
- Check if profile ID is valid: `list_profiles()` to verify
- Verify another browser isn't already running for that profile
- Check available system resources (RAM, CPU)

```python
# Check if browser is already active
if check_browser_status(profile_id):
    stop_browser(profile_id)
    time.sleep(2)

# Then start
start_browser(profile_id)
```

### Connection Timeout

**Issue**: Selenium/Puppeteer can't connect to browser

**Solutions**:
- Wait for browser to fully initialize after starting
- Increase connection timeout in automation tool
- Verify debug port is accessible

```python
import time

def start_browser_with_retry(profile_id, max_retries=3):
    """Start browser with retry logic"""
    for attempt in range(max_retries):
        browser_info = start_browser(profile_id)
        if browser_info:
            time.sleep(3)  # Wait for full initialization
            return browser_info
        time.sleep(2)
    raise Exception("Failed to start browser after retries")
```

### Profile Sync Issues

**Issue**: Profiles not syncing across team

**Solutions**:
- Verify cloud sync is enabled in AdsPower settings
- Check network connectivity
- Manually trigger sync via API

```python
def sync_profile(profile_id):
    """Force profile sync to cloud"""
    response = requests.post(f"{API_URL}/api/v1/user/sync", json={
        'user_id': profile_id
    })
    return response.json()
```

### Memory Leaks

**Issue**: Application consuming too much memory

**Solutions**:
- Always close browsers after use with `stop_browser()`
- Limit concurrent browser instances
- Clear cache periodically

```python
def cleanup_profile_cache(profile_id):
    """Clear profile cache and temp files"""
    response = requests.post(f"{API_URL}/api/v1/user/cache/clear", json={
        'user_id': profile_id
    })
    return response.json()

# Use context manager pattern
class AdsPowerBrowser:
    def __init__(self, profile_id):
        self.profile_id = profile_id
        self.driver = None
    
    def __enter__(self):
        self.driver = connect_selenium(self.profile_id)
        return self.driver
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.driver:
            self.driver.quit()
        stop_browser(self.profile_id)

# Usage
with AdsPowerBrowser('profile_id_here') as driver:
    driver.get('https://example.com')
    # Automatic cleanup
```

### API Rate Limiting

**Issue**: Too many API requests causing failures

**Solutions**:
- Implement rate limiting in your code
- Batch operations where possible
- Add delays between requests

```python
import time
from functools import wraps

def rate_limit(calls_per_second=5):
    """Decorator to limit API calls"""
    min_interval = 1.0 / calls_per_second
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            left_to_wait = min_interval - elapsed
            if left_to_wait > 0:
                time.sleep(left_to_wait)
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_second=3)
def api_call(endpoint, **kwargs):
    """Make rate-limited API call"""
    return requests.get(f"{API_URL}{endpoint}", **kwargs)
```

## Best Practices

1. **Always close browsers**: Use `stop_browser()` or context managers to prevent resource leaks
2. **Use groups**: Organize profiles logically for easier management
3. **Rotate proxies**: Assign different proxies to profiles to avoid detection
4. **Monitor resources**: Keep track of active browsers and system resources
5. **Handle errors**: Implement proper error handling and retry logic
6. **Secure credentials**: Use environment variables for all sensitive data
7. **Regular cleanup**: Periodically clear caches and remove unused profiles
8. **Profile naming**: Use descriptive names with campaign/purpose identifiers
