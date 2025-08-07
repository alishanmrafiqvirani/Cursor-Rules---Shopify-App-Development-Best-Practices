---
description: Shopify Checkout Shipping App Performance Guidelines
globs:
  [
    "**/*.js",
    "**/*.ts",
    "**/*.jsx",
    "**/*.tsx",
    "**/*.py",
    "**/*.rb",
    "**/*.php",
    "**/*.go",
    "**/*.java",
    "**/*.cs",
    "**/*.cpp",
    "**/*.c",
  ]
alwaysApply: false
---

# Shopify Checkout Shipping App Performance Guidelines

## Overview Requirements

### Performance Impact Priority

- **OPTIMIZE** shipping rate response times for checkout experience
- **MAINTAIN** fast response time and low error rate for Built for Shopify eligibility
- **PREVENT** checkout blocking due to slow shipping calculations
- **PRIORITIZE** customer checkout experience over rate accuracy
- **WHY**: Shipping app response time directly impacts customer conversion rates

## Carrier Rate Optimization

### 1. Limit External Carrier Calls

- **STORE** stable carrier retail rates internally to avoid external calls
- **ELIMINATE** unnecessary API calls to shipping carriers during checkout
- **IDENTIFY** static rate patterns that can be pre-calculated
- **CACHE** dynamic rates when possible instead of real-time fetching
- **WHY**: External carrier API calls are the primary source of checkout delays

### 2. Implement Intelligent Caching Strategy

- **BUILD** caching layer for dynamic and user-specific carrier rates
- **IDENTIFY** common response patterns for cache key creation
- **USE** cache pattern format:

  ```text
  #{carrier_name}_#{origin_postal_code}_#{destination_postal_code}_#{total_items_weight}
  ```

- **EXAMPLE** cache key implementation:

  ```javascript
  // Cache key: shopify_post_C3C3C3_K2K2K2_1
  const cacheKey = `${carrierName}_${originPostal}_${destPostal}_${totalWeight}`;
  ```

- **IMPLEMENT** cache validation strategy:
  - Store responses in Redis or similar memory database
  - Make background requests to validate cache accuracy
  - Update cache with new information when required
  - Monitor cache hit/miss ratios and response change frequency

### 3. Caching Implementation Requirements

- **ANALYZE** external call response data for patterns across use cases
- **STORE** similar requests in fast-access memory database (Redis recommended)
- **VALIDATE** cache accuracy with background external requests
- **MONITOR** cache performance metrics continuously
- **UPDATE** cache with new information when validation fails

## External System Integration

### 4. Parallelize External Calls with Timeouts

- **MAKE** calls to multiple external systems in parallel, never sequentially
- **SET** internal timeout that's shorter than Shopify's checkout timeout
- **CANCEL** requests to external systems if they fail to respond within timeout
- **RETURN** subset of available shipping rates when some carriers timeout
- **ENSURE** checkout never blocks due to slow external responses

#### Parallel Implementation Pattern

```javascript
const carrierPromises = carriers.map((carrier) =>
  Promise.race([fetchCarrierRates(carrier), timeout(INTERNAL_TIMEOUT_MS)])
);

const results = await Promise.allSettled(carrierPromises);
const successfulRates = results
  .filter((result) => result.status === "fulfilled")
  .map((result) => result.value);
```

### 5. Server Hosting Optimization

- **HOST** on Google Cloud for optimal Shopify server communication
- **LOCATE** servers in regions closest to your primary merchant base
- **MEASURE** latency using cURL timing analysis:

  ```bash
  # Create curl-format.txt with timing template
  curl -w "@curl-format.txt" -o /dev/null -s "http://your-shipping-endpoint/"
  ```

- **COMPARE** connection times with Google Cloud inter-region latency matrix
- **MIGRATE** to Google Cloud if connection time significantly exceeds optimal latency
- **WHY**: Shopify servers are hosted on Google Cloud infrastructure

#### cURL Timing Template (curl-format.txt)

```text
time_namelookup:     %{time_namelookup}s\n
time_connect:        %{time_connect}s\n
time_appconnect:     %{time_appconnect}s\n
time_pretransfer:    %{time_pretransfer}s\n
time_redirect:       %{time_redirect}s\n
time_starttransfer:  %{time_starttransfer}s\n
                    ----------\n
time_total:          %{time_total}s\n
```

## Backup Rate Implementation

### 6. Implement Comprehensive Backup Rates

- **DEFINE** backup rate logic (expensive/fast vs cheap/slow options)
- **BASE** backup rates on historical carrier rate data
- **STORE** backup rates in fast-access datastore
- **PROVIDE** backup rates when external systems fail or timeout
- **ENSURE** backup rates are close enough to avoid profitability impact

### 7. Backup Rate Structure Requirements

- **ORGANIZE** by logical groupings (city, country, weight, size ranges)
- **INCLUDE** multiple shipping options (standard, express, etc.)
- **DEFINE** conditional logic for rate selection
- **EXAMPLE** backup rate structure:

  ```yaml
  rate_definitions:
    - name: "Standard"
      price: 0
      currency: "CAD"
      rate_class_id: 10
      conditions:
        - field: "total_price"
          criteria: 100
          criteria_unit: "CAD"
          operator: "greater_than_or_equal_to"

    - name: "Standard"
      price: 14.90
      currency: "CAD"
      conditions:
        - field: "total_weight"
          criteria: 2.0
          criteria_unit: "kg"
          operator: "less_than_or_equal_to"

    - name: "Express"
      price: 21.90
      currency: "CAD"
      conditions:
        - field: "total_weight"
          criteria: 2.0
          criteria_unit: "kg"
          operator: "less_than_or_equal_to"
  ```

## Performance Monitoring

### 8. Response Time Monitoring

- **TRACK** end-to-end response times for shipping rate requests
- **MONITOR** external carrier API response times separately
- **LOG** timeout incidents and fallback rate usage
- **MEASURE** cache hit ratios and performance impact
- **ALERT** when response times exceed acceptable thresholds

### 9. Error Handling & Reliability

- **IMPLEMENT** graceful degradation when carriers are unavailable
- **LOG** all external API failures for analysis
- **PROVIDE** meaningful error messages without blocking checkout
- **ENSURE** backup rates are always available as fallback
- **MONITOR** error rates to maintain Built for Shopify eligibility

## Built for Shopify Compliance

### 10. Performance Targets

- **MAINTAIN** fast response times consistently
- **ACHIEVE** low error rates across all shipping calculations
- **ENSURE** checkout never blocks due to shipping rate calculations
- **OPTIMIZE** for customer conversion over perfect rate accuracy
- **DOCUMENT** performance improvements and monitoring strategies
