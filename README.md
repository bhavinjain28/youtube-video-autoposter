# YouTube Shorts Auto-Poster

An automated pipeline that watches a Google Drive folder, generates AI-powered titles and descriptions, and uploads videos to YouTube automatically on a schedule. No manual effort required after setup.

Built with n8n Cloud, Google Drive API, YouTube Data API v3, and OpenRouter AI.

---

## Demo

Drop a video into Google Drive → AI generates metadata → video posts to YouTube automatically.

---

## Architecture

See `docs/architecture.png` for the full pipeline diagram.

The workflow has 9 nodes running in sequence:

1. **Schedule trigger** — runs on a configurable schedule (e.g. daily at 9am IST)
2. **Get next video from folder** — fetches the next unprocessed MP4 from Google Drive
3. **Generate title & description** — sends filename and context to OpenRouter AI model
4. **Clean LLM JSON** — parses and sanitises the AI response
5. **YouTube video metadata** — formats snippet, status, category for the upload
6. **Initialize resumable upload** — starts a resumable upload session with YouTube Data API v3
7. **Download video file** — streams the file from Google Drive
8. **Upload video file** — pushes the video bytes to YouTube
9. **Move video to Posted** — moves the file to a `Posted/` folder to prevent re-uploading

---

## Tech Stack

| Tool | Purpose |
|---|---|
| n8n Cloud | Workflow automation and scheduling |
| Google Drive API | Video storage, file trigger, move-to-posted |
| YouTube Data API v3 | Resumable video upload with metadata |
| OpenRouter AI | AI-generated titles, descriptions, hashtags |
| Google Cloud Console | OAuth2 credentials and API enablement |

---

## Setup

### Prerequisites

- n8n Cloud account (free tier at [n8n.io](https://n8n.io))
- Google account with YouTube channel
- Google Cloud project with these APIs enabled:
  - YouTube Data API v3
  - Google Drive API
- OpenRouter account and API key ([openrouter.ai](https://openrouter.ai))

### Step 1 — Google Cloud Setup

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g. `n8n Automation`)
3. Enable **YouTube Data API v3** and **Google Drive API** under APIs & Services → Library
4. Go to APIs & Services → Credentials → Create OAuth Client ID
   - Application type: Web application
   - Authorized redirect URI: `https://oauth.n8n.cloud/oauth2/callback`
5. Copy your Client ID and Client Secret
6. Under OAuth Consent Screen → Data Access → add these scopes:
   - `https://www.googleapis.com/auth/drive`
   - `https://www.googleapis.com/auth/youtube`
   - `https://www.googleapis.com/auth/youtube.upload`
7. Under OAuth Consent Screen → Test Users → add your Gmail address

### Step 2 — Google Drive Folder Structure

Create this folder structure in Google Drive:

```
Content Pipeline/
├── Shorts Ready/     ← drop videos here
└── Posted/           ← n8n moves files here after upload
```

### Step 3 — n8n Setup

1. Log into your n8n Cloud instance
2. Go to Credentials → Create Credential → Google Drive OAuth2 API
   - Paste your Client ID and Client Secret
   - Click Sign in with Google
3. Create a second credential → YouTube OAuth2 API
   - Use the same Client ID and Client Secret
   - Click Sign in with Google
4. Create a third credential → OpenRouter
   - Paste your OpenRouter API key

### Step 4 — Import Workflow

1. In n8n, click New Workflow → Import from File
2. Select `workflow.json` from this repo
3. Assign credentials to each node:
   - Google Drive nodes → Google Drive OAuth2 account
   - YouTube upload node → YouTube OAuth2 account
   - OpenRouter node → OpenRouter account
4. In the `Get next video from folder` node, select your `Shorts Ready` folder
5. In the `Move video to Posted` node, select your `Posted` folder
6. Set the OpenRouter model to `google/gemini-2.0-flash-exp:free` (free tier)
7. Toggle the workflow **Active**

---

## How It Works

Once activated, the workflow runs on schedule. It picks the oldest unprocessed video from your `Shorts Ready` Google Drive folder, sends the filename to an AI model to generate a YouTube-optimised title, description, and hashtags, then uploads the video using YouTube's resumable upload API. After a successful upload, the file is moved to the `Posted` folder so it is not uploaded again.

To post a new video, simply drop an MP4 into the `Shorts Ready` folder. The next scheduled run will pick it up automatically.

---

## Challenges Solved

| Challenge | Solution |
|---|---|
| OAuth redirect URI mismatch | Used `https://oauth.n8n.cloud/oauth2/callback` — the correct n8n Cloud callback URL |
| Error 403 access denied | Added Gmail as test user in Google Cloud OAuth Consent Screen |
| OAuth callback state invalid | Cleared browser cookies and reconnected credentials |
| Drive forbidden on file move | Added `https://www.googleapis.com/auth/drive` full scope to OAuth credential |
| OpenRouter token limit | Switched to free model (`gemini-2.0-flash-exp:free`) and set max tokens to 500 |

---

## Folder Structure

```
youtube-shorts-autoposter/
├── README.md
├── workflow.json          ← import this into n8n
└── docs/
    └── architecture.png   ← pipeline diagram
```

---

## Future Improvements

- Add Instagram Reels cross-posting via Blotato
- Add long video → auto-clip pipeline using Whisper + Gemini
- Add AI speech analysis layer (summary, pros, cons, voiceover commentary)
- Replace Move-to-Posted node with Google Sheets tracking log
- Add Telegram notification on successful upload

---

## Author

Bhavin Jain — [LinkedIn](https://linkedin.com/in/bhavin-jain) | Mumbai, India

BI & Data Analyst | MS Data Science, WPI | Building automation pipelines at the intersection of data and AI.
