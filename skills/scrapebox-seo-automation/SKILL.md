---
name: scrapebox-seo-automation
description: SEO automation toolkit for backlink analysis, link building, rank tracking, and large-scale search optimization workflows on Windows
triggers:
  - how do I use ScrapeBox for SEO automation
  - scrape search engine results with ScrapeBox
  - automate backlink analysis and link building
  - track keyword rankings with ScrapeBox
  - harvest URLs for SEO campaigns
  - configure ScrapeBox for bulk SEO tasks
  - analyze backlinks at scale with automation
  - build link lists with ScrapeBox harvester
---

# ScrapeBox SEO Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

ScrapeBox is a Windows-based SEO automation toolkit designed for large-scale search optimization workflows. It provides capabilities for URL harvesting, backlink analysis, link building, rank tracking, and bulk SEO operations for marketing teams and SEO professionals.

## Overview

ScrapeBox Ultimate SEO Automation includes:

- **URL Harvester**: Scrape search engines for keyword-based URL lists
- **Backlink Analysis**: Analyze and monitor backlink profiles at scale
- **Rank Tracking**: Monitor keyword positions across search engines
- **Link Building**: Automated link prospecting and outreach workflows
- **Bulk Operations**: Process thousands of URLs simultaneously
- **Proxy Support**: Rotate proxies to avoid rate limiting
- **Export/Import**: CSV, TXT, and database integration

## Installation

### System Requirements

- **Platform**: Windows (7, 8, 10, 11)
- **RAM**: 4GB minimum, 8GB+ recommended for large-scale operations
- **Storage**: 500MB+ free space
- **.NET Framework**: 4.5 or higher

### Setup

1. Download the installation package from the repository releases
2. Extract to your preferred directory (e.g., `C:\ScrapeBox`)
3. Run `ScrapeBox.exe` as Administrator
4. Configure initial settings through the GUI

### License Activation

```plaintext
1. Launch ScrapeBox.exe
2. Navigate to: Help > Register
3. Enter license key from environment variable: %SCRAPEBOX_LICENSE_KEY%
4. Click "Activate"
```

## Core Features

### 1. URL Harvester

The harvester scrapes search engines to build targeted URL lists based on keywords.

**Basic Harvesting Workflow:**

1. Open: **Harvester** tab
2. Enter keywords (one per line)
3. Select search engines (Google, Bing, Yahoo, etc.)
4. Set results per keyword (default: 100)
5. Click **Start Harvesting**

**Configuration Settings:**

```ini
[Harvester]
MaxResults=500
Timeout=30
UseProxies=true
ProxyFile=proxies.txt
UserAgent=Mozilla/5.0 (Windows NT 10.0; Win64; x64)
DelayBetweenRequests=2
```

### 2. Backlink Analysis

Analyze backlink profiles for your domain or competitors.

**Backlink Checker Workflow:**

1. Navigate to: **Addons > Backlink Checker**
2. Load URL list or enter domains manually
3. Select backlink sources (Moz, Ahrefs API, Majestic, etc.)
4. Configure API credentials via environment variables
5. Click **Check Backlinks**

**API Configuration Example:**

Store credentials in environment variables:
```plaintext
SCRAPEBOX_MOZ_ACCESS_ID=your_access_id
SCRAPEBOX_MOZ_SECRET_KEY=your_secret_key
SCRAPEBOX_AHREFS_API_KEY=your_api_key
```

**Export Results:**

```plaintext
File > Export > CSV
Format: Domain, Backlinks, Domain Authority, Page Authority, Trust Flow
```

### 3. Rank Tracking

Monitor keyword positions across search engines over time.

**Rank Tracker Setup:**

1. Open: **Addons > Rank Checker**
2. Import keywords: `keywords.txt` (format: keyword|domain.com)
3. Select search engines and locales
4. Set tracking frequency (daily/weekly)
5. Click **Start Tracking**

**Keyword File Format (keywords.txt):**

```text
seo automation tools|yourdomain.com
backlink analysis software|yourdomain.com
keyword rank tracker|yourdomain.com
```

**Automated Tracking Script:**

