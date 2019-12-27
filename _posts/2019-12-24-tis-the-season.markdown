---
layout: post
title: "The season for code"
date: 2019-12-24 16:00:00
comments: true
categories: Christmas python architecture empathy devops
---

It's currently Dec 24th, 2019 and Santa is getting ready to make his rounds.  While presents and overeating are great, this time of year is also a time for reflection, and with that in mind, I'd like to share my thoughts about software development and empathy and hopefully make a convincing argument that having empathy will make you a better developer.

Software developers are constantly being tasked with empathizing with others whether they know it or not.  In the workplace there are generally three categories of people, that developers, need to empathize with on any given day.

1. Themselves
2. Coworkers
3. Customers

All three of these categories have multiple people functioning in different job roles at different points in their career and existing in the current or future tense.  That last point requires some clarification.  You need to show empathy to these groups in the moment but also take steps that show you are considering the future when making software development decisions.  This may seem obtuse at first, however, entire job functions exist for the sole purpose of aligning current activity with future goals because it can have a large impact on productivity.

Having a deep sense of empathy, and by extension, a deep sense of responsibility to the people surrounding you, is in my opinion, a pre-condition for doing great work.  Additionally, empathy is foundational to maintaining a collaborative organization.  Without empathy, a culture of shortcuts and blame can quickly take root.  In the simplest of scenarios:  

1. Junior manager sets aggressive deadline (a deadline lacking empathy)
2. Developer writes sloppy code (code lacking empathy)
3. Bug finds its way into production (lack of empathy for customer)
4. Senior manager fires Junior manager (lack of empathy for system as a whole) 

If one repeats this pattern enough times, a lack of empathy will quickly devolve into a nightmare work environment.

As a developer you might be wondering about what you can do to break the sociopathic cycle and introduce more empathy into your codebase.  In the spirit  of Christmas, my gift to my readers, is a list of my favorite things for showing software project empathy.  When I encounter a codebase that does all of the following things, that codebase just screams: 

*I CARE ABOUT YOU, MYSELF, AND OUR CUSTOMERS*.

# Test your code

In 2019, this should be obvious but any code that can't be tested, can't really be iterated on.  Lack of automated tests is a special kind of negligence. Lack of testing is a very clear indication that you are only thinking about yourself in the present moment.  

