# Lab guide: home SOC in Microsoft Sentinel

This is the click by click guide for the scaffold in the repository root. Do the parts in order. The virtual machine stays closed to the internet until Part 9.

Your own screenshots and numbers go in `evidence/` and in [report-template.md](report-template.md). This guide has no sample results.

Portal labels change. When a button name does not match, use the search box at the top of the Azure portal, follow the setting value in the tables, and record the label you actually used in the report.

## Time

Active work is about 6 to 8 hours. The exposure window is extra clock time. Writing the report is another 2 to 4 hours.

| Part | What you do | Active time |
| --- | --- | --- |
| 1 | Account, cost, and safety checks | 30 to 45 minutes |
| 2 | Resource group and budget | 20 minutes |
| 3 | Log Analytics, daily cap, and Sentinel | 30 minutes |
| 4 | Virtual machine, still closed to the internet | 30 to 45 minutes, then up to 20 minutes for boot |
| 5 | Windows auditing and a self test | 30 to 45 minutes |
| 6 | Data collection rule and the first `SecurityEvent` rows | 20 minutes of work, then 15 to 20 minutes of waiting |
| 7 | Hunting queries | 45 to 60 minutes |
| 8 | Four analytics rules | 45 to 60 minutes |
| 9 | Attack map workbook | 30 minutes |
| 10 | Internet exposure window | 4 to 24 hours of clock time. About 30 minutes of checks at the start and again at the end. |
| 11 | Delete the resource group | 20 minutes |

Set a phone reminder for the end of Part 10 before you start Part 10.

## 1. Before you create resources

**Time:** 30 to 45 minutes.

### 1.1 Your account

Use a subscription that belongs to you. Do not use an employer tenant. Do not put company logs, customer data, or another person's personal data in this workspace.

You need Contributor or Owner on that subscription.

**Screenshot 01.** `evidence/01-account-offer.png`

In the portal, open **Cost Management + Billing** and find the subscription offer.

* If it is still an Azure free account, write down the credit that is left and whether a spending limit is on. A spending limit can stop resources when the credit is gone. It is not a reason to skip teardown.
* If you already moved to pay as you go, Azure can charge your card for anything that is not inside a free allowance. A budget email will not stop that.

