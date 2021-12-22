---
layout: post
title: "Frequent flyers?"
date: 2019-11-28
---

Following a 2014 survey of the British public it was revealed that in the
UK 15% of people take 70% of flights and 52% of people hadn’t taken
any flights at all in the past year. This makes it sound like there is
an elite in the UK taking the vast majority of flights, whilst the
poor proletariat are condemned to a lifetime of holidays in British
beach resorts 50 years past their best. Some politicians are
minded to [implement a frequent flyer tax](https://edm.parliament.uk/early-day-motion/49825)
on the back of this and it is in the Lib Dem's 2019 Manifesto.

Scratching under the surface though reveals that the picture may not
be as divided as first meets the eye. A [full fact article](https://fullfact.org/economy/do-15-people-take-70-flights/)
was written on the subject and it details where the methodology of the study in
question - 1,000 UK residents were asked how many flights they had
taken over the past 12 months. I don’t need to point out that there is
a difference between not taking a flight in the past 12 months and
never flying.

I was curious to see if I could make the assumption that everyone in
the UK has the same flying behaviour and reach a similar statistic to
that found in the survey. Specifically I assumed that everyone in the
UK has some probability of flying in a given month, and this is
constant throughout the year. Obviously this is a gross
oversimplification of peoples actual flying behaviour, for example
many people will take flights only at particular times of the year
(summer) and will be less likely to take a flight in the month
immediately after having taken another, but serves the purpose of
assuming the same for everyone.

I then simulated a years worth of flying for 1,000,000 people at
different probability levels and computed the equivalent statistics to
those above for the artificial sample. For the more mathematically
minded, I took 1,000,000 lots of 12 draws from a binomial distribution
at various levels of p. Note that I fully appreciate that this is the
hacky computer science implementation of a simple statistics problem,
but there you go.

```python
n = 1_000_000
trial = np.random.binomial(12, 1/19, n)

print(round(len(np.where(trial==0)[0])*100/n), '% people no flights in past 12 months')
total_flights = np.sum(trial)
trial_sorted = np.sort(trial)
print(round(trial_sorted[int(0.85*n):].sum()*100/total_flights), 'flights taken by top 15%')
```

I found that for a constant probability of 1/19 of flying in a given
month 52% of the population would not fly at all in a given year, and
the top 15% of flyers take 48% of flights. This means that you can get
pretty close to a survey result that is being used as evidence of an
extremely unequal society by assuming a perfectly equal one. I suspect
that if these simulated results were the actual results of the study,
it would have received a similar reaction.

So what can we take from this exercise? Of course it is not the case
that in reality everyone’s propensity to fly is the same clearly
better off people do fly more than the less well off - but it is also
not the case that over half the British population do not fly at all,
just because they haven’t flown in the past year and it is very likely
that actual flying habit are much more similar across the UK
population that people would naturally assume from reading the high
level statistics. In fact the only reason why I did this simple exercise in the first place was
because, when thinking of everyone I know,  I could barely think of a single person that doesn’t fly at
all, no matter what their income level is.

As a result a policy to tax frequent flyers may not be as progressive
as those promoting it currently believe it to be. It could end up
impacting the vast majority of the population in much the same way. It
could also have less impact than proponents expect in discouraging
flying if, for example, there are a lot of people that fly once a year
for a holiday but occasionally must fly a second or third time to, for
example, visit friends and relatives abroad. The impact of a tax on
flights taken for people on business versus leisure will also be
markedly different given the price insensitivy of the former relative
to the latter.

That’s not to say that a frequent flyer tax is a bad idea, I strongly
support higher taxes on flying and progressive taxation in general.
The fact that you can fly from London to Barcelona and back for under
£100 but can’t get the train to Edinburgh and back for the same price
is completely unsustainable, and frankly crazy - and must be addressed
in one way or another. Simply having higher taxes on flying full stop might be a more
simple and productive solution?

P.S.

There is probably a statistical exercise that could be done to infer the
UK population's flying distribution from the study
results above. If I find some time over the holiday period to work out
how to do this I may well give it a pop. In the meantime it would be
excellent if a more detailed study was designed to work out the impact
of a frequent flyer tax.
