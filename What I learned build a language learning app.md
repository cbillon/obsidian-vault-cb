---
link: https://chadnauseam.com/language/what-I-learned-building-a-language-learning-app
---
# What I learned building a language-learning app 

I'm building an app to teach myself French, and I wanted to share what I learned in the process of creating it! More on the app at [the bottom of the article](https://chadnauseam.com/language/what-I-learned-building-a-language-learning-app#Conclusion).

We are slowly entering an era of digital personalized learning, which will be much more efficient and much more fun than traditional classroom-based learning.

Creating a digital "learning system" is at least as hard as tutoring someone. Everything that you need to know to be a good tutor, you need to implement in code in your digital learning system. How to tell when the learner is stuck, what they need to review, and so on, all needs to be specified in advance rather than relying on your intuition. But digital systems has the advantage that it can scale to a potentially unlimited number of people, whereas you can only tutor one person at a time.

Because we spend so much of our lives learning, and because classroom-based learning is so much less effective than having a good tutor is, it follows that creating an digital learning system is extremely valuable, provided that people actually use it and that it gets close to the efficiency of having a tutor.

The foundation of effective learning systems is almost always a technique called "spaced repetition". This is partially because a computer can implement spaced repetition even better than a human can, and because it is extremely effective. Spaced repetition is not applicable to every field, but it is extremely relevant to language learning, so it deserves a quick introduction for those readers who are not familiar with it.

## What is Spaced Repetition? 

(If you already know what spaced repetition is, you can skip this section.)

Spaced repetition is a learning system normally used with flashcards. The naïve way of using flashcards is to review all of them every session. But as you start to have thousands of flashcards, this becomes a huge waste of time. No one can learn a lot of flashcards this way – your workload per day would be proportional to the number of cards.

