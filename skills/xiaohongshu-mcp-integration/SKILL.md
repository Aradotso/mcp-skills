---
name: xiaohongshu-mcp-integration
description: MCP server for Xiaohongshu (Little Red Book) - search, publish posts, manage interactions, and automate content operations
triggers:
  - integrate with xiaohongshu
  - publish to little red book
  - search xiaohongshu posts
  - automate xiaohongshu content
  - xiaohongshu mcp server
  - xiaohongshu api integration
  - post to xiaohongshu
  - manage xiaohongshu account
---

# xiaohongshu-mcp-integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill enables AI coding agents to integrate with Xiaohongshu (Little Red Book / 小红书) through an MCP server. It provides capabilities for login management, content publishing (text/image/video), searching, retrieving recommendations, managing interactions (like, favorite, comment), and accessing user profiles.

## What It Does

The `xiaohongshu-mcp` project is a Model Context Protocol (MCP) server that provides programmatic access to Xiaohongshu's platform. It uses browser automation to interact with the platform and exposes tools through the MCP interface.

**Core capabilities:**
- Login and session management
- Publish image posts (up to 20 characters title, 1000 characters content)
- Publish video posts (local files only)
- Search content by keywords
- Get recommendation feed
- Retrieve post details (including engagement metrics and comments)
- Post comments and reply to comments
- Like/unlike posts
- Favorite/unfavorite posts
- Get user profile information

**Important constraints:**
- Titles must be ≤20 characters
- Content must be ≤1000 characters
- Daily posting limit is ~50 posts per account
- Cannot log in on multiple web sessions simultaneously (mobile app is OK)
- Requires login before using most features

## Installation

### Method 1: Pre-compiled Binaries (Recommended)

Download from [GitHub Releases](https://github.com/xpzouying/xiaohongshu-mcp/releases):

**Main MCP Server:**
- macOS Apple Silicon: `xiaohongshu-mcp-darwin-arm64`
- macOS Intel: `xiaohongshu-mcp-darwin-amd64`
- Windows x64: `xiaohongshu-mcp-windows-amd64.exe`
- Linux x64: `xiaohongshu-mcp-linux-amd64`

**Login Tool:**
- macOS Apple Silicon: `xiaohongshu-login-darwin-arm64`
- macOS Intel: `xiaohongshu-login-darwin-amd64`
- Windows x64: `xiaohongshu-login-windows-amd64.exe`
- Linux x64: `xiaohongshu-login-linux-amd64`

```bash
# 1. First run the login tool
chmod +x xiaohongshu-login-darwin-arm64
./xiaohongshu-login-darwin-arm64

# 2. Start the MCP server
chmod +x xiaohongshu-mcp-darwin-arm64
./xiaohongshu-mcp-darwin-arm64
```

**Note:** First run downloads a headless browser (~150MB).

### Method 2: Docker

```bash
# Pull the image
docker pull xpzouying/xiaohongshu-mcp:latest

# Run the container
docker run -d \
  --name xiaohongshu-mcp \
  -p 8080:8080 \
  -v $(pwd)/data:/app/data \
  xpzouying/xiaohongshu-mcp:latest
```

### Method 3: Build from Source

```bash
# Clone the repository
git clone https://github.com/xpzouying/xiaohongshu-mcp.git
cd xiaohongshu-mcp

# Build
go build -o xiaohongshu-mcp ./cmd/mcp
go build -o xiaohongshu-login ./cmd/login

# Run
./xiaohongshu-login  # First time setup
./xiaohongshu-mcp    # Start server
```

## Configuration

The MCP server can be configured for Claude Desktop or other MCP clients:

**Claude Desktop Configuration** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "xiaohongshu": {
      "command": "/path/to/xiaohongshu-mcp",
      "args": [],
      "env": {}
    }
  }
}
```

**Environment Variables:**

- `XHS_DATA_DIR`: Directory for storing session data (default: `./data`)
- `XHS_PORT`: HTTP server port (default: `8080`)
- `XHS_DEBUG`: Enable debug logging (default: `false`)

## MCP Tools

### 1. Login Management

**xhs_login**: Open login page for manual authentication
```json
{
  "name": "xhs_login",
  "arguments": {}
}
```

**xhs_check_login_status**: Check if currently logged in
```json
{
  "name": "xhs_check_login_status",
  "arguments": {}
}
```

### 2. Content Publishing

**xhs_create_image_note**: Publish image post

```json
{
  "name": "xhs_create_image_note",
  "arguments": {
    "title": "美食分享",
    "desc": "今天做的菜真好吃！\n#美食 #家常菜",
    "images": [
      "/Users/username/Pictures/food1.jpg",
      "/Users/username/Pictures/food2.jpg"
    ],
    "post_time": "",
    "privacy": "public",
    "tags": ["美食", "家常菜"]
  }
}
```

**Image sources supported:**
- Local absolute paths (recommended): `/path/to/image.jpg`
- HTTP/HTTPS URLs: `https://example.com/image.jpg`

