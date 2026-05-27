---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaign management
triggers:
  - how do I create AdsPower browser profiles
  - automate AdsPower browser sessions
  - manage multiple accounts with AdsPower
  - set up AdsPower API automation
  - configure AdsPower fingerprint profiles
  - integrate AdsPower with marketing automation
  - control AdsPower browsers programmatically
  - launch AdsPower profiles via API
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser that allows marketing teams to manage multiple accounts across platforms without fingerprint detection. It creates isolated browser profiles with unique fingerprints, enabling safe multi-account operations for social media marketing, e-commerce, advertising campaigns, and automated workflows.

## Overview

AdsPower provides:
- **Isolated Browser Profiles**: Each profile has unique browser fingerprints (canvas, WebGL, fonts, etc.)
- **API Control**: Programmatic profile creation, configuration, and browser automation
- **Proxy Management**: Per-profile proxy configuration for different geolocations
- **Team Collaboration**: Cloud sync and profile sharing across team members
- **RPA Integration**: Compatible with Selenium, Puppeteer, and Playwright for automation

## Installation

AdsPower runs as a local application with an API server. Download from the official website and install the desktop application.

### Starting the API Server

The AdsPower application runs a local API server (default: `http://localhost:50325`).

## API Authentication

Most AdsPower API calls require the application to be running locally. The API uses a simple REST interface without authentication for local connections.

Base URL: `http://localhost:50325/api/v1`

## Core API Operations

### Listing Browser Profiles

```python
import requests
import os

ADSPOWER_API = os.getenv('ADSPOWER_API_URL', 'http://localhost:50325/api/v1')

def list_profiles(group_id=None, page=1, page_size=100):
    """List all browser profiles"""
    params = {
        'page': page,
        'page_size': page_size
    }
    if group_id:
        params['group_id'] = group_id
    
    response = requests.get(f'{ADSPOWER_API}/user/list', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['list']
    else:
        raise Exception(f"Error listing profiles: {data['msg']}")

# Get all profiles
profiles = list_profiles()
for profile in profiles:
    print(f"Profile: {profile['name']} (ID: {profile['user_id']})")
```

### Creating a New Profile

```python
def create_profile(name, proxy_config=None, fingerprint_config=None):
    """Create a new browser profile"""
    payload = {
        'name': name,
        'group_id': '0',  # Default group
        'domain_name': '',
        'open_urls': [],
        'repeat_config': ['0']
    }
    
    # Add proxy configuration
    if proxy_config:
        payload['user_proxy_config'] = {
            'proxy_soft': 'other',  # no_proxy, other, luminati, etc.
            'proxy_type': proxy_config.get('type', 'http'),  # http, https, socks5
            'proxy_host': proxy_config.get('host'),
            'proxy_port': proxy_config.get('port'),
            'proxy_user': proxy_config.get('username', ''),
            'proxy_password': proxy_config.get('password', '')
        }
    
    # Add fingerprint customization
    if fingerprint_config:
        payload['fingerprint_config'] = fingerprint_config
    
    response = requests.post(f'{ADSPOWER_API}/user/create', json=payload)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['id']
    else:
        raise Exception(f"Error creating profile: {data['msg']}")

# Create profile with proxy
proxy = {
    'type': 'http',
    'host': os.getenv('PROXY_HOST'),
    'port': os.getenv('PROXY_PORT'),
    'username': os.getenv('PROXY_USER'),
    'password': os.getenv('PROXY_PASS')
}

profile_id = create_profile('Marketing Campaign Profile 1', proxy_config=proxy)
print(f"Created profile: {profile_id}")
```

### Starting a Browser Profile

```python
def start_profile(profile_id, headless=False):
    """Start a browser profile and get automation connection details"""
    params = {
        'user_id': profile_id,
        'headless': 1 if headless else 0
    }
    
    response = requests.get(f'{ADSPOWER_API}/browser/start', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'selenium_address': data['data']['ws']['selenium'],
            'webdriver_path': data['data']['webdriver'],
            'debug_port': data['data']['debug_port']
        }
    else:
        raise Exception(f"Error starting profile: {data['msg']}")

# Start profile and get connection info
connection = start_profile(profile_id)
print(f"Selenium: {connection['selenium_address']}")
print(f"Chrome DevTools: {connection['debug_port']}")
```

