---
link: https://alexkritchevsky.com/2018/11/05/bayes.html
byline: Alex Kritchevsky
excerpt: Some old notes which I have promoted to a blog post so they are easier to find.
slurped: 2026-06-23T08:43
title: Notes on Bayesian Inference
tags:
  - bayes
---

Some old notes which I have promoted to a blog post so they are easier to find.

## 1. Bayes’ Theorem

From basic probability we know about the concept of _conditional probability_, which says that these are equivalent:

\[P(A \and B) = P(A | B) P(B) \\ = P(B | A) P(A)\]

This immediately gives Bayes’ Theorem, which allows us to invert \(P(A \| B) \lra P(B \| A)\):

\[P(A | B) = P(A) \frac{P(B | A) }{P(B)} \tag{1}\]

We can expand the \(P(B)\) term in the denominator, like this:

\[P(B) = P(B | A) P(A) + P(B | \neg A) P(\neg A)\]

This gives:

\[P(A | B) = \frac{P(B | A) P(A)}{P(B | A) P(A) + P(B | \neg A) P(\neg A)}\]

---

## 2. Bayesian Inference

Next, we formulate Bayesian _Inference_. Bayesian Inference is the process by which we incorporate new evidence to update our beliefs (ie, our guess of the probability \(P(H)\) that a hypothesis is true). I say “_is_ the process” because we do it already, heuristically, to some degree. What follows is the mathematical version of the intuitive process of ‘inductive reasoning’.

We interpret \(P(A \| B)\) as “having learned the new information \(B\), what do we think the probability of \(A\) is?”. Let’s relabel: \(A \ra H\) for ‘hypothesis’ and \(B \ra E\) for ‘evidence’, giving:

\[P(\text{Hypothesis} | \text{Evidence}) = P(\text{Hypothesis}) \frac{P(\text{Evidence} | \text{Hypothesis}) }{P(\text{Evidence})}\]

Or, in short:

\[P(H | E) = P(H) \frac{P(E | H) }{P(E)}\]

This equation is so important that all of its parts have names:

- \(P(H)\) is the **prior probability** of \(H\): the probability we assume in a vacuum with no additional knowledge. Sometimes it’s just called the **prior**.
- \(P(H \| E)\) is called the **posterior probability** of \(H\): the probability after we have incorporated our knowledge of \(E\).
- The \(P(E \| H)\) term is called the **likelihood**: supposing \(H\) were true, how likely would evidence \(E\) have been?
- \(P(E)\) is sometimes called the “marginal likelihood”, but don’t worry that. It’s the probability of the evidence on its own.

The term \(\frac{P(E \| H)}{P(E)}\) together serves to **update** the value of \(P(H)\): “formally we thought the probability of \(H\) was \(P(H)\), but now that we’ve measured \(E\), we think it’s \(P(H \| E)\)”. The process of incorporating this new information \(E\) to compute \(P(H \| E)\) is called a Bayesian Update.

Here’s a classic example:

Suppose there is a diagnostic test for a disease which is not entirely accurate – it can have false positives (report healthy people as sick) and false negatives (report sick people as healthy).

Let’s say that:

- We know that \(\frac{1}{6}\) of people are actually sick (based on some external information, like a big survey or past experience).
- The test identifies sick people correctly \(\frac{10}{11}\) of the time and healthy people as healthy \(\frac{10}{11}\) of the time.

If you go in and get the test and it gets a positive result (“you are sick!”), then, all else being equal, what’s the probability you are actually sick? We plug in:

\[\begin{aligned} P(H | E) &= P(H) \frac{P(E | H) }{P(E)} \\ &= \frac{1}{6} \frac{\frac{10}{11}}{\frac{1}{6} \frac{10}{11} + \frac{5}{6} \frac{1}{11}} \\ &= \frac{10/11}{10/11 + 5/11} \\ &= \frac{10}{10+5} = \frac{2}{3} \end{aligned}\]

