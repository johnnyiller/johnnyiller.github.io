---
layout: post
title: "Employee Turnover"
date: 2019-8-9 16:00:00
comments: true
categories: Organization Architecture
---

Exciting stuff right?  Most managers understand that turnover can be disruptive and expensive.  The cost to recruit, train, and provide a competitive wage is typically one of an organizations highest costs in the knowledge economy.  Failure to address employee churn and its many different causes can create a downward spiral, where the churn rate, keeps increasing until the organization becomes a revolving door of employees.  

First let's level set; I'm a technologist, software developer, and systems architect.  So it may be valid to ask what business I have commenting on organizational &amp; business issues at all.  One might argue that these discussions are better left for the MBA folks and Accountants. Fair point, let me first lay out my reasoning around writing an article like this at all.

## [Conway's law](https://en.wikipedia.org/wiki/Conway%27s_law)
> organizations which design systems, are constrained to produce designs which are copies of the communication structures of these organizations.<br/>
> <em>Melvin Conway</em>

Melvin was a smart person and very influential in the early years of computer science.  If you buy into Conway's law (which I do), it's relatively easy to make the logical leap to the following axiom.

<b>organizational and technical architectures are inseparable</b>

While they are not the same thing, I reason that one simple can't have a solid technical architecture without a good organizational architecture, if one is producing technologies within the constraints of an organization whose size is greater than that of a single team.

Thus, I believe that offering my perspective around organizational architecture is not only valuable, but a core part of my responsibility as a leader in the technology field.

## Types of turnover

I believe their are at least two types of turnover that commonly occur at any organization.  They are:
1. External
1. Internal

### External

Most folks know what external turnover is, so I won't spend a ton of time on it.  External turnover is when a team member decides to leave the company for a number of reasons, including but not limited to:

1. Life changing event
1. Lack of upward mobility
1. Underpaid for level of responsibility
1. Burnout
1. Cultural mismatch leading to infighting
1. Feeling like work doesn't make a difference in the world
1. Loss of faith in senior leadership
1. Internal Turnover

For example, a life changing event, might include a new child or health issue that needs tending too for example. Burnout is usually caused by unrealistic deadlines and too much routine work, mixed with unhealthy work life balance.  Cultural mismatch, is typically the product of differing philosophies about the value of the product being produced, and the best way to produce it. Lastly, a loss of faith in senior leadership, can lead employees to sense the storm about to hit and trigger their flight reflex.

Entire books have been written about reducing external turnover, so I won't bore you with my take on it.  Instead, I'll focus the rest of this article on Internal turnover, with the goal of explaining what it is, how it can damage a team, and how to reduce it. 


### Internal Turnover

Internal turnover is primarily attributable to chaotic internal struggles, muddy product vision, or even too much external turnover. Both internal and external turnover turnover are related and reinforcing.  Teams are broken apart and moved around frequently, as management tries to solve a list of real or imagined problems.  This movement, makes it difficult for teams to reach peak performance. 