### Stopping a Browser Profile

```python
def stop_profile(profile_id):
    """Stop a running browser profile"""
    params = {'user_id': profile_id}
    
    response = requests.get(f'{ADSPOWER_API}/browser/stop', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return True
    else:
        raise Exception(f"Error stopping profile: {data['msg']}")

stop_profile(profile_id)
```

## Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_to_adspower_profile(profile_id):
    """Connect Selenium to an AdsPower profile"""
    # Start the profile
    connection = start_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option('debuggerAddress', f"127.0.0.1:{connection['debug_port']}")
    
    # Create driver
    driver = webdriver.Chrome(
        executable_path=connection['webdriver_path'],
        options=chrome_options
    )
    
    return driver

# Use with Selenium
driver = connect_to_adspower_profile(profile_id)
driver.get('https://example.com')
print(driver.title)
driver.quit()
stop_profile(profile_id)
```

## Puppeteer Integration

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const ADSPOWER_API = process.env.ADSPOWER_API_URL || 'http://localhost:50325/api/v1';

async function connectToAdsPowerProfile(profileId) {
  // Start the profile
  const response = await axios.get(`${ADSPOWER_API}/browser/start`, {
    params: { user_id: profileId }
  });
  
  if (response.data.code !== 0) {
    throw new Error(`Failed to start profile: ${response.data.msg}`);
  }
  
  const { ws } = response.data.data;
  
  // Connect Puppeteer
  const browser = await puppeteer.connect({
    browserWSEndpoint: ws.puppeteer,
    defaultViewport: null
  });
  
  return browser;
}

async function stopProfile(profileId) {
  await axios.get(`${ADSPOWER_API}/browser/stop`, {
    params: { user_id: profileId }
  });
}

// Usage
(async () => {
  const profileId = 'your_profile_id';
  const browser = await connectToAdsPowerProfile(profileId);
  
  const pages = await browser.pages();
  const page = pages[0];
  
  await page.goto('https://example.com');
  console.log(await page.title());
  
  await browser.disconnect();
  await stopProfile(profileId);
})();
```

## Profile Management Patterns

### Batch Profile Creation

```python
def create_campaign_profiles(campaign_name, count, proxy_list):
    """Create multiple profiles for a marketing campaign"""
    profile_ids = []
    
    for i in range(count):
        proxy = proxy_list[i % len(proxy_list)]
        name = f"{campaign_name} - Profile {i+1}"
        
        profile_id = create_profile(name, proxy_config=proxy)
        profile_ids.append(profile_id)
        print(f"Created: {name} ({profile_id})")
    
    return profile_ids

# Create 10 profiles for a campaign
proxies = [
    {'type': 'http', 'host': '10.0.0.1', 'port': '8080'},
    {'type': 'http', 'host': '10.0.0.2', 'port': '8080'},
    # Add more proxies
]

campaign_profiles = create_campaign_profiles('Facebook Ads Q1', 10, proxies)
```

### Profile Groups Management

```python
def create_group(group_name):
    """Create a profile group"""
    payload = {'group_name': group_name}
    response = requests.post(f'{ADSPOWER_API}/group/create', json=payload)
    data = response.json()
    return data['data']['group_id'] if data['code'] == 0 else None

def move_profile_to_group(profile_id, group_id):
    """Move profile to a group"""
    payload = {
        'user_ids': [profile_id],
        'group_id': group_id
    }
    response = requests.post(f'{ADSPOWER_API}/user/update', json=payload)
    return response.json()['code'] == 0

# Organize profiles
group_id = create_group('Instagram Marketing')
for profile_id in campaign_profiles:
    move_profile_to_group(profile_id, group_id)
```

## Advanced Fingerprint Configuration

