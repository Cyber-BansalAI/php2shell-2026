# PHP2Shell (CVE-2026-109618, CVE-2026-109619): indicators and log-hunting guide

> **Read first.** This package is aggregated from public sources by SOC-Lite by BansalAI. We have not tested it, it
> comes with no warranty, and it is not professional advice. It is as of **2026-10-24** (source terms re-checked 2026-11-05) and will go stale. See
> [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with the PHP project or any organisation credited in
> [CREDITS.md](CREDITS.md), and none of them reviewed or endorsed this package.

On 2026-10-21 the PHP project released fixes for two critical remote code execution flaws in PHP itself, nicknamed
**PHP2Shell**. Updating WordPress, plugins or any other application does not fix them. The server's PHP has to be
updated and every PHP worker restarted.

This repository collects the **indicators that security researchers have published so far** and a short guide to
searching your own logs for them.

**Not technical? Start with the plain-language version:**
[PHP2Shell: What to Ask Whoever Runs Your Website](https://blog.bansalai.com/news/php2shell-website-owner-questions/?ref=github)
on the SOC-Lite blog. It covers WordPress, WooCommerce, cPanel hosting and installed business software, and gives five
questions to put to your host or IT person.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per indicator, with the source, the date, whether two independent sources agree, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | How to search web server, PHP and process logs, what the results do and do not mean, and the look-alike traffic to rule out first |
| [`CREDITS.md`](CREDITS.md) | Everyone whose research this is built from, and what we took from each |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Where the indicators come from

**Rows:** 31 · **verified (two or more sources):** 5 · **single source:** 26

These counts match [`iocs/iocs.csv`](iocs/iocs.csv); a test checks them every time we change the file. Every value
was read on the source's own page, never from a news summary or a search snippet. The full list of what we took from
each source, and the exact wording, is in [CREDITS.md](CREDITS.md).

### Copied into `iocs.csv` (indicator values, with credit)

| Source | Rows | What we took | Link |
|---|---|---|---|
| **Hunt.io** | 16 | Scanner network, payload and botnet hosts, C2 domain families, staging servers, a scanner fingerprint | https://hunt.io/blog/php2shell-cve-2026-109618-109619-open-directories |
| **SOCRadar** | 7 | Attacker staging server, ports, probe filename and session prefixes | https://socradar.io/blog/php2shell-exploit-tooling-attacker-server |
| **KELA** | 5 | Fingerprints from the public proof-of-concept lineage, boundary prefix, markers | https://www.kelacyber.com/blog/php2shell-underground-response |
| **Fastly** | 3 | The most widespread kit's User-Agent and multipart boundary, an Apache worker-crash log line | https://www.fastly.com/blog/how-fastly-detected-and-built-a-coordinated-response-to-php2shell |
| **Total** | 31 | | |

### Linked, not copied

These publishers have indicators or detection rules of their own. We point to them instead of copying, either because
their terms limit reuse, because we could not read the page ourselves, or because detection rules carry their own
licences. Follow the links for the full, current lists.

| Source | What it has | Why we link | Link |
|---|---|---|---|
| **Arctic Wolf Labs** | Payload host and URLs, hashes, domains, scanner markers | Its terms allow personal or internal use only | https://arcticwolf.com/resources/blog/php2shell-critical-php-vulnerabilities-enable-unauthenticated-remote-code-execution |
| **Sophos Counter Threat Unit** | Source IP, payload host and URL, file path and command, hashes, C2 addresses and domains, boundary and User-Agent | Its terms need permission to reproduce and ask for links to its home page only | https://www.sophos.com/ |
| **F5 Labs** | Exploit-request patterns, User-Agents, markers, body sizes, and a long User-Agent appendix | Its terms prohibit reproduction | https://www.f5.com/labs/articles/php2shell-exploitation-attempts-observed |
| **VulnCheck** | Scanner and exploitation-attempt addresses, User-Agents, markers, upload hashes | Its terms allow personal, non-commercial use only | https://www.vulncheck.com/blog/php2shell-vulnerabilities |
| **Amazon Threat Intelligence** | The earliest known exploitation attempt and source addresses | Its terms allow personal use only and ask for home-page links only | https://aws.amazon.com/ |
| **Datadog Security Labs** | Log lines, process and response-header behaviours | Its terms allow personal, non-commercial use only | https://securitylabs.datadoghq.com/articles/php2shell-critical-vulnerabilities-in-php |
| **Huntress** | A pre-disclosure incident with its own markers | Its terms allow personal, non-commercial use only | https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities |
| **Wordfence** | A list of attacking addresses and request markers | We could not open the page to read it ourselves | https://www.wordfence.com/blog/2026/10/attacks-on-php2shell-continue-but-no-known-compromises-as-a-result-yet |
| **Sucuri** | Guidance on what to check on a server | Its terms allow personal, non-commercial use only | https://blog.sucuri.net/2026/10/critical-php-security-update-two-rce-flaws-cve-2026-109618-cve-2026-109619.html |
| **Abstract Security** | Snort detection rules | Detection rules carry their own licences | https://www.abstract.security/blog/php2shell-critical-php-rce-vulnerabilities-cve-2026-109618-and-cve-2026-109619 |

Elastic, CrowdStrike and Recorded Future also publish detection coverage. Those links are in [CREDITS.md](CREDITS.md).

### Used for context only (no rows)

The PHP project's advisories (CVE numbers, fixed versions), CISA's alert, and Sansec's write-up (which applications
are affected). Links are in [CREDITS.md](CREDITS.md).

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
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) helps small and mid-sized Central Florida businesses review
alerts and work with their IT person or MSP. [Book a free 20-minute visibility check](https://cyber.bansalai.com/?ref=github#book).
More plain-language security write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/?ref=github).
