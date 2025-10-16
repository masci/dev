---
title: "Accelerate"
description: "by Nicole Forsgren PhD, Jez Humble, Gene Kim"
category:
tags: ["devops", "agile", "management"]
date: 2022-02-03T10:58:25+01:00
slug: "accelerate"
rating: "★★★☆☆"
cover: "/images/accelerate.png"
---

The book consists of two, very different parts, plus a case study from the ING bank. Part one
is a must read explanation of how DevOps practices can massively impact the business in a positive
way and why. The second part is an explanation of the science that supports the claims presented in
Part two.

<!--more-->

Part two might not be extremely interesting for DevOps engineers, still it is written in a way that
can be understood without a strong background in statistics and sociology - I skimmed through the
more technical parts and I enjoyed the rest.

## Notes

### Definition of software delivery capability

Software delivery capabilities can be grouped in five categories:
- Continuous delivery
  - Use version control for everything
  - Automate deployments
  - Implement CI and test automation
  - Use trunk-based development
  - Support test data management
  - Integrate security into software design
  - Implement CD
- Architecture
  - Use loosely coupled architecture
  - Let engineers use the tools of choice
- Product and process
  - Gather and implement customer feedback
  - Make the flow of work visible
  - Work in small batches
  - Foster and enable team experimentation
- Lean management and monitoring
  - Have a lightweight approval process
  - Monitor across app and infra to inform business decisions
  - Check system health proactively
  - Use WIP (work-in-process) limits
  - Visualize work
- Culture
  - Support a generative culture (see Ron Westrum categories)
  - Encourage and support learning
  - Support and facilitate collaboration
  - Provide resources and tools that make work meaningful
  - Support and embody transformational leadership

### The four KPIs of delivery performance

- delivery lead time
  - time from customer request to customer satisfaction
  - time of design and validation + time to release
- deployment frequency
- MTTR (Mean Time To Restore)
- Change fail rate

### Other notes

- Improvements in software delivery are always possible as long as  leadership provides consistent
support (time, actions, resources) and demonstrating a true commitment to improvement, and as long
as team members commit themselves to the work.
- Organizational culture can exist at three levels (Schein 1985):
  - basic assumptions
    - the least “visible” of the levels—and are the things that we just “know,”
    (and may find difficult to articulate) after we have been long enough in a team.
  - values
    - They influence group interactions and activities by establishing social norms, which shape the actions of group
    members and provide contextual rules (Bansal 2003). These are quite often the “culture” we think of when we talk
    about the culture of a team and an organization.
  - artifacts
    - artifacts can include written mission statements or creeds, technology, formal procedures, or even heroes and
    rituals (See Elastic's "Source Code")
- Westrum grouped orgs into four types
  - **Pathological** (power-oriented) organizations, characterized by large amounts of fear and
  threat. People often hoard information or withhold it for political reasons, or distort it to
  make themselves look better.
  - **Bureaucratic** (rule-oriented) organizations protect departments. Those in the department
  want to maintain their “turf,” insist on their own rules, and generally do things by the
  book—their book.
  - **Generative** (performance-oriented) organizations focus on the mission. How do we accomplish
  our goal? Everything is subordinated to good performance, to doing what we are supposed to do.
- Information is key in gnerative orgs and Westrum provides three characteristics of good
information:
  - It provides answers to the questions that the receiver needs answered.
  - It is timely.
  - It is presented in such a way that it can be effectively used by the receiver.
- Westrum’s description of a rule-oriented culture is where following the rules is considered more
important than achieving the mission, there are generative orgs within the public sector for e
example, or startups that are clearly pathological.
- A good culture requires trust and cooperation between people across the organization, so it
reflects the level of collaboration and trust inside the organization.
- In a team with a good type of culture, not only is better information available for making
decisions, but those decisions are more easily reversed if they turn out to be wrong because the
team is more likely to be open and transparent rather than closed and hierarchical.
- Teams with these cultural norms are likely to do a better job with their people, since problems are more rapidly
discovered and addressed.
- The way to change culture is not to first change how people think, but instead to start by changing how people behave
- Implementing continuous delivery means creating multiple feedback loops to ensure that high-quality software gets
delivered to users more frequently and more reliably.
- **What's CD?** When implemented correctly, the process of releasing new versions to users should be a routine
activity that can be performed on demand at any time. Continuous delivery requires that developers and testers, as well
as UX, product, and operations people, collaborate effectively throughout the delivery process.
- our research found that improvements in CD brought payoffs in the way that work felt. This means that investments in
technology are also investments in people, and these investments will make our technology process more sustainable
- we first have to find some way to measure quality. This is challenging because quality is very contextual and
subjective. As software quality expert Jerry Weinberg says, “Quality is value to some person” (Weinberg 1992, p. 7).
- Proxy variables for quality:
  - The quality and performance of applications, as perceived by those working on them
  - The percentage of time spent on rework or unplanned work
  - The percentage of time spent working on defects identified by end users
- How can we ensure that paying attention to security doesn’t reduce development throughput?
  - This is the focus of the second aspect of this capability: information security should be integrated into the
  entire software delivery lifecycle from development through operations.
- Our recommendation based on these results is to use a lightweight change approval process based on peer review,
such as pair programming or intrateam code review, combined with a deployment pipeline to detect and reject bad changes.
- This idea is a form of risk management theater: we check boxes so that when something goes wrong, we can say that at
least we followed the process. At best, this process only introduces time delays and handoffs.
- Technology managers, like so many other well-meaning managers, often try to fix the person while ignoring the work
environment,
- Last but not least, employees must be given the authority to make decisions that affect their work and their jobs,
particularly in areas where they are responsible for the outcomes.
- A latent construct is a way of measuring something that can’t be measured directly.
  - We can ask for the temperature of a room or the response time of a website—these things we can measure directly.
  - A good example of something that can’t be measured directly is organizational culture.
    - We can’t take a team’s or an organization’s organizational culture “temperature”
    - we need to measure culture by measuring its component parts (called manifest variables), and we measure these
    component parts through survey questions.
