# Marketlytics SaaS - API Documentation

Complete API documentation for integrating WordPress plugin with your SaaS application.

---

## Table of Contents

1. [Quick Summary - 4 Main Steps](#-quick-summary---4-main-steps)
2. [Overview](#overview)
3. [Authentication](#authentication)
4. [API Endpoints](#api-endpoints)
5. [WordPress Plugin Integration Flow](#wordpress-plugin-integration-flow)
6. [Security](#security)
7. [Error Handling](#error-handling)
8. [Testing](#testing)
9. [Database Schema](#database-schema)
10. [Blog Post Data Structure](#blog-post-data-structure)
11. [Rate Limiting (Optional)](#rate-limiting-optional)
12. [Logging](#logging)
13. [Environment Variables](#environment-variables)
14. [Quick Start Checklist](#quick-start-checklist)
15. [Support & Resources](#support--resources)
16. [Version History](#version-history)

---

## 🎯 Quick Summary - 4 Main Steps

### Step 1: WordPress Connect Button Click

**User Action:** User clicks "Connect" button in WordPress admin panel

**What WordPress Does:**

- WordPress plugin generates a secret key
- WordPress plugin sends connection request to SaaS app

**SaaS App API Hit:**

```
POST /api/v1/connect
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "name": "admin@example.com",
  "secret": "a1b2c3d4e5f6...",
  "webhook": "http://example.com/wp-admin/admin-ajax.php?action=marketlytics-notify",
  "rest": "http://example.com/wp-json/marketlytics/v1"
}
```

**What SaaS App Does:**

- SaaS app receives connection request
- SaaS app generates unique API key
- SaaS app saves connection data to database
- SaaS app returns API key to WordPress

**SaaS App Response:**

```json
{
  "status": 1,
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "message": "Connected successfully"
}
```

**What WordPress Does:**

- WordPress plugin receives API key
- WordPress plugin saves API key and secret to WordPress database ✅

---

### Step 2: Send Blog Posts to WordPress

**What SaaS App Does:**

- SaaS app generates blog posts
- SaaS app adds HMAC signature
- SaaS app sends posts to WordPress REST API

**WordPress REST API Hit:**

```
POST {wordpress_site_url}/wp-json/marketlytics/v1/bulk-posts
```

**Request Body (SaaS App sends to WordPress):**

```json
{
  "posts": [
    {
      "title": "Complete Guide to Technology",
      "content": "<h2>Introduction</h2><p>Welcome...</p><img src=\"https://picsum.photos/1200/600?random=123\" />",
      "excerpt": "Discover best practices...",
      "status": "draft",
      "date": "2025-11-04T10:30:00.000Z",
      "categories": ["Technology"],
      "tags": ["Tips", "Guide"],
      "featured_image": "https://picsum.photos/1920/1080?random=123"
    }
  ],
  "sign": "hmac_signature_here"
}
```

**What WordPress Does:**

- WordPress plugin receives request
- WordPress plugin verifies HMAC signature
- WordPress plugin creates posts in WordPress database
- WordPress plugin downloads images to media library
- WordPress plugin sets categories, tags, featured images
- WordPress plugin logs the operation
- WordPress plugin returns results to SaaS app

**WordPress Plugin Response:**

```json
{
  "success": true,
  "results": {
    "success": 1,
    "failed": 0,
    "post_ids": [1263],
    "image_stats": {
      "total_images_processed": 1,
      "content_images": 1,
      "featured_images": 1
    }
  }
}
```

**Result:** Blog post created in WordPress! ✅

---

### Step 3: Generate New Article

**User Action:** User clicks "Generate New Article" button (located right after "Add Post" button on WordPress posts list page) and provides a topic/keyword

**What WordPress Does:**

- WordPress plugin sends topic/keyword to SaaS app
- WordPress plugin receives generated article data

**SaaS App API Hit:**

```
POST /api/v1/generate-article
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "topic": "Best practices for technology",
  "auto_publish": false
}
```

**What SaaS App Does:**

- SaaS app validates API key
- SaaS app generates complete article (title, content, excerpt, categories, tags, SEO meta, featured image, content images)
- SaaS app returns generated article data

**SaaS App Response:**

```json
{
  "success": true,
  "message": "Article generated successfully",
  "article": {
    "title": "Complete Guide to Best practices for technology",
    "content": "<h2>Introduction</h2><p>Welcome...</p><img src=\"https://picsum.photos/1200/600?random=123\" />",
    "excerpt": "Discover the best practices...",
    "categories": ["Technology", "Business"],
    "tags": ["Tips", "Guide", "Expert"],
    "featured_image": "https://picsum.photos/1920/1080?random=456",
    "seo": {
      "title": "Complete Guide to Best practices for technology | Expert Guide",
      "description": "Discover the best practices...",
      "keywords": "Technology, Business, Tips"
    }
  }
}
```

**What WordPress Does:**

- WordPress plugin creates new post (as draft)
- WordPress plugin downloads images to media library
- WordPress plugin sets categories, tags, featured image, SEO meta
- WordPress plugin shows success message

**Result:** New article created in WordPress! ✅

---

### Step 4: Optimize Existing Article

**User Action:** User clicks "Optimize Article" button on WordPress single post edit page

**What WordPress Does:**

- WordPress plugin collects existing post data (title, content, excerpt, categories, tags)
- WordPress plugin sends post data to SaaS app

**SaaS App API Hit:**

```
POST /api/v1/optimize-article
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "post_data": {
    "post_id": 123,
    "title": "Original Article Title",
    "content": "<p>Original content...</p>",
    "excerpt": "Original excerpt",
    "categories": ["Technology"],
    "tags": ["Tips"]
  }
}
```

**What SaaS App Does:**

- SaaS app validates API key
- SaaS app optimizes article content (improves title, enhances content structure, improves excerpt, generates SEO meta)
- SaaS app returns optimized content

**SaaS App Response:**

```json
{
  "success": true,
  "message": "Article optimized successfully",
  "optimized": {
    "title": "Ultimate Original Article Title",
    "content": "<h2>Introduction</h2><p>Improved content...</p>",
    "excerpt": "Discover the best practices and expert insights about Original Article Title.",
    "seo": {
      "title": "Ultimate Original Article Title",
      "description": "Discover the best practices and expert insights about Original Article Title. Read more to discover actionable insights and strategies.",
      "keywords": "Technology, Tips"
    }
  }
}
```

**What WordPress Does:**

- WordPress plugin updates post with optimized content
- WordPress plugin updates SEO meta tags (Yoast/Rank Math)
- WordPress plugin shows success message

**Result:** Article optimized and updated! ✅

---

## Overview

### Base URL

```
Production: https://your-saas-app.com/api/v1
Development: http://localhost:3000/api/v1
```

### Content Type

All requests and responses use `application/json`

### Response Format

```json
{
  "status": 1, // 1 for success, 0 for error
  "message": "...", // Human-readable message
  "data": {} // Response data (varies by endpoint)
}
```

---

## Authentication

### API Key & Secret

Each connected WordPress site has:

- **API Key**: Public identifier (sent with requests)
- **Secret Key**: Private key for signing requests (never sent directly)

### HMAC Signature

All requests are signed using HMAC-SHA256:

```javascript
// Node.js Example
const crypto = require('crypto');

function getSignature(data, secret) {
  const message = JSON.stringify(data);
  return crypto.createHmac('sha256', secret).update(message).digest('hex');
}

// Usage
const requestData = {
  posts: [...],
  timestamp: new Date().toISOString()
};
requestData.sign = getSignature(requestData, secret);
```

### Signature Verification

```javascript
function checkSign(data, secret) {
  const sign = data.sign;
  delete data.sign;
  const expectedSign = getSignature(data, secret);
  data.sign = sign;
  return sign === expectedSign;
}
```

---

## API Endpoints

### 1. Connect WordPress Site

**Endpoint:** `POST /api/v1/connect`

**Purpose:** Establish connection between WordPress site and SaaS app

**Request Body:**

```json
{
  "name": "admin@example.com", // Admin email
  "secret": "generated_secret_key", // Secret for signing
  "webhook": "https://example.com/wp-admin/admin-ajax.php?action=marketlytics-notify",
  "rest": "https://example.com/wp-json/marketlytics/v1", // REST API base URL
  "success_url": "https://example.com/wp-admin/options-general.php?page=marketlytics-setting&connected=1",
  "failure_url": "https://example.com/wp-admin/options-general.php?page=marketlytics-setting&error=1"
}
```

**Response (Success):**

```json
{
  "status": 1,
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2", // Save this in WordPress
  "message": "Connected successfully"
}
```

**Response (Error):**

```json
{
  "status": 0,
  "error": "Site already connected"
}
```

**Implementation:**

```javascript
app.post("/api/v1/connect", async (req, res) => {
  try {
    const { name, secret, webhook, rest, success_url, failure_url } = req.body;

    // Validate required fields
    if (!name || !secret || !webhook) {
      return res.status(400).json({
        status: 0,
        error: "Missing required fields: name, secret, webhook",
      });
    }

    // Extract site URL from webhook
    let siteUrl = webhook.split("/wp-admin")[0] || webhook.split("/wp-json")[0];
    siteUrl = siteUrl.replace(/\/wp-admin\/?$/, "").replace(/\/$/, "");

    // Check if site already exists
    const existingSite = await WordPressSite.findOne({ site_url: siteUrl });

    let apiKey;
    if (existingSite && existingSite.status === "connected") {
      apiKey = existingSite.api_key;
    } else {
      // Generate new API key
      apiKey = crypto.randomBytes(32).toString("hex");

      const siteData = {
        site_url: siteUrl,
        user_email: name,
        api_key: apiKey,
        secret: secret,
        webhook_url: webhook,
        rest_url: rest || "",
        status: "connected",
        updated_at: new Date(),
      };

      if (existingSite) {
        await WordPressSite.updateOne({ site_url: siteUrl }, siteData);
      } else {
        await WordPressSite.create(siteData);
      }
    }

    res.json({
      status: 1,
      api_key: apiKey,
      message: "Connected successfully",
    });
  } catch (error) {
    res.status(500).json({
      status: 0,
      error: error.message || "Internal server error",
    });
  }
});
```

---

### 2. Disconnect WordPress Site

**Endpoint:** `POST /api/v1/disconnect`

**Purpose:** Disconnect WordPress site from SaaS app

**Request Body:**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2"
}
```

**Response (Success):**

```json
{
  "status": 1,
  "message": "Disconnected successfully"
}
```

**Implementation:**

```javascript
app.post("/api/v1/disconnect", async (req, res) => {
  try {
    const { api_key } = req.body;

    if (!api_key) {
      return res.status(400).json({
        status: 0,
        error: "API key is required",
      });
    }

    const site = await WordPressSite.findOne({ api_key });

    if (!site) {
      return res.status(404).json({
        status: 0,
        error: "Site not found",
      });
    }

    site.status = "disconnected";
    site.updated_at = new Date();
    await site.save();

    res.json({
      status: 1,
      message: "Disconnected successfully",
    });
  } catch (error) {
    res.status(500).json({
      status: 0,
      error: error.message || "Internal server error",
    });
  }
});
```

---

### 3. Generate Articles (Single or Bulk)

**Endpoint:** `POST /api/v1/generate-article`

**Purpose:** Generate single article or multiple articles (bulk). This unified API handles both use cases.

**When to Use:**

- **Single Article:** When you need to generate one article based on a topic/keyword
- **Bulk Articles:** When you need to generate multiple articles (bulk import)

**Request Body (Single Article):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "topic": "WordPress Development",
  "auto_publish": false
}
```

**Request Body (Bulk Articles):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "count": 20
}
```

**Response (Single Article - Success):**

```json
{
  "success": true,
  "message": "Article generated successfully",
  "article": {
    "title": "Complete Guide to WordPress Development",
    "content": "<h2>Introduction</h2><p>Welcome...</p><img src=\"https://picsum.photos/1200/600?random=123\" />",
    "excerpt": "Discover everything you need to know...",
    "categories": ["Technology"],
    "tags": ["WordPress", "Development", "Guide"],
    "featured_image": "https://picsum.photos/1920/1080?random=456",
    "seo": {
      "title": "Complete Guide to WordPress Development | Expert Guide",
      "description": "Discover everything you need to know...",
      "keywords": "WordPress, Development, Guide"
    }
  }
}
```

**Response (Bulk Articles - Success):**

```json
{
  "success": true,
  "message": "Articles generated successfully",
  "articles": [
    {
      "title": "Complete Guide to Technology - Part 1",
      "content": "<h2>Introduction</h2><p>Welcome...</p>",
      "excerpt": "Discover the best practices...",
      "status": "draft",
      "categories": ["Technology"],
      "tags": ["Tips", "Guide"],
      "featured_image": "https://picsum.photos/1920/1080?random=123",
      "seo": {
        "title": "Complete Guide to Technology - Part 1 | Expert Guide",
        "description": "Discover the best practices...",
        "keywords": "Technology, Tips, Guide"
      }
    }
    // ... more articles
  ],
  "count": 20,
  "image_stats": {
    "total_images": 40,
    "content_images": 20,
    "featured_images": 20
  }
}
```

**Response (Error):**

```json
{
  "success": false,
  "message": "Invalid API key"
}
```

**WordPress REST API Flow (for Bulk):**

When WordPress REST endpoint `/bulk-posts` is called with `count` parameter:

1. WordPress validates HMAC signature
2. WordPress calls SaaS `/api/v1/generate-article` with `count`
3. SaaS generates articles and returns them
4. WordPress creates posts in database
5. WordPress returns results

**WordPress REST Endpoint:**

```
POST {wordpress_site_url}/wp-json/marketlytics/v1/bulk-posts
```

**WordPress REST Request Body:**

```json
{
  "count": 20,
  "sign": "hmac_signature_here"
}
```

**WordPress REST Response:**

```json
{
  "success": true,
  "results": {
    "success": 20,
    "failed": 0,
    "post_ids": [1263, 1265, 1267, ...],
    "errors": [],
    "image_stats": {
      "total_images_processed": 20,
      "content_images": 20,
      "featured_images": 20,
      "images_failed": 0
    }
  }
}
```

**Implementation (SaaS App):**

```javascript
app.post("/api/v1/generate-article", async (req, res) => {
  try {
    const { api_key, topic, count, auto_publish } = req.body;

    if (!api_key) {
      return res.status(400).json({
        success: false,
        message: "API key is required",
      });
    }

    const site = await WordPressSite.findOne({ api_key });
    if (!site) {
      return res.status(401).json({
        success: false,
        message: "Invalid API key",
      });
    }

    // Bulk generation mode
    if (count && parseInt(count) > 1) {
      const blogPosts = generateDummyBlogs(parseInt(count));
      res.json({
        success: true,
        message: "Articles generated successfully",
        articles: blogPosts,
        count: blogPosts.length,
      });
    }
    // Single generation mode
    else if (topic && topic.trim()) {
      const article = generateSingleArticle(topic);
      res.json({
        success: true,
        message: "Article generated successfully",
        article: article,
      });
    } else {
      return res.status(400).json({
        success: false,
        message:
          "Either 'topic' (for single) or 'count' (for bulk) is required",
      });
    }
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to generate article",
      error: error.message,
    });
  }
});
```

---

## WordPress Plugin Integration Flow

### 🔗 Step 1: WordPress Connect Button Click - Complete Flow

#### User Action:

User clicks "Connect" button in WordPress admin panel

---

#### Step-by-Step Flow:

**Step 1.1: WordPress Frontend (JavaScript)**

**What WordPress Does:**

- User clicks "Connect" button
- JavaScript sends AJAX request to WordPress backend

```javascript
// WordPress Plugin - assets/js/settings.js
// User clicks button → AJAX call triggered
$.ajax({
  url: ajaxurl,
  type: "POST",
  data: {
    action: "content_app_connection",
    type: "connect",
    nonce: nonce,
  },
});
```

---

**Step 1.2: WordPress Backend (PHP)**

**What WordPress Does:**

- WordPress plugin generates a random secret key (64 characters)
- WordPress plugin gets current user email
- WordPress plugin prepares connection data

```php
// WordPress Plugin - classes/API/Client.php
// Generate secret key
$secret = bin2hex(random_bytes(32));
// Result: "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6"

// Get current user
$user = wp_get_current_user();

// Prepare connection data
$data = [
    'name' => $user->user_email,  // e.g., "admin@example.com"
    'secret' => $secret,            // Generated secret key
    'webhook' => admin_url('admin-ajax.php?action=marketlytics-notify'),
    // Result: "http://localhost/wpdev/wp-admin/admin-ajax.php?action=marketlytics-notify"
    'rest' => rest_url('marketlytics/v1')
    // Result: "http://localhost/wpdev/wp-json/marketlytics/v1"
];
```

---

**Step 1.3: WordPress → SaaS App API Call**

**What WordPress Does:**

- WordPress plugin sends HTTP POST request to SaaS app

**SaaS App Endpoint:** `POST /api/v1/connect`

**Full URL:**

```
POST http://localhost:3000/api/v1/connect
POST https://your-saas-app.com/api/v1/connect  (Production)
```

**Request Headers:**

```
Content-Type: application/json
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "name": "admin@example.com",
  "secret": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6",
  "webhook": "http://localhost/wpdev/wp-admin/admin-ajax.php?action=marketlytics-notify",
  "rest": "http://localhost/wpdev/wp-json/marketlytics/v1",
  "success_url": "http://localhost/wpdev/wp-admin/options-general.php?page=marketlytics-setting&connected=1",
  "failure_url": "http://localhost/wpdev/wp-admin/options-general.php?page=marketlytics-setting&error=1"
}
```

---

**Step 1.4: SaaS App Processing**

**What SaaS App Does:**

1. Receives connection request from WordPress
2. Validates required fields (name, secret, webhook)
3. Extracts WordPress site URL from webhook URL
   - Input: `http://localhost/wpdev/wp-admin/admin-ajax.php?action=...`
   - Output: `http://localhost/wpdev`
4. Checks if site already exists in database
5. Generates unique API key (64 characters)
6. Saves connection data to MongoDB database
7. Returns API key to WordPress

```javascript
// SaaS App - server.js
app.post("/api/v1/connect", async (req, res) => {
  try {
    const { name, secret, webhook, rest } = req.body;

    // Validate required fields
    if (!name || !secret || !webhook) {
      return res.status(400).json({
        status: 0,
        error: "Missing required fields: name, secret, webhook",
      });
    }

    // Extract WordPress site URL from webhook
    let siteUrl = webhook.split("/wp-admin")[0] || webhook.split("/wp-json")[0];
    siteUrl = siteUrl.replace(/\/wp-admin\/?$/, "").replace(/\/$/, "");
    // Result: "http://localhost/wpdev"

    // Check if site already exists in database
    const existingSite = await WordPressSite.findOne({ site_url: siteUrl });

    let apiKey;
    if (existingSite && existingSite.status === "connected") {
      // Site already connected - return existing API key
      apiKey = existingSite.api_key;
    } else {
      // Generate new unique API key
      apiKey = crypto.randomBytes(32).toString("hex");
      // Result: "695f5ac984a3f7d2e8b1c9d4e6f8a0b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"

      // Prepare data to save in database
      const siteData = {
        site_url: siteUrl, // "http://localhost/wpdev"
        user_email: name, // "admin@example.com"
        api_key: apiKey, // Generated API key
        secret: secret, // Secret from WordPress
        webhook_url: webhook, // Webhook URL
        rest_url: rest || "", // REST API URL
        status: "connected", // Connection status
        updated_at: new Date(), // Current timestamp
      };

      // Save to MongoDB database
      if (existingSite) {
        await WordPressSite.updateOne({ site_url: siteUrl }, siteData);
      } else {
        await WordPressSite.create(siteData);
      }
    }

    // Return API key to WordPress
    res.json({
      status: 1,
      api_key: apiKey,
      message: "Connected successfully",
    });
  } catch (error) {
    res.status(500).json({
      status: 0,
      error: error.message || "Internal server error",
    });
  }
});
```

**What Gets Saved in SaaS App Database (MongoDB):**

```javascript
{
  _id: ObjectId("..."),
  site_url: "http://localhost/wpdev",
  user_email: "admin@example.com",
  api_key: "695f5ac984a3f7d2e8b1c9d4e6f8a0b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  secret: "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6",
  webhook_url: "http://localhost/wpdev/wp-admin/admin-ajax.php?action=marketlytics-notify",
  rest_url: "http://localhost/wpdev/wp-json/marketlytics/v1",
  status: "connected",
  created_at: ISODate("2025-11-04T10:30:00.000Z"),
  updated_at: ISODate("2025-11-04T10:30:00.000Z")
}
```

---

**Step 1.5: SaaS App Response**

**SaaS App Response (Success):**

```json
{
  "status": 1,
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "message": "Connected successfully"
}
```

**HTTP Status Code:** `200 OK`

**SaaS App Response (Error - Site Already Connected):**

```json
{
  "status": 0,
  "error": "Site already connected"
}
```

**HTTP Status Code:** `200 OK` (but status: 0 indicates error)

**SaaS App Response (Error - Missing Fields):**

```json
{
  "status": 0,
  "error": "Missing required fields: name, secret, webhook"
}
```

**HTTP Status Code:** `400 Bad Request`

---

**Step 1.6: WordPress Stores Credentials**

**What WordPress Does:**

- WordPress plugin receives API key from SaaS app
- WordPress plugin saves API key and secret to WordPress database (wp_options table)

```php
// WordPress Plugin - classes/API/Client.php
// After receiving response from SaaS app

// Save API key to WordPress database
update_option('content_generator_api_key', '695f5ac984a3f7d2e8b1c9d4e6f8a0b2...');

// Save secret key to WordPress database
update_option('content_generator_secret', 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6...');

// Save connection status
update_option('content_generator_status', 'connected');
```

**What Gets Saved in WordPress Database (wp_options table):**

```
option_name: content_generator_api_key
option_value: 695f5ac984a3f7d2e8b1c9d4e6f8a0b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6

option_name: content_generator_secret
option_value: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6

option_name: content_generator_status
option_value: connected
```

---

**Final Result:** ✅ WordPress site is now connected with SaaS app!

**Summary:**

- WordPress generates secret key
- WordPress sends connection request to SaaS app
- SaaS app generates API key and saves to database
- SaaS app returns API key to WordPress
- WordPress saves API key and secret to WordPress database

---

### 📝 Step 2: SaaS App Sends Blog Posts to WordPress - Complete Flow

#### When This Happens:

When SaaS app needs to send blog posts to WordPress (manually or automatically)

---

#### Step-by-Step Flow:

**Step 2.1: Client Calls SaaS App API (Optional)**

**What Client Does:**

- Client sends request to SaaS app to generate blog posts

**Step 2.1: Client Calls WordPress REST API (for Bulk Generation)**

**What Client Does:**

- Client sends request to WordPress REST API to generate bulk blog posts

**WordPress REST API Endpoint:** `POST {wordpress_site_url}/wp-json/marketlytics/v1/bulk-posts`

**Request Body (Client sends to WordPress):**

```json
{
  "count": 5,
  "sign": "hmac_signature_here"
}
```

---

**Step 2.2: WordPress Validates Signature and Calls SaaS API**

**What WordPress Does:**

- WordPress validates HMAC signature
- WordPress calls SaaS `/api/v1/generate-article` API with `count` parameter

```javascript
// WordPress Plugin - marketlytics.php (Main plugin file)
// Validates signature
if (!checkSign($post, $settings['secret'])) {
  return error('Invalid signature');
}

// Calls SaaS API
$body = [
  'api_key' => $api_key,
  'count' => $count
];

$response = wp_remote_post('http://localhost:3000/api/v1/generate-article', [
  'body' => wp_json_encode($body),
  'headers' => ['Content-Type' => 'application/json'],
  'timeout' => 300
]);
```

---

**Step 2.3: SaaS App Generates Articles**

**What SaaS App Does:**

- SaaS app receives request with `count` parameter
- SaaS app finds WordPress site using API key
- SaaS app generates blog post data (title, content, categories, tags, images, etc.)

```javascript
// SaaS App - server.js
app.post("/api/v1/generate-article", async (req, res) => {
  const { api_key, count } = req.body;

  const site = await WordPressSite.findOne({ api_key });

  // Generate blog posts
  const blogPosts = generateDummyBlogs(parseInt(count));

  // Result: Array of blog post objects
  // [
  //   {
  //     title: "Complete Guide to Technology",
  //     content: "<h2>Introduction</h2><p>Welcome...</p><img src=\"https://picsum.photos/1200/600?random=1\" />",
  //     excerpt: "Discover best practices...",
  //     status: "draft",
  //     categories: ["Technology"],
  //     tags: ["Tips", "Guide"],
  //     featured_image: "https://picsum.photos/1920/1080?random=123"
  //   },
  //   // ... more posts
  // ]

  res.json({
    success: true,
    articles: blogPosts,
    count: blogPosts.length,
  });
});
```

---

**Step 2.4: WordPress Creates Posts**

**What WordPress Does:**

- WordPress receives articles from SaaS API
- WordPress creates posts in database with categories, tags, images, SEO meta

```php
// WordPress Plugin - blog-handler.php
$results = $blogHandler->createBulkPosts($data['articles']);

// Creates posts, sets categories, tags, featured images, content images, SEO meta
```

#### WordPress Plugin Response (Success):

```json
{
  "success": true,
  "results": {
    "success": 2,
    "failed": 0,
    "post_ids": [1263, 1265],
    "errors": [],
    "image_stats": {
      "total_images_processed": 2,
      "content_images": 2,
      "featured_images": 2,
      "images_failed": 0
    }
  }
}
```

#### WordPress Plugin Response (Error - Invalid Signature):

```json
{
  "success": false,
  "error": "Invalid signature"
}
```

**HTTP Status Code:** `401 Unauthorized`

#### WordPress Plugin Response (Error - Missing Data):

```json
{
  "success": false,
  "error": "Posts data is required"
}
```

**HTTP Status Code:** `400 Bad Request`

**6. WordPress Plugin Processing (Complete Code):**

```php
// WordPress Plugin - marketlytics.php (Main plugin file)
public function restBulkPosts($request) {
    // Get request data from SaaS app
    $post = $request->get_json_params();
    // $post = {
    //   "posts": [...],
    //   "sign": "a7b8c9d0e1f2..."
    // }

    // Get stored settings
    $settings = $this->getSettings();
    $secret = $settings['secret']; // "a1b2c3d4e5f6..." (stored during connect)

    // Verify HMAC signature
    $receivedSign = $post['sign'];
    unset($post['sign']); // Remove sign before verification
    $expectedSign = hash_hmac('sha256', json_encode($post), $secret);

    if ($receivedSign !== $expectedSign) {
        return new WP_REST_Response([
            'success' => false,
            'error' => 'Invalid signature'
        ], 401);
    }

    // Verify posts data exists
    if (!isset($post['posts']) || !is_array($post['posts'])) {
        return new WP_REST_Response([
            'success' => false,
            'error' => 'Posts data is required'
        ], 400);
    }

    // Load blog handler
    require_once($this->plugin_path . 'classes/Blog/Handler.php');
    $blogHandler = new \ContentGenerator\Blog\Handler($this);

    // Create bulk posts
    $results = $blogHandler->createBulkPosts($post['posts']);
    // $results = {
    //   'success' => 5,
    //   'failed' => 0,
    //   'post_ids' => [1263, 1265, 1267, 1269, 1271],
    //   'image_stats' => {...}
    // }

    // Log the operation
    require_once($this->plugin_path . 'classes/log-handler.php');
    $logHandler = new \ContentGenerator\LogHandler($this);
    $totalSent = count($post['posts']);
    $logHandler->logBulkPosts($results, $totalSent);

    // Return success response to SaaS app
    return new WP_REST_Response([
        'success' => true,
        'results' => $results
    ], 200);
}
```

**What WordPress Plugin Does:**

1. ✅ Receives request from SaaS app
2. ✅ Verifies HMAC signature using stored secret
3. ✅ Validates posts data
4. ✅ Creates each post in WordPress
5. ✅ Downloads images to media library
6. ✅ Sets categories, tags, featured images
7. ✅ Logs the operation
8. ✅ Returns results to SaaS app

**7. WordPress Plugin Response (Success):**

```json
{
  "success": true,
  "results": {
    "success": 5,
    "failed": 0,
    "post_ids": [1263, 1265, 1267, 1269, 1271],
    "errors": [],
    "image_stats": {
      "total_images_processed": 5,
      "content_images": 5,
      "featured_images": 5,
      "images_failed": 0
    }
  }
}
```

**8. SaaS App returns final response to client:**

```json
{
  "status": 1,
  "message": "Blogs generated and sent successfully",
  "sent": 5,
  "results": {
    "success": 5,
    "failed": 0,
    "post_ids": [1263, 1265, 1267, 1269, 1271]
  }
}
```

**Final Result:**

- ✅ 5 posts created in WordPress database
- ✅ Images downloaded and added to WordPress media library
- ✅ Featured images set for each post
- ✅ Categories and tags assigned to posts
- ✅ Posts saved as drafts in WordPress
- ✅ Operation logged in WordPress database
- ✅ Client receives success response from SaaS app

---

**Step 2.10: SaaS App Returns Final Response**

**What SaaS App Does:**

- SaaS app receives response from WordPress
- SaaS app returns final response to client

**SaaS App returns final response to client:**

```json
{
  "status": 1,
  "message": "Blogs generated and sent successfully",
  "sent": 5,
  "results": {
    "success": 5,
    "failed": 0,
    "post_ids": [1263, 1265, 1267, 1269, 1271],
    "image_stats": {
      "total_images_processed": 5,
      "content_images": 5,
      "featured_images": 5,
      "images_failed": 0
    }
  }
}
```

---

### Complete Flow Example - Step by Step

#### Scenario: Generate 5 Blog Posts

**1. SaaS App Receives Request:**

```javascript
// SaaS App - server.js
app.post("/api/v1/generate-blogs", async (req, res) => {
  const { api_key, count = 5 } = req.body;

  // Find WordPress site
  const site = await WordPressSite.findOne({ api_key });

  // Generate blog posts
  const blogPosts = generateDummyBlogs(count);

  // Add signature
  const postsData = { posts: blogPosts };
  postsData.sign = getSignature(postsData, site.secret);

  // Send to WordPress
  const response = await axios.post(
    `${site.site_url}/wp-json/marketlytics/v1/bulk-posts`,
    postsData,
    { headers: { "Content-Type": "application/json" } }
  );

  // Return results
  res.json({
    status: 1,
    sent: count,
    results: response.data.results,
  });
});
```

**2. WordPress Plugin Receives & Processes:**

```php
// WordPress Plugin - marketlytics.php (Main plugin file)
public function restBulkPosts($request) {
    $post = $request->get_json_params();

    // Verify signature
    if (!$this->verifySignature($post, $secret)) {
        return new WP_REST_Response(['success' => false, 'error' => 'Invalid signature'], 401);
    }

    // Process each post
    foreach ($post['posts'] as $postData) {
        // Create post
        $postId = wp_insert_post([
            'post_title' => $postData['title'],
            'post_content' => $postData['content'],
            'post_status' => 'draft',
            // ... other fields
        ]);

        // Set categories, tags, images, etc.
        // ...
    }

    // Return success
    return new WP_REST_Response([
        'success' => true,
        'results' => ['success' => 5, 'failed' => 0, 'post_ids' => [...]]
    ], 200);
}
```

**3. Final Result:**

- ✅ 5 posts created in WordPress database (wp_posts table)
- ✅ Images downloaded and added to WordPress media library (wp_posts table with post_type='attachment')
- ✅ Featured images set for each post (wp_postmeta table with \_thumbnail_id)
- ✅ Categories and tags assigned to posts (wp_term_relationships table)
- ✅ Posts saved as drafts in WordPress (post_status='draft')
- ✅ Operation logged in WordPress database (wp_options table)
- ✅ Client receives success response from SaaS app

---

### 📝 Step 3: Generate New Article - Complete Flow

#### User Action:

User clicks "Generate New Article" button (located right after "Add Post" button on WordPress posts list page) and provides a topic/keyword

---

#### Step-by-Step Flow:

**Step 3.1: WordPress Frontend (JavaScript)**

**What WordPress Does:**

- User clicks "Generate New Article" button (located right after "Add Post" button)
- Popup opens asking for topic/keyword
- User enters topic and clicks "Generate Article"
- JavaScript sends AJAX request to WordPress backend

```javascript
// WordPress Plugin - assets/js/settings.js
// User clicks button → AJAX call triggered
$.ajax({
  url: ajaxurl,
  type: "POST",
  data: {
    action: "ml_generate_article",
    nonce: ajax_var.generate_article_nonce,
    topic: topic,
    auto_publish: false,
  },
});
```

---

**Step 3.2: WordPress Backend (PHP) - Call SaaS API**

**What WordPress Does:**

- WordPress plugin validates nonce and connection status
- WordPress plugin calls SaaS app API with topic

```php
// WordPress Plugin - classes/API/Client.php
// Call SaaS API
$data = $api_client->generateArticle($topic, false);

// SaaS API returns:
// {
//   "success": true,
//   "article": {
//     "title": "...",
//     "content": "...",
//     "excerpt": "...",
//     "categories": [...],
//     "tags": [...],
//     "featured_image": "...",
//     "seo": {...}
//   }
// }
```

---

**Step 3.3: WordPress → SaaS App API Call**

**What WordPress Does:**

- WordPress plugin sends HTTP POST request to SaaS app

**SaaS App Endpoint:** `POST /api/v1/generate-article`

**Full URL:**

```
POST http://localhost:3000/api/v1/generate-article
POST https://your-saas-app.com/api/v1/generate-article  (Production)
```

**Request Headers:**

```
Content-Type: application/json
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "topic": "Best practices for technology",
  "auto_publish": false
}
```

---

**Step 3.4: SaaS App Processing**

**What SaaS App Does:**

1. Receives topic from WordPress
2. Validates API key
3. Generates complete article:
   - Title (based on topic with power words)
   - Content (introduction, 3-5 main sections, conclusion)
   - Excerpt
   - Categories (1-3 random)
   - Tags (3-6 random)
   - SEO meta data (title, description, keywords)
   - Featured image URL (1920x1080)
   - Content image URLs (1-2 images inserted in content)
4. Returns generated article data

```javascript
// SaaS App - server.js
app.post("/api/v1/generate-article", async (req, res) => {
  const { api_key, topic } = req.body;

  // Find site by API key
  const site = await WordPressSite.findOne({ api_key });

  // Generate article
  const article = generateSingleArticle(topic);
  // Result: {
  //   title: "Complete Guide to Best practices for technology",
  //   content: "<h2>Introduction</h2>...<img src=\"https://picsum.photos/1200/600?random=123\" />",
  //   excerpt: "Discover the best practices...",
  //   categories: ["Technology", "Business"],
  //   tags: ["Tips", "Guide", "Expert"],
  //   featured_image: "https://picsum.photos/1920/1080?random=456",
  //   seo: {...}
  // }

  res.json({
    success: true,
    message: "Article generated successfully",
    article: article,
  });
});
```

---

**Step 3.5: SaaS App Response**

**SaaS App Response (Success):**

```json
{
  "success": true,
  "message": "Article generated successfully",
  "article": {
    "title": "Complete Guide to Best practices for technology",
    "content": "<h2>Introduction</h2><p>Welcome to our comprehensive guide...</p><img src=\"https://picsum.photos/1200/600?random=123\" alt=\"Best practices for technology\" />",
    "excerpt": "Discover the best practices and expert tips for Best practices for technology.",
    "categories": ["Technology", "Business"],
    "tags": ["Tips", "Guide", "Expert"],
    "featured_image": "https://picsum.photos/1920/1080?random=456",
    "seo": {
      "title": "Complete Guide to Best practices for technology | Expert Guide",
      "description": "Discover the best practices and expert tips for Best practices for technology.",
      "keywords": "Technology, Business, Tips"
    }
  }
}
```

**HTTP Status Code:** `200 OK`

---

**Step 3.6: WordPress Creates New Post**

**What WordPress Does:**

- WordPress plugin receives generated article from SaaS app
- WordPress plugin creates new post (always as draft)
- WordPress plugin downloads images to media library
- WordPress plugin sets categories, tags, featured image, SEO meta

```php
// WordPress Plugin - classes/Admin/SettingsForm.php
// Create new post
$post_id = wp_insert_post([
    'post_title' => $article['title'],
    'post_content' => wp_slash($article['content']),
    'post_excerpt' => $article['excerpt'],
    'post_status' => 'draft', // Always draft
    'post_type' => 'post',
]);

// Set categories
wp_set_post_categories($post_id, $article['categories']);

// Set tags
wp_set_post_tags($post_id, $article['tags']);

// Download and set featured image
$blogHandler->setFeaturedImage($post_id, $article['featured_image']);

// Process content images
$blogHandler->processContentImages($post_id, $article['content']);

// Set SEO meta
$blogHandler->setSEOMeta($post_id, $article['seo']);
```

**What Gets Created in WordPress Database:**

- **wp_posts table**: New post created with `post_status='draft'`
- **wp_term_relationships table**: Categories and tags assigned
- **wp_postmeta table**: Featured image (`_thumbnail_id`), SEO meta
- **wp_posts table** (attachments): Images downloaded to media library

---

**Final Result:** ✅ New article created in WordPress as draft!

**Summary:**

- WordPress sends topic to SaaS app
- SaaS app generates complete article with images
- WordPress creates new post (draft)
- WordPress downloads images and sets featured image
- WordPress sets categories, tags, and SEO meta

---

### 🔧 Step 4: Optimize Existing Article - Complete Flow

#### User Action:

User clicks "Optimize Article" button on WordPress single post edit page

---

#### Step-by-Step Flow:

**Step 4.1: WordPress Frontend (JavaScript)**

**What WordPress Does:**

- User clicks "Optimize Article" button in editor toolbar (next to Save draft button)
- JavaScript sends AJAX request to WordPress backend

```javascript
// WordPress Plugin - assets/js/settings.js
// User clicks button → AJAX call triggered
$.ajax({
  url: ajaxurl,
  type: "POST",
  data: {
    action: "ml_optimize_article",
    nonce: ajax_var.optimize_article_nonce,
    post_id: postId,
  },
});
```

---

**Step 4.2: WordPress Backend (PHP) - Collect Post Data**

**What WordPress Does:**

- WordPress plugin gets existing post
- WordPress plugin collects post data (title, content, excerpt, categories, tags)

```php
// WordPress Plugin - classes/Admin/SettingsForm.php
// Get post
$post = get_post($post_id);

// Collect post data
$post_data = [
    'post_id' => $post_id,
    'title' => $post->post_title,
    'content' => $post->post_content,
    'excerpt' => $post->post_excerpt,
    'categories' => wp_get_post_categories($post_id, ['fields' => 'names']),
    'tags' => wp_get_post_tags($post_id, ['fields' => 'names']),
];
```

---

**Step 4.3: WordPress → SaaS App API Call**

**What WordPress Does:**

- WordPress plugin sends HTTP POST request to SaaS app

**SaaS App Endpoint:** `POST /api/v1/optimize-article`

**Full URL:**

```
POST http://localhost:3000/api/v1/optimize-article
POST https://your-saas-app.com/api/v1/optimize-article  (Production)
```

**Request Headers:**

```
Content-Type: application/json
```

**Request Body (WordPress sends to SaaS App):**

```json
{
  "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
  "post_data": {
    "post_id": 123,
    "title": "Original Article Title",
    "content": "<p>Original content...</p>",
    "excerpt": "Original excerpt",
    "categories": ["Technology"],
    "tags": ["Tips"]
  }
}
```

---

**Step 4.4: SaaS App Processing**

**What SaaS App Does:**

1. Receives post data from WordPress
2. Validates API key
3. Optimizes article content:
   - Improves title (adds power words if not present)
   - Enhances content structure (adds headings if missing, adds more paragraphs if short)
   - Improves excerpt (generates better excerpt if missing or too short)
   - Generates improved SEO meta data (title, description, keywords)
4. Returns optimized content

```javascript
// SaaS App - server.js
app.post("/api/v1/optimize-article", async (req, res) => {
  const { api_key, post_data } = req.body;

  // Find site by API key
  const site = await WordPressSite.findOne({ api_key });

  // Optimize article content
  const optimized = optimizeArticleContent(post_data);
  // Result: {
  //   title: "Ultimate Original Article Title",
  //   content: "<h2>Introduction</h2><p>Improved content...</p>",
  //   excerpt: "Discover the best practices...",
  //   seo: {
  //     title: "Ultimate Original Article Title",
  //     description: "Discover the best practices...",
  //     keywords: "Technology, Tips"
  //   }
  // }

  res.json({
    success: true,
    message: "Article optimized successfully",
    optimized: optimized,
  });
});
```

---

**Step 4.5: SaaS App Response**

**SaaS App Response (Success):**

```json
{
  "success": true,
  "message": "Article optimized successfully",
  "optimized": {
    "title": "Ultimate Original Article Title",
    "content": "<h2>Introduction</h2><p>Improved content...</p>",
    "excerpt": "Discover the best practices and expert insights about Original Article Title.",
    "seo": {
      "title": "Ultimate Original Article Title",
      "description": "Discover the best practices and expert insights about Original Article Title. Read more to discover actionable insights and strategies.",
      "keywords": "Technology, Tips"
    }
  }
}
```

**HTTP Status Code:** `200 OK`

---

**Step 4.6: WordPress Updates Post**

**What WordPress Does:**

- WordPress plugin receives optimized content from SaaS app
- WordPress plugin updates post with optimized content
- WordPress plugin updates SEO meta tags (Yoast/Rank Math)

```php
// WordPress Plugin - classes/Admin/SettingsForm.php
// Update post with optimized content
wp_update_post([
    'ID' => $post_id,
    'post_title' => sanitize_text_field($optimized['title']),
    'post_content' => wp_slash($optimized['content']),
    'post_excerpt' => sanitize_textarea_field($optimized['excerpt']),
]);

// Update SEO meta tags (Yoast SEO)
if (is_plugin_active('wordpress-seo/wp-seo.php')) {
    update_post_meta($post_id, '_yoast_wpseo_title', sanitize_text_field($optimized['seo']['title']));
    update_post_meta($post_id, '_yoast_wpseo_metadesc', sanitize_textarea_field($optimized['seo']['description']));
}

// Update SEO meta tags (Rank Math SEO)
if (is_plugin_active('seo-by-rank-math/rank-math.php')) {
    update_post_meta($post_id, 'rank_math_title', sanitize_text_field($optimized['seo']['title']));
    update_post_meta($post_id, 'rank_math_description', sanitize_textarea_field($optimized['seo']['description']));
}
```

**What Gets Updated in WordPress Database:**

- **wp_posts table**: `post_title`, `post_content`, `post_excerpt` fields updated
- **wp_postmeta table**:
  - `_yoast_wpseo_title` meta key updated (if Yoast SEO installed)
  - `_yoast_wpseo_metadesc` meta key updated (if Yoast SEO installed)
  - `rank_math_title` meta key updated (if Rank Math SEO installed)
  - `rank_math_description` meta key updated (if Rank Math SEO installed)

---

**Final Result:** ✅ Article optimized and updated!

**Summary:**

- WordPress collects existing post data
- WordPress sends post data to SaaS app
- SaaS app optimizes content (title, content, excerpt, SEO)
- SaaS app returns optimized content
- WordPress updates post with optimized content
- WordPress updates SEO meta tags (if SEO plugin installed)

---

## WordPress Plugin Integration Flow

### Step 1: WordPress Sends Connection Request

WordPress plugin calls your SaaS `/connect` endpoint:

```php
// WordPress Plugin - classes/API/Client.php

public function connect() {
    $user = wp_get_current_user();
    $secret = bin2hex(random_bytes(32)); // Generate secret

    $data = [
        'name' => $user->user_email,
        'secret' => $secret,
        'webhook' => admin_url('admin-ajax.php?action=marketlytics-notify'),
        'rest' => rest_url('marketlytics/v1'),
        'success_url' => admin_url('options-general.php?page=marketlytics-setting&connected=1'),
        'failure_url' => admin_url('options-general.php?page=marketlytics-setting&error=1')
    ];

    $response = $this->request('connect', $data, 'POST');

    if ($response && $response['status'] == 1) {
        // Save API key and secret
        update_option('content_generator_api_key', $response['api_key']);
        update_option('content_generator_secret', $secret);
        return $response['api_key'];
    }

    return false;
}
```

### Step 2: SaaS Sends Blog Posts to WordPress

When SaaS wants to send blog posts, it calls WordPress REST API:

**WordPress REST Endpoint:** `/wp-json/marketlytics/v1/bulk-posts`

**Request Body:**

```json
{
  "posts": [
    {
      "title": "Complete Guide to Technology",
      "content": "<h2>Introduction</h2><p>Welcome to our guide...</p><figure class=\"wp-block-image\"><img src=\"https://picsum.photos/1200/600?random=123\" alt=\"Technology Test Image\" /></figure>",
      "excerpt": "Discover the best practices...",
      "status": "draft",
      "date": "2025-11-04T12:00:00.000Z",
      "categories": ["Technology", "Business"],
      "tags": ["Tips", "Guide", "Tutorial"],
      "featured_image": "https://picsum.photos/1920/1080?random=123",
      "meta": {
        "_cg_generated": true,
        "_cg_generated_at": "2025-11-04T12:00:00.000Z"
      },
      "seo": {
        "title": "Complete Guide to Technology | Expert Guide",
        "description": "Discover the best practices...",
        "keywords": "Technology, Business, Tips"
      }
    }
  ],
  "sign": "hmac_signature_here"
}
```

**WordPress Response:**

```json
{
  "success": true,
  "results": {
    "success": 20,
    "failed": 0,
    "post_ids": [1263, 1265, 1267, ...],
    "errors": [],
    "image_stats": {
      "total_images_processed": 20,
      "content_images": 20,
      "featured_images": 20,
      "images_failed": 0
    }
  }
}
```

### Step 3: WordPress Plugin Processes Posts

```php
// WordPress Plugin - marketlytics.php (Main plugin file)

public function restBulkPosts($request) {
    $post = $request->get_json_params();
    $settings = $this->getSettings();
    $client = $this->getAPIClient();

    // Verify signature
    if (!isset($settings['secret']) || !isset($post['sign']) || !$client->checkSign($post, $settings['secret'])) {
        return new WP_REST_Response([
            'success' => false,
            'error' => 'Invalid signature'
        ], 401);
    }

    // Get posts data
    if (!isset($post['posts']) || !is_array($post['posts'])) {
        return new WP_REST_Response([
            'success' => false,
            'error' => 'Posts data is required'
        ], 400);
    }

    // Load blog handler
    require_once($this->plugin_path . 'classes/Blog/Handler.php');
    $blogHandler = new \ContentGenerator\Blog\Handler($this);

    // Create bulk posts
    $results = $blogHandler->createBulkPosts($post['posts']);

    // Log the operation
    require_once($this->plugin_path . 'classes/log-handler.php');
    $logHandler = new \ContentGenerator\LogHandler($this);
    $totalSent = isset($post['posts']) ? count($post['posts']) : 0;
    $logHandler->logBulkPosts($results, $totalSent);

    return new WP_REST_Response([
        'success' => true,
        'results' => $results
    ], 200);
}
```

---

## Security

### 1. HMAC Signature

All data exchanges are signed using HMAC-SHA256:

```javascript
// Generate signature
function getSignature(data, secret) {
  const message = JSON.stringify(data);
  return crypto.createHmac("sha256", secret).update(message).digest("hex");
}

// Verify signature
function checkSign(data, secret) {
  const receivedSign = data.sign;
  delete data.sign;
  const expectedSign = getSignature(data, secret);
  data.sign = receivedSign;
  return receivedSign === expectedSign;
}
```

### 2. Secret Key Management

- Secret generated by WordPress plugin
- Stored securely in WordPress options
- Never exposed in responses
- Used only for signature verification

### 3. HTTPS Required

- All production endpoints must use HTTPS
- Prevents man-in-the-middle attacks
- Protects API keys and secrets in transit

---

## Error Handling

### HTTP Status Codes

- `200` - Success
- `400` - Bad Request (missing parameters)
- `401` - Unauthorized (invalid signature)
- `404` - Not Found (site not found)
- `500` - Internal Server Error

### Error Response Format

```json
{
  "status": 0,
  "error": "Descriptive error message"
}
```

### Common Errors

| Error                               | Cause                          | Solution                                          |
| ----------------------------------- | ------------------------------ | ------------------------------------------------- |
| "API key is required"               | Missing api_key in request     | Include api_key in request body                   |
| "Site not found"                    | Invalid or deleted API key     | Reconnect WordPress site                          |
| "Invalid signature"                 | HMAC signature mismatch        | Check secret key, ensure JSON format is identical |
| "Site is not connected"             | Site status is disconnected    | Reconnect from WordPress plugin                   |
| "Failed to send blogs to WordPress" | WordPress REST API unreachable | Check WordPress URL, firewall, REST API enabled   |

---

## Testing

### 1. Test Connection

```bash
curl -X POST https://your-saas-app.com/api/v1/connect \
  -H "Content-Type: application/json" \
  -d '{
    "name": "admin@example.com",
    "secret": "test_secret_key_12345",
    "webhook": "https://example.com/wp-admin/admin-ajax.php?action=marketlytics-notify",
    "rest": "https://example.com/wp-json/marketlytics/v1"
  }'
```

Expected Response:

```json
{
  "status": 1,
  "api_key": "generated_api_key_here",
  "message": "Connected successfully"
}
```

### 2. Test Blog Generation

```bash
curl -X POST https://your-saas-app.com/api/v1/generate-blogs \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "695f5ac984a3f7d2e8b1c9d4e6f8a0b2",
    "count": 5
  }'
```

### 3. Test Signature Verification

```javascript
// Node.js test
const crypto = require("crypto");

const testData = {
  posts: [{ title: "Test" }],
};

const secret = "test_secret_key";
const signature = crypto
  .createHmac("sha256", secret)
  .update(JSON.stringify(testData))
  .digest("hex");

console.log("Signature:", signature);

testData.sign = signature;
// Send testData to WordPress
```

---

## Database Schema

### WordPress Sites Collection (MongoDB)

```javascript
const wordPressSiteSchema = new mongoose.Schema({
  site_url: { type: String, required: true, unique: true },
  user_email: { type: String, required: true },
  api_key: { type: String, required: true, unique: true },
  secret: { type: String, required: true },
  webhook_url: { type: String, required: true },
  rest_url: String,
  status: {
    type: String,
    enum: ["connected", "disconnected"],
    default: "connected",
  },
  created_at: { type: Date, default: Date.now },
  updated_at: { type: Date, default: Date.now },
});
```

---

## Blog Post Data Structure

### Required Fields

```javascript
{
  "title": String,        // Post title (required)
  "content": String,      // Post content HTML (required)
  "excerpt": String,      // Post excerpt
  "status": String,       // "draft" or "publish"
  "date": String,         // ISO 8601 format
  "categories": Array,    // Array of category names
  "tags": Array,          // Array of tag names
  "featured_image": String, // Image URL (optional)
  "meta": Object,         // Custom meta fields
  "seo": Object          // SEO meta data (REQUIRES SEO PLUGIN - See below)
}
```

### SEO Plugin Requirement

**IMPORTANT:** To use SEO meta tags, WordPress site **MUST** have one of these SEO plugins installed:

1. **Yoast SEO** (Free or Premium)

   - Plugin: `wordpress-seo/wp-seo.php` or `wordpress-seo-premium/wp-seo-premium.php`
   - Download: https://wordpress.org/plugins/wordpress-seo/

2. **Rank Math SEO**
   - Plugin: `seo-by-rank-math/rank-math.php`
   - Download: https://wordpress.org/plugins/seo-by-rank-math/

**How It Works:**

- WordPress plugin automatically detects which SEO plugin is installed
- If Yoast SEO is installed → assigns meta tags to Yoast SEO fields
- If Rank Math SEO is installed → assigns meta tags to Rank Math SEO fields
- If no SEO plugin is installed → SEO meta tags are ignored (no error)

**What Gets Assigned:**

**For Yoast SEO:**

- `_yoast_wpseo_title` → SEO Title
- `_yoast_wpseo_metadesc` → Meta Description
- `_yoast_wpseo_focuskw` → Focus Keyword
- `_yoast_wpseo_focuskeywords` → Additional Focus Keywords (if array provided)

**For Rank Math SEO:**

- `rank_math_title` → SEO Title
- `rank_math_description` → Meta Description
- `rank_math_focus_keyword` → Focus Keyword

### Example Post Object

```json
{
  "title": "Complete Guide to Technology",
  "content": "<h2>Introduction</h2><p>Welcome to our comprehensive guide...</p><figure class=\"wp-block-image\"><img src=\"https://picsum.photos/1200/600?random=123\" alt=\"Technology\" /></figure><p>More content here...</p>",
  "excerpt": "Discover the best practices and expert tips for Technology.",
  "status": "draft",
  "date": "2025-11-04T10:30:00.000Z",
  "categories": ["Technology", "Business"],
  "tags": ["Tips", "Guide", "Tutorial", "Expert"],
  "featured_image": "https://picsum.photos/1920/1080?random=789",
  "meta": {
    "_cg_generated": true,
    "_cg_generated_at": "2025-11-04T10:30:00.000Z",
    "_cg_batch_id": "batch_123"
  },
  "seo": {
    "title": "Complete Guide to Technology | Expert Guide",
    "description": "Discover the best practices and expert tips for Technology. Learn everything you need to know in this comprehensive guide.",
    "keywords": "Technology, Business, Tips, Guide"
  }
}
```

**Note:** The `seo` object will only be processed if Yoast SEO or Rank Math SEO plugin is installed in WordPress. If no SEO plugin is detected, SEO meta tags will be ignored (no error).

---

## Rate Limiting (Optional)

```javascript
const rateLimit = require("express-rate-limit");

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    status: 0,
    error: "Too many requests, please try again later.",
  },
});

app.use("/api/v1/", apiLimiter);
```

---

## Logging

```javascript
// Log all API requests
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Log errors
app.use((err, req, res, next) => {
  console.error("Error:", err);
  res.status(500).json({
    status: 0,
    error: "Internal server error",
  });
});
```

---

## Environment Variables

```env
# Server
PORT=3000
NODE_ENV=production

# Database
MONGODB_URI=mongodb://localhost:27017/content_generator

# Security
API_SECRET_KEY=your_secret_key_here

# URLs
FRONTEND_URL=https://your-frontend.com
API_BASE_URL=https://your-saas-app.com/api/v1
```

---

## Quick Start Checklist

- [ ] Setup MongoDB database
- [ ] Install dependencies (`npm install`)
- [ ] Create `.env` file with environment variables
- [ ] Implement `/connect` endpoint
- [ ] Implement `/disconnect` endpoint
- [ ] Implement `/generate-blogs` endpoint
- [ ] Add HMAC signature generation/verification
- [ ] Test connection with WordPress plugin
- [ ] Test blog post generation
- [ ] Add error handling
- [ ] Add logging
- [ ] Deploy to production
- [ ] Enable HTTPS
- [ ] Add rate limiting (optional)
- [ ] Monitor logs and errors

---

## Support & Resources

- **WordPress Plugin Repository**: `/marketlytics/`
- **Plugin Name**: Marketlytics
- **SaaS App Repository**: `/saas-app/`
- **Test Environment**: `http://localhost:3000`
- **WordPress Test Site**: `http://localhost/wpdev`

---

## Version History

- **v1.0.0** - Initial release with connect, disconnect, and blog generation endpoints
- **v1.1.0** - Added image handling with Picsum.photos
- **v1.2.0** - Added detailed logging and error handling
- **v1.3.0** - Added statistics tracking for images
- **v1.4.0** - Removed WooCommerce product content generation (blog-focused plugin)
- **v1.5.0** - Added article generation and optimization endpoints
- **v1.5.1** - Featured images now generated for bulk articles; Generate New Article button positioned after Add Post button
- **v1.6.0** - Plugin renamed to Marketlytics; Reorganized folder structure; Removed WooCommerce integration; Updated CSS classes with ml- prefix

---

**Last Updated**: November 4, 2025
**API Version**: 1.6.0
**Plugin Name**: Marketlytics
**Plugin Structure**:

- `classes/API/Client.php` - API communication
- `classes/Blog/Handler.php` - Blog post handling
- `classes/Admin/SettingsForm.php` - Admin settings
- `templates/admin/settings/` - Settings templates
- `templates/admin/popup/` - Popup templates
