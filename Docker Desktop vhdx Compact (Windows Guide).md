# 🐳 Docker Desktop vhdx Compact (Windows Guide)

## 📌 কেন compact দরকার

Docker-এর সব ডাটা (image, container, volume) একটা ফাইলে থাকে: `C:\Users\HP\AppData\Local\Docker\wsl\disk\docker_data.vhdx`। ভেতরে কিছু মুছলেও এই ফাইলটা নিজে থেকে ছোট হয় না, তাই Windows জায়গা ফেরত পায় না। Compact করলে ভেতরের ফাঁকা অংশ ফাইল থেকে বাদ পড়ে।

* **প্রথম compact (২২ অক্টোবর ২০২৬):** ফাইল 74.57 GB থেকে 19.71 GB হয়েছে, আর C:-তে ফাঁকা জায়গা 13 GB থেকে বেড়ে 67.6 GB হয়েছে। আসল ডাটা তখন ছিল ~১৪ GB।
* **অল্প মুছলে (কয়েক GB) compact লাগে না:** Docker ওই খালি জায়গা আগে ভরে, তাই ফাইল আর বড় হয় না।
* **হাতে compact করুন:** বড় কিছু মোছার পরে (`docker system prune -a`, বড় volume, পুরনো project), অথবা C:-তে জায়গা কমে গেলে। এর বাইরে সাপ্তাহিক অটোমেটিক টাস্ক তো আছেই।

---

## ⏰ সাপ্তাহিক অটোমেটিক টাস্ক

প্রতি শনিবার দুপুর ১টায় Task Scheduler-এর **Docker vhdx Compact** টাস্ক নিজে থেকেই compact করে। ওই সময়ে PC বন্ধ থাকলে পরের বার PC চালু করলেই চলে।

| বিষয় | মান |
| --- | --- |
| টাস্কের নাম | `Docker vhdx Compact` |
| কখন চলে | প্রতি শনিবার 1:00 PM; মিস হলে PC চালু হওয়ার পর |
| Script | `C:\Users\HP\Scripts\docker-compact\compact-docker.ps1` |
| Log | `C:\Users\HP\Scripts\docker-compact\compact-docker.log` |
| সময় লাগে | ~৪-৫ মিনিট; এ সময় Docker ও PBX container বন্ধ থাকে |
| শর্ত | HP user login করা থাকতে হবে; টাস্ক Administrator privilege-এ চলে |

### টাস্ক যা করে, ক্রমানুসারে

1. Docker Desktop চালু আছে কিনা দেখে নেয়
2. চালু থাকলে ৩০ দিনের পুরনো build cache আর অব্যবহৃত image মুছে দেয় (নিচে বিস্তারিত)
3. ভেতরে TRIM চালায়, তারপর Docker Desktop বন্ধ করে
4. `wsl --shutdown` চালিয়ে vhdx ফাইল ছাড়া পাওয়া পর্যন্ত অপেক্ষা করে (সর্বোচ্চ ৬০ সেকেন্ড)
5. `Optimize-VHD -Mode Full` দিয়ে compact করে; সেটা ব্যর্থ হলে diskpart দিয়ে
6. আগে চালু থাকলে Docker Desktop আবার চালু করে; PBX container-গুলো `unless-stopped` policy-তে নিজেই উঠে যায়

### ৩০ দিনের পুরনো জঞ্জাল মোছা

Compact শুধু ভেতরে আগে থেকে খালি হওয়া জায়গা ফেরত দেয়। তাই compact-এর আগে টাস্কটা পুরনো জঞ্জাল মুছে দেয়:

```powershell
docker builder prune -af --filter "until=720h"   # ৩০ দিনে ব্যবহার না হওয়া build cache
docker image prune -af --filter "until=720h"     # কোনো container-এ নেই + ৩০ দিনের বেশি আগে তৈরি image
```

| মুছবে | মুছবে না |
| --- | --- |
| ৩০ দিনের বেশি ব্যবহার না হওয়া build cache | এই মাসের build cache (রোজকার build দ্রুত থাকে) |
| কোনো container (চালু বা বন্ধ) ব্যবহার করছে না, আর ৩০ দিনের বেশি আগে তৈরি, এমন image | চালু বা বন্ধ container যে image ব্যবহার করছে |
| | কোনো volume বা database, কখনোই না |

