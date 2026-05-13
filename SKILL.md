---
name: cloakbrowser
description: Use when user wants anti-bot stealth browsing, website access that blocks normal Playwright, or browser automation that needs to bypass detection (Cloudflare, reCAPTCHA, FingerprintJS, etc.). CloakBrowser is a source-level patched Chromium with 57 fingerprint patches — drop-in Playwright replacement.
version: "1.0.0"
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [browser, stealth, anti-bot, playwright, web-scraping, automation]
    related_skills: [python-project-installation]
---

# CloakBrowser — Stealth Anti-Bot Browser

CloakBrowser is a **source-level patched Chromium** (C++ patches, not JS injection) that passes all major bot detection tests. It's a **drop-in Playwright replacement** — same API, zero config needed.

## When to Use

### Primary Triggers
- User asks to "browse", "access", or "scrape" a website that blocks or detects normal browsers
- Cloudflare Turnstile / reCAPTCHA / FingerprintJS / BrowserScan blocks appear
- `browser_navigate` fails due to bot detection
- User explicitly asks to use CloakBrowser
- Task involves web automation on sites with aggressive anti-bot measures

### When NOT to Use
- Normal browsing tasks where regular `browser_navigate` works fine
- Site is already accessible without detection issues
- User just wants simple page access — use `browser_navigate` first

## How It Works

CloakBrowser provides a Python and JavaScript API identical to Playwright. The key difference is **57 C++ source-level fingerprint patches** that make Chromium appear as a real user browser — no JS injection, no config tweaking, works through Chrome updates.

## Quick Start

### Method 1: Docker (Recommended — Pre-built Image)

```bash
# Verify CloakBrowser Docker image
docker run --rm cloakhq/cloakbrowser cloaktest

# Run a quick page visit
docker run --rm cloakhq/cloakbrowser \
  node -e "
    const { launch } = require('cloakbrowser');
    launch().then(async browser => {
      const page = browser.newPage();
      await page.goto('https://example.com');
      console.log(await page.title());
      await browser.close();
    });
  "
```

### Method 2: Python (Requires Stealth Chromium Binary)

```python
from cloakbrowser import launch

browser = launch()          # Downloads ~200MB stealth Chromium on first run
page = browser.new_page()
page.goto("https://example.com")
print(page.title())
browser.close()
```

### Method 3: With Humanize (Human-like Behavior)

```python
from cloakbrowser import launch

browser = launch(humanize=True)  # Bézier mouse curves, realistic typing/scroll
page = browser.new_page()
page.goto("https://example.com")
page.click("@e5")
page.type("@e3", "search query")
browser.close()
```

## Migration from Playwright

```diff
- from playwright.sync_api import sync_playwright
- pw = sync_playwright().start()
- browser = pw.chromium.launch()
+ from cloakbrowser import launch
+ browser = launch()

page = browser.new_page()
page.goto("https://example.com")
# ... rest of your code works unchanged
```

## Installation

### Python Package

```bash
# Install in the Hermes Agent venv
/usr/local/lib/hermes-agent/venv/bin/python3 -m ensurepip
/usr/local/lib/hermes-agent/venv/bin/python3 -m pip install cloakbrowser

# Or if uv is available
uv pip install --system cloakbrowser
```

**Note**: First launch downloads the stealth Chromium binary (~200MB). Behind China's firewall, this download may timeout. Use Docker method instead if binary download fails.

### Docker Image

```bash
docker pull cloakhq/cloakbrowser:latest
```

## Test Results (Verified)

All tests run via Docker with Clash TUN proxy enabled:

| Detection Site | Result |
|---------------|--------|
| bot.sannysoft.com | ✅ 56/56 ALL GREEN |
| bot.incolumitas.com | ✅ 35/36 (WEBDRIVER false positive, known) |
| BrowserScan.net | ✅ Normal: 19, Abnormal: 0 |
| deviceandbrowserinfo.com | ✅ isBot: False |

## API Reference

### Python API

```python
from cloakbrowser import launch

# Basic
browser = launch()                         # Headless stealth browser
browser = launch(headless=False)           # Visible browser
browser = launch(humanize=True)            # Human-like mouse/keyboard/scroll
browser = launch(proxy="socks5://host:port")  # Proxy support

page = browser.new_page()
page.goto(url, timeout=30000)
page.snapshot()           # Accessibility tree
page.click("@e5")         # Click by ref
page.type("@e3", "text")  # Type by ref
page.screenshot()         # Screenshot
browser.close()

# Async
from cloakbrowser import launch_async
browser = await launch_async()
```

### Docker Exec Pattern

```bash
# One-liner page fetch
docker run --rm cloakhq/cloakbrowser \
  node -e "
    const { launch } = require('cloakbrowser');
    launch().then(async b => {
      const p = b.newPage();
      await p.goto(process.argv[1]);
      console.log(p.url(), await p.title());
      await b.close();
    });
  " "https://example.com"

# With proxy (Clash TUN already routes Docker, but explicit):
docker run --rm \
  -e SOCKS_PROXY=socks5://host.docker.internal:7891 \
  cloakhq/cloakbrowser [command]
```

## Environment Requirements

### TUN Mode (Recommended)

CloakBrowser inside Docker needs outbound internet access. With Clash-for-Linux:

```bash
clashctl tun on   # Enable system-wide traffic interception
clashctl status   # Verify Tun: active
```

Without TUN, Docker traffic bypasses the proxy and connections time out.

### Node / Port Discovery for Docker Proxy

If Docker can't reach the host proxy:

```bash
# Find host gateway
ip route | grep default | awk '{print $3}'
# Usually: 172.31.0.1

# Use SOCKS5 port (7891) not HTTP (7890)
-e SOCKS_PROXY=socks5://172.31.0.1:7891
```

## Key Advantages over playwright-stealth / UC

| Feature | playwright-stealth | CloakBrowser |
|---------|------------------|--------------|
| Patches | JS injection | C++ source-level |
| Chrome update | Breaks often | Survives updates |
| reCAPTCHA v3 | ~0.1 (bot) | **0.9 (human)** |
| WebDriver flag | Detected | Not detected |
| UA string | HeadlessChrome | Chrome/146 |
| TLS fingerprint | Mismatch | Identical to real Chrome |

## Pitfalls

1. **First-run binary download**: ~200MB. Behind firewall, use Docker method.
2. **Docker proxy**: Without TUN, Docker doesn't auto-route through Clash. Use SOCKS5 explicitly.
3. **humanize=True cost**: Adds latency; enable only when behavioral detection is expected.
4. **No CAPTCHA solving**: CloakBrowser prevents CAPTCHAs from appearing; doesn't solve them.
5. **No proxy rotation**: Bring your own proxies.

## References

- GitHub: https://github.com/CloakHQ/CloakBrowser
- PyPI: https://pypi.org/project/cloakbrowser
- Docker: `cloakhq/cloakbrowser`
- Latest version: v0.3.28 (Chromium 146.0.7680.177.4)
