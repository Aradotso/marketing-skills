---
name: salesforce-marketing-cloud-mcp
description: Deploy and use the Vinkius Salesforce Marketing Cloud MCP server to enable AI agents to query data extensions, trigger journeys, and manage subscribers via natural language.
triggers:
  - "integrate Salesforce Marketing Cloud with my AI agent"
  - "query Marketing Cloud data extensions from an LLM"
  - "trigger a Journey Builder workflow programmatically"
  - "manage Marketing Cloud subscribers with AI"
  - "set up MCP server for Salesforce Marketing Cloud"
  - "automate marketing campaigns with model context protocol"
  - "connect Claude to Salesforce Marketing Cloud"
  - "deploy Marketing Cloud MCP to Vinkius Edge"
---

# Salesforce Marketing Cloud MCP Server

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The **Salesforce Marketing Cloud MCP Integration** is a Model Context Protocol (MCP) server that enables AI agents (like Claude, GPT, or any LLM) to interact with Salesforce Marketing Cloud (SFMC) through natural language. It provides three core capabilities:

1. **Query Data Extensions**: Safely retrieve subscriber and customer data from SFMC Data Extensions
2. **Trigger Journeys**: Enroll contacts into Journey Builder workflows programmatically
3. **Manage Subscribers**: View, update, or unsubscribe contacts while respecting communication preferences

This server abstracts away the complexity of SFMC's SOAP/REST APIs, rate limits, and XML envelopes, providing LLMs with deterministic, safe tools for marketing automation.

## Installation

### Deploy to Vinkius Edge (Recommended)

The fastest way to deploy this MCP server is via Vinkius Edge, which provides automatic scaling, credential management, and enterprise compliance:

```bash
npx mcpfusion deploy
```

After deployment, Vinkius provides an endpoint URL that you connect to your LLM client (Claude Desktop, Cursor, etc.).

### Local Development Setup

For local development and testing:

```bash
# Clone the repository
git clone https://github.com/vinkius-labs/salesforce-marketing-mcp.git
cd salesforce-marketing-mcp

# Install dependencies
npm install

# Run in development mode
npm run dev
```

### Docker Deployment

```bash
docker pull vinkius/salesforce-marketing-mcp

docker run -d \
  -e SFMC_CLIENT_ID=$SFMC_CLIENT_ID \
  -e SFMC_CLIENT_SECRET=$SFMC_CLIENT_SECRET \
  -e SFMC_SUBDOMAIN=$SFMC_SUBDOMAIN \
  -e SFMC_ACCOUNT_ID=$SFMC_ACCOUNT_ID \
  -p 3000:3000 \
  vinkius/salesforce-marketing-mcp
```

## Configuration

### Environment Variables

The MCP server requires the following Salesforce Marketing Cloud credentials:

```bash
# Required: Your SFMC API credentials
SFMC_CLIENT_ID=your_client_id
SFMC_CLIENT_SECRET=your_client_secret

# Required: Your SFMC subdomain (e.g., mc123456789)
SFMC_SUBDOMAIN=your_subdomain

# Required: Your SFMC account MID
SFMC_ACCOUNT_ID=your_account_id

# Optional: Custom API base URL (defaults to REST endpoint)
SFMC_API_BASE_URL=https://your_subdomain.rest.marketingcloudapis.com
```

### Obtaining SFMC Credentials

1. Log into Salesforce Marketing Cloud
2. Navigate to **Setup** → **Apps** → **Installed Packages**
3. Create a new package or select an existing one
4. Add a new **API Integration** component
5. Set the required scopes:
   - **Data Extensions**: Read, Write
   - **Journeys**: Execute, Read
   - **List and Subscribers**: Read, Write
6. Copy the **Client ID** and **Client Secret**
7. Note your **Subdomain** from the Marketing Cloud URL (e.g., `mc123456789.exacttarget.com`)
8. Find your **Account MID** in **Setup** → **Account Settings**

