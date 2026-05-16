---
name: google-meta-ads-ga4-mcp
description: MCP server for Google Ads, Meta Ads & GA4 — 250+ tools for campaign management, analytics & optimization across all platforms
triggers:
  - connect my google ads account to claude
  - set up meta ads campaign management
  - how do i use the google meta ads mcp server
  - create a google ads campaign with mcp
  - analyze ga4 data through mcp
  - manage facebook ads with ai assistant
  - configure advertising mcp server
  - run cross-platform ad performance reports
---

# Google Ads + Meta Ads + GA4 MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The Google Ads + Meta Ads + GA4 MCP server is a unified Model Context Protocol server that provides 250+ tools for managing advertising campaigns and analytics across Google Ads, Meta (Facebook/Instagram) Ads, and Google Analytics 4. It enables AI assistants like Claude, ChatGPT, Cursor, and n8n to perform full read-write operations on these platforms.

**Key capabilities:**
- **150+ Google Ads tools** — Campaign CRUD, keyword research, bidding, ad creation, audiences, extensions, experiments
- **80+ Meta Ads tools** — Campaign/ad set/creative management, audience targeting, lead gen, product catalogs
- **20+ GA4 tools** — Reporting, audience analysis, property config, attribution, key events
- **Cross-platform analytics** — Unified ROAS, performance comparison, budget allocation

This is a **remote MCP server** (hosted), not a local installation. You connect via URL endpoint.

## Installation & Setup

### Claude Desktop

1. Locate your Claude config file:
   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

2. Add the server configuration:

```json
{
  "mcpServers": {
    "google-meta-ads-ga4": {
      "url": "YOUR_MCP_ENDPOINT_URL"
    }
  }
}
```

3. Restart Claude Desktop
4. Look for the MCP connection indicator (🔌) in the bottom-right corner

### ChatGPT

1. Navigate to **Settings** → **Connectors** → **Add custom connector**
2. Enter:
   - **Name**: `Google Meta Ads GA4`
   - **URL**: `YOUR_MCP_ENDPOINT_URL`
3. Click **Save**
4. The connector will appear in your available tools

### Cursor

1. Open `~/.cursor/mcp.json`
2. Add:

```json
{
  "mcpServers": {
    "google-meta-ads-ga4": {
      "url": "YOUR_MCP_ENDPOINT_URL"
    }
  }
}
```

3. Restart Cursor

### Windsurf

1. Open `~/.codeium/windsurf/mcp_config.json`
2. Add the same configuration as Cursor
3. Restart Windsurf

### Claude Code (CLI)

```bash
claude mcp add google-meta-ads-ga4 --transport sse YOUR_MCP_ENDPOINT_URL
```

### n8n

1. Add an **MCP Client** node to your workflow
2. Configure:
   - **Server URL**: `YOUR_MCP_ENDPOINT_URL`
   - **Transport**: SSE (Server-Sent Events)
   - **Authentication**: Bearer token (recommended)

