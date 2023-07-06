---
title: "How Meetic Hires Software Engineers (part 2) : The Hiring Process"
description: "Meetic's hiring process for software engineers consists of three interviews: a screening call, a technical and collaboration skills interview, and a delivery management interview. The technical interview includes a system design exercise to assess the candidate's proficiency in design patterns and ability to conceptualize features with their team. The delivery management interview focuses on collaboration, delivery, and leadership skills to determine if there is a fit with the company culture."
date: 2023-07-06T12:49:19.891Z
author: Sebastien Le Gall
categories:
  - organization
  - management
keywords:
  - engineering organization
  - hiring
  - Meetic
  - organization
  - interview
lastmod: 2023-07-06T12:58:37.991Z
---

In part 1 of this series about hiring, I described the key principles that drive our hiring process, as well as the competencies we evaluate and challenge. In a nutshell, our principles are:

- We hunt for and hire T-shaped engineers
- Our hiring process is short
- We avoid biases
- We prioritize system and software design over coding
- We prioritize collaboration and self-organization over charisma and storytelling skills
- We prioritize hiring engineers with the right mindset for the team: either tech-minded or product-minded people
- We hire to minimize turnover
- Whenever there is any doubt, there is no doubt

In this part, we'll take a deep dive into the hiring process. The goal is to give a concrete example of how we apply those principles. There are likely many ways to design an interview process that fits both our values and the needs of the company or organization. However, starting from something is often a good approach. The aim of this post is to inspire other tech leaders who may need to structure their approach to hiring and to help future Meetic applicants prepare for the interviews.

Overall, our hiring process consists of three interviews:

- The first screening call
- A technical and collaboration skills interview
- A delivery management interview

Each step helps us gain a better understanding of how the applicant thinks, their previous experience, and their potential for growth at Meetic.
<!--more-->

## Interview #1 : The screen call

The initial stage of Meetic's hiring process involves a screening call with the People team. During this call, the applicant's background and previous experiences are briefly discussed, and a quick evaluation of their hard and soft skills fit is conducted. Additionally, the applicant's compensation expectations are discussed to ensure alignment with the company's standards and avoid investing time interviewing candidates who do not meet these expectations.

This step is also an opportunity for us to evaluate some basic soft skills such as communication, collaboration, and the ability to contribute to product thinking by demonstrating an interest in the product and the business.

## Interview #2 : Technical skills and collaboration evaluation

This step is conducted by a tech lead and the hiring engineering manager. The interview contains three parts.

### Part 1: A 20-minutes presentation of Meetic

This includes who we are, what our business is, and how it works. We will also cover how we are organized, including our leadership, teams, who is responsible for what, and how people interact. Essentially, this presentation is a summary of what was described in the previous post about how Meetic works. But we also share an overview of our technical architecture and the technology we use in each piece of software.

We start by sharing how Meetic operates for two reasons. First, we assume that the interviewer has read the candidate's CV, which means that at this point, we know more about them than they know about us. **I think it is important to maintain a balance of knowledge in the interview process so that people know as much about what they are getting into as we know about who is about to join us.**

### Part 2: A 20-minutes discussion about the engineer's previous experience

This section focuses on the applicant's past experiences. The Meetic presentation is a way to start the conversation.

> Have you ever used such technologies?
Did you ever work in such an environment?
I see that you've worked in a Scrum team. How were you collaborating with the product owner?
>

While the CV and screen call provide some insight into the applicant's skills, it is during this discussion that we gain a deeper understanding of how they used technologies like ReactJS, PHP, or distributed computing with Spark. We seek to evaluate the applicant's level of understanding of these technologies and how they were involved in using them.

**We also try to understand what is often not described on an engineer's CV: collaboration, ways of working, methodology, etc.** This part is also dedicated to asking questions about the team the applicant was working with, how they work together, and what kind of product they were developing.

### Part 3 : A 45-minutes system design exercice

The last and most important part of the interview assesses the candidate's system design skills. As mentioned earlier, we look for engineers who are proficient in design patterns and can conceptualize features with their team.

To evaluate this, we have designed a simple exercise: The product manager presents a feature to build (an actual feature we have already built). The engineer must design a technical solution to implement this feature.