A much more effective way would be to only review each flashcard right as you were about to forget it. The effect is that newly introduced and more difficult flashcards are shown more frequently, while older and less difficult flashcards are shown less frequently[[1]](https://publish.obsidian.md/#fn-1-5ef04373b07cc3d8). This is accomplished using an algorithm called a "scheduler". You tell the scheduler each time you review a card, and you say whether you successfully remembered it or forgot it. The scheduler uses this information to try to figure out the difficulty of the card, and from that it can figure out the best next time to show it to you. For a very difficult card you just forgot, it might show it to you again in 3 minutes. For an easy card you just remembered, it might show it to you again in 3 years. With spaced repetition, even if you add new cards every day, your workload can remain basically constant.

Schedulers are simple and adaptable. You don't need a specialized scheduler for every task. The same one can be used for memorizing anatomy and for memorizing words. The best publicly available scheduler I know of is called FSRS. 

# Retention 

A great learning system that nobody wants to use is, in some sense, not very effective. This section covers how to make a system that people actually use.

## User interface 

Anki and other flashcard-based SRS apps solve a very general problem. You can basically use it to learn anything. The research company Ink and Switch suggests [dividing software into "knives" and "avocado slicers"](https://www.inkandswitch.com/essay/malleable-software/).

1. Knives solve a general problem, but are sometimes difficult to learn how to use (and sometimes can be used incorrectly or ineffectively).
2. Avocado slicers are trivial to use, but are only good for one specific task. 

In this analogy, Anki is a knife and my application is an avocado slicer. It is worth thinking of how to make a system like mine more general – maybe a souped-up version of Anki that supported LLM-based grading would make my system just a special case.

That said, there is one feature missing from Anki that is just critical: notifications that remind you to study. The most common reason people stop studying is because they fall out of the habit and simply forget. This use case for notifications is profoundly prosocial, as it removes the possibility of simply forgetting to study. While it may seem small or extraneous, any learning app that has these notifications will see its users much more likely to succeed. Sometimes the things that make the biggest differences are the small things that seem like they shouldn't matter.

## The Feeling of Progress 

There is a classic "formula" for motivation, which is simplistic but quite predictive. 

![procrastination-equation](http://alexvermeer.com/wp-content/uploads/procrastination-equation2.png)  
(Graphic from [How We Use the Procrastination Equation by Alex Vermeer and Jimmy Rintjema](https://alexvermeer.com/how-we-use-the-procrastination-equation/))

`Expectancy × Value` is simply the expected value of the task. `Impulsiveness` is an innate/exogenous quality of the person whose motivation is in question, and `Delay` indicates now far that value is in the future. The result is our overall motivation to do the task.

The one change I would make to this classic formula is to modify `Value` to be explicitly `(Reward - Effort)`.  

I encourage interested readers to try an app like Duolingo with this equation in mind. It is an incredible case study in how to optimize this formula. Duolingo provides the maximum feeling of reward for a minimum of effort. In each session, you learn 1-2 new words, then practice them many times. Then, the app congratulates you for having learned the new words. This is not very effortful (it's not difficult to remember a word you learned a few seconds ago), and feels very productive (at the end of the session you feel like you know the word very well). 

The issue is that it is not a very time-efficient way to learn. The most efficient thing you can do is recall a word you were about to forget. This has a huge impact on how long you will remember it. But recalling a word you were about to forget, from deep in your long-term memory, takes a surprising amount of effort. Recalling a word you learned a few seconds ago from your short term memory takes almost no effort, but it doesn't induce long-term retention.[[2]](https://publish.obsidian.md/#fn-2-5ef04373b07cc3d8)

I don't think there's a way around this. Learning a language quickly will always require effort at the beginning. Our only option is to eliminate all causes of unproductive or "wasted" effort, and then to make the reward as great and as visible as possible, which brings me to the next section.

### The feeling of ease 

To make matters worse, we are accustomed to things getting easier as we get better at them. When we learn to ride a bike, at first it is very difficult to balance, but over time we get better and the task feels easier, until riding a bike becomes an incredibly relaxing and pleasurable activity that requires virtually no conscious thought or attention. 

But, most likely everyone reading this is at about the same level of bike-riding ability. Probably few of us can do wheelies or ride backwards or ride down the center of a balance beam. This is because while we got better, the task stayed the same. Once the task became easy, we stopped learning. This works because the task of riding a bike isn't that difficult (most children can learn to do it in a day or so), and most people have no interest in more advanced skills that can't be learned in a day.

But that doesn't apply to language learning. You cannot become remotely competent in a language in one day. The only option is to start with super easy challenges, and then replace them with gradually harder challenges as you master the easy ones. 

This creates a problem. A highly effective system keeps the difficulty at the perfect level at all times. But this means it always feels moderately challenging. While it shouldn't feel overwhelming, it probably won't feel easy either. Time spent doing things that are easy is rarely productive, but it's important because it makes us feel like we have actually progressed, actually learned something. 

I can think of two classes of solution to this problem:

1. "external solutions," like progress bars and meters. I call these external because they're controlled by the app and don't necessarily reflect real progress. But they can be very motivating.
2. "internal solutions", where the user demonstrates to themselves that they can now do a task they couldn't do before.  These are "internal" because the user internally remembers a time when they couldn't do the task and compares it to the present (where they hopefully can).

One idea I'm enamored with is a progress bar is indicating how close the learners is to being able to read a particular book or watch a particular movie. This would be very motivating, especially if, once completing your progress bar, the user actually was provided the book or movie. With the right choices of books and movies, the user might "unlock" one every couple weeks.

(This could be very useful in a classroom setting, but it's a bit harder for a free language learning app that doesn't have the rights to a bunch of books and movies. Regardless, "You know 62% of the vocabulary in _Le Petit Prince_" would be incredibly motivating.)

A simpler idea is to show a short paragraph to the user that they can't yet read and tell them, "After a few days of practice, you'll be able to read this no problem." Then, after those few days of practice have passed, you provide the same paragraph to the user. Hopefully they'll remember, "Oh, I used to not be able to read this and now I can!"

# Effectiveness 

Of course, even if people use your learning system, it's not a good system unless it results in a lot of actual learning. This section covers that

## Breaking concepts into small chunks 

A key to effective teaching is to break concepts into the smallest chunks possible, and then introduce these one at a time. You might see this in math, where a concept introduced all at once would be very difficult to understand, but if it were broken up into many tiny chunks with each chunk introduced one at a time, the learner would find each chunk simple to understand and get through the whole thing without a problem. The smaller the chunk size, the more people will be able to learn the subject and the less effort it will take to learn it.

As an example, the site Math Academy has broken up a large subset of math into small "knowledge chunks", and they have these amazing graphs that show all the different chunks they represent in their system and the dependencies they have on one another:

![math-academy-graph.png](https://publish-01.obsidian.md/access/8a529a39ad1753175d73ccc6abc0547e/images/math-academy-graph.png)

Here is Math Academy's full graph:

![math-academy-graph-bigger.png](https://publish-01.obsidian.md/access/8a529a39ad1753175d73ccc6abc0547e/images/math-academy-graph-bigger.png)

Fortunately we don't need anything quite so sophisticated for language learning haha. But Math Academy and their dedication to breaking things into tiny chunks was a big inspiration for my project.

### Word Chunks 

With something like math. It can actually be a lot of work to break a complicated concept into these tiny chunks. With language learning, we have the advantage that there's a ready-made concept for us to use as our basis for chunking: words. That is, you can teach/learn words one at a time.

You might think that this is so obvious that it barely is worth mentioning. But I bring it up to say that we can do better. It turns out that words are not the smallest possible chunk that we can use as our basis for teaching a language.

Take the word "rose". As a verb, it can be a conjugation of "to rise". But as a noun, it is a type of flower. Or take the word "have". As a verb, it can mean "to possess" (I have eggs), or as an auxiliary it can mean "did in the past" (I have gone to the store).

So each word can be broken up into many `(word, meaning)` pairs. These make smaller chunks than simply using words. 

The problem is that `(word, meaning)` pairs are difficult to work with programmatically. I don't know how to take a sentence and identify the meaning of each word, in such a way that words with the same meaning will be considered equivalent. So I ended up using `(word, part-of-speech, lemma)` triples instead. (A lemma is the "dictionary form" of a word, for example the lemma of "runs" is "run".) Using `(word, part-of-speech, lemma)` as our chunks is worse than `(word, meaning)`, because it doesn't differentiate between the different meanings of "bat", "bank", "spring", etc., but it's still a huge improvement over simple `word`.

It's my experience that `(word, part-of-speech, lemma)` work very well for English, French, Spanish, and German. Languages like Turkish, Mandarin, and Japanese are different enough that it would probably work more poorly for them. Unfortunately, I don't know any of these languages well enough to speculate what would work well for teaching them.

### Multiword Chunks 

Even if you know the meaning of every word in a sentence, you might still not know the meaning of that sentence. This is because words can take on their new meanings when used as part of a phrase. For example, if someone says "you'd better not", you'd better know what `'d better` means separately from the meaning of `had` and `better`. 

Wiktionary calls these "[multiword terms](https://en.wiktionary.org/wiki/Category:English_multiword_terms)", and they have an very extensive lists of them for many languages. So in addition to teaching `(word, part-of-speech, lemma)`,  you should also teach "multiword terms".

### Introducing chunks in the right order 

The obvious order is to teach these chunks in proportion to how common they are. For some reason, lots of apps and teachers fail at this point. For example, how many people learned the Spanish words for every color very early on in Spanish class in school? You might know that "azul" means "blue", but this is not actually a very common word. In fact, it's only the 957th most common word in my database. Meaning that there are 956 words you should learn before "azul", if you're learning in order of frequency. 

This happens because classes want to be able to teach "real sentences" like "the house is blue". But those kind of sentences are actually pretty rare in real life. On the other hand, if you know the top 100 most common English words, you can say complex things like "how could you do this to me?".  Imagine you're watching a movie in English, are you more likely to hear "the house is blue" or "how could you do this to me?"

I don't know if there's a term for the difference between these types of sentences. I would say that "the house is blue" is a "concrete sentence" and "how could you do this to me" is an "abstract sentence". Concrete sentences have lots of nouns and adjectives. Abstract sentences have lots of pronouns and verbs. At the beginning of your language learning journey, if you learn words in order of frequency, you will be able to say lots of abstract sentences and not very many concrete sentences. This is a good strategy, because abstract sentences are very common in real life and you can understand most of them by learning just a hundred or two words. (Compared to the thousands of words needed to have a good understanding of concrete sentences.)

### Final notes 

The above mostly focused on chunks needed to effectively learn to read a language whose writing system you already know. More types of chunks are needed for listening and speaking and writing. And when an english speaker wants to learn Japanese,  they would also benefit from chunks specialized to the Japanese writing systems. I don't have a fully satisfactory solution to these parts of the system, so I can't go into depth here.

## Testing chunks 

The most common spaced repetition strategy is simple flashcards. Each knowledge chunk gets one flashcard and each flashcard corresponds to one chunk. When it's time to review a chunk, the flashcard is shown.

This is a very simple system, but it has the advantage of being extremely flexible and extremely effective. But it has some flaws. One issue is that it's not obvious how to get "sentence practice" with flashcards. That is, you might know what "I" means, "can" means, and "go" means. But it's important to regularly see the words in sentences such as "I can go". 

This is an area where we need to separate our concepts. Knowledge chunks should actually be something internal to the system, not overly exposed to the user. What the user actually sees are "challenges", each of which tests one or more knowledge chunks. 

For example, what "I" means, "can" means, and "go" means are all knowledge chunks. But "translate the sentence 'I can go.'" is a challenge that tests all three of those chunks.

The user will respond to the challenge, and the system is responsible for taking the user's response and determining if it counts as a successful or failed repetition for each chunk. I call this part "grading". One reason that flashcards were so popular is because grading is very easy. (You just ask the user!) On the other hand, it used to be difficult to automatically grade a user's translation.

Nowadays, LLMs are very useful for grading. They can look at the provided sentence and the user's translation, and tell you exactly what words the users translated correctly and which ones they failed at translating. LLMs are not yet equally good for all languages, and I've specifically heard lots of horror stories w.r.t. using them for learning Japanese. But for indo-european languages with lots of training data, they are fantastic.

# Conclusion 

There's a lot I wanted to get to in this post but wasn't able to (it's already long enough IMO). For example, something important is a feeling of autonomy, specifically in the user being able to choose what they study. This has the effect of making the user feel more in control of what they're learning, which makes it much more fun.  There's also the question of "placement tests", which are important to avoid wasting the time of users who already have some pre-existing knowledge from previous attempts at learning. I would like to get into all of these in a future post, but this is enough for now. I hope you learned something, and I hope you enjoyed reading!

The app is [yap.town](https://yap.town/), and in addition to French I've also added Spanish and German as those are the languages my friends wanted to learn.

It's just a free/unmonetized hobby project, so don't expect something super polished please, but I think it's pretty good! It runs in the browser, works offline, requires no account or login, is mostly written in Rust, and the source is on [GitHub](https://github.com/anchpop/yap)!

---

1. This part of the description was taken from [Wikipedia.](https://en.wikipedia.org/wiki/Spaced_repetition)[↩︎](https://publish.obsidian.md/#fnref-1-5ef04373b07cc3d8)
2. Of course, I'm sure Duolingo has thought about this issue and there are ways to use Duolingo in a higher effort, more time-efficient way.[↩︎](https://publish.obsidian.md/#fnref-2-5ef04373b07cc3d8)

Links to this page

[i-built-an-app-to-talk-to-my-dad](https://chadnauseam.com/coding/random/i-built-an-app-to-talk-to-my-dad)