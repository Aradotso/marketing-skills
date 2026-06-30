---
name: powerbi-campaign-roi-analyzer
description: Power BI dashboard for social media KPI analysis, campaign ROI tracking, and audience demographic insights with multi-channel performance metrics
triggers:
  - analyze social media campaign performance in Power BI
  - create marketing ROI dashboard with Power BI
  - track campaign KPIs across multiple channels
  - build social media analytics dashboard
  - calculate return on ad spend in Power BI
  - segment audience demographics for marketing campaigns
  - visualize product sales by social channel
  - create multi-dimensional campaign performance reports
---

# Power BI Campaign ROI Analyzer Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with the Power BI Campaign ROI Analyzer, a comprehensive social media marketing analytics dashboard that transforms raw campaign data into actionable insights. The system tracks multi-channel performance, product sales attribution, demographic segmentation, and predictive trend scoring across Facebook, Instagram, LinkedIn, TikTok, and other platforms.

## What This Project Does

The Campaign ROI Analyzer is a Power BI-based analytics platform that:

- **Multi-Dimensional Campaign Analysis**: Tracks reach, engagement, conversion, and retention metrics with weighted business value scoring
- **Product Sales Attribution**: Breaks down sales by social channel with quality metrics (AOV, repeat purchase rate, time to conversion)
- **Demographic Segmentation**: Analyzes audience clusters by psychographics, device preference, engagement patterns, and sentiment
- **Predictive Trend Scoring**: Assigns forward-looking scores to campaigns using historical patterns and seasonal variations
- **Real-Time Collaboration**: Supports team annotations, customizable alerts, and multilingual reporting
- **Unified Data Pipeline**: Normalizes data from social APIs, CRM systems, and e-commerce platforms into a single analytical model

## Installation

### Prerequisites

- Power BI Desktop (latest version recommended)
- Windows 10/11 or macOS (via browser for Power BI Service)
- Minimum 8 GB RAM (16 GB+ for large datasets)
- 500 MB storage for application files
- Active internet connection for data source APIs

### Setup Steps

1. **Clone the Repository**
```bash
git clone https://github.com/thanhtu150/powerbi-campaign-roi-analyzer.git
cd powerbi-campaign-roi-analyzer
```

2. **Open the Power BI File**
```bash
# On Windows
start "campaign-roi-analyzer.pbix"

# On macOS (Power BI Service)
open https://app.powerbi.com
# Then upload the .pbix file
```

3. **Configure Data Sources**

Power BI will prompt for data source credentials. Configure each connector:

```powerquery
// Example: Facebook/Instagram connector configuration
let
    Source = Facebook.Graph("https://graph.facebook.com/v18.0/me/insights"),
    AccessToken = Environment.GetEnvironmentVariable("FACEBOOK_ACCESS_TOKEN"),
    Headers = [#"Authorization" = "Bearer " & AccessToken],
    Data = Json.Document(Web.Contents(Source, [Headers=Headers]))
in
    Data
```

4. **Set Environment Variables**

Configure API credentials via environment variables (never hardcode):

```bash
# Windows PowerShell
$env:FACEBOOK_ACCESS_TOKEN="your_token_here"
$env:LINKEDIN_CLIENT_ID="your_client_id"
$env:LINKEDIN_CLIENT_SECRET="your_client_secret"
$env:TIKTOK_API_KEY="your_api_key"
$env:GOOGLE_ANALYTICS_PROPERTY_ID="your_property_id"
$env:SHOPIFY_API_TOKEN="your_shopify_token"

# Linux/macOS
export FACEBOOK_ACCESS_TOKEN="your_token_here"
export LINKEDIN_CLIENT_ID="your_client_id"
export LINKEDIN_CLIENT_SECRET="your_client_secret"
export TIKTOK_API_KEY="your_api_key"
export GOOGLE_ANALYTICS_PROPERTY_ID="your_property_id"
export SHOPIFY_API_TOKEN="your_shopify_token"
```

## Data Model Structure

The dashboard uses a star schema with the following key tables:

### Fact Tables
- **FactCampaigns**: Campaign performance metrics (impressions, clicks, conversions, spend)
- **FactSales**: Product sales transactions attributed to campaigns
- **FactEngagement**: Detailed engagement events (likes, comments, shares, saves)

### Dimension Tables
- **DimDate**: Time dimension with fiscal calendars and holidays
- **DimCampaign**: Campaign metadata (name, platform, type, budget)
- **DimProduct**: Product catalog with categories and pricing
- **DimAudience**: Demographic and psychographic segments
- **DimChannel**: Social media platform definitions

