---
name: salesforce-marketing-cloud-mcp
description: MCP server enabling AI agents to orchestrate Salesforce Marketing Cloud campaigns, manage subscribers, query data extensions, and trigger journeys through natural language.
triggers:
  - "connect to Salesforce Marketing Cloud"
  - "query marketing cloud data extension"
  - "trigger a journey in SFMC"
  - "manage subscribers in Marketing Cloud"
  - "set up MCP for Salesforce Marketing"
  - "integrate AI with Marketing Cloud API"
  - "automate SFMC campaigns with AI"
  - "retrieve customer data from data extensions"
---

# Salesforce Marketing Cloud MCP Integration

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The Salesforce Marketing Cloud MCP server is a Model Context Protocol implementation that enables AI agents to interact with Salesforce Marketing Cloud (SFMC) through natural language. It abstracts the complexity of SOAP and REST APIs, providing deterministic tools for querying data extensions, managing subscribers, and triggering Journey Builder workflows.

This server translates LLM intent into safe, rate-limited SFMC operations, handling authentication, XML envelope construction, and asynchronous data operations automatically.

## Installation

### Deploy to Vinkius Edge (Recommended)

For production use with zero-maintenance infrastructure:

```bash
npx mcpfusion deploy
```

This deploys the MCP server to Vinkius Edge with automatic scaling and secure credential management.

### Local Development

```bash
npm install
npm run dev
```

### Docker Deployment

```bash
docker pull vinkius/salesforce-marketing-mcp
docker run -e SFMC_CLIENT_ID=$SFMC_CLIENT_ID \
  -e SFMC_CLIENT_SECRET=$SFMC_CLIENT_SECRET \
  -e SFMC_SUBDOMAIN=$SFMC_SUBDOMAIN \
  -e SFMC_ACCOUNT_ID=$SFMC_ACCOUNT_ID \
  -p 3000:3000 vinkius/salesforce-marketing-mcp
```

## Configuration

### Environment Variables

The server requires the following credentials:

```bash
# OAuth2 credentials from Marketing Cloud
SFMC_CLIENT_ID=your_client_id
SFMC_CLIENT_SECRET=your_client_secret

# Your Marketing Cloud subdomain (e.g., mc123456789)
SFMC_SUBDOMAIN=your_subdomain

# Marketing Cloud account/business unit ID
SFMC_ACCOUNT_ID=your_account_id

# Optional: Custom API endpoint
SFMC_AUTH_BASE_URL=https://your-tenant.auth.marketingcloudapis.com
SFMC_REST_BASE_URL=https://your-tenant.rest.marketingcloudapis.com
SFMC_SOAP_BASE_URL=https://your-tenant.soap.marketingcloudapis.com
```

### Obtaining Marketing Cloud Credentials

1. Log into Salesforce Marketing Cloud
2. Navigate to **Setup** → **Apps** → **Installed Packages**
3. Create a new package or select existing
4. Add a new component with **Server-to-Server** integration
5. Grant required permissions:
   - Read/Write Data Extensions
   - Read/Write Subscribers
   - Execute Journey Builder
6. Copy the **Client ID** and **Client Secret**

### MCP Server Configuration

Add to your MCP client configuration (e.g., Claude Desktop):

```json
{
  "mcpServers": {
    "salesforce-marketing": {
      "command": "npx",
      "args": ["-y", "@vinkius/salesforce-marketing-mcp"],
      "env": {
        "SFMC_CLIENT_ID": "your_client_id",
        "SFMC_CLIENT_SECRET": "your_client_secret",
        "SFMC_SUBDOMAIN": "mc123456789",
        "SFMC_ACCOUNT_ID": "123456"
      }
    }
  }
}
```

## Core MCP Tools

### 1. query_data_extension

Query records from Marketing Cloud Data Extensions with SQL-like syntax.

**Parameters:**
- `dataExtensionKey` (string, required): External key of the data extension
- `fields` (string[], optional): Fields to retrieve (defaults to all)
- `filter` (object, optional): Filter conditions
- `limit` (number, optional): Maximum records to return

**Example Usage:**

