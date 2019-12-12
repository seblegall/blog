---
title: "Hire engineers to solve your company issues, not complex maths problems you don't have"
description: "Hires engineers to solve your company issues, not complex maths problems you don't have"
date:   2019-12-12T09:00:00+02:00
author: Sebastien Le Gall
categories: [management, management 3.0, culture, hiring]
---

Let's face it, your company is not a **Big Tech** company. You, the reader, are probably not working for Google, Microsoft or Facebook and it's ok. A lot of companies not so big have a huge impact and very high tech challenges.

But not being a big tech company also means you probably won't be good at what you're doing if you act the same way Facebook or Amazon does. This is true for tech choices and system architecture. This is also true for recruiting software engineers.

![Google Interview](https://i.ytimg.com/vi/XKu_SEDAykw/maxresdefault.jpg)
<!--more-->
Being hired by big tech companies is hard. Interviews are full of complex algorithms to implement or subtle mathematical problems to solve. And even if you succeed in your technical interview after several months of preparations reading [cracking the coding interview](https://www.amazon.fr/Cracking-Coding-Interview-6th-Programming/dp/0984782850), there are still 4 to 10 interviews to pass.

Only Microsoft and Google can afford such an interviewing process. And yet, those companies set the bar very high not only because everyone wants to work for them and they have a choice to make. But also because being a big tech company means having actual big tech issues to solve. Which is probably not the case at your company.

### Maths problems and complex algorithms are not what your company tries to solve

I won't let the troll out with an easy "nothing smart have been invented since the hash-tables". But hey! Have a look at what you're doing and what kind of issues you're trying to solve right now....  Wait! Let me guess! C-R-U-D ?

Truth is, whatever the complex algorithms you think you are working on, in the end, most engineering team mostly over-engineer solutions to simple problems or just implement Create, Read, Update and Delete operations

So, what are those issues your company wanna solve? Complex mathematical problems? Probably not. Being able to sustain millions of requests per second on your website? I doubt it.

[Russ Cox](https://twitter.com/_rsc) once said :

> Software engineering is what happens to programming when you add time and other programmers

Here it is! Issues your company needs to solve are :

- code must last
- people have to work together

That's fucking all. Forget all the pretensions you had about the so-called complex algorithms you thought you had. The reality is much more simple.

**It is much more complex to write readable and maintainable code than complex algorithms**. And when you just even try, suddenly a bunch of engineers stops whatever they were doing and start explaining to you what you should or shouldn't do, each with a different point of view. Most people have their own understanding of what is readable and that's why it generates so many debates. Plus, as each one have also its own capability of understanding code logic and complex architecture, of course, maintainable doesn't mean the same for everyone.

Then, there is this **BIG** issue to solve: working together. There is a reason why there are so many books about management, organizations, and culture: **making people work together is the hardest challenge of your company.**

It's not different in a team of software engineers. And I'm not talking about git conflict. I'm talking about making people understand each other and work together.

Have you ever been in a meeting where nobody knew why she/he was there?

Have you ever been part of a debate where nobody understands each other?

Have you ever seen someone breaking a code implementation rule just because he/she didn't know about it (and because, of course, you didn't document it?)

That's the kind of issue that costs more than you think both in energy and money.

### Hire people who know clean code principles, not tech specialists

Facing this reality should help you make better choices when hiring software engineers. Most of the time, your company cannot afford big tech companies' practices in terms of hire. Don't hire maths problem solvers. Don't hire tech specialists. Don't even hire people having great theories on what is or is not the best way to write code.

Learning a new language means learning about 900 keywords. **Hiring a Java specialist won't help you when you'll have to move to Python.** Worst, that specialist won't even try to do the move and will leave your company with a bunch of domain-specific logic undocumented.

Instead, **hire people that have read and understand the principles of Clean Code.** People that know how to implement the SOLID principles, whatever the language. Because, in the end, the tech stack doesn't matter... so much. Once an engineer has understood some of the most important key principles of writing testable code, he/she is basically a better programmer than most of the other software engineers anyway.

And even if some of the Clean Code and the Domain Driven Design principles apply differently depending on the language, in the end, it's just a matter of choosing a convention. A good example of that is the difference between Java and Go: Java allows screaming architecture (aka package named after the layer they represent such as *Domain* or *Infrastructure*.) Idiomatic Go doesn't allow such a naming convention. But that's fine. The goal stays the same: decouple domain logic from infrastructure relied code to make the core logic testable.

### Hire people who are good at communicating

Communicate with humans is harder than communicate with computers. Let's assume it: people have their own understanding of each word depending on their background, computers don't.

A keyword has a given behavior depending on the programming language. Words have many senses and drive different reactions from people hearing or reading them. Look how difficult it is to give feedback. People suck at giving feedbacks either positively or negatively.

Programming is a matter of doing actual things, produce value. **Good communication leads to a better system able to produce more value.** But the link is indirect.

However, the leverage of good communication between people inside a team and between teams is huge. It's just not so simple to do.

Hiring people good at communicating solves a lot of issues and help people understand each other. It doesn't mean you won't debate anymore. It just means you won't lose time on superficial matters and go straight to the point.

Good communication is not easy to define. But there are at least two things that are required. The first is *communicating*. **People who don't communicate are not good at communication, by design.** It may sound obvious but over-communicating is better than no communication. And most of the software engineers don't even communicate. The second is the ability to synthesize. People often don't like communication and, when forced to do, tends to minimize the time spent doing it but also the number of words. The result is a poor comprehension of their message. On the other hand, some people just don't know how to synthesize and writes long e-mail nobody will read. The result is just as bad as the former.

Companies like Amazon, Google or Microsoft also need people good both at writing maintainable code and communicating. But they already have the bests, that's the reason why they spent a lot of time interviewing candidates on complex algorithm problems. Not because it is the guaranty of recruiting the best engineers. **Focusing either on tech specialties or on the capability to solve theoretical problems won't help you.** People able to write clear code and communicate clearly about the value they have delivered or not (and why not) is a much better option in the long run.