This gets the job done. Of course, in no real life situation would you know the exact probability of every behavior of the test. What’s important is that you start from the probability theory and then refine it for real life. The theory gives intuition for the phenomena that matter – like the fact that a test can say you’re sick, yet only be right 1/10th of the time, because the false positive rate dominates.

But, in fact, this turns out to be a fairly unwieldy way of computing Bayes updates. To my surprise, doing everything in “Odds” works way better.

---

## 3. Odds

It is also possible to express probabilities in the form of _Odds_. I have long had trouble understanding why anyone would want to do this, but I might have figured it out: it’s because it miraculously makes the Bayesian inference process much simpler.

Odds are just the familiar concept of, well, odds: if I say a bet has 2:1 odds, then it’s twice as likely to occur as to not occur. We write odds as \(O(X) = a:b\), which corresponds to a probability \(P(X) = \frac{a}{a+b}\). The expression for \(P(X)\) measures the occurrences of \(X\) compared to _all outcomes_ (\(a+b\) in this case), while \(O(X)\) measures the occurrences of \(X\) compared to \(\neg X\). They express the same thing in two different ways.

Even though we write odds as ratios \(a:b\), they are still, basically, just fractions – so \(ca : cb \sim \frac{ca}{cb} = \frac{a}{b} \sim a:b\). (Well – this isn’t entirely true, because it can be meaningful to write down odds of more than two terms, like \(1:2:3\), in which case you can divide factors across all _three_ terms. But it’s usually just 2, which is basically a fraction.)

I find that thinking in Odds reminds me that probabilities are all about comparing the **rates** of different things happening (if you somehow tested multiple times). In a probability like \(P(A) = \frac{a}{a+b}\), we compare the rate of \(A\) to the rate of _anything at all_ (\(a\) or \(b\), in this case). In an odds, like \(A : \neg A\), we compare the rate of \(A\) to the rate of, specifically, not-\(A\). Thing is, both of them are rates. There’s nothing stopping us from mixing probabilities and odds in the same equation, as long as we’re careful about knowing what we mean.

---

Anyway – Bayes’ theorem can be written with odds like this:

\[P(H | E) : P(\neg H | E) = P(E \| H) P(H) : P(E \| \neg H) P(\neg H) \\ = \frac{P(E \| H)}{P(E \| \neg H)} P(H) : P(\neg H)\]

These are odds between probabilities, yes, but you can kinda see what’s going on. We initially thought the odds of \(H\) were \(P(H): \neg P(H)\). After observing \(E\), we now think they’re \(P(H \| E) : P(\neg H \| E)\). We can mostly drop some of the \(P\)’s here: since \(P(H)\) means \(\frac{H \text{ happens}}{H \text{ or } \neg H \text{ happens}}\), we can simplify \(P(H) : P(\neg H) = H : \neg H\), dropping the denominators of both terms.

So this is **Bayes’ Theorem, Odds Ratio form**:

\[(H | E) : (\neg H \| E) = \frac{P(E | H)} {P(E | \neg H)} (H : \neg H)\]

Read aloud:

> The **posterior odds** \(O(H \| E : \neg H\| E)\) of \(H : \neg H\) after observing evidence \(E\) is equal to the **prior odds** \(O(H : \neg H)\) times the **relative likelihood** \(\frac{P(E \| H)} {P(E \| \neg H)}\).

Relative Likelihood is a new term. It’s also called the _Bayes Factor_, and it measures how much \(E\) recommends \(H\) versus \(\neg H\). For instance, seeing that the ground is wet suggests a very high value for the hypothesis “it rained recently”, meaning that it recommends this hypothesis far more than the alternative (“it did not rain recently”). This term isn’t an _odds_ exactly, but it is a _ratio of rates_. It means: “the rates of \(E\) given \(H\) compared to the rate of \(E\) given \(\neg H\)”. This is why I emphasized that odds are sort of like rates above – the intuition goes a long way.

Finally, even though I just wrote everything in odds … it tends to be useful to just write odds as fractions anyway; it’s a lot more familiar that way:

