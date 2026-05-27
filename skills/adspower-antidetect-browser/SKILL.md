---
name: adspower-antidetect-browser
description: Manage AdsPower antidetect browser profiles for multi-account marketing automation and campaigns
triggers:
  - how do I create adspower browser profiles
  - manage multiple browser identities with adspower
  - automate adspower browser profiles
  - set up antidetect browser for marketing
  - use adspower api for automation
  - create fingerprint profiles in adspower
  - manage adspower profiles programmatically
  - adspower browser automation tutorial
---

# AdsPower Antidetect Browser Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that allows marketing teams to manage multiple browser profiles with unique fingerprints for multi-account operations, ad campaigns, social media management, and web automation. Each profile simulates a distinct digital identity with unique browser fingerprints, cookies, and local storage.

## Installation

AdsPower requires both the desktop application and API integration:

### Desktop Application

1. Download AdsPower from the official website
2. Install and launch the application
3. Create an account and log in
4. Enable Local API in settings (default port: 50325)

### API Access

The AdsPower Local API runs on `http://localhost:50325` by default. No additional SDK installation is required for basic automation.

For Node.js/JavaScript projects:
```bash
npm install axios
```

For Python projects:
```bash
pip install requests
```

## Configuration

### Enable Local API

1. Open AdsPower application
2. Go to Settings → Local API
3. Enable "Local API Server"
4. Note the port (default: 50325)
5. Optional: Set API authentication token

### Environment Variables

```bash
# .env
ADSPOWER_API_URL=http://localhost:50325
ADSPOWER_API_KEY=your_api_key_if_enabled
```

## Core API Endpoints

### Profile Management

#### List All Profiles

**JavaScript:**
```javascript
const axios = require('axios');

const ADSPOWER_URL = process.env.ADSPOWER_API_URL || 'http://localhost:50325';

async function listProfiles(page = 1, pageSize = 100) {
  try {
    const response = await axios.get(`${ADSPOWER_URL}/api/v1/user/list`, {
      params: {
        page_size: pageSize,
        page: page
      }
    });
    
    if (response.data.code === 0) {
      return response.data.data.list;
    } else {
      throw new Error(response.data.msg);
    }
  } catch (error) {
    console.error('Failed to list profiles:', error.message);
    throw error;
  }
}

// Usage
listProfiles().then(profiles => {
  console.log(`Found ${profiles.length} profiles`);
  profiles.forEach(p => console.log(`${p.user_id}: ${p.name}`));
});
```

**Python:**
```python
import requests
import os

ADSPOWER_URL = os.getenv('ADSPOWER_API_URL', 'http://localhost:50325')

def list_profiles(page=1, page_size=100):
    url = f"{ADSPOWER_URL}/api/v1/user/list"
    params = {
        'page_size': page_size,
        'page': page
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        return data['data']['list']
    else:
        raise Exception(data['msg'])

# Usage
profiles = list_profiles()
print(f"Found {len(profiles)} profiles")
for profile in profiles:
    print(f"{profile['user_id']}: {profile['name']}")
```

#### Create New Profile

**JavaScript:**
```javascript
async function createProfile(name, config = {}) {
  const payload = {
    name: name,
    group_id: config.group_id || '0',
    domain_name: config.domain || '',
    open_urls: config.urls || [],
    username: config.username || '',
    password: config.password || '',
    fakey: config.twoFactorKey || '',
    cookie: config.cookie || [],
    ignore_cookie_error: 1,
    ip: config.proxy_ip || '',
    country: config.proxy_country || 'US',
    region: config.proxy_region || '',
    city: config.proxy_city || '',
    remark: config.notes || '',
    ipchecker: config.ip_checker || 'ip2location',
    ...config.fingerprint
  };

  try {
    const response = await axios.post(
      `${ADSPOWER_URL}/api/v1/user/create`,
      payload
    );
    
    if (response.data.code === 0) {
      console.log(`Profile created: ${response.data.data.id}`);
      return response.data.data;
    } else {
      throw new Error(response.data.msg);
    }
  } catch (error) {
    console.error('Failed to create profile:', error.message);
    throw error;
  }
}

// Usage
const newProfile = await createProfile('Marketing Campaign 1', {
  group_id: '0',
  urls: ['https://facebook.com'],
  proxy_country: 'US',
  notes: 'Facebook ads account'
});
```

