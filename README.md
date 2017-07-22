# hypercore-protocol

Stream that implements the [hypercore](https://github.com/mafintosh/hypercore) protocol

```
npm install hypercore-protocol
```

[![build status](https://travis-ci.org/aschrijver/hypercore-protocol.svg?branch=master)](https://travis-ci.org/aschrijver/hypercore-protocol)

## Note

This fork only testdrives automatic TravisCI documentation generation for Protocol Buffers 
using [protoc-gen-doc](https://github.com/pseudomuto/protoc-gen-doc).

For the _real thing_ go to [hypercore protocol](https://github.com/mafintosh/hypercore-protocol)

## Documentation generation

### HTML from build-in template

Invoking `protoc-gen-doc` the standard way on [schema.proto](schema.proto):
```
  protoc --doc_out=html,index.html:build/html schema.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://aschrijver.github.io/hypercore-protocol/) specification ([source](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/index.html))

**Pro's**:
- Multi-format support. Works for all formats

**Con's**:
- No support for bold, italic, line-breaks, code snippets within a description

### HTML from custom html.mustache template

An adapted `html.mustache` would allow html tags to pass through to the output. Just need to add triple accolades:
```
  {{#file_description}}{{#p}}{{{file_description}}}{{/p}}{{/file_description}}
```

Using the [html.mustache](docgen/html.mustache) custom template on schema [HypercoreSpecV1_html.proto](schemas/HypercoreSpecV1_html.proto):
```
  protoc --doc_out=docgen/html.mustache,inline-html-comments.html:build/html schemas/HypercoreSpecV1_html.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://aschrijver.github.io/hypercore-protocol/html/inline-html-comments.html) specification ([source](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/html/inline-html-comments.html))

**Pro's**:
- Supports bold, italic, line-breaks, code snippets within a description

**Con's**:
- HTML-only format support. Other formats will be polluted with html tags
- Need to escape html characters that are part of the text, using `&lt;`, `&gt;`, etc.
- Not currently a viable option: [Field information not rendered with custom html.mustache templates](https://github.com/pseudomuto/protoc-gen-doc/issues/300)

### Build-in markdown template with inline markdown formatting

Invoking for markdown generation on schema [HypercoreSpecV1_md.proto](schemas/HypercoreSpecV1_md.proto):
```
  protoc --doc_out=markdown,hypercore-protocol.md:build schemas/HypercoreSpecV1_md.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol.md

**Pro's**
- All inlne markdown commands are passed through to the output, allowing rich formatting

**Con's**:
- Markdown-only format support. Other formats will be polluted with markdown commands
- Not currently a viable option: Markdown for Messages incorrectly rendered by build-in template

### Custom markdown template

With a custom markdown template and adding a newline after the anchor before message title and enum title.

Invoking with the custom [markdown.mustache](docgen/markdown.mustache) template:
```
  protoc --doc_out=docgen/markdown.mustache,hypercore-protocol_custom-template.md:build schemas/HypercoreSpecV1_md.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol_custom-template.md

## Conclusion

Markdown generation works best for having rich formatting in Protocol Buffers documentation.