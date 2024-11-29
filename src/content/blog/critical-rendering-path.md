---
title: Critical Rendering Path
description: The critical rendering path is the series of steps the browser uses to convert HTML, CSS, and Javascript into actual pixels on the screen.
pubDate: 2024-11-29
---

# Critical Rendering Path

## Overview

The **critical rendering path** is the series of steps the browser uses to convert HTML, CSS, and Javascript into actual pixels on the screen.

## Steps

<table>
  <tbody>
    <tr>
      <td>
        HTML Parsing & DOM Construction
      </td>
      <td>
        <ul>
          <li>Browser receives HTML bytes and converts them to tokens</li>
          <li>Tokens are converted to nodes</li>
          <li>Nodes form the DOM (Document Object Model) tree</li>
          <li>Process is blocked by synchronous Javascript</li>
        </ul>
      </td>
    </tr>

    <tr>
      <td>
        CSSOM Construction
      </td>
      <td>
        <ul>
          <li>CSS bytes → tokens → nodes → CSSOM tree</li>
          <li>This is a blocking process - render tree construction can't proceed without it</li>
          <li>More specific selectors can slow down CSSOM construction</li>
        </ul>
      </td>
    </tr>

    <tr>
      <td>
        Render Tree Construction
      </td>
      <td>
        <ul>
          <li>Combines DOM and CSSOM</li>
          <li>Only contains visible elements (e.g., display: none elements are excluded)</li>
          <li>Represents what will actually be rendered on the page</li>
        </ul>
      </td>
    </tr>

    <tr>
      <td>
        Layout (Reflow)
      </td>
      <td>
        <ul>
          <li>Calculates exact positions and sizes of elements</li>
          <li>Output is a "box model"</li>
          <li>Triggered by:
            <ul>
              <li>Initial page load</li>
              <li>DOM changes affecting geometry</li>
              <li>Browser window resize</li>
              <li>Getting certain element properties (e.g., offsetHeight)</li>
            </ul>
          </li>
        </ul>
      </td>
    </tr>

    <tr>
      <td>
        Paint
      </td>
      <td>
        <ul>
          <li>Converts render tree into actual pixels</li>
          <li>Fills in all visual elements (colors, borders, shadows, etc.)</li>
          <li>Usually happens in multiple layers</li>
        </ul>
      </td>
    </tr>

  </tbody>
</table>

## Key Points

### Loading Order & Blocking

HTML → CSS (CSSOM) → JavaScript → Render Tree → Layout → Paint

Blocking Behavior:

- CSS: Blocks rendering, blocks
- JavaScript
  execution JavaScript: Blocks HTML parsing (unless async/defer)
- Images: Don't block rendering

### Key Events in Order

1. First Paint (FP) - First visual change
2. First Contentful Paint - First text/image appears
3. DOMContentLoaded - DOM ready defer scripts done
4. Load - All resources loaded

### Critical Optimizations

Minimize blocking CSS

```html
<style>
  /* Critical CSS inline */
</style>
<link rel="stylesheet" href="non-critical.css" media="print" />
```

Handle JavaScript efficiently

```html
<script defer src="app.js"></script>
// For dependencies
<script async src="analytics.js"></script>
// For independent scripts
```

Preload critical resources

```html
<link rel="preload" as="script" href="critical.js" />
<link rel="preload" as="font" href="font.woff2" crossorigin />
```

### Key Performance Metrics

Time to First Byte → First Paint → First Contentful Paint → Time to Interactive
