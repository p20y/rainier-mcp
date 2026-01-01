# Available Amazon Ads API Packages

This document lists all available packages that can be added to `AMAZON_AD_API_PACKAGES` in your `.env` file.

## Current Configuration

Your current `.env` file (lines 34-35) has:
```bash
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots
```

## All Available Packages

### Core Packages (Essential)

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `profiles` | Account and profile management | `ac_` |
| `exports-snapshots` | Export and snapshot operations | `export_` |

### Account Management

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `accounts-manager-accounts` | Manager account operations | `ac_` |
| `accounts-ads-accounts` | Ads account management | `ac_` |
| `accounts-portfolios` | Portfolio management | `ac_` |
| `accounts-billing` | Billing operations | `ac_` |
| `accounts-account-budgets` | Account budget management | `ac_` |

### Campaign Management

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `campaign-manage` | Campaign creation and management | `cp_` |

### Sponsored Products

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|-------------|
| `sponsored-products` | Sponsored Products operations | `sp_` |
| `sp-suggested-keywords` | Suggested keywords for SP | `sp_` |

### Sponsored Brands

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `sponsored-brands-v4` | Sponsored Brands v4 API | `sb_` |
| `sponsored-brands-v3` | Sponsored Brands v3 API | `sb_` |

### Sponsored Display

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `sponsored-display` | Sponsored Display operations | `sd_` |

### Amazon DSP (Demand-Side Platform)

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `dsp-measurement` | DSP measurement and analytics | `dsp_` |
| `dsp-advertisers` | DSP advertiser management | `dsp_` |
| `dsp-audiences` | DSP audience management | `dsp_` |
| `dsp-conversions` | DSP conversion tracking | `dsp_` |
| `dsp-target-kpi-recommendations` | DSP KPI recommendations | `dsp_` |

### Amazon Marketing Cloud (AMC)

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `amc-administration` | AMC administration | `amc_` |
| `amc-workflow` | AMC workflow operations | `amc_` |
| `amc-rule-audience` | AMC rule-based audiences | `amc_` |
| `amc-ad-audience` | AMC ad audiences | `amc_` |

### Reporting & Analytics

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `reporting-version-3` | Reporting API v3 | `reporting_` |
| `brand-metrics` | Brand metrics and insights | `brand_` |
| `brand-benchmarks` | Brand benchmark data | `brand_` |
| `stores-analytics` | Store analytics | `stores_` |
| `marketing-mix-modeling` | Marketing mix modeling | `reporting_` |

### Recommendations & Insights

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `audience-insights` | Audience insights | `recommendations_` |
| `partner-opportunities` | Partner opportunity insights | `recommendations_` |
| `tactical-recommendations` | Tactical recommendations | `recommendations_` |
| `persona-builder` | Persona builder tools | `recommendations_` |

### Creative & Assets

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `creative-assets` | Creative asset management | `creative_` |

### Products & Eligibility

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `products-metadata` | Product metadata | `products_` |
| `products-eligibility` | Product eligibility checks | `products_` |

### Moderation

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `unified-pre-moderation-results` | Pre-moderation results | `moderation_` |
| `moderation-results` | Moderation results | `moderation_` |

### Other Services

| Package Name | Description | Tool Prefix |
|-------------|-------------|-------------|
| `amazon-attribution` | Amazon Attribution | `attribution_` |
| `audiences-discovery` | Audience discovery | `audiences_` |
| `amazon-marketing-stream` | Marketing stream subscriptions | `ams_` |
| `locations` | Location management | `locations_` |
| `forecasts` | Forecasting tools | `forecasts_` |
| `change-history` | Change history tracking | `change_` |
| `data-provider-data` | Data provider operations | `data_` |
| `data-provider-hashed` | Hashed data provider | `data_` |
| `brand-store-manangement` | Brand store management | `brand_` |
| `reach-forecasting` | Reach forecasting | `forecasts_` |
| `test-account` | Test account operations | `test_` |

### Amazon Ads API v1 (Beta)

