---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I use AdsPower for managing multiple accounts
  - set up AdsPower browser profiles for marketing automation
  - create antidetect browser profiles with AdsPower
  - automate marketing campaigns with AdsPower
  - manage multiple browser fingerprints for advertising
  - integrate AdsPower API for profile management
  - launch AdsPower browser profiles programmatically
  - configure AdsPower for multi-account operations
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that enables marketing teams to manage multiple browser profiles with unique fingerprints. This is essential for running multi-account marketing campaigns, affiliate marketing, social media management, e-commerce operations, and advertising automation while avoiding detection and account bans.

## What AdsPower Does

- **Multi-Account Management**: Create and manage hundreds of isolated browser profiles
- **Fingerprint Masking**: Each profile has unique browser fingerprints (canvas, WebGL, fonts, etc.)
- **Team Collaboration**: Share profiles and synchronize data across team members
- **Automation Support**: API and RPA integration for automated workflows
- **Proxy Management**: Configure different proxies per profile for geo-targeting
- **Cloud Sync**: Store profiles in the cloud for access from multiple devices

## Installation

### Desktop Application

1. Download AdsPower from the official website for your OS (Windows, macOS)
2. Install and launch the application
3. Create an account or log in
4. The application provides both GUI and API access

### API Access

AdsPower provides a local API server that runs when the application is active (default port: 50325).

No additional SDK installation required - use standard HTTP requests to interact with the API.

## API Configuration

The API runs locally when AdsPower is running:

```bash
# Default API endpoint
http://localhost:50325/api/v1/
```

### Environment Variables

```bash
ADSPOWER_API_URL=http://localhost:50325/api/v1
ADSPOWER_API_KEY=your_api_key_here
```

## Core API Operations

### 1. List All Profiles

```python
import requests
import os

API_URL = os.getenv('ADSPOWER_API_URL', 'http://localhost:50325/api/v1')

def list_profiles(page=1, page_size=100):
    """List all browser profiles"""
    response = requests.get(
        f'{API_URL}/user/list',
        params={
            'page': page,
            'page_size': page_size
        }
    )
    return response.json()

profiles = list_profiles()
print(f"Total profiles: {profiles['data']['total']}")
for profile in profiles['data']['list']:
    print(f"ID: {profile['user_id']}, Name: {profile['name']}")
```

### 2. Create a New Profile

```python
def create_profile(name, group_id=0, proxy_config=None):
    """Create a new browser profile with custom fingerprint"""
    payload = {
        'name': name,
        'group_id': group_id,
        'domain_name': '',
        'open_urls': ['https://www.google.com'],
        'repeat_config': [0],  # Fingerprint randomization level
        'username': '',
        'password': '',
        'fakey': '',
        'cookie': '',
        'ignore_cookie_error': 0,
        'ip': '',
        'country': 'US',
        'region': '',
        'city': '',
        'remark': '',
        'ipchecker': 'ip2location',
        'sys_app_cate_id': 0,
        'user_proxy_config': proxy_config or {}
    }
    
    response = requests.post(f'{API_URL}/user/create', json=payload)
    return response.json()

# Create profile without proxy
new_profile = create_profile('Marketing Account 1')
print(f"Created profile: {new_profile['data']['id']}")
```

### 3. Create Profile with Proxy

```python
def create_profile_with_proxy(name, proxy_type, proxy_host, proxy_port, 
                              proxy_user=None, proxy_password=None):
    """Create profile with proxy configuration"""
    proxy_config = {
        'proxy_soft': 'other',
        'proxy_type': proxy_type,  # 'http', 'https', 'socks5'
        'proxy_host': proxy_host,
        'proxy_port': proxy_port,
        'proxy_user': proxy_user or '',
        'proxy_password': proxy_password or ''
    }
    
    return create_profile(name, proxy_config=proxy_config)

# Create profile with SOCKS5 proxy
profile = create_profile_with_proxy(
    name='FB Ads Account',
    proxy_type='socks5',
    proxy_host='proxy.example.com',
    proxy_port='1080',
    proxy_user=os.getenv('PROXY_USER'),
    proxy_password=os.getenv('PROXY_PASSWORD')
)
```