**xhs_create_video_note**: Publish video post

```json
{
  "name": "xhs_create_video_note",
  "arguments": {
    "title": "旅行Vlog",
    "desc": "记录美好时光\n#旅行 #Vlog",
    "video": "/Users/username/Videos/trip.mp4",
    "cover": "/Users/username/Videos/cover.jpg",
    "post_time": "",
    "privacy": "public",
    "tags": ["旅行", "Vlog"]
  }
}
```

**Note:** Only local video files are supported (no HTTP URLs).

### 3. Content Discovery

**xhs_search**: Search for content

```json
{
  "name": "xhs_search",
  "arguments": {
    "keyword": "美食",
    "page": 1,
    "page_size": 20,
    "sort": "general"
  }
}
```

**xhs_get_recommend_feeds**: Get recommendation feed

```json
{
  "name": "xhs_get_recommend_feeds",
  "arguments": {
    "page_size": 10
  }
}
```

**xhs_get_note_detail**: Get post details with engagement data

```json
{
  "name": "xhs_get_note_detail",
  "arguments": {
    "feed_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "xsec_token": "ABCdef123456..."
  }
}
```

**Note:** `feed_id` and `xsec_token` are obtained from search results or feed lists.

### 4. Interactions

**xhs_comment_note**: Post a comment

```json
{
  "name": "xhs_comment_note",
  "arguments": {
    "feed_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "xsec_token": "ABCdef123456...",
    "content": "写得真好！"
  }
}
```

**xhs_reply_comment**: Reply to a comment

```json
{
  "name": "xhs_reply_comment",
  "arguments": {
    "feed_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "xsec_token": "ABCdef123456...",
    "comment_id": "comment_123",
    "content": "谢谢你的支持！"
  }
}
```

**xhs_like_note**: Like a post

```json
{
  "name": "xhs_like_note",
  "arguments": {
    "feed_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "xsec_token": "ABCdef123456...",
    "unlike": false
  }
}
```

**xhs_favorite_note**: Favorite a post

```json
{
  "name": "xhs_favorite_note",
  "arguments": {
    "feed_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "xsec_token": "ABCdef123456...",
    "unfavorite": false
  }
}
```

### 5. User Profiles

**xhs_get_user_profile**: Get user profile and posts

```json
{
  "name": "xhs_get_user_profile",
  "arguments": {
    "user_id": "5f9e8d7c6b5a4e3d2c1b0a9",
    "xsec_token": "ABCdef123456..."
  }
}
```

## Code Examples

### Go: Programmatic Usage

```go
package main

import (
    "context"
    "fmt"
    "log"
    
    "github.com/xpzouying/xiaohongshu-mcp/pkg/xhs"
)

func main() {
    // Initialize client
    client, err := xhs.NewClient(xhs.Config{
        DataDir: "./data",
        Debug:   false,
    })
    if err != nil {
        log.Fatal(err)
    }
    defer client.Close()
    
    ctx := context.Background()
    
    // Check login status
    loggedIn, err := client.CheckLoginStatus(ctx)
    if err != nil {
        log.Fatal(err)
    }
    
    if !loggedIn {
        // Trigger login flow
        if err := client.Login(ctx); err != nil {
            log.Fatal(err)
        }
    }
    
    // Search for content
    results, err := client.Search(ctx, xhs.SearchParams{
        Keyword:  "美食",
        Page:     1,
        PageSize: 10,
    })
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Found %d results\n", len(results.Items))
    
    // Publish an image post
    note, err := client.CreateImageNote(ctx, xhs.ImageNoteParams{
        Title: "美食分享",
        Desc:  "今天做的菜真好吃！\n#美食 #家常菜",
        Images: []string{
            "/path/to/image1.jpg",
            "/path/to/image2.jpg",
        },
        Tags: []string{"美食", "家常菜"},
    })
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Published note: %s\n", note.ID)
    
    // Like a post
    if len(results.Items) > 0 {
        item := results.Items[0]
        err = client.LikeNote(ctx, xhs.LikeParams{
            FeedID:    item.FeedID,
            XsecToken: item.XsecToken,
        })
        if err != nil {
            log.Fatal(err)
        }
        fmt.Println("Liked post successfully")
    }
}
```