```python
def create_profile_with_custom_fingerprint(name, fingerprint_config):
    """Create profile with custom fingerprint settings"""
    payload = {
        'name': name,
        'group_id': '0',
        'fingerprint_config': {
            'automatic_timezone': '1',  # Auto-detect from IP
            'webrtc': 'forward',  # forward, disabled, custom
            'location': 'ask',  # ask, allow, block
            'language': ['en-US', 'en'],
            'page_language': 'en-US',
            'ua': '',  # Empty for random
            'screen_resolution': '1920_1080',
            'fonts': ['all'],  # or specific font list
            'canvas': '1',  # Noise level
            'webgl_image': '1',
            'webgl_metadata': 'mask',
            'audio': '1',
            'do_not_track': '1',
            'hardware_concurrency': '4',
            'device_memory': '8'
        }
    }
    
    response = requests.post(f'{ADSPOWER_API}/user/create', json=payload)
    return response.json()['data']['id'] if response.json()['code'] == 0 else None

# Create high-anonymity profile
fingerprint = {
    'webrtc': 'disabled',
    'canvas': '2',  # Max noise
    'webgl_image': '2',
    'audio': '2'
}

anon_profile = create_profile_with_custom_fingerprint('High Anonymity Profile', fingerprint)
```

## Common Workflows

### Social Media Automation

```python
import time

def automate_social_media_posting(profile_id, platform_url, post_content):
    """Automate posting to social media"""
    driver = connect_to_adspower_profile(profile_id)
    
    try:
        driver.get(platform_url)
        time.sleep(3)
        
        # Your automation logic here
        # e.g., find post button, enter content, submit
        
        print(f"Posted content from profile {profile_id}")
        
    finally:
        driver.quit()
        stop_profile(profile_id)

# Run across multiple profiles
for profile in campaign_profiles[:5]:
    automate_social_media_posting(profile, 'https://facebook.com', 'Campaign post')
    time.sleep(60)  # Delay between profiles
```

## Troubleshooting

### Profile Won't Start

```python
def check_profile_status(profile_id):
    """Check if profile is already running"""
    params = {'user_id': profile_id}
    response = requests.get(f'{ADSPOWER_API}/browser/active', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['status'] == 'Active'
    return False

# Check before starting
if check_profile_status(profile_id):
    print("Profile already running, stopping first...")
    stop_profile(profile_id)
    time.sleep(2)

connection = start_profile(profile_id)
```

### API Connection Issues

```python
def test_adspower_connection():
    """Test if AdsPower API is accessible"""
    try:
        response = requests.get(f'{ADSPOWER_API}/status', timeout=5)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        return False

if not test_adspower_connection():
    print("ERROR: AdsPower application is not running or API is not accessible")
    print("Please start the AdsPower desktop application")
    exit(1)
```

### Proxy Validation

```python
def check_proxy_connection(profile_id):
    """Verify proxy is working for a profile"""
    driver = connect_to_adspower_profile(profile_id)
    
    try:
        driver.get('https://api.ipify.org?format=json')
        time.sleep(2)
        ip_data = driver.find_element('tag name', 'pre').text
        print(f"Profile IP: {ip_data}")
        return ip_data
    finally:
        driver.quit()
        stop_profile(profile_id)
```

## Environment Variables

```bash
# Optional: Custom API URL if using remote AdsPower instance
ADSPOWER_API_URL=http://localhost:50325/api/v1

# Proxy credentials
PROXY_HOST=proxy.example.com
PROXY_PORT=8080
PROXY_USER=username
PROXY_PASS=password
```

## Best Practices

1. **Always close profiles**: Call `stop_profile()` after automation to free resources
2. **Respect rate limits**: Add delays between profile actions to avoid detection
3. **Use profile groups**: Organize profiles by campaign or platform
4. **Rotate proxies**: Assign different proxies to profiles for better anonymity
5. **Regular fingerprint updates**: Periodically recreate profiles with fresh fingerprints
6. **Monitor profile health**: Check proxy connectivity and profile status before automation
