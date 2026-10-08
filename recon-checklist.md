# The Ultimate Recon Checklist

**Deep-logic, high-security-target web recon for bug bounty & pentesting.**

Built on the OWASP Web Security Testing Guide (WSTG) *Information Gathering* framework (WSTG-INFO-01 → 10), extended with modern bug-bounty tradecraft: ASN/cloud footprinting, subdomain enumeration, JS/secret mining, API/GraphQL recon, multi-tenant boundary mapping, and chain-for-impact discipline.

> **Use it like tides — ebb & flow.** Work down the list until you have 3–5 attack vectors on a target. Test them. When stuck, pin them and go back up the list to expand attack surface. Repeat. Recon is ~80% of the outcome; the map you build here is your playbook for every finding later.

> **Rules of engagement first.** Only touch in-scope assets. Respect program policy, disclosure rules, and rate limits. Everything below assumes authorized testing.

---

## Phase 0 — Scope, Setup & Discipline

- [ ] Read the full program policy: in-scope assets, **out-of-scope** assets, disallowed testing, safe-harbor terms
- [ ] Record scope type: single domain / wildcard / **wide-open** (any asset belonging to org)
- [ ] Note reward structure and which asset tiers actually pay (focus effort there)
- [ ] Build the workspace folder structure early: `scope/ subs/ live/ ports/ urls/ js/ params/ api/ screenshots/ nuclei/ notes/`
- [ ] Set up a notes system (Markdown / Notion / Obsidian) — one page per asset, log every request/response worth keeping
- [ ] Configure rate limiting / thread caps on all tooling to avoid API bans, WAF trips, and program complaints
- [ ] Separate **passive** findings from **active** findings in your notes (keeps the picture clean during exploitation)
- [ ] Identify the WAF / CDN / rate-limit posture before hammering anything

---

## Phase 1 — Passive Reconnaissance (OSINT)
*WSTG-INFO-01 — Search Engine Discovery & OSINT. Zero direct traffic to the target.*

### Search-engine dorking
- [ ] Google dorks: `site:`, `inurl:`, `intitle:`, `intext:`, `filetype:`, `-www`, `site:*.target.com`
- [ ] Sensitive file dorks: `filetype:env`, `filetype:log`, `filetype:sql`, `filetype:bak`, `ext:txt password`
- [ ] Exposed panels/config: `intitle:"index of"`, `inurl:admin`, `inurl:config`, `inurl:wp-content`
- [ ] Repeat across **Bing, DuckDuckGo, Yandex, Baidu** — each indexes different corners
- [ ] Pull from the Google Hacking Database (GHDB) for target-relevant dork templates

### Internet-wide asset search engines
- [ ] **Shodan** — `hostname:target.com`, `ssl.cert.subject.cn:target.com`, `org:"Target"`, exposed services/ports
- [ ] **Censys / FOFA / ZoomEye / Netlas** — cross-reference certs, banners, favicon hashes
- [ ] Favicon hash pivoting (`http.favicon.hash:`) to find related/forgotten infra
- [ ] Search by **HTTP response body hash** and unique strings (copyright, error pages) to cluster assets

### Code & secret OSINT
- [ ] GitHub/GitLab dorking: `org:target password`, `"target.com" api_key`, `filename:.env`, `filename:config`
- [ ] Enumerate the org's public repos and forks: `api.github.com/orgs/<org>/repos`
- [ ] Scan repos + **full git history** for secrets: TruffleHog, Nosey Parker, gitleaks, GitHound, shhgit
- [ ] Search commit messages and deleted-but-cached blobs for keys/creds
- [ ] Check Postman public workspaces, Pastebin, StackOverflow, npm/PyPI packages for leaked endpoints/tokens

### People & breach OSINT
- [ ] Enumerate employees / emails (Hunter.io, LinkedIn, theHarvester)
- [ ] Cross-check emails against breach corpora for credential-stuffing candidates (respect scope/law)
- [ ] Map naming conventions (email → likely internal usernames)

### Historical & archival
- [ ] Wayback Machine + `waybackurls` / `gau` / `gauplus` / `urlfinder` for dead endpoints, old params, retired APIs
- [ ] Diff old JS/HTML against current to spot removed-but-live endpoints
- [ ] Check `web.archive.org` for old `robots.txt` and `sitemap.xml` revealing paths

---

## Phase 2 — Attack-Surface Discovery (Asset Enumeration)
*Expand from a domain to the whole footprint. Forgotten/staging/dev infra is where the best bugs live.*

