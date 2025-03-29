---
layout: post
title:  "How-To: Create an array of random values in a catch2 unit test"
---

# Background

There can be unit tests where it makes sense to generate some random data that the test operates on.  An example could be a compression algorithm that operates on arbitrary input data.  A test that runs on random data can complement other tests that operate on predetermined data.

However, because this is a unit test it should also be reproducible.  This means that the random seed should be obtainable so that the unit test can be rerun with the same data given an input seed.

For this post I will be referencing catch2 unit test library, but it probably applies to others as well.

# Research

The reason that this page exists in the first place is that it is not so trivial and prior examples seem to be difficult to find.  The catch2 [documentation](https://github.com/catchorg/Catch2/blob/devel/docs/generators.md) focuses on generating single random values for successive runs.  This doesn't match my use case because I want a array of random values for a single test run.

There is at least one discussion on [stack overflow](https://stackoverflow.com/questions/78034419/c-catch2-generate-a-array-with-random-values).  The selected answer comments on how the catch2 documentation on this is a bit lacking (which I agree).  The code in the answer looks sensible enough, but is complicated.  I then tried it and found that the random data does not change on successive runs.  

At this point I rethought the problem and arrived at a simpler solution.

# Solution

The first piece of the puzzle that I needed was the seed value to use for the random data.  catch2 generates a seed value (if one is not provided) for its own random operations and it can be used in user code as well.  After we get that seed we can use pretty standard c++ to create the random array.

```cpp
static constexpr size_t data_array_size{ 1024 };

auto seed = Catch::rngSeed();
std::mt19937 gen(seed);
std::uniform_int_distribution<> distr(0, 255);

std::array<uint8_t, data_array_size> byte_data{};
std::ranges::for_each(byte_data, [&gen, &distr](uint8_t &n) { n = static_cast<uint8_t>(distr(gen)); });
```

First we get the seed value from catch2, then use it to seed the random data from the std classes.  Then a foreach is used to write each successive random value to the array.  Because the random data is seeded from the catch2 seed, we can exactly reproduce a set of data by providing the seed value.