* **Image-এর ক্ষেত্রে "৩০ দিন" মানে তৈরির তারিখ, শেষ ব্যবহারের তারিখ না।** Docker image-এর শেষ ব্যবহারের সময় রাখে না।
* **উদাহরণ:** কোনো project-এ `docker compose down` দিলে তার container মুছে যায়, তখন image "অব্যবহৃত" হয়ে পড়ে। যেমন `edoburu/pgbouncer` image-টা ৪ মাস আগে তৈরি। RIZ-CRM `down` করা থাকলে শনিবার এটা মুছে যাবে, আর পরের `docker compose up`-এ আবার download বা build হবে। এতে কোনো ডাটা হারাবে না, শুধু একটু সময় লাগবে।
* **কোনো image রেখে দিতে চাইলে** সেই project অন্তত `docker compose stop` করে রাখুন, `down` না। বন্ধ container থাকলে তার image মোছে না।
* **মোছা বন্ধ করতে চাইলে** script থেকে `prune` লাইন দুটো মুছে দিন। তখন টাস্ক শুধু compact করবে।

### টাস্ক নিয়ন্ত্রণ

Administrator PowerShell-এ চালান। GUI চাইলে `taskschd.msc` খুলে Task Scheduler Library-তে টাস্কটা পাবেন।

```powershell
# এখনই একবার চালান
Start-ScheduledTask -TaskName "Docker vhdx Compact"

# সাময়িক বন্ধ / আবার চালু
Disable-ScheduledTask -TaskName "Docker vhdx Compact"
Enable-ScheduledTask  -TaskName "Docker vhdx Compact"

# দিন/সময় বদলান (উদাহরণ: রবিবার রাত ১০টা)
Set-ScheduledTask -TaskName "Docker vhdx Compact" -Trigger (New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 10:00PM)

# পুরোপুরি মুছে ফেলুন
Unregister-ScheduledTask -TaskName "Docker vhdx Compact" -Confirm:$false

# শেষ কয়েকবারের log দেখুন (Administrator লাগে না)
Get-Content C:\Users\HP\Scripts\docker-compact\compact-docker.log -Tail 20
```

---

## 🚀 হাতে compact চালানো

বড় কিছু মোছার পর শনিবারের জন্য অপেক্ষা না করে এখনই compact করতে চাইলে, Start Menu-তে PowerShell-এ right-click করে **Run as administrator** দিয়ে খুলুন। তারপর নিচের যেকোনো একটা কমান্ড চালান:

```powershell
# উপায় ১: টাস্কটা এখনই চালান (background-এ চলবে)
Start-ScheduledTask -TaskName "Docker vhdx Compact"

# উপায় ২: script সরাসরি চালান (প্রতিটা ধাপ স্ক্রিনে দেখা যাবে)
powershell -ExecutionPolicy Bypass -File C:\Users\HP\Scripts\docker-compact\compact-docker.ps1
```

* কাজ শেষ হতে ~৪-৫ মিনিট লাগে। শেষে log-এ `vhdx: <আগে> -> <পরে>` আর C:-এর ফাঁকা জায়গা লেখা থাকে।
* Docker Desktop আগে চালু থাকলে script নিজেই সেটা আবার চালু করে দেয়। আগে বন্ধ থাকলে বন্ধই থাকে।

---

## 🛠️ Script ছাড়া পুরো ম্যানুয়াল ধাপ

Script কোনো কারণে কাজ না করলে এভাবে হাতে করুন।

1. Docker Desktop বন্ধ করুন: system tray আইকনে right-click করে **Quit Docker Desktop**, অথবা `docker desktop stop`
2. PowerShell-এ `wsl --shutdown` চালান, তারপর `wsl -l -v` দিয়ে দেখুন সব distro `Stopped` কিনা
3. Administrator PowerShell-এ নিচের compact কমান্ড চালান
4. Docker Desktop আবার চালু করুন: Start Menu থেকে, অথবা `docker desktop start`

Compact কমান্ড (এই মেশিনে Hyper-V module আছে, তাই এটাই আগে):

```powershell
Optimize-VHD -Path "C:\Users\HP\AppData\Local\Docker\wsl\disk\docker_data.vhdx" -Mode Full
```

`Optimize-VHD` না পাওয়া গেলে বা error দিলে diskpart দিয়ে করুন:

