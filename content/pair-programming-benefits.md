Title: Pair Programming Benefits
Date: 2014-06-20 20:21
Author: Ryan McKay
Tags: pairing, agile, pair-programming
Slug: pair-programming-benefits
Status: published

For the last year and a bit, I've been pair programming about 90% of my dev time (i.e. outside of meetings).  I've seen several major benefits compared to the previous 15 years of solo programming.  But I've seen some issues as well.  I think they are addressable, but they are issues to stay aware of.  I'm going to use this post to talk about the benefits, then follow up with another to discuss the issues I have identified.  

  

For the most part, I've been practicing what I'm going to refer to as Long Form Pairing.  Other than meetings and lunch, pair all day long.  Not a lot of attention paid to taking breaks or role switching.  However, we have dabbled in [Ping-Pong Pairing](http://c2.com/cgi/wiki?PairProgrammingPingPongPattern), [Applying the Pomodoro Technique to Pairing](http://agileworld.blogspot.com/2009/10/applying-pomodoro-technique-during-pair.html), and even [Ping-Pong Pomodoro Pair-Programming, or PPPPP](http://adam.pohorecki.pl/blog/2013/07/15/ppppp-talk-at-krug/).  In fact, at our most recent hackathon, one of my team mates wrote an [IntelliJ port](http://plugins.jetbrains.com/plugin/7467?pr=idea) of the [Pair Hero](http://www.happyprog.com/pairhero/) PPPPP Eclipse plugin.  

<div>

</p>

  

</div>

</p>

<div>

</p>

One person is doing the typing and mousing (sometimes referred to as the driver), while the other is watching and providing verbal input (the navigator).  We are typically working on a user story that has been pretty well broken down into tasks, so when code is being written, there is not a lot of discussion required about **what** needs to be done.  The conversation at that point is about **how** things should be done - things like how to factor functionality into classes and methods, which test cases we need, what to name things, third party libraries, etc.  

</div>

</p>

<div>

</p>

  

<h2>

</p>

Increased Focus

</h2>

</p>

One of the biggest benefits that I have experienced from pair programming is increased focus on the task at hand.  When you are pairing, you don't have time to check email, do research, go off on tangents in the code, etc.  You are hyper-focused on getting the current task done and moving on to the next one.  Emails come in - I ignore them.  Conversations happen around me - I don't even hear them.  I'm definitely not taking this opportunity to take care of that big refactoring that's been nagging at the back of my mind.

</div>

</p>

<div>

</p>

  

<h2>

</p>

Faster Context (Re)loading

</h2>

</p>

</div>

</p>

<div>

</p>

Some interruptions are unavoidable, for example, meetings, lunch, nights and weekends :)  Sometimes the other person has the same interruption; sometimes they don't.  Either way, it is much easier and faster to reload a context shared with another person than one you had by yourself.  Similarly, if you are just coming into a context for the first time, someone who already has it loaded can spin you up much faster than you can on your own.  

  

</div>

</p>

<h2>

</p>

Teaching/Learning Opportunity

</h2>

</p>

<div>

</p>

My current team is composed of all senior engineers (for better or worse), so there is not the same teaching opportunity I have had with junior developers in the past.  However, software engineering is a large field, and we all have varying experience in both quality and quantity, so there is still plenty of opportunity for learning from each other.  

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

We have identified several major technical areas of interest in our project, too many for any individual to specialize in all of them at the same time.  Some examples are Angular JS (and associated ecosystem), MongoDB, Spring (boot, data, IOC, etc), REST, and Testing.  Team members have selected their top three areas to focus on for this release.  Next release, we will shuffle them up, and encourage pairing between the SMEs and members who are new to the area.  

  

</div>

</p>

<h2>

</p>

Alternative Paths

</h2>

</p>

<div>

</p>

Having another pair of eyes helps you see alternatives you wouldn't otherwise see.  Period.  Some of these can save you time right now, for example:

</div>

</p>

<div>

</p>

<ul>

</p>

<p>

<li>

Remembering where some particular configuration properties are
</li>

</p>

<p>

<li>

