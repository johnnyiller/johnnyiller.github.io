---
layout: post
title: "Quick guide to command line oriented github team workflow"
date: 2015-8-26 18:00:00
comments: true
categories: git github workflow commandline
---

This is another article inspired by my work at the [thefirehoseproject](http://www.thefirehoseproject.com/).  To be clear, the firehose team does a great job of laying out a team workflow via an advanced git topics module that they provide.  This guide is meant for people *not taking thefirehoseproject* course or for those taking it that want a quick refresher along with some additional tips and tricks.

First a brief overview of the workflow that the majority of teams should be using on github.  This flow assumes you are currently on the master branch with no local commits and you want to start a new session of work on a feature branch?

- git pull origin master
- git checkout -b NEW_BRANCH_NAME
- make changes via text editor
- git add . --all
- git commit -m "COMMIT MESSAGE"
- git pull origin master
- fix conflicts
- if anything changed type: git commit -m "merging from master"
- git push origin NEW_BRANCH_NAME
- create a pull request (can be done on command line, keep reading)

Some questions you may have after looking at this workflow.

### 1. Why do we do pull origin master twice in the workflow?

The first pull from master is to get your code up to date for the start of your work session.  Nothing new here.  The second *git pull origin master* is to reduce the liklihood of a merge conflict from your pull request.  Remember you are on your feature branch so the pull will actully perform a merge from the remote branch into your local feature branch.

### 2. How do we fix conflicts?

Git will tell you the files that are conflicting when it attempts to merge.  Take note of the output and inspect the files that are conflicted.  Correct the conflicted files and create another commit with the properly edited files.  This is probably the trickiest process when working with a team.  It's important that you don't immediately remove the changes your teamate made without fully understanding what the implications are.

### 3. Why do we need to commit again if there are changes?

One artifact of merging code in Git is that if you merge and there are any changes, even if you didn't make the changes then you will need to create another commit.  So if your teamate changed something there would be a commit for the original change and another commit when you merge the code into your working copy.

### 4. How do we make a pull request?

Typically this will be done by logging into github and clicking the button to create a pull request.  Pull requests are a feature of Github not Git itself which is why it isn't built into the Git CLI.  Either way, After 300 pull requests or so, the process becomes tired.  The solution is to install *hub* and link it to your git CLI tool.


#### Steps to install (ubuntu linux)

From the command line, assumes you are using bash... If you are using something else you probaly don't need this tutorial.  

*If you are using a mac your might need to use .bash_profile instead of .bashrc*

{% highlight bash %}
sudo gem install hub
# enter your password when prompted
echo 'eval "$(hub alias -s)"' >> ~/.bashrc
source ~/.bashrc
{% endhighlight %}

after doing this you should be able to type the following on the command line and get some output other than command not found.

{% highlight bash %}
hub 

# output
[-p|--paginate|--no-pager] [--no-replace-objects] [--bare]
[--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
[-c name=value] [--help]
<command> [<args>]

Basic Commands:
   init       Create an empty git repository or reinitialize an existing one
   add        Add new or modified files to the staging area
   rm         Remove files from the working directory and staging area
   mv         Move or rename a file, a directory, or a symlink
   status     Show the status of the working directory and staging area
   commit     Record changes to the repository


# Assuming all goes well you should now be able to type the following from your feature branch.

git pull-request

# the first time you run this command it will ask for your username and password.
# after authenticating you shoudl not have to do so again in the future.

{% endhighlight %}

This will open up a text editor and give you a chance to write a message about the pull request. Once you save the message the pull-request will automatically trigger on github.  *Hub* has a bunch of other commands that make using github from the command line super smooth.  I encourage you to explore them to get a better sense of the value of the tool.

Check out the hub [README](https://github.com/github/hub) for details on all the useful utilities this gem gives you.

That concludes my short guide.  Feel free to leave me a comment should the urge strike you.

