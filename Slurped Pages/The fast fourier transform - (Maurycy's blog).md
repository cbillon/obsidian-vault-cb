---
link: https://maurycyz.com/misc/fft/
excerpt: |-
  The discrete fourier transform breaks down a into its constituent frequencies.
  That is, it converts the signal to a sum of sine and cosine waves.
slurped: 2026-08-31T09:30
title: "The fast fourier transform: (Maurycy's blog)"
---

**2023-12-27**

The discrete fourier transform breaks down a into its constituent frequencies. That is, it converts the signal to a sum of sine and cosine waves.

To do this, it helps to use complex numbers, which for our purposes are just points in 2d space. Multiplying one complex number by another is the same as rotating one point the angle of the other one.

Euler's formula, eiθ = cos(θ) + isin(θ), is any easy way to generate numbers on the unit circle. Because of this: xeiθ is the same x rotated by angle θ.

To calculate a frequency component: multiply (rotate) it by a point moving along a circle at the desired frequency and sum the result. Any other frequencies will spend an equal amount of time in sync and out of sync with point, and will cancel out in the sum. However, a component at the same frequency will stay in sync and effect the overall sum.

This whole process then has to be repeated for every frequency of interest.

A really nice visual analogy is taking the samples and wrapping them around the origin, and then taking the center-of-mass of the result. You can play around with this with [any program that can plot functions in polar coordinates](https://www.desmos.com/calculator/u5acrvmhys) plotting r=sin(a*θ). For any value of a other then 1, the graph will spend an equal amount of time on all sides of the origin.

The standard transform uses the same amount of frequency bins as samples, evenly distributed from 0 to one full revolution per sample. This spacing maintains _orthogonality_, the independance of each frequency bins allowing the transform to be reversable.

# Boring python stuff
import cmath
from cmath import pi, e
i = cmath.sqrt(-1)

def fourier(samples):
	transform = [] # Buffer for results
	for bin_no in range(len(samples)):
		# Evenly spaced freqiencies, in radians/sample
		freq = 2*pi / len(samples) * bin_no
		# Total up the rotated samples
		acc = 0
		for j in range(len(samples)):
			# Rotate the sample by the content 
			acc += samples[j] * e**(i*freq*j)
		transform.append(acc)
	return transform

If you play around with this code, make sure to use the magnitude of resulting complex number if you want the amplitude. In Python, this can be done using the abs() function.

This straightforwards implementation works, but is has a problem; At a minimum, it requires the number of samples _squared_ complex multiplications and additions.

This gets slow quick: for 1024 samples, it takes over a million multiplications, and for 44100 samples (1 second of audio) it takes almost 2 billion. It's even worse then that because a complex multiplication is actually 4 normal multiplications and 2 additions.

If doing the transform on short datasets is fast, but longer ones are very slow, could we split them up?

Yes.

The usual method is to split into two smaller transforms, one on the even and another on the odd numbered samples. The outputs can be combined by adding up the result for each bin from both sub-transforms, with the odd ones rotated to account for the time offset between even and odd samples. The rotation needed is different for each frequency, but always increases by a fixed amount between bins so it can be done after the fact.

On it's own, this would give half as many bins as the full sized transform, but that's easy to fix. The second half of the bins correspond to the same frequencies as the lower half, but with an additional 180 degree rotation between every sample: The odd FFT's output can be negated (the same as rotating by 180 degrees) before adding it to the even FFT's output to produce the missing high bins:

# Boring python stuff
import cmath
from cmath import pi, e
i = cmath.sqrt(-1)

def fft2(samples):
	"""
	Compute the Fourier transform of the input, requires the length to be a power of 2.
	Pad the input with zeros before calling if it is not already a power of 2.
	"""
	# Trivial base case
	n = len(samples)
	if n == 1: return samples
	# Compute sub FFTs
	even = fft2(samples[0::2])
	odd = fft2(samples[1::2])
	# Rotation used for combining the sub FFTs
	rot = e**(2*pi*i/n) # Extra rotation per bin
	c_rot = 1 # Current rotation between even and odd
	# Combine them
	combined = [0] * n
	for j in range(n//2):
		p = even[j]
		q = odd[j] * c_rot
		combined[j] = p + q
		combined[j + n//2] = p - q # Negation is a 180 degree rotation
		# Increase the rotation for the next bin
		c_rot *= rot
	return combined

If the input is a power of 2, this repeated division by 2 will go all they way down to a single sample. A single sample Fourier transform only has a single frequency, 0, and a single sample, so no rotation has to be done, the function just returns the sample.

Because the time taken by a transform is proportional to the number of inputs squared, the half sized transforms don't take half the time, but a quarter of the time. Furthermore, we can keep splitting to make it faster.

As a result, this algorithm only does `2 * n * log2(n)` complex multiplications, of which half could be precomputed. On 44100 samples (one second of audio), the minimum number of multiplications is just 68 thousand, almost 3 thousand times better then naive method.