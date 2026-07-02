---
name: scrapebox-ultimate-seo-automation
description: ScrapeBox Ultimate SEO Automation toolkit for large-scale search optimization, backlink analysis, rank tracking, and link building workflows on Windows.
triggers:
  - how do I use scrapebox for seo automation
  - set up scrapebox for backlink analysis
  - automate link building with scrapebox
  - scrape search results for seo
  - track keyword rankings with scrapebox
  - harvest urls for link building
  - analyze backlinks using scrapebox
  - configure scrapebox seo tools
---

# ScrapeBox Ultimate SEO Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

ScrapeBox Ultimate SEO Automation is a comprehensive Windows-based desktop application for large-scale SEO workflows including keyword harvesting, backlink analysis, rank tracking, link building, and URL scraping. This skill covers integration, automation, and workflow patterns for marketing teams.

## What It Does

- **Keyword Harvesting**: Extract keywords and search suggestions at scale
- **Backlink Analysis**: Analyze competitor backlinks and link profiles
- **Rank Tracking**: Monitor keyword rankings across search engines
- **Link Building**: Automate prospecting and outreach for link acquisition
- **URL Scraping**: Harvest URLs from search engines and websites
- **Proxy Management**: Rotate proxies to avoid rate limiting
- **Content Harvesting**: Extract content from multiple sources

## Installation

ScrapeBox is a Windows desktop application. For automation and scripting:

### System Requirements
- Windows 10/11 (64-bit recommended)
- .NET Framework 4.7.2 or higher
- Minimum 4GB RAM (8GB+ recommended for large operations)

### Setup Steps
1. Download the Full Version installer from the repository
2. Run the installer with administrator privileges
3. Activate license (if using licensed version)
4. Configure proxy settings in Settings > Proxy Settings
5. Set up user agents in Settings > User Agent Manager

## Core Components

### Command Line Interface

ScrapeBox supports command-line automation through its CLI mode:

```bash
# Basic keyword harvesting
ScrapeBox.exe /scrape /keyword:"SEO tools" /engines:google,bing /results:100

# Backlink checking
ScrapeBox.exe /backlinks /url:"example.com" /output:"backlinks.txt"

# Rank tracking
ScrapeBox.exe /rankcheck /keywords:"keywords.txt" /url:"example.com" /engine:google

# Proxy testing
ScrapeBox.exe /proxytest /input:"proxies.txt" /timeout:5 /output:"working_proxies.txt"
```

### Configuration Files

ScrapeBox uses INI-style configuration files stored in the installation directory:

**settings.ini**:
```ini
[General]
Threads=10
Timeout=30
RetryAttempts=3

[Proxies]
UseProxies=1
ProxyFile=proxies.txt
ProxyType=HTTP
RotateProxies=1

[UserAgents]
RotateUserAgents=1
UserAgentFile=useragents.txt

[Output]
DefaultOutputPath=C:\ScrapeBox\Output
SaveFormat=CSV
```

## Automation Scripts

### PowerShell Integration

```powershell
# Automated keyword harvesting workflow
$keywords = @("digital marketing", "seo tools", "backlink checker")
$outputDir = "C:\ScrapeBox\Output"

foreach ($keyword in $keywords) {
    $safeKeyword = $keyword -replace ' ', '_'
    $outputFile = Join-Path $outputDir "harvest_$safeKeyword.txt"
    
    & "C:\Program Files\ScrapeBox\ScrapeBox.exe" `
        /scrape `
        /keyword:$keyword `
        /engines:google,bing,yahoo `
        /results:500 `
        /output:$outputFile `
        /useproxies:1 `
        /threads:5
    
    Start-Sleep -Seconds 60  # Rate limiting
}

# Combine results
Get-Content "$outputDir\harvest_*.txt" | Sort-Object -Unique | Out-File "$outputDir\combined_urls.txt"
```

### Batch Processing

```batch
@echo off
REM Bulk backlink analysis script
SET SCRAPEBOX="C:\Program Files\ScrapeBox\ScrapeBox.exe"
SET URLLIST=competitor_urls.txt
SET OUTPUT=backlinks_report.csv

%SCRAPEBOX% /backlinks /urllist:%URLLIST% /output:%OUTPUT% /threads:10 /useproxies:1

REM Process results
%SCRAPEBOX% /analyze /input:%OUTPUT% /metrics:da,pa,tf /output:analyzed_%OUTPUT%

