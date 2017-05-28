## 10 Nim One Liners to Impress Your Friends

My friend sent me the 10 Swift One Liners (referenced at the bottom) which inspired me to try it out in Nim!

Like many Electrical Engineers, I cut my programming teeth on "low level" languages like standard ANSI C w/ GCC compiler, x86 assembly, Motorola HC11/HC12 assembly, and even a little GW/QBASIC! But fast forward a few years and projects start requiring GUIs, communication involving marshalling data into a specified format and HTTP requests, and project teams consisting of more than one person touching the code base. Eventually I saw the light in going "high level", letting someone else collect your memory garbage, and focusing on productivity over performance. Python 2.6ish has filled that gap in a myriad of ways including data wrangling, web scraping, and plenty of chart foo graphics for reports. But look around the 2016 language landscape and you might notice there are plenty of viable new options including all the Clojure/Scala/Groovy JVM stuff, Google's [Go](https://golang.org/), Mozilla's [Rust](https://www.rust-lang.org/), Apple's [Swift](https://developer.apple.com/swift/), and sitting out on the side making strides is the funky one about which this post refers: [Nim](http://nim-lang.org/).  (Yes there is other newish stuff out there possibly including Python 3.5+ etc, but this is flavor text not a thesis. ;))

So with JVM Functional Programming Renaissance languages, Go's killer coroutines and channels, Rust's super safe strict compiler, and Swift's sex appeal, why even try Nim?

## Why Nim?
1. It compiles down to C, so GCC lives on! [Richard Stallman - "clang vs free software"](https://gcc.gnu.org/ml/gcc/2014-01/msg00247.html)
2. Seems to be a good balance of productivity and performance.  Almost feels like a scripting language but compiles down for quick execution.
3. I like to root for the underdog!!!

## Caveat Emptor!
1. Nim 0.13 just came out a few days ago. The compiler has bugs and things still seem to be changing quickly.
2. The community is smaller so there are less good examples available out there for whatever you're trying to do.
3. Nim doesn't have the backing of an uber corporation giving you warm fuzzies and a sense of security that your code will be supported 10+ years from now.

## Enough ranting, on with the show!
Just a warning, I don't actually know `Nim`. These are the first bits of code I've scrapped together so I have no idea if it is "idiomatic Nim" (e.g. using the implicit `result` variable in proc's). Anyway, Good luck and have fun! I know I did!

### 1. Multiply each item in a list by 2

    import sequtils
    echo toSeq(1..10).mapIt(it * 2)

Compile with: `nim c --run one.nim`
### 2. Sum a list of numbers

    import sequtils
    echo toSeq(1..1000).foldl(a + b)

or

    import sequtils, math
    echo toSeq(1..1000).sum()

Compile with: `nim c --run two.nim`
### 3. Verify if exists in a string

    # return a boolean if a word in wordList exists in tweet
    import sequtils, strutils
    var wordList = @["nim", "nimlang", "nimrod", "nimble", "koch"]
    var tweet = "This is an example tweet talking about nim and nimble."

    echo wordList.mapIt(tweet.contains(it)).anyIt(it == true)

Compile with: `nim c --run three.nim`
### 4. Read in a file

    import sequtils
    # read entire file
    echo readFile("./data.txt")

    # read in a sequence of lines from ascii file splitting on newline
    echo toSeq(lines("./data.txt"))

Compile with: `nim c --run four.nim`
### 5. Happy Birthday to You!
Since I'm a Python kind of guy, let's use a List Comprehension.

    import future
    echo lc[("Happy Birthday " & (if x != 3: "to you" else: "dear Ubergarm")) | (x <- 1..4), string]

Compile with: `nim c --run five.nim`
### 6. Filter list of numbers

    import sequtils
    var vals = @[49, 58, 76, 82, 88, 90]
    var (passed, failed) = (filterIt(vals, it > 60), filterIt(vals, it <= 60))
    echo passed
    echo failed

Compile with: `nim c --run six.nim`
### 7. Fetch and parse an XML web service
Yeah yeah, not exactly xml, but relax, Nim has an xmlparser module too.

    import httpclient, htmlparser, streams
    var html = parseHtml(newStringStream(getContent("https://twitter.com/search?q=Nimlang")))

Compile with: `nim c -d:ssl --run seven.nim`.
### 8. Find minimum (or maximum) in a List

    import sequtils
    echo(@[14, 35, -7, 46, 98].foldl(min(a,b)))
    echo(@[14, 35, -7, 46, 98].foldl(max(a,b)))

or

    import sequtils
    echo(@[14, 35, -7, 46, 98].min())
    echo(@[14, 35, -7, 46, 98].max())

Compile with: `nim c --run eight.nim`
### 9. Parallel Processing
To be honest, I had some trouble with this one.  After being spoiled by the implementation simplicity of `goroutines` and the monkey-patch-black-magic of `gevent`, or even plain old pthreads with locks on global structs, its hard to like anything else for me... Even Python 3.5+ asyncio doesn't feel right, meh.

Yeah yeah Concurrency vs Parallelism blah blah...

> Nim's memory model for threads is quite different from other common programming languages (C, Pascal): Each thread has its own (garbage collected) heap and sharing of memory is restricted. [Nim Manual](http://nim-lang.org/docs/threads.html)

Nim has a few modules for these kinds of things.
1. `threads` used implicitly with --threads:on. like pthreads
2. `threadpool` with `spawn` and `sync()` and `FlowVar`
3. `threadpool` with `parallel:` with `spawn`
4. `asyncdispatch` with futures and callbacks `async` and `await`

> Once we are getting closer to a final form, we could start a discussion about parallel operations like pMap, pFilter. Lisp has a parallel foldlSeq called scan, works well if the function is costly to calculate and associative. Just wanted to highlight that this won't be a simple topic.
[Nim Forum](http://forum.nim-lang.org/t/1333/2)

So I tried the `threadpool` `parallel` block and `spawn` (even though it requires the `--experimental` compiler flag) because well, the manual says it is the preferred way:

> The parallel statement is the preferred mechanism to introduce parallelism in a Nim program. [Nim Manual](http://nim-lang.org/docs/manual.html#parallel-spawn-parallel-statement)

But after an hour of the compiler complaining `Error: cannot prove: i <= len(result) + -1 (bounds check)` no matter what I tried, even after turning off bounds checking with `--boundChecks:off`, I decided to punt.

The solution I settled on uses a `FlowVar[T]` from which the result can be accessed eventually using the `^` operator.

To get a "one liner" I had to take a crack at a `pMap` procedure based on standard `map`. ~~Even this method requires turning off threadAnalysis during compile or you'll get `Error: 'spawn' takes a GC safe call expression`.~~ *UPDATE: Use thread pragma.* I also had to use extra intermediate variable assignments to get it to compile... Dunno...

~~Finally, the compiler keeps telling me `Warning: use Thread instead; TThread is deprecated [Deprecated]`.~~ *UPDATE: This should be fixed in [PR 3106](https://github.com/nim-lang/Nim/pull/3106)*

The good news is:
> [Nim 0.14] will focus on improvements to the GC and concurrency. We will in particular be looking at ways to add multi-core support to async await [Nim Release Notes](http://nim-lang.org/news.html#Z2016-01-18-version-0-13-0-released)

The code:

    import sequtils, threadpool, os, math

    proc pMap*[T, S](data: openArray[T], op: proc (x: T): S {.closure,thread.}): seq[S]{.inline.} =
        newSeq(result, data.len)
        var vals = newSeq[FlowVar[T]](data.len)

        for i in 0..data.high:
            vals[i] = spawn op(data[i])
        sync()

        for i in 0..data.high:
            var res = ^vals[i]
            result[i] = res

    proc doWork(x: int): int =
        # sleep and print just to show it's actually parallel
        os.sleep(random(100))
        echo x
        return 2 * x

    # it takes many lines to make a this one liner
    echo toSeq(1..10).pMap(doWork)

Compile with: `nim c --threads:on --run nine.nim`
### 10. Sieve of Eratosthenes

    import sequtils, math, future
    const n = 30
    echo toSeq(2..n).filterIt(it notin lc[toSeq(countup(x * x, n, x)) | (x <- toSeq(countup(2, int(sqrt(float(n))) + 1)) ), int])

It took some research to get this one, and a special thanks to this python [implementation](http://stackoverflow.com/questions/6687296/python-sieve-of-eratosthenes-compact-python) on stackoverflow.  Here is a more verbose example:

    import sequtils, math
    proc sieve(n: int): seq[int] =
        var vals = toSeq(1..n)
        for i in 2..int(sqrt(float(n)))+1:
            for j in countup(i*2,n, i):
                vals.keepItIf(it != j)
        return vals
    echo sieve(30)

Compile with: `nim c --run ten.nim`
## Conclusion
Well there you have it, my take at 10 Nim One Liners. Though if you count the import none of these are actually one line... er...

The most interesting point for me is how compact and "script like" the language feels. I find it to be quite readable, coming from a Python background, and really like the flexible [Method Call Syntax](http://nim-lang.org/docs/manual.html#procedures-method-call-syntax). You might want to use the [Style Guide](https://github.com/nim-lang/Nim/wiki/Style-Guide-for-Nim-Code) for consistency.

The language doesn't feel super tight yet in terms of function names and consistency IMO, but it isn't even 1.0 yet so oh well.  Plus I don't even know the language, so who am I to say... haha...

## Final Thought
I like Nim, but definitely want to try it out on a more "real" project and put it through its paces and hope 1.0 comes soon.

Also thanks to `def-` on Freenode channel #nim for feedback!

## References
* [10 Swift One Liners To Impress Your Friends](https://www.uraimo.com/2016/01/06/10-Swift-One-Liners-To-Impress-Your-Friends/)
* [10 Scala One Liners to Impress Your Friends](https://mkaz.github.io/2011/05/31/10-scala-one-liners-to-impress-your-friends/)
