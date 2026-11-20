# Hunt guide: was my server probed, and is there sign it worked?

As of **2026-10-24** (source terms re-checked 2026-11-05; exploitation status updated 2026-11-19). Read [DISCLAIMER.md](../DISCLAIMER.md) first. We have not tested these commands on your systems.
Run them yourself, on a copy of the logs if you can, and adjust paths to your setup. Nothing in this package runs
anything for you.

**Two questions, kept apart.**

1. *Was this server probed?* Answered mostly from web, WAF or proxy logs. Researchers report that probing is
   widespread. A hit tells you someone, possibly a defender's scanner, sent a request.
2. *Is there sign it worked?* Answered from the server itself: PHP and web server error logs, the processes web workers
   started, new files and outbound connections. On 2026-10-26 CISA added both flaws to its Known Exploited
   Vulnerabilities catalog, so treat a server that was not updated promptly as one that may have been hit. Researchers
   also say a successful attempt can look like a failed one from outside.

## 0. Set your window

- Start no later than **2026-10-17**. Amazon Threat Intelligence reported the earliest known attempt on that date,
  four days before the fix (linked in [CREDITS.md](../CREDITS.md)).
- Most scanning was reported from **2026-10-21** onward.
- If your logs begin after that, say so in your notes: you cannot rule anything out for the earlier period.

## 1. Rule out the look-alikes first

Fastly measured that two request shapes people shared as "PHP2Shell signatures" were almost entirely **a different
campaign** (React2Shell, CVE-2025-55182): chunked multipart requests with no Content-Length, and bodies padded with
long runs of the letter A. Those requests also hit every path, including `/xmlrpc.php` and `/wp-login.php`, and carry a
`Next-Action` header, which PHP applications never use. A separate WordPress campaign (wp2shell, July 2026) uses a
boundary starting `----wp2shell`. Split any spike by those markers before you call it PHP2Shell. Fastly also warns that
User-Agent, IP address and body size can all mislead; the multipart **boundary** has been the most reliable separator.

## 2. Web, WAF or proxy logs: was it probed?

Standard access logs record the User-Agent but not the `Content-Type` header or the request body, so boundary and
field-name indicators only show up if you log them (a WAF, a reverse proxy with body logging, or a packet capture).
The commands below search what an ordinary access log holds. Replace the paths with yours.

User-Agent and URL markers (case-insensitive):

```
grep -i 'php2shell' /var/log/nginx/access.log
zgrep -i 'php2shell' /var/log/nginx/access.log.*.gz
```

The published IP addresses. Extract the `ip` and `ip-port` rows of `iocs/iocs.csv` (the port is dropped), then search
with them. `-w` stops an address matching inside a longer one, such as `192.0.2.16` inside `192.0.2.160`:

```
awk -F, '$3=="ip" || $3=="ip-port" {split($4, a, ":"); print a[1]}' iocs/iocs.csv > php2shell-ips.txt
grep -w -F -f php2shell-ips.txt /var/log/nginx/access.log
```

The `cidr` row is a whole network, not one address. Check it by hand against your own traffic first.

How to read an IP match:

- Several addresses are **shared anonymisation, VPN or hosting infrastructure**. Investigate; do not simply block.
- IP addresses are reassigned. Check the dates in the `notes` column and `review_by`.
- Fastly's advice: be careful with blocklists, because blocking VPN exits, Tor exits or mobile addresses hurts real
  users.

Boundary and field markers, **if** you log request headers or bodies (on the WAF or proxy host):

```
grep -F 'B7kQpoc618' /path/to/waf-or-proxy.log
grep -F 'B7kQpoc619' /path/to/waf-or-proxy.log
grep -E 'PHP_SESSION_UPLOAD_PROGRESS' /path/to/waf-or-proxy.log
```

`PHP_SESSION_UPLOAD_PROGRESS` (KELA lists it as a lead) is a real PHP feature, enabled by default. Applications that
report upload progress send it legitimately. It is only interesting on endpoints that do not handle uploads. Many PHP2Shell tools, notably those
that use the out-of-memory technique, do not use it at all.

## 3. Server logs: is there sign it worked?

Hunt.io, KELA and Fastly describe these server-side traces. None is conclusive alone.

**PHP and web server error logs.** Hunt.io reports bursts of PHP out-of-memory errors in PHP-FPM logs from 21 October,
and Fastly suggests checking Apache's error log for worker crashes. The first line below is PHP's own standard
out-of-memory message; the second is the Apache worker-crash code Fastly names:

