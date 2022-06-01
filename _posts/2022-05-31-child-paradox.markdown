---
layout: post
title: "A Paradoxical Error"
date: 2022-05-31
categories: etc
---

<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

I recently read [GameTek][gt] by Geoffrey Engelstein, which includes several
interesting examples of how to improve your intuition when it comes to the
design and the playing of games. Unfortunately, two of these examples are at
best _extremely misleading_ and at worst completely incorrect. However the way
in which they are wrong is fascinating and prompted several hours of thought and
discussion on my part, so I'd like to share them with you.

[gt]: https://www.amazon.com/GameTek/dp/1460757378

## The Premises

Engelstein presents two similar examples in the book, which I will reproduce
here. First, at the end of Chapter 6, "Feeling the Loss", we have a presentation
of a classic paradox in probability called The Two Child Paradox.

> You're talking to woman at a park and she say she has two children. Suddenly a
> boy runs up and grabs her hand and you ask if this is her son. She replies
> that he is.
>
> What are the chances that her other child is a boy?
>
> Most people say 50 per cent: the fact that one is a boy has no bearing on the
> gender of the other child. But this is not correct. There are four possible
> family arrangements that have two children: boy/boy, boy/girl, girl/boy and
> girl/girl. When I tell you that at least one is a boy, that eliminates one
> possible family: girl/girl. There are three combinations left: boy/boy,
> boy/girl, and girl/boy. Each is equally likely, but in only one out of three
> (and thus a ⅓ chance) is the other child a boy.

Then at the end of Chapter 14, "Tic-Tac-Toe and Entangled Pairs", we have this
other situation which might sound different but is actually intimately connected
to the first.

> Let's say four people are playing _Bridge_. One of them says, 'I have an Ace,'
> and we know she is telling the truth. The chance that she's holding more than
> one Ace is about 37 per cent. Later the same player says, again truthfully, 'I
> have the Ace of Spades.' Strangely, the chance that she has more than one Ace
> is now 56 per cent.

Take a moment to ponder these two examples. They probably feel unintuitive, but
all you need to do at this point is understand the premise of each one. When
you're ready, proceed.

## The Paradox

I don't know about you, but I find the first example extremely unintuitive, if
not totally wrong. In my view, the key point that Engelstein omits is that the
children are _distinguishable_. Since you have met one of the children at the
park, you can imagine labeling that as the "park" child and the other one as the
"home" child. Going through the possible configurations we have:

$$\text{boy}_{\text{park}}/\text{boy}_{\text{home}},\text{boy}_{\text{park}}/\text{girl}_{\text{home}},\text{girl}_{\text{park}}/\text{boy}_{\text{home}},\text{girl}_{\text{park}}/\text{girl}_{\text{home}}$$

By discovering that the child in the park is a boy, we are not left with three
possibilities as Engelstein argues, but only two, one of which involves two
boys. So in my view the intuitive answer of 50% is actually correct. Engelstein's
summary of "at least one is a boy" clashes with his earlier statement that we
learned that a specific child is a boy.

So why did Engelstein think it was 1 in 3? Is there even a paradox here? First
we should restate the example to remove some ambiguitiy. Let's imagine that you
and I are playing a game with the following steps:

1. I flip two fair coins and then hide the coins behind a screen. The random
   result is fixed but only I can see it
2. I may reveal some information about the coins to you
3. I reveal both coins and you win if both coins are Heads

Now consider how your chance of winning changes throughout the steps of the
game. After step 1 your chance will always be the same: two fair coins were
flipped, so there are four possibilities, so there is a 1 in 4 chance that both
are Heads.

HH, HT, TH, TT

Suppose in step 2 that I look at the coins and tell you that at least one of
them is Heads. What is your chance of winning? This is the situation that
Engelstein was trying for earlier: this is equivalent to me saying that it is
not the case that both coins are Tails; in other words, I have only eliminated
one of the four possibilities. So by Engelstein's argument, you have a 1 in 3
chance of winning.

