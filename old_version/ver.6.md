# Free Knowledge Format (FKF) Specification – Version 1.0 (Revised)

**Author:** The FKF Design Team  
**Status:** Final Draft  
**Date:** 2026-07-05  

---

## 1. Introduction

The **Free Knowledge Format (FKF)** is a container‑oriented, human‑readable, and AI‑friendly file format designed to bundle multiple knowledge documents – each with its own metadata header and content body – into a single file.  

FKF retains the atomicity of each knowledge unit (as in OKF) while solving the “many small files” problem. It is built on top of XML to avoid rendering conflicts with Markdown and other textual content, and it explicitly supports both full and shorthand notations to ease manual editing and reduce file size.

**Primary use cases:**
- Building a personal knowledge base that can be shared as a single file.
- Feeding structured knowledge to AI agents and models.
- Exchanging curated knowledge between different systems without file‑system dependencies.
- Allowing multiple contributors to sign their own documents within a single container.

---

## 2. Core Principles

- **One file – many documents** – all knowledge units reside in one container.  
- **Path fidelity** – each document keeps its logical path, enabling hierarchical organisation and cross‑references.  
- **Human & machine readable** – clear, symmetric tags and inline comments make the file easy to understand for both people and AI agents.  
- **Flexible content types** – headers and data can use any format (YAML, JSON, plain text, etc.), specified by the `type` attribute.  
- **Optional escaping** – CDATA is supported but not mandatory; simple text nodes can be used when safe.  
- **Dual‑level digital signatures** – the container can be cryptographically signed as a whole, and/or each individual document can be signed separately, ensuring integrity and authenticity at both levels.

---

## 3. File Structure (Full Form)

The root of an FKF file is `<fkf_root>`, which contains one or more `<fkf_document>` elements and, optionally, a `<fkf_spec>` element. Digital signatures can appear in two places:

- **Root‑level signature** – placed as the last child of `<fkf_root>` (covers the whole container, including `<fkf_spec>` and all documents).
- **Document‑level signature** – placed inside each `<fkf_document>` (covers that specific document’s header and data, plus its attributes).

A typical full‑form file with both levels of signatures:

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<fkf_root doc_root_dir="./local_dir_name"
          xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  <fkf_spec>
    A brief description of the features of the FKF document format for AI:
    1. File format extensions .fkf.md for compatibility with Markdown editors during editing, or .fkf when distributing or automatically generating the file.
    2. a heredoc‑like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end.
    3. Digital signatures: root‑level (covers all) and/or per‑document (covers individual entries).
  </fkf_spec>

  <fkf_document doc_path="./relative_doc_path" doc_name="file_name.md">
    <fkf_header type="OKF YAML">...</fkf_header>
    <fkf_data type="OKF md"><![CDATA[...]]></fkf_data>
    <ds:Signature>
      <!-- Signature over this fkf_document element (including its attributes and children) -->
    </ds:Signature>
  </fkf_document>

  <!-- more fkf_document elements, each may have its own signature -->

  <ds:Signature>
    <!-- Signature over the entire fkf_root (including fkf_spec and all documents, but excluding this root signature itself) -->
  </ds:Signature>
