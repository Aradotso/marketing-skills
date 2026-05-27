---
name: adspower-antidetect-browser
description: Use AdsPower antidetect browser for multi-account management, automation, and marketing campaigns with profile isolation
triggers:
  - how do i use adspower for multi-account management
  - set up adspower browser profiles for automation
  - configure adspower antidetect browser
  - manage multiple browser profiles with adspower
  - automate marketing campaigns with adspower
  - use adspower api for browser automation
  - create isolated browser profiles for marketing
  - integrate adspower with rpa workflows
---

# AdsPower Antidetect Browser Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform designed for managing multiple browser profiles with unique digital fingerprints. It enables marketing teams to run multiple accounts safely, automate campaigns, and manage social media profiles without triggering platform detection systems.

## What AdsPower Does

- **Profile Isolation**: Creates separate browser profiles with unique fingerprints (canvas, WebGL, fonts, timezone, etc.)
- **Multi-Account Management**: Manage hundreds of accounts across different platforms simultaneously
- **Automation Support**: Provides API and RPA integration for automated workflows
- **Team Collaboration**: Cloud-based profile sharing and team management
- **Proxy Management**: Built-in proxy rotation and configuration per profile

## Installation

### Desktop Application

Download and install AdsPower from the official website:

```bash
# Windows
# Download installer from official AdsPower website
# Run AdsPower-Setup.exe

# macOS
# Download .dmg installer
# Drag AdsPower to Applications folder

# Linux
wget https://version.adspower.net/software/AdsPower_Global.deb
sudo dpkg -i AdsPower_Global.deb
```

### API Access

AdsPower provides a local API server that runs on port 50325 by default when the application is running.

```bash
# Default API endpoint
http://localhost:50325/api/v1/
```

## Core Concepts

### Browser Profiles

Each profile represents an isolated browser instance with unique fingerprints:
- User agent
- Screen resolution
- WebGL fingerprint
- Canvas fingerprint
- Fonts
- Timezone and geolocation
- Audio context
- WebRTC settings

### Groups

Organize profiles into groups for team management and categorization.

## API Usage

### Authentication

AdsPower API uses a local connection without authentication when accessed from localhost. For remote access, configure API key in settings.

### Basic Profile Operations

```python
import requests
import json

# Base URL for AdsPower local API
BASE_URL = "http://localhost:50325/api/v1"

# Create a new browser profile
def create_profile(name, group_id=None, proxy_config=None):
    """
    Create a new browser profile with custom configuration
    """
    payload = {
        "name": name,
        "group_id": group_id or "0",
        "domain_name": "",
        "open_urls": [],
        "repeat_config": [0],
        "username": "",
        "password": "",
        "fakey": "",
        "cookie": "",
        "ignore_cookie_error": 0,
        "ip": "",
        "country": "us",
        "region": "",
        "city": "",
        "remark": "",
        "ipchecker": "ip2location",
        "sys": "Windows",
        "sys_version": "10"
    }
    
    # Add proxy configuration if provided
    if proxy_config:
        payload.update(proxy_config)
    
    response = requests.post(f"{BASE_URL}/user/create", json=payload)
    return response.json()

# Get profile list
def get_profiles(group_id=None, page=1, page_size=100):
    """
    Retrieve list of browser profiles
    """
    params = {
        "page": page,
        "page_size": page_size
    }
    if group_id:
        params["group_id"] = group_id
    
    response = requests.get(f"{BASE_URL}/user/list", params=params)
    return response.json()

# Start a browser profile
def start_profile(user_id, ip_tab=0, new_first_tab=True, launch_args=None):
    """
    Launch a browser profile and get automation endpoints
    """
    params = {
        "user_id": user_id,
        "ip_tab": ip_tab,
        "new_first_tab": 1 if new_first_tab else 0
    }
    
    if launch_args:
        params["launch_args"] = json.dumps(launch_args)
    
    response = requests.get(f"{BASE_URL}/browser/start", params=params)
    data = response.json()
    
    if data["code"] == 0:
        return {
            "selenium": data["data"]["ws"]["selenium"],
            "puppeteer": data["data"]["ws"]["puppeteer"],
            "debug_port": data["data"]["debug_port"]
        }
    return None

# Stop a browser profile
def stop_profile(user_id):
    """
    Close a running browser profile
    """
    params = {"user_id": user_id}
    response = requests.get(f"{BASE_URL}/browser/stop", params=params)
    return response.json()

# Delete a profile
def delete_profile(user_ids):
    """
    Delete one or more profiles
    user_ids: list of profile IDs or single ID
    """
    if isinstance(user_ids, str):
        user_ids = [user_ids]
    
    payload = {"user_ids": user_ids}
    response = requests.post(f"{BASE_URL}/user/delete", json=payload)
    return response.json()
```

