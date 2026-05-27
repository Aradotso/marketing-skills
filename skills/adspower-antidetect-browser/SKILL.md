---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I use AdsPower for multi-account management
  - set up AdsPower browser profiles for marketing automation
  - configure AdsPower antidetect browser profiles
  - automate campaigns with AdsPower browser
  - manage multiple browser profiles with AdsPower
  - integrate AdsPower API for browser automation
  - create AdsPower profiles for marketing teams
  - use AdsPower for RPA and multi-account workflows
---

# AdsPower Antidetect Browser

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser solution designed for managing multiple browser profiles with unique fingerprints. It enables marketing teams to run multi-account campaigns, automation workflows, and RPA tasks while avoiding detection and account bans across platforms like social media, e-commerce, and advertising networks.

## What AdsPower Does

- **Multi-Account Management**: Create and manage hundreds of isolated browser profiles
- **Fingerprint Customization**: Unique canvas, WebGL, fonts, timezone, and other browser fingerprints per profile
- **Team Collaboration**: Share profiles and manage permissions across marketing teams
- **Automation Support**: Integrate with Selenium, Puppeteer, Playwright for RPA workflows
- **Cloud Sync**: Synchronize profiles across devices via cloud storage
- **Proxy Management**: Assign different proxies to each profile for geo-targeting

## Installation

AdsPower is a desktop application with API support:

1. Download from official source
2. Install the desktop client for your OS (Windows/Mac)
3. Launch AdsPower and create an account
4. Enable API access in Settings → API

## API Configuration

AdsPower provides a local REST API (default port: 50325) for automation:

```bash
# Check API is running
curl http://localhost:50325/api/v1/status
```

Set environment variables for API access:

```bash
export ADSPOWER_API_URL="http://localhost:50325"
export ADSPOWER_API_KEY="your_api_key_from_settings"
```

## Core API Endpoints

### List All Profiles