### Root/apex discovery (wide-scope programs)
- [ ] **ASN enumeration**: find the org's Autonomous System(s) via `bgp.he.net`, `asnlookup`, `amass intel -asn`
- [ ] Pull all IPv4/IPv6 prefixes for each ASN (`api.bgpview.io/asn/ASxxxx/prefixes`)
- [ ] Reverse-lookup owned IP ranges and cloud IP ranges; grep certs for the org name to surface new apex domains
- [ ] **Certificate transparency**: `crt.sh?q=%.target.com`, Censys certs → new domains/subdomains
- [ ] WHOIS / reverse-WHOIS pivots (registrant org/email) to find sibling domains
- [ ] `amass intel -org "Target"` / `-whois` to seed apex domains

### Subdomain enumeration — passive
- [ ] `subfinder -d target.com -all -recursive` (dozens of passive sources)
- [ ] `amass enum -passive -d target.com` (deeper, slower — run in background)
- [ ] `assetfinder`, `github-subdomains`, `gitlab-subdomains`
- [ ] crt.sh JSON → parse `name_value` → `anew`
- [ ] Merge + dedupe all sources into one master list (`anew` / `sort -u`)

### Subdomain enumeration — active / brute
- [ ] DNS brute force with `puredns` / `shuffledns` + massdns using a large, current wordlist
- [ ] **Permutation/alteration**: `dnsgen`, `gotator`, `altdns`, `ripgen` on known subs → resolve the new candidates
- [ ] VHost brute forcing against known IPs (host-header fuzzing) for name-based virtual hosts
- [ ] Resolve everything with `dnsx` against trusted resolvers; keep only live records

### DNS deep-dive
- [ ] `dig` all record types (A, AAAA, CNAME, MX, TXT, NS, SOA, SRV, CAA)
- [ ] **Zone transfer attempt** (`dig axfr @ns target.com`) — rare, but a jackpot when it works
- [ ] Reverse DNS across owned IP ranges (`hakrevdns`, `dnsx -ptr`)
- [ ] TXT records for SPF/DKIM/DMARC → reveals third-party services & sending infra

### Subdomain takeover
- [ ] Check dangling CNAMEs pointing to unclaimed third-party services (`subzy`, `subjack`, nuclei `takeovers/`)
- [ ] Verify manually before reporting (fingerprint the claimable service)

---

## Phase 3 — Host Probing & Server Fingerprinting
*WSTG-INFO-02 (Fingerprint Web Server) + WSTG-INFO-04 (Enumerate Applications).*

### Live-host probing
- [ ] `httpx` over all subdomains → status code, title, content-length, tech, CDN, redirect chain, response hash
- [ ] Flag interesting statuses: 401/403 (auth-protected), 500 (error-prone), 200 on odd paths
- [ ] Cluster by response hash to spot templated apps vs unique one-offs

### Port & service scanning
- [ ] Fast full-range sweep: `naabu` or `masscan -p1-65535` (responsibly), then confirm
- [ ] `nmap -sV -sC` on open ports for service + version detail
- [ ] `rustscan` for speed on individual hosts; note non-web services (DBs, admin ports, mail)
- [ ] Identify **web server type and exact version** (Server header, error pages, nmap `-sV`)
- [ ] Map version → known CVEs (feed into Phase 9)

### Fingerprinting & visual triage
- [ ] Screenshot everything: `aquatone` / `gowitness` / `eyewitness` → eyeball for login panels, dashboards, dev tools
- [ ] Tech stack: **Wappalyzer** extension, `whatweb`, `webanalyze`
- [ ] **WAF detection**: `wafw00f` (shapes your evasion + rate strategy)
- [ ] TLS/cert inspection (`testssl.sh`) for weak config + SANs revealing more hosts

---

## Phase 4 — Metafiles, Content & Client-Side Review
*WSTG-INFO-03 (Metafiles) + WSTG-INFO-05 (Webpage content / info leakage).*

### Metafiles
- [ ] `robots.txt` — disallowed paths are a hint map, not a wall
- [ ] `sitemap.xml` (and `sitemap_index.xml`) — full URL inventory
- [ ] `humans.txt` — team/tech hints
- [ ] `security.txt` (`/.well-known/security.txt`) — disclosure contact + policy
- [ ] `/.well-known/` sweep: `openid-configuration`, `oauth-authorization-server`, `assetlinks.json`, `apple-app-site-association`, SAML metadata

