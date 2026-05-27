---
name: adspower-antidetect-browser
description: Use AdsPower antidetect browser for multi-account management, browser fingerprint protection, and marketing automation
triggers:
  - manage multiple browser profiles with adspower
  - create antidetect browser profiles for marketing
  - automate browser fingerprinting with adspower
  - set up multi-account browser automation
  - use adspower api for profile management
  - configure adspower browser profiles
  - control adspower profiles programmatically
  - integrate adspower with automation scripts
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple accounts, browser automation, and marketing campaigns. It provides isolated browser profiles with unique fingerprints to avoid detection and account linking across platforms.

## What AdsPower Does

- **Multi-Account Management**: Create and manage hundreds of isolated browser profiles
- **Fingerprint Protection**: Each profile has unique browser fingerprints (canvas, WebGL, fonts, etc.)
- **Automation API**: Control profiles programmatically via local HTTP API
- **Team Collaboration**: Cloud-based profile synchronization for marketing teams
- **RPA Integration**: Compatible with Selenium, Puppeteer, and Playwright

## Installation

### Download AdsPower Client

1. Download from official website: https://www.adspower.com/
2. Install the desktop application (Windows/macOS)
3. Launch AdsPower and create an account
4. Enable Local API in settings (default port: 50325)

### API Access Setup

AdsPower runs a local HTTP API server on your machine when the application is running.

**Default API Endpoint**: `http://local.adspower.com:50325`

No additional installation required for API usage - just ensure AdsPower app is running.

## Configuration

### Enable Local API

1. Open AdsPower application
2. Go to Settings → Advanced
3. Enable "Local API"
4. Note the API port (default: 50325)
5. Get your API key from Settings → API

### Environment Variables

```bash
ADSPOWER_API_URL=http://local.adspower.com:50325
ADSPOWER_API_KEY=your_api_key_here
```

## Core API Operations

### Check API Status

```python
import requests
import os

API_URL = os.getenv('ADSPOWER_API_URL', 'http://local.adspower.com:50325')

def check_status():
    response = requests.get(f'{API_URL}/status')
    return response.json()

status = check_status()
print(status)
```

### List All Profiles

```javascript
const axios = require('axios');

const API_URL = process.env.ADSPOWER_API_URL || 'http://local.adspower.com:50325';

async function listProfiles(groupId = null, page = 1, pageSize = 50) {
  const params = {
    page_size: pageSize,
    page: page
  };
  
  if (groupId) {
    params.group_id = groupId;
  }
  
  const response = await axios.get(`${API_URL}/api/v1/user/list`, { params });
  return response.data;
}

listProfiles().then(data => console.log(data));
```

### Create a New Profile

```python
import requests
import os

API_URL = os.getenv('ADSPOWER_API_URL', 'http://local.adspower.com:50325')

def create_profile(name, group_id=None, proxy=None):
    """
    Create a new browser profile
    
    Args:
        name: Profile name
        group_id: Group ID (optional)
        proxy: Proxy configuration dict (optional)
    """
    payload = {
        'name': name,
        'fingerprint_config': {
            'automatic_timezone': 1,
            'webrtc': 'proxy',
            'location': 'proxy',
            'language': ['en-US', 'en']
        }
    }
    
    if group_id:
        payload['group_id'] = group_id
    
    if proxy:
        payload['user_proxy_config'] = {
            'proxy_type': proxy.get('type', 'http'),
            'proxy_host': proxy['host'],
            'proxy_port': proxy['port'],
            'proxy_user': proxy.get('username'),
            'proxy_password': proxy.get('password')
        }
    
    response = requests.post(f'{API_URL}/api/v1/user/create', json=payload)
    return response.json()

# Example usage
proxy_config = {
    'type': 'http',
    'host': '192.168.1.1',
    'port': 8080,
    'username': os.getenv('PROXY_USER'),
    'password': os.getenv('PROXY_PASS')
}

profile = create_profile('Marketing Account 1', proxy=proxy_config)
print(f"Profile ID: {profile['data']['id']}")
```

### Start a Profile (Open Browser)

