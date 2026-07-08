# BRAID

Owner-controlled, DNSSEC-anchored key authorization for TLS. A domain owner publishes which Delegated Credential keys may authenticate its service; a stolen certificate key alone is no longer enough to impersonate the domain. Optional extension strands add routing and independent-witness assurance — including a third-party revocation lever any operator can provide.

Reference implementation and Phase 0 monitor for [`draft-davey-tls-braid`](https://datatracker.ietf.org/doc/draft-davey-tls-braid/).

> **Revision -01 is complete and can be previewed in this repo** ([`draft-davey-tls-braid-01.txt`](./draft-davey-tls-braid-01.txt)). The IETF submission tool is closed for the pre-meeting blackout; -01 will be posted to the IETF datatracker on **July 18, 2026**, when it reopens. Review comments are welcome now — that's why the preview is here.

## The problem

TLS revocation has never worked reliably at internet scale. OCSP soft-fails. CRLs go unchecked. The industry's answer has been to shrink certificate lifetimes so a bad certificate expires before it can do much harm — trading a permanent operational burden for a problem that was never fixed.

## The approach

BRAID adds an authorization check the certificate's owner controls. The server authenticates with a Delegated Credential ([RFC 9345](https://www.rfc-editor.org/rfc/rfc9345)); the owner lists the credential keys it authorizes in a DNSSEC-signed `_braid` record under its own domain; a BRAID client accepts the credential only if its key is on that list.

The result is a **two-control property**: impersonating a protected domain requires *both* the certificate's private key *and* the ability to write the owner's DNSSEC-signed zone. An attacker who steals the certificate key can mint credentials, but can't put their keys into the owner's record — so the theft, on its own, is useless. And revocation becomes structural and selective: remove a key's entry and it is de-authorized everywhere, while service continues on the remaining authorized keys — no OCSP query, no CRL, no CA involved, no outage.

A key refinement in -01: this property holds even when the record and the credential key are **static**. Freshness — short credential windows with automated rotation — is now an optional profile (*BRAID-Agile*) for operators who want to bound stolen-credential exposure and revocation latency. The baseline (*BRAID-Base*) needs no recurring DNS or CA-facing automation at all: the only repeating task is re-signing a credential locally, with a key you already hold, on your own schedule — a shell script, not a third-party API.

## The strands

- **Authority** — the ordinary CA-signed X.509 certificate. Unchanged. BRAID layers on top of the existing WebPKI; it does not replace it.
- **Identity** — the DNSSEC-anchored credential authorization described above. This is the core of -01, with a normative validation algorithm, deterministic anchor-location rules, explicit DNSSEC trust modes, and ten negative test vectors.
- **Routing** *(future extension profile)* — binds the certificate to a network origin authorized in the routing system (RPKI), checked at use time. Deferred pending a validated-origin query standardized in DNSOP/SIDROPS.
- **Witness** *(extension profile, companion draft coming)* — an independent third party co-signs the certificate. The role is open: it requires only a signing key and a supported ledger, and no relationship to the WebPKI — a registrar, DNS operator, auditor, or CA can serve, under its own published terms of appointment. With witnessed freshness, the co-signature renews each window, providing the one revocation lever that holds even if both the owner's DNS and the certificate's key are compromised. Unlike Routing, this profile has no missing protocol dependency and is deployable in managed environments today; a short companion draft specifying it is planned to follow -01.

Everything reuses mechanisms already deployed — Delegated Credentials, the TLS Feature extension, X.509 GeneralSubtrees, Certificate Transparency structures, DNSSEC. Nothing here requires a new TLS handshake, a browser change, a public-CA change, or a root-program change to begin.

## What changed in -01

Revision -01 is a substantial narrowing and hardening in response to expert review on the TLS list:

- The **two-control property** is now the center of the document, stated as a consequence of anchor *membership*, independent of freshness cadence.
- A precise **comparison with DANE TLSA** (usage 1 and 3): a TLSA record vouches for exactly the object a thief steals; BRAID's anchor authorizes a key the thief cannot publish.
- **Two explicit DNSSEC validation modes** (Public-Web strict / Managed-Resolver) — no more implicit resolver trust, and no unauthenticated AD bits.
- **Graceful degradation is prohibited** for strict Identity. Fail-closed means fail-closed.
- **Deterministic anchor placement** (`_braid.<DNS-ID>`), multi-SAN and wildcard rules, and signed-indirection-only for external anchors.
- The five-year validity claim is **gone**: lifetimes are root-program policy, and the draft must be useful at prevailing lifetimes.
- A **normative validation algorithm** and an appendix of **negative test vectors** — the initial test suite for the monitor in this repo.
- Routing and Witness moved to extension profiles, with an IETF work-split roadmap (LAMPS / DNSOP / SIDROPS / TRANS).

## Status

This is early, individual work. Revision -00 is filed; **-01 posts July 18, 2026** and is previewable here now. A Phase 0 monitor — which checks the credential a TLS endpoint serves against its DNSSEC-signed `_braid` record, entirely out of band and deployable today — is being demonstrated at the **IETF 126 Hackathon (Vienna, July 2026)**, running against real infrastructure: live domains, a private CA, and origin-AS space operated by the author, and exercising the -01 negative test vectors (Appendix A). Code and results will land here around that event.

## Draft

- Internet-Draft (datatracker): https://datatracker.ietf.org/doc/draft-davey-tls-braid/
- Revision -01 preview: [`draft-davey-tls-braid-01.txt`](./draft-davey-tls-braid-01.txt) *(posts to IETF July 18, 2026)*

## Author

George Davey · BRAID@cpu.io · [ORCID 0009-0005-8396-4976](https://orcid.org/0009-0005-8396-4976)

Feedback is welcome, especially critical feedback. The fastest way to improve this is to find its next problem.

## License

Apache License 2.0.

