# 📸 Instagram Media Scraper

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Requests](https://img.shields.io/badge/Requests-2.x-2CA5E0?style=for-the-badge&logo=python&logoColor=white)
![BeautifulSoup](https://img.shields.io/badge/BeautifulSoup-4.x-43B02A?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Termux%20%7C%20Windows%20%7C%20macOS-000000?style=for-the-badge&logo=linux&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

> **A resilient, multi-strategy Instagram media extractor.**  
> Fetches public post images and videos (including carousels) via Instagram's private GraphQL API, with a BeautifulSoup HTML fallback for when the API is rate-limited.

---

## 📖 Overview

**Instagram Media Scraper** is a lightweight, mobile-optimized Python tool that pulls media metadata (images, videos, and carousel children) from any **public** Instagram profile. It uses a three-tier fetch strategy:

1. **Profile Info API** — resolves `username → user_id` and grabs follower/post counts.
2. **GraphQL Feed API** — paginated `edge_owner_to_timeline_media` query (up to ~250 posts).
3. **HTML Fallback** — parses embedded JSON (`image_versions2` / `video_versions`) when APIs return empty or blocked.

All results are deduplicated and saved as a clean `media.json` file — ready for OSINT pipelines, archiving, or dataset building.

> ⚠️ **Educational / research use only.** Only works on **public** accounts. Respect Instagram's Terms of Service and applicable laws.

---

## ✨ Key Features

### 🔍 Multi-Strategy Extraction
- **Method 1 — Profile API:** Resolves user ID, follower count, and post count.
- **Method 2 — GraphQL Feed:** Fetches full media with pagination (up to 5 pages / ~250 posts).
- **Method 3 — HTML Fallback:** Recursively walks embedded JSON to find media when APIs fail.

### 🎞️ Media Handling
- Extracts **images**, **videos**, and **carousel children** (sidecar posts).
- Picks the **highest-resolution** candidate automatically.
- Handles `GraphSidecar`, `GraphVideo`, and `GraphImage` types.

### 📱 Mobile-First Design
- Spoofs a modern **Android Chrome User-Agent** — reduces bot detection.
- Uses `X-IG-App-ID` header (Instagram's mobile web app ID) to mimic real traffic.
- Optional `X-Requested-With: XMLHttpRequest` for AJAX-style requests.

### 💾 Clean Output
- Deduplicated results keyed by post ID.
- Structured `media.json` with `id`, `shortcode`, `type`, `url`, `thumb` per item.
- One-line console summary: total count + video/image split.

### ⚙️ Robustness
- Per-request timeouts (15–20s).
- `try/except` on every network call — never crashes on a bad post.
- Automatic pagination with 1s sleep between pages.

---

## 📁 Project Folder Structure

```
instagram-media-scraper/
├── instagram_scraper.py     # Main scraper script
├── media.json               # Output file (generated at runtime)
├── README.md                # This file
├── LICENSE                  # MIT license
└── .gitignore               # Ignore cache, venv, output
```

---

## 🚀 Installation & Usage

### Prerequisites
- Python **3.8+**
- `pip` access
- Internet connection

### 1. Clone the repository
```bash
git clone https://github.com/rishi-security-labs/instagram.git
cd instagram-media-scraper
```

### 2. Install dependencies

**Linux / macOS / Termux:**
```bash
pip install requests beautifulsoup4
```
> **Termux note:** If you hit SSL errors, run `pkg install openssl ca-certificates` first.

**Windows (PowerShell):**
```powershell
pip install requests beautifulsoup4
```

**Optional — virtual environment (recommended):**
```bash
python3 -m venv venv
source venv/bin/activate          # Linux/macOS
# venv\Scripts\activate           # Windows
pip install -r requirements.txt
```

### 3. Run the scraper
```bash
python3 instagram_scraper.py
```
You'll be prompted:
```
Instagram username: natgeo
```

### 4. Check the output
```bash
cat media.json
```

**Example output:**
```json
{
  "3245678901234567890": {
    "id": "3245678901234567890",
    "shortcode": "C8xYzAbCdEf",
    "type": "video",
    "url": "https://scontent.cdninstagram.com/...",
    "thumb": "https://scontent.cdninstagram.com/..."
  }
}
```

---

## ⚙️ Configuration & API Endpoints

### Headers used (in `instagram_scraper.py`)
| Header | Value | Purpose |
|---|---|---|
| `User-Agent` | Android Chrome 120 | Mobile web spoofing |
| `X-IG-App-ID` | `936619743392459` | Instagram's public web app ID |
| `X-Requested-With` | `XMLHttpRequest` | AJAX-style request |
| `Referer` | `https://www.instagram.com/` | Bypass referer checks |

### Endpoints
| Purpose | Endpoint |
|---|---|
| Resolve user ID | `https://www.instagram.com/api/v1/users/web_profile_info/?username=<user>` |
| Fetch feed (GraphQL) | `https://www.instagram.com/graphql/query/` |
| Profile HTML (fallback) | `https://www.instagram.com/<username>/` |

### Tuning knobs (top of script)
```python
count = 50          # posts per GraphQL page
pages < 5           # max pagination depth (~250 posts)
time.sleep(1)       # delay between pages
```

### No credentials required
This tool works **without login** on public accounts. There are **no** API keys, tokens, or environment variables.

---

## 🛠️ Troubleshooting

| Issue | Fix |
|---|---|
| `user_id nahi mila` | Account is private, deleted, or rate-limited. Try later or from another IP. |
| Empty `media.json` | All three methods failed — likely IP-banned. Wait 15–30 min. |
| SSL error on Termux | `pkg install openssl ca-certificates` |
| `ModuleNotFoundError` | Run `pip install requests beautifulsoup4` |

---

## 📜 License

Released under the **MIT License**.  
See [`LICENSE`](LICENSE) for full text.

```
MIT License

Copyright (c) 2025 Rishi Security Labs

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## ⚠️ Disclaimer

This project is provided **strictly for educational, research, and OSINT purposes**.  
- Only targets **public** accounts.  
- Does **not** bypass authentication or access private data.  
- Users are responsible for complying with Instagram's Terms of Service and local laws.

The author is **not liable** for misuse or any resulting damages.

---

<p align="center">
  Built with 🐍 by <a href="https://github.com/rishi-security-labs">Rishi Security Labs</a>
</p>
