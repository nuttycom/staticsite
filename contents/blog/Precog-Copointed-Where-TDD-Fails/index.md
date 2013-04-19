title: Precog-Copointed Where TDD Fails
author: Brian McKenna
date: 2013-03-01 18:07
template: post.jade

<p>I&#8217;ve just gotten back from the awesome <a href="http://mloc-js.com/">mloc.js</a> conference. There was a talk about compiling C# to JavaScript and one of the benefits explained was static types. Someone from the audience asked, <strong>who needs types when you do Test Driven Development?</strong></p>
<p>I tried to address the question in <a href="http://brianmckenna.org/files/presentations/mloc-roy/">my talk on Roy</a> but I talked to some developers afterwards and they thought that TDD is the only tool necessary for writing code that satisfied their specifications!</p>
<p>So here&#8217;s a blog post where I&#8217;ll try to show two cases where TDD doesn&#8217;t solve my specifications. In the first case, we get a free proof (i.e. TDD is unnecessary). In the second case, we aren�t able to encode some of our constraints in tests or types, so we need to use mathematical reasoning to derive a satisfactory implementation (i.e. TDD isn&#8217;t enough).</p>
<h2 dir="ltr">Free Proofs</h2>
<p>Imagine that we know that we need a function that returns its argument. The input is polymorphic &#8211; it can be any value. In a typed language the function would look like:</p>
<pre>&#945; &#8594; &#945;</pre>
<p>This means, for any value input, the output value must have the same type. For example. if we supplied the empty string, &#945; would be instantiated as the String type and the function would become:</p>
<pre>String &#8594; String</pre>
<p>Easy. Now let&#8217;s try to implement this function:</p>
<pre>id :: &#945; &#8594; &#945;
id a = ???</pre>
<p>So the problem is that we don&#8217;t know what the input type is inside the function. All we know is that it&#8217;s the polymorphic type &#945; &#8211; the same type as the value we have to return.</p>
<p>In a total language, the only possible &#945; we can return is the input. In a partial language, there are some ways to lie to the type-system and say we have an &#945;; we rely on programmers to not do that (just like we rely on programmers to write working tests).</p>
<p>This means there is only a <em>single possible implementation</em> for this function:</p>
<pre>id :: &#945; &#8594; &#945;
id a = a</pre>
<p>If this satisfies a type-checker, we&#8217;re done. No need to write a test &#8211; the compiler proved it correct! This is a concept called <a href="http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf">parametricity</a>.</p>
<p>Types can give you assurance that your code is correct. Tests only give you confidence that your code is correct. But the identity function is the most trivial of all functions, you could definitely TDD that in a few minutes and have high confidence of correctness.</p>
<p>Let&#8217;s do something more complicated but also something used everyday in functional programming: the map function.</p>
<p>The map function works for anything that is a Functor. Lists and arrays are immediate examples. The interesting thing is that for any Functor, there&#8217;s only a single possible implementation of map. In fact, Haskell can automatically write a proven implementation with the -XDeriveFunctor flag:</p>
<pre>{-# LANGUAGE DeriveFunctor #-}
data TestResult a = Success a | Failure String | Skipped deriving Functor</pre>
<p>The above defines a type called TestResult which has three separate states. The Success state has a polymorphic value which can be mapped over. Haskell finds the polymorphic value and writes the map function for us!</p>
<p>Getting the compiler to automatically derive a proven implementation: better than TDD.</p>
<h2 dir="ltr">Non-testable Constraints</h2>
<p>You can&#8217;t encode runtime guarantees as tests. Daniel Spiewak gave me an awesome example: asymptotic complexity.</p>
<p>We need a sort function. Given a list/array of integers, we want to get a sorted list back. Here&#8217;s some test-driven tests for the function:</p>
<pre>prop_head_is_minimum xs = not (null xs) ==&gt; head (sort xs) == minimum xs

prop_last_is_maximum xs = not (null xs) ==&gt; last (sort xs) == maxmimum xs

prop_is_sorted xs = isSorted $ sort xs
    where isSorted []  = True
          isSorted [x] = True
          isSorted (x:y:xs) = x &lt; y &amp;&amp; isSorted (y:xs)</pre>
<p>The above uses property-based testing (via <a href="http://en.wikipedia.org/wiki/QuickCheck">QuickCheck</a>). The xs is a value that is automatically generated by the test framework by looking at the type signature &#8211; we don&#8217;t have to worry about boundary conditions or interesting cases; they will be generated. <a href="http://book.realworldhaskell.org/">Real World Haskell</a> has a <a href="http://book.realworldhaskell.org/read/testing-and-quality-assurance.html">great Chapter about property-based testing</a> for anyone new to the concept.</p>
<p>Anyway, let&#8217;s use the tests to write an implementation for sort:</p>
<pre>sort :: [Int] -&gt; [Int]
sort [] = []
sort xs = select xs []
    where select []  ys  = ys
          select xs' ys  = select (delete (maximum xs') xs') $ maximum xs':ys</pre>
<p>Awesome! TDD gave us code that works. We&#8217;re done, right?</p>
<p>Well, hopefully we almost always want the asymptotically optimal algorithm to solve our problem. You might have noticed that the above code is selection sort &#8211; an algorithm with best/worst/average time complexity of O(n^2).</p>
<p>Sadly, we can&#8217;t write a test or a type to satisfy our specification. We need to actually perform some (asymptotic) analysis to derive our code, instead of relying on test-driving an implementation!</p>
<p>TDD isn&#8217;t enough. We still need analysis.</p>
<h2 dir="ltr">Conclusion</h2>
<p>This post isn&#8217;t saying that TDD shouldn&#8217;t be used &#8211; it&#8217;s absolutely fine to do TDD. This post is just to point out that TDD doesn&#8217;t completely replace types nor analysis of code.</p>
<p>When you work in a typed language, you don&#8217;t have to write tests for some functions. They&#8217;re proven to be correct by the type checker. But you can&#8217;t rely exclusively on types or tests. You have to use analysis to ensure runtime guarantees are satisfied.</p>
<p>Analysis is <em>the</em> method for writing code that satisfies our constraints. Neither TDD nor types are the single answer.</p>
<p>TDD can be useful. Types can be useful. Analysis is necessary.</p>