## Key DAX Measures

### 1. Campaign ROI Calculation

```dax
Campaign ROI = 
VAR TotalRevenue = SUM(FactSales[Revenue])
VAR TotalSpend = SUM(FactCampaigns[AdSpend])
VAR ROI = DIVIDE(TotalRevenue - TotalSpend, TotalSpend, 0)
RETURN
    ROI
```

### 2. Weighted Engagement Rate

```dax
Weighted Engagement Rate = 
VAR Likes = SUM(FactEngagement[Likes]) * 1
VAR Comments = SUM(FactEngagement[Comments]) * 2
VAR Shares = SUM(FactEngagement[Shares]) * 3
VAR Saves = SUM(FactEngagement[Saves]) * 2.5
VAR TotalImpressions = SUM(FactCampaigns[Impressions])
VAR WeightedEngagements = Likes + Comments + Shares + Saves
RETURN
    DIVIDE(WeightedEngagements, TotalImpressions, 0)
```

### 3. Cost Per Acquisition (CPA)

```dax
CPA = 
VAR TotalSpend = SUM(FactCampaigns[AdSpend])
VAR TotalConversions = COUNTROWS(FILTER(FactSales, FactSales[IsFirstPurchase] = TRUE))
RETURN
    DIVIDE(TotalSpend, TotalConversions, 0)
```

### 4. Channel Attribution Score

```dax
Channel Attribution Score = 
VAR ChannelRevenue = SUM(FactSales[Revenue])
VAR ChannelSpend = SUM(FactCampaigns[AdSpend])
VAR AvgOrderValue = AVERAGE(FactSales[OrderValue])
VAR RepeatPurchaseRate = 
    DIVIDE(
        COUNTROWS(FILTER(FactSales, FactSales[IsRepeatPurchase] = TRUE)),
        COUNTROWS(FactSales),
        0
    )
VAR Score = 
    (DIVIDE(ChannelRevenue, ChannelSpend, 0) * 0.4) + 
    (AvgOrderValue / 100 * 0.3) + 
    (RepeatPurchaseRate * 100 * 0.3)
RETURN
    Score
```

### 5. Predictive Trend Score

```dax
Trend Score = 
VAR CurrentPeriod = CALCULATE(SUM(FactEngagement[TotalEngagements]), DATESINPERIOD(DimDate[Date], MAX(DimDate[Date]), -7, DAY))
VAR PreviousPeriod = CALCULATE(SUM(FactEngagement[TotalEngagements]), DATESINPERIOD(DimDate[Date], MAX(DimDate[Date]), -14, DAY))
VAR TrendSlope = DIVIDE(CurrentPeriod - PreviousPeriod, PreviousPeriod, 0)
VAR SeasonalFactor = 
    SWITCH(
        TRUE(),
        MONTH(MAX(DimDate[Date])) IN {11, 12}, 1.2,  // Holiday season
        MONTH(MAX(DimDate[Date])) IN {1, 2}, 0.8,    // Post-holiday slump
        MONTH(MAX(DimDate[Date])) IN {7, 8}, 0.9,    // Summer slowdown
        1.0
    )
VAR AdjustedScore = TrendSlope * SeasonalFactor * 100
RETURN
    AdjustedScore
```

## Power Query Data Transformation

### Normalize Social Media Data