### Page-source & client-side review
- [ ] Read HTML source for comments, debug flags, internal hostnames, TODOs, dev creds
- [ ] Confirm **autocomplete disabled** on sensitive fields; note if not (weak signal, chain candidate)
- [ ] Inspect cache/`Cache-Control` headers on authed pages
- [ ] Network tab: catalog AJAX/XHR calls, API base URLs, auth headers, `postMessage` handlers, iframes, CDN asset origins

### JavaScript & secret mining (high-value)
- [ ] Harvest JS files: `katana`, `hakrawler`, `getJS`, `subjs`, `gau | grep '\.js'`, `JSFScan.sh`
- [ ] Extract endpoints/paths: `LinkFinder`, `xnLinkFinder`, `mantra`
- [ ] Extract secrets: `SecretFinder`, `trufflehog`, `js-snitch` (TruffleHog + Semgrep), `nuclei -t exposures/`
- [ ] Grep with `gf` patterns: `aws-keys`, `firebase`, `s3-buckets`, `urls`, tokens
- [ ] Pull hidden/undocumented API paths and **admin-only operations** bundled in client JS (e.g. `updateRole`, `systemDebug`)
- [ ] Hunt for exposed **source maps** (`.js.map`) → reconstruct original source & logic
- [ ] **Validate** any found key before reporting (KeyHacks) — an unvalidated key is noise
- [ ] Note client-side auth logic, feature flags, and role checks (front-end-only enforcement = server-side test targets)

---

## Phase 5 — Content & Path Discovery
*WSTG-INFO-07 — Map Execution Paths Through the Application.*

- [ ] Active crawl: `katana` / `hakrawler` / `gospider`, plus Burp Suite crawl for authenticated coverage
- [ ] Historical URL harvest: `gau` / `waybackurls` → feed into fuzzing + param discovery
- [ ] Directory/file brute force: `ffuf`, `feroxbuster` (recursive), `dirsearch`, `gobuster`
      - extensions to force: `php,asp,aspx,jsp,json,js,txt,bak,old,zip,tar.gz,config,env,sql`
- [ ] Backup/temp/config exposure: `.bak`, `.old`, `~`, `.swp`, `.env`, `config.php.save`
- [ ] VCS & metadata exposure: `.git/`, `.svn/`, `.hg/`, `.DS_Store` (dump with `git-dumper` if `.git` exposed)
- [ ] CI/CD & infra files: `Dockerfile`, `docker-compose.yml`, `.gitlab-ci.yml`, `.github/`, `Jenkinsfile`
- [ ] Use response-code + length + word-count filtering to cut false positives; recurse into interesting dirs
- [ ] Keep wordlists current (SecLists, Assetnote, raft, target-specific words scraped from the app)

---

## Phase 6 — Entry Points & Parameter Discovery
*WSTG-INFO-06 — Identify Application Entry Points.*

- [ ] Enumerate HTTP methods per endpoint (`OPTIONS`, and test `PUT/DELETE/PATCH`, verb tampering)
- [ ] Catalog **where** input enters: query params, path segments, body (JSON/form/multipart), headers, cookies
- [ ] Identify **injection points** per parameter (reflected values, DB-backed lookups, redirects, file refs)
- [ ] Parameter discovery: `arjun`, `paramspider`, `x8`, Burp **Param Miner** (also headers & cache keys)
- [ ] Mine params from historical URLs and JS, then replay/fuzz
- [ ] Tag candidate params by class with `gf`: `xss`, `ssrf`, `sqli`, `redirect`, `idor`, `rce`, `lfi`
- [ ] Map all state-changing operations, upload endpoints, and redirect params (prime for logic/IDOR/SSRF)

---

## Phase 7 — API & GraphQL Reconnaissance
*Modern APIs are the highest-signal surface — and where authz bugs (IDOR/BOLA/BFLA) concentrate.*

### REST / RPC / SOAP
- [ ] Discover API base paths: `/api`, `/api/v1..v3`, `/rest`, `/internal`, `/mobile`
- [ ] Find spec docs: `swagger.json`, `openapi.json`, `/swagger-ui`, `/api-docs`, `/redoc`
- [ ] SOAP/WSDL: `?wsdl`, `?WSDL`, `?xsd`; XML-RPC: `/xmlrpc.php`; probe for gRPC/grpc-web
- [ ] Brute API routes with `kiterunner` (Kite/route wordlists) — catches undocumented endpoints
- [ ] Import into Postman/Insomnia; map auth model, versioning, and per-endpoint object references