### Claude Desktop Configuration

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "salesforce-marketing": {
      "command": "node",
      "args": ["/path/to/salesforce-marketing-mcp/build/index.js"],
      "env": {
        "SFMC_CLIENT_ID": "your_client_id",
        "SFMC_CLIENT_SECRET": "your_client_secret",
        "SFMC_SUBDOMAIN": "mc123456789",
        "SFMC_ACCOUNT_ID": "your_account_id"
      }
    }
  }
}
```

## Available MCP Tools

### 1. query_data_extension

Query subscriber data from a Salesforce Marketing Cloud Data Extension.

**Parameters:**
- `dataExtensionKey` (string, required): The external key of the Data Extension
- `fields` (array, optional): Specific fields to retrieve (defaults to all)
- `filter` (object, optional): Filter criteria `{ field, operator, value }`
- `limit` (number, optional): Maximum records to return (default: 100)

**Example Usage:**

```typescript
// In your MCP client or AI agent context
{
  "tool": "query_data_extension",
  "arguments": {
    "dataExtensionKey": "CustomerPreferences",
    "fields": ["EmailAddress", "FirstName", "PreferredCategory"],
    "filter": {
      "field": "PreferredCategory",
      "operator": "equals",
      "value": "Electronics"
    },
    "limit": 50
  }
}
```

**Natural Language Prompt:**
> "Show me the first 50 customers from the CustomerPreferences data extension who prefer Electronics"

### 2. trigger_journey

Enroll a contact into a Journey Builder workflow.

**Parameters:**
- `journeyKey` (string, required): The unique key of the Journey
- `contactKey` (string, required): Unique identifier for the contact
- `eventData` (object, optional): Additional event data to pass into the Journey

**Example Usage:**

```typescript
{
  "tool": "trigger_journey",
  "arguments": {
    "journeyKey": "welcome-series-2024",
    "contactKey": "user@example.com",
    "eventData": {
      "RegistrationDate": "2024-01-15",
      "Source": "WebsiteSignup",
      "InterestCategory": "AI Tools"
    }
  }
}
```

**Natural Language Prompt:**
> "Enroll user@example.com into the welcome-series-2024 journey with registration date January 15, 2024"

### 3. manage_subscriber

View, update, or unsubscribe a Marketing Cloud subscriber.

**Parameters:**
- `action` (string, required): One of `get`, `update`, or `unsubscribe`
- `subscriberKey` (string, required): Unique subscriber identifier (usually email)
- `attributes` (object, optional): Attributes to update (for `update` action)

**Example Usage:**

```typescript
// Get subscriber details
{
  "tool": "manage_subscriber",
  "arguments": {
    "action": "get",
    "subscriberKey": "user@example.com"
  }
}

// Update subscriber attributes
{
  "tool": "manage_subscriber",
  "arguments": {
    "action": "update",
    "subscriberKey": "user@example.com",
    "attributes": {
      "FirstName": "Jane",
      "Industry": "Technology",
      "CompanySize": "50-200"
    }
  }
}

// Unsubscribe a contact
{
  "tool": "manage_subscriber",
  "arguments": {
    "action": "unsubscribe",
    "subscriberKey": "user@example.com"
  }
}
```

**Natural Language Prompts:**
> "Get the subscriber details for user@example.com"
> "Update Jane's industry to Technology and company size to 50-200"
> "Unsubscribe user@example.com from all communications"

## Common Patterns

### Pattern 1: Audience Segmentation for Campaign

```typescript
// Agent workflow: Identify high-value customers for a targeted campaign

// Step 1: Query customers with high engagement scores
const highValueCustomers = await queryDataExtension({
  dataExtensionKey: "CustomerEngagement",
  fields: ["EmailAddress", "EngagementScore", "LastPurchaseDate"],
  filter: {
    field: "EngagementScore",
    operator: "greaterThan",
    value: "80"
  },
  limit: 500
});

// Step 2: Trigger a journey for each qualified customer
for (const customer of highValueCustomers.records) {
  await triggerJourney({
    journeyKey: "vip-exclusive-offer",
    contactKey: customer.EmailAddress,
    eventData: {
      EngagementScore: customer.EngagementScore,
      LastPurchase: customer.LastPurchaseDate
    }
  });
}
```

### Pattern 2: Event-Driven Journey Enrollment

```typescript
// When a user completes a specific action, enroll them in a nurture journey