Instead, suppose that in step 2 I pick up one coin from behind the screen and
show you that it is Heads. In this case, we can apply _my_ previous argument:
the coins are distinguishable (we have the revealed coin and the hidden coin)
thus there are only two remaining possibilities and so the chance of winning is
1 in 2.

$$\text{H}_{\text{revealed}}/\text{H}_{\text{hidden}},\text{H}_{\text{revealed}}/\text{T}_{\text{hidden}},\color{red}{\text{T}_{\text{revealed}}/\text{H}_{\text{hidden}},\text{T}_{\text{revealed}}/\text{T}_{\text{hidden}}}$$

Now the paradox: why is there a difference between these situations? In the
first case, where at least one coin was Heads, certainly I could have also
picked up a coin and showed you that it was Heads. Why does it matter if I
actually show you or not? It's as if, by introducing completely redundant
information (physically showing you a Head rather than saying it), your chances
of winning are magically increased!

Here is where the two examples from earlier connect. In the Bridge example, I
first say "I have an Ace", then by introducing completely redundant information
and saying it's specifically the Ace of Spades, the chances of having more than
one Ace magically increase! Why does the redundant information help? Certainly
when I say I have an Ace you can imagine me showing you that it is one of the
specific Aces.

So what's going on? Is Engelstein's argument correct, or is mine? Or is
something else going on entirely?

## The Game

To help you understand how to resolve this, we need to put some money on the
line. Now you have to pay to play, but there is a prize for winning:

1. I flip two fair coins and then hide the coins behind a screen. The random
   result is fixed but only I can see it
2. I may reveal some information about the coins to you and state both the prize
   for winning $W and the cost of entry $C
3. If you pay, I reveal both coins and you win if both coins are Heads

The question is if the game is worth playing; based on the information I
revealed, the prize money, and the cost of entry, do you expect to make money by
playing? For example, if you can calculate that the probability of two Heads
based on the information revealed is 50%, the prize is $24, and the cost of
playing is $10, then

$$\text{expected winnings}=P(\text{win})\times($24-$10)-P(\text{lose})\times($10)=0.5\times $24-$10=$2$$

In other words, you expect to make $2 per play on average (assuming you have a
big enough budget to play the game repeatedly and overcome some early bad luck).
As you can see from this example, it is crucial that you are able to calculate
the probability of winning in order to make a profit in this game. So imagine
you find me at the carnival offering that exact situation: the prize is $24, the
cost $10, and I'm showing you that a coin is Heads. Do you play?

Assuming you found my argument from earlier convincing, your chances of winning
are 1 in 2. Based on the previous calculation, you will make a profit! So you
play and indeed you win! So you keep playing, following the exact same strategy:
when I show you Heads you pay and, to keep things simple, when I don't show you
Heads you decline to pay and simply wait for me to flip the coins again. With
this strategy you make a small but reliable profit ($2 per game on average);
you're going to bleed me dry if it takes all day!

Then something changes. Suddenly your strategy starts losing. Convinced it must
simply be an unlucky streak, you play on, but eventually I completely drain your
funds and you walk away with empty pockets. What happened?

Imagine you're in my shoes and you need to run this game and you'll quickly see
what is missing from the picture: when do you decide to show the player a coin?
For example, I could follow a procedure like this:

1. Flip the coins and place one on the left and one on the right
2. If the left coin is Heads, show them that coin
3. Otherwise, show them nothing.

In this case, the 1 in 2 calculation is correct: whenever I show you a coin, the
only remaining possibilities are

$$\text{H}_{\text{left}}/\text{H}_{\text{right}},\text{H}_{\text{left}}/\text{T}_{\text{right}}$$

So playing is always profitable.

But what if instead I follow this procedure:

1. Flip the coins and place one on the left and one on the right
2. If the left coin is Heads, show them that coin
3. Otherwise, if the right coin is Heads, show them that coin
4. Otherwise, show them nothing.

Now whenever I show you a coin, it could be

