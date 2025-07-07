---
title: "Design as Code: A Frictionless Low-Level Design Pipeline"
excerpt: "A frictionless low-level design process that integrates with your dev cycle and puts design artifacts at a peer level with the rest of your codebase."
header:
  og_image: /assets/collections/project-governance/design-as-code-banner.jpg
  overlay_image: /assets/collections/project-governance/design-as-code-banner.jpg
  teaser: /assets/collections/project-governance/design-as-code-square.jpg
permalink: /toolkits/project-governance/design-as-code-a-frictionless-low-level-design-pipeline/
redirect_from:
  - /blog/design-as-code-a-frictionless-low-level-design-pipeline/
swagger: true
related: true
tags:
  - agile
  - design
  - documentation
  - project-governance
  - toolkits
---

<figure class="align-left drop-image">
    <img src="/assets/collections/project-governance/design-as-code-square.jpg">
</figure>

A client of mine is building a web application that features a complex data model, supported by a collection of APIs. My client has asked me to review their design & development processes and help them achieve a better outcome for the project.

As part of the discovery process, I asked the tech lead to show me a design artifact describing one of their APIs.

## An Example By Negation

For the uninitiated: an [Application Programming Interface](https://en.wikipedia.org/wiki/API) (API) is a set of rules and protocols that allow one component of a software application to interact with other internal components or with another application. In the hierarchy of abstractions leading from an application's business requirements to its implementation in code, an API design sits right in the middle:

{% include figure image_path="/assets/diagrams/design-abstraction-hierarchy-api.png" caption="_API designs in a typical application design hierarchy._" popup=true %}

Back to my story...

It took a couple of minutes to clarify my request with my client's tech lead. But after some internal discussion, one of his developers shared his screen and brought up the [OpenAPI specification](https://swagger.io/specification/) for the API in question.

If you've never seen an OpenAPI (a.k.a. Swagger) spec, here's an example:

```json
{
  "swagger": "2.0",
  "info": {
    "version": "1.0.0",
    "title": "Swagger Petstore",
    "description": "A sample API that uses a petstore as an example to demonstrate features in the swagger-2.0 specification",
    "termsOfService": "http://swagger.io/terms/",
    "contact": {
      "name": "Swagger API Team"
    },
    "license": {
      "name": "MIT"
    }
  },
  "host": "petstore.swagger.io",
  "basePath": "/api",
  "schemes": [
    "http"
  ],
  "consumes": [
    "application/json"
  ],
  "produces": [
    "application/json"
  ],
  "paths": {
    "/pets": {
      "get": {
        "description": "Returns all pets from the system that the user has access to",
        "operationId": "findPets",
        "produces": [
          "application/json",
          "application/xml",
          "text/xml",
          "text/html"
        ],
        ...
      },
      ...
    },
    ...
  },
  ...
}
```

... only theirs went on vertically for **about a thousand lines**.

> A thousand-line JSON Swagger specification is not an API design artifact!

In this article, we will discuss:

- exactly _why_ a thousand-line JSON Swagger specification is not an API design artifact, and

- what design artifacts describing APIs and other technical abstractions actually _should_ look like (at least if you want to get any use out of them), and

- how to produce, maintain, and use these low-level design artifacts efficiently to achieve a better outcome for your project.

## Design Artifacts Redux

In a [previous article](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process) I went into some detail regarding the theory and application of artifacts in a "mechanical" design process.

I described a [_complete design artifact_](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process#completeness) as a matched pair of work items:

- a _drawing_ that represents an aspect of design visually, trading off detail for clarity, and

- a _document_ that unpacks the the drawing in detail and identifies concerns to be addressed at lower levels of design.

Beyond providing a useful balance of clarity and detail, this double presentation constitutes a source of [_internal consistency_](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process#internal-consistency) that can be used to validate the design artifact against error.

{% include figure image_path="/assets/diagrams/design-internal-consistency.png" caption="_Design artifact with internal consistency._" %}

I also described an [_external consistency_](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process#external-consistency) between design artifacts at different levels of abstraction:

- Documents at a **higher level** of abstraction raise issues that should be accounted for by some drawing at a lower level of abstraction (or, at the lowest level, by the actual implementation).

- Every element in a **lower-level** drawing should account for some issue raised in a higher-level document, and every such issue should be accounted for.

{% include figure image_path="/assets/diagrams/design-external-consistency.png" caption="_Design artifacts with external consistency._" %}

Taken together, these two forms of design consistency lend themselves to [an iterative process](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process#iterate) that should be familiar to every Agile practitioner: grooming the **_design backlog_**.

### An Aside: The Design Backlog

[Backlog Grooming](/toolkits/project-governance/a-modern-agile-project-manifesto#backlog-grooming) is an essential part of the Agile cycle.

It's also a lot of work! Most projects schedule backlog grooming weekly at least. It's a 1-2 hour meeting where participants pull a pile of issues out of the project backlog and...

- **Assign every Issue to the correct project Phase.** Is it in scope for the current Phase? If not, let’s drop it in the right bucket & forget about it for now.

- **Size every Issue.** How much work will an Issue take to resolve?

- **Prioritize every Issue.** Which are the most important?

- **Sequence every Issue.** What order must they be completed in to account for dependency?

- **Identify blockers.** What’s standing in the way of completing this Issue?

- **Decompose complex Issues.** How can we break big Issues into bite-sized pieces?

- **Fix the easy stuff on the spot, and create new tickets to account for the hard stuff**, so somebody can be assigned to do the work required to flesh those tickets out.

With all that already in play, the last thing any Scrum Master wants is to find he has to groom _another_ backlog.

**Not the idea!** _Everybody take a breath._

> In a healthy Agile project, there is only ever one backlog!

This only makes sense. You have _one_ team, focused on producing _one_ body of work. Your backlog is simply the pile of stuff that hasn't been done yet. As we saw above, an essential part of the backlog grooming job is breaking the complex bits of stuff down into simpler stuff, and a single backlog makes it _much_ easer to [maintain a trace](/toolkits/project-governance/a-modern-agile-project-manifesto#issue-links) on which bits came from which other bits. _Life is hard enough._

And yet... there are different kinds of stuff, aren't there?

In [_A Modern Agile Project Manifesto_](/toolkits/project-governance/a-modern-agile-project-manifesto) I proposed a breakdown of the various [issue types](/toolkits/project-governance/a-modern-agile-project-manifesto#jira-issues) available to a project. I did this within the context of a [Jira](https://www.atlassian.com/software/jira) implementation, but that's really beside the point.

The lessons here are:

- **You should differentiate your backlog into different issue types.** Odds are you already do this, but those distinctions should be explicit and well-understood by the team. In other words: _write them down!_

- **Design issues are a distinct type.** In the article linked above, these would fall under the _Question_ issue type, but it doesn't matter what you call them as long as they get their own, focused attention. Which leads us to...

- **Groom each issue type separately!** Well... sort of.

That last point is the important one. We've beaten the design horse half to death in recent weeks, and if you've noticed any consistent theme it should be this one:

> Design is not the same thing as implementation.

Humans are bad at multi-tasking, so _stop_. If you have a two-hour backlog grooming meeting every week, simply splitting it up into two meetings—one focused on design and the other on implementation—will make _both_ meetings more focused, more productive, and _much_ more effective in terms of project outcome!

Stories & Bugs are both arguably about implementation. What about Epics & Tasks?

**Epics** are admittedly weird. Let's revisit the definition of an Epic from [_A Modern Agile Project Manifesto_](/toolkits/project-governance/a-modern-agile-project-manifesto):

> An Epic represents a very high-level grouping of other issues. Other issues should come out of the Backlog and be resolved in a single Sprint. If they can't be, then they should be broken down until they can.<br><br>
> Epics endure across several Sprints, and generally represent a project phase with respect to a broad area of functionality. Epics can be organized into a Roadmap expressing the Project's long-term plan.<br><br>
> Most Epics are never resolved, because the functionality they represent is always subject to future development. If not, an Epic is resolved when all the issues it contains are either resolved or moved to another Epic.

This suggests that an Epic can be used as a container of both design-related issues (Questions) _and_ implementation-related issues (Stories & Bugs). This would place the Epic firmly on the _design_ end of the project hierarchy, and so Epics should be groomed in the same meeting as other design-related issues.

If you define Epics differently, you might come to a different conclusion. The important thing is to _have the definition that leads to the conclusion_.

**Tasks** fall into a different category. They tend to be procedural in nature, more focused on project management than product development. I like to isolate task grooming from design and implementation, both to keep the focus tight and to make sure the actual business of the project gets the attention it deserves.

A 30-min weekly grooming of the task list is usually sufficient for my projects.

One more very important point:

> Backlog grooming is not sprint planning!

When grooming your backlog, focus is a virtue. When planning a sprint, you need to be able to see the _whole_ backlog, so work can be assigned _across_ the different issue types! If you ever wondered just why those two tasks are handled in separate meetings, now you know.

## A Thousand Lines

By now it should be crystal clear just why that thousand-line Swagger specification I described [above](#an-example-by-negation) is not an API design artifact: because, even under the most generous assumptions, **it's not quite half of one!**

Within the framework we've developed here, a complete design artifact has two parts:

- **a drawing** that represents an aspect of design visually, trading off detail for clarity, and

- **a document** that unpacks the drawing in detail and identifies concerns to be addressed at lower levels of design.

Clearly the Swagger specification does not include a drawing.

More generally, as it represents a low-level technical abstraction (an API), we would expect this design artifact to articulate its relationships to:

- **implemented code** - Presumably each endpoint in the Swagger specification is implemented somewhere in the codebase. _But where?_

- **other technical abstractions** - Does this API call other APIs? Is it called by other APIs? Does it interact with any external services? Does it leverage shared project resources (e.g. classes, libraries, etc.) that encapsulate common functionality? _The Swagger specification doesn't say._

It is fair to observe that _none of these details actually belong in a Swagger specification!_ A Swagger specification is a _contract_ between the API and its consumers, and as such it should be focused on the _interface_ of the API, not its implementation nor its relationship with those consumers.

**Being a design artifact is not what a Swagger specification is actually _for_.**

You might argue that a Swagger spec should be included as part of a design artifact's document, but consider these points:

- A Swagger spec has a very persnickety format that is best generated by a machine.

- Even a very simple Swagger spec is very hard for human beings to read.

This is why Swagger specs are usually generated from code comments or other machine-readable sources. They are not intended to written by humans, and they are _certainly_ not intended to be _read_ by humans.

You might also argue that a simple HTML plugin can turn a complex Swagger specification document into a very pretty documentation UI like this one:

{% include figure image_path="/assets/collections/project-governance/design-swagger-ui.jpg" caption="_Swagger document UI plugin._" popup=true %}

_That's as good as a design drawing, right?_ **No, it isn't.**

As a piece of documentation, this widget offers a _huge_ improvement over the raw Swagger spec! But the information it presents is still strictly limited to the interface contract laid out in the underlying JSON document. It tells us nothing about how this API relates to other technical abstractions in the project, nor to the project's implementation in code.

The Swagger UI is no more a _design drawing_ than is the Swagger spec itself.

**I do in fact include the Swagger UI in every API design artifact!** But not as a _drawing_. It's part of the _document_ associated with the design artifact, and I include it because it is an attractive and efficient way to document the relationship between the design artifact and its implementation.
{: .notice--info}

So there's the big reveal regarding the thousand-line Swagger specification presented as a design artifact by my client's tech team: it was almost certainly generated with some kind of automation from code that had _already_ been implemented, and was **no more an deliberate artifact of design than I am my own grandfather**.

> For many teams, the first step toward improving their design process is admitting they don't have one.

## Low-Level Design For Real

In another article we explored [an example of high-level design](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process/#a-redacted-example) from the [VeteranCrowd](https://veterancrowd.com) project. We immediately had to deal with the following reality:

> Really useful design artifacts are specific to the problem they’re solving, and proprietary to the organization that’s solving it!

In that case, it meant we had to do some pretty heavy redaction in order to protect the client's interests.

In this case, we're a bit more fortunate: because VeteranCrowd's [microservice architecture](https://en.wikipedia.org/wiki/Microservices) is extremely distributed, some of its services are considerably more generic than others. For our example—and with VeteranCrowd's blessing—we will choose the `api-message` service, which their application uses to compose and send templated messages to users and is about as generic as it gets.

_No secrets here!_

The `api-message` service is implemented as an [AWS CloudFormation](https://aws.amazon.com/cloudformation/) stack using the [Serverless Framework](https://www.serverless.com/) (a design pattern addressed in detail in higher-level design artifacts) Despite its hyper-generic nature, the service offers the following interesting features, all of which are detailed in the design artifact below, both in the drawing and its associated document:

- An [Amazon API Gateway](https://aws.amazon.com/api-gateway/)-mediated RESTful API that exposes public endpoints, private endpoints secured with API keys (for inter-API communication), and another set of private endpoints secured by [Amazon Cognito](https://aws.amazon.com/pm/cognito) that are used by the application's front end.

- A layer of [AWS Lambda](https://aws.amazon.com/lambda/) functions that validate & conform API requests and generate responses.

- An AWS DynamoDB table for data storage, with entity management and scalability supported by [`entity-manager`](https://github.com/karmaniverous/entity-manager).

- Message delivery via email & SMS provided by [Amazon Pinpoint](https://aws.amazon.com/pinpoint/).

The `api-message` service handles message-generating events raised by most of the other 15 services currently running in the VeteranCrowd back end, and it calls the `aws-template` service (not detailed here) in order to generate message content based on these events.

**Look for comments like these** on the design artifact below to learn more about specific features! Also look for links to [PlantUML](https://plantuml.com/) source code in diagram captions. We'll discuss PlantUML in a later section!
{: .notice--info}

> I'll also call out key principles like this.

One last point before we dive into the design artifact: **what I am showing you below is a preserved snapshot of a _live_ artifact!**

Since the project's requirements and implementation are in a constant state of flux, _all_ of our design artifacts (including this one!) will have points where they are incomplete, internally inconsistent, or out of sync with other design documents and the implementation. This is par for the course!

The purpose of the design process is create a structured framework where we can...

- **detect** these issues,

- **capture** them in our backlog,

- **prioritize** and **schedule** them in our grooming process, and

- **resolve** them in a way that brings our implementation fully in line with the requirements of our business.

That's the job! Let's get to it.

## `api-message` Design Artifact

**A design drawing should fit on a single page!** That means different things on different platforms. On this one, I had to link to a full-page view that permits zooming in a desktop browser. But no matter what: if you _can't_ fit your design drawing on a single page, that's a design smell! Raise your design to a higher level of abstraction, and create more detailed designs at a lower level.
{: .notice--info}

> If your design diagram is too big to handle, your design is too complex to implement.

{% include figure image_path="/assets/diagrams/design-api-message.png" caption="_`api-message` design • <a href='/assets/diagrams/design-api-message.png' target='_blank'>full-page view</a> • <a href='https://github.com/karmaniverous/blog/blob/main/diagrams/design-api-message.pu'>PlantUML Source</a>_" %}

**Use a consistent visual language across all design artifacts!** In VeteranCrowd API designs, API internals are inside the gray box. Inbound calls come from the left, and outbound calls go to the right wherever possible. Calls to other VC APIs present the same convention an abbreviated form (see the `api-template` call at the bottom). This makes it easy to orient to a design at a glance.
{: .notice--info}

> A consistent & coherent design language promotes a consistent & coherent implementation.

### Processes

**Sometimes a document needs a drawing!** The purpose of a design artifact's _document_ is to unpack the relationships & issues identified in its _drawing_. Where words are required, use them. When a diagram serves the purpose better, use a diagram instead! At the end of the day, the goal of the document is to complete the picture presented by the drawing in as clear a manner as possible.
{: .notice--info}

#### Message Posting

This process ends with the creation of Message records in the database, targeted at their respective recipients. Each record will feature a `targets` array indicating external delivery channels selected according to User preferences.

A separate process will pick up on these record insertions and route traffic to external channels as appropriate.

{% include figure image_path="/assets/diagrams/design-message-post.png" caption="_Message Posting sequence • <a href='/assets/diagrams/design-message-post.png' target='_blank'>full-page view</a> • <a href='https://github.com/karmaniverous/blog/blob/main/diagrams/design-message-post.pu'>PlantUML Source</a>_" %}

### Entities

**Relationships matter!** This API happens to manage a single entity in its data store. Otherwise this would be a great spot for an [Entity Relationship Diagram](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model). Still not a ridiculous idea, since Message entities relate to external entities in other services. I'll add a note to our [design backlog](#an-aside-the-design-backlog). See how that works? 🤣
{: .notice--info}

#### Message

| Property      | Category | Type                                                  | Notes                                                                                                                                               |
| ------------- | -------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[archived]`  | system   | `number`                                              | Archived timestamp in millis.                                                                                                                       |
| `[bodyHtml]`  | content  | `string`                                              | HTML message body: <br>- Email<br>- Application inbox                                                                                               |
| `[bodyText]`  | content  | `string`                                              | Text message body.<br>- Email                                                                                                                       |
| `created`     | system   | `number`                                              | Message created timestamp in millis.                                                                                                                |
| `[distId]`    | related  | `string`                                              | Id related to `distToken`. Not valid for `applicationManager`.                                                                                      |
| `[distToken]` | related  | `string`                                              | Message distribution token. Enum:<br>- `applicationManager`<br>- `beneficiaryManager`<br>- `groupManager`<br>- `groupMember`<br>- `merchantManager` |
| `messageId`   | related  | `string`                                              | Unique Message identifier.                                                                                                                          |
| `[read] `     | system   | `number`                                              | Read timestamp in millis.                                                                                                                           |
| `[subject]`   | content  | `string`                                              | Text message subject.<br>- Email<br>- Application inbox                                                                                             |
| `[summary]`   | string   | Text message summary.<br>- SMS<br>- Push notification |
| `targets`     | related  | `array`                                               | JSON object identifying specific delivery targets:<br>- `email`<br>- `sms`                                                                          |
| `templateId`  | related  | `string`                                              | Related Template identifier.                                                                                                                        |
| `updated`     | system   | `number`                                              | Message updated timestamp in millis.                                                                                                                |
| `updater`     | system   | `string`                                              | `userId` of last updater.                                                                                                                           |
| `userId`      | related  | `string`                                              | `userId` of recipient.                                                                                                                              |

### OpenAPI

As we discussed above, this service's OpenAPI spec is a technical artifact of implementation, not a design element. But putting it here makes it easy to check the [_external consistency_](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process#external-consistency) of the design with implementation during design reviews!
{: .notice--info}

> Good design supports efficient validation of internal and external consistency.

[JSON](https://api.veterancrowd.com/message-v0/doc/openapi)

{% include swagger.html url="https://api.veterancrowd.com/message-v0/doc/openapi?includeTags=Public~Cognito~Private" %}

## Design As Code

A basic principle of economics is that the more expensive anything is, the less of it you get. There's no such thing in life as a free lunch.

> As in life, so in design!

In [_Turning the Crank_](/toolkits/project-governance/turning-the-crank-design-as-a-mechanical-process), I showed you a few very fancy high-level design diagrams like this one:

{% include figure image_path="/assets/images/architecture-vc-business-process.png" caption="_VeteranCrowd core business process._" popup=true %}

I created this diagram and the other high-level artifacts in that article with [LucidChart](https://www.lucidchart.com/). LucidChart is a _great_ tool for creating all sorts of production-quality diagrams, and I highly recommend that you use it **_as seldom as you can possibly get away with!_**

### High Touch: High Cost

As we observed in that article, high-level design is generally a business-facing activity. The entities described in high-level design artifacts are business entities, and the language used is a business language. Most critically, the _audience_ for high-level design artifacts is generally a _business audience!_

A business audience is a different breed from a technical audience. They care about substance... but they tend to be _far_ more sensitive to presentation and production qualities than the average bunch of coders. Business audiences want to see pretty pictures that tell a story.

> The audience for high-level design artifacts is a business audience.

Moreover, we observed in that article that **in a healthy project, the design artifacts _are_ the project documentation.**

At a low level, this means that great design artifacts describing technical abstractions make great technical documentation for a technical audience. At a high level, your business-facing design artifacts are likely to find their way in front of senior leadership, business partners, and prospective investors.

For that kind of audience, a certain degree of polish is worth the effort. And the good news is that your high-level business processes and relationships will change _much_ more slowly than your low-level technical abstractions... so even though creating and editing fancy diagrams with tools like LucidChart is a pain in the behind, you won't need to do it _that_ often.

### Gearing for Speed

Low level design is _very_ different.

For one thing, there's a _lot_ more of it. In a typical project, you might have a dozen or so high-level design artifacts, but _hundreds_ of low-level design artifacts... none of which are of any real use to you if they fall very far out of sync.

Also, the farther you descend the ladder of abstraction, the more often things change. Nothing influences a design quite like another pair of eyeballs, and once you commit to visual design artifacts and regular design reviews, you're going to get a _lot_ of eyeballs on your work. Expect your design repository to evolve, and _fast!_

> Nothing influences a design quite like another pair of eyeballs.

But here's the good news: unlike high-level design artifacts, the audience for your low-level design artifacts is a _technical_ one. They will care far more about the _substance_ of your design than they will about its production qualities.

In fact, just the reverse is true. It's neat to see a design laid out in a diagram one can grasp at a glance... but to see that diagram change regularly in response to feedback is downright _impressive!_ It means your team can reliably catch mistakes in design review _before_ they hit the codebase, and before anybody has to burn the midnight oil coding the wrong components.

So your design process is in place, and your technical audience is chomping at the bit to iterate on your design artifacts... and **the entire program will come crashing to a halt** if you can't pump new design drawings out as fast as your team can review them.

> In low-level design, pretty is optional, but speed is essential!

So let's talk about your design pipeline.

### The Design Pipeline

Scratch a typical software project's design repository, and for every diagram you find you'll see a dozen spreadsheets and a hundred text-based documents.

And why not? Economics strikes again: **typing is easy and cheap, so you get a lot more of it**.

> If you want your team to produce as many design drawings as text documents, you need to make drawing as easy as typing.

What would that look like? An ideal pipeline for text-based drawings might look something like this:

- Most technical drawings are some combination of boxes, arrows, and containers. I should be able to compose a drawing like this in a text editor with a simple markup language.

- The markup language should natively handle commonly-understood symbology (like UML), but I should be able to extend it with domain-specific symbols as needed (like the AWS pallette in the [`api-message` design artifact](#api-message-design-artifact) above).

- I shouldn't have to concern myself with the layout of the drawing. The drawing tool should handle that for me.

- I should be able to render my drawing on the fly and export it to a variety of image formats.

- I should be able to store my drawing in a version-controlled repository and track changes to it over time.

- I should be able to link my drawing to other design artifacts, and to other artifacts in the project repository.

Sounds reasonable, right? That's [PlantUML](https://plantuml.com/) in a nutshell.

Here's an example. Remember this diagram?

{% include figure image_path="/assets/diagrams/design-external-consistency.png" caption="_Design artifacts with external consistency._" %}

Here's the PlantUML source code that generated it:

```text
@startuml design-external-consistency

allowmixing
skinparam componentStyle rectangle

package "High-Level Artifact" as artifactHigh #LightCyan {
  component "Drawing" as drawingHigh
  component "Document" as documentHigh

  drawingHigh <--> documentHigh
}

note bottom of documentHigh
  All issues raised
  are addressed in some
  lower-level drawing.
end note

package "Low-Level Artifacts" as artifactLow #Cyan {
  component "Drawing" as drawingLow
  component "Document" as documentLow

  drawingLow <--> documentLow
}

note top of drawingLow
  Every element addresses
  an issue raised in the
  higher-level document.
end note

documentHigh <-> drawingLow: "**//external consistency//**"

@enduml
```

If you use Visual Studio Code and GitHub, setting up a PlantUML pipeline is very straightforward:

1. [Install PlantUML locally](https://plantuml.com/starting) (you'll get faster rendering than with the remote service)

1. Install the [PlantUML extension](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml) for Visual Studio Code and add the extension to the recommended extensions for all your code repositories.

1. Standardize your repository structure for diagrams. I keep diagram source files in a `diagrams` directory at the root of the repository, and render all diagram image files to an `assets` directory. Memorialize these decisions in your standard repository settings.

1. Keep low level design artifacts in the same repository that implements them. Compose your design artifacts in Markdown, directly linking to rendered PlantUML images where needed.

1. Use a tool like [TypeDoc](https://typedoc.org/) to assemble your design artifacts along with related documentation into a browsable site that is updated automatically every time you run a build. If you use [Atlassian](https://www.atlassian.com/), [Confluence](https://www.atlassian.com/software/confluence) also works well for that purpose.

At the end of the day, the goal is to frame each of your design artifacts _once_. After the first render, the only changes you should ever have to make are substantive changes to design drawings and document content. The rendering pipeline should handle the rest.

What you have now is **_Design as Code:_** a frictionless design process that is fully integrated with your development cycle and places design artifacts at a peer level with the rest of your codebase.

## Conclusion

Design isn't just a template we pull off a shelf. It's the guarantee that the product you put into production is the one your business actually needs.

Design is a process that creates and maintains a chain of continuity down a ladder of abstraction that connects the needs of the business, expressed in business language and concepts, to the implementation of those needs in code.

Some of that code runs web servers, and some of it draws pictures. Choose your own adventure.

Design is _expressed_ in a set of complete and consistent design artifacts, and those artifacts have value. But it is the process that _creates_ those artifacts, and the disciplined team culture that runs the process, that are the true technical assets of the business.

In the end, every project team's goal is to deliver a great product that can serve today's business needs with room to spare for whatever tomorrow brings. That means doing some hard things, and design is one of them... unless it isn't.

After all, isn't that what engineers do? We make hard things like launching rockets and drawing design diagrams as easy as pushing buttons or... _typing_.

Do good work! 🚀