```python
import requests
import os

API_URL = os.getenv('ADSPOWER_API_URL', 'http://localhost:50325')
API_KEY = os.getenv('ADSPOWER_API_KEY')

def list_profiles(page=1, page_size=50):
    """List all browser profiles"""
    response = requests.get(
        f"{API_URL}/api/v1/user/list",
        params={
            'page_size': page_size,
            'page': page
        },
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    return response.json()

profiles = list_profiles()
print(f"Total profiles: {profiles['data']['count']}")
for profile in profiles['data']['list']:
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

### Create a New Profile

```python
def create_profile(name, group_id=0, proxy_config=None):
    """Create a new browser profile with custom fingerprint"""
    payload = {
        'name': name,
        'group_id': group_id,
        'fingerprint_config': {
            'automatic_timezone': 1,
            'webrtc': 'proxy',
            'location': 'proxy',
            'language': ['en-US', 'en'],
            'ua': 'random',
            'screen_resolution': 'random',
            'fonts': 'random',
            'canvas': 'random',
            'webgl_image': 'random',
            'webgl_metadata': 'random',
            'audio': 'random'
        }
    }
    
    if proxy_config:
        payload['user_proxy_config'] = proxy_config
    
    response = requests.post(
        f"{API_URL}/api/v1/user/create",
        json=payload,
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    return response.json()

# Create profile with proxy
new_profile = create_profile(
    name='Campaign Profile 1',
    proxy_config={
        'proxy_type': 'http',
        'proxy_host': 'proxy.example.com',
        'proxy_port': '8080',
        'proxy_user': os.getenv('PROXY_USERNAME'),
        'proxy_password': os.getenv('PROXY_PASSWORD')
    }
)
print(f"Created profile ID: {new_profile['data']['id']}")
```

### Start a Profile Browser

```python
def start_profile(profile_id, headless=False):
    """Launch a browser profile"""
    response = requests.get(
        f"{API_URL}/api/v1/browser/start",
        params={
            'user_id': profile_id,
            'headless': 1 if headless else 0
        },
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    data = response.json()
    return {
        'webdriver': data['data']['webdriver'],
        'debug_port': data['data']['debug_port'],
        'ws_endpoint': data['data']['ws']
    }

profile_session = start_profile('abc123')
print(f"WebDriver path: {profile_session['webdriver']}")
print(f"Debug port: {profile_session['debug_port']}")
```

### Stop a Profile Browser

```python
def stop_profile(profile_id):
    """Close a running browser profile"""
    response = requests.get(
        f"{API_URL}/api/v1/browser/stop",
        params={'user_id': profile_id},
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    return response.json()

stop_profile('abc123')
```

## Automation with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service

def connect_to_adspower_profile(profile_id):
    """Connect Selenium to an AdsPower profile"""
    # Start the profile
    session = start_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_experimental_option(
        'debuggerAddress', 
        f"127.0.0.1:{session['debug_port']}"
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(
        service=Service(session['webdriver']),
        options=chrome_options
    )
    return driver

# Use in automation workflow
driver = connect_to_adspower_profile('abc123')
driver.get('https://example.com')
print(driver.title)

# Perform marketing automation tasks
driver.find_element('id', 'username').send_keys('marketing_account')
driver.find_element('id', 'password').send_keys(os.getenv('ACCOUNT_PASSWORD'))
driver.find_element('id', 'login-button').click()

# Clean up
driver.quit()
stop_profile('abc123')
```

## Automation with Puppeteer

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const API_URL = process.env.ADSPOWER_API_URL || 'http://localhost:50325';
const API_KEY = process.env.ADSPOWER_API_KEY;

async function connectToProfile(profileId) {
    // Start the profile
    const response = await axios.get(`${API_URL}/api/v1/browser/start`, {
        params: { user_id: profileId },
        headers: { Authorization: `Bearer ${API_KEY}` }
    });
    
    const wsEndpoint = response.data.data.ws;
    
    // Connect Puppeteer
    const browser = await puppeteer.connect({
        browserWSEndpoint: wsEndpoint,
        defaultViewport: null
    });
    
    return browser;
}

async function runAutomation(profileId) {
    const browser = await connectToProfile(profileId);
    const pages = await browser.pages();
    const page = pages[0] || await browser.newPage();
    
    // Run your automation
    await page.goto('https://example.com');
    await page.type('#email', 'marketing@example.com');
    await page.click('#submit');
    
    await browser.disconnect();
}

runAutomation('abc123');
```

## Common Patterns

### Bulk Profile Management

```python
def create_campaign_profiles(count, campaign_name, proxies):
    """Create multiple profiles for a marketing campaign"""
    profiles = []
    
    for i in range(count):
        proxy = proxies[i % len(proxies)]
        profile = create_profile(
            name=f'{campaign_name} - Profile {i+1}',
            proxy_config={
                'proxy_type': 'http',
                'proxy_host': proxy['host'],
                'proxy_port': proxy['port'],
                'proxy_user': proxy.get('username'),
                'proxy_password': proxy.get('password')
            }
        )
        profiles.append(profile['data']['id'])
        print(f"Created profile {i+1}/{count}")
    
    return profiles

# Create 10 profiles for campaign
proxies = [
    {'host': 'proxy1.com', 'port': '8080'},
    {'host': 'proxy2.com', 'port': '8080'},
]
campaign_profiles = create_campaign_profiles(10, 'Holiday Campaign 2026', proxies)
```

### Profile Group Management

```python
def create_profile_group(group_name):
    """Create a profile group for organization"""
    response = requests.post(
        f"{API_URL}/api/v1/group/create",
        json={'group_name': group_name},
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    return response.json()['data']['group_id']

def assign_profile_to_group(profile_id, group_id):
    """Move profile to a group"""
    response = requests.post(
        f"{API_URL}/api/v1/user/update",
        json={'user_id': profile_id, 'group_id': group_id},
        headers={'Authorization': f'Bearer {API_KEY}'}
    )
    return response.json()

# Organize profiles
social_group = create_profile_group('Social Media Campaigns')
assign_profile_to_group('abc123', social_group)
```

### Concurrent Profile Automation

```python
import concurrent.futures

def run_task_on_profile(profile_id, task_func):
    """Execute a task on a specific profile"""
    driver = connect_to_adspower_profile(profile_id)
    try:
        result = task_func(driver)
        return {'profile_id': profile_id, 'success': True, 'result': result}
    except Exception as e:
        return {'profile_id': profile_id, 'success': False, 'error': str(e)}
    finally:
        driver.quit()
        stop_profile(profile_id)

def my_marketing_task(driver):
    """Example marketing automation task"""
    driver.get('https://example.com')
    return driver.title

# Run task across multiple profiles concurrently
profile_ids = ['abc123', 'def456', 'ghi789']
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    futures = [
        executor.submit(run_task_on_profile, pid, my_marketing_task)
        for pid in profile_ids
    ]
    results = [f.result() for f in concurrent.futures.as_completed(futures)]

for result in results:
    print(f"Profile {result['profile_id']}: {result}")
```

## Troubleshooting

### API Connection Issues

```python
def check_adspower_status():
    """Verify AdsPower is running"""
    try:
        response = requests.get(f"{API_URL}/api/v1/status", timeout=5)
        if response.status_code == 200:
            print("AdsPower API is running")
            return True
    except requests.exceptions.ConnectionError:
        print("ERROR: Cannot connect to AdsPower. Ensure application is running.")
        return False

check_adspower_status()
```

### Profile Not Starting

- Ensure AdsPower desktop application is running
- Check profile isn't already open
- Verify sufficient system resources (RAM, CPU)
- Check proxy configuration is valid

### Browser Automation Failures

```python
def safe_start_profile(profile_id, max_retries=3):
    """Start profile with retry logic"""
    for attempt in range(max_retries):
        try:
            session = start_profile(profile_id)
            return session
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                time.sleep(2)
            else:
                raise
```

### Memory Management

```python
def cleanup_idle_profiles():
    """Close profiles that have been idle"""
    profiles = list_profiles()
    for profile in profiles['data']['list']:
        if profile.get('browser_status') == 'Active':
            stop_profile(profile['user_id'])
            print(f"Stopped profile: {profile['name']}")
```

## Best Practices

1. **Use environment variables** for sensitive data (API keys, passwords, proxies)
2. **Implement retry logic** for API calls and browser automation
3. **Close profiles** after use to free system resources
4. **Use profile groups** to organize campaigns and teams
5. **Rotate proxies** across profiles to avoid detection
6. **Monitor profile health** and fingerprint quality regularly
7. **Limit concurrent profiles** based on system capabilities