Informing you of a library that does what you were about to implement
</li>

</p>

<p>

<li>

Explaining how the hell that variable is getting a null value
</li>

</p>

<p>

<li>

Containing scope creep
</li>

</p>

<p>

<li>

Suggesting an easier way to do something, so you can get it done now instead of pulling it out of the story for tech debt
</li>

</p>

</ul>

</p>

Others can save you a lot of time later, for example:

</div>

</p>

<div>

</p>

<ul>

</p>

<p>

<li>

Identifying missing test cases
</li>

</p>

<p>

<li>

Suggesting a more maintainable design
</li>

</p>

</ul>

</p>

This is the area where I see the strongest relation with the driver/navigator analogy.  The driver has a very tactical view of the problem.  When the pairs' skill levels are pretty evenly matched, if both partners are going full speed, the driver simply cannot consider the same breadth and depth of strategic concerns as the navigator can.  In fact, sometimes the driver will have to stop typing in order to catch up mentally.  

</div>

</p>

<div>

</p>

  

</div>

</p>

Even in the highly unlikely case that neither of the partners learns a thing, they are producing better code, faster, right now.  Happily, both partners **will** be learning and getting better.  

  

<h2>

</p>

Adherence to Team Norms

</h2>

</p>

<div>

</p>

Over the past year, several team norms have emerged.  Some of them are explicit, and recorded in Confluence.  For example, we have discussed and decided on several policies about how we test our software - what should be tested at various levels, how to set up tests, wording of test names, etc.  Others are more implicit, like patterns in the codebase for how we address certain design elements.  Individuals tend to deviate from these standards from time to time for various reasons.  Sometimes you simply forget or were not aware.  Other times, you're just feeling a bit lackadaisical.  And yes, sometimes its a standard you didn't agree with in the first place, and you don't feel like following it right now.  

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

As an analogy, I've been doing a lot of swimming lately, trying to improve my technique.  For argument's sake, let's just say I know exactly what I need to do to have a really efficient body position and stroke.  But when I'm in the water actually doing it, trying to focus on all the different elements, and get enough oxygen, its really difficult.  I'll notice that my head position is a bit off, and shift my focus there for a while.  Then I'll notice myself slacking off on rotating my body with each stroke and try to address that.  And so on.  My point is, its easy to say, "Everyone should follow the standards 100% of the time", but even for mature, experienced software developers, that is not realistic.

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

Pair programming increases adherence to team standards in a couple ways.  Sometimes your pairing partner will see you starting to go down a path that is out of line with the standards, and remind you.  But a lot of times, they don't have to say anything - just having another person there watching what you type applies pressure to do it "right".

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

<h2>

</p>

Norming between pairing partners

</h2>

</p>

</div>

</p>

<div>

</p>

Obviously, there isn't a team standard for every situation or even most situations.  There is plenty of room for individuality.  There are many facets to software engineering, and individual developers tend to be more dogmatic on some and more pragmatic on others.  I have a couple of favorites, like making domain objects immutable and always constructed by fluent builders, or replacing conditionals with polymorphism.  I also have a few pet peeves I specifically look out for, like overuse of generic interfaces.

</div>

</p>

<div>

</p>

  

</div>

</p>

<div>

</p>

Pair programming tends to bend each developer's tendencies toward the pragmatic.  For example, if a domain object only has one or two data members, do you really need to use a builder?  Or if there are only one or two cases in the conditional, is it really worth using separate strategy implementations?  Do we really need to make every class with a single arg non-void return method implement the Handler interface?  No.  

  

</div>

</p>

``` brush:
public interface Handler<T,R> {    R handle( T t );} 
```

</p>

  

<h2>

</p>

Conclusion

</h2>

</p>

<div>

</p>

When a pair is really clicking, there is no doubt in my mind, they are going significantly faster, writing significantly better code, and learning more than an individual programmer.  I have experienced the difference first hand, and it is awesome.  But pairs don't always fire on all cylinders.  I'll talk about that in another post.

</div>

</p>

<div>

</p>

<ul>

</p>

</ul>

</p>

</div>

</p>
