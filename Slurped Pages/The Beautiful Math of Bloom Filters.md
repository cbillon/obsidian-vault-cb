---
link: https://nyadgar.com/posts/the-beautiful-math-of-bloom-filters/
excerpt: |-
  By Noam Yadgar
   
   at November 13, 2024
slurped: 2025-11-08T11:30
title: The Beautiful Math of Bloom Filters
tags:
  - go
  - filter
  - bloom
---

[By Noam Yadgar](mailto:noam.g4@gmail.com) at November 13, 2024

Probabilistic functions can model many algorithms and procedures. They can help us optimize procedures to produce the best results. Experienced software engineers know that at some point, almost any software will reach some level of non-determinism, where a solution is not absolute but approaches the best results if using the optimal configuration. Mathematically, such a solution usually boils down to finding _minima_, _maxima_ or _limits_ of some probabilistic functions.

In this article, we will explore the beautiful math behind Bloom filters. We will explore the accuracy and trade-offs of Bloom filters and see why a Bloom filter can be an excellent choice for some cases, especially in _Big Data_, [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) systems, dealing with huge and fairly static datasets.

## The Bloom Filter

The Bloom filter is a probabilistic data structure used to check the membership of elements in a dataset. It is considered one of the most compacted data structures in the family of data structures that help _organize_ datasets.

With just a few megabytes, a Bloom filter can predict, with more than 99% accuracy, the membership of an element within a dataset with millions of items. It can also tell with 100% certainty that an element is not in the dataset. These properties make the Bloom filter an excellent choice for systems that handle an intensive number of requests to read data from large datasets, as it can filter out unnecessary computations.

A Bloom filter consists of:

- An array of bits (the filter)
- A set of non-cryptographic hash functions.
- A function to `insert` an element into the filter.
- A function to `check` the existence of an element in the dataset it _reflects_.

### Example

Here’s a `32Bit` (or `4Byte`) array, with all bits set to `0`

|`0`|`0`|`0`|`0`|`0`|`0`|`0`|`0`|
|---|---|---|---|---|---|---|---|
|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|
|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|
|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|

Whenever we insert an element into the filter, we pass it through all of our hash functions to produce a set of numbers. We then treat those numbers as indices in our bits array, setting each bit to 1 for each location.

Let’s suppose we have a dataset of three items (ranging from `item_1` … `item_3`) and a set of 3 hash functions. Let’s `insert` all items of our dataset:

```
filter.intert(item_1) // => {18, 28, 26}
filter.insert(item_2) // => {4, 7, 24}
filter.insert(item_3) // => {21, 16, 24}
```

After we inserted the data, here’s a representation of the filter. All the hashes were used as indices to set their corresponding bit to `1`.

|`0`|`0`|`0`|`0`|`1`|`0`|`0`|`1`|
|---|---|---|---|---|---|---|---|
|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|**`0`**|
|**`1`**|**`0`**|**`1`**|**`0`**|**`0`**|**`1`**|**`0`**|**`0`**|
|**`1`**|**`0`**|**`1`**|**`0`**|**`1`**|**`0`**|**`0`**|**`0`**|

### Checking items

The `check` function resembles the `insert` function. The only difference is that instead of using the hashes as positions where we set bits to `1`, we’re using the hashes as positions to check if bits are set to `1`.

Let’s try to check if the item: `unknown_1` is in our dataset:

```
filter.check(unknown_1) // => {26, 13... => NOT IN THE DATASET
```

The `check` function can confidently tell us that the item `unknown_1` is **not in the dataset**. One of the hashes marks a position in the filter where the bit is unset (`0`), leading us to the conclusion that `unknown_1` was never inserted into the filter.

We have to assume that all items of the dataset were inserted to the filter

Let’s try another one.

```
filter.check(unknown_2) // => {28, 24, 26} => PROBABLY IN THE DATASET
```

