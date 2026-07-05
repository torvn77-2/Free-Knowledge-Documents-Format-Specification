# Free Knowledge Format (FKF) Specification  
**Version 1.0**  
**Author**: The FKF Design Team  

---

## 1. Introduction

The **Free Knowledge Format (FKF)** is a container-oriented, human‑readable, and AI‑friendly file format designed to bundle multiple knowledge documents – each with its own header and body – into a single file.  

FKF retains the atomicity of each knowledge unit (as in OKF) while solving the “many small files” problem. It is built on top of XML to avoid rendering conflicts with Markdown and other textual content, and it explicitly supports both full and shorthand notations to ease manual editing and reduce file size.

---

## 2. Core Principles

- **One file – many documents** – all knowledge units reside in one container.  
- **Path fidelity** – each document keeps its logical path, enabling hierarchical organisation and cross‑references.  
- **Human & machine readable** – clear, symmetric tags and inline comments make the file easy to understand for both people and AI agents.  
- **Flexible content types** – headers and data can use any format (YAML, JSON, plain text, etc.), specified by the `type` attribute.  
- **Optional escaping** – CDATA is supported but not mandatory; simple text nodes can be used when safe.

---

## 3. File Structure

The root of an FKF file is `<fkf_root>`, which contains one or more `<fkf_document>` elements. Each document consists of an optional `<fkf_header>` and a mandatory `<fkf_data>`.

A typical full‑form file looks like:

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="..." -->
<fkf_root doc_root_dir="./local_dir_name">
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
| `<fkf_document>` | A single knowledge unit. Attributes: `doc_path` – relative path (from `doc_root_dir`) and `doc_name` – file name (may be empty, useful for virtual documents). |
| `<fkf_header>` | Metadata of the document. Attribute `type` – format of the header content (e.g., `YAML`, `JSON`, `TOML`). Content is plain text (optionally wrapped in CDATA). |
| `<fkf_data>` | Main content of the knowledge document. Attribute `type` – format of the body (e.g., `Markdown`, `HTML`, `text`). Content can be plain text or CDATA‑escaped. |
| `<!-- FKF ... -->` | Top‑of‑file comments declaring version, encoding, file name, MIME type, and the `fkf_spec` (see below). |

---

## 5. Shorthand Notation

To ease manual editing and reduce size when many small documents are present, the following abbreviations are **allowed**:

- `<!--FKF-->` – replaces the full top‑of‑file comment block (version, name, etc.) when defaults are used.  
- `<fkf_root>` – may omit `doc_root_dir` if it equals `./`.  
- `<fkf_document>` – may omit `doc_path` if the document is at the root (`./`), and `doc_name` if the content is virtual (only data, no file name).  
- `<fkf_header>` – may be omitted entirely if no metadata is required.  
- `type` attributes – if omitted, the parser assumes `"YAML"` for headers and `"Markdown"` for data.

A minimal shorthand file:

```xml
<!--FKF-->
<fkf_root>
  <fkf_document>
    <fkf_data>Just a single virtual document in Markdown</fkf_data>
  </fkf_document>
</fkf_root>
```

---

## 6. CDATA Usage

**CDATA is optional but recommended** for blocks that contain characters that could be misinterpreted as XML markup (e.g., `<`, `&`, `]]>`).

- For `<fkf_data>`, if you are certain that the content does not contain problematic characters, you can use plain text.
- For `<fkf_header>`, similarly, CDATA may be used if the header contains raw XML or special characters.

**Escaping the CDATA end‑marker** – if your content may include the literal sequence `]]>`, you may use the heredoc‑style ending:
```
<![CDATA[ ... </CDATA>
]]> 
```
This is a custom extension that prevents premature termination.

---

## 7. Header and Data Types

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

## 8. Path and Root Attributes

- `doc_root_dir` – base directory for all `doc_path` values. It may be absolute or relative. When resolving internal links (e.g., `[[path/to/doc]]`), the consumer should use the full logical path: `doc_root_dir + doc_path + doc_name`.  
- If `doc_path` is omitted, the document is considered located at the root.  
- The combination of `doc_path` and `doc_name` should be unique within the file.

---

## 9. The `fkf_spec` Comment

The top‑of‑file comment containing `fkf_spec` is **essential** for AI agents and human readers. It describes:

- The version and encoding of the file.  
- Custom conventions used in this particular file (e.g., type extensions, special formatting rules).  
- Any deviations from the standard specification.  

**Recommendation**: Provide a detailed `fkf_spec` to reduce cognitive load and prevent misinterpretation. For example:

```
<!-- fkf_spec="This file uses YAML headers with mandatory 'title' and 'tags', and Markdown data with custom [[...]] links for cross‑references." -->
```

You may also include the `fkf_spec` as a separate `<!-- ... -->` comment (as shown) or as a dedicated element (if you prefer). The comment form is simpler and does not interfere with the XML structure.

---

## 10. Best Practices

- **Always include a top‑of‑file comment** with version and MIME type, even in shorthand (use `<!--FKF-->` if defaults are sufficient).  
- **Set the `type` attribute** explicitly unless you rely on the defaults – this helps both humans and machines.  
- **Use CDATA** for any data that contains `<`, `&`, or `]]>` to avoid parsing errors.  
- **Keep `fkf_spec` up‑to‑date** whenever you introduce custom conventions.  
- **Organise documents logically** using `doc_path` to reflect a folder hierarchy (e.g., `theory/physics`, `entity/people`).

---

## 11. Versioning and MIME Type

- **Version** – declared in the top comment, e.g., `version="1.0"`.  
- **MIME Type** – `text/x.fkf+xml; charset=UTF-8`.  
- **File Extensions** – use `.fkf.md` during editing (so that Markdown editors recognise the file) and `.fkf` for final distribution or automated processing.

---

## 12. Testing Recommendations

After creating or modifying an FKF file, always **test its interpretation** by feeding it to an AI model or agent in a clean environment. This ensures that:

- The `fkf_spec` is correctly understood.  
- All paths and types are resolved as intended.  
- The file is parsed without errors (e.g., no stray `]]>` outside CDATA).  

Testing prevents the accumulation of hidden ambiguities that could confuse downstream consumers.

---

## 13. Example: Full‑Form and Shorthand

**Full form** (with CDATA and explicit types):

```xml
<!-- FKF free knowledge format version="1.0" encoding="UTF-8" file_name="knowledge.fkf.md" -->
<!-- mime-type: text/x.fkf+xml; charset=UTF-8 -->
<!-- fkf_spec="Standard OKF style: YAML header + Markdown body." -->
<fkf_root doc_root_dir="./my_knowledge">
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

**Shorthand form** (no CDATA, minimal attributes):

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

## 14. Compatibility and Future Extensions

FKF is designed to be forward‑compatible. New attributes or elements may be added, but they must be ignored by older parsers. It is recommended to keep the root element `<fkf_root>` and the document wrapper `<fkf_document>` stable.

---

**End of Specification**  
*This document is the normative reference for FKF version 1.0.*
