# Lab report: Home SOC in Microsoft Sentinel

Replace every "Fill in" cell from your own lab. Delete this line when you start. Do not paste example counts, sample log lines, or someone else's screenshots.

If a query returned no rows, write "No rows". If a rule created no incident, write "No incident". Those are complete answers.

Copy the short version of your numbers into the Results table in [README.md](../README.md) so the README and this report match.

## 1. Cover

| Field | Your answer |
| --- | --- |
| Name | Mosa Mofokeng |
| Report date | Fill in |
| Lab dates | Fill in |
| Subscription offer | Free account or pay as you go. Fill in which one, and the credit left when you started if the page showed it. |
| Region | Fill in |
| Time zone used in this write up | South Africa Standard Time (UTC+2) for your narrative. Say when a timestamp is still UTC. |

## 2. Summary

Fill in after the rest of the report is done. Four to eight sentences.

* What you built.
* How long TCP 3389 was open to the internet.
* Whether failed logons arrived, in one sentence, with your count or "no rows".
* Whether any analytics rule created an incident.
* What Cost analysis showed after you deleted the resource group.
* One thing you would change.

## 3. Scope and rules

This lab used my own Azure subscription and a virtual machine I created for this purpose. I did not connect it to an employer network, a domain, or another person's data.

| Rule | What I did |
| --- | --- |
| Password storage | Fill in. Name the password manager. Do not write the password. |
| Network Level Authentication | Left on, or explain why you changed it. |
| Inbound ports besides TCP 3389 | Should be none. Fill in what the NSG showed. |
| Exposure window | Open time, close time, both in SAST. |
| Data collected | Event ID 4624 and 4625 only, unless you widened the XPath. If you widened it, say why. |

## 4. Environment

| Setting | Value you used |
| --- | --- |
| Resource group | |
| Log Analytics workspace | |
| Retention and daily cap | |
| Sentinel trial shown, yes or no | |
| Virtual machine name and size | |
| Image | |
| OS disk size and disk type | |
| Public IP SKU | |
| NSG name | |
| Data collection rule name | |
| Local admin account name | labadmin, or the name you used. Not the password. |
| Decoy local account | None, or the name. Not the password. |

## 5. Changes from the lab guide

Portal labels move. Record anything you did differently, including a size change because 1 GiB of RAM was not usable.

| Guide step | What I did instead | Why |
| --- | --- | --- |
| Fill in, or write "No changes" | | |

## 6. Collection

Describe the path in your own words: Windows Security log, Azure Monitor Agent, data collection rule, `SecurityEvent`, Sentinel.

| Check | Result |
| --- | --- |
| `auditpol` Logon setting | |
| Self test Event ID 4625 seen on the VM | Yes or no, and the logon type |
| Self test Event ID 4624 seen on the VM | Yes or no, and the logon type |
| Minutes from the self test until `SecurityEvent` showed the events | |
| Count of 4625 events in the workspace for the whole lab | |
| Count of 4624 events in the workspace for the whole lab | |
| Of the 4624 events, how many were your self test | |

Failed RDP with Network Level Authentication left on is usually logon type 3. A successful RDP session is logon type 10. Say whether your events matched that.

## 7. Hunting queries

Run the files in `kql/`. Put your numbers here.

| Query file | Time range you used | Threshold, if any | Rows returned | Note |
| --- | --- | --- | --- | --- |
| `failed-logons-4625.kql` | | | | |
| `brute-force-by-source-ip.kql` | | | | |
| `successful-logon-after-failures.kql` | | | | Mark self tests |
| `geolocation-enrichment.kql` | | Approach A, B, or C | | If you saw countries, name the count of distinct countries only. Do not paste the IP list. |

What a row meant, in your own words: Fill in.

What you would tune next time: Fill in.

## 8. Analytics rules and incidents