That said there are some real challenges to fully testing your code. Testing external services is usually what trips people up the most.  I've found that for these situations it's best to make liberal use of mocks.  At my company [agero](https://www.agero.com/) we use the following languages with the following mocking libraries.

1. Python [unittest.mock](https://docs.python.org/3/library/unittest.mock.html), [moto](http://docs.getmoto.org/en/latest/)
2. Ruby [rspec double](https://github.com/rspec/rspec-mocks)
3. Javascript [sinon](https://sinonjs.org/), [rewiremock](https://github.com/theKashey/rewiremock)

Given these great testing libraries, 2020 sure is looking bright.

# Static code analysis

Any modern software shop should be using a number of static code analysis tools.  These tools help keep your codebase clean uniform and reduce security vulnerabilities.  A clean organized codebase implies that the person working on that code is thinking about there own and others future mental health.  Just as too much clutter and disorganization in the physical world causes stress, so too, does clutter and disorganization in the digital world.   These are a few of the tools that have helped folks at my company get a handle on code clutter.

1. Python [flake8](http://flake8.pycqa.org/en/latest/), [black](https://pypi.org/project/black/), [sonarcloud](https://sonarcloud.io/)
2. Ruby [rubocop](https://github.com/rubocop-hq/rubocop), [codeclimate](https://codeclimate.com/quality/)
3. Javascript [eslint](https://eslint.org/), [eslint-airbnb](https://www.npmjs.com/package/eslint-config-airbnb)

# Document your code

This one is simple.  Write comments for your code that clearly explains what it does.  Additionally, for languages like python, you can use inline docstrings that can easily be rendered to html.

```python

"""My Module"""

class Foo:
    """My Class"""

    def __init__(self):
        """My Constructor"""
        pass

    def add(self, a, b):
        """Return a + b"""
        return(a + b)
```

Along these same lines, use variable names for magic numbers and strings.  For example, the following code requires the reader to understand what the number 25 means.

```python
def run():
    day = calculate()
    if day == 25:
      return result
```

The following code is a bit longer but clearly explains through variable naming what the number twelve means.

```python
CHRISTMAS=25

def run():
    day = calculate()
    if day == CHRISTMAS:
      return result
```

# Write (a little) less code

This is tricky and taken to an extreme, writing less code can make your code unreadable, so don't go too far.  Less code means less to manage and less bugs for others to fix in the future which shows that you are thinking about others as you develop a solution to your current problem.  Three strategies can help you write less code.  

1. Use stable, versioned libraries.  
2. Write idiomatic code that takes advantage of the standard library.
3. Use stable managed services instead of rolling your own (using stable managed services also shows empathy to operations folks).

#### Example of non-idiomatic ruby
```ruby
data=[1,2,3,4]

def square_list(d)
  new_list = []
  for i in d do
    new_list.push(i*i)
  end
  return new_list
end
```

#### Example of idiomatic ruby

This second example is easier to understand and simpler to maintain.

```ruby
data=[1,2,3,4]

def square_list(d)
  d.map { |i| i * i }
end
```

# Write infrastructure as code

Writing your infrastructure as code instead of clicking through screens shows empathy for others, by making it clear, that you realize they are human, and might accidentally change the wrong config setting otherwise.  Coding infrastructure means anyone can look at that configuration, propose changes, and reason about how those changes may impact other systems.  

This practice shows that you care about the mental health of everyone that needs to have a working environment.  Some tools we've used at [Agero](https://www.agero.com/) to great affect are:

1. [serverless framework](https://serverless.com/)
2. [helm](https://github.com/helm/helm)
3. [cloudformation](https://aws.amazon.com/cloudformation/)

# Extract reusable libraries

No matter the size of the team, it is a good idea to identify common patterns in your codebase and pull them out of that codebase into an installable, reusable, module.  Every major language comes with some form of package management in order to extend the standard library (npm, maven, rpm, pip...) so the tools for sharing are built in.  

Extra points for open sourcing anything your company allows.  Allowing others to reuse your work shows empathy for another persons time.  By giving someone else work you may have spent weeks perfecting, you are saving them the trouble of doing the same.

# Adhere to known interfaces

This is something I am a firm believer in and I believe shows great empathy for the person tasked with consuming your interface.  Put another way; extend and wrap.  Don't reimplement if an existing standard is already in place. 

If you extend an interface without breaking existing functionality, you remove the burden of someone using your software interface having to write special integrations into their codebase.  Recently, my team wrote a common logger that is now being used across the organization.  

I believe that the reason it was adopted so readily by teams is that the interface remained that of the standard logging library while only changing the output as required by our logging specification.  Had we not created this shared module with a stable known interface, every team would have had to implement the interface individually wasting weeks of time or worse they spec would be ignored altogether.

As another example, my team is currently developing a solution that we can use to proxy to 3rd party API's while extending functionality where it makes sense. The result to developers is that they will be able to use their existing tools to interface with our API and we'll be able to add value as appropriate.

# Cache like it's your money

Caching is foundational to efficient computer systems.  Efficient computer systems means customers get information faster, systems do less work, causing them to use less energy (empathy for mother nature), and often end up costing less than than uncached systems.  Caches do create an added layer of complexity, that left unaddressed, will reduce your overall empathy score for the system.  But, through clever cache invalidation approaches you can eliminate much of the headache.

Rails uses [russian doll caching](https://blog.appsignal.com/2018/04/03/russian-doll-caching-in-rails.html) to help reduce complexities associated with cache invalidation. Fastly, uses a technique called [surrogate keys](https://docs.fastly.com/en/guides/getting-started-with-surrogate-keys) to greatly improve efficiencies with cache invalidation.  Lastly, simple tools like [memoization](https://www.geeksforgeeks.org/memoization-using-decorators-in-python/) can help you cache the result of a method call that would otherwise yield the same result.  There is no one size fits all, but cache invalidation strategies are often worth getting right in order to make caching easier to use in your project.

# Mentor based programming

If you are a more advanced programmer, you can show great empathy for your coworkers by helping them level up.  Selfishly, this will help you in the long run as they will produce better more maintainable code, that you may be tasked with adding too, in the future.  Spreading knowledge is a powerful display of empathy that can help others for the rest of their careers.  Some habits that help cement this philosophy include:

1. Detailed code reviews
2. Pair programming
3. Open office hours
4. Lunch and learns
5. Book sharing
6. Technical Blogging ;)

# Use type hinting

This is a simple coding technique that applies to Python, PHP, and perhaps a few other scripting languages.  The premise is simple: Types are hinted in your code by annotating variables.  IDE's and linters can then be used to surface potential data type mismatches in your code.  

Adding type hints to your code doesn't make the code function any differently, but it does help document your code and make is more useable.  Thus, given type hinting is completely optional, it really is an act of kindness to your fellow developers when you include it. The following is an example of code with and without type hinting.

As an aside, parsing data to a typed object is often very useful, I've found [pydantic](https://pydantic-docs.helpmanual.io/) to be extremely helpful in reducing errors associated with parsing JSON documents.

With hinting
```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

Without hinting
```python
def greeting(name):
    return 'Hello ' + name
```

As you can see the difference is minimal, but in larger projects the impact can be huge.

# Named parameters

Named parameters is similar to type hinting in that it documents your code and allows IDE's deliver more value, which save the developer time and headaches debugging code and searching through docs.  Again your code will function without named parameters but will be more difficult for future folks to maintain. Named parameters simply means that you should prefer an actual parameter if you know what it's going to be ahead of time over an array or hash containing similar information. 

In the simplest case I'll illustrate via a python function.

With named parameters
```python
def greeting(firstname: str, lastname: str) -> str:
    return 'Hello ' + firstname + ' ' + lastname


# call this function via

output = greeting(firstname="Jeff", lastname="Durand")
print(output)

```

Without named parameters
```python
def greeting(args):
    firstname, lastname = args
    return 'Hello ' + firstname + ' ' + lastname

# this function via (no idea what the parameters represent)
output = greeting(["Jeff", "Durand"])
print(output)
```

A codebase that passes around a bunch of loosely understood data-structures would quickly drive other developers crazy.  This craziness could end in any number of ways, but I'd like to think named parameters helped the next developer looking at it avoid a lifetime of isolation and alcoholism after slowly losing her mind.

# Avoid long Regular expressions

[This article says it all](https://blog.codinghorror.com/regex-use-vs-regex-abuse/).  Regular expressions can be great when you are looking to filter out a little text or perform some simple validation.  But very quickly they get out of hand and when they do, they are special kind of hell.  They are impossible to reason about and often very difficult to fully test since matching can be nondeterministic. In general pre-filtering and tokenizing strings before running them through a regex, can greatly simplify things and make your code much easier to reason about.

# Use docker for development environments

I find the usage of docker for development environments to be a very empathetic choice in technologies.  The idea is simple, developers can use pre-baked development environments instead of building one from scratch.  But docker goes a step further than that because it is also a cross platform technology.  I realize it might not be perfect in windows, but it does work, and given you can get docker working, you can generally execute all the required code for a given dev environment.

Docker makes it clear that you don't expect another developer to have a full understanding of the entire environment required to run the project.  That understanding and thoughtfulness makes others more productive including yourself. 

# Merry Christmas

All of these things on my list required empathy and also improve software projects measurably.  So this year, think of someone else for a change!!

To go fast, go alone. To go far, go together.  

Happy Coding in 2020