$$\text{H}_{\text{left}}/\text{H}_{\text{right}},\text{H}_{\text{left}}/\text{T}_{\text{right}},\text{T}_{\text{left}}/\text{H}_{\text{right}}$$

and you have no way to distinguish between them. The chance of winning has
dropped to 1 in 3. Unfortunately, from your perspective, both procedures look
nearly identical. The only difference would be the frequency of me showing
nothing, but this is complicated by the fact that I can switch between these
procedures at will. If I start with the first procedure I can sucker you in by
making you think the game is rigged in your favor. When I switch strategies,
your expected winnings flip to −$2 and hopefully for me your greed will cost you
all of your profits and then some. I could even be sneakier and switch between
these strategies randomly, slightly favoring the one that pays me more.

If you approach this game naively, you will always lose in the long run. Why?
Because, like Engelstein, you will assume that the specific way in which you
were given information reveals something additional about the nature of _why_
that information was revealed; specifically, that me showing Heads versus saying
"There is at least one Heads," implies something about the choice _I_ made. In
general, there may be no connection between these things.

So this is the resolution of the paradox: it is _not_ the case that revealing
redundant information can change the chances of an event, rather it is possible
to phrase a probability problem in an ambiguous way such that most people will
make seemingly obvious (but unsupported) assumptions about it and thus come to
incorrect conclusions.

## The Resolution

Going back to the original Two Child Paradox. What were the unsupported
assumptions we made in order to arrive at the paradox in the first place?

To arrive at the answer of 1 in 2 for the chances of the other child being a
boy, we must assume that the gender of the child we met has no bearing on why we
met them at the park. If that same child were a girl, we are assuming they would
have still been brought to the park. This makes the gender irrelevant, since if
they brought a boy, we have boy/girl or boy/boy (1 in 2) and if they brought a
girl we have girl/boy or girl/girl (1 in 2). This seems like a reasonable
assumption to me, however it _is not stated_ in the problem.

To arrive at the answer of 1 in 3, we must assume some strange parental
behavior, namely that only boys are brought to parks. In that case, the fact
that we have met a boy at a park means that this could be a family with two
boys, or boy/girl, or girl/boy since all of these equally likely scenarios would
result in us meeting a boy. Again this seems like a strange assumption, but it
is also not stated and thus cannot be excluded.

Back to Bridge. How do you arrive at the probabilities that Engelstein gave? If
I say "I have an Ace" what is the probability that I have two or more Aces? In
order to answer this we need to state our assumptions, namely, why did I say I
have an Ace? If we assume that whenever I have any Aces I always say "I have an
Ace" then indeed you arrive at about 37% chance of me having two or more. This
seems like a reasonable assumption to make.

But what if I say that I have the Ace of Spades? How do we arrive at 56% chance
of two or more Spades? That is the probability we get if we assume that I _only_
tell you when I have the Ace of Spades; specifically, if I have other Aces and
no Ace of Spades _I say nothing_. Similar to the Two Child Paradox this feels
like a very strange assumption to make. If we're interested in findings Aces,
why wouldn't I tell you about the others when I have them? This means that there
would be situations where I have two or more Aces (none of them Spades) but I
don't reveal any information. A much more reasonable assumption here is if we
assume that when I have _any_ Aces I tell you the suit of one of them. It's not
hard to see that now I will tell you about an Ace (and its suit) whenever I have
any Aces, in other words we're back in the first situation. So now even though I
am revealing additional information (the suit), the probability of having two or
more Aces is still 37%, which is the intuitive result.

## My Conclusion

If you only remember one thing from this, it is to be careful in your
assumptions when turning word problems into math problems since they can
completely change your answer. Engelstein mostly does a good job in his book of
debunking common false assumptions such as the idea of "hot streaks" in
gambling, but he ironically talks himself back into such bad assumptions with
these two examples. Both answers he gives are unintuitive because they make
unintuitive assumptions. If you make the more natural assumptions&mdash;in my
opinion&mdash;you get the intuitive answers and no paradox appears.
