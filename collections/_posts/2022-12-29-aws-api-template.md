---
# prettier-ignore
title: "AWS API Template"
excerpt: "Config-driven AWS API ES6 template integrating federated Cognito User Pool authentication and a robust approach to release management."
header:
  og_image: /assets/images/logo-aws.png
  teaser: /assets/images/logo-aws.png
redirect_from:
  - /blog/aws-api-template/
tags:
  - aws
  - javascript
  - projects
  - serverless
  - template
toc: true
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-aws.png">
</figure>

Getting your application's back end up and running on [Amazon Web Services](https://aws.amazon.com/) (AWS) is not a trivial exercise, especially if you want a robust and extensible result that will support a modern development process.

Here's a plug-and-play [AWS API template](https://github.com/karmaniverous/aws-api-template) that offers the following features:

- Produces a secure [Amazon Web Services](https://aws.amazon.com/) (AWS) API using the [Serverless Framework](https://www.serverless.com/).

- Features both public and private endpoints.

- Secured by an [AWS Cognito User Pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) supporting native username/password authentication and one federated identity provider (Google).

- Configured to act as both the authentication provider and a secure remote API for my [Next.js Template](https://github.com/karmaniverous/nextjs-template) on the front end.

- Efficient and highly configurable. You should be able to get it up and running on your own domain, in your own infrastructure, in just a few minutes with edits to nothing but environment variables.

- Deployable from the command line to multiple [AWS CloudFormation](https://aws.amazon.com/cloudformation/) Stacks, each exposing an independent environment (e.g. `dev`, `test`, and `prod`), with its own authentication provider, at configurable endpoints.

- Unit testing support with [`mocha`](https://www.npmjs.com/package/mocha) and [`chai`](https://www.npmjs.com/package/chai). Includes examples and a sweet testing console!

- Built-in backwards compatibility. Every major release triggers the deployment of an independent Stack on every environment. Share key resources across Stacks so you can bring your users with you across major versions.

- Automatically build and deploy the relevant Stack following every code push with [AWS CodePipeline](https://aws.amazon.com/codepipeline/). See [Automated Deployment](#automated-deployment) below for more info.

- Code formatting at every save & paste with
  [`prettier`](https://www.npmjs.com/package/prettier).

- One-button release to GitHub with
  [`release-it`](https://www.npmjs.com/package/release-it).

{: .notice--info}

This template's repo branches are in fact hooked into CodePipeline to support deployments to demo environments. Consequently you will see many references to the `karmanivero.us` domain. Obviously, you'll need to replace these with references to your own domain.

{: .text-center}

[See it on GitHub!](https://github.com/karmaniverous/aws-api-template){:
.btn .btn--info
.btn--large}&nbsp;&nbsp;&nbsp;[Clone the Repo!](https://github.com/karmaniverous/aws-api-template/generate){:
.btn .btn--primary .btn--large}

## The Tech Stack

This template represents a specific solution to a bunch of specific problems. It makes a number of highly opinionated choices.

### Node.js & ES6

Choose your own poison. I like Javascript.

The [AWS Lambda](https://aws.amazon.com/lambda/) execution environment now supports Node `v18`, which of course features full native ES6 support. If you are accustomed to building for the lowest common denominator with [Webpack](https://webpack.js.org/) & [Babel](https://babeljs.io/), rejoice: it just isn't necessary here!

This probably makes for a much simpler project structure than you are accustomed to.

### The Serverless Framework

There are many different ways to script the creation of AWS resources.

The [AWS CLI](https://aws.amazon.com/cli/) is the gold standard. It's maximally powerful, but very low level. It's probably smarter than I am.

[AWS Amplify](https://aws.amazon.com/amplify/) is new, super powerful, and supported by a great UI. It's also very HIGH-level and doesn't yet enjoy broad community support.

The [Serverless Framework](https://www.serverless.com/) is a well-supported, open-source wrapper over the AWS CLI. It is opinionated with respect to best practices, offers a ton of plugins, and is accessible to the talented amateur, of which I am one.

There are other choices as well, but we're using the Serverless Framework here because it seemed like a reasonably safe choice that offers plenty of room to grow.

Worth noting that I tried & rejected these three key Serverless Framework plugins as either inadequate to the task or too unstable to trust:

- [`serverless-certificate-creator`](https://www.serverless.com/plugins/serverless-certificate-creator)
- [`serverless-dotenv-plugin`](https://www.serverless.com/plugins/serverless-dotenv-plugin)
- [`serverless-plugin-ifelse`](https://www.serverless.com/plugins/serverless-plugin-ifelse)

The necessary functionality is in the template; I just found better ways of getting there.

### Environments, API Versions, Stages & Stacks

**This is important!** This document uses specific terms in specific ways, and it will help if we are all on the same page.

Some key definitions:

- An **Environment** is a logical step in your development process. This template currently supports three environments: `dev`, `test`, and `prod`. See [Some Thoughts About DevOps](#some-thoughts-about-devops) below for ideas about how to use these effectively.

- An **API Version** is a distinct [major version](https://www.geeksforgeeks.org/introduction-semantic-versioning/) of your project. Major versions introduce breaking changes, so when you bump your major version, you want to keep your previous versions operating to support backward compatibility. This project defaults in API Version `v0`; subsequent values would be `v1`, `v2`, and so on.

- **Stage** relates specifically to [AWS API Gateway](https://aws.amazon.com/api-gateway/). API Gateway supports multiple "environments" and "versions" (see above) on a single API Gateway instance. We will deploy each environment version to its _own_ API Gateway, so only one Stage each. This is normally a key term in Serverless deployments, but this template handles Stage construction for you, so I'll try to use this term as little as possible in this document to avoid confusion. Examples of Stages are `aws-api-template-v0-dev`, `aws-api-template-v1-prod`, and so on.

- When you deploy a specific API Version into a specific Environment, this template creates a completely independent [CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html). A **Stack** contains all of the resources associated with that deployment, which may be updated in future deployments to the same Stack. Examples of Stacks are `aws-api-template-v0-dev`, `aws-api-template-v1-prod`, and so on. Stages and Stacks are PHYSICALLY very different, but LOGICALLY equivalent. To avoid confusion, in this document I will refer to Stacks rather than Stages wherever possible.

### Project Configuration

This template deploys multiple Environments across multiple API Versions in a highly configurable fashion. Both the project as a whole and individual environments include _secrets:_ configurations that need to exist in your dev environment to support local testing and manual deployment, but should NOT be pushed to your code repository.

The key AWS configuration file in the project is [`serverless.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/serverless.yml). Nominally this file would contain most of this stuff, but I thought it would be a good idea to abstract configuration data away from this highly structured file.

Everybody understands `.env` environment variable files, so that seemed to be a great way to go, which is nominally supported by the [Serverless Dotenv Plugin](https://www.serverless.com/plugins/serverless-dotenv-plugin). Unfortunately, I couldn't get it to work.

So I eliminated this dependency and am relying instead on my own [`get-dotenv`](https://www.npmjs.com/package/@karmaniverous/get-dotenv) package, which adds some handy features to the very robust [`dotenv-cli`](https://www.npmjs.com/package/dotenv-cli) tool, actually does everything the plugin only promises, and works like a charm.

Because of this, you will almost NEVER run `sls` directly! Instead, you will preface it with `npx getdotenv` and specify your `.env` file directories & target environment, like this:

```bash
npx getdotenv -p ./ ./env -e dev -- sls deploy --verbose
```

... except you wouldn't do THAT either, because this command is now encapsulated in the package script `npm run deploy`.

More info on this in the relevant sections below. Later I will encapsulate this approach and some other useful features into a CLI wrapper for `sls`. ([#10](https://github.com/karmaniverous/aws-api-template/issues/10))

### Release Management

This template supports automated release management with [`release-it`](https://github.com/release-it/release-it).

Releases are numbered according to [Semantic Versioning](https://www.geeksforgeeks.org/introduction-semantic-versioning/). Every release generates a release page at GitHub with release notes composed of your commit comments since the last release.

In the near future ([#10](https://github.com/karmaniverous/aws-api-template/issues/10)), the template will pick up on your major version number to populate API Version, which is a factor driving Stack creation. See [Environments, API Versions, Stages & Stacks](#environments-api-versions-stages--stacks) above for more info.

Meanwhile, you'll need to populate the `API_VERSION` environment variable manually in [`.env`](https://github.com/karmaniverous/aws-api-template/blob/main/.env).

To create a new release, just run this command:

```bash
npm run release
```

## Setting Up Your Dev Environment

Use the [Visual Studio Code](https://code.visualstudio.com/) IDE in order to leverage relevant extensions. Not an absolute requirement but you'll be happy if you do.

These instructions are written from the perspective of Windows OS. If you are a Mac or Linux user, you may need to make some adjustments.

They also assume you have local administrator permissions on your development machine.

### Install Applications & Global Packages

This template is a [Node.js](https://nodejs.org/en/) project. Node.js is now in `v19`, but the latest [AWS Lambda](https://aws.amazon.com/lambda/) Node.js execution environment is `v18`. The current Node.js LTS version is `v18.12.1`, so we'll use that one.

Rather than install Node.js directly, it is better to install it & manage versions from the the Node Version Manager (NVM). Follow these instructions to configure your system:

1. If you already have Node.js installed on your machine, uninstall it completely and remove all installation files. You can easily use NVM to reinstall & switch between other desired Node versions.

1. Follow [these instructions](https://www.freecodecamp.org/news/node-version-manager-nvm-install-guide/) to install NVM on your system. **This is a Windows-specific implementation!** Mac/Linux users, adjust accordingly.

1. Once NVM is installed on your system, open a terminal with **System Admin Permissions** and run the following commands:

   ```bash
   # windows only
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

   # all platforms
   nvm install 18.12.1
   nvm use 18.12.1
   npm install -g serverless
   ```

1. Install Git for your operating system from [this page](https://git-scm.com/download).

1. Install Visual Studio Code from [this page](https://code.visualstudio.com/).

1. **WINDOWS ONLY:** There is an alias conflict between the `serverless` package and the PowerShell `Select-String` command.

   To resolve it, open VS Code. In a terminal window, enter `code $profile`. A file named `Microsoft.PowerShell_profile.ps1` will open in your code editor. Add the following line to this file, then save and close it:

   ```bash
   Remove-Item alias:sls
   ```

   If you choose not to resolve this, than at any point in these instructions where I use `sls`, you should use `serverless` instead.

1. **Restart your machine.**

### Create a new GitHub Repository

[Click here](https://github.com/karmaniverous/aws-api-template/generate) to clone this template into your own GitHub account.

**Do you already have a repository based on this template?** No need to do this then.

### Clone the Project Repository

Navigate your VS Code terminal to the directory you use for code repositories and run this command with appropriate replacements to clone this repo. You may be asked to log into GitHub.

```bash
git clone https://github.com/<my-github-account-or-organization-name>/<my-project-name>.git
```

Open the newly created local repository folder in VS Code.

If this is your first time opening the folder, you should be asked to install recommended VS Code extensions. Install them.

If VS Code didn't ask, follow these steps to install all workspace-recommended extensions:

1. Open the VS Code Extensions tab

2. Enter `@recommended` into the search box

3. Click the Download link.

   <figure>
     <img src="/assets/images/aws-api-template-extensions.png" style="width: 250px;">
   </figure>

Zero the package version and install dependencies by running these commands:

```bash
npm version 0.0.0
npm install
```

Set the version in [`package.json`](https://github.com/karmaniverous/aws-api-template/blob/main/package.json) to `0.0.0`.

### Vulnerabilities

At the time of this writing, running `npm install` will generate the following
vulnerability warning:

```text
6 vulnerabilities (3 high, 3 critical)
```

If you run `npm audit`, you will find that all of these vulnerabilities relate
to the following dev dependencies, all of which are to do with docs generation:

```bash

> npm list trim

# @karmaniverous/aws-api-template@0.2.1
# └─┬ concat-md@0.5.0
#   └─┬ doctoc@1.4.0
#     └─┬ @textlint/markdown-to-ast@6.0.9
#       └─┬ remark-parse@5.0.0
#         └── trim@0.0.1

> npm list underscore

# @karmaniverous/aws-api-template@0.2.1
# ├─┬ concat-md@0.5.0
# │ └─┬ doctoc@1.4.0
# │   └── underscore@1.8.3
# └─┬ jsdoc-to-markdown@8.0.0
#   └─┬ jsdoc-api@8.0.0
#     └─┬ jsdoc@4.0.0
#       └── underscore@1.13.6
```

{: .notice--info}

These vulnerable dependencies support the development environment only and will NOT be included in your production code!

### Create Local Environment Variable Files

Look for these files in your project directory:

- `.env.local.template`
- `env/.env.dev.local.template`
- `env/.env.test.local.template`
- `env/.env.prod.local.template`

Copy each of these files and remove the `template` extension from the copy.

**Do not simply rename these files!** Anybody who pulls your repo will need these templates to create the same files in his own local environment.

In the future, this will be accomplished with a single CLI command. ([#10](https://github.com/karmaniverous/aws-api-template/issues/10))

### Connect to GitHub

This template supports automated release management with [`release-it`](https://github.com/release-it/release-it).

If you use GitHub, create a [Personal Access Token](https://github.com/settings/tokens/new?scopes=repo&description=release-it)
and add it as the value of `GITHUB_TOKEN` in `.env.local`.

If you use GitLab, follow [these instructions](https://github.com/release-it/release-it#gitlab-releases) and place your token in the same file.

For other release control systems, consult the [`release-it` README](https://github.com/release-it/release-it#readme).

You can now publish a release to GitHub with this command:

```
npm run release
```

### Connect to AWS

You will need to connect to AWS in order to interact with AWS resources during development.

Generate an Access Key Id and Secret Access Key for either...

- Your AWS Root account (not recommended)
- An [IAM](https://aws.amazon.com/iam/) user with admin permissions (or at least enough to support development)

If you aren't able to do this, ask your AWS administrator.

Add those two values in the appropriate spots in `.env.local`.

### Test Your Setup

Enter the following command in a terminal window:

```bash
npm run offline
```

A local server should start with an endpoint at [`http://localhost:3000/v0-dev/hello`](http://localhost:3000/v0-dev/hello). If you navigate to this endpoint in a browser, you should see a blob of JSON with the message `Hello world!`

## Deploying to AWS

Every AWS deployment of this project either creates or updates a [Stack](#environments-api-versions-stages--stacks).

Which Stack gets created or updated depends on two factors:

- The [Environment](#environments-api-versions-stages--stacks) chosen at deployment time.
- The [API Version](#environments-api-versions-stages--stacks) configured in [`.env`](https://github.com/karmaniverous/aws-api-template/blob/main/.env).

Each Stack contains all of the AWS resources associated with that deployment, and exposes a unique set of API & authentication endpoints.

The assumption is that these endpoints will map onto a custom domain, so there are some configurations to set and a little work to do in AWS before you can deploy a Stack to AWS.

### Preparing AWS

Follow these steps to prepare your AWS account to receive your deployment. Where you are asked to set an environment variable in this section, set it at [`.env`](https://github.com/karmaniverous/aws-api-template/blob/main/.env):

1. Choose a name for your service (e.g. `my-aws-api`). This will be the root of every resource ID related to your application. It must begin with a letter; contain only ASCII letters, digits, and hyphens; and not end with a hyphen or contain two consecutive hyphens. Enter your service name for environment variable `SERVICE_NAME`.

1. Choose a root domain (e.g. `mydomain.com`) and host its DNS at [AWS Route53](https://aws.amazon.com/route53/). Enter your root domain for environment variable `ROOT_DOMAIN`.

1. Request & validate a certificate from the [ACM Console](https://us-east-1.console.aws.amazon.com/acm) that covers domains `<ROOT_DOMAIN>` and `*.<ROOT_DOMAIN>`. Enter the certificate ARN for environment variable `CERTIFICATE_ARN`.

1. Choose an API subdomain (e.g. `api`). All of your APIs will be exposed at `<API_SUBDOMAIN>.<ROOT_DOMAIN>`. **Double-check your [Route 53](https://us-east-1.console.aws.amazon.com/route53) zone file to make sure it isn't already assigned to some other service!** Enter your API subdomain for environment variable `API_SUBDOMAIN`.

1. Choose your [API Version](#environments-api-versions-stages--stacks) (e.g. `v0`). Enter the appropriate value for environment variable `API_VERSION`.

   In the future, this value will be pulled directly from the package version at deploy time. ([#10](https://github.com/karmaniverous/aws-api-template/issues/10))

1. Choose an authorization subdomain token (e.g. `auth`). Your authorization endpoints will be at `<AUTH_SUBDOMAIN_TOKEN>-<API_VERSION>[-<non-prod environment>].<ROOT_DOMAIN>`. **Double-check your [Route 53](https://us-east-1.console.aws.amazon.com/route53) zone file to make sure these aren't already assigned to some other service!** Enter your authorization subdomain token for environment variable `AUTH_SUBDOMAIN_TOKEN`.

1. The template assumes your Cognito Authorization UI will be integrated with a web application at `ROOT_DOMAIN` or at subdomains representing either a `prod` or a `preview` front-end environment (e.g. my [Next.js Template](https://github.com/karmaniverous/nextjs-template)).

   Enter appropriate subdomains for environment variables `WEB_SUBDOMAIN_PROD` and `WEB_SUBDOMAIN_PREVIEW`.

   For local testing, enter the web application's `localhost` port for environment variable `WEB_LOCALHOST_PORT`.

   If you're using a different front-end application, you'll have to edit [`serverless.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/serverless.yml) accordingly. You can leave this alone for now and the application will still deploy properly!

1. Run the following command to create an API Gateway custom domain at `<API_SUBDOMAIN>.<ROOT_DOMAIN>`.

   ```bash
   npx getdotenv -- sls create_domain
   ```

That's it. You're now all set to deploy to any Stage at AWS!

### Manual Deployment

Run the following command to deploy a [Stage](#environments-api-versions-stages--stacks) to [Environment](#environments-api-versions-stages--stacks) `<env>` (e.g. `dev`, `test`, or `prod`) for the configured [API Version](<(#environments-api-versions-stages--stacks)>):

```bash
npm run deploy -- -e <env token>]
```

If this is your first time, try deploying to the `dev` Environment. In this case, you can just run

```bash
npm run deploy
```

Deployments take a few minutes and are differential, so the first one to any Stack will take the longest.

Once your deployment succeeds, your console will display a list of available resources. For example:

```text
endpoints:
  GET - https://3akn0sj501.execute-api.us-east-1.amazonaws.com/dev/hello
  GET - https://3akn0sj501.execute-api.us-east-1.amazonaws.com/dev/secure-hello
functions:
  hello: aws-api-template-dev-hello (15 MB)
  secure-hello: aws-api-template-dev-secure-hello (15 MB)
Serverless Domain Manager:
  Domain Name: aws-api-template.karmanivero.us
  Target Domain: d2jf3xe1z71uro.cloudfront.net
  Hosted Zone Id: Z2FDTNDATAQYW2
```

The public base path of your API will depend on your settings, but will look like:

```
https://<API_SUBDOMAIN>.<ROOT_DOMAIN>/<API_VERSION>[-<non-prod environment>].<ROOT_DOMAIN>
```

To test your deployment, open your hello-world public endpoint in a browser, e.g. [`https://aws-api-template.karmanivero.us/v0-dev/hello`](https://aws-api-template.karmanivero.us/v0-dev/hello). You should see the following content:

```json
{
  "message": "Hello world!"
}
```

### Automated Deployment

An [AWS CodePipeline](https://aws.amazon.com/codepipeline/) watches a code repository branch (e.g. at GitHub). When it detects a change, it...

1. Imports the code to an [AWS S3](https://aws.amazon.com/s3/) bucket.
1. Launches an [AWS CodeBuild](https://aws.amazon.com/codebuild/) project that...
   1. Allocates a virtual machine & imports the code repository.
   1. Loads all supporting applications (e.g. the Serverless Framework) and project dependencies.
   1. Deploys the [Stage](#environments-api-versions-stages--stacks) defined by `API_VERSION` and the configured [Environment](#environments-api-versions-stages--stacks), _exactly as you would do from your desktop!_
   1. Preserves the logs and deallocates the virtual machine.

Each CodePipeline is configured to deploy to the Environment corresponding to the branch it watches. For example, the CodePipeline watching the `main` branch may be configured to deploy to the `prod` Environment. On deployment, the CodePipeline will deploy to the Stack corresponding to its designated Environment and its current API Version. If that Stack does not yet exist, the deployment process will create it.

All related CodePipelines share the same CodeBuild project. Each CodePipeline is configured with environment variables that it injects into the CodeBuild project on deployment. These are the environment designator and environment secrets contained in each `.env.<ENV>.local` file, which are blocked by [`.gitignore`](https://github.com/karmaniverous/aws-api-template/blob/main/.gitignore) and are never pushed to the code repository.

The CodeBuild process is driven by [`buildspec.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/buildspec.yml).

The following sections describe how to set up your CodePipelines at AWS. If you are plugging into an ongoing project, these should already be in place.

#### Creating Your First CodePipeline

Your first pipeline in this project is significant because you will also need to create the associated CodeBuild project and maybe also associated service roles. Let's tackle all of this in reverse order.

##### Create a CodeBuild Service Role

When you create your CodeBuild project, you have the option of allowing AWS to create & manage associated a service role for you. This can get complicated fast, because you have to trust that AWS will be smart enough to figure out what policies to assign to this role as your project requirements change. If it isn't, your deployments will fail in ways that are hard to troubleshoot.

Since all of this is purely internal to your AWS account, I suggest you create AWS service role `codebuild-service` for the CodeBuild use case. Assign the `AdministratorAccess` policy.

Real AWS administrators are probably breaking out into hives right now, but you'll thank me.

##### Create Your Pipeline

If you are building this from scratch, you are probably operating from your `main` branch, which you will want to associate with your `prod` [Stage](#environments-api-versions-stages--stacks). Let's proceed on that assumption.

**Before you begin...** Your pipeline will pull your repo and start a build as soon as it is created! Commit all of your local changes and push your commits to GitHub.

From your [Pipelines Console](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines), create a new pipeline.

**Step 1: Choose Pipeline Settings**

1. Give the pipeline a name that reflects both the service name and the target [Environment](#environments-api-versions-stages--stacks): `<SERVICE_NAME>-<ENV>`

1. We will allow CodePipeline to create a service role for us, and then we will use it in all future pipelines. The arguments against this for CodeBuild don't apply here because the CodePipeline role will always need the _same_ access. Call the new role `codepipeline-service`.

1. Under Advanced Settings, choose all defaults.

**Step 2: Add Source Stage**

This is where CodePipeline gets its code. The instructions below assume you use GitHub.

1. Choose a provider. If you use GitHub, choose _GitHub (Version 2)_. Otherwise, work it out.

1. Choose an existing GitHub connection or create a new one. Note that if your code is in an Organization repo, Connections are Organization-specific.

1. Choose your repository & branch. In this example we are using `main`.

1. Leave any other defaults & click _Next_.

**Step 3: Add Build Stage**

1. Choose _AWS CodeBuild_ as your build provider.

1. Since this is our first pipeline for this project, we will need to create a CodeBuild project. Click the _Create Project_ button. A popup will appear and take you to a build project creation dialogue. In this dialogue...

   1. Choose a project name that will be consistent across the project: `<SERVICE_NAME>`

   1. Use a Managed Image with the following settings:

      - Operating System: _Ubuntu_
      - Runtime: _Standard_
      - Image: _highest version, currently `aws/codebuild/standard:6.0`_

   1. Use an existing service role and choose the `codebuild-service` role we created earlier.

   1. At the bottom of the dialogue, click _Continue to CodePipeline_.

1. Back in the CodePipeline window, use the _Add environment variable_ button to add the contents of `.env.<ENV>.local`. **These variable names & values are case-sensitive!**

   Since this is your first time through, you probably haven't populated any OAUTH client secrets (e.g. `GOOGLE_CLIENT_SECRET`). If so, just add those variables as placeholders. **Note that they may NOT be blank!** If undefined, many of these variables must be populated with `*`. See the `.env.<ENV>.local` file for specifics.

1. Click _Next_.

**Step 4: Add Deploy Stage**

We're using the Serverless Framework to deploy from [`buildspec.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/buildspec.yml), so click the _Skip deploy stage_ button & confirm.

**Step 5: Review**

Review your settings & click _Create pipeline_. The new pipeline will immediately display a UI, pull your repo, and commence a build.

#### Creating Your Next Pipeline

Once you have an operating CodePipeline, creating the next one is easy! Just follow these steps:

1. You can deploy to any [Environment](#environments-api-versions-stages--stacks) from any branch, but when you create a pipeline you have to choose a specific source branch tand target Environment. Make sure the Environment you choose has corresponsing `.env.<ENV>` & `.env.<ENV>.local` files in your repository with environment variables correctly populated.

1. Create the new branch (e.g. `dev` or `test`) and push it to your remote repository.

1. Choose an existing pipeline from the same project on your [Pipelines Console](https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines), click into it, and click the _Clone pipeline_ button.

1. Choose a pipeline name that follows your established naming pattern: `<SERVICE_NAME>-<ENV>`

1. Choose an existing service role and pick the CodePipeline service role we created earlier (e.g. `codepipeline-service`).

1. Choose all other defaults and click the _Clone_ button.

1. Once the pipeline is created, click the _Edit_ button.

1. Edit the _Source_ pipeline stage, edit the _Source_ action, and change the _Branch name_ to your selected repo branch. Click _Done_ on the popup and the _Source_ pipeline stage.

1. Edit the _Build_ pipeline stage, edit the _Build_ action, and change the environment variable values to match the `.env.<ENV>.local` file corresponding to the new pipeline's target Environment. Click _Done_ on the popup and the _Build_ pipeline stage.

1. Click the _Save_ button to save the pipeline changes and then click the _Release change_ button to run the pipeline in its new configuration. It will pull your code from the new branch and commence a build.

#### Creating a Maintenance Pipeline

The next deployment following any [API Version](#environments-api-versions-stages--stacks) upgrade will create a new [Stack](#environments-api-versions-stages--stacks) reflecting the new API Version. The old stack will still be running: it will simply no longer receive new deployments.

You may wish to continue maintenance development in the old major version to support existing users who have not yet migrated to the new version.

To accomplish this, follow these steps:

1. [release-it](https://github.com/release-it/release-it) creates a GitHub tag at each release. Execute the following commands to pull a `dev` branch from that tag and then push it back to GitHub:

   ```
   git checkout <highest previous major version tag> -b v<previous major version>-dev
   # Ex: git checkout 0.1.2 -b v0-dev

   git push
   ```

   If the previous major version were `0.1.2`, this will have created new branch `v0-dev` at that tag.

1. Follow the instructions in [Creating Your Next Pipeline](#creating-your-next-pipeline) to clone a new CodePipeline and attach it to this branch. Any new deployments on this pipeline will deploy into the previous API Version's existing `dev` Stack.

1. Repeat the previous steps for your other Environments (e.g. `test` and `prod`).

You can continue to release new versions within this major version. Just don't advance this branch to a new major version, or you will overwrite your existing next-version stacks!

{: .notice--primary}

**TODO**<br><br>
Figure out how to prevent this from happening ([#11](https://github.com/karmaniverous/aws-api-template/issues/11)).

### Deleting a Stack

Some [Stack](#environments-api-versions-stages--stacks) resources have properties that can only be set at create time (e.g. [Cognito User Pool AliasAttributes](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cognito-userpool.html#cfn-cognito-userpool-aliasattributes)). If you wish to change such a property, and if you will not unduly impact current users, the easiest option by far is to delete the entire stack and recreate it with the next deployment.

As you change your Stack configuration during the development process, your Stack might also simply become unstable and stop accepting new deployments. Deleting the Stack is often an efficient way to recover from a configuration error and get things moving again.

**Use with extreme caution!** Stack deletion will also eliminate all user accounts in the related user pool!

To delete a Stack, follow these instructions:

1. Find your API subdomain at [API Gateway Custom Domains](https://us-east-1.console.aws.amazon.com/apigateway/main/publish/domain-names). Delete all API mappings related to the Stack.

1. Find the Stack in the [CloudFormation console](https://us-east-1.console.aws.amazon.com/cloudformation) and click through to the Stack detail.

1. Under the Resources tab, find any S3 Buckets and delete their contents.

1. Delete the Stack.

{: .notice--info}

**If you intend to redeploy this stack,** you may need to wait about 20 min for the CNAME records of the deleted stack to flush from DNS. If you redeploy too soon, you may see an error indicaing a DNS conflict.

{: .notice--primary}

**TODO**<br><br>
Automate this process ([#10](https://github.com/karmaniverous/aws-api-template/issues/10)).

## Some Thoughts About DevOps

It's your project. Do what you want! But if you're interested, here's a rational way to go about this...

- During the development process you're probably working in some feature branch & trying run your project locally. The corresponding command in our case is

  ```
  npm run offline
  ```

  When this fails, you iterate until it doesn't.

- Local builds only take you so far, and there are a ton of AWS services that can only exist remotely. So your next step is to try a remote `dev` build using

  ```
  npm run deploy
  ```

  If you've [set up your CodePipelines](#automated-deployment), the same thing will happen when you merge your feature branch with the `dev` branch.

- This remote build will often fail catastrophically and require you to [delete the `dev` stack](#deleting-a-stack) & start over. Once it doesn't—and assuming all your other tests pass—you can [create a prerelease](#release-management) and deploy the stable build to your `test` Stack for integration testing by merging your `dev` branch into `test`.

- Once your integration tests pass, you can [create a release](#release-management) and deploy the new feature into production by merging your `test` branch into `main`.

- If your new release is a major release, [create a maintenance pipeline](#creating-a-maintenance-pipeline) to support maintenance & backward compatibility for the previous major release.

## Authentication

User authentication is provided by AWS Cognito.

By default, the setup supports sign up & sign in via the Cognito hosted UI. When a user signs up with an email/password combination, AWS will send an email with a code to confirm the email.

The template supports the Google federated identity provider but it is disabled by default. See [Add Google Authentication](#add-google-authentication) below for more info.

### Backwards Compatibility

An [API Version](#environments-api-versions-stages--stacks) upgrade generates a new [Stack](#environments-api-versions-stages--stacks) that includes an independent Cognito User Pool on a unique set of endpoints. You may not want this! If there have been no breaking changes to your User Pool, you probably want to retain the user accounts created in the previous API Version.

To accomplish this, simply add the relevant Cognito User Pool ARN to environment variable `COGNITO_USER_POOL_ARN` in file `.env.<ENV>`.

### Add Google Authentication

AWS Cognito supports a wide variety of federated identity providers. Google is the only one currently supported by this template.

To enable Google authentication, follow these steps:

1. Follow the Google-related instructions at [this page](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-social-idp.html) to set up a Google OAUTH2 application. Use the following Stage-spacific settings:

   - _Authorized Javascript Origins:_ `<AUTH_SUBDOMAIN_TOKEN>-<API_VERSION>[-<non-prod ENV>].<ROOT_DOMAIN>`
   - _Authorized Redirect URIs:_ `<AUTH_SUBDOMAIN_TOKEN>-<API_VERSION>[-<non-prod ENV>].<ROOT_DOMAIN>/oauth2/idpresponse`

1. Add the resulting Client ID to environment variable `GOOGLE_CLIENT_ID` in file `.env.<ENV>`

1. Add the resulting Client Secret to environment variable `GOOGLE_CLIENT_SECRET` in file `.env.<ENV>.local`

1. Add the same Client Secret to environment variable `GOOGLE_CLIENT_SECRET` in the relevant CodePipeline configuration.

### Add Support For Other Federated Identity Providers

If you configure new federated identity providers following the patterns in this template, then they will only be ENABLED when the relevant client ID, client secret, etc. are added to the project configuration.

[This page](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-social-idp.html) contains instructions for setting up Cognito-compatible OAUTH2 apps at Facebook, Amazon, Google, and Apple.

It is also possible to set up custom OpenId Connect (OIDC) providers which presumably will account for Twitter etc. There is an AWS reference [here](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-oidc-idp.html).

Once you understand the requirements of your new identity provider, follow these steps:

1. Add the relevant new non-secret environment variables (e.g. `GOOGLE_CLIENT_ID) to all `.env.<ENV>`files. If no value is available, set the value to`\*`

1. Add the relevant new secret environment variables (e.g. `GOOGLE_CLIENT_SECRET`) to all `.env.<ENV>.local` files. If no value is available, set the value to `*`.

1. Add the same secret environment variables (e.g. `GOOGLE_CLIENT_SECRET`) to all `.env.<ENV>.local.template` files. Set all values to `*`.

1. Add the same secret environment variables (e.g. `GOOGLE_CLIENT_SECRET`) to all CodePipelines. If no value is available, set the value to `*`.

1. Add a new identity provider Condition in [`serverless.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/serverless.yml). Its purpose is to test whether the necessary environment variables have been populated to support enabling the identity provider.

   The new condition should be similar to `CreateIdentityProviderGoogle` below and reference the new provider's environment variables:

   ```yml
   resources:
   Conditions:
     CreateUserPool: !Equals ["${env:COGNITO_USER_POOL_ARN}", "*"]
     CreateIdentityProviderGoogle: !And
       - !Condition CreateUserPool
       - !Not
       - !Or
         - !Equals ["${env:GOOGLE_CLIENT_ID}", "*"]
         - !Equals ["${env:GOOGLE_CLIENT_SECRET}", "*"]
   ```

1. Create a new User Pool Identity Provider in [`serverless.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/serverless.yml). It should be similar to the one below but meet the requirements of the new provider:

   ```yml
   UserPoolIdentityProviderGoogle:
     Type: AWS::Cognito::UserPoolIdentityProvider
     Condition: CreateIdentityProviderGoogle
     Properties:
       UserPoolId: !Ref UserPool
       ProviderName: Google
       ProviderDetails:
         client_id: "${env:GOOGLE_CLIENT_ID}"
         client_secret: "${env:GOOGLE_CLIENT_SECRET}"
         authorize_scopes: "profile email openid"
       ProviderType: Google
       AttributeMapping:
         email: email
   ```

1. Conditionally reference the new User Pool Identity Provider in [`serverless.yml`](https://github.com/karmaniverous/aws-api-template/blob/main/serverless.yml) under `UserClient.Properties.SupportedIdentityProviders`. Reference the new Condition and the new User Pool Identity Provider, similar to how it is done with Google below:

   ```yml
   SupportedIdentityProviders:
     - COGNITO
     - !If
       - CreateIdentityProviderGoogle
       - Google
       - Ref: AWS::NoValue
   ```

Test your setup by attempting a remote deployment. If your deployment succeeds, you will your provider configured under _Federated identity provider sign-in_ on the relevant User Pool's _Sign-in experience_ tab at your [AWS Cognito Console](https://us-east-1.console.aws.amazon.com/cognito).

## Endpoints

| Endpoint                     | Pattern                                                                                  | `dev` example                                         | `prod` example                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------- |
| Base API Path                | `https://<API_SUBDOMAIN>.<ROOT_DOMAIN>/API_VERSION[-{non-prod ENV}]`                     | `https://api.mydomain.com/v0-dev/hello`               | `https://api.mydomain.com/v0/hello`               |
| Authorized Javascript Origin | `<AUTH_SUBDOMAIN_TOKEN>-<API_VERSION>[-<non-prod ENV>].<ROOT_DOMAIN>`                    | `https://auth-v0-dev.mydomain.com`                    | `https://auth-v0.mydomain.com`                    |
| Authorized Redirect          | `<AUTH_SUBDOMAIN_TOKEN>-<API_VERSION>[-<non-prod ENV>].<ROOT_DOMAIN>/oauth2/idpresponse` | `https://auth-v0-dev.mydomain.com/oauth2/idpresponse` | `https://auth-v0.mydomain.com/oauth2/idpresponse` |

## Issues

### Secure Certificate

Normally we would specify a certificate on `karmanivero.us` and `*.karmanivero.us` using the [`serverless-certificate-creator`](https://www.serverless.com/plugins/serverless-certificate-creator) plugin. The required entry in `serverless.yaml` would look like

```yaml
custom:
  customCertificate:
    certificateName: "karmanivero.us"
    hostedZoneNames: "karmanivero.us."
    subjectAlternativeNames:
      - "*.karmanivero.us"
```

The alternative name is currently creating an issue with this. It looks like the plugin is trying to submit duplicate validation records to the zone file, resulting in an error. See [this pull request](https://github.com/schwamster/serverless-certificate-creator/pull/55) for more info.

Meanwhile, let's create & verify the certificate manually using the [ACM Console](https://us-east-1.console.aws.amazon.com/acm/home#/certificates/list) and then reference the certificate by ARN in `serverless.yaml`.

### `serverless.yml` Validation

There is a conditional function on private API endpoints that selects authentication either by the Cognito User Pool created on the stack or by the ARN passed in via the `COGNITO_USER_POOL_ARN` environment variable. It looks like this:

```yml
secure-hello:
  handler: api/secure/hello.get
  description: GET /secure-hello
  events:
    - http:
        path: secure-hello
        method: get
        cors: true
        authorizer:
        name: UserPoolAuthorizer
        type: COGNITO_USER_POOLS
        # *** CONDITIONAL ***
        arn: !If
          - CreateUserPool
          - !GetAtt UserPool.Arn
          - ${env:COGNITO_USER_POOL_ARN}
        # *******************
        claims:
          - email
```

This is perfectly valid and works just fine, but the Serverless Framework issues the following warning on deployment:

```text
Warning: Invalid configuration encountered
  at 'functions.secure-hello.events.0.http.authorizer.arn': unsupported object format

Learn more about configuration validation here: http://slss.io/configuration-validation
```

You can ignore this warning and track the issue [here](https://github.com/serverless/serverless/issues/11611).

### User Accounts

As of now, users can create multiple accounts with the same email, but different user names. ([#12](https://github.com/karmaniverous/aws-api-template/issues/12))

Federated identities (i.e. social logins) with the same email also generate new accounts. ([#13](https://github.com/karmaniverous/aws-api-template/issues/13))