### GraphQL
- [ ] Locate endpoints: `/graphql`, `/api/graphql`, `/v1/graphql`, `/query`, `/graphql/console`
- [ ] Locate IDEs/playgrounds: `/graphiql`, `/playground`, `/altair` (screenshot-scan with EyeWitness)
- [ ] **Introspection probe**: `{"query":"{__schema{queryType{name}}}"}` — if enabled, pull the full schema
- [ ] Full introspection → visualize with **GraphQL Voyager**; auto-generate requests with **InQL** (Burp)
- [ ] If introspection is **disabled**: reconstruct via field-suggestion errors ("Did you mean…?") using **Clairvoyance** + a high-frequency GraphQL wordlist
- [ ] Enumerate all queries, mutations, subscriptions; note anything sensitive (recovery codes, role changes, PII)
- [ ] Rip **undocumented/admin mutations** out of client JS even when introspection is off
- [ ] Test resolver-level **BOLA/IDOR** on `node(id:)` / object-by-id resolvers across tenants
- [ ] Field/argument fuzzing: inject `isAdmin`, `token`, `passwordHash` — resolves without error = obscurity bypass
- [ ] Note batching/alias surface (rate-limit bypass), query depth/complexity (DoS), verbose errors, PII-over-GET
- [ ] Run `graphql-cop` for quick misconfig wins; use Altair/GraphiQL for manual exploitation with custom auth headers

> Introspection-enabled *alone* is usually Informative. The value is using the schema to reach a real authz/logic bug.

---

## Phase 8 — Framework & Architecture Mapping
*WSTG-INFO-08 (Fingerprint Framework) + WSTG-INFO-10 (Map Application Architecture).*

### Framework fingerprint
- [ ] URL extensions (`.php`, `.aspx`, `.jsp`) → language/stack
- [ ] Cookie names (`PHPSESSID`, `JSESSIONID`, `laravel_session`, `csrftoken`, `__RequestVerificationToken`)
- [ ] HTTP headers (`X-Powered-By`, `Server`, `X-AspNet-Version`, framework-specific headers)
- [ ] HTML source markers (meta generator, asset paths, template comments)
- [ ] CMS-specific: `wpscan` (WordPress), `CMSeeK`, `droopescan` (Drupal/Joomla)

### Architecture map (the deliverable that drives testing)
- [ ] Draw the overall site structure: apps, subdomains, and how they relate
- [ ] Map **auth flows**: login, SSO/OAuth/OIDC, MFA, password reset, session model, token types (JWT? validate signature/alg)
- [ ] Map **trust boundaries**: CDN/WAF edge, API gateway, microservices, third-party integrations, webhooks
- [ ] Map the **multi-tenant model**: how tenants/orgs/workspaces are isolated; where an object ID crosses a boundary
- [ ] Map the **permission matrix**: roles, delegated roles, invite flows, role-change operations (rich matrices = rich BFLA/BOLA surface)
- [ ] Identify state-changing / stateful operations (candidates for race conditions & logic flaws)
- [ ] Record per-asset in a table: `subdomain | function | auth? | API type | interesting endpoints | notes`

---

## Phase 9 — Automated Vulnerability Sweep (recon-adjacent)
*Low-effort coverage before deep manual work — but expect WAFs and duplicate-heavy results here.*

- [ ] `nuclei` by category: `exposures/`, `misconfiguration/`, `cves/`, `takeovers/`, `default-logins/`
- [ ] Map fingerprinted tech + versions → specific CVEs; verify a working exploit before treating as a finding
- [ ] Misconfig sweep: `testssl.sh`, security-header check, open-directory listing, CORS misconfig (with a real exfil PoC, not wildcard-alone)
- [ ] Cloud storage recon: permutate bucket names (`company`, `company-dev/prod/backup/staging`), check public list/read on S3/GCS/Azure blobs
- [ ] Treat automated output as **leads**, not findings — everyone runs these; your edge is manual depth on what they surface

---

## Phase 10 — Triage, Prioritization & Chain-for-Impact
*Turn the recon map into a ranked test plan. This is where recon becomes bounties.*

