---
title: A Modern Agile Project Manifesto
excerpt: A battle-tested, all-inclusive starting point for a modern Agile project.
header:
  og_image: /assets/images/agile-banner.jpg
  overlay_image: /assets/images/agile-banner.jpg
  teaser: /assets/images/iteration.png
permalink: /toolkits/project-governance/a-modern-agile-project-manifesto/
related: true
redirect_from:
  - /blog/a-modern-agile-project-manifesto/
tags:
  - agile
  - atlassian
  - project-governance
  - toolkits
---

<figure class="align-left drop-image">
    <img src="/assets/images/iteration.png">
</figure>

I've participated in and led a LOT of Agile projects. Some have been more Agile than others, and many of you know the pain behind THOSE words. 🤣

If you're an Agile practitioner, you know that the method _by design_ conforms to the team, so that no two teams practice the exact same flavor of Agile.

But it helps to have a solid starting point: a plan that covers all the bases, that you can run with out of the gate and then modify as your Sprint retrospectives roll in.

Even better if that plan reflects the tools you actually use, which in many large organizations and projects are going to be [Jira](https://www.atlassian.com/software/jira) and [Confluence](https://www.atlassian.com/software/confluence).

This is the intent of the document below. I've refined it over many projects at institutions large and small. It NEVER survives more than a few Sprints intact, but then that's the point, isn't it?

I hope it serves you well.

{: .notice--warning}

**Jira & Confluence are complex products!** My goal with this article is not to teach you how to configure Atlassian tools, but to articulate a solid starting point for your Agile process. Tweaking your tools is up to you. If you get stuck, [leave a comment](#disqus_thread) and I'll see if I can help.

## Agile @Karmaniverous

The purpose of this page is to lay out the rules of the game. **This is how we do Agile at Karmaniverous.**

We want a perfect fit between our tools and our practices. Jira and Confluence are the tools we use to manage the Agile process. Therefore: **how we use Jira & Confluence is how we do Agile.**

{: .notice--info}

**None of this is set in stone!** The Agile process is also subject to the Agile process. Over time, these rules WILL change as we converge to our own best way of doing business. The goal isn't to follow some arbitrary rule book. It's to find out what GOOD looks like, and then do that on purpose instead of by accident! See [Retrospective](#retrospective) below for more info.

## Agile in a Nutshell

The Agile Methodology is a recipe for accomplishing something. It can be used to...

- build a product
- manage a business
- operate a factory

... and just about anything else. If something big and complicated needs to be done, Agile can help.

It’s easiest to understand Agile within the context of its opposite: traditional project management. Consider four basic phases of any project:

0. **CONCEPT:** Have an idea.
1. **DESIGN:** Create a design.
2. **DEVELOP:** Build something.
3. **TEST:** Prove that it works.

Making a mistake and then correcting it during the concept phase is WAY less expensive than discovering it and then trying to correct it during the testing phase, when everything has already been built. So traditional project management (a.k.a. _Waterfall_) places a high premium on taking those phases in order and NOT making any mistakes.

This approach to project management took us from the horse-and-buggy era to assembly lines and skyscrapers and battleships. But right about the time we started to send men to the Moon, this approach started breaking down.

In the real world, as project complexity increases, change becomes inevitable.

<figure >
  <img src="/assets/images/change-is-inevitable.png">
  <figcaption>Change is inevitable!</figcaption>
</figure>

Requirements change. New tools emerge, or we learn something about the old ones we didn't already know. Somebody finds a better approach.

There are a million reasons why a project might have to respond to change, and this becomes more likely as a project grows. So then the question is: _what are you going to do about it?_

Waterfall project management implements change management procedures. These are cumbersome at best. Agile takes a different approach by asking this question: _what would a project look like if change were frequent and expected?_

In Waterfall, an iteration happens when something breaks and the team goes back to the drawing board. A Waterfall iteration is an out-of-band activity.

**In Agile, the iteration is the fundamental unit of project planning and delivery.**

There are as many flavors of Agile as there are teams who practice it. This is by design: a tool should fit its purpose, so an Agile team is encouraged to adapt its approach until everything works. At its heart, Agile addresses these basic activities:

- Identify what needs to be done.
- Figure out what should be done NEXT and by whom.
- Support those people so they can get it all done.
- Maximize the quality of the outcome.

These activities are performed in a cycle... which is _itself_ one of those things that need to be done, and is therefore also subject to iterative improvement. So as requirements or project conditions change, a team's Agile practice naturally adjusts to accommodate those changes.

In the real world, EVERY project team engages in an iterative practice. The difference between Agile teams and everybody else is that Agile teams do what they do _on purpose_.

## Agile Tools

An _Issue_ in Agile is any work item that needs to be tracked. Early Agile practitioners wrote issues down on sticky notes and tracked their status by moving them across a wallboard. This is still practiced in many places, but it can be a challenge with a virtual team.

### Jira

At Karmaniverous, we will begin by tracking Issues using [Jira](https://www.atlassian.com/software/jira). Jira is a good choice because it integrates with a lot of other tools, which makes it easy to get new Issues into the Backlog from Slack, email, and wherever else makes sense.

Jira manages Issues on _tickets_. The more information a Jira ticket contains, the more useful it is! So however you organize your piece of the project—spreadsheets, documents, etc.—be sure also to drop links to those things into the relevant Jira ticket as appropriate.

**It is better to link to a document than to upload it to the ticket!** That way your document remains live, and any changes you make will be accessible from your link.

### Confluence

[Confluence](https://www.atlassian.com/software/confluence) is a companion product to Jira, and is similar to a wiki: it's a website where anybody can create & edit pages. Confluence is a great place to develop & maintain requirements documents, design artifacts, and policy documents like this one, because…

- Everything is searchable and indexed in a single place.

- It's easy to attach or link to documents (e.g. Google Sheets) that need to be maintained separately.

- A requirements page can easily display an up-to-date list of all related Jira tickets.

- Jira tickets usually back-link automatically to Confluence pages that reference them.

- Users can hold and resolve relevant discussions in comments right on a Confluence page.

- It is easy to generate Jira tickets straight from content or comments on a Confluence page.

### Slack

Any Slack channel can be attached to a Jira project using the [Jira Cloud App for Slack](https://slack.com/blog/productivity/an-easier-way-to-use-jira-with-slack).

If appropriate—say if the channel represents an Epic­—then messages reflecting changes to the relevant tickets can be automatically posted in that Slack channel. In any case, simply mentioning an Issue ID from the attached project will link the ticket to the channel. Here's an example:

<figure >
  <img src="/assets/images/slack-jira-integration.png">
  <figcaption>Jira Cloud App for Slack</figcaption>
</figure>

Slack users can also click on a message and instantly generate a ticket from it in the Jira backlog!

## Jira Issues

Issues in Jira are divided into categories. Each category carries its own [_Definition of Done_](https://www.scrum.org/resources/blog/done-understanding-definition-done).

<style>
  table.issue-type tr > th:nth-child(2), 
  table.issue-type tr > td:nth-child(2) {
    padding-left: 2em;
    padding-right: 2em;
  }

  table.issue-type > tbody > tr > td:first-child > figure {
    margin-top: 1em;
    justify-content: center;
  }

  table.issue-type > tbody > tr > td:first-child > figure > figcaption {
    font-size: 1em;
  }
</style>

<table class="issue-type">
  <thead>
    <tr>
      <th>Issue&nbsp;Type</th>
      <th>Description</th>
      <th>Definition&nbsp;of&nbsp;Done</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <figure>
          <img src="/assets/images/story_200px.png">
          <figcaption>Story</figcaption>
        </figure>
      </td>
      <td>
        <p>A <strong>Story</strong> is something somebody will MAKE.</p>    
        <p>Most Stories are first be expressed at a high level and then broken down into low-level Stories.</p>    
        <p><strong>High-level Stories</strong> usually describe an outcome on an actor in the system. For example: <em>A User can register a new account.</em></p>    
        <p>Resolving a high-level Story usually means breaking it down into a number of other issues, including lower-level Stories.</p>    
        <p><strong>Low-level Stories</strong> usually describe an outcome on the system itself. For example: <em>Add a graph component to visualize a user’s discounts over time.</em></p>    
        <p>A good low-level Story contains no open business decisions: resolving it means translating it directly into code.</p>    
        <p><strong>A good Story also expresses some validations.</strong> How do we know we did a good job? In high-level Stories, these will become integration tests. In low-level Stories, these will become unit tests.</p>    
        <p>Sometimes a new Story will contradict a previous Story. It's okay. Requirements change!</p>
      </td>
      <td>
        <p><u>High-Level Stories</u></p>
        <ul>
          <li>decomposed into lower-level issues</li>
          <li>validation criteria recorded</li>
        </ul>
        <p><u>Low-Level Stories</u></p>
        <ul>
          <li>code complete</li>
          <li>unit tests pass</li>
          <li>build succeeds</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>
        <figure>
          <img src="/assets/images/bug_200px.png">
          <figcaption>Bug</figcaption>
        </figure>
      </td>
      <td>
        <p>A <strong>Bug</strong> is something somebody will FIX.</p>
        <p>It needs to be fixed because the system's behavior as built does not match the expectations expressed in the related Story.</p>
        <p><strong>Every bug should be related to a Story!</strong> If a Bug doesn't represent the failure of the system to meet the requirements expressed in a specific Story, then it isn't a Bug. It's just undesirable behavior, and should be the subject of a new Story.</p>
        <p>A Bug can be resolved two ways. These are not mutually exclusive:</p>
        <ul>
          <li>The system's behavior is brought in line with requirements. Tests that were failing now pass, and new tests may be written to account for the bug condition.</li>
          <li>A new Story is written that makes the Bug irrelevant.</li>
        </ul>
      </td>
      <td>
        <ul>
          <li>All blocking Issues resolved.</li>
          <li>Bug condition resolved.</li>
          <li>Original unit tests pass.</li>
          <li>New unit tests cover bug condition & pass.</li>
          <li>Revised build succeeds.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>
        <figure>
          <img src="/assets/images/question_200px.png">
          <figcaption>Question</figcaption>
        </figure>
      </td>
      <td>
        <p>A <strong>Question</strong> is something somebody will DECIDE.</p>
        <p><strong>Questions are non-trivial by definition.</strong> Every issue has a comment thread where questions can be asked and answered. But if answering a question is a task that needs to be assigned, prioritized, and scheduled, create a Question.</p>
        <p>Every Question should express its own completion criteria. Decomposition is a valid completion criterion!</p>
      </td>
      <td>
        <ul>
          <li>All blocking Issues resolved.</li>
          <li>Specified completion criteria met.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>
        <figure>
          <img src="/assets/images/task_200px.png">
          <figcaption>Task</figcaption>
        </figure>
      </td>
      <td>
        <p>A <strong>Task</strong> is something somebody will DO.</p>
        <p>There are two distinctly different scenarios for using Tasks:</p>
        <ul>
          <li>When individual, non-trivial actions need to be assigned, prioritized, and tracked.</li>
          <li>When tasks are part of a workflow: sets of tasks created via automation & organized into flowcharts, where each blocked task is activated after its blocking tasks are completed.</li>
        </ul>
        <p>A task can also be broken down into other tasks. In this case, each of the child tasks should be a blocker for the parent task.</p>
        <p>Every Task should express its own completion criteria. Decomposition is a valid completion criterion!</p>
      </td>
      <td>
        <ul>
          <li>All blocking Issues resolved.</li>
          <li>Specified completion criteria met.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>
        <figure>
          <img src="/assets/images/epic_200px.png">
          <figcaption>Epic</figcaption>
        </figure>
      </td>
      <td>
        <p>An <strong>Epic</strong> is something somebody will USE.</p>
        <p>An Epic represents a very high-level grouping of other issues. Other issues should come out of the Backlog and be resolved in a single Sprint. If they can't be, then they should be broken down until they can.</p>
        <p>Epics endure across several Sprints, and generally represent a project phase with respect to a broad area of functionality. Epics can be organized into a Roadmap expressing the Project's long-term plan.</p>
        <p>Most Epics are never resolved, because the functionality they represent is always subject to future development. If not, an Epic is resolved when all the issues it contains are either resolved or moved to another Epic.</p>
      </td>
      <td>
        <ul>
          <li>All blocking Issues resolved.</li>
          <li>Specified completion criteria met.</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

One Issue Type that is NOT included here is the _Subtask_. These are a pain to manage in Jira. Instead, use the _decomposes into/derives from_ Issue Link described below to break an Issue down into more manageable parts.

## Issue Links

Any Issue can be linked to other Issues. Issue Links provide traceability, for example from low-level Stories to high-level Stories and related Questions. Links can also provide advanced functionality, for example the activation of Tasks in Task Workflows.

Here are the most common issue link types:

| Issue Link                         | Description                                                                                                                                                                                                                                                                                                  |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **blocks / is blocked by**         | A blocked Issue should not be activated (or resolved, depending) until the Issues that block it are resolved.<br /><br />Automated Task Workflows take advantage of this link type to activate blocked Tasks once their blockers are resolved.                                                               |
| **decomposes into / derives from** | Indicates that high-level Issue has been decomposed (broken down) into lower-level Issues.<br /><br />**This is a non-standard issue link!** [Click here](https://confluence.atlassian.com/adminjiraserver/configuring-issue-linking-938847862.html) for more info about creating custom link types in Jira. |
| **relates to**                     | This is a general-purpose informational link that can be created for any reason, and serves to make related Issues easily accessible.                                                                                                                                                                        |

## Issue Status

Every Issue has a Status that describes where it is in the development process. It must be one of the following:

| Issue&nbsp;Status    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Backlog**          | Every Issue starts in the Backlog. Backlog items are hidden from view on the Task Board.<br /><br />It should be easy to throw new issues into the Backlog. It is more important that they GET there than that they be well-crafted at the outset!<br /><br />Backlog issues are fleshed out & prioritized during Backlog Grooming Meetings. They are assigned to future Sprints and to individual workers during Sprint Planning Meetings. |
| **TODO**             | Think of this as the Backlog for the current Sprint. Issues in this status are live but are not currently being worked on.                                                                                                                                                                                                                                                                                                                  |
| **In&nbsp;Progress** | A worker should place an Issue In Progress when he begins work on it.                                                                                                                                                                                                                                                                                                                                                                       |
| **Done**             | Every Issue carries its own _Definition of Done_. Issues go into the _Done_ column when they have met all criteria.                                                                                                                                                                                                                                                                                                                         |

In the sections above, I mentioned an issue being _activated_. This means to change its status from _Backlog_ to either _TODO_ or _In Progress_.

## Story Points

Below you will see references to _sizing_ an Issue. Sizing is an attempt to guess the level of effort required for the assigned resource to complete the issue.

Sizing is intended to be a rough, conservative estimate. If an Issue is difficult to size without a detailed discussion, this is usually an indicator of one of two things:

- **The Issue is not well enough understood!** Before the actual work is assigned, somebody should clarify it.

- **The Issue is too big!** Before the actual work is assigned, somebody should break it up into smaller pieces.

Each of these is a great example of a Question issue, which can and should be created on the fly during Backlog grooming or any time one of these two questions comes up.

The unit of Issue Sizing is the [Story Point](https://www.atlassian.com/agile/project-management/estimation#:~:text=Story%20points%20are%20units%20of,work%2C%20and%20risk%20or%20uncertainty.).

Traditionally, Story Points have been a purely relative measure, not connected to any concrete work increment (e.g. man-hours). In practice, this is a little too abstract to be useful, so **at Karmaniverous an estimate of one Story Point will always equal one person-day of work.**

To keep things easy and conservative, Story Point estimates are drawn from a limited palette of numbers (often [some variation on a Fibonacci sequence](https://www.productplan.com/glossary/fibonacci-agile-estimation/)). Whatever your estimate is, choose the smallest number equal to or greater than it from the list.

Valid Story Point values at Karmaniverous are none (i.e. not yet sized), 0 (i.e. a trivial piece of work), 0.25, 0.5, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89...

A two-week Sprint comprises 10 working days. An Issue may only be assigned to a Sprint is it can be completed within that Sprint. The largest valid Story Point value that fits inside a Sprint is 8. Therefore no Issue worth more than 8 Story Points may be assigned to a Sprint!

{: .notice--warning}

**Any Issue larger than 8 Story Points MUST be broken down into smaller Issues** before it can be assigned to a Sprint!

## The Agile Cycle

Agile is a _practice_, not a philosophy. There are lots of different ways to practice Agile, but they all include most of the features described above.

At Karmaniverous, we will apply the [Scrum](https://www.atlassian.com/agile/scrum) method with two-week Sprints.

### Sprints & Releases

A Sprint is a package of work that can be accomplished in a two-week period.

Every Sprint should terminate with a Release: a system milestone where everything that SHOULD work at that point actually DOES work, except for defects that are known and documented with Bugs. A Release should encompass every aspect of the system that is managed by the Agile Process, including:

- production code
- unit tests
- design artifacts
- research
- requirements
- etc.

### Sprint Structure

Each two-week Sprint is framed by the following meeting structure.

<style>
  table.sprint-structure td {
    width: 20%;
  }

  table.sprint-structure td.active {
    background-color: lightYellow;
  }
</style>

<table class="sprint-structure">
  <thead>
    <tr>
      <th>Mon</th>
      <th>Tue</th>
      <th>Wed</th>
      <th>Thu</th>
      <th>Fri</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Standup<br />Backlog Grooming</td>
      <td>Standup<br />Sprint Planning</td>
      <td>Standup<br />Sprint Review</td>
      <td class="active">Standup<br />Retrospective</td>
      <td class="active">Standup<br />Peer Review</td>
    </tr>
    <tr>
      <td class="active">Standup</td>
      <td class="active">Standup<br />Backlog Grooming</td>
      <td class="active">Standup<br /></td>
      <td class="active">Standup<br />Peer Review</td>
      <td class="active">Standup</td>
    </tr>
    <tr>
      <td class="active">Standup<br />Backlog Grooming</td>
      <td class="active">Standup<br />Sprint Planning</td>
      <td class="active">Standup<br />Sprint Review</td>
      <td>Standup<br />Retrospective</td>
      <td>Standup<br />Peer Review</td>
    </tr>
  </tbody>
</table>

In the Sprint structure above, the current Sprint is highlighted.

Why is this Sprint structured the way it is?

- The mid-week start ensures continuity from one week to the next.

- Critical once-per-Sprint meetings are never scheduled on Mondays or Fridays where they can be impacted by long weekends or other weekend distractions.

- The middle week is left as light as possible so people can focus on tasking.

### Sprint Meetings

The Scrum machine runs on meetings. These meetings are regular, clearly-defined, and most are strictly time-boxed. When a meeting’s allotted time expires, the meeting ENDS.

Never fear, it will come around again!

| Meeting                               | Frequency    | Purpose                                              | Length  |
| ------------------------------------- | ------------ | ---------------------------------------------------- | ------- |
| [Standup](#standup)                   | Daily        | Report progress & eliminate blockers.                | 20m MAX |
| [Backlog Grooming](#backlog-grooming) | Weekly       | Refine, size, prioritize & sequence Backlog.         | 1h MAX  |
| [Peer Review](#peer-review)           | Weekly       | Peer-review work on critical Issues.                 | 1-2h    |
| [Sprint Planning](#sprint-planning)   | Every Sprint | Assign issues to workers in upcoming sprints.        | 1-2h    |
| [Sprint Review](#sprint-review)       | Every Sprint | Present the Sprint outcome to business stakeholders. | 1h MAX  |
| [Retrospective](#retrospective)       | Every Sprint | Evaluate & improve the Agile Cycle.                  | 1h MAX  |

#### Standup

The daily Standup is the backbone of the Agile Cycle. Effective Standups are a strong leading indicator that the Agile Cycle is working.

In an Agile Standup, everybody takes a turn to report three things:

- What they completed since the last standup.
- What they are working on today.
- Any blockers that might prevent them from completing their assigned work.

The goal of a Standup is speed and efficiency. For that reason, Standup meetings are strictly time-boxed: the Standup begins on time, and when the timer runs out, participants are encouraged to leave. Everybody should get their turn within the allotted time, and ALL follow-up conversations take place AFTER the Standup ends.

#### Backlog Grooming

Everybody is encouraged to throw Issues into the Backlog as they come up. Capturing it ALL is more important than capturing ANY of it at high quality.

The Backlog will usually contain lots of Issues that are incomplete or poorly articulated. The purpose of Backlog Grooming is to go through the Backlog and...

- **Assign every Issue to the correct project Phase.** Is it in scope for the current Phase? If not, let's drop it in the right bucket & forget about it for now.

- **Size every Issue.** How much work will an Issue take to resolve?

- **Prioritize every Issue.** Which are the most important?

- **Sequence every Issue.** What order must they be completed in to account for dependency?

- **Identify blockers.** What’s standing in the way of completing this Issue?

- **Decompose complex Issues.** How can we break big Issues into bite-sized pieces?

- **Fix the easy stuff on the spot, and create new tickets to account for the hard stuff**, so somebody can be assigned to do the work required to flesh those tickets out.

Backlog Grooming is NEVER finished!

Trying to groom the entire Backlog every time is a recipe for failure. Instead, time-box the effort, get through as much of the important stuff as possible, and then do it again regularly.

{: .notice--info}

If it looks like the Backlog is getting away from you over time, increase the frequency of Backlog Grooming meetings!

This is an iterative process! Expect to revisit the same Backlog items again and again and update dependencies, sizing, etc. as your understanding of each Issue deepens.

#### Peer Review

Peer review is an essential element of quality control.

Regularly-scheduled peer reviews give developers, designers, and analysts an opportunity to seek feedback from across the team on their most critical work items.

Peer review is NOT just about code. ANY Issue under active development is fair game!

Of course peer reviews can be performed at any time. The trouble is that they are often NOT done, even when they should be. Scheduling a regular availability creates space for this process to happen. In a project in progress, there will always be work worth reviewing!

#### Sprint Planning

The Sprint schedule is fixed: there’s a new one every two weeks. The purpose of the Sprint Planning Meeting is to decide:

- Which Issues will be assigned to future Sprints.
- Who will do the work.

What issues go into a given Sprint depends on several factors, including:

- Issue priority, dependency & size from Backlog Grooming
- Resource availability
- Business roadmap requirements

In a given Sprint, each Segment of the team (coders, BAs, etc) has a fixed amount of available bandwidth. Which Issues can fit into it, along with their known dependencies? This is the calculus of Sprint Planning.

Just like Backlog Grooming, SprintPlanning is an iterative process. Planning should look forward as many Sprints as practical, so that every Sprint is visited multiple times in subsequent Sprint Planning meetings.

#### Sprint Review

The Sprint Review is your team's official dog & pony show!

The audience for the Sprint Review is your business stakeholders. This will likely be the only time during the Sprint when many of them have direct contact with the team.

Of course your goal for the Sprint Review is to impress your bosses. But just as important is to deliver an ACCURATE picture of the health of the project. Expect questions like:

- Are we meeting our timeline?
- Are we meeting our budget?
- Are we addressing concern X and meeting requirement Y?

If you pay attention, you'll find that many new requirements are first dropped as hints in regular Sprint Reviews. So take notes!

#### Retrospective

The most powerful aspect of the Agile Methodology is that it is deliberately introspective: **every Agile process is itself subject to the Agile process!**

The purpose of the Retrospective meeting is to examine the Agile process as recently experienced by the team and make incremental improvements in order to make the team more effective.

This could take many forms, including:

- Introducing new Issue Types or changing the rules around existing ones.
- Altering Issue workflows.
- Changing the Definition of Done.
- Changing the frequency or procedures for Standups, Backlog Grooming or other meetings.

Everything is on the table! If the point of the Agile Methodology is to find out what works and then do it on purpose, then Retrospectives are how we make that a reality.

{: .notice--info}

The outcome of an effective Sprint Retrospective is a change to **this exact document!**
