# eMudhra CA key compromise: my notes

**Source:** [](https://unit42.paloaltonetworks.com/emudhra-certificate-compromise/)
**Published:** October 22, 2026
**Type:** Advisory

## 1. Summary
- Eight intermediate CA keys at eMudhra were made with weak randomness, so attackers could work out the private keys.
- A CA private key signs certificates, so with it an attacker could forge certificates that browsers trust.
- The fix is to distrust and revoke the affected certificates, replace anything that relied on them, and check the logs for misuse.

## 2. What happened
A certificate authority (CA) is like a passport office. It vouches for websites so my browser knows who to trust. Big CAs use intermediate CAs, which are like branch offices that issue certificates for them.

eMudhra made the keys for eight of these branch offices without enough randomness. When a key is not random enough, there are only a few possible values, so someone can work backwards from the public certificate and find the private key.

The private key is the signing stamp. Whoever has it can sign a certificate for any website, and browsers would believe it. Browser and OS vendors stopped trusting these CAs, and eMudhra revoked the certificates.

## 3. Why it matters
My browser shows a padlock when a certificate is signed by a CA it trusts. If an attacker has a trusted signing key, they can sit between me and a website, read or change the traffic, pretend to be the site, or take over my session. I would see nothing wrong. Anyone who used or trusted these CAs was at risk.

## 4. Was it exploited?
The article explains the risk and the response, but it does not say attackers were using the keys. So I call this a serious exposure, not a confirmed attack.

## 5. MITRE ATT&CK mapping (potential techniques)
| Technique | ID | Why it fits |
|---|---|---|
| Adversary-in-the-Middle | T1557 | Fake certificates make interception possible |
| Steal or Forge Authentication Certificates | T1649 | Derived keys allow forged certificates |

6. Response steps
- **Find:** check which of my systems use or trust certificates from these CAs, including apps, containers and trust stores.
- **Fix:** replace those certificates with ones from a trusted CA, push updated trust settings, and make sure revoked certificates get blocked.
- **Hunt:** look through proxy, network and login logs for strange certificate errors, unknown issuers, or sessions that look hijacked.
- **Harden:** watch public Certificate Transparency logs for my domains, limit which CAs may issue for them with CAA records, and use short-lived certificates that renew automatically.

## 7. How I would detect it
I would start with the proxy and network logs and look for certificate errors, unexpected issuers, or a jump in failed TLS handshakes. Then I would check login logs for sessions that look replayed or come from odd places. I'm used to reading server logs to find what changed before a problem, and I would use the same habit here.

## 8. What I learned and what I still want to learn
- A CA private key is used for signing, not for decrypting. If someone gets it, they can forge trust.
- Weak randomness makes secret numbers predictable, because there are only a few possible values to guess.
- I want to learn how Certificate Transparency logs work and how to monitor them.

## References
- Unit 42: EMudhra Intermediate CA Certificates Compromised