During the design process, the engineer needs to demonstrate the following:

- **How they will organize their code into separate classes, namespaces, or modules**
- **How the different parts of the code will communicate with each other**
- **What kind of data structures will be used**
- **How the code will be tested**

There are often no universal solutions. Instead, we encourage conversation and expect the applicant to explain:

- What they are thinking about
- What they plan to do
- How they argue for their architecture choices

Then, the tech lead and hiring manager will ask questions, suggest other ways of doing things, and inquire whether the candidate has considered them. If yes, why did they choose not to use them?

I have personally conducted more than 40 interviews using this exercise and found it to be very accurate. Quickly, we can determine whether or not the applicant will fit our needs.

Until now, none of the software engineers I have interviewed using this exercise have failed to meet our expectations once hired and onboarded onto the team.

The success of this exercise can be attributed to the many benefits it provides.

**First, the exercise challenges the applicant's knowledge of concepts and design patterns.** We expect the applicant to have not only theoretical knowledge but also the ability to use them in concrete use cases. At Meetic, our software engineering principles rely heavily on decoupling patterns and most of the concepts defined in [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design).

**Additionally, since the applicant has to share their reasoning, we can clearly see what kind of thinker they are.** We can distinguish between:

- Engineers that start by spending a long period of time asking questions to get more details and then start designing the complete solution in a straightforward manner.
- Engineers that start by drafting the happy path or the more obvious things and then iterate.
- Engineers that are kind of lost and aren't able to structure their approach to the problem.

**Moreover, the conversation that starts after the applicant has done a first draft of their design is key to evaluating collaboration skills.** Some engineers don't accept feedback at all, while others have such imposter syndrome that any question we ask tends to change their mind. Some make design choices only because "that's how I do things in my current job/project," but when we ask if they would do things differently if it were their choice, they have different views.

### The system design exercices in detail

I won’t be exaustive on the topic of the system design exercice if I don’t share it.

Then, let’s see what kind of feature we propose the applicant to design. We differentiate 2 cases :

- frontend engineers  work on designing a user interface
- backend engineers work on designing an api

#### Frontend exercice

The goal of this exercise is to replicate two main components of the "Discover" page in Meetic's mobile apps.

![frontend_exercice.png](/images/post/how-meetic-hire/frontend_exercice.png)

Upon launching the application, users are greeted with a first screen featuring two lists of profiles. Clicking on a profile thumbnail takes the user to the second screen, which displays the profile in full.

Both profile lists are provided by a backend which returns an array of profiles in the following format:

```json
[
  {
    "id": 18343224,
    "nickname": "Emily",
    "age": 29,
    "gender": "F",
    "city": "Derby Center",
    "picture": "https://images.unsplash.com/photo-1579503841516-e0bd7fca5faa?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=400&fit=max&ixid=eyJhcHBfaWQiOjE0ODUwfQ",
    "online": true
  },
]

```

On the second screen, there are two buttons: a heart and a cross. The heart moves the user to the next profile in the list, while the cross moves to the next profile and removes the current profile from the list.

If the user chooses the cross button on a profile and then returns to the first screen, that profile should no longer appear in either of the lists.

**This exercise covers topics such as:**

- **UI design (positioning elements in the lists)**
- **UI component factorization (titles, thumbnails)**
- **Business logic factorization and decoupling (how to filter the profile list)**
- **Caching and all forms of state management**

#### Backend exercice

In this exercise, the task is to design a RESTful API as part of the picture microservice. This API validates/moderates an uploaded picture and is responsible for handling the picture's bounded context.

![backend_exercice.png](/images/post/how-meetic-hire/backend_exercice.png)

The API has the following interface:

```
POST /pictures/
{
    "userID": 123456,
    "picture": "ZERTYUIOIDFGHJKOUYTRERTYUJ....." //picture blob
}

```

For a profile picture to be considered valid, it must contain only one face and be moderated by a human through a back-office. To make the human work more efficient, the API relies on another API that detects the number of faces in a given picture using machine learning or other methods. Thus, only pictures containing one face are sent for human moderation.

Human moderation is performed on a back-office that calls the picture microservice to update the moderation status of the picture.

Thus:

- If the picture contains more than one face, the user should be notified synchronously. For example, by returning a 400 status code.
- If the picture is not considered valid by the moderator, an email should be sent to the user. This is done through another API that is responsible for sending emails.

**This exercise covers topics such as:**

- **Decoupling business logic from interfaces (either HTTP or database)**
- **Data structure and storage**
- **Asynchronous patterns such as event-oriented architecture**

## Interview #3: Deep Dive into Delivery and Collaboration Evaluation

This is the ultimate interview, led by an engineering director and HR team member. During the interview, we focus on collaboration, delivery, and leadership skills.

We explore the candidate's past experiences by asking about:

- **Teamwork and organization**: How do you collaborate with others and maintain organization?
- **Collaboration with product and design teams**: What is your process for working with product and design teams?
- **Self-management**: How do you stay on track with your work?
- **Work breakdown and planning**: How do you plan your work and break it down into manageable tasks?
- **Accountability**: How do you hold yourself accountable?
- **Ability to take initiative**: Can you share an example of a time when you took initiative?
- **Willingness to challenge the team's work methods**: How do you challenge the team's approach to work?

As we continue the discussion, we will be analyzing soft skills such as:

- **Communication**: The ability to communicate effectively, clearly, and concisely.
- **Knowledge sharing**: Consistently contributing to your team's and community's documentation.
- **Teamwork**: Consistently helping your teammates overcome obstacles, resolve blockers, and complete tasks.
- **Relationship building**: The ability to build strong relationships with your teammates, manager, and product counterpart.

This step is key to determining if there is a fit with the company culture. We challenge the applicant's ability to work in our environment, with our teams, and in accordance with our approach to tech strategy and product strategy.

During this interview, we also typically gauge whether the applicant is more product-minded or tech-minded. I like to ask open-ended questions such as:

> Which company's style do you feel more aligned with: Amazon or Apple? From a customer's point of view, Amazon tends to ship features and products with a sense of "minimal viability" in mind. We often see Amazon shipping features that may look ugly but get the job done and are cheap. In contrast, Apple tends to spend a lot of energy on R&D. They rarely ship new products, but when they do, every detail has been meticulously thought out.

Also, this is a moment for the candidate to learn more about how we work. I spend some time answering questions about how we manage technical debt, how we shape features collectively, how we do code reviews, and how people interact with each other. As previously mentioned, in the end, neither the applicant nor we should be surprised when the collaboration starts. Since we tend to gather a lot of information about the applicant during the process, I see this last phase as a key opportunity to bridge the gap.

## Debrief and offer

At the end of the hiring process, each interviewer participates in a debriefing meeting that typically lasts 30 to 45 minutes. During this meeting, we attempt to answer questions such as: Is the candidate's personality a good fit for the team and the community? Is the candidate more product or tech-minded? If tech-minded, is he/she senior enough technically? What are the candidate's strengths? Technical skills or soft skills?

Most of the time, I have already received feedback from the Engineering Manager after the technical interview. Sometimes, we also decide not to move forward with the final interview, usually when the candidate's required technical skills are not at the right level.

For candidates who make it to the end, we walk around the table to give everyone a chance to share their thoughts. We generally align on three things:

- **Does the applicant meet the expected level of skill and experience?**
- **Does the applicant have the right mindset to join the team and community?**
- **What are the compensation expectations and offer?**

The last two parts are essential and take most of our time. We try to anticipate how the candidate will evolve in our ecosystem and what kind of growth opportunities we can offer in the mid-to-long term.

And of course, there is the matter of compensation. We compare the expectations with the skill level and align on whether or not they are coherent. This part may be a deal breaker. Typically, we won't hire people who overvalue their skills and ask for compensation that is not correlated with their current level. We also check the compensation expectation against the current compensation level in our teams to ensure equity. This is also a deal breaker. We typically won't make an offer if the compensation expectation exceeds the average (or new target, if applicable) by too much. However, we are usually able to spot this in the first screen call.

Overall, the process takes one to two weeks depending on availability. We try to debrief as soon as possible once the last interview is complete.

When there is a strong "yes," we send an offer either the same day or the day after. When the answer is "no," we still take the time to call the applicant and provide feedback.