The `check` function falsely assumes that the item `unknown_2` **is probably in the dataset**. This example uses a tiny dataset; looking at the items, we know that `unknown_2` is not in our dataset. However, all the `unknown_2` hashes point to positions where our filter is set to 1, leading the `check` function to a false positive result.

The filter size may be too small, or we use too many hash functions. For any reason, our filter appears too _crowded_ with 1s, resulting in too many collisions. We can use math to find the proportion of dataset size, filter size, and number of hash functions that produce the lowest collision likelihood.

## False positive: The Math

Suppose \(m\) is the size of our Bloom filter in bits, and \(k\) is the number of hash functions in our hash functions set. The probability of having an unset (`0`) bit after just one insertion is \(1-\frac{1}{m}\), however, since each insertion results in \(k\) positions to set, we raise the probability to the power of \(k\).

\[\large{ (1 - \frac{1}{m})^k }\]

Now, let’s suppose that \(n\) is the number of items in our dataset. Since each item is producing \(k\) hashes, the total size of \(n\) items results in \(nk\) insertions to the filter. So the probability of having a `0` bit after \(n\) insertions is:

\[\large{ (1 - \frac{1}{m})^{nk} }\]

Since we’re looking for _collisions_ (_i.e._, the probability of a bit being `1`) we subtract the expression from 1:

\[\large{ 1 - (1 - \frac{1}{m})^{nk} }\]

Now, when we test an item that’s not in the dataset, we check \(k\) hashes for this item. The probability that all the positions result in bits that are already set to `1`, is:

\[\large{ P = (1 - (1 - \frac{1}{m})^{nk} )^k }\]

### Approximating for large _m_

We can make a good approximation of this function that will help us simplify and _generalize_ the rest of the process, using the identity:

\[\large{e^{-1} = \lim\limits_{m\to\infty} (1 - \frac{1}{m})^m}\]

\(m\) is expected to be quite large, so the identity above is sufficiently good for real world scenarios. \(e^{-1}\) is very close to \( (1 - \frac{1}{m})^k \), so we can write:

\[\large{ (1 - \frac{1}{m})^k = ((1 - \frac{1}{m})^m)^{\frac{k}{m}} }\]

\[\large{ \approx (e^{-1})^{\frac{k}{m}} = e^{- \frac {k}{m}} }\]

Now that \( (1 - \frac{1}{m})^k \approx e^{-\frac{k}{m}} \), all other steps can be approximated accordingly. \( (1 - \frac{1}{m})^{nk} \approx e^{-\frac{n}{m}k} \) and so \( 1 - (1 - \frac{1}{m})^{nk} \approx 1 - e^{-\frac{n}{m}k} \). Finally, we approximate the probability of false positive for large sizes of \(m\) as:

\[ \large{P \approx (1 - e^{-\frac{n}{m}k})^k }\]

### Optimizing the filter

The ratio of \(\frac{n}{m}\) appears in the function and intuitively, we know that it’s better to have an \(m\) (the size of the filter) that is at least the size of \(n\) (the number of items in the dataset). We assume \(n\) is given and can only approximate its size. As an example, if we approximate the number of items in our dataset to be around 4 million. How big should \(m\) be?

- `500Kb`? Well, that’s 4 million bits which is the approximated number of items we have in our dataset. Even with a single hash function, our filter will be too crowded with `1`s and, we’ll get a very high rate of false positives.
- `1Mb`? That’s about twice the number of items we have. If we have 2 hash functions, we might be back to the same rate of false positives, since each insertion produces two positions. In this scenario, we may have too many hash functions relative to \(\frac{n}{m}\).
- `100Mb`? That will definitely leave plenty of room for bits to be inserted with fewer collisions, however, it seems a bit arbitrary to just pick a very big number. What if the size of the filter is disproportionately large for no good reason? We will be wasting too much memory.

Later in this article, we will use math to find the optimal size of \(m\) relative to \(n\). Mathematically, finding the optimal \(m\) is good for estimating the size of the filter beforehand, however, it’s important to note that \(m\) has environmental constraints.

