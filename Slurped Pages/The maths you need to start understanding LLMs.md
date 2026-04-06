---
link: https://www.gilesthomas.com/2025/09/maths-for-llms
byline: Giles Thomas
site: Giles Thomas
date: 2025-09-03T01:30
excerpt: "A quick refresher on the maths behind LLMs: vectors, matrices,
  projections, embeddings, logits and softmax."
twitter: https://twitter.com/@gpjt
slurped: 2026-04-05T11:08
title: The maths you need to start understanding LLMs
---

Archives

Categories

Blogroll

Posted on 2 [September 2025](https://www.gilesthomas.com/2025/09/) in [AI](https://www.gilesthomas.com/ai)

> This article is the second of three "state of play" posts that explain how Large Language Models work, aimed at readers with the level of understanding I had in mid-2022: techies with no deep AI knowledge. It grows out of [part 19](https://www.gilesthomas.com/2025/08/llm-from-scratch-19-wrapping-up-chapter-4) in my series working through [Sebastian Raschka](https://sebastianraschka.com/)'s book "[Build a Large Language Model (from Scratch)](https://www.manning.com/books/build-a-large-language-model-from-scratch)". You can [read the first post in this mini-series here](https://www.gilesthomas.com/2025/08/what-ai-chatbots-are-doing-under-the-hood).

Actually coming up with ideas like GPT-based LLMs and doing serious AI research requires serious maths. But the good news is that if you just want to understand how they work, while it does require some maths, if you studied it at high-school at any time since the 1960s, you did all of the groundwork then: vectors, matrices, and so on.

One thing to note -- what I'm covering here is what you need to know to understand _inference_ -- that is, using an existing AI, rather than the _training_ process used to create them. That's also not much beyond high-school maths, but I'll be writing about it later on.

So, with that caveat, let's dig in!

### Vectors and high-dimensional spaces

In [the last post](https://www.gilesthomas.com/2025/08/what-ai-chatbots-are-doing-under-the-hood) I used the word "vector" in the way it's normally used by software engineers -- pretty much as a synonym of "an array of numbers". But a vector of length n is more than that; it's a distance and direction in n-dimensional space, or (equivalently) it can be taken as a point -- you start at the origin, and then follow the vector from there to the point in question.

In 2-d space, the vector (2,−3) means "two units to the right, and three down", or the point that you reach if you move that way from the origin. In 3-d, (5,1,−7) means "five right, one up, and seven away from the viewer" (or in some schools of thought, seven toward the viewer), or the point there. With more dimensions, it becomes pretty much impossible to visualise, but conceptually it's the same.

We use vectors to mean things in LLMs. For example, the vectors of logits that come out of the LLM (see the [last post](https://www.gilesthomas.com/2025/08/what-ai-chatbots-are-doing-under-the-hood)) represent the likelihood of different next tokens for an input sequence. And when we do that, it's often useful to think of that in terms of defining a high-dimensional space that the meaning is represented in.

### Vocab space

The logits that come out of the LLM for each token are a set of numbers, one per possible token, where the value in each "slot" is the LLM's prediction of how likely the associated token is to be the next one.

The GPT-2 LLM that the book is covering uses a tokeniser with 50,257 tokens -- its vocabulary size is 50,257 -- so each logits vector is 50,257 items long. Token 464 is "The", so the number at position 464 in a logits vector is how likely the next token is to be "The", relative to the others.

We can see each logits vector as being a vector in a 50,257-dimensional space [1](https://www.gilesthomas.com/2025/09/maths-for-llms#fn-1); every point in that space is a different combination of possibilities for the next token to choose from our tokeniser's vocabulary to continue the sequence. I'll call this a _vocab space_.

That's a kind of "messy" vocab space, though -- let's consider two logits vectors, both points in that space, for an imaginary LLM that has a vocabulary of just three tokens. The first is (1,2,3), and the second (−9,−8,−7). Those both mean that the first token ID (with the smallest number) is least likely, the second is more likely than that, and the last, with the largest number, is most likely.

Having two points in the space that mean the same thing seems redundant. To tidy things up, we can run a vector in this messy vocab space through the [softmax function](https://en.wikipedia.org/wiki/Softmax_function) -- that will give us a list of probabilities. I'm personally treating softmax as a kind of magic, but the important thing about it from this perspective is that it takes these messy "likelihood" vectors and returns a set of numbers, all between zero and one, that represent probabilities -- which means that the numbers in the result set sum up to one. Importantly, all different vectors that represent the same set of probabilities when expressed as logits will map to the same vector in the post-softmax space. For example, both (1,2,3) and (−9,−8,−7) map to the same probability distribution, about (0.09,0.24,0.66) [2](https://www.gilesthomas.com/2025/09/maths-for-llms#fn-2).

> Note: the two specific "messy" vectors I used were chosen because they work out to the same probabilities. There are other vectors that express the same "ranking", with the first being least likely, the second more, and the third most likely, that have different probability distributions. For example, (1,2,5) has the same ranking, but it's hopefully obvious that we're saying that the third token is much more likely compared to the others than it was in (1,2,3) -- and that would be reflected in the softmax, which would be something like (0.02,0.05,0.94).

So, we have two kinds of vocab space. A vector in either of them represents likelihoods for a token; there's a "messy" unnormalised space, where the same probability distribution can be expressed in different ways, and a neat, tidy normalised one, where we just use real probability distributions.

One extra thing before we move on; an obvious minimal case in the normalised vocab space is a vector where all of the numbers are zero apart from one of them, which is set to one -- that is, it's saying that the probability of one particular token is 100% and it's definitely not any of the others. This is an example of a _one-hot vector_ (not super-inventive naming) and will become important in the next post.

So: that's one use of a high-dimensional space; let's look at another one.

### Embeddings

An embedding space is a high-dimensional space where vectors represent meanings. If you look at them as points rather than directions/distances, similar concepts are clustered together in the space.

Now, "meaning" is of course very dependent on what you're using the meaning for. For example, you can imagine an embedding space where the points representing "domestic cat", "lion" and "tiger" were all quite close together in one cluster, and "dog", "wolf" and "coyote" made another cluster some distance away (both clusters being within an area that meant something like "animal"). That would be a useful representation space for a zoologist, grouping felines and canines together.

But for more day-to-day use, a different space that grouped domestic animals like "cat" and "dog" closely, in a separate cluster from wild-and-possibly-dangerous animals might be more useful.

So there are vast numbers of possible embedding spaces, representing different kinds of meanings for different purposes. You can go all the way from rich spaces representing complex concepts to "dumb" spaces where you just want to cluster together concepts by the parts of speech that they represent -- verbs, nouns, adjectives, and so on.

The one counterintuitive thing about embedding spaces, at least for me, is that quite often, we don't care much about the lengths of the vectors we use. We might treat (1,2) and (8,16) as being essentially the same embedding vector in a 2-d space because they point in exactly the same direction. [3](https://www.gilesthomas.com/2025/09/maths-for-llms#fn-3)

This becomes important with embeddings because of a mathematical operation called the dot product.

### The dot product

The _dot product_ is an operation that works on two vectors of the same length. It simply means that you multiply the corresponding elements, then add up the results of those multiplications:

(abc)·(def)=a·d+b·e+c·f

Or, more concretely:

(234)·(102)=2×1+3×0+4×2=2+0+8=10

This is useful for a number of things, but the most interesting is that the dot product of two vectors of roughly the same length is quite a good measure of how close they are to pointing in the same direction -- that is, it's a measure of similarity. If you want a perfect comparison, you can scale them both so that they have a length of one, and then the dot product is exactly equal to the cosine of the angle between them (which is logically enough called _cosine similarity_).

But even without that kind of precise normalisation (which requires calculating squares and roots, so it's kind of expensive), so long as the vectors are close in length, it gives us meaningful numbers -- so, for example, it can give us a quick-and-dirty way to see how similar two embeddings are.

Unfortunately the proof of why the dot product is a measure of similarity is a bit tricky, but [this thread by Tivadar Danka](https://x.com/TivadarDanka/status/1964025774329221129) is reasonably accessible if you want to get into the details.

OK, so we have high-dimensional spaces, we're associating meaning with them, and we have a way to compare vectors in them. What can we do with them apart from that?

### Projections by matrix multiplication

A quick refresher: matrices are just vectors stacked together; if you write the vector (2,−3) like this:

(2−3)

...then you can stack it "sideways" with another vector, say (5,1), to make a matrix like this:

[25−31]

Or, if you write it horizontally like this:

(2−3)

...then it can be stacked vertically with the same other vector like this:

[2−351]

The size of a matrix is written in the form r×c, where r is the number of rows, and c is the number of columns. So both of the above are 2×2 matrices; here's a 2×3 one:

[2−3751−8]

Matrices can be multiplied together; hopefully you remember that from your schooldays, but I wrote [a refresher](https://www.gilesthomas.com/2025/02/basic-neural-network-matrix-maths-part-1) back in February if you'd like to remind yourself. It also covers some useful neural net stuff :-)

You hopefully also remember that matrix multiplications can be used to do geometric transformations. For example, this 2×2 (two rows, two columns) matrix:

[cosθ−sinθsinθcosθ]

Let's call it R. It can be used to rotate points in a 2-d space around the origin anticlockwise by θ degrees. To do that, you put all of the points into a matrix, one point per column (like the first example above), giving a 2×n matrix -- let's call it X. We multiply that one by the rotation matrix:

Y=R·X

...and you have a new matrix with the rotated points. That will have the shape 2×n as well, of course, because a 2×2 matrix times a 2×n one takes its number of rows from the first one and its number of columns from the second.

> NOTE: just to confuse things a bit: the way we're taught to do this kind of thing at school is the standard mathematical practice, and that's how I showed it above. The "points" that we're starting with are written as column vectors "stacked side-by-side" to make up a 2×n matrix and then we multiply our rotation matrix by it, R·X. However, in machine learning, people tend to "stack up vertically" a bunch of row vectors, eg. n×2, so the multiplication is the other way around: X·R. In computing terms, we are storing points in row-major rather than column-major format. [This post](https://www.gilesthomas.com/2025/02/basic-neural-network-matrix-maths-part-2) explains why, and I'll switch to using that from now on.

One way of thinking about that rotation matrix R is that it's a bit like a function, taking a set of points in a matrix and returning another set of points that are the original ones rotated.

An alternative way is to think of it projecting between two different 2-d spaces, the second space being rotated around the origin by θ degrees from the first. That's a relatively philosophical point in this case -- both models work well enough.

But when working with 3-d graphics, people use larger matrices -- simplifying a bit, you might use a 3×2 matrix to take a collection of n points in 3-d space, expressed as a n×3 matrix (remember that we're using row-major matrices now), multiply them as X·R, and wind up with those original points projected into two dimensions so that they can be displayed on a screen. [4](https://www.gilesthomas.com/2025/09/maths-for-llms#fn-4)

And that leads us to a more general statement: matrices can project between different multidimensional spaces. More specifically, when using row-major values, a d1×d2 matrix projects from a d1-dimensional space to a d2 dimensional space. The numbers in the matrix determine what kind of projection it is.

So, a 2×2 matrix projects points between different 2-d spaces, likewise a 3×3 one will project points between 3-d spaces, but a 3×2 matrix can project from a 3-d space to a 2-d one.

And we can make it even more extreme! A 50257×768 matrix can be seen as a projection from a 50,257-dimensional space to a 768-dimensional one, and a 768×50257 one would project from a 768-dimensional space to a 50,257-dimensional space. (You'll see why I chose those specific numbers in the next post, though you've probably spotted the relevance of the 50,257.)

It's important to note that the projections can be "lossy", though. If you did the two projections above, one after the other, you'd lose information when you reduced the number of dimensions that you could never get back, no matter what matrices you used.

A nice mental model for that is the 3-d to 2-d projection for computer graphics -- if you did a perspective projection of, say, two squares -- one large and distant, one smaller and closer -- to a 2-d plane, then they might wind up the same size. If you then projected back to 3-d, you just wouldn't have the information needed to work out what their respective sizes and distances were in the original.

So: matrix multiplications are projections between different spaces, with potentially different numbers of dimensions. But they're also something else.

### Neural networks

A single layer in a neural network is calculated like this (again, see my [post on matrices and neural networks](https://www.gilesthomas.com/2025/02/basic-neural-network-matrix-maths-part-1), and perhaps the [follow-up](https://www.gilesthomas.com/2025/02/basic-neural-network-matrix-maths-part-2)):

Z=ϕ(XWT+B)

If we ignore the activation function ϕ and the bias term B, we get this:

Z^=XWT

(The "hat" over the Z is just to express the fact that it's not the full calculation.)

Now, for a neural network, X is our input batch, so it's n×din -- one row for each item in the batch, and one column for each input value in that item.

Our weights matrix W is dout×din -- dout being the number of outputs. We transpose it (that's what the superscript "T" is there to say in WT), which means that we swap around rows and columns, making it a din×dout matrix.

So our result Z^ from the neural network with no bias or activation function is n×dout.

And that takes us to the final core idea I've found useful while working through this: disregarding the activation function, a single layer of a neural network (often abbreviated to _linear layer_) is not much more than a matrix multiplication -- so it is, effectively, a projection from a space with as many dimensions as it has inputs to a space with the same number of dimensions as it has outputs. The bias just adds on a linear "shift" after that.

So, while a normal neural network needs its activation function to do its job, if we want to project between dimensions, a layer without one is a quick and easy way to do that.

### Wrapping up

So, those are the basic mathematical concepts that I've needed so far to understand LLMs. As I said at the start, there really isn't much there beyond high-school maths. The matrices are larger than the ones we're taught, and the high-dimensional spaces are a bit weird, but the actual mathematics is pretty simple.

Up next: how do we put all of that together, along with the high-level stuff I described about LLMs in [my last post](https://www.gilesthomas.com/2025/08/what-ai-chatbots-are-doing-under-the-hood), to understand how an LLM works? [Here's my explanation](https://www.gilesthomas.com/2025/09/how-do-llms-work).

Thanks to everyone for the comments -- especially bad_ash and ThankYouGodBless. Their feedback prompted me to fix something misleading in the section on neural networks as projections, and to add a proper section on the dot product -- something I’d previously left tucked away in one of the posts I linked to.