### Go: HTTP API Usage

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

func main() {
    baseURL := "http://localhost:8080"
    
    // Check login status
    resp, err := http.Get(baseURL + "/api/check_login_status")
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    var loginStatus struct {
        LoggedIn bool `json:"logged_in"`
    }
    json.NewDecoder(resp.Body).Decode(&loginStatus)
    fmt.Printf("Logged in: %v\n", loginStatus.LoggedIn)
    
    // Search
    searchReq := map[string]interface{}{
        "keyword":   "美食",
        "page":      1,
        "page_size": 10,
    }
    body, _ := json.Marshal(searchReq)
    resp, err = http.Post(
        baseURL+"/api/search",
        "application/json",
        bytes.NewBuffer(body),
    )
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    var searchResult map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&searchResult)
    fmt.Printf("Search results: %+v\n", searchResult)
    
    // Publish image post
    publishReq := map[string]interface{}{
        "title": "美食分享",
        "desc":  "今天做的菜真好吃！\n#美食 #家常菜",
        "images": []string{
            "/path/to/image1.jpg",
            "/path/to/image2.jpg",
        },
        "tags": []string{"美食", "家常菜"},
    }
    body, _ = json.Marshal(publishReq)
    resp, err = http.Post(
        baseURL+"/api/create_image_note",
        "application/json",
        bytes.NewBuffer(body),
    )
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    result, _ := io.ReadAll(resp.Body)
    fmt.Printf("Published: %s\n", result)
}
```

### Python: MCP Client Integration

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    server_params = StdioServerParameters(
        command="/path/to/xiaohongshu-mcp",
        args=[],
        env={}
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # Check login status
            result = await session.call_tool(
                "xhs_check_login_status",
                arguments={}
            )
            print(f"Login status: {result}")
            
            # Search
            result = await session.call_tool(
                "xhs_search",
                arguments={
                    "keyword": "美食",
                    "page": 1,
                    "page_size": 10
                }
            )
            print(f"Search results: {result}")
            
            # Publish post
            result = await session.call_tool(
                "xhs_create_image_note",
                arguments={
                    "title": "美食分享",
                    "desc": "今天做的菜真好吃！\n#美食 #家常菜",
                    "images": [
                        "/path/to/image1.jpg",
                        "/path/to/image2.jpg"
                    ],
                    "tags": ["美食", "家常菜"]
                }
            )
            print(f"Published: {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Common Patterns

### Pattern 1: Content Automation Workflow

```go
// 1. Check login
loggedIn, _ := client.CheckLoginStatus(ctx)
if !loggedIn {
    client.Login(ctx)
}

// 2. Search for trending topics
results, _ := client.Search(ctx, xhs.SearchParams{
    Keyword:  "trending_topic",
    Page:     1,
    PageSize: 20,
})

// 3. Analyze top posts
for _, item := range results.Items[:5] {
    detail, _ := client.GetNoteDetail(ctx, xhs.NoteDetailParams{
        FeedID:    item.FeedID,
        XsecToken: item.XsecToken,
    })
    // Analyze engagement metrics
    fmt.Printf("Likes: %d, Comments: %d\n", 
        detail.LikeCount, detail.CommentCount)
}

