# Free Knowledge Format (FKF) – Container for AI Agents and LLM Chats

**Discussion draft specification:**  
[https://github.com/torvn77-2/Free-Knowledge-Documents-Format-Specification/blob/main/dev/Free%20Knowledge%20Format%20(FKF)%20Specification.2026.07.05_17%3A10.md](https://github.com/torvn77-2/Free-Knowledge-Documents-Format-Specification/blob/main/dev/Free%20Knowledge%20Format%20(FKF)%20Specification.2026.07.05_17%3A10.md)

---

## Overview

**FKF** is a universal container format designed for storing and transferring structured knowledge between artificial intelligences, agents, and large language models. It is XML‑based and allows packaging many independent knowledge documents into a single file while preserving their integrity, attribution, and legal protection.

---

## Core Structure

Every FKF file has a root `<fkf_root>` element, inside which one or more `<fkf_document>` sections are placed. Each document contains two main parts:

- **`<fkf_header>`** – metadata about the document (author, tags, description, license, etc.).
- **`<fkf_data>`** – the actual knowledge content (text, markup, code, diagrams, etc.).

For each section you can specify a **data type** using the `type` attribute. This allows any textual format: `YAML`, `JSON`, `TOML`, `Markdown`, `HTML`, `plain text`, or even custom types.

---

## Compatibility with OKF

If you use `YAML` for the header and `Markdown` for the data, each `<fkf_document>` exactly matches the **Open Knowledge Format (OKF)** specification from Google. Thus, FKF acts as a **container for multiple OKF documents**, bundling them into one file without losing structure or semantics.
>OKF (Open Knowledge Format) is a format developed by Google and distributed under the Apache 2.0 license.  
>[https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf)

---

## Additional Capabilities

Beyond simple packaging, FKF provides important features for secure and legally clear knowledge exchange:

### 1. Licensing
You can specify a license for each document individually (for example, in the header), solving copyright and distribution terms when sharing the knowledge base with third parties.

### 2. Digital Signatures (Two‑Level)
- **Block‑level signatures** – each `<fkf_header>` and `<fkf_data>` can be signed separately. This allows extracting individual blocks together with their signatures, preserving authorship and integrity even outside the container.
- **Root signature** – the whole `<fkf_root>` can be signed with a single signature that references both the block content and the block signatures. This gives full guarantee of immutability for the entire knowledge base.

This architecture solves two key problems:
- **Protection against corruption or tampering** during distribution.
- **Legal clarity** – each document can carry its own license and signature, simplifying compliance with copyright and usage terms.

---

## Summary

FKF is not just another format, but a well‑thought‑out container that combines:
- universality (any data types),
- OKF compatibility,
- built‑in licensing and cryptographic protection.

It is ideal for creating portable, secure, and legally sound knowledge bases for AI agents and large language models.

---

## Usage Example

Imagine you want to package two short notes into one FKF file. The first describes the REST principle, the second is a cheat sheet for HTTP methods. Both notes have YAML headers and Markdown bodies.

Create a file named `my_knowledge.fkf.md` with the following content:

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="my_knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->

<fkf_root doc_root_dir="./knowledge_base">

  <!-- Format description for AI (recommended) -->
  <fkf_spec>
    This file uses OKF-style documents: YAML headers + Markdown bodies.
    All documents are placed under the root directory ./knowledge_base.
    Links between documents use the [[path/to/doc]] syntax.
  </fkf_spec>

  <!-- First document: REST API concept -->
  <fkf_document doc_path="./concepts" doc_name="rest_api.md">
    <fkf_header type="YAML">
      title: REST API
      description: Architectural style for web services
      tags: [api, http, web]
    </fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# REST API

**REST** (Representational State Transfer) is a set of principles for building scalable web services.

## Key principles
- Client‑server architecture
- Statelessness
- Cacheable responses
- Uniform interface

## Related concepts
- [[http_methods]] — HTTP methods
]]></fkf_data>
  </fkf_document>

  <!-- Second document: HTTP methods -->
  <fkf_document doc_path="./concepts" doc_name="http_methods.md">
    <fkf_header type="YAML">
      title: HTTP Methods
      description: Basic HTTP protocol methods
      tags: [http, api]
    </fkf_header>
    <fkf_data type="Markdown"><![CDATA[
# HTTP Methods

## GET
Used to retrieve data. Safe and idempotent.

## POST
Used to create new resources. Not idempotent.

## PUT
Used to fully update a resource. Idempotent.

## DELETE
Used to delete a resource. Idempotent.
]]></fkf_data>
  </fkf_document>

</fkf_root>
```

### Important points:

1. **File header** – contains version, encoding, and MIME type as required by the specification.
2. **`<fkf_spec>`** – explains to an AI agent which formats are used and how to interpret links.
3. **`<fkf_root doc_root_dir="...">`** – sets the root directory for all documents.
4. **Each `<fkf_document>`** has:
   - `doc_path` and `doc_name` – forming the logical file path.
   - `<fkf_header type="YAML">` – metadata in OKF style.
   - `<fkf_data type="Markdown">` – the main text, with the ability to link to other documents using `[[...]]`.
5. **CDATA** – used to safely include Markdown markup that may contain `<`, `&`, and other special characters.

This file can be opened in any text editor, fed to an AI agent, or used as a foundation for a larger knowledge base. If needed, you can add digital signatures and licenses following the full specification.

---

> **Note:** The complete FKF specification (version 1.0) is available in the file `SPECIFICATION.md` (or via the link at the top of this document).