For now, let’s take the ratio \(\frac{n}{m}\) as a given constant, configured using the size approximation for \(n\) and the environmental constraints for \(m\).

#### Optimal number of hash functions

Following our example, let’s assume that \(n = 4,000,000\) items and, \(m = 25,000,000\) bits (`3.125Mb`). We know that too few or too much hash functions will increase the probability of false positive results, in fact, we can see how the number of hash functions \(k\) creates _boundaries_ in our function: \( (1 - e^{\textcolor{red}{-}\frac{n}{m}\textcolor{red}{k}})^{\textcolor{red}{k}} \) such that if \(k\) is too large, it brings the inner term closer to 1, and if \(k\) is too small, it lowers the outer exponent which also keeps the overall result too close to 1.

There is a _sweet spot_ where the number of hash functions, relative to \(\frac{n}{m}\) is producing the lowest probability of false positive. In terms of calculus, that is the _minimum_ of the function \(p(k) = (1 - e^{-\frac{n}{m}k})^k \).

#### Finding the minimum

Instead of directly differentiating \(p(k)\), we can take the derivative of \(\ln p(k)\). It will result in a simpler equation to solve and, the two derivates share the same \(x\) intercept, which technically points to the same minimum value of \(p(k)\).

\[\large{ p(k) = (1 - e^{-\frac{n}{m}k})^k = e^{\ln(1 - e^{-\frac{n}{m}k})^k} }\]

By applying the rule: \(\ln(a^b) = b \ln(a)\), we can _isolate_ \(k\) and simplify.

\[\large{ p(k) = e^{\textcolor{red}{k\ln(1 - e^{-\frac{n}{m}k})}} }\]

We take the first derivative of the part being marked in red, and equate to 0.

\[\large{ \frac{d}{dk}[ k\ln(1 - e^{-\frac{n}{m}k}) ] = }\]\[\large{ \ln(1-\textcolor{blue}{e^{{-\frac{n}{m}k}}}) + k\frac{\frac{n}{m} \cdot \textcolor{blue}{e^{{-\frac{n}{m}k}}}}{1 - \textcolor{blue}{e^{{-\frac{n}{m}k}}} } }\]

Let’s declare \(q = \textcolor{blue}{e^{{-\frac{n}{m}k}}} \), which also means that: \(\ln(q) = -\frac{n}{m}k\). Next, we replace all the blue parts with \(q\) and \(\ln(q)\) accordingly.

\[\large{ \ln(1-q) - \ln(q)\frac{q}{1-q} = 0 }\]

Now, it’s fairly simple to solve this equation. By solving it, we’ll find that:

\[\large{ q = \frac{1}{2} }\]

Because \(q = e^{-\frac{n}{m}k} \), the following is also true:

\[\large{ e^{-\frac{n}{m}k} = \frac{1}{2} }\]

And:

\[ -\frac{n}{m}k = ln(\frac{1}{2}) \]

Solving the above for \(k\), we finally get the minimum of \(p(k)\):

\[\large{ k = \ln(2)\cdot\frac{m}{n} }\]

