# Free Knowledge Format (FKF) Specification  
**Version 1.0**  
**Author**: The FKF Design Team  

---

## 1. Introduction

The **Free Knowledge Format (FKF)** is a container‑oriented, human‑readable, and AI‑friendly file format designed to bundle multiple knowledge documents – each with its own header and body – into a single file.  

FKF retains the atomicity of each knowledge unit (as in OKF) while solving the “many small files” problem. It is built on top of XML to avoid rendering conflicts with Markdown and other textual content, and it explicitly supports both full and shorthand notations to ease manual editing and reduce file size.

---

## 2. Core Principles

- **One file – many documents** – all knowledge units reside in one container.  
- **Path fidelity** – each document keeps its logical path, enabling hierarchical organisation and cross‑references.  
- **Human & machine readable** – clear, symmetric tags and inline comments make the file easy to understand for both people and AI agents.  
- **Flexible content types** – headers and data can use any format (YAML, JSON, plain text, etc.), specified by the `type` attribute.  
- **Optional escaping** – CDATA is supported but not mandatory; simple text nodes can be used when safe.  
- **Optional digital signature** – the whole file (including `fkf_spec` comments) can be signed to guarantee authenticity and integrity.

---

## 3. File Structure

The root of an FKF file is `<fkf_root>`, which contains one or more `<fkf_document>` elements. An optional `<fkf_signature>` element may appear as the **first child** of `<fkf_root>`.

A typical full‑form file looks like:

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="..." -->
<fkf_root doc_root_dir="./local_dir_name">
  <fkf_signature algorithm="rsa-sha256" key-id="...">
    Base64EncodedSignatureHere
  </fkf_signature>
  <fkf_document doc_path="./relative_doc_path" doc_name="file_name.md">
    <fkf_header type="OKF YAML">...</fkf_header>
    <fkf_data type="OKF md"><![CDATA[...]]></fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 4. Elements and Attributes

| Element / Attribute | Description |
|---------------------|-------------|
| `<fkf_root>` | Container for all documents. Attribute `doc_root_dir` – base directory for all document paths (optional, defaults to `./`). |
| `<fkf_signature>` | Optional digital signature. Must be the first child of `<fkf_root>`. Attributes: `algorithm` (e.g., `rsa-sha256`, `ed25519`), `key-id` (identifier of the public key). Content is the Base64‑encoded signature itself. |
| `<fkf_document>` | A single knowledge unit. Attributes: `doc_path` – relative path (from `doc_root_dir`) and `doc_name` – file name (may be empty, for virtual documents). |
| `<fkf_header>` | Metadata of the document. Attribute `type` – format of the header content (e.g., `YAML`, `JSON`, `TOML`). Content is plain text (optionally wrapped in CDATA). |
| `<fkf_data>` | Main content of the knowledge document. Attribute `type` – format of the body (e.g., `Markdown`, `HTML`, `text`). Content can be plain text or CDATA‑escaped. |
| `<!-- FKF ... -->` | Top‑of‑file comments declaring version, encoding, file name, MIME type, and the `fkf_spec` (see below). |

---

## 5. Digital Signature (Optional)

### Purpose
To allow an author or distributor to sign the entire FKF file, ensuring that both the content (`<fkf_root>` and its descendants) and the crucial meta‑information (`fkf_spec` comment) have not been altered.

### Placement
The `<fkf_signature>` element **must** be the first child of `<fkf_root>`. If present, it signals that the file is signed.

### Signature Computation
1. **Canonical form**: Take the **byte‑for‑byte** content of the whole file (including all comments, whitespace, and line breaks) **except** the `<fkf_signature>` element and its entire content (from the opening `<fkf_signature ...>` to the closing `</fkf_signature>`).  
2. Compute a cryptographic hash (e.g., SHA‑256) over that byte sequence.  
3. Sign the hash using the specified algorithm and private key.  
4. Encode the resulting signature in Base64 and place it as the text content of the `<fkf_signature>` element.

### Verification
1. Locate the `<fkf_signature>` element. If absent, no verification is performed.  
2. Remove that element (including its tags and content) from the file in memory.  
3. Compute the hash of the remaining byte sequence exactly as it appears (including all other comments and CDATA).  
4. Decode the Base64 signature and verify it against the hash using the public key identified by `key-id` (or a key embedded by other means, e.g., separate certificate).