| Rule name | Severity | Schedule | ATT&CK techniques | Incidents created |
| --- | --- | --- | --- | --- |
| Lab many failed RDP logons from one public IP | Medium | Every 1 hour, last 1 hour | T1110, T1110.001 | |
| Lab successful RDP logon after failed logons | High | Every 15 minutes, last 2 hours | T1078, T1078.003, T1110 | |
| Lab successful RDP logon from a public IP | High | Every 15 minutes, last 15 minutes | T1133, T1078, T1021.001 | |
| Lab failed logons for one account from many public IPs | Medium | Every 1 hour, last 1 hour | T1110, T1110.003 | |

If you changed a threshold, record the old value and the new value.

For each incident, or for "No incident":

| Incident title | Time (UTC) | Self test, internet noise, or unknown success | What you did |
| --- | --- | --- | --- |
| | | | |

Unknown success means a 4624 you did not cause. The lab guide says to close port 3389 and delete the resource group. Write the time you did that.

## 9. Workbook

| Question | Answer |
| --- | --- |
| Workbook name and resource group | |
| Did the map show points, or the empty message? | |
| Did the top sources table agree with the hunting query? | |
| What is a limit of the geolocation function you noticed? | |

## 10. Exposure window and teardown

| Item | Your record |
| --- | --- |
| Planned window length | |
| Actual open time (SAST) | |
| Actual close time (SAST) | |
| Ingestion used while the port was open | Read this on Usage and estimated costs. Fill in the number the page showed. |
| Daily cap hit, yes or no | |
| Time the resource group delete finished | |
| Resources left with these names after delete | Should be none. Fill in what you searched. |

## 11. Cost

A budget alert does not stop charges. Record what Azure actually showed.

| Item | Amount and currency |
| --- | --- |
| Budget you set | |
| Review + create estimate, before you built the VM | |
| Cost analysis for the resource group, on the delete day | |
| Cost analysis the next day, if you checked | |
| Any charge you did not expect | |

## 12. Lessons learned

Fill in at least five sentences.

* What worked in the collection path.
* What failed or arrived late.
* How logon type 3 and logon type 10 changed the queries.
* What you would do differently to keep the cost down.
* What this lab hands to the KQL Detection Library, the Entra ID Identity Attack Detection lab, and the Sentinel Automated Incident Response lab.

## 13. Evidence index

| File | One sentence on what it shows | Taken, or not taken and why |
| --- | --- | --- |
| `evidence/01-account-offer.png` | | |
| `evidence/02-resource-group.png` | | |
| `evidence/03-budget.png` | | |
| `evidence/04-workspace-retention-cap.png` | | |
| `evidence/05-sentinel-enabled.png` | | |
| `evidence/06-vm-cost-estimate.png` | | |
| `evidence/07-vm-overview.png` | | |
| `evidence/08-nsg-my-ip.png` | | |
| `evidence/09-auditpol-logon.png` | | |
| `evidence/10-self-test-4625.png` | | |
| `evidence/11-dcr-xpath.png` | | |
| `evidence/12-ama-extension.png` | | |
| `evidence/13-securityevent-arrived.png` | | |
| `evidence/14-failed-logons-query.png` | | |
| `evidence/15-brute-force-query.png` | | |
| `evidence/16-success-after-failures.png` | | |
| `evidence/17-geo-enrichment.png` | | |
| `evidence/18-analytics-rules-list.png` | | |
| `evidence/19-rule-attack-mapping.png` | | |
| `evidence/20-incident.png` | | Optional |
| `evidence/21-workbook-map.png` | | |
| `evidence/22-nsg-exposure.png` | | |
| `evidence/23-cost-analysis.png` | | |
| `evidence/24-resource-group-deleted.png` | | |

## 14. Redaction check

Tick these yourself before you commit. Replace the brackets with yes or no.

* [ ] I blurred IP addresses, subscription IDs, tenant IDs, and email addresses in every screenshot I added.
* [ ] I did not commit a password, an `.rdp` file, a CSV export, or a raw event log.
* [ ] Every count in this report came from my workspace or from Cost analysis.
* [ ] I deleted `rg-home-soc-lab`, or I have written why it is still up and when I will delete it.
