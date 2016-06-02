---
layout: post
title: Learn well your tools
---

## Learning your tools

This time I'd like to talk about tools, the frameworks that we use in a daily basis. 

I often relate to the Gardening metaphor when talking about software. A gardener knows his tools, he knows when to use a certain type of trowel not to damage the plant, he knows when using a bigger type of spade so that his work will be more efficent.
Furthermore a professional gardener knows how to take advantage of a given technique, one that may not relevant in theory but in a specific context may be useful.

We are professional software engineers, we need to use the right tools, we even need to look into other tools in the market and then use a similar one that we have in our storage.
We tend to make one mistake though: we get trapped into our own habits. Habits are good, habits are the realization that we have come to learn something that deeply that we don't even think on doing that.
The trap here is that we may follow certain habits when they are not applicable or even when there is a better pattern for that context.

## One Example is worth a Thousand words 

### Test cases in your testing framework:

Recently I was doing the FizzBuzz coding kata. In this kata we need to print a sequence of numbers but transforming some of them into "Fizz", "Buzz" or "FizzBuzz" given certain rules.

My tests were as follows:

{% highlight cs %}
public class FizzBuzzTest
{
  [Fact]
  public void FizzBuzz_for_two_elements_outputs_two_numbers()
  {
     var fizzBuzz = new FizzBuzz.FizzBuzz();

     string fizzBuzzSequence = fizzBuzz.Get(2);

     Assert.Equal("1 2", fizzBuzzSequence);
  }

  [Fact]
  public void FizzBuzz_for_three_elements_includes_fizz()
  {
     var fizzBuzz = new FizzBuzz.FizzBuzz();

     string fizzBuzzSequence = fizzBuzz.Get(3);

     Assert.True(fizzBuzzSequence.Contains("Fizz"));
  }
}
{% endhighlight %}

I wanted to be explicit so that people could read my tests as documentation. While that is correct, imagine following this pattern in a larger and more complex solution.
Tests become overcomplex, we need to maintain more lines of code.

In this scenario I have a single function that transforms the data: I send an input sequence and I get back an output sequence.
It seems the perfect match to use TestCases. I'm using XUnit test framework so that I did some research and found about InlineData, so I changed my tests:

{% highlight cs %}
[Theory]
[InlineData("1", 1)]
[InlineData("1 2", 2)]
[InlineData("1 2 Fizz", 3)]
[InlineData("1 2 Fizz 4", 4)]
[InlineData("1 2 Fizz 4 Buzz", 5)]
[InlineData("1 2 Fizz 4 Buzz Fizz", 6)]
public void FizzBuzz_sequence_for_given_number_of_elements(string expectedFizzBuzz, int numberOfElements)
{
    var fizzBuzz = new FizzBuzz.FizzBuzz();

    string fizzBuzzSequence = fizzBuzz.Get(numberOfElements);

    Assert.Equal(expectedFizzBuzz, fizzBuzzSequence);
}
{% endhighlight %}

Tests have become more concise and simple, yet they are explicit.

## Creating tools, test cases in Mocha.

A trully proficient gardener may find that in a given context, none of the tools do a good job. Then they just apply patterns that they know from other contexts, even creating new tools, to solve the problem.

We as software engineers should learn more than one language. Even if you are a Ruby developer, by learning different languages, different paradigms you will see *other ways*, you will find different ways to solve different problems, you will come back to your regular environment and you will be a more efficient profesional developer.
Imagine our friend the gardener using always the same shovel no matter what the context is... Probably he won't be hired so often.

A few days after doing the kata, we are doing some work in Javascript. We use Mocha as our testing framework so we did some research to find out if Mocha supports Test cases.
It didn't. It didn't matter, we knew the language and we knew the concept so we implemented something similar.

Here is a test implementation of the tests above for FizzBuzz in Javascript:

{% highlight js %}
import FizzBuzz from '../fizz-buzz';

describe('For given number of elements', () => {
  const testCases = [
    {'1', 1}, 
    {'1 2', 2}, 
    {'1 Fizz', 3},
    {'1 Fizz 4', 4},
    {'1 Fizz 4 Buzz', 5},
    {'1 Fizz 4 Buzz Fizz', 6}
  ];
  
  testCases.forEach((testCase) => {
    const { expectedFizzBuzz, numberOfElements } = testCase;
    
    it('generates FizzBuzz sequence', () => {
      let fizzBuzzSequence = FizzBuzz.Get(numberOfElements);
      
      fizzBuzzSequence.should.equal(expectedFizzBuzz);
    });
  });
});
{% endhighlight %}

Even though Mocha doesn't have an in-build functionality for TestCases, we have learnt well our tools, in this case Javascript ES2015.

Learn well your tools to reduce work. Thanks for reading!