\[\boxed{\underbrace{\frac{(H | E)}{(\neg H | E)}}_{\text{Posterior Odds}} = \underbrace{ \frac{(E | H)}{(E | \neg H)}}_{\text{Relative Likelihood }} \times \underbrace{\frac{(H)}{(\neg H)}}_{\text{Prior Odds}} \tag{Bayes, Odds Form}}\]

This is my favorite way to write Bayes’ Rule. All the terms sit happily next to each other, and multiply. In this form, the Bayesian update process is simple multiplication. Just like this:

\[\text{posterior odds} = \text{prior odds } \times \text{ relative likelihood}\]

The reason for the odds form is that, once you get used to it, it’s _really_ easy to do in your head. Here’s the classic example again:

\(\frac{1}{6}\) of people are sick, so \(\frac{(H)}{(\neg H)}\) = \(\frac{1}{5}\). The test reports sick people as sick \(\frac{10}{11}\) of the time, but healthy people as sick \(\frac{1}{11}\) of the time, so \(\frac{(E \| H)}{(E \| \neg H)} = \frac{10/11}{1/11} = 10\). Finally:

\[\frac{(H \| E)}{(\neg H \| E)} = 10 \times \frac{1}{5} = 2\]

And \(2:1\) odds means \(P(H \| E) = \frac{2}{3}\).

This is _much easier_ to do on the fly then the probability version – once you get the hang of thinking in terms of rates, which takes a bit of getting used to. Here’s how it sounds in my head:

> \(5\) times as many people are healthy compared to sick, but the rate of sick reports on sick people is 10x the rate on healthy people, so that’s \(10:5 = 2:1\) odds that a sick report is actually sick, which is \(\frac{2}{3}\).

---

## 4. In practice

You usually begin with a guess (a prior) for the probability of some hypothesis \(H\), which you turn into an odds:

\[\frac{(H)}{(\neg H)} = H : \neg H = P(H) : P(\neg H)\]

Then you go through your life or experiment or whatever, finding observations \(\vec{E} = \{ E_i \}\), which have different rates of occurring for \(H\) vs. \(\neg H\), causing updates:

\[\frac{(H | \vec{E} )}{(\neg H | \vec{E} )} = \frac{(H)}{(\neg H)} \prod_i \frac{(E_i, H)}{(E_i, \neg H)}\]

Which amounts to:

\[\text{currently belief ratio} = \text{prior} \times \prod_i \text{ratios from } E_i\]

Some entailments:

1. If each thing you see occur is more likely to occur for \(H\) than for \(\neg H\), then your belief in \(H\) gets higher and higher, approaching but never reaching infinity.
2. If you ever see something happen which is _impossible_ if \(H\), ie has odds \(0: x\), then you no longer believe in \(H\), no matter what.
3. Your prior permanently weights your resulting odds, no matter how many observations you see.
4. There is no difference between seeing one or many observations if they multiply out to the same rates of occurence for \(H\) vs \(\neg H\).
5. To make an unlikely claim \(H\) seem _more_ likely than you started out, you need likelihoods which are at least the inverse of your current belief. IE if you currently think the ratio is \(2:5\), you need ratios \(> 5:2\) to start inverting that.

There is no requirement, either, that you only consider \(H\) and \(\neg H\). You can have an entire family of (exlcusive) hypotheses \(H_j\). Fractions are only good at expressing one or the other, but ratios, in general, can be between more than two terms: \(H_1 : H_2 : H_3\).

The procedure of ‘inference’ as performed by human intuition is assumed / understood to be an analog approximation to this. One useful fact though, is that you can perform the calculation even if you don’t have ‘every available hypothesis’. Bayesian updating updates the _relative likelihoods_ of hypotheses, but it doesn’t make any claim as to their absolute likelihood. If you get a new hypothesis later you are free to re-evaluate old evidence in light of it and add it to the mix.

---