```typescript
// Query all subscribers in a segment
{
  "dataExtensionKey": "high_value_customers",
  "fields": ["EmailAddress", "FirstName", "TotalPurchases"],
  "filter": {
    "property": "TotalPurchases",
    "operator": "greaterThan",
    "value": 1000
  },
  "limit": 100
}
```

**Natural Language Prompts:**
- "Show me all subscribers in the high value customers data extension with purchases over $1000"
- "Get email addresses from the newsletter segment"
- "Retrieve customer attributes for personalization"

### 2. trigger_journey

Inject subscribers into active Journey Builder workflows.

**Parameters:**
- `journeyKey` (string, required): API event definition key
- `contactKey` (string, required): Unique identifier for the contact
- `eventData` (object, optional): Custom data to pass to journey

**Example Usage:**

```typescript
// Trigger abandoned cart journey
{
  "journeyKey": "abandoned_cart_recovery",
  "contactKey": "customer_12345",
  "eventData": {
    "cartValue": 249.99,
    "productName": "Premium Widget",
    "abandonedAt": "2024-01-15T10:30:00Z"
  }
}
```

**Natural Language Prompts:**
- "Enroll this customer in the welcome journey"
- "Trigger the abandoned cart recovery for user 12345"
- "Start the re-engagement campaign for inactive subscribers"

### 3. manage_subscriber

View, update, or unsubscribe contacts with compliance safeguards.

**Parameters:**
- `action` (string, required): "retrieve", "update", or "unsubscribe"
- `subscriberKey` (string, required): Unique subscriber identifier
- `attributes` (object, optional): Subscriber attributes to update
- `lists` (string[], optional): List IDs to subscribe/unsubscribe

**Example Usage:**

```typescript
// Update subscriber preferences
{
  "action": "update",
  "subscriberKey": "user@example.com",
  "attributes": {
    "FirstName": "John",
    "LastName": "Doe",
    "PreferredLanguage": "en-US",
    "EmailOptIn": true
  },
  "lists": ["promotional_list_123"]
}

// Retrieve subscriber status
{
  "action": "retrieve",
  "subscriberKey": "user@example.com"
}

// Unsubscribe with compliance
{
  "action": "unsubscribe",
  "subscriberKey": "user@example.com",
  "lists": ["promotional_list_123", "newsletter_456"]
}
```

**Natural Language Prompts:**
- "Update customer preferences for user@example.com"
- "Check if this email is subscribed"
- "Unsubscribe this contact from promotional emails"

## Common Patterns

### Audience Segmentation Workflow

```typescript
// Step 1: Query segment criteria
const highValueCustomers = await queryDataExtension({
  dataExtensionKey: "customer_master",
  fields: ["ContactKey", "EmailAddress", "LTV"],
  filter: {
    property: "LTV",
    operator: "greaterThan",
    value: 5000
  }
});

// Step 2: Trigger personalized journey for each
for (const customer of highValueCustomers.records) {
  await triggerJourney({
    journeyKey: "vip_nurture",
    contactKey: customer.ContactKey,
    eventData: {
      lifetimeValue: customer.LTV,
      tier: "platinum"
    }
  });
}
```

### Event-Driven Campaign Enrollment

```typescript
// Respond to purchase event
async function onPurchaseComplete(order) {
  // Trigger post-purchase journey
  await triggerJourney({
    journeyKey: "post_purchase_thankyou",
    contactKey: order.customerId,
    eventData: {
      orderId: order.id,
      orderTotal: order.total,
      products: order.items.map(i => i.name)
    }
  });
  
  // Update customer profile
  await manageSubscriber({
    action: "update",
    subscriberKey: order.customerId,
    attributes: {
      LastPurchaseDate: new Date().toISOString(),
      TotalOrders: order.totalOrderCount,
      CustomerSegment: order.total > 500 ? "VIP" : "Standard"
    }
  });
}
```

### Compliance-Safe Preference Management

