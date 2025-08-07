---
description: Shopify App Development Best Practices
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

# Shopify App Development Best Practices

## Mobile Compatibility & Performance

### 1. Viewport Meta Tag Requirements

- **ALWAYS** include viewport meta tag in HTML files for embedded apps:

  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  ```

- **WHY**: Prevents double rendering in Shopify Mobile app WebView
- **WHEN**: Required for any app that can be embedded in Shopify admin

### 2. Script Loading Optimization

- **NEVER** use parser-blocking scripts without defer/async attributes
- **USE** `defer` when script execution order matters:

  ```html
  <script src="https://cdn.shopify.com/app-code.js" defer></script>
  <script src="{{ 'app-code.js' | asset_url }}" defer></script>
  ```

- **USE** `async` when script execution order doesn't matter:

  ```html
  <script src="https://cdn.shopify.com/app-code.js" async></script>
  ```

- **WHY**: Improves First Contentful Paint and Largest Contentful Paint metrics

## JavaScript & Framework Guidelines

### 3. Framework and Library Usage

- **AVOID** third-party frameworks (React, Angular, Vue) unless absolutely necessary
- **AVOID** large utility libraries like jQuery
- **PREFER** native browser features and modern DOM APIs
- **TARGET** browsers with > 1% market share (no polyfills for very old browsers)
- **WHY**: Reduces bundle size and prevents multiple framework installations

### 4. JavaScript vs CSS Preference

- **PREFER** CSS for interactivity over JavaScript whenever possible
- **LIMIT** minified JavaScript bundle to **16 KB or less**
- **USE** CSS features for animations, transitions, and interactive elements
- **WHY**: CSS parses and renders much faster than JavaScript

### 5. Namespace Collision Prevention

- **WRAP** all JavaScript in function scopes to avoid global namespace pollution
- **USE** Immediately Invoked Function Expression (IIFE) pattern:

  ```javascript
  (function () {
    var localVariable;
    function localFunction() {}
    // Your app code here
  })();
  ```

- **WHY**: Prevents variable name collisions with other apps and themes

## Performance Optimization

### 6. Lazy Loading Strategy

- **DEFER** loading non-critical resources until user interaction
- **IMPLEMENT** import-on-interaction pattern for optional components
- **LOAD** JavaScript bundles only when needed by user actions
- **WHY**: Reduces initial page load time and unnecessary resource consumption

### 7. Bundle Size Limits

- **LIMIT** app entry point to:
  - **< 10KB JavaScript**
  - **< 50KB CSS**
- **MINIMIZE** and optimize all code bundles
- **LOAD** main functionality on user interaction, not on page load
- **WHY**: Maintains fast store performance and good user experience

### 8. Resource Loading Order

- **ALWAYS** place inline script tags BEFORE remote stylesheets
- **CORRECT** order:

  ```html
  <script>
    console.log("inline script first");
  </script>
  <link href="//example.com/app-css.css" rel="stylesheet" />
  ```

- **WHY**: Prevents stylesheets from blocking inline script execution

## Code Quality Standards

### General Development Rules

- **VALIDATE** all external inputs to prevent security vulnerabilities
- **USE** environment variables for sensitive credentials (never hardcode)
- **IMPLEMENT** robust error handling with try-catch blocks
- **DESIGN** components with loose coupling and single responsibility
- **COMMENT** complex logic explaining WHY, not just WHAT
- **TEST** app performance impact on store themes

### Performance Monitoring

- **MEASURE** First Contentful Paint and Largest Contentful Paint metrics
- **MONITOR** bundle sizes and loading times
- **VERIFY** mobile compatibility in Shopify Mobile app WebView
- **TEST** app functionality without blocking store theme interactivity
