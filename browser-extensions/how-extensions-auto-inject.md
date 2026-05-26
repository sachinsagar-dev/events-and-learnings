# How Browser Extensions Auto-Inject on Websites

**Date:** May 2026
**Category:** Browser Internals / Web Technologies
**Trigger:** Real-world observation — Buy-Hut extension auto-appeared on Flipkart after clearing browsing history

---

## My Observation

I logged into Flipkart on Brave Browser after deleting all browsing history. The Buy-Hut price comparison extension appeared automatically in the side panel — without me clicking or opening it manually. I wanted to understand why.

---

## Key Question

> Why did the extension appear automatically even after I cleared my browsing history?

---

## What I Learned

### 1. Clearing Browsing History ≠ Removing Extensions

Deleting browsing history only removes:
- Visited URLs
- Cache
- Cookies (sometimes)

Extensions are stored separately inside the **Browser Profile folder**, not in browsing history. So the extension was still installed, enabled, and ready to run.

---

### 2. Every Extension Has a `manifest.json`

This is the core config file that tells the browser:
- Which websites to run on
- What permissions it needs
- Which scripts to inject

Example from a price comparison extension:

```json
{
  "content_scripts": [
    {
      "matches": ["*://*.flipkart.com/*"],
      "js": ["content.js"]
    }
  ]
}
```

This means: *"Whenever Flipkart opens, inject content.js automatically."*

---

### 3. The Auto-Injection Flow

```
Open Flipkart
   ↓
Browser checks installed extensions
   ↓
Extension rules match flipkart.com
   ↓
content.js injected automatically
   ↓
Extension scans DOM for product info
   ↓
Sends data to Buy-Hut servers
   ↓
Server returns price comparison
   ↓
Side panel rendered on the page
```

---

### 4. Content Scripts Modify the Webpage

The extension injects JavaScript directly into the Flipkart page using **content scripts**. These scripts can:
- Read the DOM (product name, price, etc.)
- Create new HTML elements (the side panel)
- Style them with CSS
- Communicate with background scripts

Example of how it reads price:

```javascript
document.querySelector(".price")
```

---

### 5. Communication With Backend Servers

The extension sends product data to its own servers, which compare prices across platforms and return a result like:

```json
{
  "amazon": "₹68,499",
  "croma": "₹69,000",
  "flipkart": "₹69,999"
}
```

This result is then injected into the page as a UI panel.

---

### 6. Tech Stack Behind Browser Extensions

| Layer | Technology |
|---|---|
| Extension Logic | JavaScript |
| UI Overlay | HTML + CSS |
| Browser APIs | Chromium Extension APIs |
| Backend | Node.js / Python |
| Data Fetching | REST / GraphQL APIs |
| Cloud | AWS / GCP |

---

### 7. Key Browser Extension APIs Used

| API | Purpose |
|---|---|
| `chrome.tabs` | Detect opened tabs |
| `chrome.storage` | Save extension data locally |
| `chrome.runtime` | Background communication |
| `chrome.scripting` | Inject scripts into pages |
| `chrome.cookies` | Read session cookies (if permitted) |

---

### 8. Typical Extension File Structure

```
buyhut-extension/
 ├── manifest.json      → permissions + rules
 ├── background.js      → runs continuously in background
 ├── content.js         → injected into Flipkart page
 ├── popup.html         → UI shown when clicking extension icon
 ├── styles.css         → styles for the injected panel
 └── api-client.js      → handles server communication
```

---

### 9. How Extensions Detect E-Commerce Sites

| Method | Example |
|---|---|
| URL matching | `flipkart.com` in manifest |
| DOM detection | "Add to Cart" button found |
| Product schema | JSON-LD metadata on page |
| Page title analysis | Product name in `<title>` tag |

---

## My Personal Takeaway

A browser extension is basically a mini-program that lives permanently inside the browser — completely separate from browsing history or cookies. The moment you open a matching website, the browser silently checks installed extensions, finds a match, and injects the extension's scripts into the page within milliseconds. That's why clearing history made zero difference — the extension was never stored in history to begin with.

This made me realize how much power browser extensions actually have. Depending on permissions, they can read everything on a page, track your activity across sites, and even modify what you see. Worth being careful about which extensions you install.

---

## Related Concepts to Explore Next
- Chrome Extension Manifest V3 vs V2
- Content Security Policy (CSP) and how it restricts extensions
- How to build a simple browser extension from scratch
- Extension sandboxing and security model