For scheduling via Windows Task Scheduler, create `rank_check.bat`:

```batch
@echo off
cd C:\ScrapeBox
ScrapeBox.exe /autorun=rank_tracker_profile.ini /export=results\ranks_%date:~-4,4%%date:~-10,2%%date:~-7,2%.csv
```

### 4. Link Prospecting

Find link building opportunities based on footprints.

**Common Link Footprints:**

```text
"keyword" + "submit guest post"
"keyword" + "write for us"
"keyword" + "become a contributor"
"keyword" + inurl:links
"keyword" + "add url"
"keyword" + "suggest a site"
```

**Prospecting Workflow:**

1. Load footprints into Harvester
2. Harvest URLs from search engines
3. Filter results: **Manage > Filter > Remove Duplicates**
4. Export clean list: `File > Export > Text File`

### 5. Proxy Management

Essential for large-scale operations to avoid IP blocking.

**Proxy Configuration:**

1. Navigate to: **Settings > Proxy Settings**
2. Import proxy list: `proxies.txt`
3. Test proxies: **Test All Proxies**
4. Set rotation: **Rotate every X requests**

**Proxy File Format (proxies.txt):**

```text
123.45.67.89:8080
98.76.54.32:3128:username:password
https://proxy3.example.com:8080
socks5://45.67.89.12:1080
```

**Proxy Test Script:**

```batch
@echo off
ScrapeBox.exe /test_proxies=proxies.txt /export_working=working_proxies.txt /threads=50
```

## Common Workflows

### Complete SEO Campaign

```plaintext
1. KEYWORD RESEARCH
   - Import seed keywords
   - Use Harvester to find related terms
   - Export keyword list

2. COMPETITOR ANALYSIS
   - Scrape competitor URLs
   - Analyze backlink profiles
   - Identify link sources

3. LINK PROSPECTING
   - Build footprint queries
   - Harvest prospect URLs
   - Filter and clean lists

4. OUTREACH PREPARATION
   - Extract contact information
   - Validate email addresses (via addon)
   - Export to CRM/email tool

5. RANK TRACKING
   - Set up monitoring for target keywords
   - Schedule automated checks
   - Export reports for analysis
```

### Bulk URL Processing

**Check HTTP Status Codes:**

1. Load URL list: `File > Import > URL List`
2. Navigate to: **Manage > Check URLs**
3. Set timeout: 10 seconds
4. Threads: 50 (adjust based on server capacity)
5. Export results with status codes

**Page Scraping:**

1. Load URLs to scrape
2. **Addons > Page Scanner**
3. Define extraction rules (CSS selectors, regex patterns)
4. Extract: Emails, Phone Numbers, Social Links, Custom Fields
5. Export structured data

## Configuration Files

### Main Configuration (settings.ini)

```ini
[General]
MaxThreads=100
Timeout=30
RetryAttempts=3
SaveSessionOnExit=true

[Harvester]
SearchEngines=Google,Bing,Yahoo
ResultsPerEngine=100
RemoveDuplicates=true

[Export]
DefaultFormat=CSV
DateFormat=YYYY-MM-DD
Delimiter=,
IncludeHeaders=true

[Proxy]
UseProxyList=true
ProxyFile=proxies.txt
RotationInterval=10
TestTimeout=5

[Advanced]
CustomUserAgent=${SCRAPEBOX_USER_AGENT}
LogLevel=Info
LogFile=logs\scrapebox_%date%.log
```

### Custom User Agents (useragents.txt)

```text
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36
```

## Automation & Scripting

### Command Line Parameters

```batch
# Run harvester with profile
ScrapeBox.exe /autorun=profiles\harvest_keywords.ini

# Check URLs from file
ScrapeBox.exe /check_urls=urls.txt /export=checked_urls.csv

# Test proxies
ScrapeBox.exe /test_proxies=proxies.txt /export_working=working.txt

# Run rank checker
ScrapeBox.exe /rank_check=keywords.txt /export=ranks.csv

# Combine operations
ScrapeBox.exe /autorun=full_workflow.ini /silent /exit_on_complete
```

### Scheduled Tasks (Windows Task Scheduler)

