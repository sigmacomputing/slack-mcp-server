# slack-mcp-server

> **Fork of [zencoderai/slack-mcp-server](https://github.com/zencoderai/slack-mcp-server)** — customized for read-only access to the Sigma Slack workspace.

## About This Fork

This is a Sigma-specific fork of the Zencoder Slack MCP server. It is tailored for reading and searching content in the Sigma Slack workspace from Cursor IDE, with no write access (no posting messages or replying to threads). The fork adds tools for file downloads, full-text search, pins, bookmarks, and reactions that the upstream project does not provide.

## Disclaimer
This project includes [code](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/slack) originally developed by Anthropic and released under the MIT License. Substantial modifications and new functionality have been added by For Good AI Inc. (dba Zencoder Inc.), and are licensed under the Apache License, Version 2.0. Additional modifications in this fork are by Sigma.

## Overview
A read-oriented Model Context Protocol (MCP) server for interacting with the Sigma Slack workspace. This server provides tools to list channels, read messages, search content, manage reactions, download files, view pins and bookmarks, and browse user profiles.

## Available Tools

### Channels & Messages

1. **slack_list_channels**
   - List public or pre-defined channels in the workspace
   - Optional inputs:
     - `limit` (number, default: 100, max: 200): Maximum number of channels to return
     - `cursor` (string): Pagination cursor for next page
   - Returns: List of channels with their IDs and information

2. **slack_join_channel**
   - Join a public Slack channel (the bot must join before it can read history or files)
   - Required inputs:
     - `channel_id` (string): The ID of the public channel to join
   - Returns: Channel join confirmation

3. **slack_get_channel_history**
   - Get recent messages from a channel
   - Required inputs:
     - `channel_id` (string): The channel ID
   - Optional inputs:
     - `limit` (number, default: 10): Number of messages to retrieve
   - Returns: List of messages with their content and metadata

4. **slack_get_thread_replies**
   - Get all replies in a message thread
   - Required inputs:
     - `channel_id` (string): The channel containing the thread
     - `thread_ts` (string): Timestamp of the parent message
   - Returns: List of replies with their content and metadata

### Reactions

5. **slack_add_reaction**
   - Add an emoji reaction to a message
   - Required inputs:
     - `channel_id` (string): The channel containing the message
     - `timestamp` (string): Message timestamp to react to
     - `reaction` (string): Emoji name without colons
   - Returns: Reaction confirmation

6. **slack_remove_reaction**
   - Remove an emoji reaction from a message
   - Required inputs:
     - `channel_id` (string): The channel containing the message
     - `timestamp` (string): Message timestamp
     - `reaction` (string): Emoji name to remove (without colons)
   - Returns: Removal confirmation

7. **slack_get_reactions**
   - Get all emoji reactions on a specific message
   - Required inputs:
     - `channel_id` (string): The channel containing the message
     - `timestamp` (string): Message timestamp
   - Returns: List of reactions with emoji names and reacting users

### Users

8. **slack_get_users**
   - Get list of workspace users with basic profile information
   - Optional inputs:
     - `cursor` (string): Pagination cursor for next page
     - `limit` (number, default: 100, max: 200): Maximum users to return
   - Returns: List of users with their basic profiles

9. **slack_get_user_profile**
   - Get detailed profile information for a specific user
   - Required inputs:
     - `user_id` (string): The user's ID
   - Returns: Detailed user profile information

### Files

10. **slack_download_file**
    - Download a file shared in Slack by its file ID
    - Required inputs:
      - `file_id` (string): The Slack file ID (e.g., `F0123456789`) found in the `files` array of messages from `slack_get_channel_history`
    - Behavior:
      - For text-based files (code, markdown, plain text, CSV, JSON, etc.): returns the file content as a string
      - For binary files (images, PDFs, etc.): saves the file to a local temp directory and returns the file path
    - Enforces a 10MB file size limit
    - Returns: File name, mimetype, size, and either content (text) or local file path (binary)

11. **slack_list_files**
    - List recent files shared in Slack, optionally filtered by channel or user
    - Optional inputs:
      - `channel_id` (string): Filter files by channel ID
      - `user_id` (string): Filter files by user ID
      - `count` (number, default: 20, max: 100): Number of files to return
    - Returns: List of file metadata including id, name, filetype, mimetype, size, timestamp, user, channels, and download URL

### Search

12. **slack_search_messages**
    - Search for messages across public channels in the workspace
    - Required inputs:
      - `query` (string): Search query (supports Slack modifiers like `in:#channel`, `from:@user`, `before:2024-01-01`, `has:link`, `has:reaction`, etc.)
    - Optional inputs:
      - `count` (number, default: 20, max: 100): Number of results to return
      - `sort` (string, default: "timestamp"): Sort by `timestamp` or `score`
      - `sort_dir` (string, default: "desc"): Sort direction `asc` or `desc`
    - Returns: Matching messages with context

13. **slack_search_files**
    - Search for files shared across the workspace
    - Required inputs:
      - `query` (string): Search query (supports Slack modifiers like `in:#channel`, `from:@user`, `type:pdf`, etc.)
    - Optional inputs:
      - `count` (number, default: 20, max: 100): Number of results to return
      - `sort` (string, default: "timestamp"): Sort by `timestamp` or `score`
      - `sort_dir` (string, default: "desc"): Sort direction `asc` or `desc`
    - Returns: Matching files with metadata

### Pins & Bookmarks

14. **slack_list_pins**
    - List all pinned items in a channel
    - Required inputs:
      - `channel_id` (string): The channel ID
    - Returns: List of pinned messages and files

15. **slack_list_bookmarks**
    - List all bookmarks (saved links) in a channel
    - Required inputs:
      - `channel_id` (string): The channel ID
    - Returns: List of bookmarks with titles, URLs, and metadata

## Slack Bot Setup

To use this MCP server, you need to create a Slack app and configure it with the necessary permissions:

### 1. Create a Slack App
- Visit the [Slack Apps page](https://api.slack.com/apps)
- Click "Create New App"
- Choose "From scratch"
- Name your app and select your workspace

### 2. Configure Bot Token Scopes
Navigate to "OAuth & Permissions" and add these scopes:
- `bookmarks:read` - View bookmarks in channels
- `channels:history` - View messages and other content in public channels
- `channels:join` - Join public channels in the workspace
- `channels:read` - View basic channel information
- `files:read` - Access file content and metadata shared in channels
- `pins:read` - View pinned items in channels
- `reactions:read` - View emoji reactions on messages
- `reactions:write` - Add and remove emoji reactions
- `search:read.files` - Search files across the workspace
- `search:read.public` - Search messages in public channels
- `users:read` - View users and their basic information
- `users.profile:read` - View detailed profiles about users

### 3. Install App to Workspace
- Click "Install to Workspace" and authorize the app
- Save the "Bot User OAuth Token" that starts with `xoxb-`

### 4. Get Your Team ID
Get your Team ID (starts with a `T`) by following [this guidance](https://slack.com/help/articles/221769328-Locate-your-Slack-URL-or-ID#find-your-workspace-or-org-id)

### 5. Add Bot to Channels (Optional)
For the bot to access private channels or to post messages, you may need to invite it to specific channels using `/invite @your-bot-name`

## Features

- **Multiple Transport Support**: Supports both stdio and Streamable HTTP transports
- **Modern MCP SDK**: Updated to use the latest MCP SDK (v1.13.2) with modern APIs
- **Comprehensive Slack Integration**: Full set of read-oriented Slack operations including:
  - List and join channels
  - Get channel history and thread replies
  - Add, remove, and view reactions
  - List and search users with profiles
  - Download and search files
  - Search messages across public channels
  - View pinned items and bookmarks

## Installation

### Local Development
```bash
npm install
npm run build
```

### Global Installation (NPM)
```bash
npm install -g @zencoderai/slack-mcp-server
```

### Docker Installation
```bash
# Build the Docker image locally
docker build -t slack-mcp-server .

# Or pull from Docker Hub
docker pull zencoderai/slack-mcp:latest

# Or pull a specific version
docker pull zencoderai/slack-mcp:1.0.0
```

## Configuration

Set the following environment variables:

```bash
export SLACK_BOT_TOKEN="xoxb-your-bot-token"
export SLACK_TEAM_ID="your-team-id"
export SLACK_CHANNEL_IDS="channel1,channel2,channel3"  # Optional: predefined channels
export AUTH_TOKEN="your-auth-token"  # Optional: Bearer token for HTTP authorization (Streamable HTTP transport only)
```

## Usage

### Command Line Options

```bash
slack-mcp [options]

Options:
  --transport <type>     Transport type: 'stdio' or 'http' (default: stdio)
  --port <number>        Port for HTTP server when using Streamable HTTP transport (default: 3000)
  --token <token>        Bearer token for HTTP authorization (optional, can also use AUTH_TOKEN env var)
  --help, -h             Show this help message
```

### Local Usage Examples

#### Using the slack-mcp command (after global installation)
```bash
# Use stdio transport (default)
slack-mcp

# Use stdio transport explicitly
slack-mcp --transport stdio

# Use Streamable HTTP transport on default port 3000
slack-mcp --transport http

# Use Streamable HTTP transport on custom port
slack-mcp --transport http --port 8080

# Use Streamable HTTP transport with custom auth token
slack-mcp --transport http --token mytoken

# Use Streamable HTTP transport with auth token from environment variable
AUTH_TOKEN=mytoken slack-mcp --transport http
```

#### Using node directly (for development)
```bash
# Use stdio transport (default)
node dist/index.js

# Use stdio transport explicitly
node dist/index.js --transport stdio

# Use Streamable HTTP transport on default port 3000
node dist/index.js --transport http

# Use Streamable HTTP transport on custom port
node dist/index.js --transport http --port 8080

# Use Streamable HTTP transport with custom auth token
node dist/index.js --transport http --token mytoken

# Use Streamable HTTP transport with auth token from environment variable
AUTH_TOKEN=mytoken node dist/index.js --transport http
```

### Docker Usage Examples

#### Using Docker directly
```bash
# Run with stdio transport (default)
docker run --rm \
  -e SLACK_BOT_TOKEN="xoxb-your-bot-token" \
  -e SLACK_TEAM_ID="your-team-id" \
  zencoderai/slack-mcp:latest

# Run with HTTP transport on port 3000
docker run --rm -p 3000:3000 \
  -e SLACK_BOT_TOKEN="xoxb-your-bot-token" \
  -e SLACK_TEAM_ID="your-team-id" \
  zencoderai/slack-mcp:latest --transport http

# Run with HTTP transport on custom port
docker run --rm -p 8080:8080 \
  -e SLACK_BOT_TOKEN="xoxb-your-bot-token" \
  -e SLACK_TEAM_ID="your-team-id" \
  zencoderai/slack-mcp:latest --transport http --port 8080

# Run with custom auth token
docker run --rm -p 3000:3000 \
  -e SLACK_BOT_TOKEN="xoxb-your-bot-token" \
  -e SLACK_TEAM_ID="your-team-id" \
  -e AUTH_TOKEN="mytoken" \
  zencoderai/slack-mcp:latest --transport http
```

#### Using Docker Compose
Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  slack-mcp:
    # Use published image:
    image: zencoderai/slack-mcp:latest
    # Or build locally:
    # build: .
    environment:
      - SLACK_BOT_TOKEN=xoxb-your-bot-token
      - SLACK_TEAM_ID=your-team-id
      - SLACK_CHANNEL_IDS=channel1,channel2,channel3  # Optional
      - AUTH_TOKEN=your-auth-token  # Optional for HTTP transport
    ports:
      - "3000:3000"  # Only needed for HTTP transport
    command: ["--transport", "http"]  # Optional: specify transport type
    restart: unless-stopped
```

Then run:
```bash
# Start the service
docker compose up -d

# View logs
docker compose logs -f slack-mcp

# Stop the service
docker compose down
```

## Transport Types

### Stdio Transport
- **Use case**: Command-line tools and direct integrations
- **Communication**: Standard input/output streams
- **Default**: Yes

### Streamable HTTP Transport
- **Use case**: Remote servers and web-based integrations
- **Communication**: HTTP POST requests with optional Server-Sent Events streams
- **Features**: 
  - Session management
  - Bidirectional communication
  - Resumable connections
  - RESTful API endpoints
  - Bearer token authentication

## Authentication (Streamable HTTP Transport Only)

When using Streamable HTTP transport, the server supports Bearer token authentication:

1. **Command Line**: Use `--token <token>` to specify a custom token
2. **Environment Variable**: Set `AUTH_TOKEN=<token>` as a fallback
3. **Auto-generated**: If neither is provided, a random token is generated

The command line option takes precedence over the environment variable. Include the token in HTTP requests using the `Authorization: Bearer <token>` header.

## Troubleshooting

If you encounter permission errors, verify that:

1. All required scopes are added to your Slack app
2. The app is properly installed to your workspace
3. The tokens and workspace ID are correctly copied to your configuration
4. The app has been added to the channels it needs to access

## Development

### Build
```bash
npm run build
```

### Watch Mode
```bash
npm run watch
```

## API Endpoints (Streamable HTTP Transport)

When using Streamable HTTP transport, the server exposes the following endpoints:

- `POST /mcp` - Client-to-server communication
- `GET /mcp` - Server-to-client notifications (Server-Sent Events streams)
- `DELETE /mcp` - Session termination

## Changes from Previous Version

- **Updated MCP SDK**: Upgraded from v1.0.1 to v1.13.2
- **Modern API**: Migrated from low-level Server class to high-level McpServer class
- **Zod Validation**: Added proper schema validation using Zod
- **Transport Flexibility**: Added support for Streamable HTTP transport
- **Command Line Interface**: Added CLI arguments for transport selection
- **Session Management**: Implemented proper session handling for HTTP transport
- **Better Error Handling**: Improved error handling and logging