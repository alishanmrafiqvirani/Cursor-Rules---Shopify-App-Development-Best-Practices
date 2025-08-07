---
description: Shopify Storefront Performance Optimization Guidelines
globs:
  [
    "**/*.js",
    "**/*.ts",
    "**/*.jsx",
    "**/*.tsx",
    "**/*.css",
    "**/*.scss",
    "**/*.liquid",
    "**/*.html",
    "**/*.json",
    "**/*.yml",
    "**/*.yaml",
  ]
alwaysApply: false
---

# Shopify Storefront Performance Optimization Guidelines

## Performance Impact Priority

### Storefront Performance Requirements

- **OPTIMIZE** app to minimize impact on storefront performance
- **PRIORITIZE** fast storefront for higher conversion rates and repeat business
- **IMPROVE** search engine rankings through performance optimization
- **MEET** Shopify App Store requirements for low/no negative performance impact
- **WHY**: Fast storefronts directly impact merchant revenue and customer experience

## Performance Testing Methodology

### 1. Shopify's Performance Testing Standards

- **UNDERSTAND** Shopify's weighted page scoring system:
  - **Home page**: 17% weight
  - **Product details page**: 40% weight
  - **Collection page**: 43% weight
- **MEASURE** Lighthouse score before and after app installation
- **CALCULATE** weighted average performance impact
- **DEMONSTRATE** consistently low or no negative impact over time

### 2. Performance Testing Procedure

#### Step 1: Retrieve Testing URLs

- **START** with clean install of supported Shopify theme (e.g., Horizon)
- **AVOID** installing other apps during baseline testing
- **RETRIEVE** shareable URLs from Shopify admin:
  - Navigate to Online Store > Themes
  - Right-click "View your store" → Copy link address
- **FORMAT** testing URLs correctly:

```text
Home page URL:
https://your-store.myshopify.com/?key=[key]&preview_theme_id=

Product page URL:
https://your-store.myshopify.com/product/your-product-name?key=[key]&preview_theme_id=

Collection page URL:
https://your-store.myshopify.com/collection/your-collection-name?key=[key]&preview_theme_id=
```

#### Step 2: Baseline Performance Measurement

- **USE** PageSpeed Insights for performance testing
- **SELECT** Mobile tab for consistent measurements
- **RECORD** Performance scores for all three page types
- **CALCULATE** weighted average baseline score
- **RUN** multiple tests to account for Lighthouse score variation

#### Step 3: Post-Installation Performance Analysis

- **INSTALL** app with most frequently used features configured
- **VERIFY** app loads correctly on storefront
- **REPEAT** PageSpeed Insights testing on all three page types
- **CALCULATE** weighted average ending score
- **DETERMINE** performance impact: (ending score - starting score)

### 3. Performance Impact Calculation

```javascript
// Weighted average calculation
function calculateWeightedPerformance(
  homeScore,
  productScore,
  collectionScore
) {
  const weights = {
    home: 0.17,
    product: 0.4,
    collection: 0.43,
  };

  return (
    homeScore * weights.home +
    productScore * weights.product +
    collectionScore * weights.collection
  );
}

// Performance impact assessment
function assessPerformanceImpact(startingScore, endingScore) {
  const impact = endingScore - startingScore;

  if (impact >= 0) {
    return `Performance improved by ${impact} points`;
  } else {
    return `Performance degraded by ${Math.abs(impact)} points`;
  }
}
```

## Performance Optimization Strategies

### 4. Theme App Extensions Usage

- **ALWAYS** use theme app extensions instead of direct theme code modification
- **PROTECT** theme code integrity and prevent breaking changes
- **LEVERAGE** automatic CDN serving for all assets/ folder files
- **REFERENCE** assets using:
  - `javascript` and `stylesheet` schema attributes
  - `asset_url` and `asset_img_url` Liquid URL filters

#### Theme App Extension Best Practices

```liquid
<!-- Use schema attributes for JavaScript -->
{% schema %}
{
  "javascript": "app-functionality.js"
}
{% endschema %}

<!-- Use asset_url filter for manual references -->
<script src="{{ 'custom-script.js' | asset_url }}" defer></script>
<link href="{{ 'app-styles.css' | asset_url }}" rel="stylesheet">
```

### 5. App Embed Block Optimization

