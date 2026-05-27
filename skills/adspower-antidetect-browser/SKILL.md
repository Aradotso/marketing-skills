---
name: adspower-antidetect-browser
description: AdsPower antidetect browser for multi-account management, automation, and profile isolation in marketing campaigns
triggers:
  - how do I use AdsPower for multi-account management
  - set up AdsPower browser profiles for automation
  - create isolated browser profiles with AdsPower
  - automate marketing campaigns with AdsPower
  - manage multiple accounts without detection using AdsPower
  - configure AdsPower API for browser automation
  - use AdsPower with RPA tools
  - integrate AdsPower profiles into my workflow
---

# AdsPower Antidetect Browser

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

AdsPower is an antidetect browser platform that enables marketers and automation teams to manage multiple accounts across different platforms without detection. It provides isolated browser profiles with unique fingerprints, allowing teams to run marketing campaigns, manage social media accounts, and perform e-commerce operations at scale.

## Core Features

- **Profile Isolation**: Each browser profile has unique fingerprints (Canvas, WebGL, fonts, etc.)
- **Multi-Account Management**: Manage hundreds of accounts across platforms
- **API Automation**: Control profiles programmatically via REST API
- **Team Collaboration**: Cloud synchronization and team workspace features
- **Proxy Integration**: Support for HTTP/HTTPS/SOCKS5 proxies per profile
- **RPA Integration**: Compatible with Selenium, Puppeteer, and Playwright

## Installation

AdsPower requires downloading the desktop application:

1. Download from official website (Windows/macOS/Linux supported)
2. Install the application
3. Launch AdsPower and create an account
4. Obtain API credentials from Settings → API

## API Configuration

AdsPower provides a local API (default port: 50325) for automation:

```javascript
// Base API endpoint (local)
const ADSPOWER_API = 'http://local.adspower.net:50325/api/v1';

// API configuration
const config = {
  apiUrl: process.env.ADSPOWER_API_URL || 'http://local.adspower.net:50325/api/v1',
  apiKey: process.env.ADSPOWER_API_KEY, // Optional: for API authentication
};
```

## Key API Endpoints

### 1. List Browser Profiles

```javascript
async function listProfiles(groupId = null, page = 1, pageSize = 100) {
  const params = new URLSearchParams({
    page_size: pageSize,
    page: page
  });
  
  if (groupId) {
    params.append('group_id', groupId);
  }
  
  const response = await fetch(`${ADSPOWER_API}/user/list?${params}`);
  const data = await response.json();
  
  if (data.code === 0) {
    return data.data.list;
  }
  throw new Error(`Failed to list profiles: ${data.msg}`);
}
```

### 2. Create New Profile

```javascript
async function createProfile(profileConfig) {
  const payload = {
    name: profileConfig.name,
    group_id: profileConfig.groupId || '0',
    domain_name: profileConfig.domain || '',
    open_urls: profileConfig.startUrls || [],
    repeat_config: profileConfig.repeatConfig || [0],
    username: profileConfig.username || '',
    password: profileConfig.password || '',
    fakey: profileConfig.twoFactorKey || '',
    cookie: profileConfig.cookies || [],
    ...profileConfig.fingerprint
  };
  
  // Add proxy configuration if provided
  if (profileConfig.proxy) {
    payload.user_proxy_config = {
      proxy_soft: profileConfig.proxy.type || 'other',
      proxy_type: profileConfig.proxy.protocol || 'http',
      proxy_host: profileConfig.proxy.host,
      proxy_port: profileConfig.proxy.port,
      proxy_user: profileConfig.proxy.username || '',
      proxy_password: profileConfig.proxy.password || ''
    };
  }
  
  const response = await fetch(`${ADSPOWER_API}/user/create`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
  
  const data = await response.json();
  
  if (data.code === 0) {
    return data.data.id;
  }
  throw new Error(`Failed to create profile: ${data.msg}`);
}
```

### 3. Start Browser Profile