### 4. Launch a Profile

```python
def launch_profile(profile_id, open_urls=None, launch_args=None):
    """Launch a browser profile and get debugging info"""
    params = {
        'user_id': profile_id
    }
    
    if open_urls:
        params['open_urls'] = open_urls
    if launch_args:
        params['launch_args'] = launch_args
    
    response = requests.get(f'{API_URL}/browser/start', params=params)
    result = response.json()
    
    if result['code'] == 0:
        return {
            'webdriver': result['data']['webdriver'],
            'ws_endpoint': result['data']['ws']['selenium'],
            'debug_port': result['data']['debug_port']
        }
    else:
        raise Exception(f"Failed to launch: {result['msg']}")

# Launch profile
launch_info = launch_profile('profile_id_here')
print(f"WebDriver path: {launch_info['webdriver']}")
print(f"Debug port: {launch_info['debug_port']}")
```

### 5. Close a Profile

```python
def close_profile(profile_id):
    """Close an active browser profile"""
    response = requests.get(
        f'{API_URL}/browser/stop',
        params={'user_id': profile_id}
    )
    return response.json()

result = close_profile('profile_id_here')
print(f"Profile closed: {result['msg']}")
```

### 6. Update Profile

```python
def update_profile(profile_id, updates):
    """Update profile configuration"""
    payload = {
        'user_id': profile_id,
        **updates
    }
    
    response = requests.post(f'{API_URL}/user/update', json=payload)
    return response.json()

# Update profile name and proxy
update_profile('profile_id_here', {
    'name': 'Updated Profile Name',
    'user_proxy_config': {
        'proxy_soft': 'other',
        'proxy_type': 'http',
        'proxy_host': 'newproxy.example.com',
        'proxy_port': '8080'
    }
})
```

### 7. Delete Profile

```python
def delete_profile(profile_ids):
    """Delete one or more profiles"""
    response = requests.post(
        f'{API_URL}/user/delete',
        json={'user_ids': profile_ids if isinstance(profile_ids, list) else [profile_ids]}
    )
    return response.json()

delete_profile(['profile_id_1', 'profile_id_2'])
```

## Selenium Automation Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium_to_adspower(profile_id):
    """Connect Selenium to AdsPower profile"""
    # Launch the profile
    launch_info = launch_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        'debuggerAddress', 
        f"127.0.0.1:{launch_info['debug_port']}"
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(
        executable_path=launch_info['webdriver'],
        options=chrome_options
    )
    
    return driver

# Use Selenium with AdsPower
driver = connect_selenium_to_adspower('your_profile_id')
driver.get('https://www.facebook.com')
# Perform automation tasks
driver.quit()
```

## Playwright Integration

```python
from playwright.sync_api import sync_playwright

def connect_playwright_to_adspower(profile_id):
    """Connect Playwright to AdsPower profile"""
    launch_info = launch_profile(profile_id)
    
    with sync_playwright() as p:
        browser = p.chromium.connect_over_cdp(
            f"http://127.0.0.1:{launch_info['debug_port']}"
        )
        context = browser.contexts[0]
        page = context.pages[0]
        
        return browser, page

# Use Playwright with AdsPower
browser, page = connect_playwright_to_adspower('your_profile_id')
page.goto('https://www.google.com')
# Perform automation
browser.close()
```

## Common Patterns

### Bulk Profile Creation

```python
def create_bulk_profiles(count, name_prefix, proxy_list):
    """Create multiple profiles with different proxies"""
    profiles = []
    
    for i in range(count):
        proxy = proxy_list[i % len(proxy_list)]
        
        profile = create_profile_with_proxy(
            name=f'{name_prefix}_{i+1}',
            proxy_type=proxy['type'],
            proxy_host=proxy['host'],
            proxy_port=proxy['port'],
            proxy_user=proxy.get('user'),
            proxy_password=proxy.get('password')
        )
        
        profiles.append(profile)
        print(f"Created profile {i+1}/{count}")
    
    return profiles

