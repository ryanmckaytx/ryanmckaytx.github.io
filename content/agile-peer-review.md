Title: Agile Peer Review
Date: 2015-08-01 06:40
Author: Ryan McKay
Slug: agile-peer-review
Status: published

<h1>

</p>

Preface

</h1>

</p>

What follows is primarily the result of nearly 2 years of intentional refinement by an experienced, high-performing agile team. Dev team members, architects, product owner, and even members of other dev teams participated and helped shape our thoughts on review. I also incorporated ideas from several other experienced peers. But like anything agile, it's a work in progress.  

  

Don't let preconceptions about the term "review" color your reading - most of the reviews called for are only 5-15 minutes long.

  

<h1>

</p>

Overview

</h1>

</p>

Functional defects and design problems get exponentially more expensive the longer they are allowed to exist. The earlier those problems can be found and fixed, the better. Peer reviews provide an inexpensive, effective method for doing that.  

  

There are several major work products in the lifecycle of a user story that warrant review (these are discussed in detail below):  

<ul>

</p>

<p>

<li>

Domain Model - The team’s shared, fundamental, documented understanding of the domain.

</li>

</p>

<p>

<li>

Demo script - How the dev team will demonstrate to stakeholders that the story is functionally complete.

</li>

</p>

<p>

<li>

Design - Focused on the API and its test cases, but also covering important implementation details. 

</li>

</p>

<p>

<li>

Pull Request - A single coherent change to the shared codebase, in service of a story. 

</li>

</p>

<p>

<li>

Code Complete - The full set of changes to the codebase that were made to complete the story.

</li>

</p>

</ul>

</p>

<h1>

</p>

Benefits

</h1>

</p>

<h4>

</p>

These reviews deliver tangible value to the business

</h4>

</p>

<ul>

</p>

<p>

<li>

Higher-quality code 

</li>

</p>

<p>

<li>

Higher, more consistent velocity

</li>

</p>

</ul>

</p>

<h4>

</p>

And additional value to the dev team

</h4>

</p>

<ul>

</p>

<p>

<li>

Pride in work 

</li>

</p>

<p>

<li>

Self-improvement

</li>

</p>

</ul>

</p>

<h4>

</p>

As a result of the direct effects of the reviews

</h4>

</p>

<ul>

</p>

<p>

<li>

Getting a better product, sooner 

</li>

</p>

<ul>

</p>

<p>

<li>

Bugs identified, eliminated earlier in the process 

</li>

</p>

<p>

<li>

More maintainable code

</li>

</p>

<p>

<li>

Better tested

</li>

</p>

<p>

<li>

More adherent to team/organizational standards

</li>

</p>

</ul>

</p>

<p>

<li>

Knowledge sharing

</li>

</p>

<ul>

</p>

<p>

<li>

Team members sharing the same frame of mind

</li>

</p>

<p>

<li>

Better understanding of the domain

</li>

</p>

<p>

<li>

Better understanding of how system components interact

</li>

</p>

<p>

<li>

Improving our craftsmanship every day by learning from each others strengths

</li>

</p>

<p>

<li>

Better understanding of our tools and technical stack

</li>

</p>

</ul>

</p>

</ul>

</p>

<h1>

</p>

What to Review

</h1>

</p>

<h2>

</p>

Domain Model

</h2>

</p>

The fundamental, documented understanding of the domain shared among the whole team including the Product Owner. A critical part of this model is the Ubiquitous Language, which unambiguously expresses domain concepts, enabling effective communication. Any change in that understanding or expression needs to be reviewed by the rest of the team, to keep everyone on the same page, and to consider potential impact to the software design. Not every story changes the Domain Model.

  

<h2>

</p>

Demo Script

</h2>

</p>

How the dev team will demonstrate to stakeholders that the story is functionally complete. This would involve the dev team and Product Owner. Depending on team/org structure, more or less of this might be provided to the dev team by PO, or might even be developed collaboratively, in which case a separate review would be unnecessary. 

  

  

There is not a lot of detail here, just a handful of user-facing scenarios. If it is more than a handful, the story is probably too big.. It shouldn't take much time at all to produce or review. If it does take a long time, again it's probably too big, or there is a lack of understanding that needs to be addressed before proceeding. For example:  

<ul>

</p>

<p>

<li>

Create an event

</li>

</p>

<ul>

</p>

<p>

<li>

Go to the Event list page

</li>

</p>

<p>

<li>

Click "New Event" button

</li>

</p>

<p>

<li>

Fill in information in Event form, click save

</li>

</p>

<p>

<li>

Should show Event overview page with saved info

