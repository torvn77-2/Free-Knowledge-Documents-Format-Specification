# Free Knowledge Documents Format # Free Knowledge Format (FKF) Specification – Version 1.0 (Final)

**Author:** torvn77@gmail.com  
**Status:** Final  
**Date:** 2026-07-05 16:06 PM  

---

## 1. Introduction

The **Free Knowledge Format (FKF)** is a container‑oriented, human‑readable, and AI‑friendly file format designed to bundle multiple knowledge documents – each with its own metadata header and content body – into a single file.  

FKF retains the atomicity of each knowledge unit (as in OKF) while solving the “many small files” problem. It is built on top of XML to avoid rendering conflicts with Markdown and other textual content, and it explicitly supports both full and shorthand notations to ease manual editing and reduce file size.

**Primary use cases:**
- Building a personal knowledge base that can be shared as a single file.
- Feeding structured knowledge to AI agents and models.
- Exchanging curated knowledge between different systems without file‑system dependencies.
- Allowing fine‑grained integrity protection: individual header and data blocks can be signed separately, and a root signature can tie everything together for full verification.

---

## 2. Core Principles

- **One file – many documents** – all knowledge units reside in one container.  
- **Path fidelity** – each document keeps its logical path, enabling hierarchical organisation and cross‑references.  
- **Human & machine readable** – clear, symmetric tags and inline comments make the file easy to understand for both people and AI agents.  
- **Flexible content types** – headers and data can use any format (YAML, JSON, plain text, etc.), specified by the `type` attribute.  
- **Optional escaping** – CDATA is supported but not mandatory; simple text nodes can be used when safe.  
- **Granular digital signatures** – individual header and data blocks can be signed independently; a root signature can cover the entire container, including those block signatures, to provide full integrity and authenticity.

---

## 3. File Structure (Full Form)

The root of an FKF file is `<fkf_root>`, which contains one or more `<fkf_document>` elements and, optionally, a `<fkf_spec>` element. Digital signatures can appear at two levels:

- **Block‑level signatures** – inside each `<fkf_document>`:  
  - `<fkf_header_signature>` (optional) – signs the `<fkf_header>` element (including its `type` attribute).  
  - `<fkf_data_signature>` (optional) – signs the `<fkf_data>` element (including its `type` attribute).  
- **Root‑level signature** – placed as the last child of `<fkf_root>`. It covers the whole container by referencing (via `ds:Reference`) the block signatures and/or the actual block contents, as desired.

**Important:** The document itself is **not** signed as a whole. Instead, the header and data are signed individually. This allows extraction and re‑use of a header or data block with its signature intact, without losing protection. The root signature then ties everything together, optionally referencing both the block signatures and the block contents to ensure the entire knowledge base is consistent.

