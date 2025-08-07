---
description: Shopify Admin App Performance & OAuth Best Practices
globs:
  [
    "**/*.html",
    "**/*.js",
    "**/*.ts",
    "**/*.jsx",
    "**/*.tsx",
    "**/*.css",
    "**/*.scss",
    "**/*.liquid",
  ]
alwaysApply: false
---

# Shopify Admin App Performance & OAuth Guidelines

## Loading Performance Requirements

### 1. Admin App Performance Priority

- **OPTIMIZE** loading performance as critical UX factor in Shopify admin
- **MINIMIZE** visual noise including layout shifts
- **IMPLEMENT** clear loading progress indicators
- **FOLLOW** Polaris best practices for loading experiences
- **WHY**: Poor loading performance increases bounce rates and decreases adoption

### 2. Web Vitals Integration Requirements

- **MANDATORY** include App Bridge script for Web Vitals tracking:

  ```html
  <head>
    <script src="https://cdn.shopify.com/shopifycloud/app-bridge.js"></script>
  </head>
  ```

- **UPDATE** to latest App Bridge version for Built for Shopify eligibility
- **UNDERSTAND** embedded apps run in iFrames affecting measurement tools like Lighthouse
- **WHY**: Required for Shopify performance monitoring and Built for Shopify status

## Web Vitals Monitoring & Debugging

### 3. Real-Time Performance Monitoring

- **IMPLEMENT** Web Vitals API for real-time performance insights:

  ```html
  <head>
    <script src="https://cdn.shopify.com/shopifycloud/app-bridge.js"></script>
    <script>
      function processWebVitals(metrics) {
        const monitorUrl = "https://yourserver.com/web-vitals-metrics";
        const data = JSON.stringify(metrics);
        navigator.sendBeacon(monitorUrl, data);
      }

      shopify.webVitals.onReport(processWebVitals);
    </script>
  </head>
  ```

- **SEND** metrics to your own server for analysis
- **MONITOR** appId, shopId, userId, and appLoadId for context
- **TRACK** LCP, FCP, CLS, and INP metrics continuously

### 4. Performance Debugging Setup

- **ENABLE** detailed Web Vitals logging with debug flag:

  ```html
  <head>
    <meta name="shopify-debug" content="web-vitals" />
    <script src="https://cdn.shopify.com/shopifycloud/app-bridge.js" />
  </head>
  ```

- **USE** console logs for real-time Web Vitals attribution data
- **ANALYZE** send-time logs showing final recorded values
- **IDENTIFY** slow-loading elements and inefficient routines
- **WHY**: Essential for debugging specific performance issues

## Built for Shopify Performance Criteria

### 5. Largest Contentful Paint (LCP) Requirements

- **MANDATORY** target: **≤ 2.5 seconds** for 75% of loads over 28 days
- **MEASURE** time from page load start to largest image/text block display
- **OPTIMIZE** main content rendering speed
- **MONITOR** consistently to maintain Built for Shopify status

### 6. Cumulative Layout Shift (CLS) Requirements

- **MANDATORY** target: **≤ 0.1** for 75% of loads over 28 days
- **ELIMINATE** unexpected layout shifts that move interactive elements
- **RESERVE** space for dynamically loaded content
- **TEST** visual stability across different screen sizes
- **WHY**: Prevents frustrating user interactions with moving UI elements

### 7. Interaction to Next Paint (INP) Requirements

- **MANDATORY** target: **≤ 200 milliseconds** for 75% of interactions over 28 days
- **OPTIMIZE** responsiveness to clicks, taps, and keyboard interactions
- **MINIMIZE** JavaScript execution blocking the main thread
- **REQUIRE** latest App Bridge version for INP collection
- **MEASURE** throughout entire user visit lifespan

## OAuth Flow Optimization

### 8. OAuth Performance Best Practices

- **STRONGLY RECOMMEND** using Remix app template with built-in OAuth
- **USE** Shopify CLI for automatic OAuth configuration
- **IMPLEMENT** token exchange grant type for embedded apps:
  - Eliminates multiple redirects
  - Requires only session token exchange for access token
  - Significantly improves authorization performance

### 9. OAuth Implementation Strategy

- **ENABLE** Shopify managed installation for improved UX
- **ELIMINATE** need for custom installation/scope update flows
- **USE** API libraries that simplify OAuth implementation
- **ENSURE** authorization is smooth, fast, and polished first impression
- **WHY**: Authorization is users' first interaction with your app UI

## Performance Monitoring Standards

### 10. Continuous Performance Tracking

- **ESTABLISH** baseline performance metrics before optimization
- **SET UP** automated performance monitoring and alerting
- **COMPARE** your metrics with Built for Shopify thresholds
- **DOCUMENT** performance improvements and regressions
- **REVIEW** 28-day rolling averages for compliance tracking

### 11. Admin-Specific Considerations

- **ACCOUNT** for iframe embedding affecting measurement tools
- **NORMALIZE** metrics collection across different application stacks
- **UNDERSTAND** Shopify records Web Vitals in separate runtime
- **RECONCILE** potential discrepancies with third-party performance tools
- **FOCUS** on Shopify's official measurements for Built for Shopify status
