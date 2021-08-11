# Project guidelines

- [Project guidelines](#project-guidelines)
  - [Disclaimer](#disclaimer)
  - [Don't be a Tommy and don't let others be either](#dont-be-a-tommy-and-dont-let-others-be-either)
  - [Make your branches/work visible](#make-your-brancheswork-visible)

## Disclaimer

This document will likely be ever-evolving, because while there may be opinions they are much harder
to source for project management than they are for development work.

## Don't be a Tommy and don't let others be either

When a team member expresses confusion about something, i.e. they say "I was reading this code and
I found it hard to follow", under no circumstance should your response be anything close to
"No, it's not [hard to follow]". Any and all concerns about code readability and understanding
should be taken seriously unless there is a clear pattern of complaining just to agitate.

This also extends to similar scenarios. A team member and Tommy are discussing (in a meeting):

Member: I think short variable names make it needlessly hard to understand the code when there are
good names you could name the variables. If something is an insurance policy, naming it `ip` seems
pointless.

Tommy: Short variable names don't matter when you have good types.

What Tommy did above is not outright say "No, it's not", but his response is essentially the same.
He's saying that the concern is not real and we should just move on. Tommies in your team are toxic
to all discussions about the code and will almost surely lead to a decline in code quality because
discussions about code quality will stop happening.

The correct thing is always to ask follow-up questions that illuminate exactly what the alternative
to the complained-about code is and what the issue with the submitted/existing code is in more
concrete terms. Short variable names are unreadable because they require you to absorb all
surrounding context in order to see what they mean and the team member likely would have been able
to articulate this. In this particular example the only correct move from Tommy's side would've been
to **fix the code**.

## Make your branches/work visible

No one will magically know that you have work done on something unless you make a pull request. You
can make it a WIP ("work in progress") PR or use a service-specific "Draft" PR status to make it
clear that it's not yet done. Making things visible will:

1. Give people in the team a chance to look at your code for learning and discussion purposes.
2. Notify people as early as possible what you are going to be changing and how. Even if you are not
   done with what you're doing, you'll signal the theme of your changes so far.
