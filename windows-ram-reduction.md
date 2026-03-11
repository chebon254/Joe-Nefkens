# Windows 10 VM - RAM Reduction Guide

Goal: Minimize Windows 10 resource usage in GNOME Boxes (Zoom-only VM).

---

## Step 1: Disable Bloat Services

Open **PowerShell as Administrator** (right-click Start → Windows PowerShell (Admin)):

```powershell
$services = @(
  "SysMain",           # Superfetch - useless in VM
  "WSearch",           # Windows Search indexing
  "DiagTrack",         # Telemetry/spying
  "dmwappushservice",  # Telemetry
  "Fax",
  "PrintSpooler",      # No printing needed
  "XblGameSave",
  "XboxNetApiSvc",
  "XblAuthManager",
  "RetailDemo",
  "MapsBroker",
  "WbioSrvc",          # Biometrics
  "lfsvc",             # Geolocation
  "SharedAccess",      # Internet Connection Sharing
  "RemoteRegistry"
)
foreach ($s in $services) {
  Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
  Set-Service -Name $s -StartupType Disabled -ErrorAction SilentlyContinue
  Write-Host "Disabled: $s"
}
```

---

## Step 2: Remove Bloatware Apps

```powershell
Get-AppxPackage *xbox* | Remove-AppxPackage
Get-AppxPackage *bing* | Remove-AppxPackage
Get-AppxPackage *skype* | Remove-AppxPackage
Get-AppxPackage *solitaire* | Remove-AppxPackage
Get-AppxPackage *3dbuilder* | Remove-AppxPackage
Get-AppxPackage *officehub* | Remove-AppxPackage
Get-AppxPackage *getstarted* | Remove-AppxPackage
Get-AppxPackage *zune* | Remove-AppxPackage
Get-AppxPackage *maps* | Remove-AppxPackage
Get-AppxPackage *camera* | Remove-AppxPackage
Get-AppxPackage *soundrecorder* | Remove-AppxPackage
Get-AppxPackage *weather* | Remove-AppxPackage
Get-AppxPackage *news* | Remove-AppxPackage
Get-AppxPackage *oneconnect* | Remove-AppxPackage
Get-AppxPackage *people* | Remove-AppxPackage
```

---

## Step 3: Disable Visual Effects (manual)

`Win + R` → type `sysdm.cpl` → **Advanced** tab → Performance **Settings** → select **"Adjust for best performance"** → OK

---

## Step 4: Disable Telemetry via Registry

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
```

---

## Step 5: Disable Startup Items

```powershell
# Check what's running at startup
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command | Format-Table
```

Then open `Task Manager → Startup` and disable everything except Zoom.

---

## Step 6: Set Power Plan to Balanced

```powershell
powercfg /setactive SCHEME_BALANCED
```

---

## Step 7: Disable Windows Update (while actively using VM)

`Win + R` → `services.msc` → find **Windows Update** → double-click → Startup type: **Manual** → Stop → OK

---

## Expected Result

After restarting the VM, idle RAM usage should drop from ~2.5GB to **~800MB–1.2GB**, leaving sufficient memory for Zoom calls (~1GB).
