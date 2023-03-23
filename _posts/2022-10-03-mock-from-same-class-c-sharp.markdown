---
layout: post
title:  "How to mock a function from the same Class with moq"
date:   2022-10-03 10:05:01 +0100
langugage: C#
categories: tutorial
---
Disclaimer: I want to show you what is possible, I strongly recommend not mocking a function
from the same class because you should probably extract the function in a different class.
But here is a possible solution anyway.

Suppose we have the NumberGenerator class with an interface with two functions and want to [moq][moq-docs]
GetNumberFive, so that the function returns 4.
The desired function must be virtual because moq must override the function and this is only possible if you
make a function ``virtual` in C#.
{% highlight csharp %}
public class NumberGenerator : INumberGenerator
{
    public virtual int GetNumberFive()
    {
        return 5;
    }

    public int MultiplyTwoWithValueFromGetNumberFive()
    {
        return 2 * GetNumberFive();
    }
}
{% endhighlight %}

Now lets look at the testing code.

{% highlight csharp %}
public class Tests
{
    private NumberGenerator _numberGenerator = null!;
    private Mock&lt;NumberGenerator&gt; _numberGeneratorMock = null!;

    [SetUp]
    public void Setup()
    {
        _numberGenerator = new NumberGenerator();
        _numberGeneratorMock = new Mock&lt;NumberGenerator&gt;
        {
            CallBase = true
        };
    }
}
{% endhighlight %}

We created two NumberGenerator to test our code. When you want to mock a function from the same
class, you have to use the _numberGeneratorMock. The trick is to set <b>CallBase</b> true,
because it is used to specify whether the base class virtual implementation will be invoked for mocked
dependencies.

So our normal tests looks like this:
{% highlight csharp %}
[Test]
public void Test1_GetNumberFive()
{
    var num = _numberGenerator.GetNumberFive();
    
    Assert.AreEqual(5, num);
}

[Test]
public void Test2_MultiplyTwoWithValueFromGetNumberFive()
{
    var num = _numberGenerator.MultiplyTwoWithValueFromGetNumberFive();
    
    Assert.AreEqual(10, num);
}
{% endhighlight %}

And our test with `_numberGeneratorMock` can look like this:
{% highlight csharp %}
[Test]
public void Test3_MultiplyTwoWithValueFromGetNumberFive_MockFunction()
{
    // assign
    _numberGeneratorMock.Setup(n => n.GetNumberFive()).Returns(4);
    
    // act
    var num = _numberGeneratorMock.Object.MultiplyTwoWithValueFromGetNumberFive();

    // assert
    Assert.AreEqual(8, num);
}
{% endhighlight %}
Here you use the `Setup` method as usual (when using moq). And in the act step you need to
acces the `Object` from your mocked class which you want to test.

[moq-docs]: https://moq.github.io/moq4/
