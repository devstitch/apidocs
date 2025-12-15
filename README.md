# Marketlytics WordPress Plugin - API Documentation

## Base URL

```
https://clouddev.marketlytics.co.uk/back/
```

## Variables

| Variable             | Description                                     | Example               |
| -------------------- | ----------------------------------------------- | --------------------- |
| {wordpress_site_url} | Your WordPress site URL (from `get_site_url()`) | `https://example.com` |

---

## 1. Connect Integration

Connects a WordPress site to Marketlytics.

### Endpoint

```
POST /integrations/connect
```

### Request Headers

```
Content-Type: application/json
```

### Request Body

```json
{
  "api_key": "your_api_key_here",
  "platform": "wordpress",
  "webhook": "{wordpress_site_url}/wp-admin/admin-ajax.php?action=marketlytics-notify",
  "rest": "{wordpress_site_url}/wp-json/marketlytics/v1"
}
```

### Parameters

| Field    | Type   | Required | Description                         |
| -------- | ------ | -------- | ----------------------------------- |
| api_key  | string | Yes      | API key from Marketlytics dashboard |
| platform | string | Yes      | Always "wordpress"                  |
| webhook  | string | Yes      | WordPress AJAX webhook URL          |
| rest     | string | Yes      | WordPress REST API base URL         |

### Success Response

```json
{
  "success": true,
  "message": "Connected successfully"
}
```

### Error Response

```json
{
  "success": false,
  "error": "Error message here"
}
```

---

## 2. Disconnect Integration

Disconnects a WordPress site from Marketlytics.

### Endpoint

```
POST /integrations/disconnect
```

### Request Headers

```
Content-Type: application/json
```

### Request Body

```json
{
  "api_key": "your_api_key_here"
}
```

### Success Response

```json
{
  "success": true,
  "message": "Disconnected successfully"
}
```

---

## 3. Bulk Posts - Insert Blog Posts to WordPress

SaaS app sends blog posts to connected WordPress site.

### Endpoint (WordPress REST API)

```
POST {wordpress_site_url}/wp-json/marketlytics/v1/bulk-posts
```

### Request Headers

```
Content-Type: application/json
X-API-Key: your_api_key_here
```

### Request Body

```json
{
  "posts": [
    {
      "title": "Blog Post Title",
      "content": "<p>HTML content of the blog post...</p>",
      "excerpt": "Short description of the post",
      "status": "publish",
      "categories": ["Category 1", "Category 2"],
      "tags": ["tag1", "tag2", "tag3"],
      "featured_image": "https://example.com/image.jpg",
      "seo": {
        "title": "SEO Title",
        "description": "SEO meta description",
        "keywords": ["keyword1", "keyword2"]
      }
    }
  ]
}
```

### Post Object Parameters

| Field           | Type   | Required | Description                                                              |
| --------------- | ------ | -------- | ------------------------------------------------------------------------ |
| title           | string | Yes      | Post title                                                               |
| content         | string | Yes      | Post content (HTML allowed)                                              |
| excerpt         | string | No       | Post excerpt/summary                                                     |
| status          | string | No       | Post status: "publish", "draft", "pending", "private" (default: "draft") |
| categories      | array  | No       | Array of category names (created if not exist)                           |
| tags            | array  | No       | Array of tag names                                                       |
| featured_image  | string | No       | URL of featured image (downloaded and set)                               |
| seo             | object | No       | SEO metadata object                                                      |
| seo.title       | string | No       | SEO title (Yoast/RankMath)                                               |
| seo.description | string | No       | SEO meta description                                                     |
| seo.keywords    | array  | No       | Focus keywords                                                           |

### Success Response

```json
{
  "success": true,
  "results": {
    "success": 5,
    "failed": 0,
    "post_ids": [123, 124, 125, 126, 127],
    "errors": [],
    "image_stats": {
      "total_images_processed": 10,
      "content_images": 8,
      "featured_images": 5,
      "images_failed": 0
    }
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": "Invalid or missing API key"
}
```

### Authentication

- API key must be sent in the `X-API-Key` header
- The API key must match the one used during connection
- Only the connected site can receive posts

---

## WordPress Plugin Database Settings

### Option Key

```
marketlytics_settings
```

### Stored Data

```php
[
    'user_id' => 1,           // WordPress user ID who connected
    'name' => 'user@email.com', // User email
    'api_key' => 'xxx...'     // API key for authentication
]
```

---

## Complete Flow Example

### 1. User Connects WordPress to Marketlytics

```bash
curl -X POST https://clouddev.marketlytics.co.uk/back/integrations/connect \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "abc123...",
    "platform": "wordpress",
    "webhook": "{wordpress_site_url}/wp-admin/admin-ajax.php?action=marketlytics-notify",
    "rest": "{wordpress_site_url}/wp-json/marketlytics/v1"
  }'
```

### 2. SaaS Sends Blog Posts to WordPress

```bash
curl -X POST {wordpress_site_url}/wp-json/marketlytics/v1/bulk-posts \
  -H "Content-Type: application/json" \
  -H "X-API-Key: abc123..." \
  -d '{
    "posts": [
      {
        "title": "My First AI Article",
        "content": "<p>Article content here...</p>",
        "status": "publish",
        "categories": ["AI", "Technology"],
        "featured_image": "https://example.com/image.jpg"
      }
    ]
  }'
```

### 3. User Disconnects

```bash
curl -X POST https://clouddev.marketlytics.co.uk/back/integrations/disconnect \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "abc123..."
  }'
```

---

## Error Codes

| HTTP Code | Description                               |
| --------- | ----------------------------------------- |
| 200       | Success                                   |
| 400       | Bad Request - Invalid parameters          |
| 401       | Unauthorized - Invalid or missing API key |
| 500       | Server Error                              |
