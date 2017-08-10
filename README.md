# hypercore-protocol

Examples use the [hypercore](https://github.com/mafintosh/hypercore) protocol buffers specification.

[![build status](https://travis-ci.org/aschrijver/protoc-gen-doc-example.svg?branch=master)](https://travis-ci.org/aschrijver/protoc-gen-doc-example)

## Note

> This fork only serves to testdrive automatic TravisCI documentation generation for Protocol Buffers 
using [protoc-gen-doc](https://github.com/pseudomuto/protoc-gen-doc), and to provide examples/instruction for users, 
plus to demonstrate some existing bugs in the generator.

**The most complete documentation can be found in the markdown examples**

For the **_real thing_** go to original [hypercore-protocol](https://github.com/mafintosh/hypercore-protocol) github repo.

## Documentation generation

Versions:

- protoc-gen-doc `1.0.0-rc`


### HTML from build-in template

Invoking `protoc-gen-doc` the standard way on [schema.proto](schema.proto):
```sh
  docker run --rm -v $(pwd)/build:/out -v $(pwd):/protos:ro pseudomuto/protoc-gen-doc
```

Generates this output:
- [Hypercore Protocol v1.0](https://aschrijver.github.io/hypercore-protocol/) specification ([source](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/index.html))

_Pro's_:
- Multi-format support. Works for all formats

_Con's_:
- No support for bold, italic, line-breaks, code snippets within a description

### HTML with custom template

Using [custom-html.tmpl](docgen/custom-html.tmpl) on schema [HypercoreSpecV1_html.proto](schemas/html/HypercoreSpecV1_html.proto):
```sh
  docker run --rm -v $(pwd)/build/html:/out -v $(pwd)/schemas/html:/protos:ro \
      -v $(pwd)/docgen:/templates:ro pseudomuto/protoc-gen-doc \
      --doc_opt=/templates/custom-html.tmpl,inline-html-comments.html \
       protos/HypercoreSpecV1_html.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://aschrijver.github.io/hypercore-protocol/html/inline-html-comments.html) specification ([source](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/html/inline-html-comments.html))

_Pro's_:
- Supports bold, italic, line-breaks, code snippets within a description

_Con's_:
- HTML-only format support. Other formats will be polluted with html tags
- Need to escape html characters that are part of the text, using `&lt;`, `&gt;`, etc.
- Not currently a viable option: [Field information not rendered with custom html.mustache templates](https://github.com/pseudomuto/protoc-gen-doc/issues/300)

### Build-in markdown template with inline markdown formatting

Invoking for markdown generation on schema [HypercoreSpecV1_md.proto](schemas/md/HypercoreSpecV1_md.proto):
```sh
  docker run --rm -v $(pwd)/build:/out -v $(pwd)/schemas/md:/protos:ro pseudomuto/protoc-gen-doc --doc_opt=markdown,hypercore-protocol.md
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol.md)

_Pro's_
- All inlne markdown commands are passed through to the output, allowing rich formatting

_Con's_:
- Markdown-only format support. Other formats will be polluted with markdown commands
- Not currently a viable option: Markdown for Messages incorrectly rendered by build-in template

### Custom markdown template

Invoking with [custom-markdown.tmpl](docgen/custom-markdown.tmpl) template:
```sh
  docker run --rm -v $(pwd)/build:/out -v $(pwd)/schemas/md:/protos:ro \
      -v $(pwd)/docgen:/templates:ro pseudomuto/protoc-gen-doc \
      --doc_opt=/templates/custom-markdown.tmpl,hypercore-protocol_custom-template.md \
       protos/HypercoreSpecV1_md.proto
```

Generates this output:
- [Hypercore Protocol v1.0](https://github.com/aschrijver/hypercore-protocol/blob/gh-pages/hypercore-protocol_custom-template.md)

_Pro's_
- Can fully customize the markdown output
- All inlne markdown commands are passed through to the output, allowing rich formatting

_Con's_:
- Markdown-only format support. Other formats will be polluted with markdown commands
- Not currently a viable option: Markdown for Fields not rendered by custom template

## TravisCI YAML

All the above generation options use the following `.travis.yml`:

```yaml
sudo: required

services:
  - docker

before_install:
  # (PS A bit lazy here, this should really be in a separate script invoked from the yaml)

  # Create directory structure, copy files
  - mkdir build && mkdir build/html
  - cp docgen/stylesheet.css build/html

  # Create all flavours of output formats to test (see README)
  - docker run --rm -v $(pwd)/build:/out -v $(pwd):/protos:ro pseudomuto/protoc-gen-doc
  - docker run --rm -v $(pwd)/build/html:/out -v $(pwd)/schemas/html:/protos:ro -v $(pwd)/docgen:/templates:ro pseudomuto/protoc-gen-doc --doc_opt=/templates/custom-html.tmpl,inline-html-comments.html protos/HypercoreSpecV1_html.proto
  - docker run --rm -v $(pwd)/build:/out -v $(pwd)/schemas/md:/protos:ro pseudomuto/protoc-gen-doc --doc_opt=markdown,hypercore-protocol.md
  - docker run --rm -v $(pwd)/build:/out -v $(pwd)/schemas/md:/protos:ro -v $(pwd)/docgen:/templates:ro pseudomuto/protoc-gen-doc --doc_opt=/templates/custom-markdown.tmpl,hypercore-protocol_custom-template.md protos/HypercoreSpecV1_md.proto

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

### Installation and configuration

- Create a personal access token in Github and configure it as environment variable in TravisCI
  - See TravisCI documentation here: [Github Pages Deployment](https://docs.travis-ci.com/user/deployment/pages/)

## Conclusion

[TODO]