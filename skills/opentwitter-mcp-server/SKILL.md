---
name: opentwitter-mcp-server
description: Twitter/X MCP server for fetching user profiles, tweets, follower events, and KOL tracking via AI assistants
triggers:
  - "get Twitter user profile"
  - "search tweets about"
  - "monitor Twitter account for followers"
  - "check who unfollowed someone on Twitter"
  - "find deleted tweets from user"
  - "track Twitter KOL followers"
  - "get recent tweets from"
  - "search Twitter with filters"
---

# opentwitter-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The **opentwitter-mcp** is an MCP (Model Context Protocol) server that provides AI assistants with Twitter/X data access capabilities. It enables fetching user profiles, searching tweets, monitoring follower events, tracking deleted tweets, and managing watch lists for KOL (Key Opinion Leader) accounts.

The server connects to the 6551.io API service and exposes 13 tools for Twitter data operations.

## Installation

### Prerequisites

1. Get your API token from [https://6551.io/mcp](https://6551.io/mcp)
2. Clone or download the repository
3. Have `uv` package manager installed

### Quick Install (Claude Code)

```bash
claude mcp add twitter \
  -e TWITTER_TOKEN=$TWITTER_TOKEN \
  -- uv --directory /path/to/twitter-mcp run twitter-mcp
```

### Manual Configuration

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "twitter": {
      "command": "uv",
      "args": ["--directory", "/path/to/twitter-mcp", "run", "twitter-mcp"],
      "env": {
        "TWITTER_TOKEN": "${TWITTER_TOKEN}"
      }
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "twitter": {
      "command": "uv",
      "args": ["--directory", "/path/to/twitter-mcp", "run", "twitter-mcp"],
      "env": {
        "TWITTER_TOKEN": "${TWITTER_TOKEN}"
      }
    }
  }
}
```

**Continue.dev** (`~/.continue/config.yaml`):

```yaml
mcpServers:
  - name: twitter
    command: uv
    args:
      - --directory
      - /path/to/twitter-mcp
      - run
      - twitter-mcp
    env:
      TWITTER_TOKEN: ${TWITTER_TOKEN}
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `TWITTER_TOKEN` | **Yes** | Bearer token from 6551.io |
| `TWITTER_API_BASE` | No | Override API URL (default: `https://ai.6551.io`) |
| `TWITTER_MAX_ROWS` | No | Max results per query (default: 100) |

Alternative: Create `config.json` in project root:

```json
{
  "api_base_url": "https://ai.6551.io",
  "api_token": "your-token-here",
  "max_rows": 100
}
```

## Available Tools

### User Profile Tools

**`get_twitter_user`** - Get user by username
```python
# Example use from AI assistant:
# "Show me @elonmusk's profile"
{
  "username": "elonmusk"
}
```

**`get_twitter_user_by_id`** - Get user by numeric ID
```python
{
  "user_id": "44196397"
}
```

### Tweet Retrieval Tools

**`get_twitter_user_tweets`** - Get recent tweets from a user
```python
# "What did @VitalikButerin tweet recently"
{
  "username": "VitalikButerin",
  "count": 20
}
```

**`get_twitter_tweet_by_id`** - Get specific tweet with nested replies/quotes
```python
{
  "tweet_id": "1234567890"
}
```

**`get_twitter_article_by_id`** - Get Twitter article/long-form content
```python
{
  "article_id": "1234567890"
}
```

### Search Tools

**`search_twitter`** - Basic keyword search
```python
# "Search Bitcoin related tweets"
{
  "query": "Bitcoin",
  "count": 50
}
```

**`search_twitter_advanced`** - Advanced search with filters
```python
# "Popular tweets about ETH with 1000+ likes"
{
  "query": "ETH",
  "min_likes": 1000,
  "min_retweets": 100,
  "language": "en",
  "count": 30
}
```

