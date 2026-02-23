---
title: "bkctf — Personal Blog"
date: 2024-02-23
categories: [CTF, bkctf]
tags: [web, path-traversal, api-key-leak, file-read]
description: "Exploiting an exposed API key and absolute path traversal on a file serving endpoint to read /flag.txt."
math: false
mermaid: false
pin: false
---

**Challenge name:** My First Blog

**Category:** Web

## Challenge Description
```
I've been getting into this personal blog thing. 
It's been really fun but apparently you're not supposed to post certain info on the interwebs. 
Flag is at /flag.txt
```

## Step 1: Reconnaisance
<figure>
  <img src="/assets/images/posts/2026-02-21-bkctf/blog.png" alt="Screenshot of the blog main page">
  <figcaption style="font-style: italic; font-size: 15px;">The blog's main page.</figcaption>
</figure>

Navigating to the blog, posts were accessible at `/blog/<id>`. Visiting `/blog/3` revealed a **"[DELETED]" post** titled "Hire Me!!" containing an embedded PDF resume. Even though the post was marked as deleted, it was still fully accessible — a soft delete with no real access control.

<figure>
  <img src="/assets/images/posts/2026-02-21-bkctf/blog-3.png" alt="Screenshot of a hidden blog page">
  <figcaption style="font-style: italic; font-size: 15px;">A hidden blog page.</figcaption>
</figure>

Inspecting the page source revealed a critical HTML comment, hinting at a hidden document endpoint protected by an API key.

```html
<!-- for documents in the 'other' folder only people with the API key has access -->
```

## Step 2: Finding the API Key
Opening browser devtools and checking the **Network** tab while loading `/blog/3` revealed the **request that fetched the embedded PDF**:

```
GET /attachment?file=resume.pdf&apiKey=906392d25b3bd7d3af491799f89f6620
```

This exposed both:
- The document serving endpoint: `/attachment?file=`
- The API key: `906392d25b3bd7d3af491799f89f6620`
- 
<figure>
  <img src="/assets/images/posts/2026-02-21-bkctf/blog-request.png" alt="Screenshot of the network tab.">
  <figcaption style="font-style: italic; font-size: 15px;">The request for the embedded PDF reveals its endpoint and API key.</figcaption>
</figure>

## Step 3: Exploiting Path Traversal
The `file` parameter takes a filename and serves it from the server. Since the flag is at 
`/flag.txt` (an **absolute path** at the filesystem root), passing `/flag.txt` directly bypasses the app's working directory:

```
http://34.186.135.240:30000/attachment?file=/flag.txt&apiKey=906392d25b3bd7d3af491799f89f6620
```

Visiting the URL above returned the flag.

Note: `file=flag.txt` returns a 404 because the web app looks for it relative to its own directory (e.g. `/app/flag.txt`), which doesn't exist. Using the absolute path `/flag.txt` forces the server to read from the filesystem root instead.

## Flag
`bkctf{k3ys_in_th3_l0ck5}`