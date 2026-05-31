## DNS for AI Discovery (DNS-AID) Configuration Guide

### Overview

This guide explains the DNS-AID implementation for **apartca.org**. DNS-AID enables AI agents to discover your domain's capabilities and contact points through DNS records, following [draft-mozleywilliams-dnsop-dnsaid](https://datatracker.ietf.org/doc/draft-mozleywilliams-dnsop-dnsaid/) and [RFC 9460](https://www.rfc-editor.org/rfc/rfc9460) (SVCB/HTTPS RRs).

## What is DNS-AID?

DNS-AID (DNS for AI Discovery) is a mechanism that allows organizations to publish:
- 🤖 **Agent discovery points** - Where agents can learn about your services
- 🔐 **Encrypted endpoints** - Secure agent-to-agent communication channels
- ✅ **DNSSEC validation** - Authenticated discovery records

## Implementation Steps

### 1. Cloudflare Dashboard Access

1. Visit **https://dash.cloudflare.com**
2. Select **apartca.org** domain
3. Navigate to **DNS** section

### 2. Create DNS-AID Records (CORRECTED with PORT)

#### Record A: General Agent Discovery (Index)

**ServiceMode SVCB with explicit port:**

```
Type:        SVCB
Name:        _index._agents
Priority:    1
Target:      apartca.org
Parameters:  alpn=h2,h3 port=443
TTL:         3600
Proxy:       DNS only
```

**What it does**: Tells agents where to find general discovery information about your organization on port 443 (HTTPS).

#### Record B: Agent-to-Agent Communication

**ServiceMode SVCB with explicit port:**

```
Type:        SVCB
Name:        _a2a._agents
Priority:    1
Target:      apartca.org
Parameters:  alpn=h2,h3 port=443
TTL:         3600
Proxy:       DNS only
```

**What it does**: Provides a secure endpoint for agents to communicate with each other on port 443 with ALPN negotiation.

### 3. Alternative: HTTPS Records

If SVCB is not yet widely supported, you can use HTTPS records instead:

```
Type:        HTTPS
Name:        _agents
Priority:    1
Target:      apartca.org
Parameters:  alpn=h2,h3 port=443
TTL:         3600
Proxy:       DNS only
```

### 4. Enable DNSSEC

In Cloudflare dashboard:

1. Go to **DNS** → **DNSSEC**
2. Click **Enable DNSSEC**
3. Cloudflare automatically generates DS records
4. Your zone is now signed and validated

**Why DNSSEC?** Ensures agents receive authenticated (non-spoofed) discovery data.

### 5. Verify Configuration

Test your DNS-AID records with these commands:

```bash
# Check general discovery record
dig _index._agents.apartca.org SVCB

# Check agent-to-agent record
dig _a2a._agents.apartca.org SVCB

# Verify DNSSEC signature
dig apartca.org +dnssec

# Check DNSSEC validation status
delv @8.8.8.8 apartca.org

# Decode SVCB response
dig _index._agents.apartca.org SVCB +multiline
```

Expected output should show:
- SVCB records with `alpn` and `port` parameters
- DNSSEC signatures (RRSIG records)
- Ad flag (authenticated data)
- Example: `apartca.org. 3600 IN SVCB 1 apartca.org alpn=h2,h3 port=443`

## HTTP-Based Discovery (Fallback)

If DNS-AID isn't available, agents can still discover you via HTTP headers:

### RFC 8288 Link Relations

Your homepage includes:
```html
<link rel="sitemap" href="/sitemap.xml">
<link rel="alternate" href="/robots.txt">
<link rel="self" href="/">
```

### Well-Known Endpoints

- **`/.well-known/ai-discoverability.json`** - Your AI capabilities
- **`/.well-known/agent-skills/dns-aid/SKILL.md`** - DNS-AID documentation
- **`/sitemap.xml`** - Complete URL listing
- **`/robots.txt`** - Crawl directives

## Configuration Files

### `_dns-aid-config.json`
Machine-readable DNS-AID configuration for your domain.

