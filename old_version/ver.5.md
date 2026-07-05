# Free Knowledge Format (FKF) Specification – Version 2.0  
**Author**: The FKF Design Team  
**Date**: 2026-07-05  

---

## 0. Document Status

This document defines the **Free Knowledge Format (FKF) version 2.0**, an XML‑based container for bundling multiple knowledge documents with optional cryptographic signing.  

It supersedes version 1.0 and adds **digital signature support**, **key server references**, and **enhanced integrity guarantees** covering both the content (`fkf_root`) and the format description (`fkf_spec`).

---

## 1. Introduction

FKF is a human‑readable, AI‑friendly format for packaging many knowledge units (each with a header and body) into a single file. It preserves logical paths, supports multiple content types (YAML, Markdown, JSON, etc.), and allows shorthand notation for manual editing.

**Version 2.0** introduces:

- Optional **digital signatures** for authentication and integrity.  
- Protection of **both the knowledge container and the `fkf_spec`** (which defines per‑file conventions).  
- A **key server reference** mechanism, similar to Linux package signing, enabling trust verification without bundling public keys.  

---

## 2. Core Principles (unchanged)

- One file – many documents.  
- Path fidelity via `doc_path` and `doc_root_dir`.  
- Human & machine readable – symmetric, explicit tags.  
- Flexible content types via `type` attributes.  
- Optional CDATA escaping.  
- Shorthand notation for reduced verbosity.  

---

## 3. High‑Level File Structure

A full FKF v2.0 file consists of:

1. **File‑level comments** declaring version, encoding, file name, MIME type, and the `fkf_spec` (mandatory).  
2. An optional **`<fkf_signature>`** element (if signing is used).  
3. The **`<fkf_root>`** container with all documents.  

The signature, if present, is placed **before** `<fkf_root>` and covers:
- The entire `<fkf_root>` element (including its contents).  
- The `fkf_spec` comment (since it defines parsing rules).  

---

## 4. File‑Level Comments (Mandatory)

```xml
<!-- FKF free knowledge format version="2.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="... (user defined conventions) ..." -->
```

- `fkf_spec` is **required** – it must describe any custom types, link syntax, or special processing rules used in the file.  
- The `fkf_spec` content is considered part of the signed payload when a signature is present.

---

## 5. The `<fkf_signature>` Element (Optional)

If digital signing is desired, include this element **immediately after the comments and before `<fkf_root>`**:

```xml
<fkf_signature algorithm="ed25519" key_server="https://keys.example.com" key_id="0xABCD1234">
  <SignedData>base64-encoded-signature</SignedData>
</fkf_signature>
```

**Attributes**:

| Attribute | Description |
|-----------|-------------|
| `algorithm` | Cryptographic algorithm used (e.g., `ed25519`, `rsa-pss`, `ecdsa-p256`). Mandatory. |
| `key_server` | URL of a key server where the public key can be retrieved (optional but recommended). |
| `key_id` | Identifier of the public key (fingerprint, short ID, or email). Optional if `key_server` provides discovery. |
| `key_uri` | Direct URI to the public key (alternative to `key_server` + `key_id`). |

**Content**:
- Base64‑encoded signature of the **canonicalised** signed payload (see Section 6).

---

## 6. Signed Payload and Canonicalisation

The signature **MUST** cover:

1. The **entire `fkf_spec` comment** (from `<!-- fkf_spec="` to the closing `-->`).  
2. The **entire `<fkf_root>` element**, including all its attributes and children, preserving the exact byte sequence after the file is loaded into a DOM (or using a standard XML canonicalisation method).

**Canonicalisation** – to avoid ambiguity, use **XML Exclusive Canonicalisation (C14N)** with comments included (`<Signatures>` `http://www.w3.org/2001/10/xml-exc-c14n#`). This produces a deterministic byte stream for signing.

**Verification steps**:
1. Extract the `fkf_spec` and `<fkf_root>` from the file (excluding the `<fkf_signature>` element itself).  
2. Canonicalise them together (in that order).  
3. Verify the signature against the canonicalised bytes using the specified algorithm and the public key obtained from `key_server` (or `key_uri`).  

**Note**: If the file is edited after signing, the signature becomes invalid, alerting the consumer that the contents may have been tampered with.

---

## 7. Key Server Reference

To enable verification without embedding public keys, the format supports:

