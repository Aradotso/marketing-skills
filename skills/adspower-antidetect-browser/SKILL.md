---
name: adspower-antidetect-browser
description: Multi-account antidetect browser automation with profile management for marketing campaigns and RPA workflows
triggers:
  - how do I automate AdsPower browser profiles
  - set up antidetect browser for multi-account marketing
  - manage AdsPower profiles programmatically
  - automate marketing campaigns with AdsPower
  - control multiple browser fingerprints for ads
  - integrate AdsPower API with automation scripts
  - create and launch AdsPower browser profiles
  - handle multi-account browser automation safely
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple accounts across marketing campaigns, social media automation, and RPA workflows. It provides isolated browser profiles with unique fingerprints to prevent account linking and detection.

## Core Capabilities

- **Profile Management**: Create, configure, and manage isolated browser profiles
- **Fingerprint Customization**: Control canvas, WebGL, fonts, and device parameters
- **API Automation**: Programmatically launch and control browser instances
- **Multi-Account Support**: Run multiple accounts simultaneously without cross-contamination
- **Cloud Synchronization**: Share profiles across team members
- **Proxy Integration**: Assign different proxies per profile

## Installation & Setup

### Desktop Application

1. Download AdsPower client from official website
2. Install and launch the application
3. Create an account and log in
4. Navigate to API settings to enable local API access

### API Access

AdsPower provides a local REST API (default: `http://localhost:50325`) for automation:

```bash
# Check if AdsPower is running
curl http://localhost:50325/api/v1/status
```

## API Authentication

Store your API credentials as environment variables:

```bash
export ADSPOWER_API_KEY="your_api_key_here"
export ADSPOWER_LOCAL_API="http://localhost:50325"
```

## Profile Management

### Creating a Browser Profile

```python
import requests
import os

API_BASE = os.getenv('ADSPOWER_LOCAL_API', 'http://localhost:50325')

def create_profile(name, group_id=None, proxy_config=None):
    """Create a new AdsPower browser profile"""
    url = f"{API_BASE}/api/v1/user/create"
    
    payload = {
        "name": name,
        "group_id": group_id or "0",
        "domain_name": "",
        "open_urls": [],
        "repeat_config": ["0"],
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
        "ipchecker": "ip2location",
        "sys": "Windows",
        "version": "10",
    }
    
    if proxy_config:
        payload.update(proxy_config)
    
    response = requests.post(url, json=payload)
    return response.json()

# Create a new profile
result = create_profile("Marketing Campaign 01")
profile_id = result['data']['id']
print(f"Created profile: {profile_id}")
```

### Launching a Browser Profile

```python
def launch_profile(profile_id, headless=False):
    """Launch an AdsPower browser profile"""
    url = f"{API_BASE}/api/v1/browser/start"
    
    params = {
        "user_id": profile_id,
        "open_tabs": 1,
        "headless": 1 if headless else 0,
        "disable_password_filling": 0,
        "clear_cache_after_closing": 0,
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        return {
            'selenium_address': data['data']['ws']['selenium'],
            'webdriver_path': data['data']['webdriver'],
            'debug_port': data['data']['debug_port']
        }
    else:
        raise Exception(f"Failed to launch: {data['msg']}")

# Launch and get connection details
connection = launch_profile(profile_id)
print(f"Selenium: {connection['selenium_address']}")
```

### Closing a Profile

```python
def close_profile(profile_id):
    """Close a running browser profile"""
    url = f"{API_BASE}/api/v1/browser/stop"
    params = {"user_id": profile_id}
    response = requests.get(url, params=params)
    return response.json()

close_profile(profile_id)
```

## Selenium Integration

### Connecting Selenium to AdsPower Profile

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(profile_id):
    """Connect Selenium WebDriver to AdsPower profile"""
    connection = launch_profile(profile_id)
    
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress", 
        connection['selenium_address'].replace('http://', '')
    )
    
    driver = webdriver.Chrome(
        executable_path=connection['webdriver_path'],
        options=chrome_options
    )
    
    return driver

# Use with Selenium
driver = connect_selenium(profile_id)
driver.get("https://example.com")
print(driver.title)
driver.quit()
close_profile(profile_id)
```

## Playwright Integration

```python
from playwright.sync_api import sync_playwright

def connect_playwright(profile_id):
    """Connect Playwright to AdsPower profile"""
    connection = launch_profile(profile_id)
    
    with sync_playwright() as p:
        # Connect to existing browser
        browser = p.chromium.connect_over_cdp(
            f"http://localhost:{connection['debug_port']}"
        )
        context = browser.contexts[0]
        page = context.pages[0] if context.pages else context.new_page()
        
        return browser, page

# Use with Playwright
browser, page = connect_playwright(profile_id)
page.goto("https://example.com")
print(page.title())
browser.close()
close_profile(profile_id)
```

## Proxy Configuration

### Setting Up Proxies

```python
def create_profile_with_proxy(name, proxy_type, proxy_host, proxy_port, 
                               proxy_user=None, proxy_pass=None):
    """Create profile with proxy configuration"""
    proxy_config = {
        "proxy_type": proxy_type,  # http, https, socks5
        "proxy_host": proxy_host,
        "proxy_port": str(proxy_port),
        "proxy_user": proxy_user or "",
        "proxy_password": proxy_pass or "",
        "proxy_soft": "other",
    }
    
    return create_profile(name, proxy_config=proxy_config)