### `.well-known/ai-discoverability.json`
HTTP-accessible metadata describing your organization and capabilities.

### `DNS-AID-SETUP.md` (this file)
Human-readable guide for understanding and maintaining DNS-AID.

## DNS Record Parameters Explained

### `port` (REQUIRED in ServiceMode SVCB)
- **443** = HTTPS (standard)
- **80** = HTTP (not recommended)
- **Custom** = Your application port
- Must be included in ServiceMode SVCB records per RFC 9460

### `alpn` (Application Layer Protocol)
- `h2` = HTTP/2
- `h3` = HTTP/3 (QUIC)
- `http/1.1` = HTTP/1.1 (fallback)
- Tells agents which protocols you support

### `ech` (Encrypted Client Hello)
- Optional parameter for encrypted SNI
- Enhances privacy for agent requests

### Priority (in SVCB/HTTPS records)
- Lower numbers = higher priority
- Set to `1` for immediate selection

## Common Issues & Solutions

### "SVCB ServiceMode record is missing port"
**Solution**: Add `port=443` to your SVCB record parameters

```
❌ WRONG:  alpn=h2,h3
✅ CORRECT: alpn=h2,h3 port=443
```

### DNSSEC validation failing
- Check DS records are published at your registrar
- Verify DNSSEC status: `delv @8.8.8.8 apartca.org`
- Wait 24-48 hours for full propagation

### Agents not discovering your domain
- Ensure `port` parameter is present in SVCB records
- Verify both `_index._agents` and `_a2a._agents` records exist
- Check `.well-known/ai-discoverability.json` is valid JSON
- Test with: `https://isitagentready.com`

## Testing with External Tools

- **[DNS Checker](https://dnschecker.org)** - Verify DNS propagation
- **[MXToolbox](https://mxtoolbox.com)** - DNSSEC validation
- **[IsItAgentReady](https://isitagentready.com)** - Agent discoverability score
- **[Cloudflare DNS Test](https://www.cloudflare.com/en-gb/learning/dns/dns-security/)** - Validate DNSSEC

## Security Best Practices

✅ **DO:**
- Include `port=443` in all ServiceMode SVCB records
- Keep DNSSEC enabled
- Monitor DNS changes in Cloudflare audit logs
- Use TTL of 3600 or higher
- Test records after changes
- Document all DNS modifications

❌ **DON'T:**
- Omit the `port` parameter (causes validation errors)
- Disable DNSSEC (breaks agent validation)
- Set TTL below 300 seconds
- Mix DNS only and proxied records
- Change records without testing

## Standards & References

| Standard | Purpose | Link |
|----------|---------|------|
| RFC 9460 | SVCB/HTTPS RRs | https://www.rfc-editor.org/rfc/rfc9460 |
| RFC 8288 | Web Linking | https://tools.ietf.org/html/rfc8288 |
| RFC 4033-4035 | DNSSEC | https://tools.ietf.org/html/rfc4033 |
| DNS-AID Draft | AI Discovery | https://datatracker.ietf.org/doc/draft-mozleywilliams-dnsop-dnsaid/ |

## Support & Questions

- **Contact**: hello@apartca.org
- **GitHub Repository**: https://github.com/apartca/apartca
- **Issue Tracker**: https://github.com/apartca/apartca/issues
- **Specification Discussions**: https://datatracker.ietf.org/doc/draft-mozleywilliams-dnsop-dnsaid/
- **Agent Readiness Score**: https://isitagentready.com

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-05-31 | 1.1 | **CRITICAL FIX**: Added `port=443` parameter to ServiceMode SVCB records |
| 2026-05-31 | 1.0 | Initial DNS-AID setup guide |

---

**Last Updated**: 2026-05-31  
**Domain**: apartca.org  
**DNSSEC Status**: ✅ Enabled  
**Agent Discovery**: ✅ Ready (with corrected SVCB records)  
**RFC 9460 Compliance**: ✅ ServiceMode port parameter included