### Proxy Configuration

```python
def configure_socks5_proxy(host, port, username=None, password=None):
    """
    Create SOCKS5 proxy configuration for profile
    """
    config = {
        "proxy_type": "socks5",
        "proxy_host": host,
        "proxy_port": str(port),
        "proxy_soft": "other"
    }
    
    if username and password:
        config["proxy_user"] = username
        config["proxy_password"] = password
    
    return config

def configure_http_proxy(host, port, username=None, password=None):
    """
    Create HTTP proxy configuration for profile
    """
    config = {
        "proxy_type": "http",
        "proxy_host": host,
        "proxy_port": str(port),
        "proxy_soft": "other"
    }
    
    if username and password:
        config["proxy_user"] = username
        config["proxy_password"] = password
    
    return config

# Example: Create profile with proxy
proxy = configure_socks5_proxy(
    "proxy.example.com",
    1080,
    username="user",
    password="pass"
)
profile = create_profile("Marketing Account 1", proxy_config=proxy)
```

### Selenium Integration

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_selenium(user_id):
    """
    Connect Selenium WebDriver to AdsPower profile
    """
    # Start the profile
    browser_info = start_profile(user_id)
    
    if not browser_info:
        raise Exception("Failed to start browser profile")
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress",
        browser_info["selenium"]
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(options=chrome_options)
    return driver

# Usage example
driver = connect_selenium("profile_id_here")
driver.get("https://www.example.com")
# Perform automation tasks
driver.quit()
stop_profile("profile_id_here")
```

### Puppeteer Integration

```javascript
const puppeteer = require('puppeteer-core');
const axios = require('axios');

const BASE_URL = 'http://localhost:50325/api/v1';

async function startProfile(userId) {
    const response = await axios.get(`${BASE_URL}/browser/start`, {
        params: {
            user_id: userId,
            ip_tab: 0,
            new_first_tab: 1
        }
    });
    
    return response.data;
}

async function connectPuppeteer(userId) {
    // Start the profile
    const result = await startProfile(userId);
    
    if (result.code !== 0) {
        throw new Error('Failed to start profile');
    }
    
    // Connect Puppeteer
    const browser = await puppeteer.connect({
        browserWSEndpoint: result.data.ws.puppeteer,
        defaultViewport: null
    });
    
    return browser;
}

// Usage
(async () => {
    const browser = await connectPuppeteer('profile_id_here');
    const pages = await browser.pages();
    const page = pages[0];
    
    await page.goto('https://www.example.com');
    // Perform automation tasks
    
    await browser.disconnect();
    
    // Stop the profile
    await axios.get(`${BASE_URL}/browser/stop`, {
        params: { user_id: 'profile_id_here' }
    });
})();
```

## Common Patterns

### Batch Profile Creation

```python
def create_multiple_profiles(count, prefix="Profile", group_id=None):
    """
    Create multiple profiles at once
    """
    profiles = []
    
    for i in range(count):
        name = f"{prefix} {i+1}"
        profile = create_profile(name, group_id=group_id)
        
        if profile.get("code") == 0:
            profiles.append(profile["data"])
            print(f"Created: {name} (ID: {profile['data']['id']})")
        else:
            print(f"Failed to create: {name}")
    
    return profiles

# Create 10 profiles
profiles = create_multiple_profiles(10, prefix="Marketing Account")
```

### Profile Management with Context Manager

```python
from contextlib import contextmanager

@contextmanager
def adspower_session(user_id):
    """
    Context manager for AdsPower browser sessions
    """
    browser_info = start_profile(user_id)
    
    if not browser_info:
        raise Exception(f"Failed to start profile: {user_id}")
    
    try:
        yield browser_info
    finally:
        stop_profile(user_id)

# Usage
with adspower_session("profile_id") as browser:
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress",
        browser["selenium"]
    )
    driver = webdriver.Chrome(options=chrome_options)
    driver.get("https://www.example.com")
    # Do work
    driver.quit()
```

### Rotating Profiles for Automation

```python
import time

def run_task_on_profiles(profile_ids, task_function):
    """
    Execute a task across multiple profiles sequentially
    """
    results = []
    
    for profile_id in profile_ids:
        try:
            with adspower_session(profile_id) as browser:
                result = task_function(browser, profile_id)
                results.append({
                    "profile_id": profile_id,
                    "success": True,
                    "result": result
                })
        except Exception as e:
            results.append({
                "profile_id": profile_id,
                "success": False,
                "error": str(e)
            })
        
        # Wait between profiles to avoid detection
        time.sleep(5)
    
    return results

