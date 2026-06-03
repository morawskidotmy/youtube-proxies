# youtube-proxies

A continuously refreshed list of free public HTTP/SOCKS proxies that **actually
pass a live yt-dlp probe against YouTube**.

The list is regenerated every hour from a server running `proxymanager`: the
pool is re-fetched from dozens of public sources, every candidate is
dial-tested, and only those that successfully simulate a download of a real
YouTube video make it into [`youtube-proxies.txt`](./youtube-proxies.txt).

> [!NOTE]
> `proxymanager` itself is a **private repository** and is not publicly
> available. This repo only publishes the output it produces.

> [!NOTE]
> These are **public, untrusted** proxies. They go up and down constantly,
> may inject ads, log your traffic, or rate-limit aggressively. Use them
> for non-sensitive, easily-replayable workloads only (yt-dlp, scrapers,
> rotation pools, IP reputation tests).

## Get the latest list

```bash
curl -fsSL https://raw.githubusercontent.com/morawskidotmy/youtube-proxies/main/youtube-proxies.txt
```

The file is plain text, one proxy URL per line, sorted best-first
(`socks5://`, `socks4://`, `http://`). Lines starting with `#` are comments
and should be skipped.

## Use it with yt-dlp

Pick one and go:

```bash
proxy="$(curl -fsSL https://raw.githubusercontent.com/morawskidotmy/youtube-proxies/main/youtube-proxies.txt \
  | grep -v '^#' | grep . | shuf -n 1)"

yt-dlp --proxy "$proxy" "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

Or rotate through the whole list until one works:

```bash
while read -r proxy; do
  [[ "$proxy" =~ ^# || -z "$proxy" ]] && continue
  yt-dlp --proxy "$proxy" "$URL" && break
done < <(curl -fsSL https://raw.githubusercontent.com/morawskidotmy/youtube-proxies/main/youtube-proxies.txt)
```

## Use it from Python / requests

```python
import random, requests

raw = requests.get(
    "https://raw.githubusercontent.com/morawskidotmy/youtube-proxies/main/youtube-proxies.txt",
    timeout=10,
).text
pool = [l for l in raw.splitlines() if l and not l.startswith("#")]

proxy = random.choice(pool)
r = requests.get("https://example.com", proxies={"http": proxy, "https": proxy}, timeout=15)
```

## How fresh is it?

Look at the latest commit on this repo — it is also the timestamp of the
last successful regeneration. Updates are pushed only when the working set
actually changes; expect several pushes per day.

## How it works

```diagram
╭─────────────────╮   fetch    ╭──────────────╮   yt-dlp probe   ╭──────────────────────╮
│ ~30 public      │──────────▶│ proxymanager │─────────────────▶│ youtube-proxies.txt  │
│ proxy sources   │            │  (hourly)    │   (real video)   │  → git push (this)   │
╰─────────────────╯            ╰──────────────╯                  ╰──────────────────────╯
```

Pipeline lives in [proxymanager](https://github.com/morawskidotmy/proxymanager).
This repo only stores the output, so cloning and shipping the file is cheap.

> [!TIP]
> If you need other test targets (not YouTube), run `proxymanager` yourself —
> it supports `test-against <URL>` and writes a per-URL working-list file.