I’ve never really used it, but it’s worth mentioning: some people think that it can be easier to think in **log odds**, which makes all the math additive by taking the logarithm of everything:

\[\log (H | \vec{E}) - \log (\neg H | \vec{E}) = \log H - \log \neg H + \sum_i \log(E_i, H) - \log(E_i, \neg H)\]

This will be easiest is your rates are all given as ratios of powers of the same number. How would that work?

It’s easy if you’re counting the number of ways things can happen. If there are \(2\) ways that \(H\) can happen and \(4\) ways that \(\neg H\) can happen, then \(\log_2 \frac{H}{\neg H} = \log 2 - \log 4 = -1\). To get back to an odds ratio, compute \(2^{-1} = \frac{1}{2} = 1:2 \lra P(H) = \frac{1}{3}\).

In log odds, if \(A\) is “twice as likely as” \(B\), then \(\log (A) = 1 + \log (B)\); you’re doing something like adding up “how many times is \(A\) twice as likely as \(B\)”, which isn’t too bad to do in your head.

---

## 5. Conservation of Expected Evidence

This is an obscure idea which I think is quite helpful enlightening. It formalizes the idea that “absence of evidence _is_ evidence of absence”, in the sense that if a positive result gives evidence for a hypothesis, a negative one _must_ give evidence against it.

We can write \(P(H)\), the prior probability of \(H\), like this:

\[P(H) = P(H | E) P(E) + P(H | \neg E) P(\neg E)\]

We can interpret this in an interesting way. Since \(P(E)\) and \(P(\neg E)\) are effectively constant, this is the form of an _expectation value_, specifically, of the value \(\bb{E}[P(H)]\). It’s the expected value of \(P(H)\) after the observation of \(E\) or \(\neg E\): \(\bb{E}[P(H)]\), based on the probabilities of \(E\) and \(\neg E\).

Suppose that \(P(H \| E) > P(H)\), so \(E\) is positive evidence for \(H\). Then the formula above shows that we expect to see our estimate of \(H\) go up exactly \(P(E)\) of the time, and down \(P(\neg E)\) of the time – and the amounts that our estimate of \(H\) changes by are proportional to how like \(E\) is compared to \(\neg E\). Specifically:

\[\D P(H | E) P(E) = - \D P(H | \neg E) P(\neg E)\]

This has been called **conservation of expected evidence**: If seeing \(E\) makes you think that \(H\) is _more_ likely, then seeing \(\neg E\) _must_ make you think that \(H\) is _less_ likely. Moreover, if you already believe that \(P(H)\) is the _true_ probability of \(H\), then you don’t _expect_, on average, for evidence to change your opinion on that _at all_ – the expected change \(\D P(H)\) is \(0\).

On the other hand, if you’re _wrong_ about \(P(H)\), then on average you will change your mind after testing \(E\), and on average it will lead you in the direction of the true value of \(P(H)\). If you had _expected_ to change your mind about \(P(H)\) upon observing \(E\) (in one direction or the other), you could just _imagine_ testing \(E\) and then making the change anyway - you already know enough to update your hypothesis without doing another experiment. This means that the only stable state, where you have incorporated all available evidence, is when you don’t the test of \(E\) to change your estimate of \(P(H)\) at all.

