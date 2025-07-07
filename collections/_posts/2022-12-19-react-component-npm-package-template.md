---
# prettier-ignore
title: "React Component NPM Package Template"
excerpt: "An ES6 React component NPM package project template featuring front & back end test support, automated API docs, release management & more!"
header:
  og_image: /assets/images/logo-react.png
  teaser: /assets/images/logo-react.png
redirect_from:
  - /blog/react-component-npm-package-template/
tags:
  - javascript
  - npm
  - projects
  - react
  - template
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-react.png">
</figure>

You wrote a sweet React component! Releasing it on [NPM](https://www.npmjs.com/)
seems like the obvious next step. Right?

Try it. Not as easy to do from scratch as you might think.

{: .notice--warning}

**I just completely refactored this template** but haven't yet updated this documentaion. Everything works beautifully, and for a gist of the changes see the very similar [NPM Package Template](/blog/npm-package-template). I'll bring this page in sync with the changes soon!

So here's a plug-and-play
[react component NPM package template](https://github.com/karmaniverous/react-component-npm-package-template)
that offers the following features:

- Support for the latest ES6 + JSX goodies with
  [`eslint`](https://www.npmjs.com/package/eslint) _uber alles_.

- Automated [`lodash`](https://www.npmjs.com/package/lodash) cherry-picking with
  [`babel-plugin-lodash`](https://www.npmjs.com/package/babel-plugin-lodash).

- Front & back-end testing with [`mocha`](https://www.npmjs.com/package/mocha),
  [`chai`](https://www.npmjs.com/package/chai), and the
  [React Testing Library](https://www.npmjs.com/package/@testing-library/react).
  Includes examples and a sweet testing console!

- Code formatting at every save & paste with
  [`prettier`](https://www.npmjs.com/package/prettier).

- Automated documentation of your API with
  [`jsdoc-to-markdown`](https://www.npmjs.com/package/jsdoc-to-markdown) and
  assembly of your README with
  [`concat-md`](https://www.npmjs.com/package/concat-md).

- One-button release to GitHub & publish to NPM with
  [`release-it`](https://www.npmjs.com/package/release-it).

{: .text-center}

[See it on GitHub!](https://github.com/karmaniverous/react-component-npm-package-template){:
.btn .btn--info
.btn--large}&nbsp;&nbsp;&nbsp;[Clone the Repo!](https://github.com/karmaniverous/react-component-npm-package-template/generate){:
.btn .btn--primary .btn--large}

{: .notice--info}

If you want to create a non-React NPM package, try my regular
[NPM Package Template](/blog/npm-package-template) instead!

## Setting Up Your Dev Environment

**Use [VS Code](https://code.visualstudio.com/) as your code editor!** Not an
absolute requirement, but you'll be glad you did.

1.  [Click here](https://github.com/karmaniverous/npm-package-template/generate)
    to generate a new repository from this template.

1.  Clone the new repository to your local machine.

1.  VS Code will ask to install a bunch of recommended extensions. Accept all of them. If you miss this prompt, follow these steps:

    1. Open the VS Code Extensions tab
    1. Enter `@recommended` into the search box
    1. Click the Download link.

       <figure>
        <img src="/assets/images/react-component-npm-package-template-extensions.png" style="width: 250px;">
       </figure>

1.  Zero the package version and install dependencies by running these commands:

    ```bash
    npm version 0.0.0
    npm install
    ```

    This may produce an audit report. See [Vulnerabilities](#vulnerabilities)
    below for more info.

1.  Run your tests from the command line:

    ```bash
    npm run test

    # Component
    #   missing testid
    #     ✔ renders
    #   with testid
    #     ✔ renders
    #     ✔ is unaffected by click
    #
    # 3 passing (99ms)
    ```

    If you installed the VS Code extensions referenced above, use the `Testing`
    panel to visualize & run your unit tests.

     <figure>
      <a href="/assets/images/react-component-npm-package-template-testing-panel.png">
       <img src="/assets/images/react-component-npm-package-template-testing-panel.png">
      </a>
     </figure>

### Create Local Environment Variable File

Look for
[`.env.local.template`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/.env.local.template)
in your project directory. Copy this file and remove the `.template` extension
from the copy.

{: .notice--warning}

**Do not simply rename this file!** Anybody who pulls your repo will need this
template to create the same file in his own local environment.

### Connect to GitHub

This template supports automated release management with
[`release-it`](https://github.com/release-it/release-it).

If you use GitHub, create a
[Personal Access Token](https://github.com/settings/tokens/new?scopes=repo&description=release-it)
(`release-it` only needs "repo" access; no "admin" or other scopes). Add it as
the value of `GITHUB_TOKEN` in `.env.local`.

If you use GitLab, follow
[these instructions](https://github.com/release-it/release-it#gitlab-releases)
and place your token in the same file.

For other release control systems, consult the
[`release-it` README](https://github.com/release-it/release-it#readme).

You can now create a release at GitHub and optionally publish it to NPM with
this command:

```
npm run release
```

### Vulnerabilities

At the time of this writing, running `npm install` will generate the following
vulnerability warning:

```text
6 vulnerabilities (3 high, 3 critical)
```

If you run `npm audit`, you will find that all of these vulnerabilities relate
to the following dev dependencies, all of which are to do with docs generation:

```bash
npm list underscore

# @karmaniverous/npm-package-template@0.5.1-0
# ├─┬ concat-md@0.5.0
# │ └─┬ doctoc@1.4.0
# │   └── underscore@1.8.3
# └─┬ jsdoc-to-markdown@8.0.0
#   └─┬ jsdoc-api@8.0.0
#     └─┬ jsdoc@4.0.0
#       └── underscore@1.13.6

npm list trim

# @karmaniverous/npm-package-template@0.5.1-0
# └─┬ concat-md@0.5.0
#   └─┬ doctoc@1.4.0
#     └─┬ @textlint/markdown-to-ast@6.0.9
#       └─┬ remark-parse@5.0.0
#         └── trim@0.0.1
```

## NPM Scripts

| Script            | Description                                                                   |
| ----------------- | ----------------------------------------------------------------------------- |
| `npm run test`    | Runs all unit tests from the command line.                                    |
| `npm run build`   | Builds the project into the `lib` directory.                                  |
| `npm run doc`     | Builds the README file from the `doc` directory.                              |
| `npm run package` | Runs `test`, `build`, and `doc` to exercise your full packaging process.      |
| `npm run release` | Packages your code, creates a GitHub release, and publishes your code to NPM. |

## Common Tasks

### Develop Package Exports

All custom package code lives in the
[`src`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/src)
directory. Structure the contents of this directory however you like.

All package exports come together in [`src/index.jsx`](src/index.jsx). You can
cherry-pick from your own source and organize your exports however you like. You
can even [re-export](https://jamesknelson.com/re-exporting-es6-modules/) imports
from other packages!

{: .notice--warning}

Do not move or rename [`src/index.jsx`](src/index.jsx) or your build will break.

### Create & Run Unit Tests

By default, this template supports
[`mocha`](https://www.npmjs.com/package/mocha) tests using the
[`chai`](https://www.npmjs.com/package/chai) assertion library. The included
sample tests express the [`should`](https://www.chaijs.com/guide/styles/#should)
assertion syntax.

The default configuration will recognize any file as a test file that...

- has `.test.` just before its file name extension (i.e. `example.test.js`).
- is not located in the `node_modules` or `lib` directories.

The sample code packages tests next to the source code they exercise. If you
prefer to segregate your tests into a directory outside
[`src`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/src/)
(e.g. `test`), that will work as well.

Either way, all test files meeting the above conditions will be excluded from
the build.

To enable `mocha`-specific linting in your test files, add the following
directive at the top of every test file:

```js
/* eslint-env mocha */
```

The recommended
[Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)
extension will suface all of your tests into a sidebar console, nested to
reflect your `describe` hierarchy. It will also decorate your test source code
with test running and status reporting controls.

<figure>
  <img src="/assets/images/react-component-npm-package-template-testing-panel.png">
</figure>

### Test Your Build

#### Back-End Tests

`TODO`

#### Front-End Tests

This template supports front-end React component testing using the
[React Testing Library](https://www.npmjs.com/package/@testing-library/react)
(RTL). This library allows your tests to interact with your component in a
headless browser environment.

See
[`Component.test.jsx`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/src/Component/Component.test.jsx)
for a simple example, and visit the
[RTL documentation](https://testing-library.com/docs/react-testing-library/intro/)
for more info!

### Generate Documentation

`TODO`

### Integration-Test Your Package

A React component only makes sense when embedded in a React application.

To package your code and add it directly to your local development environment
as a global package, WITHOUT publishing it first to NPM, run these commands:

```bash
npm run package
npm link
```

You can now import your package assets into your React application like this:

```js
import Component from "@karmaniverous/react-component-npm-package-template"; // default export
import { useComponent } from "@karmaniverous/react-component-npm-package-template"; // named export
```

When you're finished, clean up your global environment by unlinking your
package:

```bash
npm unlink -g @karmaniverous/react-component-npm-package-template
```

### Create & Publish a Release

Before you can publish a package to [NPM](https://www.npmjs.com/), you'll need
to set up an NPM account.

#### Package Scope & Access

Your NPM user name is a [_scope_](https://docs.npmjs.com/about-scopes). If you
create an organization, its unique organization name is also a scope.

_Unscoped_ packages have names like
[`lodash`](https://www.npmjs.com/package/lodash). An unscoped package name must
be unique across NPM.

_Scoped_ packages have names like
[`@karmaniverous/serify-deserify`](https://www.npmjs.com/package/@karmaniverous/serify-deserify).
`@karmaniverous` in this case is the scope. A scoped package name only needs to
be unique within its scope.

NPM packages may be _public_ or _private_. A
[public package](https://docs.npmjs.com/about-public-packages) can be seen and
used by anyone. A
[private package](https://www.npmjs.com/package/@karmaniverous/serify-deserify)
can only be seen & used by your collaborators or other users with access to your
organization scope.

Only scoped packages can be private. Only paid NPM accounts can create private
packages.

Even if you are only creating public packages, it is a good idea to create
_scoped_ packages because it groups them logically and gives you much more
flexibility in naming them.

[Click here](https://docs.npmjs.com/packages-and-modules/introduction-to-packages-and-modules)
for more info about NPM package scope & access.

#### Configuring `package.json`

When you publish an NPM package, NPM gets most of its info from your
[`package.json`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/package.json)
file.

Set the following values in
[`package.json`](https://github.com/karmaniverous/react-component-npm-package-template/blob/main/package.json),
using the template file as an example.

This info is critical. You can't publish your package properly without it:

- `name` – The desired package name on NPM. Include scope if relevant. See
  [Package Scope & Access](#package-scope--access) for more info.

- `version` - Your package version. Uses
  [semantic versioning](https://www.geeksforgeeks.org/introduction-semantic-versioning/).
  Set this initially to `0.0.0` and the template's
  [release process](#create--publish-a-release) will manage it from there.

- `publishConfig.access` - `restricted` for private packages, otherwise
  `public`. See [Package Scope & Access](#package-scope--access) for more info.

- `repository.url` - GitHub repository URL.

This info is important but you can always update it in the next release:

- `author` - Your name, however you'd like it to appear.

- `bugs.url` - A URL for users to report bugs. By default, use the
  [issues page](https://github.com/karmaniverous/npm-package-template/issues) of
  your GitHub repo.

- `description` - A text description of your package. Will be used as the META
  description of your NPM package page, so keep it under 160 chars.

- `homepage` - The main web page of your project. By default, use your GitHub
  repo's
  [README link](https://github.com/karmaniverous/npm-package-template#readme).

- `keywords` - An array of strings that will appear as tags on the NPM package
  page.

- `license` - The license associated with your package. See this list of
  [valid license identifiers](https://spdx.org/licenses/).

#### Generating the Release

Before you begin, ensure you have committed all changes to your working branch.

Run this command:

```bash
npm run package
```

This will run all of your tests, generate all of your documentation, and create
your build. If there are any issues, fix them. If you make any changes, commit
them.

Now run this command:

```bash
npm run release
```

This will generate your package _again_, just to validate there are no more
changes. You will then be asked to select a release increment. Otherwise accept
all defaults.

Your release will be generated on GitHub and published to NPM.

Note that if you have configured Two-Factor Authentication at NPM you will be
asked to enter a One-Time Password (OTP).

Add other [`release-it`](https://github.com/release-it/release-it#readme)
options after `---` (Windows) or `--` (Mac/Linux). For example, to specify a
patch release, and accept all defaults with no user interaction, run this
command:

```bash
# Windows only.
npm run release --- patch --ci

# Mac/Linux.
npm run release -- patch --ci
```

See the [`release-it` README](https://github.com/release-it/release-it#readme)
for more info on available options.

### Integrate a Template Update

Follow
[these instructions](https://karmanivero.us/blog/installing-github-repo-template-updates/).
