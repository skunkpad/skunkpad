# SkunkPad

Welcome To SkunkPad!

Checkout the site at [https://skunkpad.com](https://skunkpad.com)
![https://skunkpad.com/images/og/index.png](https://skunkpad.com/images/og/index.png)

We haven't decided to open-source the source code for the site just yet, but let us know if you are interested and we will consider it...
We ARE considering options for self-hosting, so let us know if you have interest in that.
The purposed of this public repo is to
- provides comprehensive instructions for how to expore SkunkPad and interact with the public API
- document bugs
- respond to users

## Table of Contents

- [Overview](#overview)
- [Site Exploration](#site-exploration)
- [Public API](#public-api)
- [Finding Users](#finding-users)
- [Finding Public Ideas (Tiles)](#finding-public-ideas-tiles)
- [Best Practices](#best-practices)
- [Rate Limits](#rate-limits)
- [Authentication](#authentication)

---

## Overview

**SkunkPad** is a platform for sharing and discovering ideas. The site is designed to be crawlable and accessible, with both web pages and a public API.

- **Main Site**: https://skunkpad.com
- **API Base**: https://api.skunkpad.com
- **API Documentation**: https://api.skunkpad.com/docs
- **OpenAPI Spec**: https://api.skunkpad.com/openapi.json

---

## Site Exploration

### Sitemaps

SkunkPad provides comprehensive sitemaps for efficient crawling:

1. **Sitemap Index**: https://skunkpad.com/sitemap-index.xml
   - Lists all available sitemaps
   - Updated regularly

2. **Static Sitemap**: https://skunkpad.com/sitemaps/static.xml
   - Contains fixed pages (homepage, about, terms, privacy, etc.)

3. **User Sitemaps**: https://skunkpad.com/sitemaps/users-{chunk}.xml
   - Contains user profile pages
   - Chunked into files of 50,000 users each
   - Format: `/username`

4. **Idea Sitemaps**: https://skunkpad.com/sitemaps/ideas-{chunk}.xml
   - Contains public idea pages
   - Chunked into files of 50,000 ideas each
   - Format: `/username/idea-name/overview`

### URL Structure

- **User Profile**: `https://skunkpad.com/{username}`
- **Idea Page**: `https://skunkpad.com/{username}/{idea-name}/{page}`
  - Pages include: `overview`, `details`, `updates`, `comments`, etc.
- **Public Ideas Feed**: `https://skunkpad.com/public/ideas`
- **Following Feed**: `https://skunkpad.com/following/ideas`

---

## Public API

### Base URL

All API requests should use the base URL:
```
https://api.skunkpad.com
```

For local development:
```
http://localhost:3002
```

### Request Headers

AI agents **should** include identifying headers:

```http
User-Agent: [AI-System-Name]/[Version] (AI Assistant)
X-AI-Agent: [AI-System-Identifier]
X-AI-Purpose: [Brief description of use case]
X-AI-Contact: [Contact information] (optional)
```

**Example:**
```http
User-Agent: ClaudeBot/1.0 (AI Assistant)
X-AI-Agent: anthropic-claude
X-AI-Purpose: Research and content discovery
X-AI-Contact: support@anthropic.com
```

### Response Format

All API responses are in JSON format. Paginated endpoints return:

```json
{
  "items": [...],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 1234
  }
}
```

---

## Finding Users

### 1. Get User Count

Get the total number of users on the platform:

```http
GET /api/user-count
```

**Query Parameters:**
- `search` (optional): Search term to filter users

**Response:**
```json
1234
```

**Example:**
```bash
curl "https://api.skunkpad.com/api/user-count"
```

---

### 2. Search/List Users

Retrieve a paginated list of users:

```http
GET /api/users
```

**Query Parameters:**
- `search` (optional): Search term (matches username, name, bio, etc.)
- `limit` (optional): Number of results per page (default: 50, max: 100)
- `offset` (optional): Offset for pagination (default: 0)

**Response:**
```json
[
  {
    "username": "johndoe",
    "name": "John Doe",
    "preferredName": "John",
    "bio": "Software engineer and idea enthusiast",
    "photo": "assets/profile-photos/johndoe.jpg",
    "jobTitle": "Senior Developer",
    "skills": ["JavaScript", "React", "Node.js"],
    "interests": ["AI", "Web Development", "Open Source"],
    "createdAt": "2024-01-15T10:30:00Z",
    "ideaCount": 42,
    "followerCount": 156,
    "followingCount": 89
  }
]
```

**Example:**
```bash
# Get first 50 users
curl "https://api.skunkpad.com/api/users?limit=50&offset=0"

# Search for users matching "engineer"
curl "https://api.skunkpad.com/api/users?search=engineer&limit=20"

# Pagination: Get next page
curl "https://api.skunkpad.com/api/users?limit=50&offset=50"
```

---

### 3. Get User Profile

Get detailed information about a specific user:

```http
GET /api/users/:username
```

**Path Parameters:**
- `username`: The user's username

**Query Parameters:**
- `field` (optional): Specific field to retrieve (e.g., "bio", "skills")

**Response:**
```json
{
  "username": "johndoe",
  "name": "John Doe",
  "preferredName": "John",
  "bio": "Software engineer and idea enthusiast",
  "photo": "assets/profile-photos/johndoe.jpg",
  "jobTitle": "Senior Developer",
  "skills": ["JavaScript", "React", "Node.js"],
  "interests": ["AI", "Web Development", "Open Source"],
  "socialLinks": [
    {
      "platform": "github",
      "url": "https://github.com/johndoe",
      "label": "GitHub"
    }
  ],
  "createdAt": "2024-01-15T10:30:00Z",
  "ideaCount": 42,
  "followerCount": 156,
  "followingCount": 89
}
```

**Example:**
```bash
# Get full profile
curl "https://api.skunkpad.com/api/users/johndoe"

# Get specific field
curl "https://api.skunkpad.com/api/users/johndoe?field=bio"
```

---

## Finding Public Ideas (Tiles)

Ideas on SkunkPad are called "tiles" internally. Public tiles are ideas that are visible to everyone.

### 1. Get Public Idea Count

Get the total number of public ideas:

```http
GET /api/tiles/public-count
```

**Query Parameters:**
- `search` (optional): Search term to filter ideas

**Response:**
```json
5678
```

**Example:**
```bash
curl "https://api.skunkpad.com/api/tiles/public-count"
```

---

### 2. List Public Ideas

Retrieve a paginated list of public ideas:

```http
GET /api/tiles/public
```

**Query Parameters:**
- `search` (optional): Search term (matches idea name, description, tags, username, etc.)
- `limit` (optional): Number of results per page (default: 50)
- `offset` (optional): Offset for pagination (default: 0)
- `field` (optional): Specific fields to retrieve (can be array or comma-separated)

**Response:**
```json
{
  "items": [
    {
      "id": "abc123",
      "username": "johndoe",
      "name": "my-awesome-app",
      "displayName": "My Awesome App",
      "shortDescription": "A revolutionary app for managing tasks",
      "tags": ["productivity", "mobile", "app"],
      "createdAt": "2024-03-20T14:30:00Z",
      "updatedAt": "2024-03-22T09:15:00Z",
      "viewCount": 1234,
      "commentCount": 45,
      "followerCount": 89,
      "rankingScore": 156.7,
      "ownerName": "John Doe",
      "ownerPreferredName": "John",
      "ownerPhoto": "assets/profile-photos/johndoe.jpg"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 5678
  }
}
```

**Field Descriptions:**
- `id`: Unique identifier for the idea
- `username`: Owner's username
- `name`: URL-safe idea name (slug)
- `displayName`: Human-readable idea name
- `shortDescription`: Brief description of the idea
- `tags`: Array of tags associated with the idea
- `rankingScore`: Popularity/relevance score (higher = more popular)
- `viewCount`, `commentCount`, `followerCount`: Engagement metrics

**Example:**
```bash
# Get first 50 public ideas
curl "https://api.skunkpad.com/api/tiles/public?limit=50&offset=0"

# Search for ideas about "AI"
curl "https://api.skunkpad.com/api/tiles/public?search=AI&limit=20"

# Get specific fields only
curl "https://api.skunkpad.com/api/tiles/public?field=name&field=shortDescription&limit=10"

# Pagination: Get next page
curl "https://api.skunkpad.com/api/tiles/public?limit=50&offset=50"
```

---

### 3. Get User's Ideas

Get all ideas (tiles) from a specific user:

```http
GET /api/tiles/{username}
```

**Path Parameters:**
- `username`: The user's username

**Query Parameters:**
- `limit` (optional): Number of results per page
- `offset` (optional): Offset for pagination

**Response:** Same format as public tiles

**Example:**
```bash
curl "https://api.skunkpad.com/api/tiles/johndoe?limit=20"
```

---

### 4. Get Individual Idea Details

To get full details about a specific idea, use:

```http
GET /api/ideas/{username}/{ideaName}
```

**Note:** This endpoint may require authentication depending on the idea's access level.

---

## Best Practices

### 1. Be a Good Citizen

- **Respect robots.txt**: The main site allows crawling, but the API (`api.skunkpad.com`) disallows all crawlers in robots.txt. Use the API programmatically instead.
- **Use appropriate headers**: Always identify your bot with proper User-Agent and X-AI-Agent headers
- **Honor rate limits**: Don't overwhelm the server with requests
- **Cache responses**: Cache data when appropriate to reduce load

### 2. Efficient Crawling

- **Use sitemaps**: Start with the sitemap index to discover all content systematically
- **Pagination**: Use the `limit` and `offset` parameters to page through results
  ```bash
  # Page 1
  curl "https://api.skunkpad.com/api/tiles/public?limit=100&offset=0"
  # Page 2
  curl "https://api.skunkpad.com/api/tiles/public?limit=100&offset=100"
  # Page 3
  curl "https://api.skunkpad.com/api/tiles/public?limit=100&offset=200"
  ```
- **Use counts**: Check total counts first to plan your crawl
  ```bash
  TOTAL=$(curl -s "https://api.skunkpad.com/api/tiles/public-count")
  PAGES=$((TOTAL / 100 + 1))
  ```

### 3. Search vs. Full Listing

- **Search** is best for targeted queries:
  ```bash
  curl "https://api.skunkpad.com/api/tiles/public?search=machine+learning"
  ```
- **Full listing** (no search param) gives you everything:
  ```bash
  curl "https://api.skunkpad.com/api/tiles/public?limit=100"
  ```

### 4. Field Selection

To reduce payload size, request only the fields you need:

```bash
# Just names and descriptions
curl "https://api.skunkpad.com/api/tiles/public?field=name&field=shortDescription"
```

---

## Rate Limits

### Public Endpoints (No Authentication)

Public endpoints have rate limits to prevent abuse:

- **Users endpoints**: ~100 requests/minute per IP
- **Public tiles endpoints**: ~100 requests/minute per IP
- **Count endpoints**: ~500 requests/minute per IP

### Authenticated Endpoints

For higher rate limits and access to additional features, consider registering an API account.

### Rate Limit Headers

Response headers include rate limit information:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```

### What to Do When Rate Limited

If you receive a `429 Too Many Requests` response:

1. **Back off**: Wait before retrying
2. **Use exponential backoff**: Double wait time with each retry
3. **Respect Retry-After header**: If provided, wait that many seconds
4. **Consider authentication**: Authenticated requests have higher limits

---

## Authentication

### When Do You Need It?

Authentication is **optional** for:
- Viewing public ideas (`/api/tiles/public`)
- Viewing user profiles (`/api/users/*`)
- Searching users and ideas

Authentication is **required** for:
- Viewing following feed (`/api/tiles/following`)
- Creating/editing ideas
- Posting comments
- Following users
- Most write operations

### How to Authenticate

#### Option 1: AI Bot Account (Recommended)

Register an AI bot account:

1. Visit https://skunkpad.com/ai-bot-signup
2. Create an account with a username containing "bot" or "ai" (e.g., "mycompany-bot")
3. Generate an API key from your account settings
4. Include the API key in your requests:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.skunkpad.com/api/tiles/following"
```

**Bot Account Features:**
- Higher rate limits
- Read-only access to authenticated endpoints
- Cannot create ideas or upload images
- Clearly identified as automated

#### Option 2: Access Codes

For specific protected content, you may be given an access code:

```bash
curl "https://api.skunkpad.com/api/ideas/username/idea?code=ACCESS_CODE"
```

---

## Complete Examples

### Example 1: Find All Ideas About "AI"

```bash
#!/bin/bash

# Search for AI-related ideas
SEARCH="AI"
LIMIT=50
OFFSET=0

# Get total count
TOTAL=$(curl -s "https://api.skunkpad.com/api/tiles/public-count?search=$SEARCH")
echo "Found $TOTAL ideas about '$SEARCH'"

# Fetch all results
while [ $OFFSET -lt $TOTAL ]; do
  echo "Fetching ideas $OFFSET to $((OFFSET + LIMIT))..."

  curl -s "https://api.skunkpad.com/api/tiles/public?search=$SEARCH&limit=$LIMIT&offset=$OFFSET" \
    -H "User-Agent: MyBot/1.0 (AI Assistant)" \
    -H "X-AI-Agent: mybot" \
    | jq '.items[] | {name: .name, username: .username, description: .shortDescription}'

  OFFSET=$((OFFSET + LIMIT))

  # Be polite - add delay between requests
  sleep 1
done
```

### Example 2: Get All User Profiles

```bash
#!/bin/bash

# Get total user count
TOTAL=$(curl -s "https://api.skunkpad.com/api/user-count")
echo "Total users: $TOTAL"

LIMIT=100
OFFSET=0

# Fetch all users
while [ $OFFSET -lt $TOTAL ]; do
  echo "Fetching users $OFFSET to $((OFFSET + LIMIT))..."

  curl -s "https://api.skunkpad.com/api/users?limit=$LIMIT&offset=$OFFSET" \
    -H "User-Agent: MyBot/1.0 (AI Assistant)" \
    | jq '.[] | {username, name, bio, ideaCount}'

  OFFSET=$((OFFSET + LIMIT))
  sleep 1
done
```

### Example 3: Build a Local Index

```python
import requests
import time
import json

BASE_URL = "https://api.skunkpad.com"
HEADERS = {
    "User-Agent": "MyBot/1.0 (AI Assistant)",
    "X-AI-Agent": "mybot",
    "X-AI-Purpose": "Building local search index"
}

def fetch_all_ideas():
    """Fetch all public ideas and build a local index."""

    # Get total count
    count_response = requests.get(
        f"{BASE_URL}/api/tiles/public-count",
        headers=HEADERS
    )
    total = count_response.json()
    print(f"Total ideas: {total}")

    # Fetch all ideas
    ideas = []
    limit = 100
    offset = 0

    while offset < total:
        print(f"Fetching ideas {offset} to {offset + limit}...")

        response = requests.get(
            f"{BASE_URL}/api/tiles/public",
            params={"limit": limit, "offset": offset},
            headers=HEADERS
        )

        if response.status_code == 200:
            data = response.json()
            ideas.extend(data["items"])
            offset += limit

            # Be polite
            time.sleep(0.5)
        elif response.status_code == 429:
            print("Rate limited. Waiting 60 seconds...")
            time.sleep(60)
        else:
            print(f"Error: {response.status_code}")
            break

    # Save to file
    with open("skunkpad_ideas.json", "w") as f:
        json.dump(ideas, f, indent=2)

    print(f"Saved {len(ideas)} ideas to skunkpad_ideas.json")
    return ideas

if __name__ == "__main__":
    fetch_all_ideas()
```

---

## Support

### Questions or Issues?

- **API Documentation**: https://api.skunkpad.com/docs
- **OpenAPI Spec**: https://api.skunkpad.com/openapi.json
- **Contact Form**: https://skunkpad.com/contact
- **Report Issues**: Contact the SkunkPad team through the website

### Stay Updated

Check the API documentation regularly for updates and new features.

---

## Summary

**Quick Start Checklist:**

1. ✅ Use sitemaps for efficient crawling: https://skunkpad.com/sitemap-index.xml
2. ✅ Add proper headers (User-Agent, X-AI-Agent) to all requests
3. ✅ Start with count endpoints to plan your crawl
4. ✅ Use pagination (limit/offset) for large datasets
5. ✅ Search when you need specific content, full listing for everything
6. ✅ Respect rate limits (add delays between requests)
7. ✅ Consider registering an AI bot account for higher limits
8. ✅ Cache responses when appropriate

**Key Endpoints:**

```
GET /api/user-count           # Total users
GET /api/users                # List users
GET /api/users/:username      # User profile

GET /api/tiles/public-count   # Total public ideas
GET /api/tiles/public         # List public ideas
GET /api/tiles/:username      # User's ideas
```

Happy exploring! 🚀
