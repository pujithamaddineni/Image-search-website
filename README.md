# Unsplash Image Search Website

A simple, responsive image search website that uses the Unsplash API to let users search, view, and download high-quality photos. This README explains project structure, setup, API usage, deployment tips, and best practices (including how to protect your Unsplash access key).

---

## Features

* Search photos by keyword (auto-complete optional)
* Infinite scroll or paginated results
* Photo detail view (large preview, photographer name, link to Unsplash, download button)
* Save favorite images (localStorage or backend)
* Option to switch between grid and list view
* Responsive layout for mobile / tablet / desktop
* Optional backend proxy to keep API key secret

---

## Important: Unsplash API Terms & Attribution

* You must follow Unsplash API Guidelines and the Unsplash License/Terms of Use. Always give proper credit to the photographer and link back to Unsplash when displaying images.
* Do not expose your Unsplash Access Key in public repositories or client-side code in production. For production, use a backend proxy to make API calls.

API docs: [https://unsplash.com/documentation](https://unsplash.com/documentation)

---

## Quick Start (Recommended: Frontend + Backend proxy)

### Prerequisites

* Node.js (v16+ recommended) and npm or yarn
* Unsplash Developer account and Access Key

### 1. Create a backend proxy (Express example)

**Why a proxy?** Keeps your `UNSPLASH_ACCESS_KEY` secret, avoids exposing it in frontend code, and allows you to enforce caching, rate limit handling, and request sanitization.

**Example `server/index.js`:**

```js
require('dotenv').config();
const express = require('express');
const fetch = require('node-fetch');
const app = express();
const PORT = process.env.PORT || 4000;

const UNSPLASH_KEY = process.env.UNSPLASH_ACCESS_KEY;

app.get('/api/search', async (req, res) => {
  const { q = '', page = 1, per_page = 12 } = req.query;
  try {
    const url = `https://api.unsplash.com/search/photos?query=${encodeURIComponent(q)}&page=${page}&per_page=${per_page}`;
    const response = await fetch(url, {
      headers: { Authorization: `Client-ID ${UNSPLASH_KEY}` }
    });
    const data = await response.json();
    res.json(data);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server error' });
  }
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

**.env (server):**

```
UNSPLASH_ACCESS_KEY=your_actual_access_key_here
PORT=4000
```

### 2. Frontend (React) - example fetch

**Example fetch from React:**

```js
// src/api/unsplash.js
export async function searchPhotos(query, page = 1, perPage = 12) {
  const res = await fetch(`/api/search?q=${encodeURIComponent(query)}&page=${page}&per_page=${perPage}`);
  if (!res.ok) throw new Error('Network error');
  return res.json();
}
```

Then call `searchPhotos('mountains')` from your search component and render results.

---

## Quick Start (Demo: Client-only - NOT recommended for production)

If you only want a quick demo or prototype, you can call Unsplash directly from the frontend using the access key. **Do not** commit the key to a public repo.

```js
const ACCESS_KEY = 'YOUR_ACCESS_KEY';
const url = `https://api.unsplash.com/search/photos?query=${q}&page=${page}&per_page=12&client_id=${ACCESS_KEY}`;
```

Keep this approach only for local testing or short-lived demos.

---

## UI / UX Notes

* Use lazy-loading for images (`loading="lazy"`) to improve performance.
* Show photographer name and link to Unsplash on hover or in the caption.
* Provide download link using the `links.download_location` from Unsplash API to respect tracking requirements.
* Cache results on the server for repeated queries to reduce rate-limit usage.

---

## Rate Limits & Best Practices

* Unsplash enforces rate limits on API usage (check docs for current limits). Implement caching, debouncing search input (e.g., 300–500ms), and pagination/infinite scroll.
* Use `Accept-Version` header if you rely on a specific API version.

---

## Folder Structure (suggested)

```
/ (project root)
├─ server/               # Express backend proxy
│  ├─ index.js
│  └─ .env
├─ client/               # React app (create-react-app or Vite)
│  ├─ public/
│  ├─ src/
│  │  ├─ components/
│  │  ├─ pages/
│  │  ├─ api/            # wrapper for /api calls
│  │  └─ App.jsx
│  └─ package.json
├─ README.md             # <- you are here
└─ package.json          # monorepo scripts (optional)
```

---

## Deployment

* Deploy the server (Express) to platforms like Heroku, Render, Railway, or DigitalOcean App Platform.
* Deploy the client (React) to Vercel, Netlify, or GitHub Pages, and configure it to call your server's API base URL.
* Protect environment variables in your hosting platform.

---

## Example environment variables for production (server)

```
UNSPLASH_ACCESS_KEY=live_production_key
NODE_ENV=production
PORT=4000
```

---

## Troubleshooting

* **Blank results:** Check your query, page number, and ensure the API key is valid.
* **CORS issues:** If calling Unsplash directly from the client, you may face CORS or rate-limit headers. Use the server proxy to avoid these.
* **Missing attribution:** Ensure you display photographer name and link to Unsplash.

---

## Contributing

Contributions are welcome! If you add features (e.g., collections, login with Unsplash, favorites backend), open a PR and update the README to document new env variables or endpoints.

---

## License

This project is released under the MIT License. Replace with your preferred license.

---

## Acknowledgements

* Unsplash for free photos and a great API
* Any UI library you used (Tailwind CSS, Material UI, Chakra, etc.)

