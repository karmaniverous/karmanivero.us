---
# prettier-ignore
title: "NPM Package Template"
excerpt: "An ES6 NPM package project template featuring a CLI, test support, automated API docs, release management & more!"
header:
  og_image: /assets/images/logo-npm.png
  teaser: /assets/images/logo-npm.png
redirect_from:
  - /blog/npm-package-template/
tags:   
  - javascript
  - npm
  - projects
  - template
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-npm.png">
</figure>

You wrote a sweet piece of code! Releasing it on [NPM](https://www.npmjs.com/)
seems like the obvious next step. Right?

_Try it!_ Not as easy to do as you might think. At high quality. From scratch.

So here's a plug-and-play
[NPM package template](https://github.com/karmaniverous/npm-package-template)
that offers the following features:

- Tree-shakable support for the latest ES6 goodies with
  [`eslint`](https://www.npmjs.com/package/eslint) _uber alles_.

- CJS distributions targeting specific browser support scenarios.

- Command line interfaces for your widget with
  [`commander`](https://www.npmjs.com/package/commander).

- Automated [`lodash`](https://www.npmjs.com/package/lodash) cherry-picking with
  [`babel-plugin-lodash`](https://www.npmjs.com/package/babel-plugin-lodash).

- [`mocha`](https://www.npmjs.com/package/mocha) &
  [`chai`](https://www.npmjs.com/package/chai) for testing, with examples, and a
  sweet testing console.

- Code formatting at every save & paste with
  [`prettier`](https://www.npmjs.com/package/prettier).

- Automated documentation of your API with
  [`jsdoc-to-markdown`](https://www.npmjs.com/package/jsdoc-to-markdown) and
  assembly of your README with
  [`concat-md`](https://www.npmjs.com/package/concat-md).

- One-button release to GitHub & publish to NPM with
  [`release-it`](https://www.npmjs.com/package/release-it).

{: .text-center}

[See it on GitHub!](https://github.com/karmaniverous/npm-package-template){:
.btn .btn--info
.btn--large}&nbsp;&nbsp;&nbsp;[Clone the Repo!](https://github.com/karmaniverous/npm-package-template/generate){:
.btn .btn--primary .btn--large}

{: .notice--info}

**If you want to create a React component** in an NPM package, try my
[React Component NPM Package Template](/blog/react-component-npm-package-template)
instead!

## Setting Up Your Dev Environment

**Use [VS Code](https://code.visualstudio.com/) as your code editor!** Not an absolute requirement, but you'll be glad you did.

1.  [Click here](https://github.com/karmaniverous/npm-package-template/generate) to generate a new repository from this template.

1.  Clone the new repository to your local machine.

1.  VS Code will ask to install a bunch of recommended extensions. Accept all of them. If you miss this prompt, follow these steps:

    - Open the VS Code Extensions tab
    - Enter `@recommended` into the search box
    - Click the Download link.

       <figure>
        <a href="/assets/images/npm-package-template-extensions.png">
          <img src="/assets/images/npm-package-template-extensions.png" style="width: 250px;">
        </a>
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

    #  foo
    #    ✔ with input
    #    ✔ without input
    #
    #  2 passing (5ms)
    ```

    If you installed the VS Code extensions referenced above, use the `Testing`
    panel to visualize & run your unit tests.

     <figure>
      <a href="/assets/images/npm-package-template-testing-panel.png">
       <img src="/assets/images/npm-package-template-testing-panel.png">
      </a>
     </figure>

1.  Package your code and link it locally by running:

    ```bash
    npm run package
    npm link
    ```

1.  Enter a few of your package CLI commands:

    ```bash
    mycli

    # foo nil!

    mycli -f bar

    # foo bar!

    mycli -v

    # 0.0.0
    ```

1.  Clean up by unlinking your package.

    ```bash
    npm unlink -g @karmaniverous/npm-package-template
    ```

### Create Local Environment Variable File

Look for
[`.env.local.template`](https://github.com/karmaniverous/npm-package-template/blob/main/.env.local.template)
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

{: .notice--info}

These vulnerable dependencies will NOT be included in your NPM package!

## Package Structure

This template produces a hybrid NPM package that features both native ES and
platform-specific CommonJS module entry points as well as a native ES CLI.

The template supports tree-shakable `import` from its native ES library entry
point at at
[`lib/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/lib/index.js).
**This is where you write your source code!** See
[`import` Entry Points](#import-entry-points) for more info on
developing to this entry point and adding new ones.

The template supports `require` from its platform-specific CommonJS library
entry point configured at
[`dist/default`](https://github.com/karmaniverous/npm-package-template/tree/main/dist/default).
**This code is generated by Babel** and not pushed to the repository, so you
won't see it here. See
[`require` Entry Points](#require-entry-points) for more
info on configuring this entry point and adding new ones.

The template features a native ES command-line interface (CLI) entry point at
[`bin/mycli/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/bin/mycli/index.js).
**This is where you write your CLI!** See
[`bin` Entry Points](#bin-entry-points) below for more info on
configuring this entry point and adding new ones.

After you've [set up your dev environment](#setting-up-your-dev-environment),
your package will look like this. The `git` & `npm` columns indicate which files
participate in `git push` and `npm publish`, respectively.

```text
git npm    Item                         Description
---------------------------------------------------------------------------
        └─ npm-package-template ....... package root
 x ....... ├─ .babelrc ................ global Babel config
 x . x ... ├─ .env .................... global env variables
 ......... ├─ .env.local .............. create from template
 x . x ... ├─ .env.local.template ..... local env secrets template
 x ....... ├─ .eslintrc.json .......... global ESLint config
 x ....... ├─ .gitignore .............. package git ignore
 x ....... ├─ .npmignore .............. package npm ignore
 x ....... ├─ .vscode ................. VS Code project config
 x ....... │  ├─ extensions.json ...... recommended extensions
 x ....... │  └─ settings.json ........ project settings
 x . x ... ├─ bin ..................... all CLI source code (ES)
 x . x ... │  └─ mycli ................ mycli code (ES)
 x . x ... │     └─ index.js .......... mycli entry point (ES)
 x . x ... ├─ dist .................... all CommonJS distributions
 x . x ... │  ├─ default .............. specific CommonJS distribution
 x ....... │  │  ├─ .babelrc .......... Babel target settings
 x ....... │  │  ├─ .gitignore ........ don't send the lib to git...
 x ....... │  │  ├─ .npmignore ........ but DO send it to NPM
 x . x ... │  │  └─ lib/ .............. dist code (generated by Babel)
 x . x ... │  └─ package.json ......... dist code is CommonJS
 x ....... ├─ doc ..................... all documentation config
 x ....... │  ├─ .prettierignore ...... ignore handlebars formatting
 x ....... │  ├─ 1-main.md ............ your main README content
 x ....... │  ├─ 2-cli.md ............. your CLI documentation
 x ....... │  ├─ 3-api.md ............. api docs (generated by jsdoc)
 x ....... │  ├─ 4-footer.md .......... your README footer
 x ....... │  ├─ api-template.hbs ..... your API docs template
 x ....... │  └─ jsdoc.config.json .... ignore test code
 x . x ... ├─ lib ..................... your library source code (ES)
 x . x ... │  ├─ foo .................. structure your code how you like
 x . x ... │  │  ├─ foo.js ............ source code module (ES)
 x ....... │  │  └─ foo.test.js ....... test script (ES)
 x . x ... │  ├─ index.js ............. library import entry point (ES)
 x ....... ├─ package-lock.json ....... dependencies (managed by npm)
 x . x ... ├─ package.json ............ package config
 x . x ... └─ README.md ............... generated by jsdoc
```

## NPM Scripts

| Script                                                      | Description                                                                                                                                                                                       |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `npm run lint`                                              | Executes [`eslint`](https://www.npmjs.com/package/eslint) to find errors in your code.                                                                                                            |
| `npm run test`                                              | Executes [`mocha`](https://www.npmjs.com/package/mocha) to run all unit tests.                                                                                                                    |
| `npm run build`                                             | Executes [`babel`](https://babeljs.io/docs/en/babel-cli) to build all CJS distributions.                                                                                                          |
| `npm run doc`                                               | Executes [`jsdoc2md`](https://www.npmjs.com/package/jsdoc-to-markdown) to build your README file from the [`doc`](https://github.com/karmaniverous/npm-package-template/tree/main/doc) directory. |
| `npm run package`                                           | Runs `lint`, `test`, `build` & `doc` to exercise your full packaging process.                                                                                                                     |
| <span style="white-space: nowrap;">`npm run release`</span> | Runs `package` & executes [`release-it`](https://www.npmjs.com/package/release-it) to create a GitHub release & publish your code to NPM.                                                         |

## Package Entry Points

### `import` Entry Points

All custom library code lives in the
[`lib`](https://github.com/karmaniverous/npm-package-template/blob/main/lib)
directory. This is native ES code and you can structure it however you like.

Package `import` entry points are defined in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json)
like this:

```json
{
  "exports": {
    ".": {
      "import": "./lib/index.js"
    }
  }
}
```

There is currently a single `import` entry point defined at the package root,
which points to
[`lib/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/lib/index.js).
The entry point will expose whatever this module exports, and can be referenced
in an ES module like this:

```js
import { foo, PACKAGE_INFO } from '@karmaniverous/npm-package-template`;
```

Some key guidelines:

- Avoid using module default exports, even internally. This can create
  unexpected weirdness in the Babel-generated CJS distributions. Use named
  exports instead.

See the
[Node Package Entry Points](https://nodejs.org/docs/latest-v18.x/api/packages.html#package-entry-points)
documentation for more info on defining additional entry points.

{: .notice--warning}

Do not move or rename
[`lib/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/lib/index.js)
without making matching changes in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json),
or your ES exports will not work!

### `require` Entry Points

The CJS distributions behind your `require` entry points are generated by Babel
from your ES source code in
[`lib`](https://github.com/karmaniverous/npm-package-template/blob/main/lib).
Each distribution is optimized for a set of browser constraints as expressed by
a [browserslist query](https://github.com/browserslist/browserslist#queries).

The template's default distribution is located at
[`dist/default`](https://github.com/karmaniverous/npm-package-template/tree/main/dist/default)
and uses
[`.babelrc`](https://github.com/karmaniverous/npm-package-template/blob/main/dist/default/.babelrc)
to set `browserslist` query `"defaults"`.

Package `require` entry points are defined in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json)
like this:

```json
{
  "exports": {
    ".": {
      "require": "./dist/default/lib/index.js"
    }
  }
}
```

There is currently a single `require` entry point defined at the package root,
which points to `dist/default/lib/index.js`. The entry point will expose
whatever this module exports, and can be referenced in CJS code like this:

```js
const { foo, PACKAGE_INFO } = require('@karmaniverous/npm-package-template`);
```

See the
[Node Package Entry Points](https://nodejs.org/docs/latest-v18.x/api/packages.html#package-entry-points)
documentation for more info on defining additional entry points.

To create a new CJS distribution:

1. Duplicate the `default` directory in `dist` and give the copy a meaningful
   name (e.g. `mydist`).

1. In the new directory's `.babelrc` file, change the `targets` value to reflect
   this distribution's
   [`browserslist` query](https://github.com/browserslist/browserslist#queries):

   ```json
   {
     "targets": "new query"
   }
   ```

1. Add a new `require` entry point and a new `build` script step to your
   project's
   [`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json):

   ```json
   {
     "exports": {
       "./mydist": {
         "require": "./dist/mydist/lib/index.js"
       }
     },
     "scripts": {
       "build": "babel lib -d dist/default/lib --delete-dir-on-start --config-file ./dist/default/.babelrc && babel lib -d dist/mydist/lib --delete-dir-on-start --config-file ./dist/mydist/.babelrc"
     }
   }
   ```

1. Build the new distribution with `npm run build`.

{: .notice--warning}

Do not move or rename
[`dist/default.js`](https://github.com/karmaniverous/npm-package-template/blob/main/dist/default)
without making matching changes in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json),
or your CJS exports will not work!

### `bin` Entry Points

Each command-line interface (CLI) entry point is a subdirectory of your
[`bin`](https://github.com/karmaniverous/npm-package-template/blob/main/bin)
directory. These are native ES code and you can structure them internally
however you like.

CLI entry points are defined in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json)
like this:

```json
{
  "bin": {
    "mycli": "bin/mycli/index.js"
  }
}
```

There is currently a single CLI entry point defined at the package root, which
points to
[`bin/mycli/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/bin/mycli/index.js)
and exposes the `mycli` command.

See the
[NPM documentaion](https://docs.npmjs.com/cli/v9/configuring-npm/package-json#bin)
documentation for more info on defining CLI entry points.

To create a new CLI entry point:

1. Create a new subdirectory under
   [`bin`](https://github.com/karmaniverous/npm-package-template/blob/main/bin)
   (e.g. `myothercli`).

1. Add an `index.js` file expressing your command logic, along with other
   supporting code. Use [`commander`](https://www.npmjs.com/package/commander)
   for great results!

1. Add the new CLI entry point to your
   [`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json)
   like this:

```json
{
  "bin": {
    "myothercli": "bin/myothercli/index.js"
  }
}
```

{: .notice--warning}

Do not move or rename
[`bin/mycli/index.js`](https://github.com/karmaniverous/npm-package-template/blob/main/bin/mycli/index.js)
without making matching changes in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json),
or your CLI will not work!

## Unit Testing

By default, this template supports
[`mocha`](https://www.npmjs.com/package/mocha) tests using the
[`chai`](https://www.npmjs.com/package/chai) assertion library. The included
sample tests express the [`should`](https://www.chaijs.com/guide/styles/#should)
assertion syntax.

The default configuration will recognize any file as a test file that...

- has `.test.` just before its file name extension (i.e. `example.test.js`).
- is not located in the `node_modules` or `dist` directories.

The sample code packages tests next to the source code they exercise. If you
prefer to segregate your tests and related artifacts into a `test` directory,
that will work as well.

Either way, all test files meeting the above conditions and _anything_ under a
`test` directory will be excluded from the build.

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
  <a href="/assets/images/npm-package-template-testing-panel.png">
    <img src="/assets/images/npm-package-template-testing-panel.png">
  </a>
</figure>

## Package Documentation

When you run `npm run doc`, these two steps happen in order:

1. [`jsdoc2md`](https://www.npmjs.com/package/jsdoc-to-markdown) parses the
   source code in
   [`lib`](https://github.com/karmaniverous/npm-package-template/tree/main/lib)
   for [JSDoc](https://jsdoc.app/index.html)-formatted comments, and generates
   API documenation in
   [`doc/3-api.md`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/3-api.md)
   using
   [`doc/api-template.hbs`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/api-template.hbs)
   as a template. It ignores test files and anything with the
   [`@private`](https://jsdoc.app/tags-private.html) tag.

2. [`concat-md`](https://www.npmjs.com/package/concat-md) concatenates every
   `.md` file in
   [`doc`](https://github.com/karmaniverous/npm-package-template/blob/main/doc)
   into your main
   [`README`](https://github.com/karmaniverous/npm-package-template/blob/main/README.md)
   file. Files are concatenated in filename order (hence the numbered files).

You can exert fine control over the final result by doing the following:

- **Edit
  [`1-main.md`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/1-main.md).**
  This is the main content of your README file. At a minimum it should contain a
  header and a brief description of the project, but it can be whatever you
  want.

- **Edit or delete
  [`2-cli.md`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/2-cli.md).**
  This is your CLI documentation. The easiest way to get this is to run your
  CLI's help function on the command line & copy the result (e.g. `mycli -h`).
  If your package has no CLI, just delete this file.

- **Edit
  [`api-template.hbs`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/api-template.hbs).**
  This is your API documentation template.
  [`jsdoc2md`](https://www.npmjs.com/package/jsdoc-to-markdown) will render
  documentation based on the [JSDoc](https://jsdoc.app/index.html)-formatted
  comments in your source code and render them at the `{{ '{{' }}>main}}` tag in
  the template. Add any other content in Markdown format.

- **Edit
  [`4-footer.md`](https://github.com/karmaniverous/npm-package-template/blob/main/doc/4-footer.md).**
  This footer will appear at the bottom of your
  [`README`](https://github.com/karmaniverous/npm-package-template/blob/main/README.md)
  file.

Some tips:

- Add whatever additional `.md` files you like! Just remember they will be
  rendered in filename order.

- If your main content is really long, consider linking to a blog post instead
  (like this one!). That way you can update your documentation without having to
  create a new release.

- Use [`commander`](https://www.npmjs.com/package/commander) to
  [create your CLI](#develop-cli-entry-points) and take full advantage of its
  help system (e.g. program & option descriptions). This gives you a more usable
  CLI, and as a side bonus you just need to run `mycli -h` to copy & paste your
  full CLI documentation.

  ```text
  > mycli -h
  Usage: mycli [options]

  Foos your bar.

  Options:
    -b, --bar <string>  foo what?
    -v, --version       display package version
    -h, --help          display help for command
  ```

- Take full advantage of [JSDoc](https://jsdoc.app/index.html) by leveraging
  [`@typedef`](https://jsdoc.app/tags-typedef.html) syntax (an
  [example](https://github.com/karmaniverous/serify-deserify/blob/main/lib/options/types.js))
  and importing types from a common file (another
  [example](https://github.com/karmaniverous/serify-deserify/blob/d935ad091d43b4aefc4677c067ac706b89f4cf66/lib/serify/serify.js#L5)).

{: .notice--warning}

**Don't edit your
[`README`](https://github.com/karmaniverous/npm-package-template/blob/main/README.md)
file directly!** Any changes you make will be lost the next time you run
`npm run doc`.

## Integration Testing

Your unit tests exercise your code _internally_. Integration testing validates
your code _externally_, and might include the following:

- Testing your `import` entry points from external code.

- Testing your `require` entry points from external code.

- Testing your CLI entry points from the command line.

There are several ways to accomplish these:

1. Run `npm run link` to install a global symlink to your local package. This
   has an effect similar to `npm install -g my-package`, and will enable you to
   test your CLI entry points from your local command line without publishing to
   NPM.

1. Run `npm install c:/package-path/my-package` to install your package as a
   file dependency in another package. This will enable you to test your
   `import` and `require` entry points without publishing to NPM. Run a global
   install or use `npx mycli` to test your CLI entry points.

1. [Publish a release](#create--publish-a-release) to NPM with a pre-release
   version number. Then run `npm install my-package 0.0.1-0` (reflecting the
   pre-release version) to install your package into another package as a
   dependency, and test as required.

Each of these approaches has its trade-offs, but a smart plan is to skip the
first and do the last two, in that order.

**_If the second approach passes, why bother with the third?_**

Because your local project contains _all_ your files, whereas your package
`.npmignore` files determine which ones actually get published to NPM. That last
approach validates that your published package has the expected contents.

As a final check, review your package's Code tab at NPM in order to validate
that your `.npmignore` files _blocked_ everything they should have (e.g.
environment secrets, dev tool configs).

<figure>
  <a href="/assets/images/serify-deserify-code-tab.png">
    <img src="/assets/images/serify-deserify-code-tab.png">
  </a>
</figure>

## Create & Publish a Release

Before you can publish a package to [NPM](https://www.npmjs.com/), you'll need
to set up an NPM account.

### Package Scope & Access

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

### Configuring `package.json`

When you publish an NPM package, NPM gets most of its info from your
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json)
file.

Set the following values in
[`package.json`](https://github.com/karmaniverous/npm-package-template/blob/main/package.json),
using the template file as an example.

This info is critical. You can't publish your package properly without it:

- `name` – The desired package name on NPM. Include scope if relevant. See
  [Package Scope & Access](#package-scope--access) for more info.

- `version` – Your package version. Uses
  [semantic versioning](https://www.geeksforgeeks.org/introduction-semantic-versioning/).
  Set this initially to `0.0.0` and the template's
  [release process](#create--publish-a-release) will manage it from there.

- `publishConfig.access` – `restricted` for private packages, otherwise
  `public`. See [Package Scope & Access](#package-scope--access) for more info.

- `exports` – Your [`import`](#develop-import-entry-points) and
  [`require`](#configure-require-entry-points) entry points. Click the links for
  more info.

- `bin` – Your CLI entry points. See
  [`bin` Entry Points](#bin-entry-points) above for more info.

This info is important but you can always update it in the next release:

- `author` – Your name, however you'd like it to appear.

- `bugs.url` – A URL for users to report bugs. By default, use the
  [issues page](https://github.com/karmaniverous/npm-package-template/issues) of
  your GitHub repo.

- `description` – A text description of your package. Will be used as the META
  description of your NPM package page, so keep it under 160 chars.

- `homepage` – The main web page of your project. By default, use your GitHub
  repo's
  [README link](https://github.com/karmaniverous/npm-package-template#readme).

- `keywords` – An array of strings that will appear as tags on the NPM package
  page.

- `license` – The license associated with your package. See this list of
  [valid license identifiers](https://spdx.org/licenses/).

- `repository.url` – GitHub repository URL.

### Generating the Release

Before you begin, ensure you have committed all changes to your working branch.

Run this command:

```bash
npm run package
```

This will check your code for errors, run all of your tests, generate all of
your documentation, and create your build. If there are any issues, fix them. If
you make any changes, commit them.

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

## Integrate a Template Update

Follow
[these instructions](https://karmanivero.us/blog/installing-github-repo-template-updates/).

## Issues

- The documentation system is choking on dynamic import assertions for for now I've added an exclusion to the one directory that uses them ([packageInfo](https://github.com/karmaniverous/npm-package-template/tree/main/lib/packageInfo)).