```powerquery
let
    // Import raw data from multiple sources
    FacebookData = Facebook.Graph(Environment.GetEnvironmentVariable("FACEBOOK_PAGE_ID")),
    LinkedInData = LinkedIn.Campaigns(Environment.GetEnvironmentVariable("LINKEDIN_ACCOUNT_ID")),
    TikTokData = Json.Document(Web.Contents("https://open-api.tiktok.com/v1.3/", [Headers=[#"Access-Token"=Environment.GetEnvironmentVariable("TIKTOK_API_KEY")]])),
    
    // Standardize Facebook data
    NormalizedFacebook = Table.TransformColumns(FacebookData, {
        {"impressions", each _, Int64.Type},
        {"reach", each _, Int64.Type},
        {"clicks", each _, Int64.Type},
        {"spend", each _, Currency.Type},
        {"date_start", each Date.From(_), type date},
        {"campaign_name", each Text.Proper(_), type text}
    }),
    AddedPlatformFacebook = Table.AddColumn(NormalizedFacebook, "Platform", each "Facebook", type text),
    
    // Standardize LinkedIn data
    NormalizedLinkedIn = Table.TransformColumns(LinkedInData, {
        {"impressions", each _, Int64.Type},
        {"clicks", each _, Int64.Type},
        {"costInLocalCurrency", each _, Currency.Type},
        {"startDate", each Date.From(_), type date},
        {"name", each Text.Proper(_), type text}
    }),
    RenamedLinkedIn = Table.RenameColumns(NormalizedLinkedIn, {
        {"costInLocalCurrency", "spend"},
        {"startDate", "date_start"},
        {"name", "campaign_name"}
    }),
    AddedPlatformLinkedIn = Table.AddColumn(RenamedLinkedIn, "Platform", each "LinkedIn", type text),
    
    // Combine all sources
    CombinedData = Table.Combine({AddedPlatformFacebook, AddedPlatformLinkedIn}),
    
    // Add derived metrics
    AddEngagementRate = Table.AddColumn(CombinedData, "EngagementRate", each 
        if [impressions] > 0 then ([clicks] / [impressions]) else 0, type number),
    AddCPC = Table.AddColumn(AddEngagementRate, "CPC", each 
        if [clicks] > 0 then ([spend] / [clicks]) else 0, Currency.Type),
    
    // Sort by date descending
    SortedData = Table.Sort(AddCPC, {{"date_start", Order.Descending}})
in
    SortedData
```

### Calculate Demographic Segments

```powerquery
let
    Source = FactEngagement,
    
    // Group by user demographics
    GroupedByDemo = Table.Group(Source, {"UserID", "AgeGroup", "Gender", "Location"}, {
        {"TotalEngagements", each List.Sum([Engagements]), type number},
        {"AvgSessionDuration", each List.Average([SessionDuration]), type number},
        {"PreferredContentType", each List.Mode([ContentType]), type text}
    }),
    
    // Calculate engagement intensity
    AddIntensityScore = Table.AddColumn(GroupedByDemo, "IntensityScore", each 
        ([TotalEngagements] * 0.6) + ([AvgSessionDuration] / 60 * 0.4), type number),
    
    // Assign psychographic cluster
    AddCluster = Table.AddColumn(AddIntensityScore, "PsychographicCluster", each 
        if [IntensityScore] >= 75 then "Power Users"
        else if [IntensityScore] >= 50 then "Active Engagers"
        else if [IntensityScore] >= 25 then "Casual Browsers"
        else "Passive Viewers", type text)
in
    AddCluster
```

## Common Dashboard Configurations

### 1. Set Custom Alert Thresholds

Navigate to the dashboard settings and configure alerts using DAX:

```dax
Engagement Alert = 
VAR CurrentRate = [Weighted Engagement Rate]
VAR Threshold = 0.02  // 2% threshold
VAR IsUnderperforming = CurrentRate < Threshold
RETURN
    IF(IsUnderperforming, 
        "⚠️ Engagement below 2%", 
        "✓ Performance normal")
```

### 2. Configure Time Granularity

```dax
// Dynamic time grouping based on date range
Time Grouping = 
VAR DateRange = DATEDIFF(MIN(DimDate[Date]), MAX(DimDate[Date]), DAY)
VAR Grouping = 
    SWITCH(
        TRUE(),
        DateRange <= 7, "Hour",
        DateRange <= 31, "Day",
        DateRange <= 90, "Week",
        "Month"
    )
RETURN
    Grouping
```

### 3. Set Metric Weighting

```dax
// Custom weighted performance score
Custom Performance Score = 
VAR ReachWeight = 0.15
VAR EngagementWeight = 0.30
VAR ConversionWeight = 0.40
VAR RetentionWeight = 0.15

VAR NormalizedReach = DIVIDE([Total Reach], MAX(FactCampaigns[Reach]), 0)
VAR NormalizedEngagement = DIVIDE([Weighted Engagement Rate], 0.10, 0)  // Assume 10% is max
VAR NormalizedConversion = DIVIDE([Conversion Rate], 0.05, 0)  // Assume 5% is max
VAR NormalizedRetention = DIVIDE([Retention Rate], 0.80, 0)  // Assume 80% is max

VAR Score = 
    (NormalizedReach * ReachWeight) +
    (NormalizedEngagement * EngagementWeight) +
    (NormalizedConversion * ConversionWeight) +
    (NormalizedRetention * RetentionWeight)

RETURN
    Score * 100  // Convert to 0-100 scale
```

## Real-World Usage Patterns

### Pattern 1: Product Launch Analysis

