# PHP2Shell (CVE-2026-109618, CVE-2026-109619): indicators and log-hunting guide

> **Read first.** This package is aggregated from public sources by SOC-Lite by BansalAI. We have not tested it, it
> comes with no warranty, and it is not professional advice. It is as of **2026-10-24** and will go stale. See
> [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with the PHP project or any organisation credited in
> [CREDITS.md](CREDITS.md), and none of them reviewed or endorsed this package.

On 2026-10-21 the PHP project released fixes for two critical remote code execution flaws in PHP itself, nicknamed
**PHP2Shell**. Updating WordPress, plugins or any other application does not fix them. The server's PHP has to be
updated and every PHP worker restarted.

This repository collects the **indicators that security researchers have published so far** and a short guide to
searching your own logs for them.

**Not technical? Start with the plain-language version:**
[PHP2Shell: What to Ask Whoever Runs Your Website](https://blog.bansalai.com/news/php2shell-website-owner-questions/)
on the SOC-Lite blog. It covers WordPress, WooCommerce, cPanel hosting and installed business software, and gives five
questions to put to your host or IT person.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per indicator, with the source, the date, whether two independent sources agree, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | How to search web server, PHP and process logs, what the results do and do not mean, and the look-alike traffic to rule out first |
| [`CREDITS.md`](CREDITS.md) | Everyone whose research this is built from, and what we took from each |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Read the results carefully

- **An indicator hit shows you were probed, not that you were compromised.** As of 2026-10-23, researchers reported
  widespread exploitation attempts and no confirmed successful compromise. That can change quickly.
- **Most of these indicators are cheap for an attacker to change.** User-Agent strings, multipart boundaries and
  markers belong to specific tools. Tool names changed within hours of disclosure. A clean result is not an all-clear.
- **Many hits will be defenders.** Researchers report that roughly a third of PHP2Shell-named scanning traffic
  labels itself as a defensive check. Your own vendors' scanners may appear.
- **Do not mass-block addresses.** Several published addresses are shared anonymisation, VPN or hosting infrastructure
  that also carries ordinary traffic. Investigate matches first.
- **The fix is the update.** Patched PHP is 8.5.12, 8.4.27, 8.3.36 or 8.2.35, or your operating system or panel
  vendor's patched package (which may keep the old version number). Restart PHP workers afterwards.

## Using `iocs.csv`

| Column | Meaning |
|---|---|
| `id` | Stable row ID (`PS-nnnn`) |
| `wave` | Always `php2shell` in this release |
| `type` | `ip`, `ip-port`, `cidr`, `user-agent`, `multipart-boundary`, `param`, `string`, `filename`, `session-id`, `content-length`, `uri-pattern`, `header`, `log-line`, `behaviour`, `command`, `file-path`, `url`, `domain`, `sha256`, `sha1`, `md5`, `ja4h` |
| `value` | The indicator. URLs and addresses are written as normal text, not defanged. Never open them in a browser |
| `match` | `exact`, `substring`, `prefix`, `regex` or `wildcard` |
| `evidence` | Where it would show: `access-log`, `request-header`, `request-body`, `response-header`, `response-body`, `error-log`, `process`, `file` or `network` |
| `fragile` | `yes` if the attacker can change it trivially |
| `verified` | `yes` only when two independent primary sources report it. `no` means a single source |
| `source_url`, `source_date` | Where we took it from and the date of that source |
| `fetched`, `review_by` | When we read it, and when it should be re-checked. Addresses and domains have short review dates |
| `notes` | Context, caveats and corroborating sources |

Search your own logs with the commands in the hunt guide. This package does not ship anything that runs on your
systems or collects data from them.

## Licence

The text we wrote is licensed under CC BY 4.0 ([LICENSE](LICENSE)). The indicator values and facts belong to the
research that reported them: they are credited in [CREDITS.md](CREDITS.md), and we do not relicense them. Details
and how to credit us are in [NOTICE.md](NOTICE.md).

## Where to go for the full, current picture

Sources keep updating. For lists longer than we reproduce, and for detection rules (which carry their own licences),
follow the links in [CREDITS.md](CREDITS.md).

## Help with a suspected compromise

If something in your logs looks real, preserve the logs first, isolate the server, and call your host or IT provider.
[SOC-Lite by BansalAI](https://cyber.bansalai.com/) helps small and mid-sized Central Florida businesses review
alerts and work with their IT person or MSP. [Book a free 20-minute visibility check](https://cyber.bansalai.com/#book).
More plain-language security write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/).
