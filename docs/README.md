# Docs as Code

***Documentation as Code*** (Docs as Code) refers to a philosophy that you should be writing documentation with the same tools as code:

* Issue Trackers
* Version Control (Git)
* Plain Text Markup (Markdown, reStructuredText, Asciidoc)
* Code Reviews
* Automated Tests

## Asciidoctor

**AsciiDoc** is a lightweight markup language that is used as a popular tool for generating any type of software documentation. The AsciiDoc syntax is readable, concise, comprehensive, extensible, and extremely intuitive. One of the core advantages of using AsciiDoc is the ability to leverage include directives. Since this is a native feature of AsciiDoc, writers and developers alike can single-source content much like a React component. Rather than updating the same content in multiple places, writers simply have a source file that is included where necessary, and changes cascade to all locations.

## Antora

**Antora** is a static-site generator that enables technical writers to generate an online documentation site from their AsciiDoc source files which are stored in one or more source control repositories. The generated site enforces a structured hierarchy of versions, components, and modules, and provides useful features out-of-the-box, like navigation and cross-referencing. Antora makes use of AsciiDoc’s native include directives through partials which can then be included in any page regardless of the developer repository from which the content is fetched. **Antora** is an open source documentation site generator. It allows you to host your product documentation in a version control system like `GitHub`, `GitLab` or `Bitbucket` and from that build a pretty website to display the content using an easily configurable *playbook* that controls the presentation and content versioning.

Antora Folder structure:

```txt
📒 docs 
  📄 antora-playbook.yml 
  📂 modules 
    📂 ROOT 
      📁 attachments 
      📁 examples 
      📁 images 
      📁 pages 
      📁 partials 
      📄 nav.adoc 
    📂 named-module 
      📁 pages
      📄 nav.adoc 
```

`antora-playbook-demo.yaml`

```yaml
site:
  title: Antora Demo Site
  # the 404 page and sitemap files only get generated when the url property is set
  url: https://antora.gitlab.io/demo/docs-site
  start_page: component-b::index.adoc
content:
  sources:
  - url: https://gitlab.com/antora/demo/demo-component-a.git
    branches: HEAD
    # setting edit_url to false disables the Edit this Page link for any page that originates from this repository
    # the same thing can be achieved by adding empty credentials (i.e., @) in front of the domain in the URL
    edit_url: false
  - url: https://gitlab.com/antora/demo/demo-component-b.git
    branches: [main, v2.0, v1.0]
    start_path: docs
asciidoc:
  attributes:
    experimental: ''
    idprefix: ''
    idseparator: '-'
    page-pagination: ''
ui:
  bundle:
    url: https://gitlab.com/antora/antora-ui-default/-/jobs/artifacts/HEAD/raw/build/ui-bundle.zip?job=bundle-stable
    snapshot: true
```

### Build

Antora build the html content from the AsciiDoc using Antora's playbook.

#### Package

A meta package for `Antora` that installs both the CLI (`@antora/cli`) and site generator (`@antora/site-generator`), as well as any dependencies of these packages. The CLI provides the antora command (i.e., bin script) to run Antora. The site generator provides the function invoked by the generate (default) command of the CLI. Other packages may be included with this package in the future.

```json
{
  "name": "docs",
  "version": "1.0.0",
  "description": "Documentation as Code (Docs as Code)",
  "main": "index.js",
  "scripts": {
    "build": "npx antora --clean --fetch antora-playbook.yaml --stacktrace",
    "start": "npx antora --clean --fetch antora-playbook-local.yaml --stacktrace && npx serve ./build/site",
    "verify": "npm ci && npm run build"
  },
  "license": "ISC",
  "dependencies": {
    "antora": "^3.1.4",
    "@antora/lunr-extension": "^1.0.0-alpha.8",
    "asciidoctor-emoji": "^0.5.0",
    "asciidoctor-kroki": "^0.17.0"
  }
}
```

In order to build the Documentation using Antora use following commands.

```bash
# If using nvm as npm package manager, select the desired npm version.
nvm use v18.12.1

# Install node modules dependencies from package.json
npm install

# Run the Antora playbook using antora cli.
npx antora antora-playbook.yml

# Build the remote Antora Playbook
npm run build

# Build the local Antora Playbook and start a Http sever at http://localhost:3000
npm run start
```

#### Container Image

Using Container images to build the Antora content generated content-

`Dockerfile`

```bash
FROM antora/antora:3.1.4

RUN apk update && apk add make git yq jq

RUN yarn cache clean
RUN yarn global add antora @antora/lunr-extension asciidoctor-kroki asciidoctor-emoji

# These environment variables are required since Antora 2.2
# to customize the "edit this page" URL
# https://docs.antora.org/antora/2.2/whats-new/#customizable-edit-url
ENV FORCE_SHOW_EDIT_PAGE_LINK=1
ENV CI=1
```

Use the following commands on the root of an Antora project.

In both cases you should have an output folder with the documentation following the instructions in the `antora-playbook-demo.yaml`file.

```bash
# Login to Github Container Registry (ghcr.io) using Personal Access Token (PAT)
echo $GITHUB_PAT | docker login ghcr.io -u $GITHUB_USERNAME --password-stdin

# Build image localy
docker build --pull -t ghcr.io/jsa4000/antora:3.1.4-custom .

# Push to Github Container Registry
docker push ghcr.io/jsa4000/antora:3.1.4-custom

# Build docs website
docker run --rm --user="$(id -u)" --volume "${PWD}":/antora ghcr.io/jsa4000/antora:3.1.4-custom docs/antora-playbook-demo.yaml --cache-dir=.cache/antora
docker run --rm --user="$(id -u)" --volume "${PWD}":/antora ghcr.io/jsa4000/antora:3.1.4-custom docs/antora-playbook-local.yaml --cache-dir=.cache/antora
```

> If you don’t specify the `--user` parameter, Docker will create output folders belonging to root.
