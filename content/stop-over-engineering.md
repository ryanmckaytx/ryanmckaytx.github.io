Title: Stop Over-Engineering
Date: 2017-05-24 09:09
Author: Ryan McKay
Tags: Readings+Videos
Slug: stop-over-engineering
Status: published

I just watched [Greg Young's Build Stuff 2016 Keynote - Stop Over-Engineering](https://www.youtube.com/watch?v=GRr4xeMn1uU), and had a lot of good takeaways.  

Your software is only part of an overall business process system.  You don't have to solve every problem using software.  Definitely not early on, and typically not ever.  Why wouldn't you try to solve all the problems?  Because the cost benefit tradeoff doesn't make sense.  Many tasks are less expensive for humans to do than to try to automate.  Instead of trying to handle every edge case, regardless of how infrequently it might be encountered, focus on handling the happy path, and detecting when the user has gone off the happy path.  When that happens, hand it off to humans.  

How do you know what is the happy path?  He used the analogy, "Stop watering the weeds in your life and start watering the flowers".  How do you know where the flowers are? **Data**.  

This is where a distinction was made between brown-field projects and green-field projects.  Brown-field projects have the advantage of usage data.  You can see which features are used the most and how they are used.  So it is easer to make data-driven decisions about where to invest effort.  

He gave an example of an invoicing app.  If you try to capture all the requirements for an invoicing system, you will be in endless meetings.  This struck a chord with me, because in a past job, I worked on a brownfield project in the consumer financial sector, and I had exactly this same experience.  In Greg's case, after 2 weeks of meetings, they decided to stop capturing exhaustive requirements, and instead, look at the past year's worth of actual invoices.  Then the domain experts were simply asked to classify the invoices wrt whether they were on the happy path or not, and how to detect getting outside the happy path.  In one day, they were able to implement a solution that solved 60-70% of the previous year's invoices.  Then they looked at the next most common case (15%) and solved that the next day.  And the next (6%).  After 2 weeks they got to 99%, and then they stopped.  They had reached the point of diminishing returns on investment.  That project had been budgeted to take 9 months.  **Don't spend 4 days of modeling to automate a task that takes a human 5 minutes of work once a year!**  

The problem with green-field projects is that there is no usage data.  He asks the question, how many of us have built features that have never been used?  How many of us have build whole products that have never been used?  So how do we know where the flowers are?  We need to get data.  How do we get data?  Greg described two complementary approaches: Throw sh\*t at the wall (and see what sticks), and human concierge service.  The first approach is to build small, unpolished pieces of functionality and see what people actually like.  Taken to the extreme, you get the [Feature Fake](https://www.industriallogic.com/blog/fast-frugal-learning-with-a-feature-fake/), where you make it look like you have added a new feature to your application, and see who shows interest in it.  

The human concierge service actually has no automation at all.  Humans do all the work for a while, so that you can gather usage data, and find the happy path to automate first.  Human concierge is one of several [lean experimentation techniques](http://www.movestheneedle.com/blog/enterprise-lean-startup-experiment-examples/).  

Some other bullet points:  

* Most things in software aren't worth building
* Cost Benefit Analyze everything!  Where is your break even point?  If you can't figure it out - stop.  And figure it out.
* Every developer should hire another developer to build something, to see the CBA from the other side.