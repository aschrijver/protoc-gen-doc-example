# hypercore-protocol

Stream that implements the [hypercore](https://github.com/mafintosh/hypercore) protocol

[![build status](https://travis-ci.org/aschrijver/hypercore-protocol.svg?branch=master)](https://travis-ci.org/aschrijver/hypercore-protocol)

## Note

> This fork only serves to testdrive automatic TravisCI documentation generation for Protocol Buffers 
using [protoc-gen-doc](https://github.com/pseudomuto/protoc-gen-doc), and to provide examples/instruction for users, 
plus to demonstrate some existing bugs in the generator.

**The most complete documentation can be found in the markdown examples**

For the **_real thing_** go to original [hypercore-protocol](https://github.com/mafintosh/hypercore-protocol) github repo.

## Documentation generation

Versions:

- Ubuntu `16.04`
- libprotoc `2.6.1`
- protoc-gen-doc `0.9` (presumably, but [`protoc-gen-doc --version`](https://github.com/pseudomuto/protoc-gen-doc/issues/299) is not supported to confirm that)


### HTML from build-in template

Invoking `protoc-gen-doc` the standard way on [schema.proto](schema.proto):
```sh
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
```sh
  protoc --doc_out=docgen/datproject_htm.mustache,inline-html-comments.html:build/html \
      schemas/HypercoreSpecV1_html.proto
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
```sh
  protoc --doc_out=markdown,hypercore-protocol.md:build schemas/HypercoreSpecV1_md.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol.md)

**Pro's**
- All inlne markdown commands are passed through to the output, allowing rich formatting

**Con's**:
- Markdown-only format support. Other formats will be polluted with markdown commands
- Not currently a viable option: Markdown for Messages incorrectly rendered by build-in template

### Custom markdown template

With a custom markdown template and adding a newline after the anchor before message title and enum title.

Invoking with the custom [markdown.mustache](docgen/markdown.mustache) template:
```sh
  protoc --doc_out=docgen/datproject_md.mustache,hypercore-protocol_custom-template.md:build \
      schemas/HypercoreSpecV1_md.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol_custom-template.md)

**Pro's**
- Can fully customize the markdown output
- All inlne markdown commands are passed through to the output, allowing rich formatting

**Con's**:
- Markdown-only format support. Other formats will be polluted with markdown commands
- Not currently a viable option: Markdown for Fields not rendered by custom template

## TravisCI YAML

All the above generation options use the following `.travis.yml`:

```yaml
before_install:
  # (PS A bit lazy here, this should really be in a separate script invoked from the yaml)
  #
  # Install protobuf-compiler and protoc-doc-gen using apt. TravisCI uses Ubuntu 14.04 by default.
  - sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/estan:/protoc-gen-doc/xUbuntu_14.04/ /' > /etc/apt/sources.list.d/protoc-gen-doc.list"
  - wget -nv http://download.opensuse.org/repositories/home:estan:protoc-gen-doc/xUbuntu_14.04/Release.key -O Release.key
  - sudo apt-key add - < Release.key
  - sudo apt-get update
  - sudo apt-get install protobuf-compiler
  # Note: Currently invalidly signed, key expired (https://github.com/pseudomuto/protoc-gen-doc/issues/295)
  - sudo apt-get --allow-unauthenticated install protoc-gen-doc

  # Create directory structure, copy files
  - mkdir build && mkdir build/html
  - cp docgen/stylesheet.css build/html

  # Create all flavours of output formats to test (see README)
  - protoc --doc_out=html,index.html:build schema.proto
  - protoc --doc_out=docgen/html.mustache,inline-html-comments.html:build/html schemas/HypercoreSpecV1_html.proto
  - protoc --doc_out=markdown,hypercore-protocol.md:build schemas/HypercoreSpecV1_md.proto
  - protoc --doc_out=docgen/markdown.mustache,hypercore-protocol_custom-template.md:build schemas/HypercoreSpecV1_md.proto
language: node_js
node_js:
  - "6"
  - "4"
  - "0.12"
  - "0.10"
deploy:
  provider: pages
  skip_cleanup: true          # Do not forget, or the whole gh-pages branch is cleaned
  name: datproject            # Name of the committer in gh-pages branch
  local_dir: build            # Take files from the 'build' output directory
  github_token: $GITHUB_TOKEN # Set in travis-ci.org dashboard (see README)
  on:
    all_branches: true        # Could be set to 'branch: master' in production
    node: '6'                 # Needs only to run once. In this case when Node 6 tests have passed
```

## Conclusion

Markdown generation works best for having rich formatting in Protocol Buffers documentation, 
but needs fixing before it is usable.