Official list of what a new free account includes: [Azure free account](https://azure.microsoft.com/pricing/purchase-options/azure-account/). Read it the day you build. The notes below are the items that surprise people on this lab.

### 1.2 What can cost money

| Resource | What to expect | What you do |
| --- | --- | --- |
| Virtual machine compute | `Standard_B1s`, `Standard_B2pts_v2`, and `Standard_B2ats_v2` are the 12 month free sizes on a new free account, 750 hours each per month. Each has 1 GiB of RAM. Any other size is billed per hour while the VM is running. | Prefer `Standard_B2ats_v2` (x64, 2 vCPU, 1 GiB). If it is missing, use `Standard_B1s`. Read the price on the size row before you select a bigger size. |
| Windows on 1 GiB | The desktop can be very slow or can fail to become ready. | Give it 20 minutes. If it never becomes usable, deallocate it the same day. A 4 GiB size such as `Standard_B2s` is easier to use. It sits outside the free size list, so Azure bills it per hour. Multiply the hourly price by the hours you plan, and decide before you resize. |
| OS disk | The free allowance for new accounts is two 64 GiB P6 disks for 12 months. Many Windows Server images ask for about 127 GiB. | Prefer a Microsoft `smalldisk` image when one is listed. Otherwise accept the larger disk only after you read the estimate. Check **Delete with VM**. |
| Public IP | New NICs use a Standard public IP. Standard addresses are static and are billed per hour until the IP resource is deleted. Basic public IP is retired, so the old free dynamic address is not available on a new VM. Stopping the VM does not delete a Standard IP. | Create it inside `rg-home-soc-lab` and delete the whole group at the end. |
| Log Analytics and Sentinel | The first 10 GB per day is free for 31 days when you enable Sentinel on a workspace, for up to 20 workspaces in the tenant. The waiver covers the analytics ingestion charge and the Sentinel analysis charge up to that daily amount. It does not cover the VM, the disk, or the public IP. | If the portal shows no trial, stop and read the price. Do not pick a commitment tier. Set a 1 GB daily cap anyway. |
| Bandwidth | Inbound transfer is free. Outbound has a monthly free allowance. Check the Bandwidth pricing page the day you build. | A short RDP window stays small. Leaving the VM up for days is what grows the bill. |
| Defender for Servers, data lake, UEBA, logic apps | Easy to turn on from a recommendation blade, and not needed here. Defender for Servers Plan 2 is paid. | Leave them off. |

Prices move. The number on **Review + create**, and the number in Cost analysis, are the numbers you report. Do not replace them with a figure from this guide.

### 1.3 Safety rules for the honeypot

* The VM is a lab host. It is not joined to a domain. It stores no personal files.
* Username `labadmin`. Password from a generator, 16 characters or more, saved only in a password manager. Do not reuse a password. Do not put it in git, in an `.rdp` file, or in a screenshot.
* Leave Network Level Authentication on. Failed RDP logons are then usually Event ID 4625 with logon type 3. A successful RDP session is Event ID 4624 with logon type 10. You do not need to weaken the server to see failures.
* Inbound TCP 3389 is the only port you will open, and only in Part 10, and only for the window you set.
* Do not give the VM a managed identity. Do not install tools you do not need.
* Do not run password lists or exploit tools against the VM. Scanners on the internet will try an open RDP port on their own. Your deliberate test is one wrong password from your own PC, then a correct logon.
* If Event ID 4624 shows a success you did not perform, go to section 10.4.

### 1.4 Names and region

Use one region for every resource so you can delete one resource group and so traffic does not cross regions.

| Item | Value |
| --- | --- |
| Region | **South Africa North** (Johannesburg). South Africa West, in Cape Town, does not offer every size. If a size or Sentinel is missing in South Africa North, pick one region that has all of them and use it for the whole lab. Write the region in the report. |
| Resource group | `rg-home-soc-lab` |
| Budget | `budget-home-soc-lab` |
| Workspace | `law-home-soc-lab` |
| Virtual machine | `vm-honeypot-rdp` |
| Virtual network | `vnet-home-soc-lab`, address space `10.10.0.0/16` |
| Subnet | `snet-honeypot`, `10.10.1.0/24` |
| Public IP | `pip-honeypot-rdp` |
| Network security group | `nsg-honeypot-rdp` if the form lets you name it. Otherwise record the name the wizard generates. |
| Data collection rule | `dcr-windows-security-honeypot` |
| Local admin | `labadmin` |

Tag every resource the wizard allows:

| Tag name | Tag value |
| --- | --- |
| purpose | `home-soc-lab` |
| delete-after | The date you will delete the group, as `YYYY-MM-DD` |

## 2. Resource group and budget

**Time:** 20 minutes.

### 2.1 Resource group

Portal search: **Resource groups** > **Create**.

| Field | Value |
| --- | --- |
| Subscription | Your subscription |
| Resource group | `rg-home-soc-lab` |
| Region | South Africa North, or the single region you chose |

Review + create > Create.

**Screenshot 02.** `evidence/02-resource-group.png`

### 2.2 Budget alert

A budget does not turn resources off. It sends email. You still delete the group in Part 11.

Portal search: **Cost Management** > **Budgets** > **Add**.

Set the scope to the resource group `rg-home-soc-lab` before you fill in the amount. If the scope stays on the whole subscription, the alert is harder to read.

| Field | Value |
| --- | --- |
| Name | `budget-home-soc-lab` |
| Reset period | Monthly |
| Creation date | Today |
| Expiration | Leave the default, or a date a month out. You will delete the lab long before that. |
| Amount | A number you can afford to lose, in your billing currency. 10 is a reasonable starting point only if that currency amount is acceptable to you. Change it if it is not. |

Alert conditions: 50 percent, 80 percent, and 100 percent of the budget. Send them to your own email. Do not put that email address in git.

**Screenshot 03.** `evidence/03-budget.png` Blur the email address.

## 3. Log Analytics and Microsoft Sentinel

**Time:** 30 minutes.

### 3.1 Workspace

Portal search: **Log Analytics workspaces** > **Create**.

| Field | Value |
| --- | --- |
| Subscription | Your subscription |
| Resource group | `rg-home-soc-lab` |
| Name | `law-home-soc-lab` |
| Region | The same region as the resource group |
| Pricing tier | Pay as you go. If you see a commitment tier (100 GB per day or any reserved capacity), do not select it. |

Review + create > Create. Wait until the workspace opens.

### 3.2 Retention and daily cap

On the workspace, open **Usage and estimated costs**.

| Setting | Value |
| --- | --- |
| Data retention | 90 days. With Sentinel enabled, 90 days of analytics retention is included. Do not turn on archive. |
| Daily cap | On |
| Daily cap amount | 1 GB |

Read the reset time on that page and write it in your notes. When the cap is reached, collection stops until the reset. That looks like "the attacks stopped" if you forget to check. The cap is there so a wrong data collection rule cannot ingest without a limit.

If retention and the cap are on different screens, take both pictures listed below.

**Screenshot 04.** `evidence/04-workspace-retention-cap.png`

Optional second file if needed: `evidence/04b-daily-cap.png`

### 3.3 Enable Sentinel

Portal search: **Microsoft Sentinel** > **Create** (or **Add**).

Select `law-home-soc-lab` > **Add**.

* If a banner says the first 31 days or the first 10 GB per day are free, leave that banner in the screenshot.
* If the page asks you to choose a commitment tier or to enable the data lake, cancel and stay on pay as you go. Do not enable UEBA.
* Do not install content hub solutions yet. This lab uses one data connector, which you add in Part 6.

**Screenshot 05.** `evidence/05-sentinel-enabled.png`

Open **Settings** or the workspace usage page again and confirm the daily cap is still 1 GB after Sentinel is enabled.

## 4. Virtual machine

**Time:** 30 to 45 minutes, plus boot.

Do not select RDP as a public inbound port in the wizard. You will add a rule for your own IP after the VM exists.

Portal search: **Virtual machines** > **Create** > **Azure virtual machine**.

### 4.1 Basics

| Field | Value |
| --- | --- |
| Subscription | Your subscription |
| Resource group | `rg-home-soc-lab` |
| Virtual machine name | `vm-honeypot-rdp` |
| Region | Same region as the workspace |
| Availability options | No infrastructure redundancy required |
| Security type | Standard. If Trusted Launch hides the free size, use Standard. |
| Image | Windows Server 2022 Datacenter with a desktop, publisher Microsoft. Prefer a `smalldisk` image when the list has one. Do not pick a Core image unless you are happy with no GUI. Do not pick a third party marketplace image. |
| VM architecture | x64 |
| Size | `Standard_B2ats_v2` (2 vCPU, 1 GiB). If it is not offered, `Standard_B1s` (1 vCPU, 1 GiB). Open **See all sizes** and read the hourly price. Do not pick `Standard_B2pts_v2` for an x64 image. That size is Arm. |
| Username | `labadmin` |
| Password | Generated, 16 characters or more, stored in your password manager |
| Public inbound ports | **None** |
| Azure Hybrid Benefit or "I already have a Windows licence" | Leave unchecked, unless you personally have Software Assurance and you intend to use it. It is not required for this lab. |

### 4.2 Disks

| Field | Value |
| --- | --- |
| OS disk size | The smallest the image allows |
| OS disk type | Standard SSD, unless the image only offers another type. Read the size in GiB. |
| Delete with VM | Checked, if the box exists |
| Data disks | None |

### 4.3 Networking

Create a new virtual network.

| Field | Value |
| --- | --- |
| Virtual network name | `vnet-home-soc-lab` |
| Address space | `10.10.0.0/16` |
| Subnet name | `snet-honeypot` |
| Subnet address range | `10.10.1.0/24` |
| Public IP name | `pip-honeypot-rdp` |
| Public IP SKU | Standard |
| Assignment | Static. Standard public IPs are static. This is a billed resource. |
| NIC network security group | Basic or Advanced, named `nsg-honeypot-rdp` if the form allows |
| Public inbound ports | None |
| Load balancing | None |
| Delete public IP and NIC with the VM | Checked, if the box exists |

Place the virtual network in `rg-home-soc-lab`, not in a new group.

### 4.4 Management, monitoring, tags

| Field | Value |
| --- | --- |
| Boot diagnostics | Disable. If the portal forces a storage account, put that account in `rg-home-soc-lab`. |
| System assigned managed identity | Off |
| Auto shutdown | Off. You delete the lab instead. Shutdown does not stop a Standard public IP charge, and it stops collection during the window. |
| Guest configuration, insights, or recommended alerts that show a price | Off |
| Tags | `purpose` = `home-soc-lab`, `delete-after` = your delete date |

### 4.5 Review

Open **Review + create** and stop.

**Screenshot 06.** `evidence/06-vm-cost-estimate.png`

Read the estimated cost. Confirm:

* Inbound ports are None
* The size is the one you chose
* The image is a Microsoft Windows Server image
* Hybrid benefit is off
* The disk size is what you expected

If the estimate is more than you want to spend, do not click Create. Change the image or the size, or stop the lab.

When you accept the estimate, click **Create**. Wait until deployment succeeds.

### 4.6 After create

Open the virtual machine. Wait until the status is Running.

**Screenshot 07.** `evidence/07-vm-overview.png` Blur the public IP address. Copy the address into your password manager so you can use it for the management rule. Do not copy it into a file in this repository.

If the status stays on Updating for more than 20 minutes, or the VM never accepts a connection in Part 5, deallocate it (Stop, and wait until the status is Stopped, deallocated). Compute billing pauses. The disk and the public IP do not. Decide whether a paid 4 GiB size is worth the hourly price. If it is not, go to Part 11 and delete the group.

### 4.7 Management NSG rule

On your own computer, note your current public IP (for example the address shown by `https://ifconfig.me`). Append `/32`.

On the VM page open **Networking**. Select the network security group.

**Add inbound port rule** with these values only.

| Field | Value |
| --- | --- |
| Source | IP Addresses |
| Source IP addresses/CIDR ranges | Your public IP and `/32` |
| Source port ranges | `*` |
| Destination | Any |
| Service | RDP |
| Destination port ranges | `3389` |
| Protocol | TCP |
| Action | Allow |
| Priority | `100` |
| Name | `Allow-RDP-MyIP` |
| Description | Management only |

Leave the default **DenyAllInBound** rule in place. Do not add rules for any other port.

**Screenshot 08.** `evidence/08-nsg-my-ip.png` Blur your IP address.

If your ISP changes your address, this rule stops matching. You can update the source, or use the portal **Run command** path in Part 5. You do not need to open the port to the internet to recover.

## 5. Windows auditing and a self test

**Time:** 30 to 45 minutes.

Connect with Remote Desktop to the public IP as `labadmin`. When the client offers to save the password, decline.

If the desktop never appears because the VM is out of memory, use **Virtual machine > Run command > RunPowerShellScript** for the `auditpol` commands below, and skip the GUI steps. Record that in the report.

### 5.1 Security log size

On the VM:

1. Open **Event Viewer** (`eventvwr.msc`).
2. **Windows Logs** > **Security** > **Properties**.
3. Maximum log size: `131072` KB.
4. When the log is full: **Overwrite events as needed**.
5. OK.

### 5.2 Logon auditing

Open **Command Prompt** as administrator and run:

```bat
auditpol /get /subcategory:"Logon"
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /get /subcategory:"Logon"
```

The last command must show Success and Failure both enabled.

**Screenshot 09.** `evidence/09-auditpol-logon.png`

Optional check in the GUI: `secpol.msc` > **Advanced Audit Policy Configuration** > **System Audit Policies** > **Logon/Logoff** > **Audit Logon** > Success and Failure. If the GUI says "Not configured" but `auditpol` shows Success and Failure, trust `auditpol`.

### 5.3 One failed logon you cause

From your PC, disconnect if you are connected. Start Remote Desktop again. Type `labadmin` and a wrong password once. You should get a failure.

Connect again with the real password so you can capture the event.

On the VM, in an administrator PowerShell window:

```powershell
$XPath = '*[System[(EventID=4624 or EventID=4625)]]'
Get-WinEvent -LogName Security -FilterXPath $XPath -MaxEvents 5 | Format-List TimeCreated, Id
```

The local XPath has no `Security!` prefix because `-LogName Security` already selects the channel. The data collection rule in Part 6 needs the prefix. They are not the same string.

You want at least one Id `4625`. A `4624` from your successful connection may be there too. Open one 4625 in Event Viewer and note the logon type. With Network Level Authentication left on, a failed RDP attempt is usually logon type 3.

**Screenshot 10.** `evidence/10-self-test-4625.png` Blur account names and addresses.

If `Get-WinEvent` returns no events, auditing is off or the failed attempt never reached the VM. Check the NSG source IP, rerun the `auditpol` set command, and try the wrong password once more. Do not open the NSG to Any to fix this.

### 5.4 Optional decoy account

This account is not a trap with a weak password. It is a normal local user so a guess of a common name is not an administrator.

1. `lusrmgr.msc` > **Users** > **New User**.
2. User name: `svc-reports`
3. Password: a new long random password in your password manager. Not the admin password.
4. Clear **User must change password at next logon**. Check **Password never expires**. The VM will be deleted.
5. Create.
6. Confirm `svc-reports` is not a member of **Administrators** or **Remote Desktop Users**.

Skip this step if you are short of time. Record "no decoy" in the report.

Disconnect from Remote Desktop when you finish. Leave the VM running.

## 6. Data collection rule

**Time:** 20 minutes, then wait 15 to 20 minutes.

Use the connector named **Windows Security Events via AMA**. Do not use the older **Security Events** connector. That older connector depends on the deprecated Log Analytics agent.

In Microsoft Sentinel open **Configuration** > **Data connectors**. Search `Windows Security Events via AMA`. Open it.

You need read and write access on the workspace. Contributor on the resource group is enough in this personal lab.

**Create data collection rule** (the button may say **+ Add data collection rule**).

| Step | Value |
| --- | --- |
| Rule name | `dcr-windows-security-honeypot` |
| Subscription | Your subscription |
| Resource group | `rg-home-soc-lab` |
| Resources | `vm-honeypot-rdp` only |
| Collect | **Custom**. Do not select All events, Common, or Minimal. Minimal includes process creation (Event ID 4688), which is noisy. |
| XPath | `Security!*[System[(EventID=4624 or EventID=4625)]]` |

Select **Add** so the XPath appears in the list. If the form rejects the `or`, add two expressions instead: `Security!*[System[(EventID=4624)]]` and `Security!*[System[(EventID=4625)]]`. Then **Review + create** > **Create**.

The wizard installs the extension `AzureMonitorWindowsAgent` on the VM if it is missing. That can take several minutes.

**Screenshot 11.** `evidence/11-dcr-xpath.png`

**Screenshot 12.** `evidence/12-ama-extension.png` from **Virtual machine > Extensions + applications**, or from the data collection rule **Resources** tab.

### 6.1 Generate fresh events

The rule collects events that happen after it is active. It does not backfill the self test you did in Part 5.

From your PC, fail one RDP password, then log on successfully, then disconnect. Same method as Part 5.

### 6.2 Confirm the table

Wait 15 to 20 minutes. In Sentinel, open **Logs**. Set the time picker to the last 4 hours. The table name is `SecurityEvent`. It is not `SecurityEvents`.

```kql
SecurityEvent
| where TimeGenerated > ago(4h)
| summarize Count = count() by EventID, LogonType
| order by EventID asc
```

**Screenshot 13.** `evidence/13-securityevent-arrived.png` Crop away IP addresses if they are on screen. A count is enough.

`TimeGenerated` is UTC. South Africa Standard Time is UTC+2, with no daylight saving.

If the query returns no rows after 30 minutes:

1. Confirm the extension status is succeeded.
2. Confirm the data collection rule lists the VM.
3. Run the local `Get-WinEvent` command again. If the VM has no local 4625, the cloud has nothing to collect.
4. Check the daily cap was not already reached. This query is safe to run in the same Logs blade:

```kql
Operation
| where OperationCategory == "Data Collection Status"
| project TimeGenerated, OperationStatus, Detail
| order by TimeGenerated desc
| take 20
```

5. Confirm you did not select the old MMA connector.

Do not switch the data collection rule to All events to "make data appear".

## 7. Hunting queries

**Time:** 45 to 60 minutes.

Open **Microsoft Sentinel** > **Logs**. Paste one file at a time from `kql/`. Run them against `law-home-soc-lab`.

The files use `ago(1d)`. Change `1d` if your lab window is shorter, and write the value you used in the report.

| File | What it does | Screenshot |
| --- | --- | --- |
| `kql/failed-logons-4625.kql` | Event ID 4625, logon type 3 or 10, public source, with a label for common SubStatus codes | **14.** `evidence/14-failed-logons-query.png` |
| `kql/brute-force-by-source-ip.kql` | Same failures, grouped by IP. Starter threshold is 20 failures. Change it only after you see your own volume, and record the new number. | **15.** `evidence/15-brute-force-query.png` |
| `kql/successful-logon-after-failures.kql` | Event ID 4624 logon type 10 after failures for the same account in the previous hour. Your self test can be the only row. | **16.** `evidence/16-success-after-failures.png` |
| `kql/geolocation-enrichment.kql` | Approach A uses `geo_info_from_ip_address()`. That is the one to run. | **17.** `evidence/17-geo-enrichment.png` |

An empty grid is a valid result. Screenshot the empty grid and write "no rows" in the report. Do not add `print` rows, a sample CSV, or a made up count.

Blur IP addresses and account names in 14, 15, 16, and 17.

### 7.1 Geolocation approaches

Approach A is enough for the workbook and for the report. The function is documented for Microsoft Sentinel. It returns country, state, city, latitude, and longitude from GeoLite2 data that ships with the function. Location is approximate. A city result is not a street address. Some addresses return an empty location. Leave those empty.

Approach B is a Sentinel watchlist, if you want the practice. Build it only from addresses you have already seen in your own query.

1. Create a CSV on your machine. Header row:

```text
SearchKey,Country,City,Latitude,Longitude
```

2. Add rows only for IPs you looked up yourself. Do not download a full city database into this lab, and do not commit the CSV. `*.csv` is in `.gitignore`.
3. In Sentinel open **Configuration** > **Watchlist** > **Add new**.
4. Name: `GeoIP`. Alias: `GeoIP`. Search key column: `SearchKey`. Upload the file.
5. In `kql/geolocation-enrichment.kql`, comment out Approach A and uncomment Approach B. Run it from the Sentinel Logs blade. `_GetWatchlist()` is a Sentinel function.

Approach C, `ipv4_lookup`, is documented for Azure Data Explorer and Microsoft Fabric. Sentinel may reject it. If you see a syntax error, stop and use Approach A or B. The commented example uses the documentation range `203.0.113.0/24` (TEST-NET-3). It is not a location feed. Replace it only with ranges you looked up, and do not commit those ranges if you are not ready to publish them.

## 8. Analytics rules

**Time:** 45 to 60 minutes.

The files in `analytics-rules/` are the definitions. The portal does not read the git files for you. Create each rule by hand. Paste the KQL that sits under `query: |` in the YAML. Do not paste the YAML itself into the query box. If the query box rejects a line that starts with `//`, delete those comment lines and keep the rest of the KQL.

**Microsoft Sentinel** > **Configuration** > **Analytics** > **Create** > **Scheduled query rule**.

Shared settings for every rule:

| Portal field | Value |
| --- | --- |
| Status | Enabled |
| Query | The KQL from that rule's YAML file |
| Alert threshold | Greater than `0` |
| Event grouping | Trigger an alert for each event |
| Suppression ("stop running the query after an alert") | Off |
| Create incidents | Enabled |
| Group related alerts | Enabled |
| Grouping lookback | 6 hours |
| Reopen closed incidents | Off |

Run the query once in the wizard. If it returns no rows, you can still save the rule. If the wizard refuses to save an empty result, widen the time range or repeat your self test. Do not insert fake rows into the rule.

Entity mapping is under alert enrichment. Custom details are optional. If the portal rejects a custom detail, skip that detail and note it in the report. The entities are the part that matters. Custom detail columns in the queries are already strings (`FailedLogonsText` and the other `*Text` columns) because Sentinel expects strings there.

Incident grouping entities are listed per rule below.

### 8.1 Many failures from one IP

File: `analytics-rules/rdp-brute-force-single-source.yaml`

| Field | Value |
| --- | --- |
| Name | `Lab many failed RDP logons from one public IP` |
| Severity | Medium |
| Tactics | Credential Access |
| Techniques | T1110, T1110.001 |
| Run query every | 1 hour |
| Lookup data from the last | 1 hour |
| Query threshold inside the KQL | `FailedLogons >= 20` |
| Entities | IP Address = `IpAddress`. Host FullName = `Computer`. |
| Group incidents by | IP |
| Custom detail | `FailedLogons` = column `FailedLogonsText` |

This rule can fire often once the port is public. Grouping by IP keeps that to one open incident per address. Raise the count of 20 only after you have seen your own volume.

### 8.2 Success after failures

File: `analytics-rules/successful-rdp-after-failures.yaml`

| Field | Value |
| --- | --- |
| Name | `Lab successful RDP logon after failed logons` |
| Severity | High |
| Tactics | Initial Access, Credential Access |
| Techniques | T1078, T1078.003, T1110 |
| Run query every | 15 minutes |
| Lookup data from the last | 2 hours |
| Entities | Account FullName = `Account`, Name = `AccountName`, NTDomain = `NTDomain`. Host FullName = `Computer`. IP Address = `IpAddress`. |
| Group incidents by | Account and IP |
| Custom detail | `FailureCount` = column `FailureCountText` |

Your self test can create this incident. Close it and write "self test" in the report. A success you do not recognise is section 10.4.

### 8.3 Any public RDP success

File: `analytics-rules/external-rdp-logon.yaml`

| Field | Value |
| --- | --- |
| Name | `Lab successful RDP logon from a public IP` |
| Severity | High |
| Tactics | Initial Access, Persistence, Lateral Movement |
| Techniques | T1133, T1078, T1021.001 |
| Run query every | 15 minutes |
| Lookup data from the last | 15 minutes |
| Entities | Same account, host, and IP mapping as rule 8.2 |
| Group incidents by | Account and IP |
| Custom detail | `LogonType` = column `LogonTypeText` |

T1021.001 on this lab means the logon used RDP. It does not mean the attacker moved through a network. Say that in the report.

This rule matches your own logons. After you know your management IP, you may add this line in the portal query only:

```kql
| where IpAddress != "YOUR.MANAGEMENT.IP"
```

Put the real address in the portal. Do not paste it back into the YAML file in git.

### 8.4 Many sources, one account

File: `analytics-rules/many-sources-one-account.yaml`

| Field | Value |
| --- | --- |
| Name | `Lab failed logons for one account from many public IPs` |
| Severity | Medium |
| Tactics | Credential Access |
| Techniques | T1110, T1110.003 |
| Run query every | 1 hour |
| Lookup data from the last | 1 hour |
| Query thresholds inside the KQL | `SourceCount >= 5` and `FailedLogons >= 20` |
| Entities | Account FullName = `Account`, Name = `AccountName`, NTDomain = `NTDomain`. Host FullName = `Computer`. There is no single IP entity, because the row is many sources. |
| Group incidents by | Account |
| Custom details | `FailedLogons` = `FailedLogonsText`, `SourceCount` = `SourceCountText` |

T1110.003 here means many public addresses failed against one account name on this VM. It is not proof of a password spray across many organisations. Write that limit in the report.

**Screenshot 18.** `evidence/18-analytics-rules-list.png`

**Screenshot 19.** `evidence/19-rule-attack-mapping.png` Open the brute force rule and show the schedule, the threshold, and the techniques.

**Screenshot 20, optional.** `evidence/20-incident.png` only if a rule has already created an incident from your events. If the list is empty, skip the file. Do not hand build an incident.

## 9. Attack map workbook

**Time:** 30 minutes.

The file `workbook/honeypot-attack-map.json` is a skeleton. It has a map of failed public logons, a table of the top sources, and a table of successful RDP logons. The map uses Approach A. Empty tiles stay empty until `SecurityEvent` has matching rows.

1. **Microsoft Sentinel** > **Workbooks** > **Add workbook**.
2. Select **Edit**, then open **Advanced Editor** (the `</>` control).
3. Replace the contents with the JSON file. Select **Apply**.
4. If Apply shows an error, do not spend long repairing the gallery JSON. Build the tiles yourself:
   * **Add** > **Add query**. Data source **Logs**. Resource type **Log Analytics**. Resource: `law-home-soc-lab`.
   * Paste the map query from the JSON (the query titled "Failed logons by source location").
   * Visualization: **Map**. Map settings: location info **Latitude/Longitude**, latitude column `Latitude`, longitude column `Longitude`, size by `FailedLogons`.
   * Add a second query for the top sources table, visualization **Table**.
   * Add a third query for successful RDP logons, visualization **Table**.
5. Set the workbook time range to the last 24 hours, or to your real window.
6. **Save**. Name: `Honeypot attack map`. Resource group: `rg-home-soc-lab`. Region: the lab region.

**Screenshot 21.** `evidence/21-workbook-map.png`

The empty state message is a valid screenshot. Do not drop points onto the map.

## 10. Open RDP to the internet

**Time:** a window you choose. Start with 4 hours. Do not go past 24 hours on the first run. Active checking is about 30 minutes at the start and 30 minutes at the end.

Do not start this part until all of these are true:

* The budget exists.
* The daily cap is 1 GB.
* Screenshot 13 shows real `SecurityEvent` rows from your self test.
* You have a phone reminder for the close time.
* You know you can delete `Allow-RDP-Exposure` quickly.

### 10.1 Open the port

On the NSG, add a second inbound rule. Leave `Allow-RDP-MyIP` as it is.

| Field | Value |
| --- | --- |
| Source | Any |
| Source port ranges | `*` |
| Destination | Any |
| Service | RDP |
| Destination port ranges | `3389` |
| Protocol | TCP |
| Action | Allow |
| Priority | `110` |
| Name | `Allow-RDP-Exposure` |

Write the open time in SAST and in UTC.

**Screenshot 22.** `evidence/22-nsg-exposure.png`

### 10.2 Check after the first hour

* **Usage and estimated costs** on the workspace. Write down the volume ingested today. If it is climbing toward 1 GB, delete `Allow-RDP-Exposure` now.
* Run `kql/failed-logons-4625.kql` and `kql/brute-force-by-source-ip.kql` again.
* You do not need to stay logged on to the VM. Staying logged on adds your own 4624 events.

### 10.3 Close the port

When the reminder fires, delete the rule `Allow-RDP-Exposure`. Confirm the inbound list no longer allows 3389 from Any. `Allow-RDP-MyIP` can stay until you delete the resource group, so you can still reach the VM if you forgot a screenshot.

Write the close time.

Optional: `evidence/22b-nsg-closed.png`

Take any missing screenshots from Parts 7 to 9 now, while the workspace still has data.

### 10.4 If a logon succeeds and it was not you

1. Delete `Allow-RDP-Exposure` immediately. If you are unsure which rule that is, set every inbound Allow on TCP 3389 to Deny, or delete those rules.
2. Do not use the VM to browse, install software, or "look around" beyond the event you need to screenshot.
3. Screenshot the 4624 row for your private notes. For the public repo, blur the IP and the account, or skip the image and describe the event without the address.
4. In the portal, look for resource groups you did not create.
5. Go to Part 11 and delete `rg-home-soc-lab`.
6. Retire the lab password in your password manager. Do not reuse it.
7. In the report, label the event as an unknown success and record the time you closed the port.

## 11. Delete the resource group

**Time:** 20 minutes, plus an optional look the next day.

Deleting the resource group deletes the virtual machine, the disk, the public IP, the NIC, the network security group, the virtual network, the data collection rule, the workspace, Sentinel on that workspace, the analytics rules, the incidents, and the workbook if you saved it in this group. Copy screenshots and notes out first. The logs do not survive.

A stopped or deallocated VM is not teardown. A Standard public IP and the OS disk keep charging until those resources are deleted.

1. Portal > **Resource groups** > `rg-home-soc-lab`.
2. **Delete resource group**.
3. Type `rg-home-soc-lab`.
4. Delete. Wait until the notification says the deletion has finished. Refresh the list.
5. Search **Virtual machines**, **Public IP addresses**, **Disks**, **Log Analytics workspaces**, and **Microsoft Sentinel**. Nothing with the names from section 1.4 should remain.
6. **Cost Management** > **Cost analysis**. Filter to `rg-home-soc-lab` if it is still offered, or group by resource group. Record the amount, including zero.

**Screenshot 23.** `evidence/23-cost-analysis.png` before you forget what the page showed. You can take it just before delete, and again the next day.

**Screenshot 24.** `evidence/24-resource-group-deleted.png`

The next day, open Cost analysis once more. A last partial hour sometimes appears late. If you take that picture, save it as `evidence/23b-cost-next-day.png` and put both figures in the report.

The budget object can stay. It has no hourly charge. Delete it if you want a clean subscription: **Cost Management** > **Budgets** > `budget-home-soc-lab` > **Delete**.

This guide does not create a resource lock. If **Delete** is greyed out, open **Locks** on the resource group, remove the lock, and delete again.

## 12. Write up

Use [report-template.md](report-template.md). Then fill the Results and Lessons learned sections in the README with the same facts, in shorter form.

Redact every screenshot using [evidence/README.md](../evidence/README.md) before `git add`.

What the later labs can reuse, once you build them:

* **KQL Detection Library.** Start from the four files in `kql/` and keep the same header style: time window, threshold, and a note when a row can be a self test.
* **Entra ID Identity Attack Detection.** Same pattern of workspace, daily cap, budget, and scheduled rules. The data source changes from `SecurityEvent` to sign in logs. Do not point that lab at an employer tenant either.
* **Sentinel Automated Incident Response.** Take incidents from the rules in `analytics-rules/` and add automation. Build that only after you have seen a real incident, or after you have recorded that none fired.

## 13. If something else breaks

| What you see | What to do |
| --- | --- |
| VM never reaches the desktop | Wait 20 minutes, then deallocate. See section 4.6. Do not leave a broken VM running overnight. |
| Locked out of RDP because your IP changed | Update `Allow-RDP-MyIP`, or use **Run command**. Do not set the source to Any to fix access. |
| `SecurityEvent` is empty | Section 6.2. Repeat the self test after the data collection rule exists. |
| Query says `ipv4_is_private` is unknown | Replace that filter with `ipv4_is_in_range` checks for `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, and `127.0.0.0/8`. |
| Map has no points but the table has rows | Some IPs have no GeoLite2 coordinates. That is an empty location, not a broken workbook. |
| Analytics wizard will not save | Read the error. The usual causes are a missing entity column, a custom detail that is not a string, or a technique that is not in the selected tactics. Fix the field. Do not delete the threshold to force a hit. |
| Cost looks wrong | Stop the exposure rule, deallocate the VM, and delete the resource group. Then read Cost analysis. Do not open a second copy of the lab beside the first one. |

## References

* [Azure free account](https://azure.microsoft.com/pricing/purchase-options/azure-account/)
* [Create free services](https://learn.microsoft.com/azure/cost-management-billing/manage/create-free-services)
* [Microsoft Sentinel billing](https://learn.microsoft.com/azure/sentinel/billing)
* [Windows security event sets, including Minimal and Custom](https://learn.microsoft.com/azure/sentinel/windows-security-event-id-reference)
* [Windows Security Events via AMA](https://learn.microsoft.com/azure/sentinel/connect-services-windows-based)
* [Scheduled analytics rules](https://learn.microsoft.com/azure/sentinel/scheduled-rules-overview)
* [Map entity fields](https://learn.microsoft.com/azure/sentinel/map-data-fields-to-entities)
* [geo_info_from_ip_address()](https://learn.microsoft.com/kusto/query/geo-info-from-ip-address-function)
* [Workbook map settings](https://learn.microsoft.com/azure/azure-monitor/visualize/workbooks-map-visualizations)
* [ATT&CK T1110](https://attack.mitre.org/techniques/T1110/), [T1078](https://attack.mitre.org/techniques/T1078/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1021.001](https://attack.mitre.org/techniques/T1021/001/)