```
diskpart
select vdisk file="C:\Users\HP\AppData\Local\Docker\wsl\disk\docker_data.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

---

## 📊 আগে-পরে চেক

vhdx-এর সাইজ যদি `docker system df`-এর মোট সাইজের চেয়ে অনেক বেশি হয় (যেমন ২০ GB বা তার বেশি), তাহলে compact করলে ভালো পরিমাণ জায়গা ফেরত আসবে।

```powershell
# Docker-এর ভেতরে আসল ডাটা কতটা
docker system df

# vhdx ফাইল Windows-এ কতটা জায়গা নিচ্ছে
"{0:N2} GB" -f ((Get-Item "C:\Users\HP\AppData\Local\Docker\wsl\disk\docker_data.vhdx").Length / 1GB)

# C: ড্রাইভে ফাঁকা জায়গা
"{0:N1} GB" -f ((Get-PSDrive C).Free / 1GB)

# compact-এর পর container-গুলো আবার উঠেছে কিনা
docker ps
```

---

## ⚠️ সতর্কতা

* **Docker Desktop-এর Troubleshoot → "Clean / Purge data" কখনো চাপবেন না।** এটা সব volume, মানে সব project-এর database, মুছে দেয়।
* **Volume মোছার আগে তালিকা দেখুন:** `docker volume ls -f dangling=true`। আসল ডাটা থাকে নামওয়ালা volume-এ (যেমন `riz-crm_pgdata`, `gemten-erp_pg_data`)। `docker volume prune --all` চালাবেন না, এটা অব্যবহৃত নামওয়ালা volume-ও মুছে দেয়।
* **Sparse VHD (`wsl --manage ... --set-sparse`) চালু করবেন না।** এটা এখনো experimental, আর data corruption-এর রিপোর্ট আছে ([WSL issue #13075](https://github.com/microsoft/WSL/issues/13075))। এই disk-এ সব database আছে।
* **Compact চলার সময় ~৪-৫ মিনিট PBX বন্ধ থাকে** (asterisk, pbx-web, gopbx-bridge, gopbx-voice, pbx-cloudflared)। ওই সময় চলমান কল কেটে যাবে।
* **`wsl --shutdown` Ubuntu distro-ও বন্ধ করে দেয়।** Ubuntu-তে কোনো কাজ চললে আগে save করে নিন।

---

## 🧪 সমস্যা হলে

প্রথমে log দেখুন: `Get-Content C:\Users\HP\Scripts\docker-compact\compact-docker.log -Tail 20`

| সমস্যা | কী করবেন |
| --- | --- |
| log-এ `ERROR: vhdx still locked` | Tray থেকে Docker Desktop Quit করুন, `wsl --shutdown` দিন, তারপর টাস্ক আবার চালান। এখনো locked থাকলে Task Manager-এ `vmmem` / `vmmemWSL` চলছে কিনা দেখুন |
| `Optimize-VHD` is not recognized | Hyper-V module নেই। Script নিজেই diskpart দিয়ে চেষ্টা করে; হাতে করলে উপরের diskpart ধাপ দেখুন |
| `Access is denied` | PowerShell Administrator হিসেবে খোলা হয়নি |
| Compact-এর পর Docker চালু হয়নি | Start Menu থেকে Docker Desktop চালু করুন, অথবা `docker desktop start`; তারপর `docker ps` |
| Container ওঠেনি | `docker ps -a` দিয়ে অবস্থা দেখুন; দরকার হলে সেই project folder-এ `docker compose up -d` |
| টাস্ক চলেনি | `Get-ScheduledTaskInfo -TaskName "Docker vhdx Compact"` চালিয়ে LastRunTime আর LastTaskResult দেখুন (0 = সফল)। HP user login করা ছিল কিনাও দেখুন |
| log-এ fstrim error | ক্ষতিকর না; এর পরেও compact চলে |

---

## 📜 Appendix: নতুন PC-তে আবার সেট করা

### ১. Script

নিচের কোড `C:\Users\HP\Scripts\docker-compact\compact-docker.ps1` নামে save করুন। Script-এর ভেতরে বাংলা লিখবেন না। Windows PowerShell 5.1 BOM ছাড়া UTF-8 ফাইল ঠিকমতো পড়তে পারে না, তাই বাংলা থাকলে script ভেঙে যেতে পারে।

```powershell
# Compacts Docker Desktop's docker_data.vhdx so Windows gets freed space back.
# Must run as Administrator (Optimize-VHD / diskpart). Run by the weekly
# "Docker vhdx Compact" scheduled task, or by hand from an elevated PowerShell.

