---
link: https://github.com/bits-and-blooms/bloom
byline: bits-and-blooms
site: GitHub
excerpt: Go package implementing Bloom filters, used by Milvus and Beego. - bits-and-blooms/bloom
twitter: https://twitter.com/@github
slurped: 2025-11-26T11:56
title: "GitHub - bits-and-blooms/bloom: Go package implementing Bloom filters, used by Milvus and Beego."
tags:
  - go
  - filter
  - bloom
---

## Bloom filters

[](https://github.com/bits-and-blooms/bloom#bloom-filters)

This library is used by popular systems such as [Milvus](https://github.com/milvus-io/milvus) and [beego](https://github.com/beego/Beego).

A Bloom filter is a concise/compressed representation of a set, where the main requirement is to make membership queries; _i.e._, whether an item is a member of a set. A Bloom filter will always correctly report the presence of an element in the set when the element is indeed present. A Bloom filter can use much less storage than the original set, but it allows for some 'false positives': it may sometimes report that an element is in the set whereas it is not.

When you construct, you need to know how many elements you have (the desired capacity), and what is the desired false positive rate you are willing to tolerate. A common false-positive rate is 1%. The lower the false-positive rate, the more memory you are going to require. Similarly, the higher the capacity, the more memory you will use. You may construct the Bloom filter capable of receiving 1 million elements with a false-positive rate of 1% in the following manner.

    filter := bloom.NewWithEstimates(1000000, 0.01) 

You should call `NewWithEstimates` conservatively: if you specify a number of elements that it is too small, the false-positive bound might be exceeded. A Bloom filter is not a dynamic data structure: you must know ahead of time what your desired capacity is.

Our implementation accepts keys for setting and testing as `[]byte`. Thus, to add a string item, `"Love"`:

    filter.Add([]byte("Love"))

Similarly, to test if `"Love"` is in bloom:

    if filter.Test([]byte("Love"))

For numerical data, we recommend that you look into the encoding/binary library. But, for example, to add a `uint32` to the filter:

    i := uint32(100)
    n1 := make([]byte, 4)
    binary.BigEndian.PutUint32(n1, i)
    filter.Add(n1)

Godoc documentation: [https://pkg.go.dev/github.com/bits-and-blooms/bloom/v3](https://pkg.go.dev/github.com/bits-and-blooms/bloom/v3)

## Installation

[](https://github.com/bits-and-blooms/bloom#installation)

go get -u github.com/bits-and-blooms/bloom/v3

## Verifying the False Positive Rate

[](https://github.com/bits-and-blooms/bloom#verifying-the-false-positive-rate)

Sometimes, the actual false positive rate may differ (slightly) from the theoretical false positive rate. We have a function to estimate the false positive rate of a Bloom filter with _m_ bits and _k_ hashing functions for a set of size _n_:

    if bloom.EstimateFalsePositiveRate(20*n, 5, n) > 0.001 ...

You can use it to validate the computed m, k parameters:

    m, k := bloom.EstimateParameters(n, fp)
    ActualfpRate := bloom.EstimateFalsePositiveRate(m, k, n)

or

    f := bloom.NewWithEstimates(n, fp)
    ActualfpRate := bloom.EstimateFalsePositiveRate(f.m, f.k, n)

You would expect `ActualfpRate` to be close to the desired false-positive rate `fp` in these cases.

The `EstimateFalsePositiveRate` function creates a temporary Bloom filter. It is also relatively expensive and only meant for validation.

## Serialization

[](https://github.com/bits-and-blooms/bloom#serialization)

You can read and write the Bloom filters as follows:

	f := New(1000, 4)
	var buf bytes.Buffer
	bytesWritten, err := f.WriteTo(&buf)
	if err != nil {
		t.Fatal(err.Error())
	}
	var g BloomFilter
	bytesRead, err := g.ReadFrom(&buf)
	if err != nil {
		t.Fatal(err.Error())
	}
	if bytesRead != bytesWritten {
		t.Errorf("read unexpected number of bytes %d != %d", bytesRead, bytesWritten)
	}

_Performance tip_: When reading and writing to a file or a network connection, you may get better performance by wrapping your streams with `bufio` instances.

E.g.,

	f, err := os.Create("myfile")
	w := bufio.NewWriter(f)

	f, err := os.Open("myfile")
	r := bufio.NewReader(f)

## Contributing

[](https://github.com/bits-and-blooms/bloom#contributing)

If you wish to contribute to this project, please branch and issue a pull request against master ("[GitHub Flow](https://guides.github.com/introduction/flow/)")

This project includes a Makefile that allows you to test and build the project with simple commands. To see all available options:

## Running all tests

[](https://github.com/bits-and-blooms/bloom#running-all-tests)

Before committing the code, please check if it passes all tests using (note: this will install some dependencies):

## Design

[](https://github.com/bits-and-blooms/bloom#design)

A Bloom filter has two parameters: _m_, the number of bits used in storage, and _k_, the number of hashing functions on elements of the set. (The actual hashing functions are important, too, but this is not a parameter for this implementation). A Bloom filter is backed by a [BitSet](https://github.com/bits-and-blooms/bitset); a key is represented in the filter by setting the bits at each value of the hashing functions (modulo _m_). Set membership is done by _testing_ whether the bits at each value of the hashing functions (again, modulo _m_) are set. If so, the item is in the set. If the item is actually in the set, a Bloom filter will never fail (the true positive rate is 1.0); but it is susceptible to false positives. The art is to choose _k_ and _m_ correctly.

In this implementation, the hashing functions used is [murmurhash](https://github.com/bits-and-blooms/bloom/blob/master/github.com/twmb/murmur3), a non-cryptographic hashing function.

Given the particular hashing scheme, it's best to be empirical about this. Note that estimating the FP rate will clear the Bloom filter.

### Goroutine safety

[](https://github.com/bits-and-blooms/bloom#goroutine-safety)

In general, it not safe to access the same filter using different goroutines--they are unsynchronized for performance. Should you want to access a filter from more than one goroutine, you should provide synchronization. Typically this is done by using channels (in Go style; so there is only ever one owner), or by using `sync.Mutex` to serialize operations. Exceptionally, you may access the same filter from different goroutines if you never modify the content of the filter.

## Stars

[](https://github.com/bits-and-blooms/bloom#stars)

[![Star History Chart](https://camo.githubusercontent.com/4dfcf3e93fed143e8d09e0894cec76090741411057829c6d3acf2510bb049fc1/68747470733a2f2f6170692e737461722d686973746f72792e636f6d2f7376673f7265706f733d626974732d616e642d626c6f6f6d732f626c6f6f6d26747970653d44617465)](https://www.star-history.com/#bits-and-blooms/bloom&Date)

## Further reading

[](https://github.com/bits-and-blooms/bloom#further-reading)

Mastering Programming: From Testing to Performance in Go

[![](https://camo.githubusercontent.com/15c89eddb1f26a70a7024cf9f7a671831dc085b6f6488bbb8bc1be5f4371470f/68747470733a2f2f6d2e6d656469612d616d617a6f6e2e636f6d2f696d616765732f492f363166656e654853376b4c2e5f534c313439395f2e6a7067)](https://www.amazon.com/dp/B0FMPGSWR5)