```javascript
async function startProfile(profileId, options = {}) {
  const params = new URLSearchParams({
    user_id: profileId,
    ip_tab: options.checkIp ? '1' : '0',
    new_first_tab: options.newTab ? '1' : '0',
    launch_args: options.launchArgs || '',
    headless: options.headless ? '1' : '0'
  });
  
  const response = await fetch(`${ADSPOWER_API}/browser/start?${params}`);
  const data = await response.json();
  
  if (data.code === 0) {
    return {
      ws: data.data.ws,
      debug_port: data.data.debug_port,
      webdriver: data.data.webdriver
    };
  }
  throw new Error(`Failed to start profile: ${data.msg}`);
}
```

### 4. Stop Browser Profile

```javascript
async function stopProfile(profileId) {
  const params = new URLSearchParams({ user_id: profileId });
  
  const response = await fetch(`${ADSPOWER_API}/browser/stop?${params}`);
  const data = await response.json();
  
  if (data.code === 0) {
    return true;
  }
  throw new Error(`Failed to stop profile: ${data.msg}`);
}
```

### 5. Check Profile Status

```javascript
async function checkProfileStatus(profileId) {
  const params = new URLSearchParams({ user_id: profileId });
  
  const response = await fetch(`${ADSPOWER_API}/browser/active?${params}`);
  const data = await response.json();
  
  if (data.code === 0) {
    return data.data.status === 'Active';
  }
  return false;
}
```

## Automation with Selenium

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import requests

ADSPOWER_API = 'http://local.adspower.net:50325/api/v1'

def start_browser_with_selenium(profile_id):
    # Start AdsPower profile
    response = requests.get(
        f'{ADSPOWER_API}/browser/start',
        params={'user_id': profile_id}
    )
    data = response.json()
    
    if data['code'] != 0:
        raise Exception(f"Failed to start profile: {data['msg']}")
    
    # Get Chrome debugger address
    chrome_driver_path = data['data']['webdriver']
    debug_address = f"127.0.0.1:{data['data']['debug_port']}"
    
    # Configure Selenium
    chrome_options = Options()
    chrome_options.add_experimental_option("debuggerAddress", debug_address)
    
    # Create driver instance
    driver = webdriver.Chrome(
        executable_path=chrome_driver_path,
        options=chrome_options
    )
    
    return driver

# Usage
driver = start_browser_with_selenium('profile_id_here')
driver.get('https://example.com')
# Perform automation tasks
driver.quit()
```

## Automation with Puppeteer

```javascript
const puppeteer = require('puppeteer-core');
const fetch = require('node-fetch');

async function connectToBrowser(profileId) {
  // Start AdsPower profile
  const startResponse = await fetch(
    `${ADSPOWER_API}/browser/start?user_id=${profileId}`
  );
  const startData = await startResponse.json();
  
  if (startData.code !== 0) {
    throw new Error(`Failed to start profile: ${startData.msg}`);
  }
  
  // Connect Puppeteer to the browser
  const browser = await puppeteer.connect({
    browserWSEndpoint: startData.data.ws.puppeteer,
    defaultViewport: null
  });
  
  return browser;
}

// Usage example
(async () => {
  const browser = await connectToBrowser('profile_id_here');
  const pages = await browser.pages();
  const page = pages[0] || await browser.newPage();
  
  await page.goto('https://example.com');
  
  // Perform automation tasks
  const title = await page.title();
  console.log(`Page title: ${title}`);
  
  await browser.disconnect();
  
  // Stop profile
  await fetch(`${ADSPOWER_API}/browser/stop?user_id=profile_id_here`);
})();
```

## Common Patterns

### Profile Management Workflow

```javascript
class AdsPowerManager {
  constructor(apiUrl = 'http://local.adspower.net:50325/api/v1') {
    this.apiUrl = apiUrl;
  }
  
  async createCampaignProfiles(campaignName, count, proxyList) {
    const profileIds = [];
    
    for (let i = 0; i < count; i++) {
      const proxy = proxyList[i % proxyList.length];
      
      const profileId = await this.createProfile({
        name: `${campaignName}_${i + 1}`,
        proxy: {
          type: 'other',
          protocol: 'http',
          host: proxy.host,
          port: proxy.port,
          username: proxy.username,
          password: proxy.password
        }
      });
      
      profileIds.push(profileId);
    }
    
    return profileIds;
  }
  
