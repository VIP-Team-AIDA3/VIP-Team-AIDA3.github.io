# Bayesian and Frequentist Statistics

*AIDA3 Mini-Tutorial. Draft.*

You don't need any stats background for this. Fractions are enough.

## Two ways of thinking about an unknown number

Most of what we care about in this program is something we can't measure directly. How often does the aircraft land inside the box. How trustworthy is the runway detector. Whether the model we fit to yesterday's flight data will hold up tomorrow. In each case we have some data and we want a number that the data doesn't hand us outright.

Statisticians have two traditions for doing this, and they've been arguing about it for roughly a century.

The frequentist tradition says the true value is a fixed number that we happen not to know. Your job is to estimate it from the data you collected, and only from that data.

The Bayesian tradition says you should treat your uncertainty about the number as itself a kind of probability. You start with whatever you believed beforehand, the data comes in, and you update.

The rest of this is working out what that difference does in practice.

## Ten landings

Say we flew ten autolands and eight of them landed in the box.

A frequentist estimate is eight out of ten, so 80%. You used the data, that's the answer, and there isn't much more to say about it.

For a Bayesian estimate you have to ask what you believed before the test. Suppose you genuinely had no idea. A way to represent that is to pretend you'd already seen one success and one failure before the test started, which is a mild way of saying "could go either way." Add the real results to those two pretend ones and you get nine successes out of twelve attempts, or 75%.

The Bayesian number came out lower because we admitted up front that we weren't sure. That matters more than it might look. Eight out of ten is a fragile number. One more failed landing and it drops to 73%, which is a large move for a single flight, and it should make you suspicious of how much the original 80% was really telling you.

Run the same comparison with more data and the disagreement evaporates. If we'd flown 200 times with 160 successes, the frequentist answer is 80% and the Bayesian answer is 161 out of 202, which is 79.7%. For practical purposes those are the same number. This is the general pattern: the two traditions converge when data is plentiful, and the Bayesian approach is the more cautious of the two when it isn't.

## Bringing in what you already knew

In the example above we pretended to know nothing. Often that's false. Suppose last semester's flights pointed to something closer to 60% success. You can put that in by pretending you'd seen six successes and four failures instead of one and one. Adding the new data gives fourteen out of twenty, or 70%. The older information pulled the estimate down.

This is the step people object to, and the objection is reasonable. If you get to choose what you assumed beforehand, you can nudge the result wherever you like. Two things keep it honest. First, write down what you assumed and show what the answer becomes under a different assumption. If your conclusion flips, you didn't have a conclusion. Second, the assumption stops mattering once you have real data, as the 200-flight version showed. Prior beliefs carry weight in exactly the situation where you have nothing else, which is also the situation where pretending to be certain would be worse.

## Rare things stay rare

Two questions that sound like the same question:

If there's a runway in frame, how often does the detector fire?

If the detector fires, how often is there a runway in frame?

Suppose our detector catches 99% of real runways and falsely fires on 5% of empty frames. That sounds like a good detector. Now take a thousand frames from a wide-area search where only ten of them actually contain the runway. The detector will fire on essentially all ten of the real ones. It will also fire on about 5% of the 990 empty frames, which is fifty more. So out of sixty alarms, ten are real, and an alarm means roughly a one in six chance that anything is there.

The detector isn't broken. Runways were rare to begin with, and a good alarm doesn't change that as much as intuition suggests. The same arithmetic governs fault detection, medical screening, and every ROC curve any of us will produce, and it's probably the most immediately useful thing in this lesson.

## Ranges, and what you're allowed to claim about them

Neither tradition stops at a single number. Both give you a range, the ranges often look similar, and they do not mean the same thing.

A Bayesian range, called a credible interval, supports the sentence you'd want to say: there's a 95% chance the true value falls between these two numbers. It's a claim about the value.

A frequentist range, called a confidence interval, supports a different sentence: if we repeated this entire flight test campaign many times and computed a range each time, 95% of those ranges would contain the true value. The 95% describes the procedure. It doesn't attach to the particular range sitting in front of you, which either contains the true value or doesn't.

In practice almost everyone reports a confidence interval and then says the credible interval sentence out loud. It's a real error and reviewers do notice. If the sentence you want is the intuitive one, use the Bayesian version.

## p-values

You'll run into p-values constantly in the literature. A p-value answers the question: if nothing interesting were going on, how surprising would data like this be? Small values mean surprising.

It is not the probability that your idea is correct, though it gets read that way. A large p-value doesn't establish that there's no effect; more often it means the sample was too small to see one, which for us usually means we didn't fly enough tests. And a small p-value says nothing about whether the effect is large enough to care about, since with a big enough sample almost any difference becomes statistically significant.

## What we already use

Anyone who has written a Kalman filter has done Bayesian inference without calling it that. You predict where the aircraft should be, a sensor reading arrives, and you blend the two according to how much you trust each. Prediction, measurement, update. Particle filters are the same idea when the math doesn't close nicely.

Least squares model fitting, which is most of what system ID does, sits on the frequentist side. It uses the data in front of it and nothing else.

Our small-sample work leans Bayesian for the reasons in the ten-landings example. With eight flights or thirty test cases the standard frequentist interval formulas can hand you a 95% range whose upper end is above 100%, which tells you something has gone wrong.

Frequentist reporting still matters for anything leaving the program. Reviewers and conferences expect p-values and confidence intervals, so know how to produce them and how to describe them without overstating.

## The formula, if you want it

$$P(\theta \mid D) = \frac{P(D \mid \theta)\, P(\theta)}{P(D)}$$

Here $\theta$ is the unknown you're after, $D$ is the data, $P(\theta)$ is what you believed beforehand, $P(D \mid \theta)$ is how well the data fits a candidate value, and $P(\theta \mid D)$ is what you believe once the data is in. The usual shorthand is that the new belief is proportional to the old belief times how well the data fits. All the counting we did earlier was this formula in disguise.

## Where to read more

For a first pass, the 3Blue1Brown video on Bayes' theorem is about as clear as this material gets, and StatQuest on YouTube covers every term in this lesson at a slower pace.

If you want a book, Richard McElreath's *Statistical Rethinking* is the most approachable serious treatment of Bayesian thinking, and his lecture series is free online. Allen Downey's *Think Bayes* is also free, is built around Python, and stays light on the math. On the frequentist side, Wasserman's *All of Statistics* is a compact reference for looking up what a given test actually does.

Before you write up any results, read Greenland et al., "Statistical tests, P values, confidence intervals, and power: a guide to misinterpretations" (2016). It's a numbered list of twenty-five specific mistakes and it will save you from most of them.

For our side of things, Thrun, Burgard and Fox's *Probabilistic Robotics* derives Kalman filters and particle filters as repeated applications of Bayes' rule, which connects this lesson directly to the GNC work.

## Problems

1. A fault detector catches 98% of real faults and falsely alarms on 2% of healthy flights. Faults occur on one flight in 500. Take 10,000 flights, count how many alarms are real, and decide whether an alarm should worry you.

2. Five tests were run and four passed. Give the frequentist estimate and the Bayesian estimate using the counting method from the ten-landings section, and explain what's shaky about the frequentist one here.

3. Find a sentence in your subteam's most recent report that states a probability or a range. Check whether it's stated correctly.
