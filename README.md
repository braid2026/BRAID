# BRAID
Rapid structural multi-strand architecture ends legacy TLS certificate revocation delays. An optional independent-witness strand adds third-party revocation any operator can provide. Reference implementation for draft-davey-tls-braid.

Reference implementation and Phase 0 monitor for [`draft-davey-tls-braid`](https://datatracker.ietf.org/doc/draft-davey-tls-braid/).

## The problem

TLS revocation has never worked reliably at internet scale. OCSP soft-fails. CRLs go unchecked. The industry's answer has been to shrink certificate lifetimes so a bad certificate expires before it can do much harm — trading a permanent operational burden for a problem that was never fixed.

## The approach

BRAID makes a certificate's freshness a precondition of its use, published and controlled by the certificate's own owner rather than a third-party status service.

A short-lived Delegated Credential ([RFC 9345](https://www.rfc-editor.org/rfc/rfc9345)) carries the freshness. Its authorized keys are listed in a DNSSEC-signed `_braid` record under the owner's domain. A certificate validates only while its live credential is authorized there. Withdraw the record, and validity lapses within the freshness window — no OCSP query, no CRL, no CA involved, no waiting.

Validity lapses unless the owner keeps authorizing it. Revocation is therefore structural: it is a property of the system, not an action anyone must remember to take.

## The strands

BRAID weaves independent checks, each anchored in a different trust system, so defeating one is not enough:

- **Authority** — the ordinary CA-signed X.509 certificate. Unchanged. BRAID layers on top of the existing WebPKI; it does not replace it.
- **Identity** — the DNSSEC-anchored freshness credential described above.
- **Routing** *(optional)* — the certificate is bound to a network origin authorized in the routing system (RPKI), checked at use time.
- **Witness** *(optional)* — an independent third party co-signs the certificate's freshness. The role is open: it requires only a signing key and a supported ledger, and no relationship to the WebPKI. A registrar, DNS operator, auditor, or CA can serve as a witness. With witnessed freshness enabled, the co-signature is renewed each window, providing a revocation lever that holds even if both the owner's DNS and the certificate's key are compromised.

Everything reuses mechanisms already deployed — Delegated Credentials, the TLS Feature extension, RFC 3779 resource types, Certificate Transparency structures, DNSSEC. Nothing here requires a new TLS handshake, a browser change, a public-CA change, or a root-program change to begin.

## Status

This is early, individual work. The Internet-Draft is filed; the design has been through several review passes. A Phase 0 monitor — which checks the credential a TLS endpoint serves against its DNSSEC-signed `_braid` record, entirely out of band and deployable today — is being demonstrated at the **IETF 126 Hackathon (Vienna, July 2026)**, running against real infrastructure: live domains, a private CA, and origin-AS space operated by the author. Code and results will land here around that event.

## Draft

- Internet-Draft: https://datatracker.ietf.org/doc/draft-davey-tls-braid/

## Author

George Davey · BRAID@cpu.io · [ORCID 0009-0005-8396-4976](https://orcid.org/0009-0005-8396-4976)

Feedback is welcome, especially critical feedback. The fastest way to improve this is to find its next problem.

## License

Apache License 2.0.