### Security Considerations
- The signature **protects** the `fkf_spec` comment because that comment is part of the file content outside `<fkf_root>`. Since we sign the entire file except the signature element itself, all top‑level comments (including `fkf_spec`) are covered.  
- The `key-id` attribute should allow the verifier to retrieve the correct public key (e.g., from a trusted key server or embedded in the file as a separate `<fkf_certificate>` element – not defined here, but implementers may extend).  
- Recommended algorithms: `rsa-sha256` (RSA with PKCS#1 v1.5), `rsa-pss-sha256`, `ed25519` (for modern, shorter signatures).  

### Example
```xml
<fkf_root>
  <fkf_signature algorithm="ed25519" key-id="author@example.com">
    TGlrZSB0aGlzIGlzIGEgZmFrZSBzaWduYXR1cmUuLi4=
  </fkf_signature>
  <!-- ... documents ... -->
</fkf_root>
```

---

## 6. Shorthand Notation

To ease manual editing and reduce size when many small documents are present, the following abbreviations are **allowed**:

- `<!--FKF-->` – replaces the full top‑of‑file comment block (version, name, etc.) when defaults are used.  
- `<fkf_root>` – may omit `doc_root_dir` if it equals `./`.  
- `<fkf_document>` – may omit `doc_path` if the document is at the root (`./`), and `doc_name` if the content is virtual (only data, no file name).  
- `<fkf_header>` – may be omitted entirely if no metadata is required.  
- `type` attributes – if omitted, the parser assumes `"YAML"` for headers and `"Markdown"` for data.  
- `<fkf_signature>` – may be omitted when no signature is needed.

A minimal shorthand file (unsigned):

```xml
<!--FKF-->
<fkf_root>
  <fkf_document>
    <fkf_data>Just a single virtual document in Markdown</fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 7. CDATA Usage

**CDATA is optional but recommended** for blocks that contain characters that could be misinterpreted as XML markup (e.g., `<`, `&`, `]]>`).

- For `<fkf_data>`, if you are certain that the content does not contain problematic characters, you can use plain text.
- For `<fkf_header>`, similarly, CDATA may be used if the header contains raw XML or special characters.
- CDATA **may** also be used inside `<fkf_signature>` if the Base64 string contains characters that might be misinterpreted, though Base64 is safe.

**Escaping the CDATA end‑marker** – if your content may include the literal sequence `]]>`, you may use the heredoc‑style ending:
```
<![CDATA[ ... </CDATA>
]]> 
```
This is a custom extension that prevents premature termination.

---

## 8. Header and Data Types

The `type` attribute in `<fkf_header>` and `<fkf_data>` tells the consumer how to interpret the content. Recommended values:

| Type | Description |
|------|-------------|
| `YAML` | YAML frontmatter (default for header) |
| `JSON` | JSON object |
| `TOML` | TOML configuration |
| `Markdown` | Markdown text (default for data) |
| `HTML` | HTML fragment |
| `text` | Plain text |

You are free to define custom types, but they should be documented in `fkf_spec`.

---

## 9. Path and Root Attributes

- `doc_root_dir` – base directory for all `doc_path` values. It may be absolute or relative. When resolving internal links (e.g., `[[path/to/doc]]`), the consumer should use the full logical path: `doc_root_dir + doc_path + doc_name`.  
- If `doc_path` is omitted, the document is considered located at the root.  
- The combination of `doc_path` and `doc_name` should be unique within the file.

---

## 10. The `fkf_spec` Comment

The top‑of‑file comment containing `fkf_spec` is **essential** for AI agents and human readers. It describes:

- The version and encoding of the file.  
- Custom conventions used in this particular file (e.g., type extensions, special formatting rules).  
- Any deviations from the standard specification.  

**Recommendation**: Provide a detailed `fkf_spec` to reduce cognitive load and prevent misinterpretation. For example:

```
<!-- fkf_spec="This file uses YAML headers with mandatory 'title' and 'tags', and Markdown data with custom [[...]] links for cross‑references." -->
```

You may also include the `fkf_spec` as a separate `<!-- ... -->` comment (as shown) or as a dedicated element (if you prefer). The comment form is simpler and does not interfere with the XML structure.

**Important**: Because the `fkf_spec` comment is outside `<fkf_root>`, it is **not** protected by the XML signature unless the signature covers the entire file (as described in Section 5). Our signature method does cover all file content except the signature element, so `fkf_spec` is always signed if a signature is present.

---

## 11. Best Practices

- **Always include a top‑of‑file comment** with version and MIME type, even in shorthand (use `<!--FKF-->` if defaults are sufficient).  
- **Set the `type` attribute** explicitly unless you rely on the defaults – this helps both humans and machines.  
- **Use CDATA** for any data that contains `<`, `&`, or `]]>` to avoid parsing errors.  
- **Keep `fkf_spec` up‑to‑date** whenever you introduce custom conventions.  
- **Organise documents logically** using `doc_path` to reflect a folder hierarchy (e.g., `theory/physics`, `entity/people`).  
- **If signing**, ensure the `<fkf_signature>` element is the very first child of `<fkf_root>` and that the signature is computed over the entire file *except* that element.  

---

## 12. Versioning and MIME Type

- **Version** – declared in the top comment, e.g., `version="1.0"`.  
- **MIME Type** – `text/x.fkf+xml; charset=UTF-8`.  
- **File Extensions** – use `.fkf.md` during editing (so that Markdown editors recognise the file) and `.fkf` for final distribution or automated processing.

---

## 13. Testing Recommendations

After creating or modifying an FKF file, always **test its interpretation** by feeding it to an AI model or agent in a clean environment. This ensures that:

- The `fkf_spec` is correctly understood.  
- All paths and types are resolved as intended.  
- The file is parsed without errors (e.g., no stray `]]>` outside CDATA).  
- **If signed**, verify that the signature validates correctly after any edits – this is crucial to prevent unnoticed tampering.  

Testing prevents the accumulation of hidden ambiguities that could confuse downstream consumers.

---

## 14. Example: Full‑Form and Shorthand

**Full form** (with CDATA, explicit types, and a signature):

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="Standard OKF style: YAML header + Markdown body." -->
<fkf_root doc_root_dir="./my_knowledge">
  <fkf_signature algorithm="rsa-sha256" key-id="alice@example.com">
    U2lnbmF0dXJlIGJ5IEFsaWNl...
  </fkf_signature>
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

**Shorthand form** (no CDATA, no signature, minimal attributes):

```xml
<!--FKF-->
<fkf_root>
  <fkf_document>
    <fkf_data># Just a note
No header, no path.</fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 15. Compatibility and Future Extensions

FKF is designed to be forward‑compatible. New attributes or elements may be added, but they must be ignored by older parsers. It is recommended to keep the root element `<fkf_root>` and the document wrapper `<fkf_document>` stable.

For the signature, future versions might support additional algorithms or include a certificate chain. Such extensions should be introduced via new attributes or child elements under `<fkf_signature>`.

---

**End of Specification**  
*This document is the normative reference for FKF version 1.0.*