// 4. Publish optimized content
client.CreateImageNote(ctx, xhs.ImageNoteParams{
    Title:  "Title ≤20 chars",
    Desc:   "Optimized content based on analysis\n#trending",
    Images: []string{"/path/to/image.jpg"},
    Tags:   []string{"trending", "topic"},
})
```

### Pattern 2: Engagement Automation

```go
// Get recommendations
feeds, _ := client.GetRecommendFeeds(ctx, xhs.FeedParams{
    PageSize: 20,
})

// Engage with relevant content
for _, feed := range feeds.Items {
    // Like posts in your niche
    if isRelevant(feed.Title, feed.Desc) {
        client.LikeNote(ctx, xhs.LikeParams{
            FeedID:    feed.FeedID,
            XsecToken: feed.XsecToken,
        })
        
        // Add thoughtful comment
        client.CommentNote(ctx, xhs.CommentParams{
            FeedID:    feed.FeedID,
            XsecToken: feed.XsecToken,
            Content:   generateComment(feed),
        })
    }
}
```

### Pattern 3: Scheduled Publishing

```go
// Prepare content queue
posts := []xhs.ImageNoteParams{
    {
        Title:  "Morning Post",
        Desc:   "Content 1\n#tag1",
        Images: []string{"/images/1.jpg"},
    },
    {
        Title:  "Evening Post",
        Desc:   "Content 2\n#tag2",
        Images: []string{"/images/2.jpg"},
    },
}

// Schedule publishing (respect 50 posts/day limit)
for i, post := range posts {
    if i >= 50 {
        break // Daily limit
    }
    
    client.CreateImageNote(ctx, post)
    
    // Wait between posts (avoid rate limiting)
    time.Sleep(5 * time.Minute)
}
```

## Troubleshooting

### Issue: Login Session Expired

**Symptom:** API calls return "not logged in" errors

**Solution:**
```go
// Re-authenticate
err := client.Login(ctx)
if err != nil {
    log.Fatal("Login failed:", err)
}
```

### Issue: Title/Content Length Violations

**Symptom:** Post creation fails with validation error

**Solution:**
```go
func validatePost(title, content string) error {
    if len([]rune(title)) > 20 {
        return fmt.Errorf("title too long: %d characters (max 20)", 
            len([]rune(title)))
    }
    if len([]rune(content)) > 1000 {
        return fmt.Errorf("content too long: %d characters (max 1000)", 
            len([]rune(content)))
    }
    return nil
}
```

### Issue: Multiple Web Sessions

**Symptom:** Account gets logged out unexpectedly

**Solution:** Ensure only one web session is active. Use mobile app for manual checks while MCP is running.

### Issue: Image Upload Fails

**Symptom:** Image post creation returns error

**Solution:** Use local absolute paths instead of URLs:
```go
// ❌ Avoid
images := []string{"https://example.com/image.jpg"}

// ✅ Prefer
images := []string{"/Users/username/Pictures/image.jpg"}
```

### Issue: Rate Limiting

**Symptom:** Actions fail with rate limit errors

**Solution:** Add delays between operations:
```go
const (
    postDelay    = 5 * time.Minute  // Between posts
    likeDelay    = 2 * time.Second  // Between likes
    commentDelay = 10 * time.Second // Between comments
)

time.Sleep(postDelay)
```

### Issue: Browser Download Fails

**Symptom:** First run hangs or fails downloading browser

**Solution:** 
- Ensure stable internet connection
- Check firewall/proxy settings
- Manually download browser and set `PLAYWRIGHT_BROWSERS_PATH` env var

### Issue: Account Verification Required

**Symptom:** Platform requests identity verification

**Solution:** This is normal for new/unverified accounts. Complete identity verification through the mobile app, then retry.

## Best Practices

1. **Respect platform limits:** Max 50 posts/day per account
2. **Avoid spam behavior:** Add delays between automated actions
3. **Use quality content:** Platform penalizes low-quality/duplicate content
4. **Monitor account health:** Watch for verification requests or restrictions
5. **Use local files:** More reliable than URL-based image/video uploads
6. **Validate content:** Check title/content length before posting
7. **Handle errors gracefully:** Implement retry logic with exponential backoff
8. **Keep sessions alive:** Re-authenticate when sessions expire
9. **Single web session:** Don't log in on multiple web browsers simultaneously
10. **Avoid prohibited content:** Use content screening tools to check for violations
