# DNS for AI Discovery (DNS-AID) Configuration Guide

## Overview

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

### 2. Create DNS-AID Records

#### Record A: General Agent Discovery (Index)

```
Type:        SVCB
Name:        _index._agents
Priority:    1
Target:      apartca.org
Parameters:  alpn=h2,h3
TTL:         3600
Proxy:       DNS only
```

**What it does**: Tells agents where to find general discovery information about your organization.

#### Record B: Agent-to-Agent Communication

```
Type:        SVCB
Name:        _a2a._agents
Priority:    1
Target:      apartca.org
Parameters:  alpn=h2,h3
TTL:         3600
Proxy:       DNS only
```

**What it does**: Provides a secure endpoint for agents to communicate with each other about your services.

### 3. Enable DNSSEC

In Cloudflare dashboard:

1. Go to **DNS** → **DNSSEC**
2. Click **Enable DNSSEC**
3. Cloudflare automatically generates DS records
4. Your zone is now signed and validated

**Why DNSSEC?** Ensures agents receive authenticated (non-spoofed) discovery data.

### 4. Verify Configuration

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
```

Expected output should show:
- SVCB records with `alpn` parameters
- DNSSEC signatures (RRSIG records)
- Ad flag (authenticated data)

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

### `alpn` (Application Layer Protocol)
- `h2` = HTTP/2
- `h3` = HTTP/3 (QUIC)
- Tells agents which protocols you support

### `ech` (Encrypted Client Hello)
- Optional parameter for encrypted SNI
- Enhances privacy for agent requests

### Priority
- Lower numbers = higher priority
- Set to `1` for immediate selection

## Testing with External Tools

- **[DNS Checker](https://dnschecker.org)** - Verify DNS propagation
- **[MXToolbox](https://mxtoolbox.com)** - DNSSEC validation
- **[IsItAgentReady](https://isitagentready.com)** - Agent discoverability score

## Security Best Practices

✅ **DO:**
- Keep DNSSEC enabled
- Monitor DNS changes in Cloudflare audit logs
- Use TTL of 3600 or higher
- Test records after changes
- Document all DNS modifications

❌ **DON'T:**
- Disable DNSSEC (breaks agent validation)
- Set TTL below 300 seconds
- Mix DNS only and proxied records
- Change records without testing

## Standards & References

| Standard | Purpose | Link |
|----------|---------|------|
| RFC 8288 | Web Linking | https://tools.ietf.org/html/rfc8288 |
| RFC 9460 | SVCB/HTTPS RRs | https://www.rfc-editor.org/rfc/rfc9460 |
| RFC 4033-4035 | DNSSEC | https://tools.ietf.org/html/rfc4033 |
| DNS-AID Draft | AI Discovery | https://datatracker.ietf.org/doc/draft-mozleywilliams-dnsop-dnsaid/ |

## Troubleshooting

### Record not showing in DNS lookups
- Wait 5-10 minutes for propagation
- Clear local DNS cache: `sudo systemctl restart systemd-resolved`
- Verify record was saved in Cloudflare

### DNSSEC validation failing
- Check DS records are published by your registrar
- Verify DNSSEC status: `delv @8.8.8.8 apartca.org`
- Wait 24-48 hours for full propagation

### Agents not discovering your domain
- Ensure both `_index._agents` and `_a2a._agents` records exist
- Verify HTTP fallback endpoints are accessible
- Check `.well-known/ai-discoverability.json` is valid JSON

## Support & Questions

- **Contact**: hello@apartca.org
- **GitHub Repository**: https://github.com/apartca/apartca
- **Issue Tracker**: https://github.com/apartca/apartca/issues
- **Specification Discussions**: https://datatracker.ietf.org/doc/draft-mozleywilliams-dnsop-dnsaid/

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-05-31 | 1.0 | Initial DNS-AID setup guide |

---

**Last Updated**: 2026-05-31  
**Domain**: apartca.org  
**DNSSEC Status**: ✅ Enabled  
**Agent Discovery**: ✅ Ready