- [ ] Score each asset/endpoint: **Impact 1–5** × **Effort 1–5**; test high-impact/low-effort first
- [ ] Pick 3–5 concrete attack vectors per target; timebox them (**one-hour rule** — no progress in an hour, switch context)
- [ ] **Two-eye approach**: run the systematic checklist *and* watch for anomalies/unexpected behavior in parallel
- [ ] **Bug-class rotation**: don't fixate on one class. Rotate through authz (IDOR/BOLA/BFLA), business logic, race conditions, OAuth/OIDC chains, cache poisoning, CI/CD — the less-saturated classes have less competition
- [ ] Favor **fresh/low-saturation scope**: new API surfaces, MCP servers, staging with few prior reports > heavily-hunted wildcards
- [ ] **Chain-for-impact** loop for every candidate:
      1. **Confirm** the bug is real with a raw HTTP request
      2. **Map siblings** — every endpoint in the same controller/module/API group
      3. **Test siblings** — apply the same pattern across all of them
      4. **Chain** — combine with a different bug class if a sibling exposes one
      5. **Quantify** — "affects N users / exposes $X / N records"
      6. **Report** one clean report per chain (chains pay 3–10× single bugs)
- [ ] Before spending a report slot, require **demonstrated cross-boundary impact** (real user, real data, no unusual victim action)
- [ ] Know the usual **non-qualifying / Informative-alone** items (missing headers, SPF/DMARC, GraphQL introspection alone, self-XSS, CORS wildcard without exfil PoC, banner disclosure) — only submit these when *chained* into real impact

---

## Appendix A — Core Toolbox (by phase)

| Phase | Tools |
|---|---|
| OSINT / passive | Shodan, Censys, FOFA, Netlas, crt.sh, GHDB, theHarvester, TruffleHog, Nosey Parker, GitHound, gau, waybackurls |
| ASN / apex | bgp.he.net, asnlookup, amass intel, bgpview API |
| Subdomains | subfinder, amass, assetfinder, github-subdomains, puredns, shuffledns, dnsgen, gotator, dnsx, massdns |
| Probing / ports | httpx, naabu, nmap, masscan, rustscan, wafw00f, testssl.sh |
| Screens / tech | aquatone, gowitness, eyewitness, whatweb, webanalyze, Wappalyzer |
| Content / paths | katana, hakrawler, gospider, ffuf, feroxbuster, dirsearch, gobuster, git-dumper |
| JS / secrets | JSFScan.sh, LinkFinder, xnLinkFinder, SecretFinder, mantra, js-snitch, gf, KeyHacks |
| Params | arjun, paramspider, x8, Burp Param Miner |
| API / GraphQL | kiterunner, InQL, Clairvoyance, GraphQL Voyager, graphql-cop, Altair, GraphQLmap |
| Scanning | nuclei (+ templates), wpscan, CMSeeK, subzy |
| Proxy / manual | Burp Suite (+ extensions), Postman/Insomnia |
| Takeover | subzy, subjack, nuclei takeovers |

## Appendix B — Wordlists

- **SecLists** — the baseline for everything (discovery, params, subdomains, fuzzing)
- **Assetnote wordlists** — high-quality, large, regularly refreshed for content + subdomain brute
- **params.txt / Arjun wordlists** — parameter discovery
- **GraphQL high-frequency vocabulary** (nicholasaleks) — for Clairvoyance schema recovery
- Build **target-specific wordlists** by scraping words from the app's own JS/HTML

## Appendix C — Key References & Further Reading

- **OWASP Web Security Testing Guide (WSTG)** — the canonical methodology this checklist extends
- **OWASP Amass** & the **OWASP Testing Checklist** — asset discovery + coverage
- **PortSwigger Web Security Academy** — free, authoritative labs (esp. GraphQL, access control, API testing)
- **HackTricks** — practical pentest/bug-bounty reference per technique
- **PayloadsAllTheThings** — payloads & bypasses by vuln class
- **Bug Bounty Methodology 2025** (amrelsagaei) & **DEFCON 32 Bug Bounty Village recon methodology** (R-s0n)
- **awesome-bugbounty-tools** (vavkamil) — curated, current tool index
- **The Web Application Hacker's Handbook**, **Bug Bounty Bootcamp** (Vickie Li), **Real-World Bug Hunting** (Peter Yaworski) — foundational books
- **checklist.hackwithsingh.com** & **h0tak88r/Sec-88** — large interactive test-case checklists (Web/API/GraphQL/Web3)

---

*This is a living document — prune what doesn't fit your targets, add classes as new techniques emerge, and let the tides carry you back up the list whenever a target goes quiet.*