Available filters:
- `min_likes`, `max_likes`
- `min_retweets`, `max_retweets`
- `min_replies`, `max_replies`
- `language` (ISO code: en, ja, etc.)
- `since`, `until` (dates)
- `verified_only` (boolean)

### Engagement Tools

**`get_twitter_quote_tweets_by_id`** - Get tweets quoting a specific tweet
```python
# "Who quoted this tweet"
{
  "tweet_id": "1234567890",
  "count": 50
}
```

**`get_twitter_retweet_users_by_id`** - Get users who retweeted
```python
# "Who retweeted this tweet"
{
  "tweet_id": "1234567890",
  "count": 100
}
```

### Monitoring Tools

**`get_twitter_watch`** - List all monitored accounts
```python
# "Show my Twitter watch list"
{}
```

**`add_twitter_watch`** - Add account to monitoring
```python
# "Monitor @elonmusk with follower tracking"
{
  "username": "elonmusk",
  "event_types": ["NEW_FOLLOWER", "NEW_UNFOLLOWER", "NEW_TWEET"],
  "remark": "Tesla CEO",
  "ca": "0x1234..."  # Optional contract address
}
```

Event types:
- `NEW_TWEET` - New tweets
- `NEW_TWEET_REPLY` - Replies
- `NEW_TWEET_QUOTE` - Quote tweets
- `NEW_RETWEET` - Retweets
- `NEW_FOLLOWER` - New followers
- `NEW_UNFOLLOWER` - Unfollowers
- `DELETE` - Deleted tweets
- `UPDATE_NAME` - Username changes
- `UPDATE_DESCRIPTION` - Bio updates
- `UPDATE_AVATAR` - Profile picture changes
- `UPDATE_BANNER` - Banner changes
- `CA` - Tweets with contract addresses

**`delete_twitter_watch`** - Remove from monitoring
```python
{
  "username": "elonmusk"
}
```

### Event Tracking Tools

**`get_twitter_follower_events`** - Get follower/unfollower events
```python
# "Who followed @elonmusk recently"
{
  "username": "elonmusk",
  "event_type": "NEW_FOLLOWER",
  "count": 50
}
```

**`get_twitter_deleted_tweets`** - Get deleted tweets
```python
# "What tweets did @elonmusk delete"
{
  "username": "elonmusk",
  "count": 20
}
```

**`get_twitter_kol_followers`** - Get KOL (influential) followers
```python
# "Which KOLs follow @elonmusk"
{
  "username": "elonmusk",
  "count": 100
}
```

## Data Structures

### User Object
```python
{
  "userId": "44196397",
  "screenName": "elonmusk",
  "name": "Elon Musk",
  "description": "Bio text...",
  "followersCount": 170000000,
  "friendsCount": 500,
  "statusesCount": 30000,
  "verified": true,
  "profileImageUrl": "https://...",
  "profileBannerUrl": "https://...",
  "createdAt": "2009-06-02T20:12:29Z"
}
```

### Tweet Object
```python
{
  "id": "1234567890",
  "text": "Tweet content...",
  "createdAt": "2024-02-20T12:00:00Z",
  "language": "en",
  "retweetCount": 100,
  "favoriteCount": 500,
  "replyCount": 20,
  "quoteCount": 10,
  "viewCount": 10000,
  "userScreenName": "elonmusk",
  "userName": "Elon Musk",
  "userIdStr": "44196397",
  "userFollowers": 170000000,
  "userVerified": true,
  "conversationId": "1234567890",
  "isReply": false,
  "isQuote": false,
  "hashtags": ["crypto", "bitcoin"],
  "media": [
    {
      "type": "photo",
      "url": "https://...",
      "thumbUrl": "https://..."
    }
  ],
  "urls": [
    {
      "url": "https://t.co/...",
      "expandedUrl": "https://example.com",
      "displayUrl": "example.com"
    }
  ],
  "mentions": [
    {
      "username": "VitalikButerin",
      "name": "Vitalik Buterin"
    }
  ]
}
```

