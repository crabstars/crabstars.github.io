---
layout: post
title:  "C# and Rust Iterator Comparison"
date:   2023-08-03 21:27:00 +0100
langugage: C#
categories: interesting
---
I recently learned something interesting about the difference between IEnumerable<T> and List<T>. It's about how we use them when going through data. For example, if we want to find all the even numbers in a list using LINQ, and then we use the "Last" method on it, there's something important to know and that is that a IEnumerable is evaluated when we access the result.

Imagine we have numbers from 1 to 6 in our list. If we use "Last," we'd expect to get 6. But what if we add the number 8 to the list and calling Last again?
Surprisingly or not, when I tried this, it did give me 8! This got me thinking about whether this is normal and if all programming languages allow it.
 It turns out that some languages, like Rust, don't let you do this. It's pretty interesting how different languages handle things in their own ways!


{% highlight csharp %}
var numbers = new List<int>() { 1, 2, 3, 4, 5, 6 };

var evenNumbers = numbers.Where(x => x % 2 == 0);

Console.WriteLine(evenNumbers.Last());
// Output: 6

numbers.Add(8);

Console.WriteLine(evenNumbers.Last());
// Output: 8
{% endhighlight %}

Once I made the LINQ equivalent in Rust, the "iter" and "filter" operations do what's called an "immutable borrow" on the List
(which is a Vec in Rust, but I'll call it List for comparison). This means we can't modify the List anymore because doing so would 
need another "immutable borrow," which is not allowed. We're only allowed to borrow the data to look at or copy it, but not to make changes after the "immutable borrow".
If you want to learn something about the Rust Ownership than I recommend the offical [documentation][https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html]

{% highlight rust %}
fn main() {
    let mut numbers: Vec<i32> = vec![1, 2, 3, 4, 5, 6];

    let evenNumbers = numbers.iter().filter(|&&x| x % 2 == 0);

    numbers.push(8); // cannot borrow `numbers` as mutable because it is also borrowed as immutable

    println!("{}", evenNumbers.last().unwrap());
}
{% endhighlight %}


In Rust, a solution could be to make a copy of the List when we make an Iterator. This lets us modify the original List after creating the Iterator. However, any additions or changes we make to the List won't affect the Iterator we created.

{% highlight rust %}
fn main() {
    let mut numbers: Vec<i32> = vec![1, 2, 3, 4, 5, 6];

    let evenNumbers = numbers.clone().into_iter().filter(|x| x % 2 == 0);

    numbers.push(8);

    println!("{}", evenNumbers.last().unwrap());
}
{% endhighlight %}

In the end, I believe Rust helps you write better and safer code. Once you declare an Iterator for a List, Rust doesn't let you change it, which adds more safety to your programming.