![Bloom prob](https://nyadgar.com/posts/the-beautiful-math-of-bloom-filters/bloom-filter-prob.png)

\(\ln(2) \frac{m}{n} \),is the number of hash functions that produces the lowest probability of false positives. But there’s a problem. In our example: \(\ln(2) \frac{m}{n} = 4.3321 \), however, the result must be an integer. The solution is simple, we can round it up and down, plug both numbers to the function \(p(k)\) and, see what number produces lower probability. With that said, if the difference between the values is insignificant, you should pick the rounded-down value.

In our case, rounding \(k\) down is a tiny bit better. \(p(\lfloor\frac{m}{n}\cdot\ln(2)\rfloor) = 0.0499\). So a filter that’s `3.125Mb` in size, along with \(4\) hash functions can check membership for an item in a dataset that has \(\approx 4,000,000\) items with a rate of about \(\approx 5\%\) false positive results.

If we have spare memory and more CPU, we can dramatically lower the probability of false positives without siginficantly affecting performance. The following table will illustrate the trade-off between probability and size + performance.

|Size/4 million items|Hash functions|False positive|
|---|---|---|
|3.125Mb|4|4.99%|
|3.75Mb|5|2.7%|
|4.79Mb|6|1%|
|6.25Mb|8|0.25%|

#### Optimal filter size

Now that we know how to express \(k\) in terms of \(m\) and \(n\), we can reverse our original question of false positive probability and write a fully generalized equation of which we can find the optimal ratio between the filter size to the number of items that yields the lowest false positive probability. In other words, we can ask: “How many bits per item should we use if the probability is \(\varepsilon\)?”

First, we take our false positive probability expression \((1-e^{-\frac{n}{m}k})^k\) and replace \(k\) with the expressions for optimal number hash functions:

\(\ln(2) \frac {m}{n} \).

\[\large{(1-e^{\textcolor{red}{-\frac{n}{m} \cdot \ln(2)\frac{m}{n}}})^{\ln(2)\frac{m}{n}} }\]

Notice that the expression, marked in red is: \(-\frac{n}{m} \cdot \ln(2)\frac{m}{n} = -\ln(2) = \ln(\frac{1}{2}) \) and

\(e^{\ln(\frac{1}{2})}\) is simply \(\frac{1}{2}\). The entire expression, equated to the desired false positive probability \(\varepsilon\),

can be simplified to:

\[\large { \varepsilon = \frac{1}{2}^{\frac{\ln(2)m}{n}} }\]

We can rewrite \(\varepsilon\) as \(\frac{1}{2}^{\log_\frac{1}{2}(\varepsilon)}\) so:

\[\large{ \frac{1}{2}^{\textcolor{blue}{\log_\frac{1}{2}(\varepsilon)}} = \frac{1}{2}^{\textcolor{blue}{\ln(2)\frac{m}{n}}} }\]

Now, we can remove the exponent base:

\[\large{ log_\frac{1}{2}(\varepsilon) = \ln(2)\frac{m}{n} }\]

Finally, let’s rearrange it, to get the ratio of bits per item (\(\frac{m}{n}\)).

\[\large{ \frac{m}{n} = \frac{\log_\frac{1}{2}(\varepsilon)}{\ln(2)} }\]

### Final Calculation

What makes the expression above so powerful, is that it’s completely detached from actual sizes. For example, we can generally ask how many bits per item should we use in order to have a Bloom filter that has \(1\%\) probability of false positives (assuming we will use the optimal number of hash functions).

\[\large{ \frac{\log_\frac{1}{2}(0.01)}{\ln(2)} = 9.585... }\]

So if our dataset has 4 million items, that optimal number of bits in our filter is:

\[\large{ 4\times10^6 \cdot 9.585... = 38,340,233.5 }\]

Rounding the result, our filter size should be roughly `4.79Mb` in size. We can plug these numbers to our expression for optimal number of hash functions:

\[\large{ \lfloor \ln(2) \cdot \frac{38,340,233}{4,000,000} \rfloor = 6 }\]

A filter of \(38,340,233\) bits (~`4.79Mb`) with \(6\) hash functions, can check membership of items against a dataset with \(4,000,000\) items. All with a false positive rate of about 1%, that’s quite impressive.

## Summary

Using elegant math, we designed a process that predicts items’ membership in an extensive dataset with an outstanding 1% false positive rate. With just a few megabytes and a small set of non-cryptographic hash functions, it’s easy to see the usefulness of a Bloom filter, especially in _BigData_ systems that provide free text-based searches in enormous datasets.

I will leave you with a final thought and a small exercise. You may have already realized that we cannot remove bits from a Bloom filter since multiple items can share the same bits. How can we represent deleted items using a Bloom filter?

We would like to use third party cookies and scripts to improve the functionality of this website. Approve Deny [More info](https://nyadgar.com/privacy)