def example_task(browser_info, profile_id):
    """
    Example task to run on each profile
    """
    chrome_options = Options()
    chrome_options.add_experimental_option(
        "debuggerAddress",
        browser_info["selenium"]
    )
    
    driver = webdriver.Chrome(options=chrome_options)
    driver.get("https://www.example.com")
    
    # Perform actions
    title = driver.title
    
    driver.quit()
    return title

# Run task on multiple profiles
profile_ids = ["id1", "id2", "id3"]
results = run_task_on_profiles(profile_ids, example_task)
```

### Cookie Management

```python
def get_profile_cookies(user_id):
    """
    Retrieve cookies from a profile
    """
    response = requests.get(
        f"{BASE_URL}/user/cookie/get",
        params={"user_id": user_id}
    )
    return response.json()

def update_profile_cookies(user_id, cookies):
    """
    Update profile cookies
    cookies: list of cookie objects or cookie string
    """
    payload = {
        "user_id": user_id,
        "cookies": cookies
    }
    
    response = requests.post(
        f"{BASE_URL}/user/cookie/update",
        json=payload
    )
    return response.json()

# Example: Transfer cookies between profiles
cookies = get_profile_cookies("source_profile_id")
if cookies["code"] == 0:
    update_profile_cookies("target_profile_id", cookies["data"])
```

## Group Management

```python
def create_group(group_name, remark=""):
    """
    Create a new profile group
    """
    payload = {
        "group_name": group_name,
        "remark": remark
    }
    response = requests.post(f"{BASE_URL}/group/create", json=payload)
    return response.json()

def get_groups():
    """
    Get all profile groups
    """
    response = requests.get(f"{BASE_URL}/group/list")
    return response.json()

def move_profiles_to_group(user_ids, group_id):
    """
    Move profiles to a specific group
    """
    payload = {
        "user_ids": user_ids if isinstance(user_ids, list) else [user_ids],
        "group_id": group_id
    }
    response = requests.post(f"{BASE_URL}/user/regroup", json=payload)
    return response.json()
```

## Configuration

### Environment Variables

```bash
# Set custom API port (if changed from default)
export ADSPOWER_API_PORT=50325

# Set custom API host for remote access
export ADSPOWER_API_HOST=localhost
```

### Application Settings

Access settings through AdsPower GUI:
- **API Settings**: Enable/disable API, change port, set API key for remote access
- **Proxy Settings**: Configure default proxy providers
- **Fingerprint Settings**: Customize fingerprint generation rules
- **Automation Settings**: Configure browser launch parameters

## Troubleshooting

### Profile Won't Start

```python
def check_profile_status(user_id):
    """
    Check if a profile is currently running
    """
    response = requests.get(
        f"{BASE_URL}/browser/active",
        params={"user_id": user_id}
    )
    data = response.json()
    
    if data["code"] == 0 and data["data"]["status"] == "Active":
        print(f"Profile {user_id} is already running")
        return True
    return False

# Force stop before starting
stop_profile(user_id)
time.sleep(2)
start_profile(user_id)
```

### Connection Issues

```python
def test_api_connection():
    """
    Test if AdsPower API is accessible
    """
    try:
        response = requests.get(f"{BASE_URL}/status", timeout=5)
        return response.status_code == 200
    except requests.exceptions.ConnectionError:
        print("Cannot connect to AdsPower API. Ensure the application is running.")
        return False
    except requests.exceptions.Timeout:
        print("API request timed out.")
        return False
```

### Port Already in Use

If Selenium/Puppeteer can't connect, ensure the debug port isn't occupied:

```python
def start_profile_with_custom_port(user_id, debug_port=9222):
    """
    Start profile with specific debug port
    """
    launch_args = [f"--remote-debugging-port={debug_port}"]
    return start_profile(user_id, launch_args=launch_args)
```

### Rate Limiting

Implement delays between API calls to avoid overwhelming the system:

```python
import time
from functools import wraps

def rate_limit(min_interval=1):
    """
    Decorator to rate limit API calls
    """
    def decorator(func):
        last_called = [0.0]
        
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

@rate_limit(min_interval=2)
def create_profile_rate_limited(*args, **kwargs):
    return create_profile(*args, **kwargs)
```

## Best Practices

1. **Always close profiles**: Use context managers or ensure `stop_profile()` is called
2. **Handle errors gracefully**: API calls can fail; implement retry logic
3. **Rotate profiles**: Don't use the same profile continuously for long periods
4. **Use groups**: Organize profiles by campaign, client, or platform
5. **Monitor proxy health**: Check proxy connectivity before automation tasks
6. **Keep fingerprints updated**: Periodically update browser profiles to match current browser versions
7. **Backup cookies**: Save session cookies to maintain logged-in states
8. **Use delays**: Add random delays between actions to appear more human-like