$vhd       = "C:\Users\HP\AppData\Local\Docker\wsl\disk\docker_data.vhdx"
$docker    = "C:\Program Files\Docker\Docker\resources\bin\docker.exe"
$dockerApp = "C:\Program Files\Docker\Docker\Docker Desktop.exe"
$log       = Join-Path $PSScriptRoot "compact-docker.log"

function Log($msg) {
    $line = "{0:yyyy-MM-dd HH:mm:ss}  {1}" -f (Get-Date), $msg
    Add-Content -Path $log -Value $line
    Write-Host $line
}
function SizeGB { "{0:N2} GB" -f ((Get-Item $vhd).Length / 1GB) }
function IsLocked {
    try { $s = [IO.File]::Open($vhd, 'Open', 'Read', 'None'); $s.Close(); $false } catch { $true }
}

Log "===== compact start ====="
$wasRunning = (& $docker desktop status 2>$null | Out-String) -match 'running'
Log "Docker Desktop running before: $wasRunning"

if ($wasRunning) {
    # Build cache not used for 30 days; images with no container (running or stopped) created 30+ days ago.
    # Volumes are never touched.
    Log "pruning build cache and unused images older than 30 days"
    & $docker builder prune -af --filter "until=720h" 2>&1 | Select-Object -Last 1 | ForEach-Object { Log "  build cache: $_" }
    & $docker image prune -af --filter "until=720h" 2>&1 | Select-Object -Last 1 | ForEach-Object { Log "  images: $_" }
    Log "fstrim inside docker-desktop"
    wsl -d docker-desktop -u root -e sh -c "fstrim -av" 2>&1 | ForEach-Object { Log "  $_" }
    Log "stopping Docker Desktop"
    & $docker desktop stop 2>&1 | Out-Null
}
Log "wsl --shutdown"
wsl --shutdown

# Wait up to 60s for WSL to release the disk
for ($i = 0; $i -lt 12 -and (IsLocked); $i++) { Start-Sleep -Seconds 5 }

if (IsLocked) {
    Log "ERROR: vhdx still locked, compact skipped"
} else {
    $before = SizeGB
    try {
        Optimize-VHD -Path $vhd -Mode Full -ErrorAction Stop
        Log "Optimize-VHD finished"
    } catch {
        Log "Optimize-VHD failed ($_), trying diskpart"
        $dp = Join-Path $env:TEMP "compact-docker.diskpart.txt"
        "select vdisk file=`"$vhd`"`r`nattach vdisk readonly`r`ncompact vdisk`r`ndetach vdisk" |
            Set-Content -Path $dp -Encoding ascii
        diskpart /s $dp | Out-Null
        Remove-Item $dp -ErrorAction SilentlyContinue
    }
    Log "vhdx: $before -> $(SizeGB)"
}

if ($wasRunning) {
    # Launching through explorer starts Docker Desktop as the normal (non-admin) user
    Log "starting Docker Desktop"
    Start-Process explorer.exe -ArgumentList "`"$dockerApp`""
}
Log ("C: free: {0:N1} GB" -f ((Get-PSDrive C).Free / 1GB))
Log "===== compact end ====="
```

### ২. সাপ্তাহিক টাস্ক বানানো

Administrator PowerShell-এ একবার চালান:

```powershell
$script    = "C:\Users\HP\Scripts\docker-compact\compact-docker.ps1"
$action    = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File `"$script`""
$trigger   = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Saturday -At 1:00PM
$principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType Interactive -RunLevel Highest
$settings  = New-ScheduledTaskSettingsSet -StartWhenAvailable -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -ExecutionTimeLimit (New-TimeSpan -Hours 1)
Register-ScheduledTask -TaskName "Docker vhdx Compact" -Action $action -Trigger $trigger -Principal $principal -Settings $settings -Force
```

সেট করার পর `Start-ScheduledTask -TaskName "Docker vhdx Compact"` দিয়ে একবার চালিয়ে log দেখে নিন সব ঠিকমতো চলছে কিনা।

---
