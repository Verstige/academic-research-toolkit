# Web Infrastructure Audit — Methodology

Comprehensive methodology for auditing a web platform's full technology stack, API surface, and business systems.

---

## Phase 1: Domain Reconnaissance

### DNS Enumeration

```bash
# Basic DNS records
dig {domain} A {domain} NS {domain} MX {domain} TXT +short

# Subdomain bruteforce — common patterns
for sub in api app dashboard admin portal trading platform app-v2; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "https://${sub}.{domain}" -A "Mozilla/5.0")
  [ "$code" != "000" ] && echo "${sub}.${domain} -> $code"
done
```

### SSL Certificate Inspection

```bash
# Get full cert details
echo | openssl s_client -connect {domain}:443 -servername {domain} 2>/dev/null | \
  openssl x509 -noout -text 2>/dev/null | grep -A10 "Subject Alternative"

# Issuer and dates
echo | openssl s_client -connect {domain}:443 -servername {domain} 2>/dev/null | \
  openssl x509 -noout -issuer -dates -subject
```

**Key signals from SSL cert:**
- `Let's Encrypt` → cheap/automated, common for scammers AND small businesses
- `Google Trust Services` → Vercel, Cloudflare, or serious infrastructure
- `DigiCert` → enterprise-grade
- **SAN entries** → multiple domains on same cert = shared hosting OR white-label operation
- Cert CN matching `something.belfry-capital.online` → white-label infrastructure client

### HTTP Headers

```bash
curl -sI "https://{domain}" -A "Mozilla/5.0" 2>&1 | head -20
```

**Key headers:**
- `x-deployment-id: <uuid>` → **Vercel** deployment
- `server: cloudflare` → Cloudflare CDN/proxy
- `strict-transport-security` → HSTS enabled
- `cf-ray` → Cloudflare proxied
- `x-nextjs-cache` → Next.js ISR
- `server: LiteSpeed` → cheap PHP hosting (often HYIP)
- `X-Powered-By` → reveals framework

### Hosting/Network

```bash
# IP to org lookup
dig +short {domain} | head -1 | xargs -I{} whois {} | \
  grep -i "OrgName\|netname\|country\|abuse\|ISP"

# Reverse DNS
dig +short -x $(dig +short {domain} | head -1)
```

---

## Phase 2: Technology Stack Fingerprinting

### Web Framework Detection

```bash
# Next.js (Vercel) detection
curl -sI "https://{domain}" | grep -i "x-deployment-id"
# → Vercel deployment UUID

# Laravel/PHP detection
curl -s "https://{domain}" | grep -i "laravel\|PHPSESSID\|csrf-token"

# React/Vue detection
curl -s "https://{domain}" | grep -iE "react|react-dom|__NEXT_DATA__|vue"
```

### JS Bundle Extraction

```bash
# Get HTML and find JS chunks
html=$(curl -s "https://{domain}" -A "Mozilla/5.0")
echo "$html" | grep -oE 'src="[^"]*\.js"' | head -20

# Download main bundle
curl -s "https://{domain}/assets/{chunk-name}.js" -o /tmp/target_main.js
wc -c /tmp/target_main.js

# Search for API endpoints in bundle
grep -oE '"(/api[^"]+)"' /tmp/target_main.js | sort -u

# Search for full URLs
grep -oE 'https?://[a-z0-9._-]+\.[a-z]{2,}[a-z0-9._/-]*' /tmp/target_main.js | sort -u

# Search for Supabase
grep -oE '[a-z0-9]{20,}\.supabase\.co[a-z0-9._/-]*' /tmp/target_main.js | sort -u

# Search for fetch/axios calls
grep -oE '(fetch|axios)\s*\(\s*["'"'"'][^"'"'"']+' /tmp/target_main.js | sort -u
```

### Common SaaS/API Fingerprints

```bash
# Supabase
grep -oE '[a-z0-9]{20,}\.supabase\.co' /tmp/target_main.js

# Intercom
grep -oE 'intercom\.io/widget/[a-z0-9]+' /tmp/target_main.js

# Hyros
grep -oE 'hyros\.com/v1/[a-z0-9]+' /tmp/target_main.js

# Cloudflare
curl -sI "https://{domain}" | grep -i "cf-ray\|__cf_bm"

# Google reCAPTCHA
curl -s "https://{domain}" | grep -i "recaptcha"

# Tidio/Smartsupp
curl -s "https://{domain}" | grep -iE 'tidio|smartsupp'
```

---

## Phase 3: SPA Route Enumeration

**Critical for Next.js/Vercel apps** — server returns 404 for unknown routes, client handles routing.

```bash
# Common SPA routes to test
for path in \
  "/signin" "/signup" "/login" "/register" \
  "/dashboard" "/account" "/settings" \
  "/copy-trading" "/pamm" "/education" \
  "/api/portfolio" "/api/users" "/api/accounts"; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    "https://{domain}${path}" -A "Mozilla/5.0")
  echo "${path} -> $code"
done
```

**If all return 200 with HTML** → Next.js SPA (all routes handled client-side)
**If 404** → SSR or route doesn't exist

---

## Phase 4: API Surface Discovery

### Subdomain Enumeration

```bash
for sub in api api-v1 api-demo dashboard app my portal admin trading-v2 platform; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    "https://${sub}.{domain}" -A "Mozilla/5.0")
  [ "$code" != "000" ] && echo "${sub}.${domain} -> $code"
done
```

### Supabase Backend

If Supabase detected (`*.supabase.co`):

