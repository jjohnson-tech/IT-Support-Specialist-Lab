## 04 - Printer Troubleshooting

A loan officer called the helpdesk reporting their printer had stopped working. They were printing fine the previous day but nothing was coming out now. Other users in the department could still print to the same printer — isolating the problem to the individual user's machine. The task was to diagnose and resolve the issue using a systematic approach covering printer status, print queue, spooler service, and driver integrity.

---

## Step 1: Check Printer Status with Get-Printer

`Get-Printer` was run to list all printers installed on the machine and check their current status. Since other users could print successfully, the printer hardware was not the issue — the problem was specific to this workstation.

```powershell
Get-Printer
```

The Marketing-Printer was confirmed present and shared, using the Generic / Text Only driver on the Printerport port. No error status was visible — the printer object exists on the machine.

<img width="1918" height="727" alt="get-printer to check if users has printer" src="https://github.com/user-attachments/assets/650c0ffc-95e4-4db7-bea0-e6f8a16c7f2b" />



---

## Step 2: Check the Print Queue with Get-PrintJob

`Get-PrintJob` was run against the Marketing-Printer to check for stuck jobs in the queue. A jammed print queue prevents any new jobs from processing even if the printer hardware and driver are healthy.

```powershell
Get-PrintJob -PrinterName "Marketing-Printer"
```

The queue returned no results — no stuck jobs were present. This ruled out a jammed queue as the cause and moved troubleshooting to the next layer.

<img width="1918" height="846" alt="get-printjob printername" src="https://github.com/user-attachments/assets/ffcf737a-b126-446f-b1b3-6f9a835deb7e" />


---

## Step 3: Attempt to Clear the Queue with Remove-PrintJob

`Remove-PrintJob` was run to attempt clearing any remaining print jobs. The command initially threw an error — **"The specified server does not exist, or the server or printer name is invalid"** — because the printer name used was Marketing-Share instead of the correct name Marketing-Printer.

```powershell
Remove-PrintJob -PrinterName "Marketing-Printer" -ID *
```

**Troubleshooting Note:** This is a real-world troubleshooting moment — always verify the exact printer name from `Get-Printer` output before running follow-up commands. The error was identified, the correct printer name was confirmed, and troubleshooting continued.

<img width="1918" height="833" alt="remove printjob printer name" src="https://github.com/user-attachments/assets/f87ee25c-4acb-47ff-9e8d-73a8a3f877d0" />


---

## Step 4: Restart the Print Spooler Service

`Restart-Service Spooler` was run to restart the Windows Print Spooler service. The Print Spooler is the background service that manages all print jobs on the machine — when it becomes stuck or corrupted, no jobs can process regardless of queue status. Restarting it clears any stuck service state and is the most reliable fix when queue clearing alone does not resolve the issue.

```powershell
Restart-Service Spooler
```

No output on the restart command confirms the spooler restarted cleanly — no output on success is expected PowerShell behavior.

<img width="1918" height="862" alt="restart spooler" src="https://github.com/user-attachments/assets/2b5243a9-4d6b-498e-9b74-b27507b4f53c" />


---

## Step 5: Check Printer Drivers with Get-PrinterDriver

`Get-PrinterDriver` was run to review all installed printer drivers and their versions. Driver issues are the next most common cause of printer failure after queue and spooler problems — especially after Windows updates which can corrupt or replace drivers silently.

```powershell
Get-PrinterDriver
```

The Generic / Text Only driver was confirmed present for Marketing-Printer. If the driver version was outdated or mismatched, the fix would be to remove the printer and driver and reconnect to the print server to pull a fresh driver automatically.

<img width="1918" height="798" alt="get printerdriver" src="https://github.com/user-attachments/assets/b59b03aa-85a7-4035-b60d-9fd830514277" />


---

## Summary

| Step | Command | Purpose |
|---|---|---|
| Check printer status | Get-Printer | Confirms printer exists and is not in error state |
| Check print queue | Get-PrintJob | Identifies stuck jobs blocking new print jobs |
| Clear queue | Remove-PrintJob | Removes stuck jobs from the queue |
| Restart spooler | Restart-Service Spooler | Clears stuck service state — most reliable fix |
| Check drivers | Get-PrinterDriver | Identifies outdated or corrupted drivers |

**Printer troubleshooting order:**
1. Confirm the printer exists and is not in error state — `Get-Printer`
2. Check for stuck jobs in the queue — `Get-PrintJob`
3. Clear the queue — `Remove-PrintJob`
4. Restart the Print Spooler — `Restart-Service Spooler`
5. Check driver integrity — `Get-PrinterDriver`
6. If driver is corrupted — remove printer, remove driver, reconnect to print server

Always run `Get-Printer` first to confirm the exact printer name before running any follow-up print management commands. The Print Spooler restart is the most reliable fix for persistent printing issues that do not resolve through queue clearing alone.