# Create profile with SOCKS5 proxy
profile = create_profile_with_proxy(
    "Campaign Profile",
    proxy_type="socks5",
    proxy_host="proxy.example.com",
    proxy_port=1080,
    proxy_user=os.getenv('PROXY_USER'),
    proxy_pass=os.getenv('PROXY_PASS')
)
```

## Batch Operations

### Managing Multiple Profiles

```python
def get_all_profiles(group_id=None, page=1, page_size=100):
    """Retrieve all browser profiles"""
    url = f"{API_BASE}/api/v1/user/list"
    params = {
        "page": page,
        "page_size": page_size,
    }
    if group_id:
        params["group_id"] = group_id
    
    response = requests.get(url, params=params)
    return response.json()['data']['list']

def batch_launch_profiles(profile_ids):
    """Launch multiple profiles concurrently"""
    connections = {}
    for pid in profile_ids:
        try:
            conn = launch_profile(pid)
            connections[pid] = conn
        except Exception as e:
            print(f"Failed to launch {pid}: {e}")
    return connections

# Get all profiles and launch first 3
profiles = get_all_profiles()
profile_ids = [p['user_id'] for p in profiles[:3]]
active_connections = batch_launch_profiles(profile_ids)
```

## Profile Configuration

### Customizing Fingerprints

```python
def update_profile_fingerprint(profile_id, config):
    """Update profile fingerprint settings"""
    url = f"{API_BASE}/api/v1/user/update"
    
    payload = {
        "user_id": profile_id,
        **config
    }
    
    response = requests.post(url, json=payload)
    return response.json()

# Update fingerprint parameters
update_profile_fingerprint(profile_id, {
    "sys": "MacOS",
    "version": "13",
    "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...",
    "screen": "1920x1080",
    "webrtc": "1",  # Enable WebRTC
    "location": "1",  # Enable geolocation
    "language": ["en-US", "en"],
    "timezone": "America/New_York",
})
```

## Common Patterns

### Campaign Automation Workflow

```python
import time

def run_marketing_campaign(profile_id, urls_to_visit):
    """Automate marketing campaign across multiple URLs"""
    driver = connect_selenium(profile_id)
    
    try:
        for url in urls_to_visit:
            driver.get(url)
            time.sleep(3)  # Wait for page load
            
            # Perform actions (click ads, fill forms, etc.)
            # Example: Click first button
            try:
                button = driver.find_element("css selector", "button")
                button.click()
                time.sleep(2)
            except:
                pass
            
        return {"status": "success", "urls_visited": len(urls_to_visit)}
    
    finally:
        driver.quit()
        close_profile(profile_id)

# Run campaign
result = run_marketing_campaign(profile_id, [
    "https://example1.com",
    "https://example2.com",
    "https://example3.com"
])
```

### Profile Rotation Strategy

```python
from itertools import cycle

class ProfileRotator:
    def __init__(self, profile_ids):
        self.profiles = cycle(profile_ids)
        self.current = None
    
    def get_next_profile(self):
        """Get next profile in rotation"""
        if self.current:
            close_profile(self.current)
        self.current = next(self.profiles)
        return self.current
    
    def execute_with_rotation(self, task_func, tasks):
        """Execute tasks with profile rotation"""
        results = []
        for task in tasks:
            profile_id = self.get_next_profile()
            result = task_func(profile_id, task)
            results.append(result)
        return results

# Use rotator
rotator = ProfileRotator(["profile_1", "profile_2", "profile_3"])
results = rotator.execute_with_rotation(run_marketing_campaign, task_list)
```

## Troubleshooting

### Profile Won't Launch

- Verify AdsPower application is running
- Check API endpoint: `curl http://localhost:50325/api/v1/status`
- Ensure profile exists and isn't already running
- Check available system resources (RAM, CPU)

### Selenium Connection Fails

- Verify ChromeDriver version matches Chrome version in profile
- Check `webdriver_path` from launch response is accessible
- Ensure no firewall blocking local ports
- Try increasing connection timeout

### Proxy Issues

- Test proxy connectivity independently before assigning to profile
- Verify proxy credentials are correct
- Check proxy type matches (HTTP/HTTPS/SOCKS5)
- Use IP checker in AdsPower to verify proxy is working

### API Returns Error Code

Common error codes:
- `-1`: Profile not found
- `-2`: Profile already running
- `-3`: Invalid parameters
- `-4`: Insufficient permissions

### Memory/Performance Issues

- Limit concurrent profiles (recommended: 5-10 per machine)
- Enable "clear cache after closing" for profiles
- Close profiles properly after use
- Monitor system resources and scale horizontally if needed

## Environment Variables Reference

```bash
ADSPOWER_LOCAL_API=http://localhost:50325
ADSPOWER_API_KEY=your_api_key
PROXY_USER=proxy_username
PROXY_PASS=proxy_password
```

## Best Practices

1. **Always close profiles** after use to free resources
2. **Use unique fingerprints** per campaign to avoid detection
3. **Rotate proxies** regularly for long-running campaigns
4. **Handle exceptions** gracefully to prevent orphaned browser instances
5. **Monitor API rate limits** if using cloud API features
6. **Test profiles individually** before batch operations
7. **Keep profiles organized** using groups for different campaigns