| Package Name | Description | Tool Prefix | Status |
|-------------|-------------|-------------|--------|
| `ads-api-v1-sp` | Sponsored Products v1 | `spv1_` | Beta |
| `ads-api-v1-sb` | Sponsored Brands v1 | `sbv1_` | Beta |
| `ads-api-v1-dsp` | Amazon DSP v1 | `dspv1_` | Beta |
| `ads-api-v1-sd` | Sponsored Display v1 | `sdv1_` | Beta |
| `ads-api-v1-st` | Sponsored Television v1 | `stv1_` | Beta |

> **⚠️ Beta Notice**: API v1 packages are in beta. Features may change.

## Package Groups

The packages are organized into logical groups for easier management:

### Core Group
```bash
AMAZON_AD_API_PACKAGES=profiles,exports-snapshots
```

### Sponsored Ads Group
```bash
AMAZON_AD_API_PACKAGES=sponsored-products,sponsored-brands-v4,sponsored-display
```

### AMC Group
```bash
AMAZON_AD_API_PACKAGES=amc-administration,amc-workflow,amc-rule-audience,amc-ad-audience
```

### DSP Group
```bash
AMAZON_AD_API_PACKAGES=dsp-measurement,dsp-advertisers,dsp-audiences,dsp-conversions,dsp-target-kpi-recommendations
```

### Reporting Group
```bash
AMAZON_AD_API_PACKAGES=reporting-version-3,brand-metrics,stores-analytics,exports-snapshots,marketing-mix-modeling
```

## Example Configurations

### Minimal Configuration (Current)
```bash
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots
```
**Use Case**: Basic profile and account management with exports

### Campaign Management Focus
```bash
AMAZON_AD_API_PACKAGES=profiles,campaign-manage,sponsored-products,sponsored-brands-v4,sponsored-display
```
**Use Case**: Full campaign management across all ad types

### Reporting & Analytics Focus
```bash
AMAZON_AD_API_PACKAGES=profiles,reporting-version-3,brand-metrics,stores-analytics,exports-snapshots,marketing-mix-modeling
```
**Use Case**: Comprehensive reporting and analytics

### AMC Workflow Focus
```bash
AMAZON_AD_API_PACKAGES=profiles,amc-administration,amc-workflow,amc-rule-audience,amc-ad-audience
```
**Use Case**: Amazon Marketing Cloud operations

### DSP Operations Focus
```bash
AMAZON_AD_API_PACKAGES=profiles,dsp-measurement,dsp-advertisers,dsp-audiences,dsp-conversions
```
**Use Case**: Programmatic advertising with DSP

### Complete Package Set (All Features)
```bash
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,accounts-portfolios,accounts-billing,campaign-manage,sponsored-products,sponsored-brands-v4,sponsored-display,dsp-measurement,dsp-advertisers,reporting-version-3,brand-metrics,amc-workflow,exports-snapshots
```
**Use Case**: Maximum functionality (note: may increase context usage)

## Important Notes

### Context Limits
- **More packages = More tools = More context usage**
- Each package adds multiple tools to the MCP server
- Large package sets can consume significant context in AI clients
- Start with minimal packages and add as needed

### Package Dependencies
- `profiles` is typically required for most operations
- Some packages may depend on others (check Amazon Ads API docs)
- Not all packages are available for all account types

### Performance Considerations
- Loading many packages increases server startup time
- More tools = larger tool registry = more memory usage
- Consider your actual use case when selecting packages

## How to Update Your .env

To add more packages, edit line 34-35 in your `.env` file:

```bash
# Before
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots

# After (example: add campaign management)
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots,campaign-manage,sponsored-products
```

**Remember**: After updating `.env`, restart the server for changes to take effect.

## Package Aliases

The system supports package aliases defined in `packages.json`. The aliases map to full namespace names, but you can use either the alias or the full name in your configuration.

## Finding Package Names

To see all available packages and their mappings:
1. Check `dist/openapi/resources/packages.json`
2. See the "aliases" section for package name mappings
3. Check the README.md for the full list