A typical full‑form file with both header and data signatures, plus a root signature:

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<fkf_root doc_root_dir="./local_dir_name"
          xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  <fkf_spec>
    A brief description of the features of the FKF document format for AI:
    1. File extensions .fkf.md for editing, .fkf for distribution.
    2. heredoc‑like CDATA ending: <![CDATA[ ... </CDATA>&#10;]]&gt;.
    3. Digital signatures: header and data blocks can be signed individually.
       The root signature covers the entire container, referencing block signatures.
  </fkf_spec>

  <fkf_document doc_path="./relative_doc_path" doc_name="file_name.md">
    <fkf_header type="OKF YAML">...</fkf_header>
    <fkf_header_signature>
      <!-- Signature over this fkf_header element (including its type attribute) -->
    </fkf_header_signature>

    <fkf_data type="OKF md"><![CDATA[...]]></fkf_data>
    <fkf_data_signature>
      <!-- Signature over this fkf_data element (including its type attribute) -->
    </fkf_data_signature>
  </fkf_document>

  <!-- more fkf_document elements, each may have header/data signatures -->

  <ds:Signature>
    <!-- Root signature covering the whole fkf_root.
         It may reference the block signatures, the block contents, or both,
         using ds:Reference URIs. -->
  </ds:Signature>
</fkf_root>
```

---

## 4. Elements and Attributes

| Element / Attribute | Description |
|---------------------|-------------|
| `<fkf_root>` | Container for all documents. Attributes: `doc_root_dir` – base directory for all document paths (optional, defaults to `./`). May declare XML namespaces for signatures. |
| `<fkf_spec>` | (Optional) Human‑ and machine‑readable description of file‑specific conventions, custom types, and signature policies. Placed inside `<fkf_root>`. |
| `<fkf_document>` | A single knowledge unit. Attributes: `doc_path` – relative path (from `doc_root_dir`) and `doc_name` – file name (may be empty). |
| `<fkf_header>` | Metadata of the document. Attribute `type` – format of the header content. |
| `<fkf_header_signature>` | (Optional) XML Digital Signature over the `<fkf_header>` element. Uses standard XMLDSig with an enveloped transform. |
| `<fkf_data>` | Main content. Attribute `type` – format of the body. |
| `<fkf_data_signature>` | (Optional) XML Digital Signature over the `<fkf_data>` element. |
| `<ds:Signature>` | (Optional) XML Digital Signature at the root level, covering the entire container (excluding itself). It can reference any element via `ds:Reference`. |

---

## 5. Digital Signatures – Detailed Specification

### 5.1. Block‑Level Signatures (`<fkf_header_signature>` and `<fkf_data_signature>`)

- **Purpose**: Ensure the integrity and authenticity of an individual header or data block, independently of the rest of the document.
- **Scope**: The signature is computed over the entire `<fkf_header>` (or `<fkf_data>`) element, including its `type` attribute and all its content (text or CDATA).  
- **Placement**: Immediately after the corresponding `<fkf_header>` or `<fkf_data>` element, inside the `<fkf_document>`.  
- **Key Management**: Each block signature must contain a `<ds:KeyInfo>` that identifies the signer (key name, certificate, or key server reference).  
- **Verification**: The block signature can be verified independently, even when the document is extracted or moved to another container. If a block signature is present and invalid, that block **must** be rejected.

### 5.2. Root Signature (`<ds:Signature>` as last child of `<fkf_root>`)

- **Purpose**: Provide full container‑level integrity and authenticity. It ties together all documents, the specification, and optionally the block signatures.
- **Scope**: The root signature covers the entire `<fkf_root>` element (excluding the root `<ds:Signature>` itself). However, it uses `ds:Reference` elements to point to specific parts of the document, giving flexibility:
  - It **may** reference the `<fkf_spec>` element.
  - It **may** reference each `<fkf_document>` element, or individual `<fkf_header>` and `<fkf_data>` elements.
  - It **may** reference the block signatures (`<fkf_header_signature>` and `<fkf_data_signature>`) to ensure that those signatures are part of the container and haven't been tampered with.
  - Typically, the root signature will reference **both** the block content (e.g., the `<fkf_data>` elements) and the block signatures themselves, so that any change to the content or the signature is detected.
- **Verification**: The root signature must be validated first. If it is present, all referenced items must match their digests. Block signatures, if referenced, are then validated independently. If any referenced item fails, the whole file is rejected.

### 5.3. Relationship Between Block Signatures and Root Signature

- **Independence**: Block signatures can exist without a root signature. This allows a document to be extracted and used elsewhere with its own protection.
- **Root with block references**: The root signature can reference the block signatures themselves (by referencing the `<fkf_header_signature>` and `<fkf_data_signature>` elements). This ensures that the block signatures are part of the container and have not been replaced or removed.
- **Root with content references**: Alternatively, the root signature can directly reference the `<fkf_header>` and `<fkf_data>` elements, ignoring the block signatures. This gives container‑level integrity but does not enforce block‑level signing.
- **Recommended practice**: For a typical knowledge base, include both header and data signatures for each document, and have the root signature reference both the block signatures and the block content. This provides the strongest protection: the whole container is signed, each block is individually signed, and any extraction retains block‑level signatures.

### 5.4. Key Management and Default Key Server

- **Default key server**: `https://keys.fkf-format.org` – a public service for publishing FKF‑related keys.  
- **Inline keys**: Keys can be embedded inside `<ds:KeyInfo>` using `<ds:X509Data>` or `<ds:KeyValue>`.  
- **Custom key servers**: Any URI can be used in `<ds:RetrievalMethod>`.

Example `<ds:KeyInfo>` for a block signature:

```xml
<ds:KeyInfo>
  <ds:KeyName>Data Author</ds:KeyName>
  <ds:RetrievalMethod URI="https://keys.fkf-format.org/data_author.asc"/>
</ds:KeyInfo>
```

### 5.5. Verification Rules

1. If a root signature is present, validate it. If invalid, reject the entire file.
2. For each document, if header or data signatures are present, validate them. If invalid, reject that specific block (and optionally mark the document as partially invalid).
3. If both root and block signatures are present, both must pass. The root signature must have valid references to the block signatures (or block contents) to ensure full consistency.

---

## 6. Shorthand Notation (Reaffirmed)

All shorthand rules remain: `<!--FKF-->`, omitted attributes, etc. Signatures are always explicit XML elements, so shorthand does not remove them.

Example with block signatures in shorthand:

```xml
<!--FKF-->
<fkf_root>
  <fkf_document>
    <fkf_data># Signed note</fkf_data>
    <fkf_data_signature>...</fkf_data_signature>
  </fkf_document>
</fkf_root>
```

---

## 7. CDATA Usage (Unchanged)

CDATA is optional but recommended for blocks that may contain XML special characters. The heredoc‑style ending is supported.

---

## 8. Header and Data Types (Unchanged)

Recommended `type` values: `YAML`, `JSON`, `TOML`, `Markdown`, `HTML`, `text`. Custom types must be described in `<fkf_spec>`.

---

## 9. Path and Root Attributes (Unchanged)

`doc_root_dir` + `doc_path` + `doc_name` form the logical full path for each document.

---

## 10. The `fkf_spec` Element (Enhanced)

The `<fkf_spec>` element **should** include:
- Explanation of signature usage (which blocks are signed, which keys are used).
- Custom processing rules for the content.
- Any deviations from the standard.

Example:

```xml
<fkf_spec>
  This file uses block signatures for all data blocks. The root signature references both data blocks and their signatures. Keys are available at the default key server.
  Custom types: 'YAML' for headers, 'Markdown' for data.
</fkf_spec>
```

---

## 11. Versioning and MIME Type (Unchanged)

- Version in top comment: `version="1.0"`.
- MIME type: `text/x.fkf+xml; charset=UTF-8`.
- File extensions: `.fkf.md` (editing) and `.fkf` (distribution).

---

## 12. Best Practices (Updated)

- **Always include a top‑of‑file comment** with version and MIME type.  
- **Provide a detailed `<fkf_spec>`** to document signature policies and custom conventions.  
- **Use CDATA** for any content that may contain `<`, `&`, or `]]>`.  
- **Sign both header and data** for each document to allow independent extraction.  
- **Include a root signature** that references all block signatures and block contents for full container integrity.  
- **Test verification** with a clean AI model after creation.

---

## 13. Full Example (with Both Block and Root Signatures)

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<fkf_root doc_root_dir="./my_knowledge"
          xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  <fkf_spec>
    All documents have signed data blocks. Root signature references both data and data signatures.
  </fkf_spec>

  <fkf_document doc_path="./concepts" doc_name="rest.md">
    <fkf_header type="YAML">title: REST API
tags: [web, api]</fkf_header>
    <!-- No header signature in this example, only data -->
    <fkf_data type="Markdown"><![CDATA[
# REST API
...
]]></fkf_data>
    <fkf_data_signature>
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
        <ds:KeyName>Alice</ds:KeyName>
        <ds:RetrievalMethod URI="https://keys.fkf-format.org/alice.asc"/>
      </ds:KeyInfo>
    </fkf_data_signature>
  </fkf_document>

  <!-- More documents similarly -->

  <ds:Signature>
    <ds:SignedInfo>
      <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
      <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
      <!-- Reference to the specification -->
      <ds:Reference URI="#fkf_spec_id">
        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <ds:DigestValue>...</ds:DigestValue>
      </ds:Reference>
      <!-- Reference to the first document's data block -->
      <ds:Reference URI="#_data_rest">
        <ds:Transforms>
          <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
        </ds:Transforms>
        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <ds:DigestValue>...</ds:DigestValue>
      </ds:Reference>
      <!-- Reference to the first document's data signature -->
      <ds:Reference URI="#_data_sig_rest">
        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <ds:DigestValue>...</ds:DigestValue>
      </ds:Reference>
      <!-- Additional references for other documents -->
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

FKF is forward‑compatible. New attributes or elements may be added, but parsers must ignore unrecognised elements. The core structure (root, documents, header, data, and signature placements) is stable.

---

## 15. Testing Recommendations (Reaffirmed)

After creation, test the file with a clean AI model to ensure:
- Parsing works.
- `<fkf_spec>` is understood.
- Signatures (if present) are validated correctly.
- Extracted blocks retain their signatures and remain verifiable.

---

**End of Specification**  
*This document is the normative reference for FKF version 1.0.*Specification
Specification for a digitally signed knowledge file container, the format of metadata and knowledge is specified in tags, Google OKF by default
