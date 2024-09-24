Title: Agile Peer Review
Date: 2015-08-01 06:40
Author: Ryan McKay
Slug: agile-peer-review
Status: published

What follows is primarily the result of nearly 2 years of intentional refinement by an experienced, high-performing agile team. Dev team members, architects, product owner, and even members of other dev teams participated and helped shape our thoughts on review. I also incorporated ideas from several other experienced peers. But like anything agile, it's a work in progress.  

Don't let preconceptions about the term "review" color your reading - most of the reviews called for are only 5-15 minutes long.

# Overview
Functional defects and design problems get exponentially more expensive the longer they are allowed to exist. The earlier those problems can be found and fixed, the better. Peer reviews provide an inexpensive, effective method for doing that.  

There are several major work products in the lifecycle of a user story that warrant review (these are discussed in detail below):  
* Domain Model - The team’s shared, fundamental, documented understanding of the domain.
* Demo script - How the dev team will demonstrate to stakeholders that the story is functionally complete.
* Design - Focused on the API and its test cases, but also covering important implementation details. 
* Pull Request - A single coherent change to the shared codebase, in service of a story. 
* Code Complete - The full set of changes to the codebase that were made to complete the story.

# Benefits
### These reviews deliver tangible value to the business
* Higher-quality code 
* Higher, more consistent velocity

### And additional value to the dev team
* Pride in work 
* Self-improvement

### As a result of the direct effects of the reviews
* Getting a better product, sooner 
* Bugs identified, eliminated earlier in the process 
* More maintainable code
* Better tested
* More adherent to team/organizational standards
* Knowledge sharing
* Team members sharing the same frame of mind
* Better understanding of the domain
* Better understanding of how system components interact
* Improving our craftsmanship every day by learning from each others strengths
* Better understanding of our tools and technical stack

# What to Review
## Domain Model
The fundamental, documented understanding of the domain shared among the whole team including the Product Owner. A critical part of this model is the Ubiquitous Language, which unambiguously expresses domain concepts, enabling effective communication. Any change in that understanding or expression needs to be reviewed by the rest of the team, to keep everyone on the same page, and to consider potential impact to the software design. Not every story changes the Domain Model.

## Demo Script
How the dev team will demonstrate to stakeholders that the story is functionally complete. This would involve the dev team and Product Owner. Depending on team/org structure, more or less of this might be provided to the dev team by PO, or might even be developed collaboratively, in which case a separate review would be unnecessary. 

There is not a lot of detail here, just a handful of user-facing scenarios. If it is more than a handful, the story is probably too big.. It shouldn't take much time at all to produce or review. If it does take a long time, again it's probably too big, or there is a lack of understanding that needs to be addressed before proceeding. For example:  

* Create an event
    * Go to the Event list page
    * Click "New Event" button
    * Fill in information in Event form, click save
    * Should show Event overview page with saved info
    * Go back to Event list page, new event should be there
* Register for an Event
    * Registrant Role
        * Go to Event Registration page
        * Fill in form, submit
    * Admin Role
        * Go to Event overview page, see new Registration

## Design
For a microservice, this would focus on the API and its test cases, but would also cover important implementation details. Generally should be developed collaboratively with the major client(s) of the API, e.g. UI. In which case, the review with the rest of the team shouldn't take long.  

* Resource endpoints
* Methods on those endpoints
* API objects
* API test cases
* API versioning
* New runtime dependencies, e.g. now we're going to have to use the Org service
* Major changes to infrastructure, e.g. using a new database, new message queue
* Data migration