```bash
# Supabase REST API
curl -s "https://{ref}.supabase.co/rest/v1/" \
  -H "apikey: {anon_key}" \
  -H "Authorization: Bearer {anon_key}"

# Supabase storage
curl -s "https://{ref}.supabase.co/storage/v1/object/public/"

# Supabase edge functions
curl -s "https://{ref}.supabase.co/functions/v1/{function-name}"
```

### Edge Function Probing

```bash
# Try common methods
curl -s "https://{url}" -X POST -H "Content-Type: application/json" -d '{}'
curl -s "https://{url}" -A "Mozilla/5.0"

# Expected responses:
# Unauthorized → valid endpoint, needs auth
# Method not allowed → valid, wrong HTTP method
# Invalid path → valid Supabase function
# Could not find → function doesn't exist
```

### SSE (Server-Sent Events) Detection

```bash
curl -s -N "https://{url}" \
  -H "Accept: text/event-stream" \
  -A "Mozilla/5.0" 2>&1 | head -5
# Connected heartbeat → live data feed
```

---

## Phase 5: Business System Audit

### Systems to Identify

| System | Detection | Routes to Check |
|---|---|---|
| Portfolio tracker | Balance/P&L in JS bundle | `/portfolio`, `/wallets` |
| Copy trading | Strategy providers | `/copy-trading`, `/social` |
| PAMM | Manager/investor split | `/pamm`, `/pamm/create`, `/pamm/investor` |
| 10X/Challenge | Challenge program | `/10x-challenge`, `/accounts` |
| Education | Course content | `/academy`, `/education` |
| AI Trading Coach | Chat API | `/ai-studio`, `*coach-chat*` |
| Partner/IB | Commission tracking | `/partner*` |
| Virtual Card | Card management | `/card*`, `/zypto*` |
| KYC | Identity verification | `/get-sumsub-link`, SumSub in bundle |

### Legal Doc Mining

Always check:
- `/terms-conditions` → company name, TAB in nav reveals products
- `/privacy-policy` → registered address (reveals jurisdiction)
- `/risk-disclosure` → leverage limits, regulatory framework
- `/anti-money-laundering` → compliance procedures

---

## Phase 6: Competitive/White-Label Pattern Detection

### Cross-Domain Infrastructure Sharing

```bash
# Same IP = same hosting provider
dig +short {domain1} 
dig +short {domain2}

# SSL cert SANs = same certificate = same operator
echo | openssl s_client -connect {domain}:443 -servername {domain} 2>/dev/null | \
  openssl x509 -noout -text 2>/dev/null | grep -A20 "Subject Alternative"
```

**Red flags for white-label/HYIP:**
- Multiple domains on same IP
- SSL cert covering unrelated brand names
- Cheap hosting (LiteSpeed on Romanian/German VPS) for "financial institution"
- Template fingerprints in HTML (`hyipcustomize.com`, HTTrack comments)
- Tidio/Smartsupp live chat (common in HYIP templates)

---

## Phase 7: JS Bundle Deep Dive

Full extraction pattern for complex SPAs:

```bash
# 1. Get the main JS chunk
curl -s "https://{domain}" -A "Mozilla/5.0" | \
  grep -oE 'src="(/assets/[^"]+\.js)"' | head -5

# 2. Download (large files common)
curl -s "https://{domain}{chunk}" -o /tmp/bundle.js

# 3. Extract all URL patterns
grep -oE 'https?://[a-zA-Z0-9._][a-zA-Z0-9._-]*\.[a-zA-Z]{2,}[a-zA-Z0-9._/?&=%-]*' \
  /tmp/bundle.js | sort -u > /tmp/urls.txt

# 4. Extract path patterns (quoted strings starting with /)
grep -oE '"/[a-z][a-z0-9/_-]{3,50}"' /tmp/bundle.js | sort -u

# 5. Extract Supabase refs
grep -oE '[a-z0-9]{20,}\.supabase\.co[a-z0-9._/-]*' /tmp/bundle.js | sort -u

# 6. Look for auth patterns
grep -oE 'Authorization["'"'"']?\s*[:=]\s*["'"'"']?Bearer [^"'"'"'\n]{10,50}' /tmp/bundle.js
```

---

## Common Findings Patterns

### Professional Broker (genesisfxmarkets.com type)
- Vercel + Cloudflare
- Google Trust Services SSL
- Next.js SPA
- Supabase backend
- the broker integration
- Intercom + Hyros tracking
- Saint Lucia / offshore jurisdiction

### HYIP Scam (genesis-fx.com type)
- LiteSpeed + cheap VPS (Romania/Germany)
- Let's Encrypt SSL
- Laravel/PHP
- Smartsupp + Google reCAPTCHA
- HYIP calculator template (hyipcustomize.com)
- Multi-brand on same cert (thodexfinance.com chain)
- No regulatory disclosure

---

## Supabase Edge Functions (Genesis FX pattern)

```
ai-studio-chat              → AI chat interface
ai-trading-coach-chat      → Trading-specific AI coach
trade-chat                  → Trade-related chat
copy-strategy-invites       → Copy trading allocation
partner-api                 → Partner/IB external API
bluefix-quotes-sse          → Live price SSE stream
the broker-quotes-sse      → the broker price feed
the broker-user-quotes-sse → User-specific quotes
client-error-report         → Client-side error reporting
```

---

## Output Format

For each system audited, document:

1. **Endpoint/URL** — exact path
2. **Backend** — what powers it (Supabase, custom, third-party)
3. **Auth** — open, auth-required, cloudflare-gated
4. **Functionality** — what it does based on route names + legal docs
5. **Data** — what data is stored/processed
6. **Integration** — how it connects to other systems

**Domain distinction rule:** Always confirm exact domain. `genesis-fx.com` ≠ `genesisfxmarkets.com` ≠ `genesisfxmarkets.com`. One wrong character = completely different company.