More [here](https://www.lesswrong.com/posts/jiBFC7DcCrZjGmZnJ/conservation-of-expected-evidence), including the delightful real-life example that if living a sinful life makes a medieval woman more likely to be a witch, then living a virtuous life _must_ make a medieval woman _less_ likely to be a witch, which is unfortunately not the opinion that some witch-burners held.

---

## 6. Frequentism

Inference is sometimes framed as having two ideological camps: frequentist and Bayesian. Frequentist statistics includes such ideas as \(p\)-values and confidence intervals and the like.

Regardless of that ideological battle, it’s useful to figure out where frequentist statistics like confidence intervals and P-values fit into the framework of Bayesian inference. I believe the explanation is: frequentist statistics only give the probability of results conditioned on hypotheses, never the _probability of hypotheses_. The hypothesis’s probability is commonly then _deduced via intuition_, using the frequentist statistic as the ‘experiment’. Bayesianism can make this step mathematical.

Here is how you represent frequentism in light of Bayes:

A \(p\)-value fundamentally says that “Given a (null) hypothesis \(H\), the observed data \(E\) has probability \(P(E \| H)\)”.[1](https://alexkritchevsky.com/2018/11/05/bayes.html#fn:aside) A study which reports a \(p\)-value implicitly says “see how small that number is? surely that makes you doubt that \(H\) is true!”. This leaves the job of determining the truth value of \(H\) to the reader – which is intuitive, but could be modeled by the _explicit_ step of computing a new \(P(H)\) via Bayesian Inference:

\[P(H | E) = P(E | H) \frac{P(H)}{P(E)}\]

Since this step is omitted, it’s not necessary to ever discuss the prior probability \(P(H)\). Your intuitive inference, upon reading the \(p\)-value \(P(E \| H)\), is to imagine that \(P(H)\) is maybe sort of likely, maybe not, but regardless having seen the (hopefully miniscule) value of \(P(E \| H)\), you certainly should change your perceived value of \(P(H)\) by that factor.

The Bayesian expression of this inference lets us look more carefully at what assumptions are being made when we see a low \(p\)-value and take it to mean that \(H\) is false. It turns out to be pretty important what \(P(E)\) is. Or rather, it’s important how much higher \(P(E \| H)\) is than \(P(E \| \neg H)\). Assuming \(P(E \| H) \neq 0\) and \(P(H) \neq 0\), we can rearrange Bayes’ Rule like this:

\[\begin{aligned} P(H | E) &= \frac{ P(E | H) P(H) }{P(E | H)P(H) + P(E | \neg H) P(\neg H)} \\ &= \frac{1}{1 + \frac{P(E \| \neg H)}{P(E \| H)} \frac{P(\neg H)}{P( H)}} \\ &= \frac{1}{1 + \frac{P(E \| \neg H)}{p} \frac{1 - \text{Prior}}{\text{Prior}}} \end{aligned}\]

This forms makes it somewhat easier to see what’s going on. The value \(P(E \|H )\) is the \(p\)-value; but the other values are not usually available.

But we can imagine: supposing that \(p\) is very low. What _else_ is necessary to conclude \(\neg H\)? Well:

1. \(P(E \| \neg H)\) is not very small compared to \(P(E \| H)\), so \(\frac{P(E \| \neg H)}{P(E \| H)} = \frac{P(E \| \neg H)}{p}\gg 1\)
2. the prior odds \(\frac{P(\neg H)}{P( H)}\) are _not_ small enough to matter, meaning that it’s also true that \(\frac{P(E \| \neg H)}{P(E \| H)} \frac{P(\neg H)}{P( H)} \gg 1\).

.. _only then_ is it true that \(P(H \| E) =\frac{1}{1 + \frac{P(E \| \neg H)}{P(E \| H)} \frac{P(\neg H)}{P( H)}} \ll 1\), from which you conclude that

> evidence \(E\) indicates that \(H\) is very likely not to be true

What this shows is that frequentism doesn’t manage to _avoid_ dealing with priors; it just avoids the issue entirely by leaving you to do that step in your head. Which is fine if the priors are obvious, or if you collect enough information that \(\frac{P(E \| \neg H)}{P(E \| H)}\) is so massive that your prior probability almost doesn’t matter at all.

---

## 7. Priors

How do you calculate a prior if it _does_ matter?

For scientific experiments like “determining which hypothesis is true”, I tend to take the opinion that if your precise choice of prior matters, then you’re already in dangerous territory. If a statement \(H\) about reality is _unequivocally true_, then you should be able to collect, given enough time and money, _enough_ information supporting \(H\) to overcome _any_ prior probability. If you find your results’ persuasiveness to be dependent on whether your prior odds are \(1:2\) or \(1:20\), then it seems likely that you really don’t have enough data to say anything convincing anyway. When choosing what to _believe_ that you really want to just have one theory rise to the top and be obviously the best.

But suppose you’re gambling, or programming a self-driving car, or buying stocks, or doing _anything_ which has numeric values expected with probabilities, and you want to come out ahead. Well, you don’t get to just say “well, priors shouldn’t matter”. You could say that, and you’d lose money to the person who doesn’t, and computes priors and does more math than you. In _real_ calculations, priors matter. When choosing _what the distribution of possible states of the world is_, then you start with a distribution of hypotheses and update your way to a better one, and then take action based on it, and if you do this perfectly you will have the highest possible expected results (you won’t do it perfectly, but doing it better will be better than doing it worse.)

For example: suppose you have narrowed everything down to two theories about the stock price of a company, \(A\) and \(B\). You start with priors and update on evidence and eventually compute \(P(A)\) and \(P(B)\).

At this point, if your goal is to choose _what to believe about the world_, then you wait until \(P(A)\) is overwhelmingly higher than \(P(B)\), and then you declare “\(P(A)\) is higher by a lot, so I believe that”, and you’re fine. Human belief only requires getting an obvious theory. But if you’re actually trying to _make money_ (or maximize any numeric quantity) with your theories, then you don’t operate on the assumption that the most likely theory is true; you pick the strategy which maximizes \(\bb{E}[Q]\). It’s entirely possible that you may believe \(P(A)\) to be vastly greater than \(P(B)\), but the returns on \(B\) are _so high_ that betting on it is still worth it. And in this case priors might matter, because if you pick them wrong you’ll pick strategies suboptimally.

(Note that in practice, if you, as a human, come to accept theory \(A\), you might not want to pick a strategy around \(B\), because you’re also optimizing to minimize embarrassment and be risk-averse, maybe, and taking big risks on something you don’t believe in can be _socially_ wrong even if it is _mathematically_ optimal.)

---

So how do you pick a prior distribution?

Well, if you know things about what it _should_ be, you estimate it from that. If you _don’t_, you can try to come up with what’s called an _uninformative prior_ – a prior which expresses what little objective information you have, like “the value is positive”. Fortunately of the rules of probability still apply, so we can use facts like:

1. \(P(A \cap B) \leq P(A)\): your prior for any composite result should be strictly less than your prior for its components
2. The central limit theorem: if \(A\) is the sum or average of many values, the result should be normally distributed, even if constituent values aren’t
3. \(0 < P(A) < 1\): [Cromwell’s Rule](https://en.wikipedia.org/wiki/Cromwell%27s_rule): nothing can have probabilities \(0\) or \(1\)
    - This can be relaxed sometimes for _definitional_ statements. If you’re trying to assign probabilities to propositions, then the logical entailment of a premise has \(P=1\) given the premise. For instance \(P(2+2=4)\) may be said to have probability \(1\).
    - And if you protest that “what if you’re computing things wrong!” then maybe stop trying to optimize probabilities anyway. Otherwise it gets _really_ complicated.
4. Arguments from symmetry: if two states are identical up to relabeling, they should have the same probability. More generally, the prior distribution should reflect the underlying _symmetry group_ of the system in question.
    - Special case: argument from indifference. If we have \(N\) states and there’s no reason to prefer one over the other, they should each have prior probability \(\frac{1}{N}\).

---

Finally if none of these works, there is one “correct” – but theoretically uncomputable – way to do it, which is called [Solomonoff Induction](https://en.wikipedia.org/wiki/Solomonoff%27s_theory_of_inductive_inference), and amounts to a mathematical formulation of Occam’s Razor.

Heuristically:

> Simpler theories are more likely than complicated ones

Algorithmic complexity theory promises us that the concept of a theory being ‘simple’ is not a feature of what language you write it down in, but it’s somehow not actually possible to tell given a theory how complex it is. Whatever. Point is, you order theories by complexity and prefer simpler ones, and that is the most natural prior you can get.