```python
def start_profile(profile_id, headless=False):
    """
    Start/open a browser profile
    
    Returns selenium/puppeteer connection details
    """
    params = {
        'user_id': profile_id,
        'ip_tab': 0,  # Don't open IP check tab
        'headless': 1 if headless else 0
    }
    
    response = requests.get(f'{API_URL}/api/v1/browser/start', params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'selenium_address': data['data']['ws']['selenium'],
            'webdriver_path': data['data']['webdriver'],
            'debug_port': data['data']['debug_port']
        }
    else:
        raise Exception(f"Failed to start profile: {data['msg']}")

# Start profile and get connection details
connection = start_profile('profile_id_here')
print(f"Selenium address: {connection['selenium_address']}")
```

### Control Profile with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(profile_id):
    """Connect to AdsPower profile using Selenium"""
    
    # Start the profile
    connection = start_profile(profile_id)
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option('debuggerAddress', f"127.0.0.1:{connection['debug_port']}")
    
    # Connect to the browser
    driver = webdriver.Chrome(
        executable_path=connection['webdriver_path'],
        options=chrome_options
    )
    
    return driver

# Example usage
driver = connect_selenium('your_profile_id')
driver.get('https://www.google.com')
print(driver.title)
# ... perform automation tasks
driver.quit()
```

### Control Profile with Puppeteer

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const API_URL = process.env.ADSPOWER_API_URL || 'http://local.adspower.com:50325';

async function connectPuppeteer(profileId) {
  // Start the profile
  const startResponse = await axios.get(`${API_URL}/api/v1/browser/start`, {
    params: {
      user_id: profileId,
      ip_tab: 0
    }
  });
  
  const { ws } = startResponse.data.data;
  
  // Connect puppeteer to the browser
  const browser = await puppeteer.connect({
    browserWSEndpoint: ws.puppeteer,
    defaultViewport: null
  });
  
  return browser;
}

// Example usage
(async () => {
  const browser = await connectPuppeteer('your_profile_id');
  const pages = await browser.pages();
  const page = pages[0] || await browser.newPage();
  
  await page.goto('https://www.example.com');
  console.log(await page.title());
  
  // ... automation tasks
  
  await browser.disconnect();
})();
```

### Stop/Close a Profile

```python
def stop_profile(profile_id):
    """Close/stop a browser profile"""
    params = {'user_id': profile_id}
    response = requests.get(f'{API_URL}/api/v1/browser/stop', params=params)
    return response.json()

stop_profile('profile_id_here')
```

### Update Profile Configuration

```python
def update_profile(profile_id, updates):
    """
    Update profile settings
    
    Args:
        profile_id: Profile ID
        updates: Dict with fields to update
    """
    payload = {
        'user_id': profile_id,
        **updates
    }
    
    response = requests.post(f'{API_URL}/api/v1/user/update', json=payload)
    return response.json()

# Update proxy settings
update_profile('profile_id', {
    'user_proxy_config': {
        'proxy_type': 'socks5',
        'proxy_host': '192.168.1.100',
        'proxy_port': 1080
    }
})
```

### Delete a Profile

```python
def delete_profile(profile_id):
    """Permanently delete a profile"""
    payload = {'user_ids': [profile_id]}
    response = requests.post(f'{API_URL}/api/v1/user/delete', json=payload)
    return response.json()
```

## Common Patterns

### Bulk Profile Creation for Campaign

```python
def create_campaign_profiles(campaign_name, count, proxies):
    """Create multiple profiles for a marketing campaign"""
    profile_ids = []
    
    for i in range(count):
        proxy = proxies[i % len(proxies)]  # Rotate through proxies
        
        profile = create_profile(
            name=f"{campaign_name} - Account {i+1}",
            proxy=proxy
        )
        
        if profile['code'] == 0:
            profile_ids.append(profile['data']['id'])
            print(f"Created profile {i+1}/{count}")
        else:
            print(f"Failed to create profile {i+1}: {profile['msg']}")
    
    return profile_ids

# Example usage
proxies = [
    {'host': '192.168.1.1', 'port': 8080},
    {'host': '192.168.1.2', 'port': 8080},
]

profiles = create_campaign_profiles('Facebook Campaign 2026', 10, proxies)
```