</li>

</p>

<p>

<li>

Go back to Event list page, new event should be there

</li>

</p>

</ul>

</p>

<p>

<li>

Register for an Event

</li>

</p>

<ul>

</p>

<p>

<li>

Registrant Role

</li>

</p>

<ul>

</p>

<p>

<li>

Go to Event Registration page

</p>

Fill in form, submit

</li>

</p>

</ul>

</p>

<p>

<li>

Admin Role

</li>

</p>

<ul>

</p>

<p>

<li>

Go to Event overview page, see new Registration

</li>

</p>

</ul>

</p>

</ul>

</p>

</ul>

</p>

<h2>

</p>

Design

</h2>

</p>

For a microservice, this would focus on the API and its test cases, but would also cover important implementation details. Generally should be developed collaboratively with the major client(s) of the API, e.g. UI. In which case, the review with the rest of the team shouldn't take long.  

<ul>

</p>

<p>

<li>

Resource endpoints

</li>

</p>

<p>

<li>

Methods on those endpoints

</li>

</p>

<p>

<li>

API objects

</li>

</p>

<p>

<li>

API test cases

</li>

</p>

<p>

<li>

API versioning

</li>

</p>

<p>

<li>

New runtime dependencies, e.g. now we're going to have to use the Org service

</li>

</p>

<p>

<li>

Major changes to infrastructure, e.g. using a new database, new message queue

</li>

</p>

<p>

<li>

Data migration

</li>

</p>

</ul>

</p>

<h2>

</p>

Pair Programming

</h2>

</p>