- **USE** app embed blocks for page-specific functionality
- **LOAD** scripts only on pages where needed
- **MINIMIZE** performance impact through selective loading
- **AVOID** global script loading when functionality is page-specific

#### Selective Loading Pattern

```liquid
{% if template contains 'product' %}
  <!-- Only load product-specific functionality -->
  <script src="{{ 'product-enhancement.js' | asset_url }}" defer></script>
{% endif %}

{% if template contains 'collection' %}
  <!-- Only load collection-specific functionality -->
  <script src="{{ 'collection-filters.js' | asset_url }}" defer></script>
{% endif %}
```

### 6. Shopify CDN Asset Hosting

- **HOST** assets on Shopify servers using File GraphQL resource
- **DELIVER** through Shopify's globally distributed CDN
- **AVOID** unnecessary HTTP connections to external hosts
- **LEVERAGE** HTTP/2 prioritization for blocking resources
- **BENEFIT** from built-in caching, compression, and fast performance

#### CDN Asset Implementation

```javascript
// Use Shopify's File GraphQL resource for static assets
const CREATE_FILE = `
  mutation fileCreate($files: [FileCreateInput!]!) {
    fileCreate(files: $files) {
      files {
        id
        fileStatus
        ... on GenericFile {
          url
          mimeType
        }
      }
      userErrors {
        field
        message
      }
    }
  }
`;

// Upload and reference assets through Shopify CDN
async function uploadAssetToShopifyCDN(fileContent, filename) {
  const response = await shopify.graphql(CREATE_FILE, {
    variables: {
      files: [
        {
          originalSource: fileContent,
          filename: filename,
          contentType: getMimeType(filename),
        },
      ],
    },
  });

  return response.fileCreate.files[0].url;
}
```

## Performance Monitoring & Optimization

### 7. Continuous Performance Testing

- **IMPLEMENT** automated performance testing in CI/CD pipeline
- **MONITOR** performance impact across theme updates
- **TEST** with multiple Shopify themes to ensure compatibility
- **TRACK** performance trends over time
- **ALERT** when performance degradation exceeds acceptable thresholds

### 8. Asset Optimization Strategies

- **MINIMIZE** JavaScript and CSS bundle sizes
- **COMPRESS** images and media assets before hosting
- **USE** lazy loading for non-critical resources
- **IMPLEMENT** code splitting for large applications
- **DEFER** non-essential script execution

#### Asset Optimization Examples

```javascript
// Lazy load non-critical functionality
function loadFeatureOnDemand() {
  return import("./advanced-features.js")
    .then((module) => module.initializeFeature())
    .catch((error) => console.warn("Feature loading failed:", error));
}

// Defer script execution until user interaction
document.addEventListener(
  "click",
  function () {
    loadFeatureOnDemand();
  },
  { once: true }
);
```

### 9. Network Optimization

- **REDUCE** number of external HTTP requests
- **COMBINE** multiple asset files when possible
- **USE** resource hints (preload, prefetch) strategically
- **IMPLEMENT** efficient caching strategies
- **OPTIMIZE** critical rendering path

### 10. Performance Budget Management

- **ESTABLISH** performance budgets for different page types
- **MONITOR** asset sizes and loading times
- **PREVENT** performance regressions through automated checks
- **DOCUMENT** performance optimization decisions
- **REVIEW** performance impact of new features before deployment

#### Performance Budget Example

```javascript
// Performance budget configuration
const PERFORMANCE_BUDGETS = {
  home: {
    maxLighthouseImpact: -2,
    maxAssetSize: "50KB",
    maxRequests: 5,
  },
  product: {
    maxLighthouseImpact: -3,
    maxAssetSize: "75KB",
    maxRequests: 7,
  },
  collection: {
    maxLighthouseImpact: -3,
    maxAssetSize: "60KB",
    maxRequests: 6,
  },
};
```

## App Store Compliance

### 11. Performance Standards for App Store

- **MAINTAIN** consistently low negative impact on merchant stores
- **DEMONSTRATE** performance optimization over time
- **PROVIDE** clear documentation of performance considerations
- **TEST** across multiple merchant scenarios and themes
- **ENSURE** compliance with Shopify App Store performance requirements

### 12. Performance Documentation Requirements

- **DOCUMENT** performance testing procedures used
- **PROVIDE** before/after performance comparisons
- **EXPLAIN** optimization strategies implemented
- **SHARE** best practices for merchant configuration
- **MAINTAIN** performance improvement changelog