**Python:**
```python
def create_profile(name, config=None):
    if config is None:
        config = {}
    
    payload = {
        'name': name,
        'group_id': config.get('group_id', '0'),
        'domain_name': config.get('domain', ''),
        'open_urls': config.get('urls', []),
        'username': config.get('username', ''),
        'password': config.get('password', ''),
        'fakey': config.get('two_factor_key', ''),
        'cookie': config.get('cookie', []),
        'ignore_cookie_error': 1,
        'ip': config.get('proxy_ip', ''),
        'country': config.get('proxy_country', 'US'),
        'region': config.get('proxy_region', ''),
        'city': config.get('proxy_city', ''),
        'remark': config.get('notes', ''),
        'ipchecker': config.get('ip_checker', 'ip2location')
    }
    
    url = f"{ADSPOWER_URL}/api/v1/user/create"
    response = requests.post(url, json=payload)
    data = response.json()
    
    if data['code'] == 0:
        print(f"Profile created: {data['data']['id']}")
        return data['data']
    else:
        raise Exception(data['msg'])

# Usage
new_profile = create_profile('Marketing Campaign 1', {
    'urls': ['https://facebook.com'],
    'proxy_country': 'US',
    'notes': 'Facebook ads account'
})
```

#### Launch Browser Profile

**JavaScript:**
```javascript
async function startBrowser(profileId, options = {}) {
  const params = {
    user_id: profileId,
    open_tabs: options.open_tabs !== false ? 1 : 0,
    ip_tab: options.ip_tab !== false ? 1 : 0,
    new_first_tab: options.new_first_tab !== false ? 1 : 0,
    launch_args: options.launch_args || [],
    headless: options.headless ? 1 : 0,
    disable_password_filling: options.disable_password_filling ? 1 : 0,
    clear_cache_after_closing: options.clear_cache ? 1 : 0
  };

  try {
    const response = await axios.get(`${ADSPOWER_URL}/api/v1/browser/start`, {
      params: params
    });
    
    if (response.data.code === 0) {
      console.log(`Browser started for profile ${profileId}`);
      return {
        ws: response.data.data.ws,
        debug_port: response.data.data.debug_port,
        webdriver: response.data.data.webdriver
      };
    } else {
      throw new Error(response.data.msg);
    }
  } catch (error) {
    console.error('Failed to start browser:', error.message);
    throw error;
  }
}

// Usage
const browserInfo = await startBrowser('abc123', {
  headless: false,
  clear_cache: false
});
console.log('WebSocket endpoint:', browserInfo.ws);
console.log('Debug port:', browserInfo.debug_port);
```

**Python:**
```python
def start_browser(profile_id, options=None):
    if options is None:
        options = {}
    
    params = {
        'user_id': profile_id,
        'open_tabs': 0 if options.get('open_tabs') is False else 1,
        'ip_tab': 0 if options.get('ip_tab') is False else 1,
        'new_first_tab': 0 if options.get('new_first_tab') is False else 1,
        'launch_args': options.get('launch_args', []),
        'headless': 1 if options.get('headless') else 0,
        'disable_password_filling': 1 if options.get('disable_password_filling') else 0,
        'clear_cache_after_closing': 1 if options.get('clear_cache') else 0
    }
    
    url = f"{ADSPOWER_URL}/api/v1/browser/start"
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        print(f"Browser started for profile {profile_id}")
        return {
            'ws': data['data']['ws'],
            'debug_port': data['data']['debug_port'],
            'webdriver': data['data']['webdriver']
        }
    else:
        raise Exception(data['msg'])

# Usage
browser_info = start_browser('abc123', {
    'headless': False,
    'clear_cache': False
})
print(f"WebSocket endpoint: {browser_info['ws']}")
print(f"Debug port: {browser_info['debug_port']}")
```

