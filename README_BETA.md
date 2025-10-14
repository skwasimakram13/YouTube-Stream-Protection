# YouTube Bot / Fake View Protection — Advanced Guide (Windows)

## 4. ⚡ Limit Access with Dynamic Push — Detailed Steps (Windows)

### Goal
Make it harder for attackers to predict and reuse your ingest by rotating the RTMP push target (e.g., switching between a.rtmp.youtube.com/live2/KEY, b.rtmp.youtube.com/live2/KEY, or multiple keys). This is done on the relay (nginx) side so OBS never needs to change.

### Why This Helps
SMM panels attempt repeated pushes or scrape known ingest endpoints. If your relay regularly rotates which YouTube ingest (host/key) it uses, attackers can’t reliably hit the active ingest.

Rotation can be scheduled (hourly/random) or triggered (by the bot-detection monitor).

### What You’ll Need
- `C:\nginx\conf\nginx.conf` (your existing nginx config)
- A file with spare YouTube keys: `C:\stream\keys.txt` (one key per line)
- PowerShell (built-in on Windows)
- Admin rights to reload nginx and edit config

### Safety Notes
- Always keep a backup copy of nginx.conf before scripts edit it.
- Use `nginx -s reload` to apply config changes without stopping nginx.
- Keep keys file secure (NTFS permissions).

---

## A — Simple Deterministic Rotation (PowerShell Script)

This rotates keys sequentially (useful if you have KEY_A, KEY_B, KEY_C).

**Create file:** `C:\stream\rotate_key.ps1`

```powershell
# rotate_key.ps1
# Sequentially rotate YouTube stream key in nginx.conf and reload nginx.
# Paths - adjust to your setup:
$nginxConf = "C:\nginx\conf\nginx.conf"
$keysFile  = "C:\stream\keys.txt"       # one key per line
$nginxExe  = "C:\nginx\nginx.exe"

# Backup
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Copy-Item $nginxConf "$nginxConf.bak.$timestamp"

# Read keys
$keys = Get-Content $keysFile | Where-Object { $_.Trim() -ne "" }
if ($keys.Count -eq 0) { Write-Error "No keys found in $keysFile"; exit 1 }

# Get current key from nginx.conf
$conf = Get-Content $nginxConf -Raw
if ($conf -match "push rtmp://[^\s;]+/live2/([A-Za-z0-9_-]+);") {
    $current = $matches[1]
} else {
    $current = $null
}

# Determine next key
if ($current -and ($keys -contains $current)) {
    $idx = [Array]::IndexOf($keys, $current)
    $next = $keys[($idx + 1) % $keys.Count]
} else {
    $next = $keys[0]
}

# Replace push line(s)
$pattern = "push rtmp://[^\s;]+/live2/[A-Za-z0-9_-]+;"
$replacement = "push rtmp://a.rtmp.youtube.com/live2/$next;"
$newConf = [regex]::Replace($conf, $pattern, $replacement, 1)

Set-Content -Path $nginxConf -Value $newConf -Force
Write-Output "Switched key to $next in $nginxConf"

# Reload nginx
Start-Process -FilePath $nginxExe -ArgumentList "-s","reload" -NoNewWindow -Wait
Write-Output "nginx reloaded"
```

### How to Use
1. Create `C:\stream\keys.txt` with each YouTube key on its own line.  
2. Run PowerShell as Administrator:  
   ```powershell
   cd C:\stream
   .\rotate_key.ps1
   ```  
3. Check nginx logs and YouTube Live Control Room to confirm new session started.

---

## B — Randomized Rotation (Harder to Predict)

**File:** `C:\stream\rotate_key_random.ps1`

```powershell
# rotate_key_random.ps1
$nginxConf = "C:\nginx\conf\nginx.conf"
$keysFile  = "C:\stream\keys.txt"
$nginxExe  = "C:\nginx\nginx.exe"

Copy-Item $nginxConf "$nginxConf.bak.$((Get-Date).ToString('yyyyMMdd_HHmmss'))"

$keys = Get-Content $keysFile | Where-Object { $_.Trim() -ne "" }
if ($keys.Count -eq 0) { Write-Error "No keys found"; exit 1 }

$rand = Get-Random -Minimum 0 -Maximum $keys.Count
$next = $keys[$rand]

$pattern = "push rtmp://[^\s;]+/live2/[A-Za-z0-9_-]+;"
$replacement = "push rtmp://a.rtmp.youtube.com/live2/$next;"
$conf = Get-Content $nginxConf -Raw
$newConf = [regex]::Replace($conf, $pattern, $replacement, 1)
Set-Content -Path $nginxConf -Value $newConf -Force

Start-Process -FilePath $nginxExe -ArgumentList "-s","reload" -NoNewWindow -Wait
Write-Output "Randomly switched to $next"
```

---

## C — Schedule Automatic Rotation (Task Scheduler)

1. Open **Task Scheduler** → Create Task.  
2. Name: **RotateStreamKey**  
3. Security options:  
   - Run whether user is logged on or not  
   - Run with highest privileges  
4. Trigger: New → Daily → Repeat task every **1 hour** indefinitely.  
5. Action:  
   - Program/script: `powershell.exe`  
   - Arguments: `-ExecutionPolicy Bypass -File "C:\stream\rotate_key.ps1"`  
6. Save task and confirm with admin password.  

---

## D — Optional: Rotate Whole Push Host
Alternate between a.rtmp and b.rtmp hosts to further confuse bots:

```powershell
$hosts = @("a.rtmp.youtube.com","b.rtmp.youtube.com")
$host = $hosts | Get-Random
$replacement = "push rtmp://$host/live2/$next;"
```

---

## E — Best Practices
- Keep only 3–6 active stream keys.
- Secure `keys.txt` (NTFS permissions).
- Backup nginx.conf before any script edit.
- Don’t rotate too often — every 10–60 minutes is safe.
- If rotating mid-stream creates a new YouTube video ID, use it only as last resort.

---

## 5. 🧩 Use YouTube API to Monitor View Source — Detailed Steps (Windows + Python)

### Goal
Automatically detect suspicious spikes (concurrent viewers, short watch time, or odd traffic sources) and alert or trigger rotation.

### What This Script Does
1. Authenticates with your YouTube channel via OAuth2.  
2. Finds the active broadcast.  
3. Monitors live concurrent viewers.  
4. Optionally analyzes traffic source.  
5. Auto-rotates RTMP key or sends Discord alert when spike detected.

---

## A — Setup Google Cloud Credentials
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project.  
3. Enable:
   - YouTube Data API v3  
   - YouTube Analytics API  
4. Create OAuth 2.0 credentials → Desktop app.  
5. Download as `C:\stream\credentials.json`.

---

## B — Python Monitor Script

**File:** `C:\stream\yt_monitor.py`

*(Full script from conversation retained here; see file for complete version.)*

Run it using:
```bash
python C:\stream\yt_monitor.py
```

Authorize on first run; token.pickle will store refresh token.

---

## F — Tuning Thresholds (Recommended)
- POLL_INTERVAL: 30s  
- MIN_CONCURRENT_FOR_ACTION: 200  
- CONCURRENT_SPIKE_FACTOR: 5.0  
- COOLDOWN: 600s  

---

## ✅ Final Checklist
- [ ] Backup nginx.conf  
- [ ] Secure keys.txt  
- [ ] Test rotate_key.ps1 manually  
- [ ] Install Python dependencies  
- [ ] Authorize OAuth  
- [ ] Run monitor in alert-only mode first  
- [ ] Enable auto-rotation later if stable  