- **`key_server`** – a base URL (e.g., `https://keys.fkf.io`) that follows the **Web Key Directory (WKD)** or a simple GET request pattern:  
  `{key_server}/pks/lookup?op=get&search=0x{key_id}` (similar to OpenPGP HKP).  
- **`key_id`** – can be a fingerprint, a short hex ID, or an email (if the server supports email‑based lookup).  

If both are provided, the consumer attempts to retrieve the key. If the key cannot be fetched, the signature check **MAY** be skipped (if the consumer policy allows), but the presence of a signature without a retrievable key should be treated as a warning.

---

## 8. Shorthand Notation (Updated)

All shorthand rules from v1.0 remain valid. However, if a signature is present, **the short comment `<!--FKF-->` is not sufficient** – the full version comment must be used to include the `fkf_spec` because the spec is part of the signed payload.  

Example of minimal signed shorthand:

```xml
<!-- FKF version="2.0" file_name="doc.fkf" -->
<!-- fkf_spec="Plain Markdown only, no custom types." -->
<fkf_signature algorithm="ed25519" key_server="https://keys.fkf.io" key_id="0xDEADBEEF">
  <SignedData>...</SignedData>
</fkf_signature>
<fkf_root>
  <fkf_document>
    <fkf_data># Hello</fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 9. CDATA and Escaping – Unchanged

- CDATA usage remains **optional** but recommended for data containing `<`, `&`, or `]]>`.  
- The heredoc‑style CDATA ending (`</CDATA>&#10;]]&gt;`) is still supported for safe termination.  
- Signatures are always applied to the canonicalised **byte** content; CDATA markers are treated as literal characters during canonicalisation.

---

## 10. Header and Data Types – Unchanged

Same as v1.0: `type` attribute in `<fkf_header>` and `<fkf_data>` can be any format; default is `YAML` for header, `Markdown` for data.

---

## 11. Path and Root Attributes – Unchanged

`doc_root_dir`, `doc_path`, `doc_name` preserve the logical file hierarchy.

---

## 12. Versioning and MIME – Extended

- **Version** in top comment: `version="2.0"`.  
- **MIME Type**: `text/x.fkf+xml; charset=UTF-8` (unchanged).  
- **File Extensions**: `.fkf.md` for editing, `.fkf` for distribution.

---

## 13. Best Practices (Updated)

- **Always include a detailed `fkf_spec`** – it is now part of the signed contract.  
- **Use a key server** for public key distribution; avoid embedding keys directly to reduce file size.  
- **Test signature verification** with at least one consumer (AI agent or tool) after each signed release.  
- **For manual editing**, omit the signature until the final version is ready; sign only before distribution.

---

## 14. Complete Example (Signed, Full Form)

```xml
<!-- FKF free knowledge format version="2.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="OKF style: YAML header, Markdown body, [[link]] references." -->
<fkf_signature algorithm="ed25519" key_server="https://keys.fkf.io" key_id="0xA1B2C3D4">
  <SignedData>ZGVtb25zdHJhdGlvbiBzaWduYXR1cmUgaGVyZQ==</SignedData>
</fkf_signature>
<fkf_root doc_root_dir="./knowledge">
  <fkf_document doc_path="./concepts" doc_name="rest.md">
    <fkf_header type="YAML">title: REST API
tags: [web, api]</fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# REST API
...
]]></fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 15. Security Considerations

- The signature does **not** encrypt the content – it only provides integrity and origin authentication.  
- If the key server is unavailable, consumers **may** cache previously fetched keys or rely on a local trust store.  
- The `fkf_spec` itself is signed, so an attacker cannot alter the parsing rules without breaking the signature.  

---

## 16. Forward Compatibility

Future versions of FKF may introduce new elements or attributes. Parsers **SHOULD** ignore unknown elements but **MUST** preserve the signature‑covered canonicalisation order. The root element may gain new attributes, but the signature will still cover the entire `<fkf_root>` as long as the canonicalisation includes all content.

---

## 17. Testing and Validation

- Validate the XML syntax (well‑formedness).  
- Verify the signature using the `algorithm` and public key from the key server.  
- If the signature is invalid, the consumer **SHOULD** reject the file (unless policy allows fallback).

---

**End of Specification**  
*This document is the normative reference for FKF version 2.0.*