```dax
// Compare product performance across channels
Product Channel Performance = 
SUMMARIZE(
    FactSales,
    DimProduct[ProductName],
    DimChannel[ChannelName],
    "Total Revenue", SUM(FactSales[Revenue]),
    "Units Sold", SUM(FactSales[Quantity]),
    "Avg Order Value", AVERAGE(FactSales[OrderValue]),
    "Conversion Rate", DIVIDE(
        COUNTROWS(FactSales),
        CALCULATE(SUM(FactCampaigns[Clicks]), USERELATIONSHIP(FactCampaigns[CampaignID], FactSales[CampaignID])),
        0
    )
)
```

### Pattern 2: Budget Optimization

```dax
// Calculate optimal budget allocation based on ROI
Optimal Budget Allocation = 
VAR TotalBudget = SUM(FactCampaigns[Budget])
VAR ChannelROI = [Campaign ROI]
VAR TotalROI = CALCULATE([Campaign ROI], ALL(DimChannel))

VAR OptimalAllocation = 
    DIVIDE(ChannelROI, TotalROI, 0) * TotalBudget

RETURN
    OptimalAllocation
```

### Pattern 3: Audience Segment Targeting

```dax
// Identify high-value audience segments
High Value Segments = 
CALCULATETABLE(
    SUMMARIZE(
        DimAudience,
        DimAudience[SegmentName],
        "Avg LTV", AVERAGE(FactSales[CustomerLifetimeValue]),
        "Purchase Frequency", COUNTROWS(FactSales) / DISTINCTCOUNT(FactSales[CustomerID]),
        "Engagement Score", [Weighted Engagement Rate]
    ),
    FILTER(
        DimAudience,
        [Avg LTV] > 500 && [Engagement Score] > 0.03
    )
)
```

### Pattern 4: Campaign Fatigue Detection

```dax
// Detect campaigns showing declining performance
Campaign Fatigue Score = 
VAR DaysActive = DATEDIFF(MIN(FactCampaigns[StartDate]), MAX(DimDate[Date]), DAY)
VAR InitialEngagement = CALCULATE([Weighted Engagement Rate], DATESINPERIOD(DimDate[Date], MIN(FactCampaigns[StartDate]), 7, DAY))
VAR CurrentEngagement = CALCULATE([Weighted Engagement Rate], DATESINPERIOD(DimDate[Date], MAX(DimDate[Date]), -7, DAY))
VAR PerformanceDecline = DIVIDE(InitialEngagement - CurrentEngagement, InitialEngagement, 0)

VAR FatigueScore = 
    IF(
        DaysActive > 14 && PerformanceDecline > 0.20,
        "High Fatigue",
        IF(
            DaysActive > 7 && PerformanceDecline > 0.10,
            "Moderate Fatigue",
            "Normal"
        )
    )

RETURN
    FatigueScore
```

## Data Refresh Configuration

### Scheduled Refresh via Power BI Service

1. Publish the dashboard to Power BI Service
2. Configure data source credentials in the Power BI Service settings
3. Set refresh schedule:

```json
{
  "refreshSchedule": {
    "days": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"],
    "times": ["06:00", "12:00", "18:00"],
    "localTimeZoneId": "UTC",
    "enabled": true
  },
  "notifyOption": "MailOnFailure"
}
```

### Manual Refresh via Power BI Desktop

```powerquery
// Add refresh timestamp to track data freshness
let
    Source = #table(
        {"RefreshTimestamp", "Status"},
        {{DateTime.LocalNow(), "Success"}}
    )
in
    Source
```

## Troubleshooting

### Issue: API Rate Limit Exceeded

**Solution**: Implement exponential backoff in Power Query

```powerquery
let
    FetchWithRetry = (url as text, maxRetries as number) as any =>
        let
            TryFetch = (attempt as number) as any =>
                try
                    Json.Document(Web.Contents(url))
                otherwise
                    if attempt < maxRetries then
                        let
                            WaitTime = Number.Power(2, attempt) * 1000,
                            Wait = Function.InvokeAfter(() => TryFetch(attempt + 1), #duration(0, 0, 0, WaitTime / 1000))
                        in
                            Wait
                    else
                        error "Max retries exceeded"
        in
            TryFetch(0)
in
    FetchWithRetry
```

### Issue: Data Source Authentication Failure

**Solution**: Refresh OAuth tokens programmatically