proxies = [
    {'type': 'http', 'host': 'proxy1.com', 'port': '8080'},
    {'type': 'socks5', 'host': 'proxy2.com', 'port': '1080'},
]

profiles = create_bulk_profiles(10, 'Campaign_Profile', proxies)
```

### Profile Group Management

```python
def get_groups():
    """Get all profile groups"""
    response = requests.get(f'{API_URL}/group/list')
    return response.json()

def create_group(group_name, parent_id=0):
    """Create a new profile group"""
    response = requests.post(
        f'{API_URL}/group/create',
        json={
            'group_name': group_name,
            'parent_id': parent_id
        }
    )
    return response.json()

# Organize profiles by campaign
facebook_group = create_group('Facebook Campaigns')
twitter_group = create_group('Twitter Campaigns')
```

### Batch Operations

```python
def batch_launch_profiles(profile_ids, max_concurrent=5):
    """Launch multiple profiles with concurrency limit"""
    import concurrent.futures
    
    def launch_single(pid):
        try:
            return launch_profile(pid)
        except Exception as e:
            return {'error': str(e), 'profile_id': pid}
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_concurrent) as executor:
        results = list(executor.map(launch_single, profile_ids))
    
    return results

# Launch 10 profiles with max 5 concurrent
results = batch_launch_profiles(['id1', 'id2', 'id3'], max_concurrent=5)
```

## Configuration Best Practices

### Fingerprint Randomization

```python
def create_randomized_profile(name, randomization_level=2):
    """
    Create profile with fingerprint randomization
    Levels: 0 (minimal), 1 (medium), 2 (high)
    """
    payload = {
        'name': name,
        'repeat_config': [randomization_level],
        'fingerprint_config': {
            'automatic_timezone': 1,
            'webrtc': 'proxy',
            'location': 'ask',
            'language': ['en-US', 'en'],
            'page_language': 'en-US',
            'hardware_concurrency': 4,
            'device_memory': 8
        }
    }
    
    response = requests.post(f'{API_URL}/user/create', json=payload)
    return response.json()
```

### Cookie Management

```python
def import_cookies(profile_id, cookies):
    """Import cookies to a profile"""
    payload = {
        'user_id': profile_id,
        'cookie': cookies  # JSON string or cookie format
    }
    
    response = requests.post(f'{API_URL}/user/update', json=payload)
    return response.json()

def export_cookies(profile_id):
    """Export cookies from a profile"""
    response = requests.get(
        f'{API_URL}/user/cookie/export',
        params={'user_id': profile_id}
    )
    return response.json()
```

## Troubleshooting

### Profile Won't Launch

```python
def check_profile_status(profile_id):
    """Check if profile is available"""
    response = requests.get(
        f'{API_URL}/browser/active',
        params={'user_id': profile_id}
    )
    result = response.json()
    
    if result['code'] == 0:
        return result['data']['status']  # 'Active' or 'Inactive'
    return None

# Ensure profile is closed before launching
status = check_profile_status('profile_id')
if status == 'Active':
    close_profile('profile_id')
```

### Proxy Verification

```python
def check_proxy_connection(profile_id):
    """Verify proxy is working"""
    launch_info = launch_profile(profile_id, open_urls=['https://api.ipify.org'])
    # Manual verification or automated check
    return launch_info
```

### API Connection Issues

```python
def test_api_connection():
    """Test if AdsPower API is accessible"""
    try:
        response = requests.get(f'{API_URL}/status/browser', timeout=5)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        return False

if not test_api_connection():
    print("AdsPower application is not running or API is not accessible")
```

### Rate Limiting

AdsPower API may have rate limits. Implement retry logic:

```python
import time
from functools import wraps

def retry_on_failure(max_retries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
            return None
        return wrapper
    return decorator

@retry_on_failure(max_retries=3)
def safe_launch_profile(profile_id):
    return launch_profile(profile_id)
```

## Additional Resources

- API runs locally on port 50325 by default
- WebDriver paths are provided by the launch API response
- Each profile maintains isolated cookies, cache, and local storage
- Profiles can be shared across team members via cloud sync
- Use environment variables for sensitive data (API keys, proxy credentials)