Import the [pre-built workflow template](https://github.com/irinabuht12-oss/google-meta-ads-ga4-mcp/blob/main/configs/n8n_workflow.json) to get started.

### Codex CLI

Add to `~/.codex/config.toml`:

```toml
[mcp.google-meta-ads-ga4]
url = "YOUR_MCP_ENDPOINT_URL"
transport = "sse"
```

## Authentication & Credentials

This server uses **OAuth 2.1 with PKCE** for secure authentication. The authentication flow is handled by the hosted MCP server — you don't manage API keys locally.

**First-time setup:**
1. When you first use a tool, you'll be prompted to authenticate
2. Follow the OAuth flow for the specific platform (Google Ads, Meta, or GA4)
3. Grant the required permissions
4. Credentials are securely stored on the server side

**Environment variables** (if self-hosting):
- `GOOGLE_ADS_DEVELOPER_TOKEN` — Google Ads API developer token
- `GOOGLE_ADS_CLIENT_ID` — OAuth client ID
- `GOOGLE_ADS_CLIENT_SECRET` — OAuth client secret
- `META_APP_ID` — Meta app ID
- `META_APP_SECRET` — Meta app secret
- `GA4_PROPERTY_ID` — GA4 property ID

## Google Ads Tools

### List All Campaigns

```javascript
// Tool: list_campaigns
// Lists all campaigns with performance metrics

const campaigns = await mcp.call_tool("list_campaigns", {
  customer_id: "1234567890",
  include_metrics: true,
  date_range: "LAST_30_DAYS"
});

// Response includes: campaign name, status, budget, impressions, clicks, cost, conversions, ROAS
```

### Create a Search Campaign

```javascript
// Tool: create_search_campaign
// Creates a new Google Search campaign

const campaign = await mcp.call_tool("create_search_campaign", {
  customer_id: "1234567890",
  campaign_name: "Q2 2026 CRM Software Campaign",
  budget_amount_micros: 50000000, // $50 daily budget (in micros)
  target_cpa_micros: 25000000, // $25 target CPA
  networks: ["SEARCH", "SEARCH_PARTNERS"],
  locations: ["US"], // Country codes
  languages: ["en"],
  start_date: "2026-04-01",
  end_date: "2026-06-30"
});

// Returns: campaign resource name and ID
```

### Add Keywords to Ad Group

```javascript
// Tool: add_keywords
// Adds keywords with match types and bids

const keywords = await mcp.call_tool("add_keywords", {
  customer_id: "1234567890",
  ad_group_id: "987654321",
  keywords: [
    {
      text: "best crm software",
      match_type: "EXACT",
      cpc_bid_micros: 5000000 // $5 max CPC
    },
    {
      text: "crm for small business",
      match_type: "PHRASE",
      cpc_bid_micros: 3000000
    },
    {
      text: "customer relationship management",
      match_type: "BROAD",
      cpc_bid_micros: 2000000
    }
  ]
});
```

### Keyword Research with Keyword Planner

```javascript
// Tool: generate_keyword_ideas
// Gets keyword suggestions with volume and CPC data

const ideas = await mcp.call_tool("generate_keyword_ideas", {
  customer_id: "1234567890",
  seed_keywords: ["project management software", "task tracking tool"],
  location_ids: ["2840"], // United States
  language_id: "1000", // English
  include_adult_keywords: false
});

// Returns: keyword text, avg monthly searches, competition, low/high CPC bid
```

### Create Responsive Search Ad

```javascript
// Tool: create_responsive_search_ad
// Creates an RSA with multiple headlines and descriptions

const ad = await mcp.call_tool("create_responsive_search_ad", {
  customer_id: "1234567890",
  ad_group_id: "987654321",
  headlines: [
    "Best CRM Software 2026",
    "Try Our CRM Free Today",
    "Top-Rated CRM Platform",
    "Manage Customers Easily",
    "CRM Built for Growth"
  ],
  descriptions: [
    "Free 14-day trial. No credit card required. Get started in minutes.",
    "Loved by 10,000+ businesses. Intuitive interface, powerful features."
  ],
  final_urls: ["https://example.com/crm"],
  path1: "crm",
  path2: "free-trial"
});
```

### Get Campaign Performance Report

```javascript
// Tool: get_campaign_performance
// Retrieves detailed campaign metrics

const performance = await mcp.call_tool("get_campaign_performance", {
  customer_id: "1234567890",
  campaign_id: "111222333",
  date_range: "LAST_7_DAYS",
  metrics: [
    "impressions",
    "clicks",
    "ctr",
    "cost_micros",
    "conversions",
    "conversion_value",
    "cost_per_conversion",
    "roas"
  ],
  segment_by: "date" // Daily breakdown
});
```

### Manage Negative Keywords

```javascript
// Tool: add_negative_keywords
// Adds negative keywords to prevent unwanted clicks

const negatives = await mcp.call_tool("add_negative_keywords", {
  customer_id: "1234567890",
  campaign_id: "111222333",
  keywords: [
    { text: "free", match_type: "BROAD" },
    { text: "cheap", match_type: "BROAD" },
    { text: "download", match_type: "PHRASE" }
  ]
});
```

### Create Ad Extensions

```javascript
// Tool: create_sitelinks
// Adds sitelink extensions to campaigns

const sitelinks = await mcp.call_tool("create_sitelinks", {
  customer_id: "1234567890",
  campaign_id: "111222333",
  sitelinks: [
    {
      link_text: "Pricing",
      final_urls: ["https://example.com/pricing"],
      description1: "Flexible plans",
      description2: "Start from $29/mo"
    },
    {
      link_text: "Features",
      final_urls: ["https://example.com/features"],
      description1: "All-in-one CRM",
      description2: "Sales, marketing, support"
    }
  ]
});
```

## Meta Ads Tools

### List All Campaigns

```javascript
// Tool: meta_list_campaigns
// Lists all Meta ad campaigns

const campaigns = await mcp.call_tool("meta_list_campaigns", {
  account_id: "act_1234567890",
  fields: ["name", "status", "objective", "daily_budget", "lifetime_budget"],
  include_insights: true,
  date_preset: "last_30d"
});
```

### Create a Campaign

```javascript
// Tool: meta_create_campaign
// Creates a new Meta Ads campaign

const campaign = await mcp.call_tool("meta_create_campaign", {
  account_id: "act_1234567890",
  name: "Q2 Lead Generation Campaign",
  objective: "OUTCOME_LEADS", // OUTCOME_AWARENESS, OUTCOME_ENGAGEMENT, OUTCOME_TRAFFIC, OUTCOME_SALES
  status: "PAUSED", // Create paused, enable later
  special_ad_categories: [] // Or ["CREDIT", "EMPLOYMENT", "HOUSING"]
});
```

### Create Ad Set with Targeting

```javascript
// Tool: meta_create_ad_set
// Creates an ad set with budget, schedule, and targeting

const adSet = await mcp.call_tool("meta_create_ad_set", {
  account_id: "act_1234567890",
  campaign_id: "23850000000000000",
  name: "US Adults 25-45 Desktop",
  optimization_goal: "LEAD_GENERATION",
  billing_event: "IMPRESSIONS",
  bid_amount: 500, // $5.00 in cents
  daily_budget: 5000, // $50.00
  start_time: "2026-04-01T00:00:00+0000",
  end_time: "2026-06-30T23:59:59+0000",
  targeting: {
    geo_locations: {
      countries: ["US"]
    },
    age_min: 25,
    age_max: 45,
    genders: [0], // 0 = all, 1 = male, 2 = female
    publisher_platforms: ["facebook", "instagram"],
    facebook_positions: ["feed", "right_hand_column"],
    device_platforms: ["desktop"]
  }
});
```

### Create Lead Form

```javascript
// Tool: meta_create_lead_form
// Creates a native lead generation form

const leadForm = await mcp.call_tool("meta_create_lead_form", {
  page_id: "1234567890",
  name: "CRM Demo Request Form",
  follow_up_action_url: "https://example.com/thank-you",
  privacy_policy_url: "https://example.com/privacy",
  questions: [
    { type: "FULL_NAME" },
    { type: "EMAIL" },
    { type: "PHONE_NUMBER" },
    {
      type: "CUSTOM",
      key: "company_size",
      label: "Company Size",
      type: "MULTIPLE_CHOICE",
      options: ["1-10", "11-50", "51-200", "200+"]
    }
  ],
  context_card: {
    title: "Get a Free Demo",
    content: "See how our CRM can help your business grow.",
    button_text: "Get Started"
  }
});
```

### Upload Image Creative

```javascript
// Tool: meta_upload_image
// Uploads an image for use in ads

const image = await mcp.call_tool("meta_upload_image", {
  account_id: "act_1234567890",
  file_url: "https://example.com/ad-image.jpg",
  // OR
  // file_path: "/path/to/local/image.jpg"
});

// Returns: image hash to use in ad creatives
```

### Create Ad Creative

```javascript
// Tool: meta_create_ad_creative
// Creates an ad creative with image/video

const creative = await mcp.call_tool("meta_create_ad_creative", {
  account_id: "act_1234567890",
  name: "CRM Demo Ad Creative",
  object_story_spec: {
    page_id: "1234567890",
    link_data: {
      image_hash: "a1b2c3d4e5f6", // From meta_upload_image
      link: "https://example.com/demo",
      message: "Manage your customers better with our all-in-one CRM. Start your free trial today!",
      name: "Try Our CRM Free",
      description: "No credit card required. Get started in minutes.",
      call_to_action: {
        type: "SIGN_UP",
        value: {
          link: "https://example.com/signup"
        }
      }
    }
  }
});
```

### Create Custom Audience

```javascript
// Tool: meta_create_custom_audience
// Creates a custom audience for targeting

const audience = await mcp.call_tool("meta_create_custom_audience", {
  account_id: "act_1234567890",
  name: "Website Visitors - Last 30 Days",
  subtype: "WEBSITE", // WEBSITE, CUSTOMER_LIST, APP_USERS, ENGAGEMENT
  retention_days: 30,
  pixel_id: "1234567890123456",
  rule: {
    inclusions: {
      operator: "or",
      rules: [
        {
          event_sources: [{
            type: "pixel",
            id: "1234567890123456"
          }],
          retention_seconds: 2592000, // 30 days
          filter: {
            operator: "and",
            filters: [
              {
                field: "url",
                operator: "i_contains",
                value: "/product"
              }
            ]
          }
        }
      ]
    }
  }
});
```

### Create Lookalike Audience

```javascript
// Tool: meta_create_lookalike_audience
// Creates a lookalike audience from a seed audience

const lookalike = await mcp.call_tool("meta_create_lookalike_audience", {
  account_id: "act_1234567890",
  name: "Lookalike - Top Customers",
  origin_audience_id: "23850000000000000", // Seed custom audience
  lookalike_spec: {
    type: "similarity",
    ratio: 0.01, // Top 1% similarity
    country: "US"
  }
});
```

### Get Ad Performance Insights

```javascript
// Tool: meta_get_insights
// Retrieves performance metrics with breakdowns

const insights = await mcp.call_tool("meta_get_insights", {
  object_id: "act_1234567890", // Account, campaign, ad set, or ad ID
  level: "ad", // account, campaign, adset, or ad
  date_preset: "last_30d",
  fields: [
    "impressions",
    "clicks",
    "ctr",
    "spend",
    "cpc",
    "cpm",
    "reach",
    "frequency",
    "conversions",
    "cost_per_conversion",
    "roas"
  ],
  breakdowns: ["age", "gender"] // Segment data by demographics
});
```

## Google Analytics 4 Tools

### Run Standard Report

```javascript
// Tool: ga4_run_report
// Runs a standard GA4 report

const report = await mcp.call_tool("ga4_run_report", {
  property_id: "properties/123456789",
  date_ranges: [
    {
      start_date: "30daysAgo",
      end_date: "today"
    }
  ],
  dimensions: [
    { name: "sessionSource" },
    { name: "sessionMedium" },
    { name: "sessionCampaignName" }
  ],
  metrics: [
    { name: "sessions" },
    { name: "totalUsers" },
    { name: "conversions" },
    { name: "totalRevenue" }
  ],
  order_bys: [
    {
      metric: { metric_name: "sessions" },
      desc: true
    }
  ],
  limit: 10
});
```

### Get Realtime Data

```javascript
// Tool: ga4_run_realtime_report
// Gets current active users and events

const realtime = await mcp.call_tool("ga4_run_realtime_report", {
  property_id: "properties/123456789",
  dimensions: [
    { name: "country" },
    { name: "deviceCategory" }
  ],
  metrics: [
    { name: "activeUsers" },
    { name: "screenPageViews" }
  ],
  minute_ranges: [
    { start_minutes_ago: 5, end_minutes_ago: 0 }
  ]
});
```

### List Key Events (Conversions)

```javascript
// Tool: ga4_list_key_events
// Lists all conversion events

const keyEvents = await mcp.call_tool("ga4_list_key_events", {
  property_id: "properties/123456789"
});

// Returns: event name, counting_method, custom/default
```

### Get Available Dimensions and Metrics

```javascript
// Tool: ga4_get_metadata
// Lists all available dimensions and metrics

const metadata = await mcp.call_tool("ga4_get_metadata", {
  property_id: "properties/123456789"
});

// Returns: dimension/metric API names, UI names, descriptions, data types
```

### Run Pivot Report

```javascript
// Tool: ga4_run_pivot_report
// Creates a pivot table style report

const pivotReport = await mcp.call_tool("ga4_run_pivot_report", {
  property_id: "properties/123456789",
  date_ranges: [
    { start_date: "2026-01-01", end_date: "2026-03-31" }
  ],
  dimensions: [
    { name: "sessionSource" },
    { name: "deviceCategory" }
  ],
  metrics: [
    { name: "sessions" },
    { name: "conversions" }
  ],
  pivots: [
    {
      field_names: ["deviceCategory"],
      limit: 5,
      order_bys: [
        {
          metric: { metric_name: "sessions" },
          desc: true
        }
      ]
    }
  ]
});
```

## Cross-Platform Patterns

### Compare Google Ads vs Meta Ads Performance

```javascript
// Get Google Ads performance
const googleAds = await mcp.call_tool("get_campaign_performance", {
  customer_id: "1234567890",
  date_range: "LAST_30_DAYS",
  metrics: ["cost_micros", "conversions", "conversion_value"]
});

// Get Meta Ads performance
const metaAds = await mcp.call_tool("meta_get_insights", {
  object_id: "act_1234567890",
  level: "account",
  date_preset: "last_30d",
  fields: ["spend", "conversions", "conversion_values"]
});

// Calculate ROAS for each
const googleROAS = googleAds.conversion_value / (googleAds.cost_micros / 1000000);
const metaROAS = metaAds.conversion_values / metaAds.spend;

// Compare and allocate budget
```

### Correlate Ad Spend with GA4 Conversions

```javascript
// Get ad spend from both platforms
const googleSpend = await mcp.call_tool("get_campaign_performance", {
  customer_id: "1234567890",
  date_range: "LAST_30_DAYS",
  metrics: ["cost_micros"]
});

const metaSpend = await mcp.call_tool("meta_get_insights", {
  object_id: "act_1234567890",
  date_preset: "last_30d",
  fields: ["spend"]
});

// Get GA4 conversion data by source
const ga4Conversions = await mcp.call_tool("ga4_run_report", {
  property_id: "properties/123456789",
  date_ranges: [{ start_date: "30daysAgo", end_date: "today" }],
  dimensions: [{ name: "sessionSource" }],
  metrics: [{ name: "conversions" }, { name: "totalRevenue" }],
  dimension_filter: {
    filter: {
      field_name: "sessionSource",
      in_list_filter: {
        values: ["google", "facebook", "instagram"]
      }
    }
  }
});

// Analyze attribution discrepancies
```

### Unified Campaign Health Dashboard

```javascript
// Function to check all campaigns across platforms
async function getCampaignHealth() {
  const googleCampaigns = await mcp.call_tool("list_campaigns", {
    customer_id: "1234567890",
    include_metrics: true,
    date_range: "LAST_7_DAYS"
  });
  
  const metaCampaigns = await mcp.call_tool("meta_list_campaigns", {
    account_id: "act_1234567890",
    include_insights: true,
    date_preset: "last_7d"
  });
  
  const ga4Traffic = await mcp.call_tool("ga4_run_report", {
    property_id: "properties/123456789",
    date_ranges: [{ start_date: "7daysAgo", end_date: "today" }],
    dimensions: [{ name: "sessionSource" }],
    metrics: [{ name: "sessions" }, { name: "bounceRate" }, { name: "conversions" }]
  });
  
  // Flag campaigns with ROAS < 2.0, high bounce rate, or declining performance
  const alerts = [];
  
  googleCampaigns.forEach(campaign => {
    if (campaign.roas < 2.0) {
      alerts.push({
        platform: "Google Ads",
        campaign: campaign.name,
        issue: "Low ROAS",
        value: campaign.roas
      });
    }
  });
  
  return { googleCampaigns, metaCampaigns, ga4Traffic, alerts };
}
```

## Common Workflows

### Complete Campaign Launch (Google Ads)

1. **Create campaign** with `create_search_campaign`
2. **Create ad groups** with `create_ad_group`
3. **Add keywords** with `add_keywords`
4. **Create ads** with `create_responsive_search_ad`
5. **Add extensions** with `create_sitelinks`, `create_callouts`
6. **Set up negative keywords** with `add_negative_keywords`
7. **Enable campaign** with `resume_campaign`

### A/B Test Setup (Meta Ads)

1. **Create campaign** with `meta_create_campaign`
2. **Create two ad sets** with different targeting using `meta_create_ad_set`
3. **Upload images** with `meta_upload_image`
4. **Create creatives** with `meta_create_ad_creative` (different copy/images)
5. **Create ads** linking creatives to ad sets
6. **Monitor performance** with `meta_get_insights` and breakdown by ad set

### Audience Retargeting Flow

**Google Ads:**
1. Create remarketing list with `create_remarketing_list`
2. Add website visitors based on URL rules
3. Apply to campaign with `update_campaign_targeting`

**Meta Ads:**
1. Create custom audience with `meta_create_custom_audience` (website visitors)
2. Create lookalike with `meta_create_lookalike_audience`
3. Use in ad set targeting

### Performance Optimization Routine

```javascript
// Weekly optimization script
async function optimizeWeeklyCampaigns() {
  // 1. Identify underperforming keywords
  const keywords = await mcp.call_tool("list_keywords", {
    customer_id: "1234567890",
    include_metrics: true,
    date_range: "LAST_7_DAYS"
  });
  
  const lowQualityKeywords = keywords.filter(kw => 
    kw.quality_score < 5 || kw.ctr < 0.02
  );
  
  // 2. Pause low performers
  for (const kw of lowQualityKeywords) {
    await mcp.call_tool("pause_keyword", {
      customer_id: "1234567890",
      keyword_id: kw.id
    });
  }
  
  // 3. Find high-converting search terms
  const searchTerms = await mcp.call_tool("get_search_terms", {
    customer_id: "1234567890",
    date_range: "LAST_30_DAYS"
  });
  
  const winningTerms = searchTerms.filter(st => 
    st.conversions > 5 && st.conversion_rate > 0.05
  );
  
  // 4. Add as exact match keywords
  for (const term of winningTerms) {
    await mcp.call_tool("add_keywords", {
      customer_id: "1234567890",
      ad_group_id: term.ad_group_id,
      keywords: [
        { text: term.text, match_type: "EXACT", cpc_bid_micros: term.avg_cpc * 1.2 }
      ]
    });
  }
  
  // 5. Check budget pacing
  const campaigns = await mcp.call_tool("list_campaigns", {
    customer_id: "1234567890",
    include_metrics: true
  });
  
  for (const campaign of campaigns) {
    const pacingRatio = campaign.spend_last_7d / (campaign.daily_budget * 7);
    
    if (pacingRatio > 0.9 && campaign.roas > 3.0) {
      // Increase budget for high performers
      await mcp.call_tool("update_campaign", {
        customer_id: "1234567890",
        campaign_id: campaign.id,
        daily_budget_micros: campaign.daily_budget * 1.2 * 1000000
      });
    }
  }
}
```

## Troubleshooting

### Connection Issues

**Symptom**: MCP server not appearing in Claude/ChatGPT

**Solutions:**
1. Verify the URL is correct in your config file
2. Restart the AI assistant completely (quit and relaunch)
3. Check for JSON syntax errors in config (use a JSON validator)
4. Ensure you have network connectivity to the MCP endpoint

### Authentication Failures

**Symptom**: "Authentication required" or "Invalid token" errors

**Solutions:**
1. Re-run the OAuth flow by using any tool that requires authentication
2. Check that you've granted all required permissions during OAuth
3. Verify your account has appropriate access (MCC structure for Google Ads)
4. For Google Ads: ensure your developer token is approved (not in test mode)
5. For Meta Ads: check that your app has the required permissions (ads_management, ads_read)

### API Errors

**Symptom**: "RESOURCE_EXHAUSTED" or rate limit errors

**Solutions:**
- Google Ads has per-developer and per-account rate limits
- Meta Ads has per-app rate limits
- Implement exponential backoff in automation scripts
- Batch operations when possible (e.g., add multiple keywords at once)

**Symptom**: "PERMISSION_DENIED" errors

**Solutions:**
- Google Ads: Verify you have the correct customer ID (10 digits, no dashes)
- Meta Ads: Ensure the ad account ID includes "act_" prefix
- GA4: Use the full property ID format "properties/123456789"
- Check account access levels (standard vs. admin)

### Data Discrepancies

**Symptom**: GA4 conversions don't match Google Ads conversions

**Solutions:**
- Check attribution window settings (GA4 vs. Google Ads may differ)
- Verify conversion import settings if using GA4 goals in Google Ads
- Compare date ranges (GA4 uses property timezone, Google Ads uses account timezone)
- Check for conversion tag implementation issues

### Tool Not Found

**Symptom**: AI assistant says it can't find a specific tool

**Solutions:**
1. Use `list_tools()` to see all available tools
2. Check the exact tool name (case-sensitive, underscores not hyphens)
3. Verify your MCP server version supports that tool
4. Some tools require platform-specific authentication first

## Best Practices

### Security
- Never hardcode credentials in scripts — always use environment variables
- Use the OAuth flow provided by the MCP server, don't try to manage tokens manually
- For production automation, implement proper error handling and logging
- Rotate access tokens regularly if self-hosting

### Performance
- Batch API calls when possible (add multiple keywords, create multiple ads)
- Use date range filters to reduce data transfer
- Cache frequently accessed data (account hierarchies, metadata)
- Implement rate limiting in automation scripts

### Campaign Management
- Always create campaigns in `PAUSED` status, review before enabling
- Use labels extensively for organization and bulk operations
- Set up conversion tracking before launching campaigns
- Use experiments for A/B testing instead of manual splits

### Data Analysis
- Segment data by time period to identify trends
- Use GA4 as source of truth for cross-platform attribution
- Compare platform-reported conversions with GA4 for discrepancies
- Export large datasets to BigQuery for advanced analysis

## Additional Resources

- **Official Documentation**: [Model Context Protocol Spec](https://modelcontextprotocol.io)
- **Project Repository**: [github.com/irinabuht12-oss