  async runParallelTasks(profileIds, taskFunction) {
    const results = await Promise.all(
      profileIds.map(async (profileId) => {
        try {
          // Start profile
          const browserInfo = await this.startProfile(profileId);
          
          // Execute task
          const result = await taskFunction(profileId, browserInfo);
          
          // Stop profile
          await this.stopProfile(profileId);
          
          return { profileId, success: true, result };
        } catch (error) {
          return { profileId, success: false, error: error.message };
        }
      })
    );
    
    return results;
  }
}
```

### Cookie Import/Export

```javascript
async function importCookies(profileId, cookies) {
  const response = await fetch(`${ADSPOWER_API}/user/update`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      user_id: profileId,
      cookie: cookies // Array of cookie objects
    })
  });
  
  const data = await response.json();
  return data.code === 0;
}

async function exportCookies(profileId) {
  const params = new URLSearchParams({ user_id: profileId });
  
  const response = await fetch(`${ADSPOWER_API}/user/cookie?${params}`);
  const data = await response.json();
  
  if (data.code === 0) {
    return data.data.cookie;
  }
  throw new Error(`Failed to export cookies: ${data.msg}`);
}
```

### Proxy Rotation

```javascript
async function rotateProxy(profileId, newProxy) {
  const response = await fetch(`${ADSPOWER_API}/user/update`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      user_id: profileId,
      user_proxy_config: {
        proxy_soft: 'other',
        proxy_type: newProxy.protocol,
        proxy_host: newProxy.host,
        proxy_port: newProxy.port,
        proxy_user: newProxy.username,
        proxy_password: newProxy.password
      }
    })
  });
  
  const data = await response.json();
  return data.code === 0;
}
```

## Troubleshooting

### Profile Won't Start

**Issue**: Profile fails to start with error code
**Solution**: 
- Check if AdsPower application is running
- Verify profile exists: `GET /api/v1/user/list`
- Ensure no other profile is using the same port
- Check system resources (RAM, CPU)

### WebSocket Connection Failed

**Issue**: Cannot connect Puppeteer/Playwright to browser
**Solution**:
- Wait 2-3 seconds after starting profile before connecting
- Verify WebSocket endpoint from start response
- Check firewall/antivirus isn't blocking connections

### Proxy Not Working

**Issue**: Profile shows different IP than proxy
**Solution**:
- Verify proxy credentials are correct
- Test proxy connectivity outside AdsPower
- Ensure `ip_tab: 1` is set when starting to verify IP
- Check proxy type matches (HTTP/HTTPS/SOCKS5)

### API Returns Code -1

**Issue**: Generic API error
**Solution**:
- Check API endpoint URL is correct
- Verify AdsPower version supports the endpoint
- Review full error message in `msg` field
- Ensure required parameters are provided

### Profile Fingerprint Detection

**Issue**: Accounts still getting flagged
**Solution**:
- Use unique proxy per profile
- Vary browser fingerprints (canvas, fonts, WebGL)
- Add realistic user behavior delays
- Don't reuse profiles across different platforms
- Clear cookies between sessions if needed

## Environment Variables

```bash
# .env file example
ADSPOWER_API_URL=http://local.adspower.net:50325/api/v1
ADSPOWER_API_KEY=your_api_key_here
PROXY_HOST=proxy.example.com
PROXY_PORT=8080
PROXY_USERNAME=user
PROXY_PASSWORD=pass
```

## Best Practices

1. **Profile Organization**: Group profiles by campaign/platform
2. **Proxy Management**: Use dedicated proxies per profile
3. **Resource Management**: Limit concurrent profiles based on system specs
4. **Error Handling**: Always implement retry logic for API calls
5. **Profile Cleanup**: Stop profiles after tasks complete
6. **Data Backup**: Export cookies and profile data regularly
7. **Rate Limiting**: Add delays between actions to mimic human behavior