</fkf_root>
```

**Important:**  
- The root signature covers everything inside `<fkf_root>` **except** the root signature element itself. It **does** cover the `<fkf_spec>` and all document signatures (if present).  
- A document‑level signature covers the entire `<fkf_document>` element, including its `doc_path`, `doc_name`, `<fkf_header>`, `<fkf_data>`, **but not** any sibling elements outside the document.  
- Either level can be used independently; both are optional.

---

## 4. Elements and Attributes

| Element / Attribute | Description |
|---------------------|-------------|
| `<fkf_root>` | Container for all documents. Attributes: `doc_root_dir` – base directory for all document paths (optional, defaults to `./`). May also declare XML namespaces for signatures. |
| `<fkf_spec>` | (Optional) Human‑ and machine‑readable description of file‑specific conventions, custom types, and special rules. Placed inside `<fkf_root>` so it can be covered by the root signature. |
| `<fkf_document>` | A single knowledge unit. Attributes: `doc_path` – relative path (from `doc_root_dir`) and `doc_name` – file name (may be empty for virtual documents). |
| `<fkf_header>` | Metadata of the document. Attribute `type` – format of the header content (e.g., `YAML`, `JSON`, `TOML`). Content is plain text (optionally wrapped in CDATA). |
| `<fkf_data>` | Main content of the knowledge document. Attribute `type` – format of the body (e.g., `Markdown`, `HTML`, `text`). Content can be plain text or CDATA‑escaped. |
| `<ds:Signature>` | (Optional) XML Digital Signature (W3C XMLDSig). Can appear as a child of `<fkf_root>` (root signature) or as a child of `<fkf_document>` (document signature). |

---

## 5. Digital Signatures – Detailed Specification

### 5.1. Purpose and Scope

- **Root signature** – ensures the integrity and authenticity of the entire knowledge container. It protects the container structure, the specification (`<fkf_spec>`), and all documents as a whole. Useful when the file is distributed by a single authority.
- **Document signature** – allows individual documents to be signed by different authors or to be verified independently, even when the root is not signed. It protects the specific document's content and its path/name attributes.

### 5.2. Signature Placement

- **Root signature**: placed as the **last child** of `<fkf_root>`. The signature is computed over the canonicalised `<fkf_root>` element, excluding the `<ds:Signature>` itself (but including all other children and attributes).
- **Document signature**: placed as the **last child** of the corresponding `<fkf_document>` element. The signature is computed over the canonicalised `<fkf_document>` element, excluding its own `<ds:Signature>`.

### 5.3. Key Management

Each signature must contain a `<ds:KeyInfo>` element that either embeds the public key or provides a reference to a key server or a key identifier.

**Default key server**: `https://keys.fkf-format.org` – a public service for publishing FKF‑related keys.  
However, users may specify any custom key server, use an inline key, or use a certificate.

Example `<ds:KeyInfo>` for document signature:

```xml
<ds:KeyInfo>
  <ds:KeyName>Document Author (doc1)</ds:KeyName>
  <ds:RetrievalMethod URI="https://keys.fkf-format.org/author_doc1.asc"/>
</ds:KeyInfo>
```

### 5.4. Verification Rules

- A consumer **must** verify any signature present. If a root signature is present, it must be validated before trusting any content from the file. If validation fails, the entire file must be rejected.
- If document signatures are present, the consumer **should** verify each one independently. A document with an invalid signature **must** be rejected, even if the root signature is valid. This allows fine‑grained trust.
- If both root and document signatures are present, both must pass validation for the file to be considered fully trusted. However, a consumer might choose to trust only root‑signed documents or only individually signed ones, depending on policy.

### 5.5. Recommendations

- Use root signature when the whole knowledge base comes from a single trusted source.
- Use document signatures when multiple authors contribute, or when documents may be extracted and used outside the container.
- Keep the `<fkf_spec>` up‑to‑date with signature policies (e.g., which keys are used for which documents).
- Test signature verification with a clean AI model to ensure it works as expected.

---

## 6. Shorthand Notation (Reaffirmed)

All shorthand rules from previous version remain valid. In shorthand mode, signatures are still possible by adding the `<ds:Signature>` elements explicitly (either inside `<fkf_document>` or at the root). Minimal example with a document signature:

```xml
<!--FKF-->
<fkf_root>
  <fkf_document>
    <fkf_data># Signed note</fkf_data>
    <ds:Signature>...</ds:Signature>
  </fkf_document>
</fkf_root>
```

---

## 7. CDATA Usage (Unchanged)

CDATA is optional but recommended for content that may contain XML special characters. For headers, CDATA is also optional. The heredoc‑style ending (`</CDATA>&#10;]]>`) is supported to avoid premature termination.

---

## 8. Header and Data Types (Unchanged)

The `type` attribute values remain the same (`YAML`, `JSON`, `TOML`, `Markdown`, `HTML`, `text`). Custom types must be described in `<fkf_spec>`.

---

## 9. Path and Root Attributes (Unchanged)

Same as previous specification. `doc_root_dir` + `doc_path` + `doc_name` form the logical full path for each document.

---

## 10. The `fkf_spec` Element (Enhanced)

The `<fkf_spec>` element now **should** include information about signature usage, such as:
- Which keys are used for root and/or document signatures.
- Any special processing rules for signature verification (e.g., allowed algorithms).
- How to obtain public keys if not embedded.

