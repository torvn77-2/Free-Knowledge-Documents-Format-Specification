# Free Knowledge Format (FKF) Specification  
**Version 1.0**  
*Author: The FKF Design Team*  

---

## 1. Purpose and Motivation

The **Free Knowledge Format (FKF)** is a container format designed to bundle multiple structured knowledge documents—such as **Open Knowledge Format (OKF)** files, JSON schemas, CSV tables, or any other textual data—into a **single, human‑readable file**.

It addresses two main challenges:

- **Portability** – A single file is easier to share, version, and attach to AI prompts than a directory tree.
- **Markdown compatibility** – Using XML‑style tags avoids the escaping hell that occurs when triple backticks or YAML frontmatter appear inside Markdown.

FKF does **not** replace OKF; it **wraps** it (and other formats) in a transparent container, preserving the original hierarchical structure through path attributes.

---

## 2. File Naming and MIME Type (Recommended)

- Recommended file extension: **`.fkf.md`** (so Markdown editors can still render it).
- Alternative: **`.fkf`**.
- No official MIME type yet; treat as `text/xml` or `text/plain`.

---

## 3. Overall Structure

An FKF file is a well‑formed XML document with the following mandatory components:

1. **Header comments** – Contain metadata and a processing hint for AI agents.
2. **Single root element** – `<fkf_root>` with a mandatory `doc_root_dir` attribute.
3. **One or more document containers** – `<fkf_document>` elements, each representing a separate knowledge unit.
4. **Inside each document**:
   - `<fkf_header>` – for machine‑readable metadata (YAML, JSON, etc.).
   - `<fkf_data>` – for the main content (Markdown, CSV, etc.), typically wrapped in a CDATA section.

---

## 4. The Header Comments

Every FKF file **must** begin with two XML comments:

```xml
<!-- free knowledge format version="1.0"  file_name="knowledge.fkf.md"  -->
<!-- fkf_spec="A brief description of the features of the fkf document format for AI, a heredoc-like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end"  -->
```

- **`version`** – the FKF version used.
- **`file_name`** – the suggested name of this file (useful when the file is extracted or renamed).
- **`fkf_spec`** – a human‑ and AI‑readable string that explains the special CDATA termination convention (see Section 6). The encoded `&#10;` and `]]&gt;` make the comment safe.

These comments serve as a **self‑describing header**; an AI agent can read them first to know how to parse the rest.

---

## 5. The Root Element

```xml
<fkf_root doc_root_dir="./local_dir_name" >
  ...
</fkf_root>
```

- **`doc_root_dir`** (required) – the base directory path (relative or absolute) against which all `doc_path` attributes are resolved. This allows the container to emulate a folder hierarchy.

---

## 6. Document Containers

Each knowledge unit is placed inside a `<fkf_document>` element:

```xml
<fkf_documents doc_path="./relative_doc_path" doc_name="file_name.md">
  <fkf_header type="OKF YAML" > ... </fkf_header>
  <fkf_data type="OKF md"> <![CDATA[ ... ]]></fkf_data>
</fkf_documents>
```

> **Note:** The plural tag name `fkf_documents` is intentional – it emphasises that the element may contain multiple sub‑items, though each is a separate document.

### Attributes of `<fkf_documents>`:

- **`doc_path`** (required) – the relative path (including directories) where this document would reside in a real file system. Example: `./theories/physics/relativity.md`. This enables cross‑references using the `[[path]]` convention inside the data.
- **`doc_name`** (optional) – the base file name (e.g., `relativity.md`). If omitted, the last part of `doc_path` is used.

---

## 7. Header and Data Elements

### `<fkf_header>`

- Contains **metadata** about the document (e.g., title, tags, version, author, type).
- **`type`** attribute (required) – specifies the format of the header content. Common values: `"OKF YAML"`, `"JSON"`, `"TOML"`, `"YAML"`.
- The content is plain text (not CDATA) because metadata should not contain ambiguous symbols; if needed, they can be escaped as XML entities.

### `<fkf_data>`

- Contains the **main body** of the knowledge.
- **`type`** attribute (required) – indicates the format of the data, e.g., `"OKF md"` (Markdown with OKF conventions), `"CSV"`, `"Plain text"`.
- **Must** be wrapped in a CDATA section to preserve line breaks, indentation, and special characters without escaping.

---

## 8. Special CDATA Termination Convention

Standard XML CDATA ends with `]]>`. However, if the Markdown itself contains `]]>` (rare but possible), it would prematurely close the CDATA block. To avoid this, FKF uses a **two‑step termination**:

```xml
<![CDATA[ ... 
</CDATA>
]]>
```

Interpretation:
- The CDATA block **ends** at the first `]]>` it encounters.
- By placing `</CDATA>` immediately before `]]>`, we create a distinctive marker that is unlikely to appear in the actual content.
- If the user truly needs to write `</CDATA>` or `]]>` inside the text, they can split it across multiple CDATA sections, but this is not recommended.

This convention is explained in the `fkf_spec` comment so that parsers know to treat `</CDATA>\n]]>` as the intended closing sequence.

---

## 9. Path Resolution and Cross‑References

- The combination of `doc_root_dir` (from root) and `doc_path` (from document) gives the full logical path.
- Inside the `fkf_data` content, references to other documents **should** use the `[[path]]` syntax (common in OKF). The path **must** be relative to `doc_root_dir` and match the `doc_path` of another document.
- When an AI agent reads the FKF file, it can build an internal map of all documents and resolve those links.

---

## 10. Full Example

```xml
<!-- free knowledge format version="1.0"  file_name="knowledge.fkf.md"  -->
<!-- fkf_spec="A brief description of the features of the fkf document format for AI, a heredoc-like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end"  -->
<fkf_root doc_root_dir="./" >
  <fkf_documents doc_path="./concepts/rest-api.md" doc_name="rest-api.md">
    <fkf_header type="OKF YAML" >
---
type: concept
title: REST API
tags: [api, http]
---
    </fkf_header>
    <fkf_data type="OKF md">
<![CDATA[
# REST API

REST is an architectural style for distributed systems.

## Example request
```http
GET /users/123 HTTP/1.1
Host: api.example.com
```

## See also
- [[http-methods]]
- [[json-api]]
</CDATA>
]]>
    </fkf_data>
  </fkf_document>

  <fkf_documents doc_path="./concepts/http-methods.md" doc_name="http-methods.md">
    <fkf_header type="OKF YAML" >
---
type: concept
title: HTTP Methods
---
    </fkf_header>
    <fkf_data type="OKF md">
<![CDATA[
# HTTP Methods

- GET – retrieve resource
- POST – create resource
- PUT – update resource
- DELETE – remove resource
</CDATA>
]]>
    </fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 11. Extensibility and Future Versions

- New attributes or child elements may be added in later versions without breaking basic parsers (they should ignore unknown attributes).
- The `type` attribute allows any textual format to be embedded, so the container can evolve alongside new serialisation standards.
- The `fkf_spec` comment can be updated to reflect new conventions.

---

## 12. Parsing Guidelines for AI Agents

1. Extract the two initial comments to understand the file’s version and special rules.
2. Parse the XML structure (use a robust XML parser that respects CDATA).
3. For each `<fkf_document>`, read `doc_path` and `doc_name` to build a virtual file tree.
4. Decode the `fkf_header` and `fkf_data` according to their `type` attributes.
5. Resolve cross‑references (`[[...]]`) using the resolved paths.

---

**This specification is released under the terms of the MIT License.**  
*Feel free to implement, extend, and share.*

---