#### Close Browser Profile

**JavaScript:**
```javascript
async function stopBrowser(profileId) {
  try {
    const response = await axios.get(`${ADSPOWER_URL}/api/v1/browser/stop`, {
      params: { user_id: profileId }
    });
    
    if (response.data.code === 0) {
      console.log(`Browser stopped for profile ${profileId}`);
      return true;
    } else {
      throw new Error(response.data.msg);
    }
  } catch (error) {
    console.error('Failed to stop browser:', error.message);
    throw error;
  }
}

// Usage
await stopBrowser('abc123');
```

**Python:**
```python
def stop_browser(profile_id):
    url = f"{ADSPOWER_URL}/api/v1/browser/stop"
    params = {'user_id': profile_id}
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['code'] == 0:
        print(f"Browser stopped for profile {profile_id}")
        return True
    else:
        raise Exception(data['msg'])

# Usage
stop_browser('abc123')
```

## Automation with Selenium/Puppeteer

### Selenium Integration (Python)

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def connect_to_adspower_profile(profile_id):
    # Start browser and get debug port
    browser_info = start_browser(profile_id)
    debug_port = browser_info['debug_port']
    
    # Configure Chrome options
    chrome_options = Options()
    chrome_options.add_experimental_option(
        'debuggerAddress', 
        f'127.0.0.1:{debug_port}'
    )
    
    # Connect to the browser
    driver = webdriver.Chrome(options=chrome_options)
    return driver

# Usage
profile_id = 'your_profile_id'
driver = connect_to_adspower_profile(profile_id)

try:
    driver.get('https://facebook.com')
    # Perform automation tasks
    print(driver.title)
finally:
    driver.quit()
    stop_browser(profile_id)
```

### Puppeteer Integration (JavaScript)

```javascript
const puppeteer = require('puppeteer-core');

async function connectToAdsPowerProfile(profileId) {
  // Start browser and get WebSocket endpoint
  const browserInfo = await startBrowser(profileId);
  
  // Connect via CDP
  const browser = await puppeteer.connect({
    browserWSEndpoint: browserInfo.ws,
    defaultViewport: null
  });
  
  return browser;
}

// Usage
async function automateProfile(profileId) {
  const browser = await connectToAdsPowerProfile(profileId);
  
  try {
    const pages = await browser.pages();
    const page = pages[0] || await browser.newPage();
    
    await page.goto('https://facebook.com');
    console.log(await page.title());
    
    // Perform automation tasks
    
  } finally {
    await browser.disconnect();
    await stopBrowser(profileId);
  }
}

automateProfile('your_profile_id');
```

## Common Patterns

### Bulk Profile Creation

```javascript
async function createBulkProfiles(count, prefix, config) {
  const profiles = [];
  
  for (let i = 1; i <= count; i++) {
    const name = `${prefix} ${i}`;
    try {
      const profile = await createProfile(name, {
        ...config,
        notes: `Auto-generated profile ${i}`
      });
      profiles.push(profile);
      console.log(`Created profile ${i}/${count}`);
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 1000));
    } catch (error) {
      console.error(`Failed to create profile ${i}:`, error.message);
    }
  }
  
  return profiles;
}

// Usage
const profiles = await createBulkProfiles(10, 'Campaign Profile', {
  group_id: '0',
  proxy_country: 'US',
  urls: ['https://facebook.com']
});
```

### Profile Group Management

```javascript
async function getGroups() {
  try {
    const response = await axios.get(`${ADSPOWER_URL}/api/v1/group/list`);
    if (response.data.code === 0) {
      return response.data.data.list;
    }
    throw new Error(response.data.msg);
  } catch (error) {
    console.error('Failed to get groups:', error.message);
    throw error;
  }
}

