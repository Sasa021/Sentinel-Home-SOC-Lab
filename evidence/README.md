# Evidence

This folder is where Mosa puts screenshots and notes from her own lab run. Nothing in this folder is a result yet.

The file names to use are listed in [docs/screenshot-checklist.md](../docs/screenshot-checklist.md). Add the image files only after you take them. Do not draw, download, or generate pictures to fill a gap. If a step produced nothing, skip that file and say so in the report.

## Redact before you commit

Screenshots and notes from Azure and Windows often show data that should not be public. Check every file before `git add`.

Blur or crop these if they appear:

* Public and private IP addresses
* Subscription IDs and tenant IDs
* Resource IDs, because they contain the subscription ID
* Email addresses
* Real names other than the name you want on the portfolio
* Usernames that you also use outside this lab
* Passwords, keys, tokens, connection strings, and storage account keys
* Your home or workplace name if a device name shows it

A written note next to a screenshot should say "Source IP 1" or "lab account", not the real value.

Do not put secrets in file names. Do not commit `.rdp` files, CSV exports, or copies of the Security event log. Raw exports are ignored by `.gitignore` on purpose.

This lab uses your own Azure account only. Do not put employer data, customer data, or another person's personal data in this folder.