## WebSocket Real-time Subscriptions

Connect to `wss://ai.6551.io/open/twitter_wss?token=YOUR_TOKEN` for real-time events.

### Subscribe
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "twitter.subscribe"
}
```

### Event Notification
```json
{
  "jsonrpc": "2.0",
  "method": "twitter.event",
  "params": {
    "id": 123456,
    "twAccount": "elonmusk",
    "twUserName": "Elon Musk",
    "profileUrl": "https://twitter.com/elonmusk",
    "eventType": "NEW_TWEET",
    "content": { /* tweet object */ },
    "ca": "0x1234...",
    "remark": "Custom note",
    "createdAt": "2026-03-06T10:00:00Z"
  }
}
```

### Unsubscribe
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "twitter.unsubscribe"
}
```

## Common Patterns

### Search with Filters
```python
# High-engagement tweets in specific timeframe
search_twitter_advanced(
    query="AI",
    min_likes=1000,
    min_retweets=500,
    since="2024-01-01",
    until="2024-12-31",
    verified_only=True,
    language="en"
)
```

### Monitor Multiple Accounts
```python
# Add multiple KOLs to watch list
for username in ["elonmusk", "VitalikButerin", "naval"]:
    add_twitter_watch(
        username=username,
        event_types=["NEW_TWEET", "NEW_FOLLOWER"],
        remark=f"Tracking {username}"
    )
```

### Track Engagement
```python
# Get full engagement picture for a tweet
tweet = get_twitter_tweet_by_id(tweet_id="1234567890")
quotes = get_twitter_quote_tweets_by_id(tweet_id="1234567890")
retweet_users = get_twitter_retweet_users_by_id(tweet_id="1234567890")
```

### Follower Analysis
```python
# Compare followers vs KOL followers
all_followers = get_twitter_follower_events(
    username="elonmusk",
    event_type="NEW_FOLLOWER"
)
kol_followers = get_twitter_kol_followers(username="elonmusk")
```

## Development

### Run Locally
```bash
cd /path/to/twitter-mcp
uv sync
export TWITTER_TOKEN=your-token
uv run twitter-mcp
```

### Debug with MCP Inspector
```bash
npx @modelcontextprotocol/inspector \
  uv --directory /path/to/twitter-mcp run twitter-mcp
```

### Project Structure
```
src/twitter_mcp/
├── server.py      # Entry point
├── app.py         # FastMCP instance
├── config.py      # Config loader
├── api_client.py  # HTTP client
└── tools.py       # Tool implementations
```

## Troubleshooting

### Authentication Errors
- Verify `TWITTER_TOKEN` is set correctly
- Check token validity at https://6551.io/mcp
- Ensure no extra whitespace in token value

### Rate Limiting
- Default `max_rows` is 100; reduce if hitting limits
- Space out requests when monitoring multiple accounts

### Connection Issues
- Verify `TWITTER_API_BASE` if using custom endpoint
- Check firewall/proxy settings for `ai.6551.io` access
- For WebSocket: ensure WSS protocol is allowed

### Empty Results
- Username lookup is case-insensitive but must be exact
- Some accounts may have restricted data access
- Recent tweets may take moments to appear in search

### Tool Not Found
- Restart your AI client after config changes
- Verify `uv` is in PATH
- Check project path is absolute, not relative

## Common Use Cases

**Content Research**: Search tweets with advanced filters to find trending topics, viral content, or specific conversations.

**Influencer Monitoring**: Track follower growth, engagement patterns, and deleted tweets for transparency.

**Community Management**: Monitor mentions, replies, and engagement on your own tweets or brand accounts.

**Competitive Analysis**: Track competitors' follower changes, KOL supporters, and content strategies.

**Event Tracking**: Set up watches for breaking news, product launches, or community events via real-time WebSocket feeds.