echo Backlink analysis complete!
pause
```

## Common Workflows

### 1. Keyword Research Pipeline

```python
# Python wrapper for ScrapeBox automation
import subprocess
import csv
import os
from datetime import datetime

class ScrapeBoxAutomation:
    def __init__(self, scrapebox_path="C:\\Program Files\\ScrapeBox\\ScrapeBox.exe"):
        self.scrapebox_path = scrapebox_path
        
    def harvest_keywords(self, seed_keyword, engines=['google', 'bing'], results=500):
        """Harvest related keywords from search engines"""
        output_file = f"harvest_{seed_keyword.replace(' ', '_')}_{datetime.now().strftime('%Y%m%d')}.txt"
        
        cmd = [
            self.scrapebox_path,
            "/scrape",
            f"/keyword:{seed_keyword}",
            f"/engines:{','.join(engines)}",
            f"/results:{results}",
            f"/output:{output_file}",
            "/useproxies:1"
        ]
        
        subprocess.run(cmd, check=True)
        return output_file
    
    def check_backlinks(self, urls, output_csv="backlinks.csv"):
        """Analyze backlinks for list of URLs"""
        url_file = "temp_urls.txt"
        
        with open(url_file, 'w') as f:
            f.write('\n'.join(urls))
        
        cmd = [
            self.scrapebox_path,
            "/backlinks",
            f"/urllist:{url_file}",
            f"/output:{output_csv}",
            "/threads:10",
            "/useproxies:1"
        ]
        
        subprocess.run(cmd, check=True)
        os.remove(url_file)
        return output_csv
    
    def track_rankings(self, keywords_file, domain, engine='google'):
        """Track keyword rankings for a domain"""
        output_file = f"rankings_{domain}_{datetime.now().strftime('%Y%m%d')}.csv"
        
        cmd = [
            self.scrapebox_path,
            "/rankcheck",
            f"/keywords:{keywords_file}",
            f"/url:{domain}",
            f"/engine:{engine}",
            f"/output:{output_file}",
            "/useproxies:1"
        ]
        
        subprocess.run(cmd, check=True)
        return output_file

# Usage example
sb = ScrapeBoxAutomation()

# Harvest keywords
harvested = sb.harvest_keywords("digital marketing tools", results=1000)
print(f"Keywords harvested to: {harvested}")

# Check competitor backlinks
competitors = ["competitor1.com", "competitor2.com", "competitor3.com"]
backlinks = sb.check_backlinks(competitors)
print(f"Backlink analysis saved to: {backlinks}")

# Track rankings
rankings = sb.track_rankings("keywords.txt", "mywebsite.com")
print(f"Rankings tracked to: {rankings}")
```

### 2. Proxy Management

```python
# Proxy testing and validation
def test_proxies(proxy_file, output_file="working_proxies.txt", timeout=5):
    """Test and filter working proxies"""
    cmd = [
        "C:\\Program Files\\ScrapeBox\\ScrapeBox.exe",
        "/proxytest",
        f"/input:{proxy_file}",
        f"/timeout:{timeout}",
        f"/output:{output_file}",
        "/testurl:https://www.google.com"
    ]
    
    subprocess.run(cmd, check=True)
    
    # Count working proxies
    with open(output_file, 'r') as f:
        working = len(f.readlines())
    
    return working

# Load proxies from environment or file
proxy_list = os.getenv('SCRAPEBOX_PROXIES', 'proxies.txt')
working_proxies = test_proxies(proxy_list)
print(f"Working proxies: {working_proxies}")
```

### 3. Link Prospecting Workflow

```python
# Find link building opportunities
class LinkProspector:
    def __init__(self, sb_automation):
        self.sb = sb_automation
    
    def find_guest_post_opportunities(self, niche_keywords):
        """Find guest posting opportunities"""
        footprints = [
            f'"{keyword}" + "write for us"',
            f'"{keyword}" + "guest post"',
            f'"{keyword}" + "contribute article"',
            f'"{keyword}" + "submit guest post"'
        ]
        
        all_urls = []
        
        for keyword in niche_keywords:
            for footprint in footprints:
                query = footprint.replace("{keyword}", keyword)
                urls_file = self.sb.harvest_keywords(query, results=100)
                
                with open(urls_file, 'r') as f:
                    all_urls.extend(f.readlines())
        
        # Deduplicate
        unique_urls = list(set(all_urls))
        
        with open("guest_post_opportunities.txt", 'w') as f:
            f.write('\n'.join(unique_urls))
        
        return unique_urls
    
    def extract_contact_info(self, urls_file, output_csv="contacts.csv"):
        """Extract contact information from URLs"""
        cmd = [
            self.sb.scrapebox_path,
            "/harvest",
            f"/input:{urls_file}",
            "/pattern:email",
            f"/output:{output_csv}"
        ]
        
        subprocess.run(cmd, check=True)
        return output_csv

