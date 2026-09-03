---
link: https://maurycyz.com/misc/fec/
excerpt: >-
  This article is not about any particular method of error correction, but about
  the general aprinciple that underlies all of them.

  The goal is to convey information as reliably as possible over a noisy
  communication channel using fixed power and time.
slurped: 2026-08-31T09:37
title: "The principle of error correction: (Maurycy's blog)"
---

**2024-01-12**

This article is not about any particular method of error correction, but about the general aprinciple that underlies all of them. The goal is to convey information as reliably as possible over a noisy communication channel using fixed power and time.

I will try to send a 10 bit message over a simulated pair of wires. Let's use 1 bit per second: a one as 1 volt, and a zero is -1 volts.

With this scheme, the message 0b0110111100, looks like this:

![](app://obsidian.md/data.png)

Now, let's add a lot of noise:

![](app://obsidian.md/noisy.png)

Notice the scale change.

With the added noise, the data looks completely unrecognizable. But while the plot shows 100 Hz of bandwidth, because the signal only changes once per second, the receiver can average one second of data, removing out most of the noise:

![](app://obsidian.md/avg.png)

The message can be recovering by comparing the averaged voltage to 0. Anything above it is a one, anything below is a zero:

![](app://obsidian.md/rx.png)

The recieved bits are identical to the transmitted ones, but this isn't always the case: Out of 10,000 simulation runs, only 59% were sucessfull.

## A Crazy Idea:

Let's replace the 10 bit message with a 100 bit message. This 100 bit "codeword" comes from a "codebook" of random 100 bit sequences for each possible 10 bit message. To avoid confusion, I will refer to the bits of the codeword as a symbols, and bits of the original 10 bit message as a bits.

This is what the codeword for 0b0110111100 looks like:

![](app://obsidian.md/data2.png)

This is that same codeword with the same level of noise as in the last example:

![](app://obsidian.md/noise2.png)

The received signal is averaged to recover each symbol of codeword:

![](app://obsidian.md/avg2.png)

Because each average is only 1/10th of a second, there's a lot more residual noise then when sending the data directly. After thresholding, it looks like this:

![](app://obsidian.md/rx2.png)

Because of the increased noise, a whole 30 symbols (a third of the codeword) are wrong here. However, the original message can still be recovered by finding the closest codeword in the codebook to the received data (least amount of different symbols).

Running the simulation 10,000 times, a whole 77% of messages are received correctly: half the error rate from before!

It seems weird that sending more symbols, _increasing_ bandwidth and _reducing_ the signal-to-noise ratio would improve reliability.

Symbol errors are more frequent, but because noise is proportional the square root of bandwith, the noise is only increased by sqrt(10). Additionally, since the codewords are randomly chosen, it takes a lot of symbol-flips to change them:

There are 2100 = 1 267 650 600 228 229 401 496 703 205 376 possible codewords.

... only 210 = 1024 are valid messages.

In my randomly generated codebook, valid messages are separated by a minimum of 32 bits, so at least 16 errors are needed to change the message, as opposed to just 1. With luck, messages with many more errors can be successfully decoded, like in the example. (The average seperation is on the order of 40 bits.)

We don't have to stop here: As the codeword length and message length increases, the [redundancy continues to outpace the noise, approaching 100% accuracy.](https://en.wikipedia.org/wiki/Noisy-channel_coding_theorem) ... or at least when the ratio of (Power per bit)/(Noise power in 1 Hz) is above -2.1 dB.

Without changing the codebook, it's possible to further improve the result. Instead of thresholding the signal to binary finding the codeword with the smallest amount of symbol flips, the receiver can directly compare the received signal with the expected one for each codeword:

![](app://obsidian.md/corr.png)

Codeword 444 wins, which in binary is 0110111100, the original message!

Doing this improves the success rate to 95%. It also makes it clear why the symbol error rate does not cause more errors matter: The decoder isn't even looking at symbols. It's matching entire codewords.

The symbol rate and bandwidth can and should be as high as possible to keep the codewords maximally far apart.

## The codebook:

The codebook I used was randomly generated, with a constraint that each new codeword must be at least 32 bits apart from existing ones. A purely random codebook does work fine, but it is possible to get particularly bad pairs of codewords if you are unlucky.

I did this to demonstrate that there is absolutely nothing special about the code itself that allows error correction to work, just that it takes a lot of symbol-flips to move between valid codewords. In fact, random codebooks are _optimal_ (as the codebook size increases) but very expensive to decode.

As the message gets longer, codeword matching becomes exponentially harder: Each codeword is longer, and the number is 2(number of bits).

If you wanted to use 32 bit messages, the decoder would have to check 4 billion codewords!