This is effectively continuous code review between two team members while the code is being written. I wrote up most of my thoughts on this practice [here](http://againstentropy.blogspot.com/2014/06/pair-programming-benefits.html). Because of the normalizing and quality control forces exerted here, the pull request and code complete reviews go faster.

  

<h2>

</p>

Pull Request

</h2>

</p>

Code gets merged into the shared repository by issueing a pull request. This should occur frequently, typically multiple times per day per developer. It should be coherent and correct, but not necessarily complete. Each pull request gets reviewed by one or more other team members. In my experience, the types of things caught here are team standards violations, code/test readability (e.g. method and variable names), and code factoring.

  

<h2>

</p>

Code Complete

</h2>

</p>

When the dev(s) working on a story consider it code complete, the code and its tests should be reviewed by the rest of the dev team, typically plus an architect.  

<ul>

</p>

<p>

<li>

The person(s) who did the coding walk everyone else through it

</li>

</p>

<ul>

</p>

<p>

<li>

Generally start with brief review of the story

</li>

</p>

<p>

<li>

Then walk through the important parts of the code

</li>

</p>

<ul>

</p>

<p>

<li>

And the tests!

</li>

</p>

</ul>

</p>

</ul>

</p>

<p>

<li>

While those persons are presenting, someone else is taking action item notes

</li>

</p>

<ul>

</p>

<p>

<li>

Somewhere public, or just email out after

</li>

</p>

</ul>

</p>

<p>

<li>

The story is not considered dev-complete until those action items are addressed.

</li>

</p>

<ul>

</p>

<p>

<li>

Creating a tech debt story to do it later is a valid option but should not be abused 

</li>

</p>

</ul>

</p>

<p>

<li>

This is at a level similar to Design review, focusing on API classes and tests, and any deviations from the original design.

</li>

</p>

</ul>

</p>

<h1>

</p>

How to Review

</h1>

</p>

<h2>

</p>

When to review?

</h2>

</p>

Each work product should be reviewed as early as feasible, because course correction is cheaper the earlier you do it. Also, the less time elapses between creation and review, the more of the context the creator(s) will still have loaded. Finally, by spreading the reviews out over the lifetime of the story (as opposed to reviewing everything at the end), you avoid the tendency to get mentally overwhelmed by the volume of work to be reviewed and just rubber-stamp it. 

  

  

On my team, we had one-week sprints, Monday to Friday. We took Monday morning to commit to a set of stories for the sprint, produce and review demo plans for all the committed stories, and produce and review design and test plan for the first one or two stories we were starting work on. All other reviews were performed as needed at the end of each day.

  

<h2>

</p>

Why not just create together instead of reviewing later?

</h2>

</p>

There is a tradeoff to be made between direct collaboration and ex post facto review. In my experience, you typically want to keep creative activities for a given story limited to 2-3 people, 4 at the most. Otherwise the cost/benefit starts to deteriorate.

  

<h2>

</p>

How much time to invest?

</h2>

</p>

For a team that has been together a while, the amount of time to target for each review is ~30 minutes for code complete review, and 5-15 minutes for the others. Expect more review time early on in a team's formation. However, team norming will drive the time down significantly, and performing these reviews accelerates team norming. You don't keep finding the same things in review, because you incorporate the feedback and get better.

  

<h2>

</p>

Who does the reviewing?

</h2>

</p>

The composition of the reviewing group varies based on what is being reviewed, and larger review groups will tend to use more person-hours, because:  

<ul>

</p>

<p>

<li>

There are more people spending the time

</li>

</p>

<p>

<li>

More people talk more

</li>

</p>

<p>

<li>

The members of large groups will be less familiar with what is being reviewed

</li>

</p>

</ul>

</p>

<h1>

</p>

Putting it all Together: A Sprint in the Life of a Story

</h1>

</p>

<ul>

</p>

<p>

<li>

**Monday morning** - Dev team meets to commit to stories for sprint, and devise demo script for those stories. Right after that meeting, they meet with Product Owner to review the demo script, adjust as necessary.

</li>

</p>

<p>

<li>

**Monday afternoon** - One pair of developers start on a medium-sized story. They come up with a design together. During the design, they realize that their Domain Model and Ubiquitous Language will need to change. Previously, Constituents could only register for Events. Now a Constituent can be designated as the "Event Organizer". They ask the rest of the team and an architect for a design review, scheduled for 30 minutes later. In the mean time, they check the proposed new terminology with the Product Owner, and add the term to the project's wiki. Then they go ahead and start implementing the design.

</p>

  
In the design review, an overlooked test case is identified and added, and the format of a timestamp field is changed to suit the UI. Minimal rework in the codebase is required. They also take that opportunity to inform the rest of the dev team and the architect of the new Ubiquitous Language term.

Monday end of day - The pair issues a pull request. One other developer is still in the office, spends 5 minutes reviewing it. Catches a few issues, which he comments on, but nothing serious, so he merges the pull request.

</li>

</p>

<p>

<li>

**Tuesday, Wednesday** - Several more pull requests issued. Since the rest of the team is there, they both review both pull requests. The architect notices the pull requests going by and adds a few comments as well. Late Wednesday afternoon, the pair working on the story determines that it is code complete, and schedules a team code review for end of day.

</li>

</p>

<p>

<li>

**Wednesday end of day** - Dev team + Architect code review for the story. One of the dev pair refreshes everyone's memory about the story, then walks through the code and tests. The other member of the dev pair is taking notes. A few minor issues are caught, and one potentially serious scalability issue. The team decides to consult historical metrics to see if it will be a problem short- or long-term. Once they determine that it would take about a year of usage at current rates to become a problem, they decide to add monitoring for this story, and add a tech debt story to address the scaling longer-term.

</li>

</p>

<p>

<li>

**Thursday morning** - Dev pair demos the story to the Product Owner according to the demo plan.

</li>

</p>

</ul>

</p>

<h1>

</p>

Further Reading

</h1>

</p>

<ul>

</p>

<p>

<li>

[Code Reviews: Just Do It](http://blog.codinghorror.com/code-reviews-just-do-it/) - Quotes some great case study numbers on the efficacy of code review

</li>

</p>

<p>

<li>

[Effective Code Reviews: 9 Tips from a Converted Skeptic](http://blog.fogcreek.com/effective-code-reviews-9-tips-from-a-converted-skeptic/)

</li>

</p>

<p>

<li>

Smart Bear: [Best Practices for Code Reviews](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/)

</li>

</p>

<p>

<li>

Atlassian: [Creating Optimal Reviews](http://blogs.atlassian.com/2011/07/creating_optimal_reviews/)

</li>

</p>

<p>

<li>

[Domain-Driven Design: Tackling Complexity in the Heart of Software, Ch. 2, Section 1: Ubiquitous Language](http://techbus.safaribooksonline.com/book/project-management/0321125215/communication-and-the-use-of-language/ch02lev1sec1)

</li>

</p>

<p>

<li>

[The Art of Agile Development: Ubiquitous Language](http://www.jamesshore.com/Agile-Book/ubiquitous_language.html) - Shorter version of above - definition and benefits of Ubiquitous Language

</li>

</p>

<p>

<li>

[Pair Programming Benefits](http://againstentropy.blogspot.com/2014/06/pair-programming-benefits.html) - My experience with Pair Programming

</li>

</p>

<p>

<li>

[Pair Programming Benefits](http://www.summa-tech.com/blog/2013/05/16/pair-programming-benefits-part-1-the-good) - Another blog on the pluses and minuses of Pair Programming

</li>

</p>

</ul>

</p>
