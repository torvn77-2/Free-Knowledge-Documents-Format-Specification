**Summary: Free Knowledge Format (FKF) – A Container for Structured Knowledge**

---

### 1. Background and Motivation

The conversation began with a discussion of the **Open Knowledge Format (OKF)** – an open, vendor-agnostic standard by Google that represents knowledge as a collection of Markdown files with YAML frontmatter, each file describing a single concept (atomicity). OKF is designed for both human readability and AI-agent consumption, supporting navigation via `[[links]]` and hierarchical directories.

However, the user identified a practical limitation: they wanted to deliver **multiple OKF documents (theories, entities, concepts) in a single file** for easy sharing, attachment, and ingestion by an AI. The standard OKF expects a directory tree with separate files, which is inconvenient for single-file distribution.

To solve this, the user proposed a **container format** that wraps any number of knowledge documents – including OKF-structured ones – into one file, while preserving the internal structure (hierarchy, metadata, content). This gave birth to the **Free Knowledge Format (FKF)**.

---

### 2. Design Principles

- **Single-file delivery** – all knowledge in one portable file.
- **Human‑readable and AI‑parseable** – the file should be easy to open and understand for both humans and agents.
- **No collisions** – using XML‑style tags avoids conflicts with Markdown syntax (e.g., triple backticks, YAML separators).
- **Flexibility** – each contained document can have its own internal format (OKF YAML+Markdown, JSON, CSV, plain text, etc.).
- **Self‑descriptive** – the file declares its own version, filename, and specification so that an AI knows how to interpret it.
- **Hierarchical preservation** – each document retains its path in the original knowledge tree, allowing `[[links]]` to resolve correctly.

---

### 3. FKF Specification (Final Version)

```xml
<!-- free knowledge format version="1.0"  file_name="knowledge.fkf.md"  -->
<!-- fkf_spec="A brief description of the features of the fkf document format for AI, a heredoc-like format for ending a CDATA block <![CDATA[ ... </CDATA>&#10;]]&gt; compatible with the standard block ending in XML documents ]]> end"  -->
<fkf_root doc_root_dir="./local_dir_name" >
  <fkf_documents doc_path="./relative_doc_path" doc_name="file_name.md">
    <fkf_header type="OKF YAML" > head data like OKF YAML header</fkf_header>
    <fkf_data type="OKF md"><![CDATA[ бла бла в md формате
</CDATA>
]]> </fkf_data>
  </fkf_document>
</fkf_root>
```

#### Key Elements:

- **Prologue comments** – declare version, file name, and a human/AI-readable specification (`fkf_spec`). The spec uses XML entities for special characters (`&#10;` for newline, `&gt;` for `>`), since it resides in a plain text attribute.
- **`<fkf_root>`** – the root container. Attribute `doc_root_dir` specifies the base directory path (e.g., `./local_dir_name`), establishing the root of the knowledge tree.
- **`<fkf_documents>`** – each document within the container. Attributes:
  - `doc_path` – relative path from `doc_root_dir` (e.g., `./relative_doc_path`).
  - `doc_name` – the file name (e.g., `file_name.md`). Together they form the full virtual path.
- **`<fkf_header>`** – contains metadata in a structured format (like OKF YAML frontmatter). The `type` attribute indicates the format (e.g., `OKF YAML`, `JSON`, `TOML`).
- **`<fkf_data>`** – contains the actual content (e.g., Markdown text). The `type` attribute specifies the format (e.g., `OKF md`, `plain text`, `CSV`). Content is wrapped in a **CDATA** block to allow any characters without escaping, except for the CDATA closing sequence `]]>` – which is handled by a heredoc‑like ending (`</CDATA>&#10;]]&gt;`) as described in the spec comment.

---

### 4. Rationale Behind the Design Choices

- **Full tag names (`fkf_document` vs. `doc`)** – the user deliberately chose longer, symmetrical tags to enhance human readability. Full tags are visually distinct and reduce ambiguity, making it easier for both eyes and AI parsers to recognise structural boundaries.
- **`type` per header and data** – by separating metadata format from content format, the container gains flexibility. A document could have a YAML header and Markdown body, or JSON header and CSV data, etc.
- **`doc_path` and `doc_name`** – these preserve the original directory structure, enabling cross‑references (e.g., `[[concepts/rest-api]]`) to work seamlessly even within a single file.
- **CDATA usage** – protects content from XML parsing errors (e.g., `<`, `&`, line breaks) and allows the author to write natural Markdown without escaping.
- **Self‑contained spec comment** – the `fkf_spec` comment instructs the AI on how to properly close a CDATA block, using a unique ending pattern (`</CDATA>&#10;]]&gt;`) to avoid collisions with standard `]]>`.

---

### 5. Advantages Over Plain OKF (for this use case)

| Feature | OKF (standard) | FKF |
|---------|---------------|-----|
| Delivery | Directory of files | Single file |
| Atomicity | One concept per file | Multiple concepts wrapped as separate documents |
| Hierarchy | File system folders | Virtual paths via attributes |
| Format flexibility | Only YAML + Markdown | Any format per header/data |
| Parsing overhead | Low (file system) | Slightly higher (XML parsing) but acceptable |
| Human readability | Very good | Good (tagged structure) |

---

### 6. Future Extensibility

The FKF design is intentionally open:
- Add more metadata attributes to `<fkf_documents>` (e.g., `tags`, `id`).
- Support for non‑XML data using different CDATA termination schemes.
- Versioning of the container spec can be declared in the prologue, allowing backward‑compatible changes.

---

### 7. Conclusion

The **Free Knowledge Format** is a pragmatic, human‑centric container that marries the richness of OKF with the simplicity of a single distributable file. It preserves all essential metadata, hierarchical linking, and content integrity, while providing clear instructions for AI agents. The format is ready for use and can serve as a robust foundation for personal knowledge bases, collaborative exchanges, or machine‑learning pipelines.

---

*Prepared as a concise technical summary of the design conversation.*