**Daily Rank Check:**

```batch
@echo off
SET SCRAPEBOX_PATH=C:\ScrapeBox
SET OUTPUT_PATH=C:\SEO_Reports\%date:~-4,4%\%date:~-10,2%

cd %SCRAPEBOX_PATH%
ScrapeBox.exe /rank_check=keywords.txt /export=%OUTPUT_PATH%\ranks_%date:~-4,4%%date:~-10,2%%date:~-7,2%.csv /silent /exit_on_complete

# Upload to cloud storage (optional)
rclone copy %OUTPUT_PATH% remote:seo-reports/
```

## Data Export & Integration

### Export Formats

**CSV Export Structure:**

```csv
URL,Status,PageTitle,MetaDescription,H1,Backlinks,DomainAuthority
https://example.com,200,"Example Page","Description text","Main Heading",150,45
```

**Database Integration:**

Export to SQL Server/MySQL:

1. **File > Export > Database**
2. Configure connection:
   - Server: `${DB_SERVER}`
   - Database: `${DB_NAME}`
   - Username: `${DB_USER}`
   - Password: `${DB_PASSWORD}`
3. Map fields to table columns
4. Execute export

## Troubleshooting

### Common Issues

**1. Harvester Returns No Results**

- Check proxy connectivity: Test proxies first
- Verify search engine isn't blocking: Reduce thread count
- Update user agents: Import fresh useragents.txt
- Check API quotas if using paid search APIs

**2. Slow Performance**

- Reduce concurrent threads (Settings > MaxThreads)
- Increase timeout values for slow networks
- Clear cache: Tools > Clear Cache
- Disable unnecessary addons

**3. Proxy Errors**

```plaintext
Error: "Proxy connection failed"
Solution:
1. Test proxies individually
2. Check proxy authentication format
3. Verify proxy type (HTTP/HTTPS/SOCKS5)
4. Reduce proxy rotation frequency
```

**4. Export/Import Failures**

- Ensure proper file permissions (Run as Administrator)
- Check file path lengths (Windows 260 char limit)
- Verify CSV delimiter settings match import format
- Use UTF-8 encoding for international characters

**5. Rate Limiting / Captchas**

- Increase delays between requests (3-5 seconds)
- Use more proxies with better rotation
- Implement captcha solving service integration
- Reduce concurrent threads per IP

### Log Analysis

Logs location: `C:\ScrapeBox\logs\`

**Check recent errors:**

```batch
findstr /i "error" logs\scrapebox_latest.log > errors.txt
```

**Monitor performance:**

```batch
findstr /i "completed timeout" logs\scrapebox_latest.log
```

## Best Practices

1. **Always use proxies** for large-scale operations (100+ requests)
2. **Respect robots.txt** and rate limits to avoid blocking
3. **Backup configuration files** regularly (profiles, proxy lists)
4. **Test on small datasets** before running bulk operations
5. **Schedule intensive tasks** during off-peak hours
6. **Monitor success rates** and adjust thread/timeout settings
7. **Keep proxy lists updated** for consistent performance
8. **Use environment variables** for sensitive credentials
9. **Export data frequently** to avoid data loss on crashes
10. **Version control** your keyword lists and footprints

## Environment Variables Reference

```plaintext
SCRAPEBOX_LICENSE_KEY          # Product license key
SCRAPEBOX_MOZ_ACCESS_ID        # Moz API access ID
SCRAPEBOX_MOZ_SECRET_KEY       # Moz API secret key
SCRAPEBOX_AHREFS_API_KEY       # Ahrefs API key
SCRAPEBOX_USER_AGENT           # Custom user agent string
DB_SERVER                      # Database server for exports
DB_NAME                        # Database name
DB_USER                        # Database username
DB_PASSWORD                    # Database password
```

## Additional Resources

- Official documentation in Help menu
- Community plugins via Addon Manager
- Regular updates via auto-updater
- Support forums for advanced configurations

---

This skill provides comprehensive coverage of ScrapeBox SEO automation workflows for AI coding agents to assist developers in implementing large-scale SEO campaigns, backlink analysis, and rank tracking operations.