Team forming, and cohesion take a lot of time and effort.  [Bruck Tuckman](https://en.wikipedia.org/wiki/Bruce_Tuckman) was a famous psychologist who studied and wrote extensively about group dynamics.  His most notable work, known as [Tuckman's stages of group development](https://en.wikipedia.org/wiki/Tuckman%27s_stages_of_group_development), identified four key stages of group development that teams typically go through in order to become productive.  The following graphic provides some context around the process.  Note: you can't skip stages, you must progress linearly.  This means that teams that are constantly changing are constantly being "reset".

![GroupDynamics]({{ site.url }}/assets/diagrams/TCTC-Forming-Storming.png)

In Ray Dalio's book [Principles](https://www.amazon.com/Principles-Life-Work-Ray-Dalio/dp/1501124021) he outlines how important it is for teams to get in sync.  His core principle, states that everyone doesn't have to agree with one another, but that everyone does need to understand what's happening and be able to argue for or against it and eventually fall in line when a decision is made.  The relevance of this insight is that if teams are continually changing it can be next to impossible to get in sync and can thus be impossible to produce anything of lasting value, because teams spend all of their time re-litigating things that have already been decided.

Both Tuckman and Dalio speak to the idea, that if teams are constantly changing, they can never reach the performance stage and will be doomed to failure. To make matters worse, a common "remedy" managers often try to apply to improve teams that have not reached the performance stage, is to try and add more humans, either from within the company, or as new hires.  The result is illustrated in the following diagram.

![GroupDynamicsProcess]({{site.url}}/assets/diagrams/teamturnover-color.png)

As you can see, the addition of new team members over time could have the result of keeping the team locked in a constant state of <b>forming</b>.  When teams are stuck in the earlier stages of development, they are unable to function on their own, and will require much more direction.  If managers are unable to move on to other teams, the organization may fail to scale properly, as teams will always need a dedicated hands on manager.  Ideally, managers would be able to manage multiple teams, but may be unable too if teams can't progress to the performing stage.

Adding more people to a team that hasn't reached the performance stage yet, my also result in something known as [The Mythical Man Month](https://en.wikipedia.org/wiki/The_Mythical_Man-Month).  The Mythical Man Month states that; adding another person to an already late project will just make it later.  

The notion that you can just add another person and the project will be completed on time, is a hold out from the industrial age.  In the industrial age, adding another worker to a factory production floor would increase output and help fulfill an order that was running late.  In the knowledge economy, this is no longer true, since communication is at the center of all valuable work being done by the team.  Ultimately, when managers add another person, they increase the communication complexity and thus slow the overall team down, preventing the team from reaching the performance stage, and completing the project.

My own take on this subject, is that, years of evolution has conditioned humans to view outsiders as inherently less trustworthy than insiders.  Groups that are frequently having to assimilate new members, have to spend enormous amounts of time building trust, which can slow down the workflow and ultimately lead to even the top performers becoming frustrated.  This frustration will likely lead to top individuals leaving the company, which has widely been documented as an high [expense for companies.](https://www.peoplekeep.com/blog/bid/312123/employee-retention-the-real-cost-of-losing-an-employee).

To restate the core problem simply: 

<b>If teams are changing membership frequently, the team will never reach peak performance, leading to top performers jumping ship, which will doom the organization to a lifetime of substandard results.</b>

![GroupCycleDynamics]({{site.url}}/assets/diagrams/team_dynamics.png)

Hopefully, I've convinced you that high <b>internal turnover</b> is a real thing and when it's high, it can cost organizations dearly in the form of increased external turnover and sub-par team performance.  If you do not agree with my premise, you may stop reading here, I won't be offended.

![HowFar]({{site.url}}/assets/diagrams/bluepillredpill.png)

If you are reading this sentence, I'll assume you'd like a few pointers on how to limit <b>internal turnover</b> and see your organization thrive.  This list isn't meant to be exhaustive so please feel free to leave a comment, if you think of other ways to reduce internal turnover rates.

## Tips for reducing Internal Turnover

### Change Projects not teams

Often, it is better to let a high performing teams take complete ownership of a new project than it is to try and stitch together a new team for a new project.  Often this will lead to some members of the team having less than perfect skills for the new project.  This should be viewed as a growth opportunity for those individuals and they should be required to learn the new skills.  This style of team development is at the core of the agile development process.  Teams adjust the project requirements and their skillset over time rather adjusting the personnel.

The core principle is that it's easier and thus more cost effective to learn new skills, and assimilate them, than it is to assimilate new team members.  Learning new skills is a win for everyone at the company. Conversely, changing teams, and as a result, being forced to go through the stages of group development again is costly and time consuming with no direct payoff for the employee or the organization.

Tasking a stable team with a new project is much preferred, to breaking up the team, but this too can be disruptive if it's done too often and without solid reasoning.  Moving from half finished project to half finished project can eat up large chunks of a teams time and make them feel like they are never really adding value to the organization.  Most projects have periods of lesser activity; if possible, use slower periods to switch projects if possible as it will be less disruptive and deflating to team moral.

### Hire learning generalists in favor of specialists

Learning generalists can complete many different tasks depending on what the project calls for. They are usually inquisitive by nature, which allows them to quickly pick up skills they might be lacking.  This makes it easy to task a team of generalists with a new project, removing the need to break teams apart in the first place, as project requirements change.  

Generalists will likely have to learn less overall than a team of specialists, since good enough knowledge of a subject is often good enough.  Secondly, because generalists have a wide range of knowledge they will have many common interests with one and other, which will help them socialize and form tighter bonds with each other more quickly (birds of a feather flock together).  

Many of today's tasks in the workplace are tasks of integration, not invention.  These types of tasks require folks to process a large amount of information, socialize it to others, and integrate it into a complete picture.  Generalists tend to be much better at this, as they have a wide palette from which to paint their solutions from.  

Obviously, if you are trying to solve a very specific problem, then it may be worth hiring a specialist but I would advise all hiring managers to consider if a specialist is really needed or if it makes more sense to hire a generalist that could learn to specialize in the subject area of interest.

### Hire entire teams

If there is a project that demands a specific sized team, try to hire everyone for the project or form the team in one round of hiring.  Interview for all positions simultaneously and try to onboard the team as a unit to avoid being reset when each new member joins.

Hiring entire teams can be a great strategy for organizations with the resources to purchase smaller, high preforming, undercapitalized startups.  Often the team that makes up a startup is of much higher value than the market share of that same startup.  If done correctly, large organizations can jumpstart their team building efforts by purchasing a company.  It's a nice bonus if the company can provide a symbiotic product offering, but that may not even be required if the price is right.  

Be careful when purchasing a small startup to make sure a majority of people on the team are on board with the purchase and willing to work for the new company.  If the new team is combative with the parent company, this strategy could be more trouble than it is worth.

### Avoid mixing outsourced with in-sourced

At a very basic level, team members should have aligned incentives.   Without aligned incentives it can be extremely difficult to get team members to work together cohesively.  This can be hard or impossible to achieve in an environment where there are two different classes of employee, with different incentive structures, and different bosses; working on the same project.  

Often in-sourced employees are more permanent than outsourced.  This means that in-sourced employees often want to create things that not only meet today's requirements but also allow for change and the meeting of future requirements.  

Outsourced employees are often tasked with spending only the minimum amount of hours to complete the task they have been assigned, after which, they can be assigned to a new team, project, or even another company altogether.  The turnover of the outsourced employees can throw off they dynamic of the in-sourced employees by causing them to constantly be adapting to new personality types and skill levels.  

The tension created from this dynamic can end up leading great employees to seek teams that don't have outsourced team members and when that's not available they may eventually leave the organization altogether.  Either way, the result is increased turnover and lower overall productivity.