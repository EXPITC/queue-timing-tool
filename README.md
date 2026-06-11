# Queue Timing Tool

A browser-based tool that auto-redirects your tab to a queue URL at the mathematically optimal moment — so one of Queue-it's refetch polls lands exactly when the queue opens.

---

## How Queue-it works

When you open a Queue-it waiting room page:

1. The page loads and Queue-it's JavaScript initializes
2. It makes an initial API call to the Queue-it server
3. From the moment that response comes back, it schedules a **refetch every 30 seconds**

So if your tab loads at time **T**, refetches fire at **T+30s, T+60s, T+90s…**

The goal is to have one of those refetches land at the **exact millisecond the queue opens** (e.g. `12:00:00.500`). That's when the server flips the "you may proceed" flag — if your poll lands there, you get through before people whose next poll is 29 seconds later.

---

## The math

Given a target time `TARGET` and a poll interval of `30000ms`:

```
redirect_fires_at = TARGET - load_offset - N × 30000ms
```

Where:

- `N` is the largest integer such that `redirect_fires_at` is still in the future
- `load_offset` compensates for page load time (see below)

The tool picks the latest possible redirect moment so you're not waiting longer than necessary.

---

## Load offset

This is the most important tuning parameter.

The chain from "redirect fires" to "Queue-it timer starts" looks like:

```
redirect → DNS + TCP + TLS + HTML download → JS parse → Queue-it init → first API call → response
```

That whole chain takes **1–3 seconds** normally, and **3–5 seconds** on event day under heavy server load. If you don't account for this, your poll fires that many seconds late.

**Default: 1500ms.** Increase to 2000–3000ms on a big event day.

---

## Why redirect (not open new tab)

Modern browsers **block `window.open()`** when it's called from a timer callback (`setTimeout`/`setInterval`) — it requires a direct user gesture. Your auto-fire would silently do nothing at the critical moment.

`window.location.href` navigates the current tab and is **never blocked**, regardless of how it's triggered.

---

## Usage

1. Open `queue_timer.html` in your browser
2. Paste your queue URL
3. Set the target date, time, milliseconds, and UTC offset
4. Adjust **load offset** if needed (Advanced section)
5. Leave the tab open — it will auto-redirect at the right moment

Settings are saved to `localStorage` so you don't need to re-enter them on reload.

---

## Settings reference

| Field           | Description                                                 | Default |
| --------------- | ----------------------------------------------------------- | ------- |
| Queue URL       | The full queue page URL to redirect to                      | —       |
| Date            | Date the queue opens                                        | Today   |
| Time (HH:MM:SS) | Time the queue opens                                        | —       |
| .ms             | Milliseconds component of target time                       | 500     |
| UTC offset      | Your target timezone offset (e.g. `7` for WIB, `8` for SGT) | 7       |
| Poll interval   | Queue-it's refetch interval in ms                           | 30000   |
| Load offset     | How many ms early to fire redirect (page load compensation) | 1500    |

---

## Tips

- **Open this tool on every device you're using** — each device gets its own redirect moment, independently aligned
- **Bump load offset to 2000–3000ms** on big event days when servers are under load
- **Keep the tab focused** — some browsers throttle timers on backgrounded tabs, which can throw off the timing by hundreds of milliseconds
- The timeline at the bottom shows you exactly which poll will hit the target so you can verify the alignment at a glance
