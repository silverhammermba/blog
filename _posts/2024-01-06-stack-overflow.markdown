---
layout: post
title: "Stack Overflow is working as intended"
categories: etc
---

I see Stack Overflow getting a lot of hate these days, especially from less experienced programmers.
The common complaint is that the site has a toxic culture that punishes and humiliates newbies, discouraging participation.
That is true, but that is also intended.
Weirdly, this is also what makes Stack Overflow such a useful site.
In this blog post, I will explain why that is as well as how to rake in those sweet, sweet fake Internet points on Stack Overflow.

## Instructions unclear

The welcome page for Stack Overflow says:

> [This] is a question and answer site for professional and enthusiast programmers.

So that means if you have a programming question, you should click the "Ask Question" button, right?

**WRONG.**

This misleading introduction is the primary cause of almost every complaint I have seen about Stack Overflow.
Yes the site is in the format of a Q&A and yes it is programming focused, but there are a million _completely valid_ programming-related questions out there that will get you downvoted into oblivion.
What determines if a question will win you those Internet points or shame you into quitting programming?
The easiest way to know is to first understand what the site _really_ is.

## The true goal

Here is what the welcome page _should_ say:

> [This] is a database of every programming question that can have an objectively correct answer.
> Every such question you could ask has **already** been asked on this site and every reasonable answer has **already** been given.
> All questions and answers are already ranked according to their utility to the programming community at large.
> If you have a question, simply search for the existing instance of it and upvote it, then upvote the answer you found most useful.
> In the extremely unlikely event that such a question has not been asked or answered, you may attempt to do so.

This explains almost all downvotes you will get on the site.
Let's see why.

### Valid questions that get downvoted and why

"How do I learn ABC?" or "Which tool should I use for XYZ?" will _always_ get downvoted.
These questions do not have objectively correct answers since they depend on the individual asking and their ultimate goals, which are both impossible to capture in the question.
Even if they could be answered, the "right" answer would likely change dramatically over time as programming evolves.
This runs counter to the site's goal of being a definitive database of objectively ranked answers.

"What is the bug in this complete program? (Source code attached)" will _almost_ always get downvoted.
It is _extremely_ rare that this has an objectively correct answer.
Usually the expected behavior of the program is entirely unclear from the question and the reader is left to infer that from context clues.
This makes it hard to understand what the bug even is, so it gets downvoted.
Even if the expected behavior is clear, the design goals of the program itself are usually unknown.
Is this a toy project meant for learning?
Is this a throwaway utility that will be used once and never again?
Is this for enterprise software that needs to be understood and maintained years from now?
Does it need to run under specific memory/speed constraints?
_If_ you manage to clear up all of those details in the question you probably won't get downvoted.
If the resulting question ends up relatively short, you _might_ get an answer and upvotes.
Answering these kinds of questions is extremely labor-intensive, so they often get ignored.

"How do I do [COMMON TASK] in [COMMON LANGUAGE]?" will _almost_ always get downvoted; not because it's a bad question, but because there is probably an existing instance of the question with answers!
Want to reverse a string in C#?
Want to invert a map in Javascript?
Great!
**For the love of God do not ask on Stack Overflow.**
First ask yourself, "Is it likely that a whole bunch of people in the history of programming have tried to do something like this before?"
If so, it's _probably_ already on Stack Overflow.
After all, that is the whole point of the site: to be a database of such questions!
If you care about imaginary Internet points, you are much better off spending 20 minutes rewording Google searches until you get a Stack Overflow result than asking a new question.

"How do I do [BIZARRE TASK] in [COMMON LANGUAGE]?" will _almost_ always get downvoted.
These are questions like "How do I delete the fourth instance of the letter J from a string in Go?" or "How do I treat a UDP socket as if it's a `void` heap pointer in C++?"
These questions may have objectively correct answers, but they make the reader question the asker's sanity.
If it seems like no one else would ever benefit from such a question being answered, you get downvoted.
Such questions, even if objectively answered, would fill the database with noise, making it harder to use Stack Overflow for its intended purpose.
Usually these questions are posed by two kinds of people:

1. Students who were given homework.
   Homework contains these bizarre tasks specifically so that students cannot easily Google the answers.
   Filling Stack Overflow with a database of every weird scenario a programming teacher cooked up would qualify as noise, hence the downvotes.
