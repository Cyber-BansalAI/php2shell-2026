# Credits and sources

<!-- DRAFT 2026-10-24. Public file. Before publishing, check every entry against docs/research/source-licences.md,
     and check that every source_url in iocs/iocs.csv appears below (publish gate, tests/test_iocs.py). -->

This package is aggregated from public research by the organisations and people below. **We thank them for
publishing.** Each indicator in `iocs/iocs.csv` names the source it came from. Being listed here does not mean any of
them reviewed, tested or endorsed this package.

We copy facts (indicator values and figures) with attribution, and link to everything else. Where a source publishes
a longer list than we reproduce, follow the link for the full, current version. Where a publisher's terms limit reuse
to personal or non-commercial use, we do not copy its indicator values at all; we link to it below. Where a publisher
asks that links go to its home page only, we link the home page.

## Discovery and official guidance

| Who | What we used | Link |
|---|---|---|
| **Calif** (Ryan Austin), **Wiz Research** (Sagi Tzadik, Shir Tamari, Nir Ohfeld, Tomer Schwartz) and **OpenAI**: found and reported the vulnerabilities, as credited by Sansec | The discovery credit | https://sansec.io/research/php2shell |
| **PHP project / PHP security team**: advisories GHSA-w8hw-w9fx-j79w (CVE-2026-109618) and GHSA-6wmf-4rv5-p3rp (CVE-2026-109619), fixed releases | CVE IDs, fixed versions 8.5.12, 8.4.27, 8.3.36 and 8.2.35, affected deployment models | https://www.php.net/news.php |
| **CISA**: alert "Critical Vulnerabilities in PHP Require Immediate Patching" (updated 2026-10-23) | Affected deployment models, the statement that CISA had not observed successful exploitation at publication | https://www.cisa.gov/news-events/alerts/2026/10/21/critical-vulnerabilities-php-require-immediate-patching |

## Research and incident reports (indicators)

| Who | What we used | Link |
|---|---|---|
| **Arctic Wolf Labs** | Payload host and URLs, Potassium hashes and domains, scanner markers, request shapes, User-Agent | https://arcticwolf.com/resources/blog/php2shell-critical-php-vulnerabilities-enable-unauthenticated-remote-code-execution |
| **Sophos Counter Threat Unit**: blog post on the released proof-of-concept exploit (Sophos asks for links to its home page only, so find the post from there) | Source IP, payload host and URL, file path and command, hashes, Potassium C2 addresses and domains, boundary and User-Agent | https://www.sophos.com/ |
| **SOCRadar Threat Intelligence Unit** | Attacker staging server, ports, probe filename and session prefixes | https://socradar.io/blog/php2shell-exploit-tooling-attacker-server |
| **Hunt.io** | Scanner network, payload and botnet hosts, C2 domain families, staging servers, a scanner fingerprint | https://hunt.io/blog/php2shell-cve-2026-109618-109619-open-directories |
| **F5 Labs** | Exploit-request patterns, User-Agents, markers, body sizes | https://www.f5.com/labs/articles/php2shell-exploitation-attempts-observed |
| **Fastly** | Boundary-based hunting advice, the look-alike campaigns to rule out, early-activity timeline | https://www.fastly.com/blog/how-fastly-detected-and-built-a-coordinated-response-to-php2shell |
| **KELA** | Fingerprints from the public PoC lineage, boundary prefix, markers, underground context | https://www.kelacyber.com/blog/php2shell-underground-response |
| **Sansec** | Affected-application context, the response-cookie memory-disclosure behaviour | https://sansec.io/research/php2shell |

## Further indicators and detections published by others (linked, not reproduced)

These are longer, still changing, or carry their own licences. Follow the links.

- **F5 Labs** publishes an appendix of roughly 150 PHP2Shell-named User-Agent strings. Many are defenders' checkers,
  so we did not copy them: https://www.f5.com/labs/articles/php2shell-exploitation-attempts-observed
- **Arctic Wolf Labs**: its IOC list may carry further file hashes than the two we could read on the page. Check the
  current version: https://arcticwolf.com/resources/blog/php2shell-critical-php-vulnerabilities-enable-unauthenticated-remote-code-execution
- **Wordfence** publishes a list of attacking IP addresses and further request markers (we could not open the page to
  read it ourselves): https://www.wordfence.com/blog/2026/10/attacks-on-php2shell-continue-but-no-known-compromises-as-a-result-yet
- **VulnCheck** (Canary Intelligence) published the scanner and exploitation-attempt addresses, User-Agents, markers and
  upload hashes it saw on its canary hosts. Its terms allow personal, non-commercial use, so we link and do not copy:
  https://www.vulncheck.com/blog/php2shell-vulnerabilities
- **Amazon Threat Intelligence** (AWS Security Blog) published the earliest known exploitation attempt (2026-10-17)
  and the source addresses it saw on its honeypots. AWS allows personal use only and asks for links to its home page
  only, so we link and do not copy; look for the post from there: https://aws.amazon.com/
- **Datadog Security Labs** published log lines, process and response-header behaviours and request-shape guidance. Its
  terms allow personal, non-commercial use, so we link and do not copy:
  https://securitylabs.datadoghq.com/articles/php2shell-critical-vulnerabilities-in-php
- **Huntress** wrote up a pre-disclosure incident with its own markers. Its terms allow personal, non-commercial use,
  so we link and do not copy: https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities
- **Sophos** lists its firewall, endpoint and XDR countermeasure IDs on its page (see the Sophos entry above).
- **CISA** links a SIGMA detection rule from its alert (see the CISA link above).
- **Abstract Security** publishes Snort rules for the request patterns: https://www.abstract.security/blog/php2shell-critical-php-rce-vulnerabilities-cve-2026-109618-and-cve-2026-109619
- **Elastic**, **CrowdStrike** and **Recorded Future** publish detection coverage:
  https://www.elastic.co/security-labs/threat-command/php2shell-php-rce-detection ·
  https://www.crowdstrike.com/en-us/blog/crowdstrike-protects-customers-from-php-rce-zero-days ·
  https://www.recordedfuture.com/blog/php2shell-vulnerability-exploitation
- **Censys** tracks patching progress on a public dashboard.
- **Sucuri**'s guidance on what to check on a server: https://blog.sucuri.net/2026/10/critical-php-security-update-two-rce-flaws-cve-2026-109618-cve-2026-109619.html

We do not link public exploit repositories.
