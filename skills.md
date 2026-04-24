---

name: clawcaster-agent
description: Agent-optimized skill for OpenClaw to generate and publish content across Twitter/X, LinkedIn, and YouTube using Zernio API.
metadata:
{
"openclaw":
{
"emoji": "🤖",
"requires": { "bins": ["bash", "curl"], "os": ["linux", "darwin"] }
}
}
---

# Social Publisher Agent (OpenClaw × Zernio)

## What this skill does

This skill enables OpenClaw to:

- interpret a content request
- generate platform-optimized posts
- choose one or multiple social platforms
- publish via Zernio API
- optionally schedule posts

Supported platforms:

- Twitter / X
- LinkedIn
- YouTube

---

# Core Agent Behavior (IMPORTANT)

## 1. Platform Routing Logic

OpenClaw MUST decide platform(s) using the rules below:

```text
IF user explicitly specifies platform → use that platform only

ELSE IF content includes video + title intent → YouTube

ELSE IF content is professional / long-form → LinkedIn

ELSE IF content is short / announcement → Twitter/X

ELSE → publish to Twitter + LinkedIn
```

---

## 2. Normalize Input First (MANDATORY STEP)

OpenClaw MUST convert any request into this internal format before publishing:

```json
{
	"content": "",
	"title": "",
	"media": [],
	"platforms": []
}
```

---

## 3. Publishing Rule

- Always generate platform-specific adaptation BEFORE sending to API
- Never send raw user text without transformation
- Always validate platform constraints

---

# Execution Workflow

## Step 1: Parse request

- extract intent (post, video, article, update)
- detect media presence
- detect tone (professional, casual, technical)

## Step 2: Decide platform(s)

- apply routing logic

## Step 3: Generate content variants

- Twitter/X → short, punchy
- LinkedIn → structured, professional
- YouTube → title + description

## Step 4: Build Zernio payload

## Step 5: Send request

POST [https://zernio.com/api/v1/posts](https://zernio.com/api/v1/posts)

Headers:

- Authorization: Bearer LATE_API_KEY
- Content-Type: application/json

## Step 6: Return result

- post ID
- platform(s)
- status
- URL if available

---

# Platform Capabilities

## Twitter / X

Use when:

- short updates
- announcements
- quick engagement content

Rules:

- max ~280 characters
- up to 4 images
- 1 video per post

Example platform object:

```json
{
	"platform": "twitter",
	"accountId": "ACCOUNT_ID"
}
```

---

## LinkedIn

Use when:

- professional updates
- product launches
- technical insights

Rules:

- up to 3000 characters
- supports images, video, documents
- hook in first 210 characters

Example platform object:

```json
{
	"platform": "linkedin",
	"accountId": "ACCOUNT_ID"
}
```

---

## YouTube

Use when:

- video content exists
- tutorials, demos, explainers

Rules:

- requires video file
- requires title
- supports Shorts (auto-detected)
- visibility: public | unlisted | private

Example platform object:

```json
{
	"platform": "youtube",
	"accountId": "ACCOUNT_ID",
	"platformSpecificData": {
		"title": "Video Title",
		"visibility": "public"
	}
}
```

---

# Unified Zernio Request Format

```json
{
	"content": "main post content",
	"mediaItems": [
		{
			"type": "image | video | document",
			"url": "https://..."
		}
	],
	"platforms": [
		{
			"platform": "twitter | linkedin | youtube",
			"accountId": "ACCOUNT_ID"
		}
	],
	"publishNow": true
}
```

---

# Guardrails

- NEVER expose LATE_API_KEY
- NEVER post empty content
- NEVER send YouTube request without video + title
- RESPECT platform limits
- DO NOT retry failed requests automatically

---

# Failure Handling

If API fails:

- return HTTP status
- return error message
- stop execution

If network fails:

- output curl request used
- include error logs

---

# Output Format

Return:

- platforms used
- post content summary
- post ID
- published URL (if available)
- scheduled time (if applicable)

---

# Example Usage

## User:

"Post a product update about OpenClaw"

## OpenClaw behavior:

- detects professional announcement
- chooses LinkedIn + Twitter
- generates 2 versions
- sends via Zernio

---

## Example Twitter Request

```bash
curl -X POST https://zernio.com/api/v1/posts \
 -H "Authorization: Bearer $LATE_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
  "content": "Just shipped a new feature 🚀",
  "platforms": [{"platform": "twitter", "accountId": "acc_123"}],
  "publishNow": true
}'
```

---

## Example LinkedIn Request

```bash
curl -X POST https://zernio.com/api/v1/posts \
 -H "Authorization: Bearer $LATE_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
  "content": "We just launched a new AI feature that improves automation.",
  "platforms": [{"platform": "linkedin", "accountId": "acc_123"}],
  "publishNow": true
}'
```

---

## Example YouTube Request

```bash
curl -X POST https://zernio.com/api/v1/posts \
 -H "Authorization: Bearer $LATE_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
  "content": "Watch our full tutorial",
  "mediaItems": [{"type": "video", "url": "https://example.com/video.mp4"}],
  "platforms": [{
    "platform": "youtube",
    "accountId": "acc_123",
    "platformSpecificData": {
      "title": "OpenClaw Tutorial",
      "visibility": "public"
    }
  }],
  "publishNow": true
}'
```

---

# Summary

This skill transforms OpenClaw into an autonomous social media agent that:

- understands intent
- chooses platforms
- adapts content
- publishes via Zernio
- handles multi-platform distribution intelligently