2. Someone suffering from [the XY problem][XY].
   It is more likely that people will search for ways to do X (rather than the asker's weird fixation on Y).
   Thus Stack Overflow punishes questions about Y and rewards only those about X.

[XY]: https://xyproblem.info/

Sometimes, more often than I'd like, you will get downvoted even for a question with objectively correct answers, which has never been asked before, which would be helpful to other people.
Why?
Because it's the Internet and it's full of lazy assholes with poor reading comprehension.
_Usually_ they will leave a comment explaining why.
Don't argue, just edit your question to address their comment (usually requires adding further clarification), and leave a comment @ing them saying their concern was addressed.
Though it may grate on your nerves, doing this is important because potential answerers often ignore questions with downvotes, so the extra effort maximizes your chance of a successful answer.
This also feeds into the site's ultimate goal of becoming the definitive question database since placating the asshole likely resulted in your question becoming even more searchable and even less ambiguous such that any future prospective asker will stumble upon your question before asking.

### Valid answers that get downvoted and why

Answers that just link to some external site which has an answer will _always_ get downvoted.
Remember, Stack Overflow wants to be the complete database itself, so external links are counter to that goal.
An external link could break, making what was once a valuable answer useless.
You can include an external link for extra information, but Stack Overflow always wants the answer to work on its own.

Answers that just include code with no explanation will _almost_ always get downvoted.
Without an explanation of what the code does or why, it is hard to evaluate the quality of the answer.
Very simple self-evident code is sometimes okay.

## How to easily get points on Stack Overflow

First, I do know what I'm talking about.
I currently have 21,453 points, 5 gold badges, and 50 silver badges on Stack Overflow.
So I'm not talking out of my ass.

Remember that it's all about the database: improve the quality of the Q&A database and you get points.
Unintuitively, it's not about asking a lot of questions or posting a lot of answers!

### Fill in Google gaps

If you make what seems like a reasonably clear Google search and get no Stack Overflow hits, and then spend 45 minutes trying different searches until you find your answer, that often indicates an opportunity for points.
Take your original Google search and ask it as a question!
Even if it gets closed as a duplicate, you will still rake in the points in the long run.
Why?
Because chances are someone else will make that reasonably clear Google search, see _your_ question, see that it has answers, and upvote it!
This makes sense even if it gets closed as a duplicate because your more-easily-Googleable question improves the utility of the database!

A lot of the following tips also tie into this same mechanism: make the Google results for Stack Overflow better and you will be rewarded.

### Chase new technology

The main situation where the database is missing questions and answers is for new tech that doesn't have wide adoption.
If you're an early adopter and need to figure out stuff on your own, ask those questions!
Even if you already figured out the answer, ask them anyway (remember that you can answer your own question)!
If the new technology takes off, you'll start raking in points once the Google searches start hitting your questions.

This is actually not _that_ easy because you'll be racing against millions of other programmers trying to do the same thing.
But as the profession expands, you might be able to find a niche where you can be the first to ask.

### Turn comments into answers

For some reason, people love to answer questions in the comments.
This is bad for the database!
Comments have worse visibility and cannot be as easily ranked and voted on as answers.
It may seem cheap, but if you ever see someone answer in the comments, or a highly upvoted answer sucks but has better info in the comments, just steal it!
Consolidate the valuable comment info into a single, cohesive answer and you're helping everyone else see that!
It's better for the database, so you _will_ get points for this.

### Leave clarifying comments

You don't get many points for comments, but it can add up.
Highly voted answers get high traffic by definition, so see if you can improve them, even in little ways.
Is it missing a link to official documentation?
Add it in a comment.
Does a function have an optional parameter that wasn't explained?
Leave a comment explaining the broader usage of it.

### Answer by tag

I almost never look at the front page.
Technically you can get crazy points for answering there due to the high visibility but practically it's extremely difficult.
Instead, try to find a few tags that you are very experienced with and follow them.
Remember to also set related tags which you know nothing about as "ignored".

For example, I'm very good with Ruby but I know nothing about Ruby on Rails, so I follow the former and ignore the latter.
When I'm in the mood for trying to answer on Stack Overflow, I can click on the Ruby tag and I'm much more likely to see questions in my area of expertise.
I find this much easier to produce high quality content for the database rather than spreading myself thin across all of my areas of expertise.
If I see nothing interesting for Ruby, I'll go down my list of followed tags.

By focusing narrowly for answers and comments, you'll have a much higher chance of being first to respond.
Faster responses is valuable to the database, and also to you since duplicates are almost always downvoted.

### Wait

Remember that the points represent general utility to the programming community, not volume of work.
One of my single most successful days on Stack Overflow was when I posted an accepted answer which quickly racked up 11 upvotes.
That's 135 points!
But it was a total quirk: the question was about a very esoteric topic and I think I just got lucky with a bunch of bored programmers browsing the site that day.
The question has had literally no activity since.

Don't waste your time trying to hit highs like this.

Contrast that with a few answers I've left on highly voted questions (lots of Google hits) that simply fill in the gaps with more situational approaches not mentioned in the top answers.
Just two of these answers (which aren't even accepted) are worth _over 2000 points_!
The questions are very commonly Googled, so even though my answers are more situational, by pure volume they happen to help out a lot more people!
These answers reliably get a couple upvotes a week.
By having enough answers like that out there and by waiting, you will eventually develop a passive income on Stack Overflow.

When you see someone with thousands of points like me, remember you're not necessarily seeing thousands of hours of effort.
They could just have a few highly useful contributions.

## The value of Stack Overflow

All of the implicit rules certainly create a hostile environment for the clueless newbies attracted to Stack Overflow.
But they are also what make it such a precise, highly focused, Q&A machine.
That is ultimately the genius of Stack Overflow: if you try to game the system, you just end up making the database better.
That's what all of my previous tips accomplish!

Even in this age of generative AI, we need Stack Overflow around to extract all of that human knowledge and to store it in a highly structured, searchable database so that the large language models can ingest it.
That is probably what 50% of ChatGPT-written-code is at the end of the day: a glorified Stack Overflow search.

Is it difficult and frustrating to ask and answer questions on Stack Overflow?
Absolutely.
I wouldn't have it any other way.