```
grep -E 'Allowed memory size of [0-9]+ bytes exhausted' /var/log/php8.3-fpm.log
grep -E 'AH00051' /var/log/apache2/error.log
```

A short burst on one worker, alongside oversized multipart requests, is worth a closer look. Legitimate workloads can
also run out of memory, and a clean error log is not proof of safety.

**What the web server started.** The most direct evidence of success is a web worker starting a shell or a download
tool. KELA names `php-fpm`, `php-cgi`, `httpd` and `apache2` as the processes to watch, and CISA points the same way.
If you have process auditing, search it. A quick look at the current state:

```
ps -eo pid,ppid,user,etime,args --forest
```

Look for `sh`, `bash`, `wget` or `curl` beneath a web worker, unfamiliar outbound connections from the web server, and
anything running from `/tmp`. KELA describes a payload fetched into `/tmp` and run from there; Sophos publishes the
file name and command (linked in [CREDITS.md](../CREDITS.md)).

**Files and tasks.** Sucuri and KELA advise checking for new or unfamiliar PHP files, new scheduled tasks and new
administrator accounts. For PHP files changed since the earliest attempt:

```
find /var/www -name '*.php' -type f -newermt 2026-10-17
```

**Response cookies.** Sansec describes the memory-disclosure stage returning freed process memory to the client inside
the `PHPSESSID` response cookie, so compare the cookie values your server sent with what your application normally
sets. Valid session IDs are not limited to hex characters, so a different-looking value is a reason to look, not proof.
Sansec notes the leaked memory can contain passwords, session tokens, API keys and TLS private keys.

## 4. What the results mean

| You found | It suggests | Next step |
|---|---|---|
| A User-Agent, boundary or IP hit in access logs, no server-side trace | You were probed. Many are scanners, some defenders' | Confirm PHP is patched and workers restarted. Keep the logs |
| Memory-exhausted errors with matching oversized requests | An attempt reached PHP's request parsing | Same. Check the process and file items in section 3 |
| A shell or download tool started by a web worker, a new file in the web root, an unexpected scheduled task or admin account, or an unexplained outbound connection | Possible compromise | Treat as an incident |
| Nothing | Not proof of safety: these indicators change quickly and your logs may not record what matters | Patch anyway |

## 5. If you think it worked

Preserve first, clean later:

1. Isolate the server from the network if you can, and keep the web server, PHP and system logs unchanged.
2. Call your host or IT provider.
3. Assume anything the PHP process could read may be exposed. Sansec describes passwords, tokens and keys, so plan to
   rotate them from a known-clean machine once the server is under control.
4. Patching closes the flaw but does not remove anything an attacker already installed.

## 6. Patch and restart

The fix is a patched PHP, not an application update: **8.5.12, 8.4.27, 8.3.36 or 8.2.35**, or your operating system,
panel or hosting provider's patched package for older branches (a patched package can keep the old version number, so
read the package changelog for the CVE). After updating, **restart every PHP worker** (and Apache for `mod_php`) so the
old code stops running. KELA stresses the restart. PHP says updating is the only complete fix;
the temporary measures it lists reduce exposure but do not make an unpatched server safe.

## Detections you can use (linked, not reproduced)

Vendors have published detection rules under their own licences. See [CREDITS.md](../CREDITS.md): Sophos' firewall and
endpoint signatures, CISA's SIGMA rule, Abstract Security's Snort rules, and Elastic's and CrowdStrike's coverage notes.

## Not sure what you found?

Bring the log lines to your host or IT person first. If you want a second pair of eyes,
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) offers a free 20-minute
[visibility check](https://cyber.bansalai.com/?ref=github#book). For the business-owner version of this page, with the five
questions to ask your host, read
[PHP2Shell: What to Ask Whoever Runs Your Website](https://blog.bansalai.com/news/php2shell-website-owner-questions/?ref=github).

## Sources for this guide

Fastly (look-alikes, boundary advice, Apache worker crashes), Hunt.io (out-of-memory bursts), KELA (fingerprints, the
upload-progress field, the processes to watch, files and tasks, the restart advice), Sansec (memory disclosure in the
response cookie, update advice), CISA, and Sucuri. Amazon Threat Intelligence (earliest attempt) is cited
for that date only. Sophos, F5 Labs, VulnCheck, Datadog Security Labs and Huntress also published detailed findings;
their terms limit reuse, so we link to them and do not copy. Full links are in [CREDITS.md](../CREDITS.md).