```typescript
// Handle unsubscribe request
async function handleUnsubscribeRequest(email, reason) {
  // Retrieve current status
  const subscriber = await manageSubscriber({
    action: "retrieve",
    subscriberKey: email
  });
  
  // Unsubscribe from all promotional lists
  await manageSubscriber({
    action: "unsubscribe",
    subscriberKey: email,
    lists: subscriber.lists.filter(l => l.type === "promotional")
  });
  
  // Log reason in data extension
  await queryDataExtension({
    dataExtensionKey: "unsubscribe_log",
    // Note: Use appropriate upsert endpoint
    action: "insert",
    data: {
      EmailAddress: email,
      Reason: reason,
      UnsubscribedAt: new Date().toISOString()
    }
  });
}
```

### Dynamic Content Personalization

```typescript
// Fetch subscriber data for email personalization
async function getPersonalizationData(subscriberKey) {
  const profile = await manageSubscriber({
    action: "retrieve",
    subscriberKey: subscriberKey
  });
  
  const purchaseHistory = await queryDataExtension({
    dataExtensionKey: "order_history",
    fields: ["ProductName", "PurchaseDate", "Category"],
    filter: {
      property: "ContactKey",
      operator: "equals",
      value: subscriberKey
    },
    limit: 5
  });
  
  return {
    firstName: profile.attributes.FirstName,
    recommendedProducts: getRecommendations(purchaseHistory.records),
    loyaltyTier: profile.attributes.LoyaltyTier
  };
}
```

## Troubleshooting

### Authentication Failures

**Problem:** `401 Unauthorized` or `Invalid client credentials`

**Solutions:**
- Verify `SFMC_CLIENT_ID` and `SFMC_CLIENT_SECRET` are correct
- Ensure the installed package has not expired
- Check that the subdomain matches your Marketing Cloud instance
- Confirm API integration is enabled in your Marketing Cloud account

### Data Extension Not Found

**Problem:** `Data Extension with key 'xyz' does not exist`

**Solutions:**
- Verify the external key (not the name) is correct
- Check that the data extension exists in the specified business unit
- Ensure your API user has permission to access the data extension
- Use the exact case-sensitive external key

### Rate Limiting

**Problem:** `429 Too Many Requests`

**Solutions:**
- The MCP server implements automatic rate limiting
- For high-volume operations, implement exponential backoff
- Consider batching operations where possible
- Vinkius Edge deployment handles rate limiting automatically

### Journey Not Triggering

**Problem:** Journey fires but contact doesn't enter

**Solutions:**
- Verify the journey is in "Running" status
- Check that `journeyKey` matches the API Event Definition Key (not journey name)
- Ensure `contactKey` exists in All Subscribers or the entry source
- Validate `eventData` schema matches journey's data extension
- Check journey entry criteria and filters

### SOAP vs REST Endpoint Confusion

**Problem:** Operations failing with endpoint errors

**Solutions:**
- Subscriber operations use SOAP endpoints
- Data Extensions primarily use REST endpoints
- Journey triggers use REST interaction endpoints
- Verify `SFMC_SUBDOMAIN` matches your stack (s1, s4, s7, etc.)
- Check Marketing Cloud documentation for your specific instance region

## Advanced Configuration

### Custom Retry Logic

```typescript
// Implement exponential backoff for resilience
const retryConfig = {
  maxRetries: 3,
  baseDelay: 1000,
  maxDelay: 10000
};
```

### Multi-Business Unit Support

Set `SFMC_ACCOUNT_ID` to target specific business units. For operations across multiple BUs, deploy separate MCP instances with different account IDs.

### Logging and Monitoring

Enable verbose logging for debugging:

```bash
LOG_LEVEL=debug npm run dev
```

For production monitoring on Vinkius Edge, access logs through the Vinkius dashboard.

## Security Best Practices

1. **Never hardcode credentials** — always use environment variables
2. **Rotate client secrets** regularly through Marketing Cloud setup
3. **Use least-privilege permissions** — grant only required API scopes
4. **Validate subscriber consent** before triggering communications
5. **Implement audit logging** for all subscriber modifications
6. **Use Vinkius vault** for production credential management

## Resources

- [Official Documentation](https://vinkius.com/mcp/salesforce-marketing-cloud)
- [Docker Hub](https://hub.docker.com/r/vinkius/salesforce-marketing-mcp)
- [Salesforce Marketing Cloud API Docs](https://developer.salesforce.com/docs/marketing/marketing-cloud/overview)
- [Model Context Protocol Specification](https://modelcontextprotocol.io)
