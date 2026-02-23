---
title: "bkctf — Bridge OSINT challenge"
date: 2024-02-23
categories: [CTF, bkctf]
tags: [osint, photo-geolocation, google-maps, reverse-image-search]
description: "Identifying a bridge in Battambang, Cambodia using reverse image search, landmark recognition, and a clever music reference."
math: false
mermaid: false
pin: false
---

**Challenge name:** more like oh shit amirite

**Category:** Osint

## Challenge Description
We're given a photo taken from an unknown bridge overlooking a calm urban river, and asked to identify the bridge. The flag format confirms we need the bridge's name.

**Challenge text**
```
I love listening to the Dead Kennedy's while looking at the water.

Flag format is the name of the bridge I'm on: bkctf{golden_gate_bridge}
```
<img src="/assets/images/posts/2026-02-21-bkctf/river.jpg" alt="Photo taken from an unknown bridge overlooking a calm urban river" width="500">

## Step 1: Reverse Image Search
First, we conduct a **reverse image search** of the photo on Google Lens, which returns results suggesting that the river is the **Sangker River** flowing through **Battambang, Cambodia**.

## Step 2: Confirming the Country -- The Dead Kennedys Hint
The challenge text mentions "I love listening to the Dead Kennedys while looking at the water." This is a nod to the Dead Kennedys song **"Holiday in Cambodia"** — a subtle confirmation that we're looking at Cambodia, not Vietnam or another Southeast Asian country.

## Step 3: Identifying the Landmark Building
Looking at the right bank of the river in the photo, there's a **distinctive white multi-story building** with partial signage visible — readable as **"King"**. Searching Google Maps around the Sangker River in Battambang for hotels matching this description leads to **King Fy Hotel**, whose appearance matches the building in the photo.

## Step 4: Finding the Bridge
With King Fy Hotel pinpointed on the map, the next step is identifying which bridge the photo was taken from. The photo is clearly shot from a bridge looking down the river, with the hotel visible on the right bank. Checking the bridges nearest to King Fy Hotel on Google Maps and Street View reveals Sor Kheng Bridge (also spelled Sar Kheng Bridge) as the match.

## Flag

`bkctf{sor_kheng_bridge}`