async function createGroup(groupName, remark = '') {
  try {
    const response = await axios.post(`${ADSPOWER_URL}/api/v1/group/create`, {
      group_name: groupName,
      remark: remark
    });
    if (response.data.code === 0) {
      return response.data.data.group_id;
    }
    throw new Error(response.data.msg);
  } catch (error) {
    console.error('Failed to create group:', error.message);
    throw error;
  }
}

// Usage
const groupId = await createGroup('Facebook Campaigns', 'Q1 2026 campaigns');
const profile = await createProfile('FB Profile 1', { group_id: groupId });
```

### Sequential Browser Automation

```javascript
async function runSequentialAutomation(profileIds, task) {
  const results = [];
  
  for (const profileId of profileIds) {
    console.log(`Processing profile: ${profileId}`);
    const browser = await connectToAdsPowerProfile(profileId);
    
    try {
      const result = await task(browser, profileId);
      results.push({ profileId, success: true, result });
    } catch (error) {
      console.error(`Task failed for ${profileId}:`, error.message);
      results.push({ profileId, success: false, error: error.message });
    } finally {
      await browser.disconnect();
      await stopBrowser(profileId);
      
      // Delay between profiles
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
  
  return results;
}

// Usage
const profileIds = ['profile1', 'profile2', 'profile3'];
const results = await runSequentialAutomation(profileIds, async (browser, profileId) => {
  const pages = await browser.pages();
  const page = pages[0] || await browser.newPage();
  
  await page.goto('https://example.com');
  const title = await page.title();
  
  return { title };
});
```

## Troubleshooting

### Browser Won't Start

**Problem:** API returns error when starting browser

**Solutions:**
- Ensure AdsPower desktop application is running
- Verify Local API is enabled in settings
- Check that the profile ID exists
- Ensure no other instance of the profile is already running

```javascript
async function safeBrowserStart(profileId, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      // Try to stop any existing instance first
      await stopBrowser(profileId).catch(() => {});
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      const browserInfo = await startBrowser(profileId);
      return browserInfo;
    } catch (error) {
      console.log(`Attempt ${i + 1} failed: ${error.message}`);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 3000));
    }
  }
}
```

### Connection Refused

**Problem:** Cannot connect to Local API

**Solutions:**
- Verify AdsPower is running
- Check API port in application settings
- Ensure no firewall is blocking localhost connections
- Verify the API URL is correct

```javascript
async function checkApiConnection() {
  try {
    const response = await axios.get(`${ADSPOWER_URL}/api/v1/user/list`, {
      timeout: 5000
    });
    console.log('API connection successful');
    return true;
  } catch (error) {
    console.error('API connection failed:', error.message);
    console.log('Make sure AdsPower application is running and Local API is enabled');
    return false;
  }
}
```

### Profile Fingerprint Issues

**Problem:** Websites detect antidetect browser

**Solutions:**
- Update AdsPower to latest version
- Use higher quality proxies
- Adjust fingerprint settings
- Enable WebRTC protection
- Clear cookies and cache regularly

### Memory/Performance Issues

**Problem:** System runs slowly with multiple profiles

**Solutions:**
- Limit concurrent browser instances
- Enable "clear cache after closing"
- Close unused profiles
- Increase system RAM allocation

```javascript
async function runWithConcurrencyLimit(profileIds, task, limit = 3) {
  const results = [];
  
  for (let i = 0; i < profileIds.length; i += limit) {
    const batch = profileIds.slice(i, i + limit);
    console.log(`Processing batch ${i / limit + 1}`);
    
    const batchResults = await Promise.allSettled(
      batch.map(async profileId => {
        const browser = await connectToAdsPowerProfile(profileId);
        try {
          return await task(browser, profileId);
        } finally {
          await browser.disconnect();
          await stopBrowser(profileId);
        }
      })
    );
    
    results.push(...batchResults);
    
    // Cleanup delay between batches
    if (i + limit < profileIds.length) {
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
  
  return results;
}
```