# Usage
sb = ScrapeBoxAutomation()
prospector = LinkProspector(sb)

niche = ["digital marketing", "content marketing", "seo"]
opportunities = prospector.find_guest_post_opportunities(niche)
print(f"Found {len(opportunities)} guest post opportunities")
```

## Advanced Configuration

### Custom Footprints File

**footprints.txt**:
```
intitle:"keyword" + "guest post"
inurl:write-for-us "keyword"
"keyword" + "submit article"
"keyword" + "contributor guidelines"
allintitle:"keyword" resources
"keyword" + inurl:links
```

### Scheduling with Task Scheduler

```powershell
# Create scheduled task for daily rank tracking
$action = New-ScheduledTaskAction -Execute "C:\Program Files\ScrapeBox\ScrapeBox.exe" `
    -Argument "/rankcheck /keywords:C:\ScrapeBox\keywords.txt /url:mysite.com /output:C:\ScrapeBox\Output\rankings.csv /useproxies:1"

$trigger = New-ScheduledTaskTrigger -Daily -At 3am

$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -RunOnlyIfNetworkAvailable

Register-ScheduledTask -TaskName "ScrapeBox Daily Rankings" `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -User "SYSTEM" `
    -RunLevel Highest
```

## Troubleshooting

### Common Issues

**Proxy Connection Failures**:
```python
# Implement retry logic with proxy rotation
def scrape_with_retry(keyword, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = sb.harvest_keywords(keyword)
            return result
        except subprocess.CalledProcessError:
            print(f"Attempt {attempt + 1} failed, rotating proxies...")
            test_proxies("proxies.txt", timeout=3)
            if attempt == max_retries - 1:
                raise
```

**Rate Limiting**:
```python
import time
import random

# Add delays between requests
def scrape_with_delays(keywords, min_delay=30, max_delay=60):
    results = []
    for keyword in keywords:
        result = sb.harvest_keywords(keyword)
        results.append(result)
        
        # Random delay to appear more human
        delay = random.randint(min_delay, max_delay)
        print(f"Waiting {delay} seconds...")
        time.sleep(delay)
    
    return results
```

**Memory Issues with Large Operations**:
```powershell
# Process in batches
$allKeywords = Get-Content "large_keyword_list.txt"
$batchSize = 100

for ($i = 0; $i -lt $allKeywords.Count; $i += $batchSize) {
    $batch = $allKeywords[$i..($i + $batchSize - 1)]
    $batchFile = "batch_$($i / $batchSize).txt"
    $batch | Out-File $batchFile
    
    & "C:\Program Files\ScrapeBox\ScrapeBox.exe" /scrape /keywordlist:$batchFile /output:"results_batch_$($i / $batchSize).txt"
    
    Remove-Item $batchFile
}
```

## Best Practices

1. **Always use proxies** for large-scale operations to avoid IP bans
2. **Rotate user agents** to simulate different browsers
3. **Implement delays** between requests (30-60 seconds minimum)
4. **Monitor success rates** and adjust thread counts accordingly
5. **Store credentials** in environment variables: `SCRAPEBOX_LICENSE_KEY`
6. **Backup configurations** regularly
7. **Test on small datasets** before scaling operations
8. **Respect robots.txt** and terms of service
9. **Use dedicated proxy pools** for different projects
10. **Log all operations** for troubleshooting and reporting

## Integration with Marketing Stack

```python
# Export results to Google Sheets, Airtable, or databases
import pandas as pd

def export_to_csv(scrapebox_output, formatted_csv):
    """Convert ScrapeBox output to structured CSV"""
    df = pd.read_csv(scrapebox_output, header=None, names=['URL', 'Title', 'Description'])
    df['timestamp'] = datetime.now()
    df.to_csv(formatted_csv, index=False)
    return formatted_csv

# Send results via API
def send_to_webhook(data, webhook_url=None):
    """Send scraped data to webhook endpoint"""
    import requests
    
    webhook_url = webhook_url or os.getenv('MARKETING_WEBHOOK_URL')
    response = requests.post(webhook_url, json=data)
    return response.status_code == 200
```

This skill enables AI agents to automate comprehensive SEO workflows using ScrapeBox Ultimate for Windows-based marketing operations.
