# YouTube Stream Protection (Anti-Bot View System for OBS)

---
[![Help Gaza](http://skwasimakram.com/upload/help-gaza.svg)](https://www.muslimglobalrelief.org/gaza-emergency-appeal/)
---

## 🎯 Problem
Someone is using SMM panels to send **bot/fake live views** to your YouTube stream.  
These fake views can:
- Trigger YouTube’s anti-spam system.
- Lead to view freezing or even demonetization.
- Disrupt your analytics and genuine engagement.

You do **daily public gaming live streams** using **OBS on Windows** and want a safe, professional setup to handle this issue.

---

## ✅ Goal
Create a streaming setup that:
- Lets you **continue the same stream** even if attacked.
- Or optionally **switch to a new stream safely** if necessary.
- Uses a **private RTMP relay** as a “shield” between OBS and YouTube.

---

## ⚙️ Prerequisites
You’ll need:

| Tool | Purpose |
|------|----------|
| **OBS Studio** | To broadcast your gameplay |
| **Nginx with RTMP module** | To act as your private relay server |
| **YouTube Stream Key(s)** | From your YouTube Live Dashboard |
| **Windows 10/11** | Your streaming environment |
| (Optional) Static IP or DDNS | For consistent relay connection |

---

## 🧩 1. Continue Same Stream (No Break, Same URL)

This keeps your stream **running under the same YouTube URL** even if you switch RTMP relay inputs.

### Step-by-Step:

1. **Install Nginx + RTMP Module**
   - Download prebuilt package:  
     👉 [https://nginx.org/en/download.html](https://nginx.org/en/download.html)
   - Extract to: `C:\nginx`
   - Edit config file: `C:\nginx\conf\nginx.conf`

2. **Add RTMP Relay Block**
   ```nginx
   rtmp {
       server {
           listen 1935;
           chunk_size 4096;

           application live {
               live on;
               record off;

               # Primary YouTube ingest
               push rtmp://a.rtmp.youtube.com/live2/YOUR_STREAM_KEY;
           }

           # Backup relay (you can create more apps if needed)
           application backup {
               live on;
               record off;

               push rtmp://b.rtmp.youtube.com/live2/YOUR_STREAM_KEY;
           }
       }
   }
   ```

3. **Start Nginx**
   - Open Command Prompt in `C:\nginx`
   - Run: `start nginx`
   - Test: Visit `rtmp://localhost/live` in OBS custom server setting.

4. **Set OBS Stream URL**
   - **Server:** `rtmp://localhost/live`
   - **Stream Key:** `main` (or whatever app name you used)

5. **Monitor**
   - If SMM bots flood the stream, **switch the push target** in `nginx.conf` to another region (e.g., `b.rtmp.youtube.com`).
   - Reload Nginx without ending stream:
     ```
     nginx -s reload
     ```
   ✅ YouTube keeps same live URL and continues the stream seamlessly.

---

## 🔄 2. Start a New Stream Intentionally (New Key = New Stream)

Use this when the attack is too heavy or analytics get stuck.

### Steps:

1. Go to **YouTube Live Dashboard → Stream Settings**.
2. Click **"Reset Stream Key"** → Copy the new key.
3. Update `nginx.conf` → Replace the old key with the new one.
4. Run:
   ```
   nginx -s reload
   ```
5. Stop streaming in OBS for a few seconds, then click **Start Streaming** again.

📌 Result:  
A **new stream** begins with a **fresh URL** — clean analytics, zero fake viewers.

---

## 🧠 3. Practical Hybrid Setup (Recommended)

Use **both** methods smartly:
- Default: Stream through your private RTMP (`localhost/live`).
- Backup: Have multiple relay apps pre-configured for quick switch.

### Example Nginx Config

```nginx
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application main {
            live on;
            record off;
            push rtmp://a.rtmp.youtube.com/live2/YOUR_KEY1;
        }

        application alt1 {
            live on;
            record off;
            push rtmp://b.rtmp.youtube.com/live2/YOUR_KEY1;
        }

        application alt2 {
            live on;
            record off;
            push rtmp://a.rtmp.youtube.com/live2/YOUR_KEY2;
        }
    }
}
```

Then:
- **OBS → Server:** `rtmp://localhost/main`
- If attacked → Run:
  ```
  nginx -s reload
  ```
  or switch OBS input to:
  - `rtmp://localhost/alt1` (same key = same stream)
  - `rtmp://localhost/alt2` (new key = new stream)

---

## 🧭 Extra Tips

- Use **Live Control Room → Analytics → Concurrent viewers** to detect bot surge.
- Use **“Unlisted” test streams** to experiment safely.
- Use a **VPN or proxy for relay** if your IP is being targeted.
- Don’t expose your **actual YouTube RTMP URL or key** publicly.
- Use YouTube Live Redirect feature (you can set it up in advance!)
   → It automatically sends everyone from the ended stream to your new one.
- Disable “Allow embedding”: → Prevents bots from viewing your stream through hidden iframes (a common SMM method).
(Go to Channel Settings → Advanced → uncheck "Allow embedding")

---

## 🧰 Folder Example (Windows)
```
C:\nginx\
├── conf\
│   └── nginx.conf
├── logs\
└── nginx.exe
```

---

## 🧾 License
This guide is open for personal and educational use.  
No resale or commercial redistribution permitted.

---

# Contribution Guide
- Fork repository
- Create branch: `feature/xxx` or `bugfix/yyy`
- Follow code style
- Create PR with description and linked issue
- CI must pass before merge


# License & Contact
- MIT License — you may use and modify the code for your organization. Include attribution if you redistribute.
- For commercial / closed-source product consider proprietary license.
- This guide is open for personal and educational use.
- No resale or commercial redistribution permitted.

**Contact**: Project owner / maintainer - hello@skwasimakram.com

---
## Author
**Develope By** - [Sk Wasim Akram](https://github.com/skwasimakram13)

- 👨‍💻 All of my projects are available at [https://skwasimakram.com](https://skwasimakram.com)

- 📝 I regularly write articles on [https://blog.skwasimakram.com](https://blog.skwasimakram.com)

- 📫 How to reach me **hello@skwasimakram.com**

- 🧑‍💻 Google Developer Profile [https://g.dev/skwasimakram](https://g.dev/skwasimakram)

- 📲 LinkedIn [https://www.linkedin.com/in/sk-wasim-akram](https://www.linkedin.com/in/sk-wasim-akram)

---

💡 *Built with ❤️ and creativity by Wassu.*