async function enrollAfterWebinar(attendeeEmail: string, webinarTitle: string) {
  await triggerJourney({
    journeyKey: "post-webinar-nurture",
    contactKey: attendeeEmail,
    eventData: {
      WebinarTitle: webinarTitle,
      AttendedDate: new Date().toISOString(),
      FollowUpPriority: "High"
    }
  });
}
```

### Pattern 3: Preference Management

```typescript
// Update subscriber preferences based on website activity

async function updatePreferencesFromBehavior(email: string, pageViews: string[]) {
  // Determine interest categories from page views
  const interests = categorizeInterests(pageViews);
  
  await manageSubscriber({
    action: "update",
    subscriberKey: email,
    attributes: {
      InterestCategories: interests.join(","),
      LastActivityDate: new Date().toISOString(),
      PreferenceSource: "WebsiteBehavior"
    }
  });
}
```

### Pattern 4: Compliance-Safe Unsubscribe

```typescript
// Process unsubscribe requests with audit trail

async function processUnsubscribeRequest(email: string, reason?: string) {
  // Log the unsubscribe reason to a data extension
  await queryDataExtension({
    dataExtensionKey: "UnsubscribeLog",
    // This would be an upsert operation in practice
  });
  
  // Actually unsubscribe the contact
  await manageSubscriber({
    action: "unsubscribe",
    subscriberKey: email
  });
}
```

## Troubleshooting

### Authentication Failures

**Error**: `401 Unauthorized` or `Invalid client credentials`

**Solution**: Verify your SFMC credentials:
- Double-check `SFMC_CLIENT_ID` and `SFMC_CLIENT_SECRET`
- Ensure the API Integration has not expired
- Confirm the required scopes are enabled in your Installed Package

```bash
# Test credentials manually
curl -X POST https://$SFMC_SUBDOMAIN.auth.marketingcloudapis.com/v2/token \
  -H "Content-Type: application/json" \
  -d "{\"grant_type\":\"client_credentials\",\"client_id\":\"$SFMC_CLIENT_ID\",\"client_secret\":\"$SFMC_CLIENT_SECRET\"}"
```

### Data Extension Not Found

**Error**: `Data Extension with key 'XYZ' not found`

**Solution**:
- Verify the external key (not the name) of the Data Extension
- Check that the Data Extension exists in the correct Business Unit
- Ensure your API user has permission to access the Data Extension

### Rate Limiting

**Error**: `429 Too Many Requests`

**Solution**: 
- Implement exponential backoff in your agent logic
- When deploying on Vinkius Edge, rate limiting is handled automatically
- For local deployments, add retry logic:

```typescript
async function withRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Journey Not Triggering

**Error**: Contact enrolled but journey does not fire

**Solution**:
- Verify the Journey is in **Running** state (not Draft or Stopped)
- Check that the `journeyKey` matches the API Event Definition Key
- Ensure the contact meets the Journey entry criteria
- Review Journey Builder logs in SFMC for entry rejections

### TypeScript Build Errors

**Error**: Module resolution or type errors during local development

**Solution**:
```bash
# Clean build artifacts
rm -rf build/ node_modules/

# Reinstall dependencies
npm install

# Rebuild
npm run build
```

## Advanced Configuration

### Custom Timeout Settings

For long-running queries, adjust timeout values:

```typescript
// In your local deployment
export const CONFIG = {
  SFMC_TIMEOUT_MS: 30000, // 30 seconds
  MAX_RECORDS_PER_QUERY: 1000
};
```

### Multi-Business Unit Support

When working with multiple SFMC Business Units:

```bash
# Set the parent Account MID
SFMC_ACCOUNT_ID=parent_mid

# In tool calls, specify the target BU
{
  "tool": "query_data_extension",
  "arguments": {
    "dataExtensionKey": "CustomerData",
    "businessUnitId": "child_bu_mid"
  }
}
```

## Resources

- [Official Documentation](https://vinkius.com/mcp/salesforce-marketing-cloud)
- [Docker Hub](https://hub.docker.com/r/vinkius/salesforce-marketing-mcp)
- [GitHub Repository](https://github.com/vinkius-labs/salesforce-marketing-mcp)
- [Salesforce Marketing Cloud API Docs](https://developer.salesforce.com/docs/marketing/marketing-cloud/overview)
- [Model Context Protocol Specification](https://modelcontextprotocol.io)