```powerquery
let
    RefreshToken = Environment.GetEnvironmentVariable("OAUTH_REFRESH_TOKEN"),
    ClientId = Environment.GetEnvironmentVariable("OAUTH_CLIENT_ID"),
    ClientSecret = Environment.GetEnvironmentVariable("OAUTH_CLIENT_SECRET"),
    
    TokenEndpoint = "https://oauth.platform.com/token",
    
    Response = Json.Document(Web.Contents(TokenEndpoint, [
        Content = Text.ToBinary(Uri.BuildQueryString([
            grant_type = "refresh_token",
            refresh_token = RefreshToken,
            client_id = ClientId,
            client_secret = ClientSecret
        ])),
        Headers = [#"Content-Type" = "application/x-www-form-urlencoded"]
    ])),
    
    NewAccessToken = Response[access_token]
in
    NewAccessToken
```

### Issue: Metric Calculation Discrepancies

**Solution**: Validate data quality with diagnostic queries

```dax
// Data quality check
Data Quality Report = 
SUMMARIZE(
    FactCampaigns,
    DimChannel[ChannelName],
    "Total Records", COUNTROWS(FactCampaigns),
    "Missing Impressions", CALCULATE(COUNTROWS(FactCampaigns), ISBLANK(FactCampaigns[Impressions])),
    "Negative Spend", CALCULATE(COUNTROWS(FactCampaigns), FactCampaigns[AdSpend] < 0),
    "Future Dates", CALCULATE(COUNTROWS(FactCampaigns), FactCampaigns[Date] > TODAY()),
    "Duplicate CampaignIDs", COUNTROWS(FactCampaigns) - DISTINCTCOUNT(FactCampaigns[CampaignID])
)
```

### Issue: Slow Dashboard Performance

**Solution**: Optimize measures with variables and reduce cardinality

```dax
// Optimized measure using variables
Optimized ROI = 
VAR SalesTable = SUMMARIZE(FactSales, FactSales[CampaignID], "Revenue", SUM(FactSales[Revenue]))
VAR SpendTable = SUMMARIZE(FactCampaigns, FactCampaigns[CampaignID], "Spend", SUM(FactCampaigns[AdSpend]))
VAR JoinedTable = NATURALLEFTOUTERJOIN(SpendTable, SalesTable)
VAR TotalRevenue = SUMX(JoinedTable, [Revenue])
VAR TotalSpend = SUMX(JoinedTable, [Spend])
RETURN
    DIVIDE(TotalRevenue - TotalSpend, TotalSpend, 0)
```

## Export and Reporting

### Export Data to CSV

```powerquery
// Export summarized campaign data
let
    Source = FactCampaigns,
    Grouped = Table.Group(Source, {"CampaignID", "CampaignName", "Platform"}, {
        {"Total Impressions", each List.Sum([Impressions]), type number},
        {"Total Clicks", each List.Sum([Clicks]), type number},
        {"Total Spend", each List.Sum([AdSpend]), Currency.Type},
        {"ROI", each List.Sum([Revenue]) / List.Sum([AdSpend]) - 1, type number}
    }),
    Export = Csv.Document(Table.ToColumns(Grouped))
in
    Export
```

### Generate Automated Email Reports

Use Power Automate integration to send scheduled reports:

```json
{
  "trigger": {
    "type": "Recurrence",
    "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
        "weekDays": ["Monday"],
        "hours": [8]
      }
    }
  },
  "action": {
    "type": "ExportPowerBIReport",
    "inputs": {
      "workspaceId": "ENV:POWERBI_WORKSPACE_ID",
      "reportId": "ENV:POWERBI_REPORT_ID",
      "format": "PDF"
    }
  }
}
```

## Best Practices

1. **Never Hardcode Credentials**: Always use environment variables or Azure Key Vault
2. **Implement Incremental Refresh**: For large datasets, configure incremental refresh to load only new/changed data
3. **Use Aggregations**: Pre-aggregate data at appropriate granularity to improve query performance
4. **Document Measure Logic**: Add descriptions to all custom measures for team collaboration
5. **Version Control**: Store .pbix files in Git with clear commit messages for change tracking
6. **Test Data Quality**: Build automated data validation checks before visualization
7. **Optimize Relationships**: Use bidirectional filters sparingly and prefer one-way relationships
8. **Monitor Performance**: Use Performance Analyzer in Power BI Desktop to identify bottlenecks

## Additional Resources

- **Sample Data Generator**: Create synthetic campaign data for testing
- **Custom Visuals**: Install additional visuals from AppSource for advanced charts
- **DAX Studio**: Use for advanced query profiling and optimization
- **Power BI REST API**: Automate report distribution and embedding

This skill provides comprehensive guidance for working with the Campaign ROI Analyzer dashboard, from initial setup through advanced analytics patterns and troubleshooting.