## Pair Programming
This is effectively continuous code review between two team members while the code is being written. I wrote up most of my thoughts on this practice [here](http://againstentropy.blogspot.com/2014/06/pair-programming-benefits.html). Because of the normalizing and quality control forces exerted here, the pull request and code complete reviews go faster.

## Pull Request
Code gets merged into the shared repository by issueing a pull request. This should occur frequently, typically multiple times per day per developer. It should be coherent and correct, but not necessarily complete. Each pull request gets reviewed by one or more other team members. In my experience, the types of things caught here are team standards violations, code/test readability (e.g. method and variable names), and code factoring.

## Code Complete
When the dev(s) working on a story consider it code complete, the code and its tests should be reviewed by the rest of the dev team, typically plus an architect.  

* The person(s) who did the coding walk everyone else through it
    * Generally start with brief review of the story
    * Then walk through the important parts of the code
        * And the tests!
* While those persons are presenting, someone else is taking action item notes
    * Somewhere public, or just email out after
* The story is not considered dev-complete until those action items are addressed.
    * Creating a tech debt story to do it later is a valid option but should not be abused 
* This is at a level similar to Design review, focusing on API classes and tests, and any deviations from the original design.

# How to Review
## When to review?
Each work product should be reviewed as early as feasible, because course correction is cheaper the earlier you do it. Also, the less time elapses between creation and review, the more of the context the creator(s) will still have loaded. Finally, by spreading the reviews out over the lifetime of the story (as opposed to reviewing everything at the end), you avoid the tendency to get mentally overwhelmed by the volume of work to be reviewed and just rubber-stamp it. 

On my team, we had one-week sprints, Monday to Friday. We took Monday morning to commit to a set of stories for the sprint, produce and review demo plans for all the committed stories, and produce and review design and test plan for the first one or two stories we were starting work on. All other reviews were performed as needed at the end of each day.

## Why not just create together instead of reviewing later?
There is a tradeoff to be made between direct collaboration and ex post facto review. In my experience, you typically want to keep creative activities for a given story limited to 2-3 people, 4 at the most. Otherwise the cost/benefit starts to deteriorate.

## How much time to invest?
For a team that has been together a while, the amount of time to target for each review is ~30 minutes for code complete review, and 5-15 minutes for the others. Expect more review time early on in a team's formation. However, team norming will drive the time down significantly, and performing these reviews accelerates team norming. You don't keep finding the same things in review, because you incorporate the feedback and get better.

## Who does the reviewing?
The composition of the reviewing group varies based on what is being reviewed, and larger review groups will tend to use more person-hours, because:  
* There are more people spending the time
* More people talk more
* The members of large groups will be less familiar with what is being reviewed

# Putting it all Together: A Sprint in the Life of a Story
## Monday morning
Dev team meets to commit to stories for sprint, and devise demo script for those stories. Right after that meeting, they meet with Product Owner to review the demo script, adjust as necessary.
## Monday afternoon
One pair of developers start on a medium-sized story. They come up with a design together. During the design, they realize that their Domain Model and Ubiquitous Language will need to change. Previously, Constituents could only register for Events. Now a Constituent can be designated as the "Event Organizer". They ask the rest of the team and an architect for a design review, scheduled for 30 minutes later. In the mean time, they check the proposed new terminology with the Product Owner, and add the term to the project's wiki. Then they go ahead and start implementing the design.

In the design review, an overlooked test case is identified and added, and the format of a timestamp field is changed to suit the UI. Minimal rework in the codebase is required. They also take that opportunity to inform the rest of the dev team and the architect of the new Ubiquitous Language term.

Monday end of day - The pair issues a pull request. One other developer is still in the office, spends 5 minutes reviewing it. Catches a few issues, which he comments on, but nothing serious, so he merges the pull request.

## Tuesday, Wednesday
Several more pull requests issued. Since the rest of the team is there, they both review both pull requests. The architect notices the pull requests going by and adds a few comments as well. Late Wednesday afternoon, the pair working on the story determines that it is code complete, and schedules a team code review for end of day.

## Wednesday end of day
Dev team + Architect code review for the story. One of the dev pair refreshes everyone's memory about the story, then walks through the code and tests. The other member of the dev pair is taking notes. A few minor issues are caught, and one potentially serious scalability issue. The team decides to consult historical metrics to see if it will be a problem short- or long-term. Once they determine that it would take about a year of usage at current rates to become a problem, they decide to add monitoring for this story, and add a tech debt story to address the scaling longer-term.

## Thursday morning
Dev pair demos the story to the Product Owner according to the demo plan.

# Further Reading
* [Code Reviews: Just Do It](http://blog.codinghorror.com/code-reviews-just-do-it/) - Quotes some great case study numbers on the efficacy of code review
* [Effective Code Reviews: 9 Tips from a Converted Skeptic](http://blog.fogcreek.com/effective-code-reviews-9-tips-from-a-converted-skeptic/)
* Smart Bear: [Best Practices for Code Reviews](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/)
* Atlassian: [Creating Optimal Reviews](http://blogs.atlassian.com/2011/07/creating_optimal_reviews/)
* [Domain-Driven Design: Tackling Complexity in the Heart of Software, Ch. 2, Section 1: Ubiquitous Language](http://techbus.safaribooksonline.com/book/project-management/0321125215/communication-and-the-use-of-language/ch02lev1sec1)
* [The Art of Agile Development: Ubiquitous Language](http://www.jamesshore.com/Agile-Book/ubiquitous_language.html) - Shorter version of above - definition and benefits of Ubiquitous Language
* [Pair Programming Benefits](/pair-programming-benefits.html) - My experience with Pair Programming
* [Pair Programming Benefits](http://www.summa-tech.com/blog/2013/05/16/pair-programming-benefits-part-1-the-good) - Another blog on the pluses and minuses of Pair Programming