### Parallel Automation Across Profiles

```python
from concurrent.futures import ThreadPoolExecutor
import time

def automate_profile(profile_id, task):
    """Run automation task on a single profile"""
    try:
        driver = connect_selenium(profile_id)
        
        # Execute task
        task(driver)
        
        driver.quit()
        stop_profile(profile_id)
        
        return {'profile_id': profile_id, 'status': 'success'}
    except Exception as e:
        return {'profile_id': profile_id, 'status': 'failed', 'error': str(e)}

def my_automation_task(driver):
    """Your automation logic here"""
    driver.get('https://example.com')
    time.sleep(2)
    # ... perform actions

# Run automation on multiple profiles in parallel
profile_ids = ['id1', 'id2', 'id3', 'id4']

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(
        lambda pid: automate_profile(pid, my_automation_task),
        profile_ids
    ))

for result in results:
    print(f"Profile {result['profile_id']}: {result['status']}")
```

### Profile Manager Class

```python
class AdsPowerProfileManager:
    def __init__(self, api_url=None):
        self.api_url = api_url or os.getenv('ADSPOWER_API_URL', 'http://local.adspower.com:50325')
    
    def create(self, name, **kwargs):
        return create_profile(name, **kwargs)
    
    def start(self, profile_id):
        return start_profile(profile_id)
    
    def stop(self, profile_id):
        return stop_profile(profile_id)
    
    def list_all(self, group_id=None):
        response = requests.get(f'{self.api_url}/api/v1/user/list', params={
            'group_id': group_id,
            'page_size': 100
        })
        return response.json()['data']['list']
    
    def get_by_name(self, name):
        profiles = self.list_all()
        return next((p for p in profiles if p['name'] == name), None)

# Usage
manager = AdsPowerProfileManager()
profile = manager.create('Test Account')
connection = manager.start(profile['data']['id'])
```

## Troubleshooting

### API Connection Errors

**Issue**: Cannot connect to API endpoint

**Solutions**:
- Ensure AdsPower application is running
- Check API is enabled in Settings → Advanced
- Verify port 50325 is not blocked by firewall
- Try accessing `http://local.adspower.com:50325/status` in browser

### Profile Won't Start

**Issue**: Browser profile fails to start

**Solutions**:
- Check if profile is already running (stop it first)
- Verify proxy settings are valid
- Ensure sufficient system resources
- Check AdsPower app logs for errors

### Selenium Connection Issues

**Issue**: Cannot connect Selenium to profile

**Solutions**:
- Use the correct `webdriver_path` from start response
- Ensure Chrome version matches webdriver version
- Use `debuggerAddress` option instead of connecting to WebSocket directly

### Fingerprint Detection

**Issue**: Accounts still getting linked/detected

**Solutions**:
- Use different proxies for each profile
- Enable automatic timezone based on proxy location
- Configure realistic fingerprint settings (language, resolution, etc.)
- Don't use same payment methods across profiles

### Rate Limiting

**Issue**: API returns rate limit errors

**Solutions**:
- Add delays between API calls (100-500ms recommended)
- Use batch operations where available
- Implement exponential backoff retry logic

```python
import time
from functools import wraps

def retry_with_backoff(retries=3, backoff_in_seconds=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            x = 0
            while True:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if x == retries:
                        raise
                    wait = backoff_in_seconds * (2 ** x)
                    time.sleep(wait)
                    x += 1
        return wrapper
    return decorator

@retry_with_backoff(retries=3)
def safe_api_call(url, **kwargs):
    return requests.get(url, **kwargs)
```

## Best Practices

1. **Always close profiles** after automation to free resources
2. **Use unique proxies** for each profile to avoid fingerprint correlation
3. **Implement proper error handling** for API calls
4. **Store profile IDs** in database for campaign management
5. **Monitor resource usage** when running multiple profiles
6. **Use groups** to organize profiles by campaign/client
7. **Regular backup** of profile data through API export
8. **Test fingerprints** using services like pixelscan.net before deployment
