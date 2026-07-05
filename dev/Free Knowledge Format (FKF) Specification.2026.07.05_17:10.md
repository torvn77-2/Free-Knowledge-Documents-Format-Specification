# Free Knowledge Format (FKF) Specification – Complete Consolidated Version

**Author:** torvn77@gmail.com  
**Status:** Final  
**Version:** 1.0  
**Date:** 2026-07-05

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Core Principles](#2-core-principles)
3. [License and Legal Framework](#3-license-and-legal-framework)
4. [File Structure (Full Form)](#4-file-structure-full-form)
5. [Elements and Attributes](#5-elements-and-attributes)
6. [Digital Signatures – Detailed Specification](#6-digital-signatures--detailed-specification)
7. [Shorthand Notation](#7-shorthand-notation)
8. [CDATA Usage](#8-cdata-usage)
9. [Header and Data Types](#9-header-and-data-types)
10. [Path and Root Attributes](#10-path-and-root-attributes)
11. [The `fkf_spec` Element](#11-the-fkf_spec-element)
12. [Versioning and MIME Type](#12-versioning-and-mime-type)
13. [Best Practices](#13-best-practices)
14. [Full Example](#14-full-example-with-both-block-and-root-signatures)
15. [Compatibility and Future Extensions](#15-compatibility-and-future-extensions)
16. [Testing Recommendations](#16-testing-recommendations)

---

## 1. Introduction

The **Free Knowledge Format (FKF)** is a container‑oriented, human‑readable, and AI‑friendly file format designed to bundle multiple knowledge documents – each with its own metadata header and content body – into a single file.

FKF retains the atomicity of each knowledge unit (as in OKF) while solving the "many small files" problem. It is built on top of XML to avoid rendering conflicts with Markdown and other textual content, and it explicitly supports both full and shorthand notations to ease manual editing and reduce file size.

**Primary use cases:**
- Building a personal knowledge base that can be shared as a single file
- Feeding structured knowledge to AI agents and models
- Exchanging curated knowledge between different systems without file‑system dependencies
- Allowing fine‑grained integrity protection: individual header and data blocks can be signed separately, and a root signature can tie everything together for full verification

---

## 2. Core Principles

- **One file – many documents** – all knowledge units reside in one container
- **Path fidelity** – each document keeps its logical path, enabling hierarchical organisation and cross‑references
- **Human & machine readable** – clear, symmetric tags and inline comments make the file easy to understand for both people and AI agents
- **Flexible content types** – headers and data can use any format (YAML, JSON, plain text, etc.), specified by the `type` attribute
- **Optional escaping** – CDATA is supported but not mandatory; simple text nodes can be used when safe
- **Granular digital signatures** – optional individual header and data blocks can be signed independently; a root signature can cover the entire container, including those block signatures, to provide full integrity and authenticity
- **Licensing transparency** – each document can declare its license, with full metadata including license type, name, and official publication URL

---

## 3. License and Legal Framework

### 3.1. License Declaration Section

If there are documents with licenses in the FKF container, then root MUST contain a `<fkf_licenses>` section that declares all licenses used within the container. This section is placed before any `<fkf_document>` elements and after the optional `<fkf_spec>`.

### 3.2. License Structure

```xml
<fkf_licenses>
    <fkf_license id="no_license"
               type="no_license"
               name="No License"
               url="none">
               No license granted, all rights reserved</fkf_license>
    <fkf_license id="unlicense"
               type="unlicense"
               name="Public Domain License"
               url="https://unlicense.org">
               This is free and unencumbered software, data and other contents released into the public domain."</fkf_license>
  <fkf_license id="optional example license2" 
               type="copyleft" 
               name="GNU General Public License v3.0"
               url="https://www.gnu.org/licenses/gpl-3.0.en.html">
               Example full license text" </fkf_license>
  <fkf_license id="optional example license3" 
               type="permissive" 
               name="MIT License"
               url="https://opensource.org/licenses/MIT">
               Example full license text"</fkf_license>
  <fkf_license id="optional example license4"
               type="proprietary"
               name="Custom Enterprise License v1.0"
               url="https://example.com/license">
               Example full license text</fkf_license>
    <fkf_license id="optional example license5"
               type="public-domain"
               name="CC0 1.0 Universal"
               url="https://creativecommons.org/publicdomain/zero/1.0/">
               Example full license text</fkf_license>
</fkf_licenses>
```

### 3.3. License Attributes

| Attribute | Description | Required |
|-----------|-------------|----------|
| `id` | Unique identifier for referencing this license within the container | Yes |
| `type` | License type classification: `copyleft`, `permissive`, `proprietary`, `copyright`, `public-domain`, or custom value | Yes |
| `name` | Full human-readable license name | Yes |
| `url` | Official URL where the full license text is published | Yes |

### 3.4. Document License Declaration

Section <fkf_document>`maybe include a `<fkf_license>` element that references one of the licenses declared in the root section and if they are specified must duplicate the main data of the license in addition to its text:

```xml
<fkf_document doc_path="./concepts" doc_name="rest.md">
  <fkf_license ref="license1" 
               type="copyleft" 
               name="GNU General Public License v3.0"
               url="https://www.gnu.org/licenses/gpl-3.0.en.html">Optionally specify license text.</fkf_license>
  <fkf_header type="YAML"><![CDATA[ ... </CDATA>\&\#10;]]>;</fkf_header>
  <fkf_data type="Markdown"><![CDATA[ ... </CDATA>\&\#10\;]]>;</fkf_data>
</fkf_document>
```

**Document License Attributes:**
- `ref` – References the `id` of a license declared in `<fkf_licenses>`

### 3.5. License Verification Rules

1. The document specifying the license must necessarily indicate the type of license, the name of the license, a valid link to the place of official publication of the original source of the license by a Nonprofit Foundation organization; if the license was published by a commercial organization, then in addition to the link it is MANDATORY to refer to the valid license identifier declared in the root of the <fkf_licenses> section, the declaration tag of which contains the full text of the license.
2. The root license section must contain all properly executed licenses whose identifier is declared in the document.
3. If a document references a non-existent license ID, the document MUST be rejected
4. License type classification helps with automated compatibility checking

---

## 4. File Structure (Full Form)

The root of an FKF file is `<fkf_root>`, which contains:
- Optional `<fkf_spec>` – A description of specific header and data conventions and other information required by the AI ​​to correctly interpret the metadata and knowledge contained in containers. Be careful when composing this section and thoroughly test its operation!
- Optional `<fkf_licenses>` – all licenses used in the container
- One or more `<fkf_document>` elements – each containing knowledge units and optional signatures
- Optional `<ds:Signature>` – root-level signature

**Full Structure:**

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- Place of publication of the FKF format specification https://github.com/torvn77-2/Free-Knowledge-Documents-Format-Specification -->
<fkf_root doc_root_dir="./local_dir_name" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  
 <fkf_spec>
    A brief description of the features of the FKF document format for AI:
    1. File format extensions .fkf.md for compatibility with Markdown editors during editing, or .fkf when distributing or automatically generating the file.
    2. Optional a heredoc‑like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end.
    3. Digital signatures: root‑level (covers all) and/or per‑document (covers individual entries).
    4. Root signature covers the entire container, referencing block signatures
    5. License declaration for each document with reference to central license registry
  </fkf_spec>

  <fkf_licenses>
    <fkf_license id="gpl3" type="copyleft" name="GNU General Public License v3.0" 
                 url="https://www.gnu.org/licenses/gpl-3.0.en.html"> GPL licanse txt </fkf_license>
  </fkf_licenses>

  <fkf_document doc_path="./relative_doc_path" doc_name="file_name.md">
  <fkf_license ref="gpl3" 
               type="copyleft" 
               name="GNU General Public License v3.0"
               url="https://www.gnu.org/licenses/gpl-3.0.en.html"/>
    <fkf_header type="OKF YAML"><![CDATA[ ... </CDATA>&#10;]]&gt;</fkf_header>
    <fkf_header_signature>...</fkf_header_signature>
    <fkf_data type="OKF md"><![CDATA[ ... </CDATA>&#10;]]&gt;</fkf_data>
    <fkf_data_signature>...</fkf_data_signature>
  </fkf_document>

  <!-- more fkf_document elements -->

  <ds:Signature>
    <!-- Root signature covering the whole fkf_root -->
  </ds:Signature>
</fkf_root>
```

---

## 5. Elements and Attributes

| Element / Attribute | Description |
|---------------------|-------------|
| **`<fkf_root>`** | Container for all documents. Attributes: `doc_root_dir` – base directory for all document paths (optional, defaults to `./`). May declare XML namespaces for signatures. |
| **`<fkf_spec>`** | (Optional) Human‑ and machine‑readable description of file‑specific conventions, custom types, and signature policies. Placed inside `<fkf_root>`. Carefully test the contents of the section and any changes you make to ensure your AI Agent or model understands them. It's best to do this in a clean file that doesn't contain any data. |
| **`<fkf_licenses>`** | (Optional) Container for all license declarations used in the file. |
| **`<fkf_license>`** | (Optional) Individual license declaration. Attributes: `id` – unique identifier, `type` – license classification, `name` – full license name, `url` – official publication URL. |
| **`<fkf_document>`** | A single knowledge unit. Attributes: `doc_path` – relative path (from `doc_root_dir`), `doc_name` – file name (may be empty). |
| **`<fkf_license>`** | (Optional) Document license reference. Attribute: `ref` – references a license ID from the root licenses section. |
| **`<fkf_header>`** | (Optional for personal use and drafts) Metadata of the document. Attribute: `type` – format of the header content. |
| **`<fkf_header_signature>`** | (Optional) XML Digital Signature over the `<fkf_header>` element. Uses standard XMLDSig with an enveloped transform. |
| **`<fkf_data>`** | Main content. Attribute: `type` – format of the body. |
| **`<fkf_data_signature>`** | (Optional) XML Digital Signature over the `<fkf_data>` element. |
| **`<ds:Signature>`** | (Optional) XML Digital Signature at the root level, covering the entire container (excluding itself). Can reference any element via `ds:Reference`. |

---

## 6. Digital Signatures – Detailed Specification

### 6.1. Block‑Level Signatures (`<fkf_header_signature>` and `<fkf_data_signature>`)

- **Purpose**: Ensure the integrity and authenticity of an individual header or data block, independently of the rest of the document
- **Scope**: The signature is computed over the entire `<fkf_header>` (or `<fkf_data>`) element, including its `type` attribute and all its content (text or CDATA)
- **Placement**: Immediately after the corresponding `<fkf_header>` or `<fkf_data>` element, inside the `<fkf_document>`
- **Key Management**: Each block signature must contain a `<ds:KeyInfo>` that identifies the signer (key name, certificate, or key server reference)
- **Verification**: The block signature can be verified independently, even when the document is extracted or moved to another container. If a block signature is present and invalid, that block **must** be rejected

### 6.2. Root Signature (`<ds:Signature>` as last child of `<fkf_root>`)

- **Purpose**: Provide full container‑level integrity and authenticity. It ties together all documents, the specification, and optionally the block signatures
- **Scope**: The root signature covers the entire `<fkf_root>` element (excluding the root `<ds:Signature>` itself). However, it uses `ds:Reference` elements to point to specific parts of the document
- **Flexibility**: The root signature may reference:
  - The `<fkf_spec>` element
  - The `<fkf_licenses>` section
  - Each `<fkf_document>` element, or individual `<fkf_header>` and `<fkf_data>` elements
  - Block signatures (`<fkf_header_signature>` and `<fkf_data_signature>`)
- **Recommended Practice**: Reference **both** the block content and the block signatures to detect any change to content or signatures
- **Verification**: The root signature must be validated first. If it is present, all referenced items must match their digests

### 6.3. Relationship Between Block Signatures and Root Signature

- **Independence**: Block signatures can exist without a root signature, allowing document extraction with its own protection
- **Root with block references**: The root signature can reference block signatures to ensure they haven't been replaced or removed
- **Root with content references**: The root signature can directly reference content, ignoring block signatures
- **Recommended practice**: Include both header and data signatures for each document, and have the root signature reference both the block signatures and the block content

### 6.4. Key Management and Default Key Server

- **Default key server**: `https://keys.fkf-format.org` – a public service for publishing FKF‑related keys
- **Inline keys**: Keys can be embedded inside `<ds:KeyInfo>` using `<ds:X509Data>` or `<ds:KeyValue>`
- **Custom key servers**: Any URI can be used in `<ds:RetrievalMethod>`

Example `<ds:KeyInfo>` for a block signature:

```xml
<ds:KeyInfo>
  <ds:KeyName>Data Author</ds:KeyName>
  <ds:RetrievalMethod URI="https://keys.example_your_domain.org/data_author.asc"/>
</ds:KeyInfo>
```

### 6.5. Verification Rules

1. If a root signature is present, validate it. If invalid, reject the entire file
2. For each document, if header or data signatures are present, validate them. If invalid, reject that specific block (and optionally mark the document as partially invalid)
3. If both root and block signatures are present, both must pass. The root signature must have valid references to the block signatures (or block contents) to ensure full consistency

---

## 7. Shorthand Notation

All shorthand rules remain in place:
- `<!--FKF-->` at the beginning of the file indicates shorthand mode
- Omitted attributes use default values
- Signatures are always explicit XML elements, so shorthand does not remove them

Example with block signatures in shorthand:

```xml
<!--FKF-->
<fkf_root>
  <fkf_licenses>
    <fkf_license id="mit" type="permissive" name="MIT License" url="https://opensource.org/licenses/MIT"/>
  </fkf_licenses>
  <fkf_document>
    <fkf_license ref="mit"/>
    <fkf_data># Signed note</fkf_data>
    <fkf_data_signature>...</fkf_data_signature>
  </fkf_document>
</fkf_root>
```
For personal use 

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

## 8. CDATA Usage

CDATA is optional but recommended for blocks that may contain XML special characters. The heredoc‑style ending is supported:

```xml
<fkf_data type="Markdown"><![CDATA[
# Heading
This contains <tags> and &amp; symbols that would otherwise need escaping.
</CDATA>
]]></fkf_data>
```

---

## 9. Header and Data Types

**Recommended `type` values**: `YAML`, `JSON`, `TOML`, `Markdown`, `HTML`, `text`, `XML`, `CSV`

Custom types must be described in `<fkf_spec>` with their parsing and validation rules.

---

## 10. Path and Root Attributes

`doc_root_dir` + `doc_path` + `doc_name` form the logical full path for each document.

| Attribute | Description | Default |
|-----------|-------------|---------|
| `doc_root_dir` | Base directory for all document paths | `./` |
| `doc_path` | Relative path from `doc_root_dir` | Required per document |
| `doc_name` | File name of the document | Required per document |

---

## 11. The `fkf_spec` Element

The `<fkf_spec>` element **should** include:

- Explanation of signature usage (which blocks are signed, which keys are used)
- Custom processing rules for the content
- Any deviations from the standard
- License compatibility information

Example:

```xml
<fkf_spec>
    A brief description of the features of the FKF document format for AI:
    1. File format extensions .fkf.md for compatibility with Markdown editors during editing, or .fkf when distributing or automatically generating the file.
    2. Optional a heredoc‑like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end.
    3. Digital signatures: root‑level (covers all) and/or per‑document (covers individual entries).
    4. Root signature covers the entire container, referencing block signatures
    5. License declaration for each document with reference to central license registry
  This file uses block signatures for all data blocks. The root signature references both data blocks and their signatures. Keys are available at the default key server.
  Custom types: 'YAML' for headers, 'Markdown' for data.
  All documents are licensed under GPLv3 or MIT as indicated by the license reference.
</fkf_spec>
```

---

## 12. Versioning and MIME Type

- **Version**: In top comment: `version="1.0"`
- **MIME type**: `text/x.fkf+xml; charset=UTF-8`
- **File extensions**: `.fkf.md` (editing) and `.fkf` (distribution)

---

## 13. Best Practices

1. **Always include a top‑of‑file comment** with version and MIME type
2. **Provide a detailed `<fkf_spec>`** to document signature policies and custom conventions
3. **Include the `<fkf_licenses>` section** with all licenses used in the container
4. **Reference a license for every document** using the `<fkf_license>` element
5. **Use CDATA** for any content that may contain `<`, `&`, or `]]>`
6. **Sign both header and data** for each document to allow independent extraction
7. **Include a root signature** that references all block signatures and block contents for full container integrity
8. **Test verification** with a clean AI model after creation
9. **Document custom types** in the `<fkf_spec>` section

---

## 14. Full Example (with Both Block and Root Signatures)

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<fkf_root doc_root_dir="./my_knowledge" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  
  <fkf_spec>
    All documents have signed data blocks. Root signature references both data and data signatures.
    License: Documents are either GPLv3 or MIT licensed as indicated.
  </fkf_spec>

  <fkf_licenses>
    <fkf_license id="gpl3" type="copyleft" name="GNU General Public License v3.0" 
                 url="https://www.gnu.org/licenses/gpl-3.0.en.html"> GPL licanse txt </fkf_license>
    <fkf_license id="mit" type="permissive" name="MIT License" 
                 url="https://opensource.org/licenses/MIT"> MIT licanse txt </fkf_license>
  </fkf_licenses>

  <fkf_document doc_path="./concepts" doc_name="rest.md">
    <fkf_license ref="gpl3"/>
    <fkf_header type="YAML">title: REST API
tags: [web, api]
author: Alice</fkf_header>
    <fkf_header_signature>
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
        <ds:RetrievalMethod URI="https://keys.example.org/alice.asc"/>
      </ds:KeyInfo>
    </fkf_header_signature>

    <fkf_data type="Markdown"><![CDATA[
# REST API

REST (Representational State Transfer) is an architectural style for distributed systems.

## Key Principles
1. Client-Server separation
2. Statelessness
3. Cacheability
4. Uniform interface
5. Layered system
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
        <ds:RetrievalMethod URI="https://keys.example.org/alice.asc"/>
      </ds:KeyInfo>
    </fkf_data_signature>
  </fkf_document>

  <fkf_document doc_path="./guides" doc_name="api-design.md">
    <fkf_license ref="mit"/>
    <fkf_header type="YAML">title: API Design Guidelines
tags: [design, best-practices]
author: Bob</fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# API Design Guidelines

## Naming Conventions
- Use nouns for resources
- Use HTTP methods for operations
- Version your API from day one
    ]]></fkf_data>
  </fkf_document>

  <ds:Signature>
    <ds:SignedInfo>
      <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
      <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
      
      <!-- Reference to the specification -->
      <ds:Reference URI="#fkf_spec">
        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <ds:DigestValue>...</ds:DigestValue>
      </ds:Reference>
      
      <!-- Reference to the licenses -->
      <ds:Reference URI="#fkf_licenses">
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

## 15. Compatibility and Future Extensions

FKF is forward‑compatible. New attributes or elements may be added, but parsers must ignore unrecognised elements. The core structure (root, documents, header, data, licenses, and signature placements) is stable.

**Extension Guidelines:**
- Custom elements should use a namespace prefix
- Future versions must maintain backward compatibility
- Deprecated elements should be clearly marked

---

## 16. Testing Recommendations

After creation, test the file with a clean AI model to ensure:

- Parsing works correctly
- `<fkf_spec>` is understood
- License declarations are parsed and validated
- Signatures (if present) are validated correctly
- Extracted blocks retain their signatures and remain verifiable
- Documents with invalid license references are rejected

---

**End of Specification**

*This document is the normative reference for FKF version 1.0. All implementations must conform to this specification.*

---

## Summary of Key Changes in This Consolidated Version

| Feature | Description |
|---------|-------------|
| **License Section** | New `<fkf_licenses>` section at root level containing all license declarations |
| **License Types** | Support for `copyleft`, `permissive`, `proprietary`, `copyright`, `public-domain` |
| **Document License** | Each document includes `<fkf_license ref="id"/>` referencing root license |
| **License URL** | Each license includes official publication URL |
| **Full Example** | Updated with complete license declarations and references |
| **Best Practices** | Updated to include license management guidance |