Example:

```xml
<fkf_spec>
  Standard OKF style: YAML header + Markdown body.
  Signatures: root signed by 'maintainer@example.com' (key server: https://keys.fkf-format.org/maintainer.asc).
  Document signatures are used for individual contributions; each document's key is referenced inside its signature.
</fkf_spec>
```

---

## 11. Versioning and MIME Type (Unchanged)

- Version declared in top comment, e.g., `version="1.0"`.
- MIME type: `text/x.fkf+xml; charset=UTF-8`.
- File extensions: `.fkf.md` (editing) and `.fkf` (distribution).

---

## 12. Best Practices (Updated)

- **Always include a top‑of‑file comment** with version and MIME type.  
- **Provide a detailed `<fkf_spec>`** to document signature policies and custom conventions.  
- **Use CDATA** for any content that may contain `<`, `&`, or `]]>`.  
- **Choose signature level(s) wisely**: root for whole‑file integrity, document signatures for per‑author accountability.  
- **Test verification** after creation using a clean AI model to ensure signatures are correctly processed.  
- **Keep key references stable** – if you change keys, update the signatures.

---

## 13. Full Example (with Both Signature Levels)

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<fkf_root doc_root_dir="./my_knowledge"
          xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  <fkf_spec>
    This file contains two documents. Root signed by maintainer@example.com.
    Each document is signed by its respective author.
    Links use [[path/to/doc]] syntax.
  </fkf_spec>

  <fkf_document doc_path="./concepts" doc_name="rest.md">
    <fkf_header type="YAML">title: REST API
tags: [web, api]</fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# REST API
...
]]></fkf_data>
    <ds:Signature>
      <ds:SignedInfo>...</ds:SignedInfo>
      <ds:SignatureValue>...</ds:SignatureValue>
      <ds:KeyInfo>
        <ds:KeyName>Alice</ds:KeyName>
        <ds:RetrievalMethod URI="https://keys.fkf-format.org/alice.asc"/>
      </ds:KeyInfo>
    </ds:Signature>
  </fkf_document>

  <fkf_document doc_path="./concepts" doc_name="graphql.md">
    <fkf_header type="YAML">title: GraphQL
tags: [api, query]</fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# GraphQL
...
]]></fkf_data>
    <ds:Signature>
      <ds:SignedInfo>...</ds:SignedInfo>
      <ds:SignatureValue>...</ds:SignatureValue>
      <ds:KeyInfo>
        <ds:KeyName>Bob</ds:KeyName>
        <ds:RetrievalMethod URI="https://keys.fkf-format.org/bob.asc"/>
      </ds:KeyInfo>
    </ds:Signature>
  </fkf_document>

  <ds:Signature>
    <ds:SignedInfo>
      <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
      <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
      <ds:Reference URI="">
        <ds:Transforms>
          <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
        </ds:Transforms>
        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <ds:DigestValue>...</ds:DigestValue>
      </ds:Reference>
    </ds:SignedInfo>
    <ds:SignatureValue>...</ds:SignatureValue>
    <ds:KeyInfo>
      <ds:KeyName>Maintainer</ds:KeyName>
      <ds:RetrievalMethod URI="https://keys.fkf-format.org/maintainer.asc"/>
    </ds:KeyInfo>
  </ds:Signature>
</fkf_root>
```

---

## 14. Compatibility and Future Extensions

FKF is designed to be forward‑compatible. New attributes or elements may be added in future versions, but they must be ignored by older parsers. The core structure (`<fkf_root>`, `<fkf_document>`, `<fkf_data>`, `<fkf_spec>`, and signature placements) is stable.

---

## 15. Testing Recommendations (Reaffirmed)

After creating or modifying an FKF file, always test its interpretation by feeding it to an AI model or agent in a clean environment (without prior context about FKF). Specifically test:

- Correct parsing of all documents and attributes.
- Understanding of the `<fkf_spec>` instructions.
- Validation of signatures (if present) – ensure that invalid signatures are rejected.
- Resolution of internal links and paths.

Testing prevents hidden ambiguities and ensures that the file is ready for distribution.

---

**End of Specification**  
*This document is the normative reference for FKF version 1.0.*
