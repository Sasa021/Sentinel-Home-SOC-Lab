# Screenshot checklist

Take these yourself while you follow [lab-guide.md](lab-guide.md). Save them under `evidence/` with the file names below. Redact first. The rules are in [evidence/README.md](../evidence/README.md).

Blur public IP addresses, private IP addresses, subscription IDs, tenant IDs, email addresses, and passwords. A cropped count is better than a wide table full of addresses.

If a step does not produce the thing in the picture, do not invent it. Skip the optional files and write "not taken" plus the reason in the report.

| No. | File | What it must show |
| --- | --- | --- |
| 01 | `evidence/01-account-offer.png` | The subscription offer. Show the free account credit if you still have it, or show that the subscription is pay as you go. Include the spending limit state when the page shows it. |
| 02 | `evidence/02-resource-group.png` | Resource group `rg-home-soc-lab` and the region you used. |
| 03 | `evidence/03-budget.png` | Budget name, scope set to that resource group, the amount you chose, and alert thresholds at 50, 80, and 100 percent. |
| 04 | `evidence/04-workspace-retention-cap.png` | Workspace `law-home-soc-lab`, retention (90 days unless you set another value), and the daily cap at 1 GB. If the cap is on a second screen, also save `evidence/04b-daily-cap.png` with the cap value and the reset time. |
| 05 | `evidence/05-sentinel-enabled.png` | Microsoft Sentinel open against `law-home-soc-lab`. Include the trial banner when the portal shows one. An empty incident list is fine. |
| 06 | `evidence/06-vm-cost-estimate.png` | The virtual machine **Review + create** page before you click Create. The estimated cost, the size, and the image name must be visible. |
| 07 | `evidence/07-vm-overview.png` | The virtual machine overview after create: name, size, region, resource group, and a public IP resource. Blur the address. |
| 08 | `evidence/08-nsg-my-ip.png` | Inbound rule `Allow-RDP-MyIP`: TCP 3389, source is one `/32`, action Allow. The default deny inbound rule should still be in the list. Blur your address. |
| 09 | `evidence/09-auditpol-logon.png` | `auditpol /get /subcategory:"Logon"` with Success and Failure both enabled. |
| 10 | `evidence/10-self-test-4625.png` | One failed logon you caused yourself, in Event Viewer or in the local PowerShell output. Blur the account name and any address. |
| 11 | `evidence/11-dcr-xpath.png` | The data collection rule Collect step set to Custom, with XPath `Security!*[System[(EventID=4624 or EventID=4625)]]` visible. |
| 12 | `evidence/12-ama-extension.png` | `AzureMonitorWindowsAgent` on the virtual machine with a succeeded status, or the data collection rule Resources tab showing `vm-honeypot-rdp`. |
| 13 | `evidence/13-securityevent-arrived.png` | A Logs query that counts Event ID 4624 and 4625 after the agent has reported. Crop to the counts if the rows contain addresses. Do not substitute a typed table. |
| 14 | `evidence/14-failed-logons-query.png` | `kql/failed-logons-4625.kql` in the query window, with the result or the empty result. Blur addresses and account names. |
| 15 | `evidence/15-brute-force-query.png` | `kql/brute-force-by-source-ip.kql`, including the threshold line, with the result or the empty result. |
| 16 | `evidence/16-success-after-failures.png` | `kql/successful-logon-after-failures.kql`, with the result or the empty result. If the only success is your self test, say that in the report, not only in the file name. |
| 17 | `evidence/17-geo-enrichment.png` | Geolocation approach A, or B if you built the watchlist. Show the query and the result or the empty result. Blur IP addresses. A country value is optional. Empty locations stay empty. |
| 18 | `evidence/18-analytics-rules-list.png` | The Analytics blade with all four lab rules enabled. |
| 19 | `evidence/19-rule-attack-mapping.png` | One rule open far enough to see the severity, the schedule, the threshold, and the ATT&CK tactics and techniques. Use the brute force rule unless you have a reason to show another. |
| 20 | `evidence/20-incident.png` | Optional. Only if a rule created an incident from your events. Show the title, severity, time, and entities. Blur addresses and account names. If nothing fired, do not add this file. |
| 21 | `evidence/21-workbook-map.png` | The saved workbook. Show the time range and the map tile. The empty message is a valid picture. Do not add pins by hand. |
| 22 | `evidence/22-nsg-exposure.png` | Inbound rule `Allow-RDP-Exposure`: source Any, TCP 3389, action Allow. Take it when you open the window. Write the open and close times in the report. Optional close picture: `evidence/22b-nsg-closed.png` after you delete that rule. |
| 23 | `evidence/23-cost-analysis.png` | Cost analysis scoped to `rg-home-soc-lab`. The figure can be zero. If you check again the next day, add `evidence/23b-cost-next-day.png`. |
| 24 | `evidence/24-resource-group-deleted.png` | The resource group list after delete, without `rg-home-soc-lab`, or the portal notification once deletion has finished. |

## Order

You will not take them in one sitting. The lab guide names the screenshot at the step that produces it. Take 22 before you delete the resource group. Take 23 before delete, and again the next day if you can. Take 24 last.

## Caption

In the report, under each file name, add one sentence: what the screen shows, and whether any row is a self test. Do not put the unredacted IP address in that sentence.
