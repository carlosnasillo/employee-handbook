# Scala Best Practices

<img src="https://raw.githubusercontent.com/monifu/scala-best-practices/master/assets/scala-logo-256.png"  align="right" width="100" height="100" />

Guidelines helps us write consistently standardized code. Best practices on the other hand, help us make sure our colleagues don't _facepalm_ while reading our code :D 



## 1. Hygienic Rules

These are general purpose hygienic rules that transcend the language
or platform rules. Programming language is a form of communication,
targeting computer systems, but also your colleagues and your future
self, so respecting these rules is just like washing your hands after
going to the bathroom.

### 1.1. SHOULD enforce a reasonable line length

There's a whole science on typography which says that people lose
their focus when the line of text is too wide, a long line makes it
difficult to gauge where the line starts or ends and it makes it
difficult to continue on the next line below it, as your eyes have to
move a lot from the right to the left. It also makes it difficult to
scan for important details.

In typography, the optimal line length is considered to be somewhere
between 50 and 70 chars.

In programming, we've got indentation, so it's not feasible to impose
a 60 chars length for lines. 80 chars is usually acceptable, but not
in Scala, because in Scala we use a lot of closures and if you want
long and descriptive names for functions and classes, well, 80 chars
is way too short.

120 chars, as IntelliJ IDEA is configured by default, may be too wide
on the other hand.  Yes, I know that we've got 16:9 wide monitors, but
this doesn't help readability and with shorter lines we can put these
wide monitors to good use when doing side-by-side diffs. And with long
lines it takes effort to notice important details that happen at the
end of those lines.

So as a balance:

- strive for 80 chars as the soft limit and if it gets ugly,
- then 100 chars is enough, except for ...
- function signatures, which can get really ugly if limited

On the other hand, anything that goes beyond 120 chars is an abomination.

### 1.2. MUST NOT rely on a SBT or IDE plugin to do the formatting for you

IDEs and SBT plugins can be of great help, however if you're thinking
about using one to automatically format your code, beware.

You won’t find a plugin that is able to infer the developer’s intent,
since that requires a human-like understanding of the code and would be 
near impossible to make. The purpose of proper indentation and formatting 
isn't to follow some rigid rules set upon you in a cargo-cult way, but to
make the code more logical, more readable, more
approachable. Indentation is actually an art form, which is not awful
since all you need is a nose for awful code and the urge of fixing
it. And it is in the developer's job description to make sure that his
code doesn't stink.

So automated means are fine, BUT BE CAREFUL to not ruin other people's
carefully formatted code, otherwise I'll slap you in prose.

Lets think about what I said - if the line is too long, how is a
plugin supposed to break it? Lets talk about this line (real code):

```scala
    val dp = new DispatchPlan(new Set(filteredAssets), start = startDate, end = endDate, product, scheduleMap, availabilityMap, Set(activationIntervals.get), contractRepository, priceRepository)
```

In most cases, a plugin will just do truncation and I've seen a lot of these in practice:

```scala
    val dp = new DispatchPlan(Set(filteredAssets), start =
      startDate, end = endDate, product, scheduleMap, availabilityMap,
      Set(activationIntervals), contractRepository, priceRepository)
```

Now that's not readable, is it? I mean, seriously, that looks like
barf. And that's exactly the kind of output I see coming from people
relying on plugins to work. We could also have this version:

```scala
    val dp = new DispatchPlan(
      Set(filteredAssets),
      startDate,
      endDate,
      product,
      scheduleMap,
      availabilityMap,
      Set(activationIntervals),
      contractRepository,
      priceRepository
    )
```

Looks much better. But truth is, this isn't so good in other
instances. Like say we've got a line that we want to break:

```scala
   val result = service.something(param1, param2, param3, param4).map(transform)
```

Now placing those parameters on their own line is awful, no matter how you deal with it:

```scala
    // awful because that transform call is not visible
    val result = service.something(
      param1,
      param2,
      param3,
      param4).map(transform)

    // awful because it breaks the logical flow
    val result = service.something(
      param1,
      param2,
      param3,
      param4
    ).map(transform)
```

This would be much better:

```scala
    val result = service
      .something(param1, param2, param3, param4)
      .map(transform)
```

Now that's better, isn't it? Of course, sometimes that call is so long
that this doesn't cut it. So you need to resort to a temporary value
of some sort, e.g...

```scala
    val result = {
      val instance =
        object.something(
          myAwesomeParam1,
          otherParam2,
          someSeriousParam3,
          anEvenMoreSoParam4,
          lonelyParam5,
          catchSomeFn6,
          startDate7
        )

      for (x <- instance) yield
        transform(x)
    }
```

Of course, sometimes if the code stinks so badly, you need to get into
refactoring - as in, maybe too many parameters are too much for a
function ;-)

And we are talking strictly about line lengths - once we get into
other issues, things get even more complicated. So really, you won't
find a plugin that does this analysis and that can make the right
decision for you.

### 1.3. SHOULD break long functions

Ideally functions should only be a couple of lines long. If the lines
get too big, then we need to break them into smaller functions and
give them a name.

Note that in Scala we don't necessarily have to make such intermediate
functions available in other scopes, the purpose here is to primarily
aid readability, so in Scala we can do inner-functions to break logic
into pieces.

### 1.4. MUST NOT introduce spelling errors in names and comments

Spelling errors are freakishly annoying, interrupting a reader's flow.
Use a spell-checker. Intelligent IDEs have built-in
spell-checkers. Note the underlined spelling warnings and fix them.

### 1.5. Names MUST be meaningful

*"There are only two hard things in Computer Science: cache
invalidation and naming things."* -- Phil Karlton

We've got three guidelines here:

1. give descriptive names, but don't go overboard, four words is a
   little too much already
2. you can be terse in naming if the type / purpose can be easily
   inferred from the immediate context, or if there's already an
   established convention
3. if going the descriptive route, don't do bullshit words that are
   meaningless

For example this is acceptable:

```scala
for (p <- people) yield
  transformed(p)
```

We can see that `p` is a person from the immediate context, so a short
one letter name is OK. This is also acceptable because `i` is an
established convention to use as an index:

```scala
for (i <- 0 until limit) yield ???
```

This is in general not acceptable, because usually with tuples the
naming of the collection doesn't reflect well what's contained (if you
haven't given those elements a name, then as a consequence the
collection itself is going to have a bad name):

```
someCollection.map(_._2)
```

Implicit parameters on the other hand are OK with short names, because
being passed implicitly, we don't care about them unless they are
missing:

```scala
def query(id: Long)(implicit ec: ExecutionContext, c: WSClient): Future[Response]
```

This is not acceptable because the name is utterly meaningless, even
if there's a clear attempt at being descriptive:

```scala
def processItems(people: Seq[Person]) = ???
```

It's not acceptable because the naming of this function indicates a
side-effect (`process` is a verb indicating a command), yet it doesn't
describe what we are doing with those `people`. The `Items` suffix is
meaningless, because we might have said `processThingy`,
`processRows`, `processStuff` and it would still say exactly the same
thing - absolutely nothing. It also increases visual clutter, as more
words is more text to read and meaningless words are just noise.

Properly chosen descriptive names - good. Bullshit names - bad.





## 2. Language Rules



### 2.1. MUST NOT use "return"

The `return` statement from Java signals a side-effect - unwind the
stack and give this value to the caller. In a language in which the
emphasis is on side-effect-full programming, this makes sense. However
Scala is an expression oriented language in which the emphasis is on
controlling/limiting side-effects and `return` is not idiomatic.

To make matters worse, `return` probably doesn't behave as you think
it does. For example in a Play controller, try doing this:

```scala
def action = Action { request =>
  if (someInvalidationOf(request))
    return BadRequest("bad")

 Ok("all ok")
}
```

In Scala, a `return` statement inside a nested anonymous function is
implemented by throwing and catching a `NonLocalReturnException`. It
says so in the
[Scala Language Specification, section 6.20](http://www.scala-lang.org/docu/files/ScalaReference.pdf).

Besides, `return` is anti structural programming, as functions can be
described with multiple exit points and if you need `return`, like in
those gigantic methods with lots of if/else branches, the presence of
a `return` is a clear signal that the code stinks, a magnet for future
bugs and is thus in urgent need of refactoring.

### 2.2. SHOULD use immutable data-structures

Immutable data-structures are facts that can be compared and reasoned
about. Mutable things are error-prone buckets. You should never use a
mutable data-structure unless you're able to defend it and there are
really, really few places in which a mutable data-structure is
defensible.

Lets exemplify:

```scala
trait Producer[T] {
 def fetchList: List[T]
}

// consumer side
someProducer.fetchList
```

Question - if the `List` returned above is mutable, what does that say
about the `Producer` interface?

Here are some problems:

1. if this List is produced on another thread than the consumer, one
   can have both visibility and atomicity problems - you can't know
   whether that will happen, unless you take a look at the Producer's
   implementation.

2. even if this List is effectively immutable (i.e. still mutable, but
   no longer modified after being signaled to the Consumer), you don't
   know if it will be signaled to other Consumers that may modify it
   by themselves, so you can't reason about what you can do with it.

3. even if it is described that access to this List must be
   synchronized, problem is - on which lock are you going to
   synchronize?  Are you sure you'll get the locking order right?
   Locks are not composable.

So there you have it - a public API exposing a mutable data-structure
is an abomination of nature, leading to problems that can be worse
than what happens when doing manual memory management.

### 2.3. SHOULD NOT update a `var` using loops or conditions

It's a mistake that most Java developers do when they come to Scala. Example:

```scala
var sum = 0
for (elem <- elements) {
  sum += elem.value
}
```

Avoid doing this, prefer the available operators instead, like `foldLeft`:

```scala
val sum = elements.foldLeft(0)((acc, e) => acc + e.value)
```

Or even better, know thy standard library and always prefer to
use the built-in functions - the more expressive you go, the less bugs
you'll have:

```scala
val sum = elements.map(_.value).sum
```

In the same spirit, you shouldn't update a partial result with a condition.
Example:


```scala
def compute(x) = {
  var result = resultFrom(x)

  if(needToAddTwo) {
    result += 2
  }
  else {
  	result += 1
  }

  result
}
```

Prefer expressions and immutability. The code will be more readable
and less error-prone, since it makes the branches more explicit and that's
a good thing:

```scala
def computeResult(x) = {
  val r = resultFrom(x)
  if (needToAddTwo)
    r + 2
  else
  	r + 1
}
```

And you know, as soon as the branches get too complex, just as was said in the
discussion on `return`, that's a sign that the *code smells* and is in need
of refactoring, which is a good thing.

### 2.4. SHOULD NOT define useless traits

There was this Java Best Practice that said "*program to an interface,
not to an implementation*", a best practice that has been cargo-cult-ed
to the point that people started defining completely useless
interfaces in their code. Generally speaking, that rule is healthy,
but it refers to the general engineering need of hiding implementation
details especially details of modifying state (encapsulation) and not
to slap interface declarations that often leak implementation details
anyway.

Defining traits is also a burden for readers of that code, because it
signals a need for polymorphism. Example:

```scala
trait PersonLike {
  def name: String
  def age: Int
}

case class Person(name: String, age: Int)
  extends PersonLike
```

Readers of this code might come to the conclusion that there are
instances in which overriding `PersonLike` is desirable. That couldn't
be further from the truth - `Person` is perfectly described by its
case class as a data-structure without behavior. In other words it
describes the shape of your data and if you need to override this
shape for some unknown reason, then this trait is badly defined
because it imposes the shape of your data and that's about the only
thing you can override. You can always come up with traits later, if
you're in need of polymorphism, after your needs evolve.

And if you're thinking that you may need to override the source of
this (as in to fetch the person's `name` from the DB on first access),
OMG don't do that!

Note that I'm not talking about algebraic data structures (i.e. sealed
traits that are signaling a closed set of choices - like `Option`).

Even in those cases in which you think the issue is clear-cut, it may
not be. Lets take this example:

```scala
trait DBService {
  def getAssets: Future[Seq[(AssetConfig, AssetPersistedState)]]

  def persistFlexValue(flex: FlexValue): Future[Unit]
}
```

This snippet is taken from real-world code - we've got a `DBService`
that handles either queries or persistence in a database. Those two
methods are actually unrelated, so if you only need to fetch the
assets, why depend on things you don't need in components that require
DB interaction?

Lately my code looks a lot like this:

```scala
final class AssetsObservable
    (f: => Future[Seq[(AssetConfig, AssetPersistedState)]])
  extends Observable[AssetConfigEvent] {

  // ...
}

object AssetsObservable {
  // constructor
  def apply(db: DBService) = new AssetsObservable(db.getAssets)
}
```

See, I do not need to mock an entire `DBService` in order to test the
above.

### 2.5. MUST NOT use "var" inside a case class

Case classes are syntactic sugar for defining classes in which - all
constructor arguments are public and immutable and thus part of the
value's identity, have structural equality, a corresponding hashCode
implementation and apply/unapply auto-generated functions provided by
the compiler.

By doing this:

```scala
case class Sample(str: String, var number: Int)
```

You just broke its equality and hashCode operation. Now try using it
as a key in a map.

As a general rule of thumb, structural equality only works for
immutable things, because the equality operation must be stable (and
not change according to the object's history). Case classes are for
strictly immutable things. If you need to mutate stuff, don't use case
classes.

In the approximate words of Fogus in "The Joy of Clojure" or Baker in
his paper from 1993: if any two mutable objects resolve as being equal
now, then there’s no guarantee that they will a moment from now. And
if two objects aren’t equal forever, then they’re technically never
equal ;-)

### 2.6. SHOULD NOT declare abstract "val" or "var" or "lazy val" members

It's a bad practice to declare abstract vals or vars or lazy vals in
abstract classes or traits. Do not do this:

```scala
trait Foo {
 val value: String
}
```

Instead, prefer to always declare abstract things as `def`:

```scala
trait Foo {
 def value: String
}

// can then be overridden as anything, including val
class Bar(val value: String) extends Foo
```

The reason has to do with the imposed restriction - a `val` can only
be overridden with a `val`, a `var` can only be overridden with a
`var`, etc. The only way to allow freedom to choose on inheritance is
to use `def` for abstract members. And this freedom is important,
because a `val` for example restricts the way this value can get
initialized - only at construction time. Take this example:

```scala
trait Foo { val value: String }

trait Bar extends Foo { val uppercase = value.toUpperCase }

trait MyValue extends Foo { val value = "hello" }

// this triggers a NullPointerException
new Bar with MyValue

// this works
new MyValue with Bar
```

In the sample above, a gotcha is exemplified related to the order in
which the JVM + Scala executes things at construction time. It's a
catch 22 between Scala's ideals and the JVM's limits. Reversing the
order on inheritance works, but this is a fragile / error-prone
solution. And because we have the `value` defined as a `val`, we
cannot fix the above example by overriding `value` as a `lazy val`, an
otherwise decent way of fixing such a sample.

There's no good reason to impose on inheritors the way a value should
get initialized. `def` is generic so use `def` instead.

### 2.7. MUST NOT Throw Exceptions for Validations of User Input or Flow Control

Two reasons:

1. it goes against the principles of structured programming as a
   routine ends up having multiple exit points and are thus harder to
   reason about - with the stack unwinding happening being an awful and
   often unpredictable side-effect
2. exceptions aren't documented in the function's signature - Java
   tried fixing this with the checked exceptions concept, which in
   practice was awful as people simply ignored them

Exceptions are useful for only one thing - signaling unexpected errors
(bugs) up the stack, such that a supervisor can catch those errors and
decide to do things, like log the errors, send notifications,
restarting the guilty component, etc...

As an appeal to authority, it's reasonable to reference
[Functional Programming with Scala](http://www.manning.com/bjarnason/),
chapter 4.

### 2.8. MUST NOT catch Throwable when catching Exceptions

Never, never, never do this:

```scala
try {
 something()
} catch {
 case ex: Throwable =>
   blaBla()
}
```

Never catch `Throwable` because we could be talking about extremely
fatal exceptions that should never be caught and that should crash the
process. For example if the JVM throws an out of memory error, even if
you re-throw that exception in that catch clause, it may be too late -
given that the process is out of memory, the garbage collector
probably took over and froze everything, with the process ending in a
zombie unrecoverable state. Which means that an external supervisor
(like Upstart) will not get an opportunity to restart it.

Instead do this:

```scala
import scala.util.control.NonFatal

try {
 something()
} catch {
 case NonFatal(ex) =>
   blaBla()
}
```

### 2.9. MUST NOT use "null"

You must avoid using `null`. Prefer Scala's `Option[T]` instead. Null
values are error prone, because the compiler cannot protect
you. Nullable values that happen in function definitions are not
documented in those definitions. So avoid doing this:

```scala
def hello(name: String) =
  if (name != null)
    println(s"Hello, $name")
  else
    println("Hello, anonymous")
```

As a first step, you could be doing this:

```scala
def hello(name: Option[String]) = {
  val n = name.getOrElse("anonymous")
  println(s"Hello, $n")
}
```

The point of using `Option[T]` is that the compiler forces you to deal
with it, one way or another:

1. you either have to deal with it right away (e.g. by providing a
   default, throwing an exception, etc..)
2. or you can propagate the resulting `Option` up the call stack

Also remember that `Option` is just like a collection of 0 or 1
elements, so you can use foreach, which is totally idiomatic:

```scala
val name: Option[String] = ???

for (n <- name) {
  // executes only when the name is defined
  println(n)
}
```

Combining multiple options is also easy:

```scala
val name: Option[String] = ???
val age: Option[Int] = ???

for (n <- name; a <- age)
  println(s"Name: $n, age: $a")
```

And since `Option` is seen as an `Iterable` too, you can use `flatMap`
on collections to get rid of `None` values:

```scala
val list = Seq(1,2,3,4,5,6)

list.flatMap(x => Some(x).filter(_ % 2 == 0))
// => 2,4,6
```

### 2.10. MUST NOT use `Option.get`

You might be tempted to do this:

```scala
val someValue: Option[Double] = ???

// ....
val result = someValue.get + 1
```

Don't ever do that, since your trading a `NullPointerException` for a
`NoSuchElementException` and that defeats the purpose of using
`Option` in the first place.

Alternatives:

1. using `Option.getOrElse`
2. using `Option.fold`
3. using pattern matching and dealing with the `None` branch explicitly
4. not taking the value out of its optional context

As an example for (4), not taking the value out of its context means
this:

```scala
val result = someValue.map(_ + 1)
```

### 2.11. MUST NOT use Java's Date or Calendar, instead use Joda-Time or JSR-310

Java's Date and Calendar classes from the standard library are awful
because:

1. resulting objects are mutable, which doesn't make sense for
   expressing a date, which should be a value (how would you feel if
   you had to work with StringBuffer everywhere you have Strings?)
2. months numbering is zero based
3. Date in particular does not keep timezone info, so Date values are completely useless
4. it doesn't make a difference between GMT and UTC
5. years are expressed as 2 digits instead of 4

Always use [Joda-Time](http://www.joda.org/joda-time/) - of if you can
afford to switch to Java 8, there's a shinny new
[JSR-310](http://www.threeten.org/) that's based on Joda-Time and that
will be the new standard once people adopt Java 8.

### 2.12. SHOULD NOT use Any or AnyRef or isInstanceOf / asInstanceOf

Avoid using Any or AnyRef or explicit casting, unless you've got a
really good reason for it. Scala is a language that derives value from
its expressive type system, usage of Any or of typecasting represents
a hole in this expressive type system and the compiler doesn't know
how to help you there. In general, something like this is bad:

```scala
val json: Any = ???

if (json.isInstanceOf[String])
  doSomethingWithString(json.asInstanceOf[String])
else if (json.isInstanceOf[Map])
  doSomethingWithMap(json.asInstanceOf[Map])
else
  ???
```

Often we are using Any when doing deserialization. Instead of working
with Any, think about the generic type you want and the set of
sub-types you need, and come up with an Algebraic Data-Type:

```scala
sealed trait JsValue

case class JsNumber(v: Double) extends JsValue
case class JsBool(v: Boolean) extends JsValue
case class JsString(v: String) extends JsValue
case class JsObject(map: Map[String, JsValue]) extends JsValue
case class JsArray(list: Seq[JsValue]) extends JsValue
case object JsNull extends JsValue
```

Now, instead of operating on Any, we can do pattern matching on
JsValue and the compiler can help us here on missing branches, since
the choice is finite. This will trigger a warning on missing branches:

```scala
val json: JsValue = ???
json match {
  case JsString(v) => doSomethingWithString(v)
  case JsNumber(v) => doSomethingWithNumber(v)
  // ...
}
```

### 2.13. MUST serialize dates as either Unix timestamp, or as ISO 8601

Unix timestamps, provided that we are talking about the number of
seconds or milliseconds since 1970-01-01 00:00:00 UTC (with emphasis
on UTC) are a decent cross-platform serialization format. It does have
the disadvantage that it has limits in what it can express. ISO-8601
is a decent serialization format supported by most libraries.

Avoid anything else and also when storing dates without a timezone
attached (like in MySQL), always express that info in UTC.

### 2.14. MUST NOT use magic values

Although not uncommon in other languages to use "magic" (special)
values like `-1` to signal particular outcomes, in Scala there are a
range of types to make intent clear. `Option`, `Either`, `Try` are
examples. Also, in case you want to express more than a boolean
success or failure, you can always come up with an algebraic data
type.

Don't do this:

```scala
val index = list.find(someTest).getOrElse(-1)
```

### 2.15. SHOULD NOT use "var" as shared state

Avoid using "var" at least when speaking about shared mutable
state. Because if you do have shared state expressed as vars, you'd
better synchronize it and it gets ugly fast. Much better is to avoid
it. In case you really need mutable shared state, use an atomic
reference and store immutable things in it. Also checkout
[Scala-STM](https://nbronson.github.io/scala-stm/).

So instead of something like this:

```scala
class Something {
  private var cache = Map.empty[String, String]
}
```

If you can't really avoid that variable, prefer doing this:

```scala
import java.util.concurrent.atomic._

class Something {
  private val cache =
    new AtomicReference(Map.empty[String, String])
}
```

Yes, it introduces overhead due to the synchronization required, which
in the case of an atomic reference means spin loops. But it will save
you from lots and lots of headaches later. And it's best to avoid
mutation entirely.

### 2.16. Public functions SHOULD have an explicit return type

Prefer this:

```scala
def someFunction(param1: T1, param2: T2): Result = {
  ???
}
```

To this:

```scala
def someFunction(param1: T1, param2: T2) = {
  ???
}
```

Yeah, type inference on the result of a function is great and all, but
for public methods:

1. it's not OK to rely on an IDE or to inspect the implementation in
   order to see the returned type
2. Scala currently infers the most specialized type possible, because
   in Scala the return type on functions is covariant, so you might
   actually get a really ugly type back

For example, what is the returned type of this function:

```scala
def sayHelloRunnable(name: String) = new Runnable {
  def sayIt() = println(s"Hello, $name")
  def run() = sayIt()
}
```

Do you think it's `Runnable`?
Wrong, it's `Runnable{def sayIt(): Unit}`.

As a side-effect, this also increases compilation times, as whenever
`sayHelloRunnable` changes implementation, it also changes the
signature so everything that depends on it must be recompiled.

### 2.17. SHOULD NOT define case classes nested in other classes

It is tempting, but you should almost never define nested case classes
inside another object/class because it messes with Java's
serialization. The reason is that when you serialize a case class it
closes over the "this" pointer and serializes the whole object, which
if you are putting in your App object means for every instance of a
case class you serialize the whole world.

And the thing with case classes specifically is that:

1. one expects a case class to be immutable (a value, a fact) and hence
2. one expects a case class to be easily serializable

Prefer flat hierachies.





## 3. Application Architecture



### 3.1. SHOULD NOT use the Cake Pattern

The Cake Pattern is a very
[good idea in theory](https://www.youtube.com/watch?v=yLbdw06tKPQ) -
using traits as modules that can be composed, giving you the ability
to override `import`, with compile-time dependency injection as a
side-effect.

In practice all the Cake implementations I've seen have been awful,
new projects should steer away and existing projects should be
migrated off Cake.

People are not implementing Cake correctly, being a poorly understood
design pattern. I haven't seen Cake implementations in which the
traits are designed to be abstract modules, or that pay proper
attention to life-cycle issues. What happens in practice is sloppiness,
with the result being a big hairball. It's awesome that Scala allows
you to do things like the Cake pattern, highlighting the real power of
OOP, but just because you can doesn't mean you should, because if the
purpose is doing dependency injection and decoupling between various
components, you'll fail hard and impose that maintenance burden on
your colleagues.

In fact, one should strive to not do dependency injection at all, or
to do it only at the edges (like Play's controllers). Because if a
component depends on too many things, that's *code smell*. If a
component depends on hard to initialize arguments, that's also *code
smell*. Don't hide painful things under the rug, fix it instead.

### 3.2. MUST NOT put things in Play's Global

I'm seeing this over and over again.

Folks,
[Play's Global](https://www.playframework.com/documentation/2.3.x/ScalaGlobal)
object is not a bucket in which you can shove your orphaned pieces of
code. Its purpose is to hook into Play's configuration and life-cycle,
nothing more.

Come up with your own freaking namespace for your utilities.

### 3.3. SHOULD NOT apply optimizations without profiling

Profiling is a prerequisite for doing optimizations. Never work on
optimizations, unless through profiling you discover the actual
bottlenecks.

This is because our intuition about how the system behaves often fails
us and multiple effects could happen by applying optimizations without
having hard numbers:

- you could complicate the code or the architecture, thus making it
  harder to apply later optimizations globally
- your work could be in vain or it could actually lead to more
  performance degradation

Multiple strategies available and you should preferably do all of
them:

- a good profiler can tell you about bottlenecks that aren't obvious,
  my favorite being YourKit Profiler, but Oracle's VisualVM is free
  and often good enough
- collect metrics from the running production systems, by means of a
  library such as
  [Dropwizard Metrics](https://dropwizard.github.io/metrics/3.1.0/)
  and push them in something like
  [Graphite](http://graphite.wikidot.com/), a strategy that can lead
  you in the right direction
- compare solutions by writing benchmarking code, but note that
  benchmarking is not easy and you should at least use a library like
  [Google Caliper](https://code.google.com/p/caliper/)

Overall - measure, don't guess.

### 3.4. SHOULD be mindful of the garbage collector

Don't over allocate resources, unless you need to. We want to avoid
micro optimizations, but always be mindful about the effects
allocations can have on your system.

In the
[words of Martin Thomson](http://www.infoq.com/presentations/top-10-performance-myths),
if you stress the garbage collector, you'll increase the latency on
stop-the-world freezes and the number of such occurrences, with the
garbage collector acting like a GIL and thus limiting performance and
vertical scalability.

Example:

```scala
query.filter(_.someField.inSet(Set(name)))
```

This is a sample that occurred in our project due to a problem with
Slick's API. So instead of a `===` test, the developer chose to do an
`inSet` operation with a sequence of 1 element. This allocation of a
collection of 1 element happens on every method call. Now that's not
good, what can be avoided should be avoided.

Another example:

```scala
someCollection
 .filter(Set(a,b,c).contains)
 .map(_.name)
```

First of all, this creates a Set every single time, on each element of
our collection. Second of all, filter and map can be compressed in one
operation, otherwise we end up with more garbage and more time spent
building the final collection:

```scala
val isIDValid = Set(a,b,c)

someCollection.collect {
  case x if isIDValid(x) => x.name
}
```

A generic example that often pops up, exemplifying useless traversals
and operators that could be compressed:

```scala
collection
  .filter(bySomething)
  .map(toSomethingElse)
  .filter(again)
  .headOption
```

Also, take notice of your requirements and use the data-structure
suitable for your use-case. You want to build a stack? That's a
`List`. You want to index a list? That's a `Vector`. You want to
append to the end of a list? That's again a `Vector`. You want to push
to the front and pull from the back? That's a `Queue`. You have a set
of things and want to check for membership? That's a `Set`. You have a
list of things that you want to keep ordered? That's a
`SortedSet`. This isn't rocket science, just computer science 101.

We are not talking about extreme micro optimizations here, we aren't
even talking about something that's Scala, or FP, or JVM specific
here, but be mindful of what you're doing and try to not do
unnecessary allocations, as it's much harder fixing it later.

BTW, there is an obvious solution for keeping expressiveness while
doing filtering and mapping - lazy collections, which in Scala means
[Stream](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Stream)
if you need memoization or
[Iterable](http://docs.oracle.com/javase/7/docs/api/java/lang/Iterable.html)
if you don't need memoization.

Also, make sure to read the
[Rule 3.3](#33-should-not-apply-optimizations-without-profiling) on
profiling.





## 4. Concurrency and Parallelism



### 4.1. SHOULD avoid concurrency like the plague it is

Avoid having to deal with concurrency as much as possible. People good
at concurrency avoid it like the plague it is.

**WARNING:** concurrency issues happen not only when speaking about
shared memory and threads, but also between processes when contention
on any kind of resource (like a database) is involved.

Example - when a job is scheduled to execute every minute by using
cron.d in Linux and that job fetches and updates items from a queue
persisted in MySQL, that job can take longer than 1 minute to execute
and thus you can end up with 2 or 3 processes executing at the same
time and contending on the same MySQL table.

### 4.2. SHOULD use appropriate abstractions only where suitable - Future, Actors, Rx

Learn about the abstractions available and choose between them
depending on the task at hand. There is no silver bullet that can be
generally applied. The more high-level the abstraction, the less scope
it has in solving issues. For example many developers in the Scala
community are overusing Akka Actors - which are great, but not when
misapplied. Like don't use an Akka Actor when a `Future` would do.

Scala's Futures and Promises are good because:

- they are inherently parallelizable by eliminating concurrency
  concerns
- fairly efficient, because when submitting a task to the implicit
  `ExecutionContext`, this execution context efficiently multiplexes
  between few threads by default (the number of threads in the
  thread-pool is often directly proportional to the number of CPU cores
  you have)
- the model is inherently simple and easy to use

Futures and Promises are bad because they signal only one value from
the producer to the consumer and that's it - if you need a stream or
bi-directional communications, a Future might not be the best
abstraction.

Akka Actors are good because:

- they make bidirectional communications over asynchronous boundaries
  easy - for example WebSocket is a prime candidate for actors
- with Akka's Actors you can easily model state machines (see
  `context.become`)
- the processing of messages has a strong guarantee of
  non-concurrency - messages are processed one by one, so there's no
  need to worry about concurrency issues while in the context of an
  actor
- instead of implementing a half-assed in-memory queue for processing
  of things, you could just use an actor, since queuing of messages and
  acting on those messages is what they do

Akka's Actors are bad because:

- they are fairly low-level for many tasks
- it's extremely easy to model actors that keep a lot of state, ending
  up with a system that can't be horizontally scaled
- because of the bidirectional communications capability, it's
  extremely easy to end up with data flows that are so complex as to be
  unmanageable
- the model in general is actor A sending a message to actor B - but
  if you need to model a stream of events, this tight coupling between A
  and B is not acceptable

Rx and Iteratees - see Play's
[Iteratees](https://www.playframework.com/documentation/2.3.x/Iteratees)
/ [RxJava](https://github.com/ReactiveX/RxJava) /
[Reactive Streams](http://www.reactive-streams.org/) /
[Monifu](https://github.com/alexandru/monifu) are good because:

- they model unidirectional communications between a producer and
  consumers - with an `Observable`/`Enumerator` being a channel that doesn't
  care what listeners it has (think about a stream of data like a river
  of information - the river doesn't care about who drinks from it)
- depending on implementation, they address back-pressure concerns by
  default, with consumers signaling demand to the producer and the
  producer insuring that it doesn't send more data than the consumer can
  chew on
- just as in the case of Future, because of the limitations, the model
  is simple to use - and the exposed operators / combinators are awesome

Rx / Iteratees are bad because:

- they are only about unidirectional communications, it gets
  complicated if you want bidirectional communications, so choose
  actors instead

- because of the strong contract they come with (i.e. no concurrent
  notifications for example), implementing new operators and data-source
  can be problematic, but usage on the consumer side is kept simple
  because of this

So there you have it. Learn and pick wisely - don't apply abstractions
like some sort of special sauce without thinking about it

### 4.3. SHOULD NOT wrap purely CPU-bound operations in Futures

This is in general an anti-pattern:

```scala
def add(x: Int, y: Int) = Future { x + y }
```

If you don't see any kind of I/O in there, then that's a red
flag. Shoving stuff in Futures without thinking is not going to solve
your performance problems. Especially in the case of a web server, in
which the requests are already paralellized and the above would get
executed in response to requests, shoving purely CPU-bound in that
Future constructor will make your logic slower to execute, not faster.

Also, in case you want to initialize a `Future[T]` with a constant,
always use `Future.successful()`.

### 4.4. MUST use Scala's BlockContext on blocking I/O

This includes all blocking I/O, including SQL queries. Real sample:

```scala
Future {
  DB.withConnection { implicit connection =>
    val query = SQL("select * from bar")
    query()
  }
}
```

Blocking calls are error-prone because one has to be aware of exactly
what thread-pool gets affected and given the default configuration of
the backend app, this can lead to non-deterministic dead-locks. It's a
bug waiting to happen in production.

Here's a simplified example demonstrating the issue for didactic purposes:

```scala
implicit val ec = ExecutionContext
  .fromExecutor(Executors.newFixedThreadPool(1))

def addOne(x: Int) = Future(x + 1)

def multiply(x: Int, y: Int) = Future {
  val a = addOne(x)
  val b = addOne(y)
  val result = for (r1 <- a; r2 <- b) yield r1 * r2

  // this will dead-lock
  Await.result(result, Duration.Inf)
}
```

This sample is simplified to make the effect deterministic, but all
thread-pools configured with upper bounds will sooner or later be
affected by this.

Blocking calls have to be marked with a `blocking` call that signals
to the `BlockContext` a blocking operation. It's a very neat mechanism
in Scala that lets the `ExecutionContext` know that a blocking operation
happens, such that the `ExecutionContext` can decide what to do about
it, such as adding more threads to the thread-pool (which is what
Scala's ForkJoin thread-pool does).

All the code has to be reviewed and whenever a blocking call happens,
this is the fix:

```scala
import scala.concurrent.blocking
// ...
blocking {
  someBlockingCallHere()
}
```

NOTE: the `blocking` call also serves as documentation, even if the
underlying thread-pool doesn't support `BlockContext`, as things that
block are totally non-obvious.

### 4.5. SHOULD NOT block

Sometimes you have to block the thread underneath - unfortunately JDBC
doesn't have a non-blocking API. However, when you have a choice,
never, ever block. For example, don't do this:

```scala
def fetchSomething: Future[String] = ???

// later ...
val result = Await.result(fetchSomething, 3.seconds)
result.toUpperCase
```

Prefer keeping the context of that Future all the way:

```scala
def fetchSomething: Future[String] = ???

fetchSomething.map(_.toUpperCase)
```

Also checkout [Scala-Async](https://github.com/scala/async) to make
this easier.

### 4.6. SHOULD use a separate thread-pool for blocking I/O

Related to
[Rule 4.4](#44-must-use-scalas-blockcontext-on-blocking-io), if you're
doing a lot of blocking I/O (e.g. a lot of calls to JDBC), it's better
to create a second thread-pool / execution context and execute all
blocking calls on that, leaving the application's thread-pool to deal
with CPU-bound stuff.

So you could do initialize this second execution context like:

```scala
import java.util.concurrent.Executors

// ...
private val ioThreadPool = Executors.newCachedThreadPool(
  new ThreadFactory {
    private val counter = new AtomicLong(0L)

    def newThread(r: Runnable) = {
      val th = new Thread(r)
      th.setName("eon-io-thread-" +
      counter.getAndIncrement.toString)
      th.setDaemon(true)
      th
    }
  })
```

Note that here I prefer to use an unbounded "cached thread-pool", so
it doesn't have a limit. When doing blocking I/O the idea is that
you've got to have enough threads that you can block. But if unbounded
is too much, depending on use-case, you can later fine-tune it, the
idea with this sample being that you get the ball rolling.

And then you could provide a helper, like:

```scala
def executeBlockingIO[T](cb: => T): Future[T] = {
  val p = Promise[T]()

  ioThreadPool.execute(new Runnable {
    def run() = try {
      p.success(blocking(cb))
    }
    catch {
      case NonFatal(ex) =>
        logger.error(s"Uncaught I/O exception", ex)
        p.failure(ex)
    }
  })

  p.future
}
```

Also, don't expose this I/O thread-pool as an execution context that
can be imported as an implicit in scope, because then people would
start using it for CPU-bound stuff by mistake, so it's better to hide
it and provide this helper.

### 4.7. All public APIs SHOULD BE thread-safe

As a general rule of software engineering on top of the JVM,
absolutely all public APIs from inside your process will end up being
used in a context in which multiple threads are using these APIs at
the same time. It's best practice for all public APIs (all components
in your Cake for example) to be designed as thread-safe from the get
go, because you do not want to chase concurrency bugs.

And if a public API is not thread-safe for some reason (like the usual
compromises made in software development), then state this fact in
BOLD CAPITAL LETTERS.

The reason is simple - if an API is not thread-safe, then there's no
way a user of that API can know about the way to synchronize on
it. Example:

```scala
val list = mutable.List.empty[String]
```

Lets say this mutable list is exposed. On what lock is a user supposed
to synchronize? On the list itself? That's not enough, being a pretty
useless lock to have in a larger context. Remember, locks are not
composable and are very error-prone. Never leave the responsibility of
synchronizing for contention on your users.

### 4.8. SHOULD avoid contention on shared reads

Meet
[Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl's_law). Synchronizing
with locks drastically limits the parallelization possible and thus
vertical scalability. Reads are embarrassingly paralellizable, so
avoid at all times doing this:

```scala
def fetch = synchronized { someValue }
```

Come up with better synchronization schemes that does not involve
synchronizing reads, like atomic references or STM. If you aren't able
to do that, then avoid this altogether by using proper abstractions.

### 4.9. MUST provide a clearly defined and documented protocol for each component or actor that communicates over async boundaries

A function signature is not enough for documenting the protocol of
problematic components. Especially when talking about communications
over asynchronous boundaries (between threads, between processes on
the network, etc...), the protocol needs a lot of detail on what you
can and cannot rely on.

As a guideline, don't shy away from writing comments and document:

- concurrency and latency concerns
- the proper ordering of calls
- everything that can go wrong

### 4.10. SHOULD always prefer single-producer scenarios

Shared writes are not parallelizable, whereas shared reads are
embarrassingly parallelizable. As a metaphor, 100,000 people can watch
the same soccer game on the same stadium at the same time (reads), but
100,000 people cannot all use the same bathroom (writes). In a
multi-threading scenario, prefer single producer / multi consumer
scenarios. This has the effect of avoiding contention and performance
problems. An app does not scale vertically with multiple producers
pounding on the same resource, because Amdahl's Law.

Checkout [LMAX Disruptor](https://lmax-exchange.github.io/disruptor/).

### 4.11. MUST NOT hardcode the thread-pool / execution context

This is a general design issue related to the project as a whole, but don't do this:

```scala
import scala.concurrent.ExecutionContext.Implicits.global

def doSomething: Future[String] = ???
```

Tight coupling between the execution context and your logic is not
good and that import is tight-coupling, especially since in the
context of a Play2 application you need to use a
[different thread-pool](https://www.playframework.com/documentation/2.3.x/ThreadPools).

Just pass the ExecutionContext around as an implicit parameter. It's
idiomatic and acceptable this way:

```scala
def doSomething(implicit ec: ExecutionContext): Future[String] = ???
```





## 5. Akka Actors



### 5.1. SHOULD evolve the state of actors only in response to messages received from the outside

When using Akka actors, their mutable state should always evolve in
response to messages received from the outside. An anti-pattern that
comes up a lot is this:

```scala
class SomeActor extends Actor {
  private var counter = 0
  private val scheduler = context.system.scheduler
    .schedule(3.seconds, 3.seconds, self, Tick)

  def receive = {
    case Tick =>
      counter += 1
  }
}
```

In the example above the actor schedules a Tick every 3 seconds that
evolves its state. This is an extremely costly mistake. The actor's
behavior becomes totally non-deterministic and impossible to test
right.

If you really need to periodically do something inside an actor, then
that scheduler must not be initialized inside the actor. Take it out.

### 5.2. SHOULD mutate state in actors only with context.become

Say we've got an actor that mutates its state (most actors do),
doesn't even matter what state that is:

```scala
class MyActor extends Actor {
  val isInSet = mutable.Set.empty[String]

  def receive = {
    case Add(key) =>
      isInSet += key
      
    case Contains(key) =>
      sender() ! isInSet(key)
  }
}

// Messages
case class Add(key: String)
case class Contains(key: String)
```

Since we are using Scala, we want to be as pure as practically
possible, we want to deal with immutable data-structures and pure
functions, we want to go FP to reduce the area for
[accidental complexity](https://en.wikipedia.org/wiki/No_Silver_Bullet)
and let me tell you, there's nothing pure, immutable or referentially
transparent about the above ;-)

Meet [context.become](http://doc.akka.io/docs/akka/2.3.4/scala/actors.html#Become_Unbecome):

```scala
import collection.immutable.Set

class MyActor extends Actor {
  def receive = active(Set.empty)

  def active(isInSet: Set[String]): Receive = {
    case Add(key) =>
      context become active(isInSet + key)
      
    case Contains(key) =>
      sender() ! isInSet(key)
  }
}
```

If that doesn't instantly ring a bell, just wait until you'll have to
model a state machine with 10 states in it and dozens of possible
transitions and effects to go along with it, then you'll get it.

### 5.3. MUST NOT leak the internal state of an actor in asynchronous closures

Again with the mutable state, spot the problem:

```scala
class MyActor extends Actor {
  val isInSet = mutable.Set.empty[String]

  def receive = {
    case Add(key) =>
      for (shouldAdd <- validate(key)) {
        if (shouldAdd) isInSet += key
      }
        
    // ...
  }
  
  def validate(key: String): Future[Boolean] = ???
}
```

Chaos ensues, hell's doors open for a whole range of non-deterministic
bugs that could happen due to multi-threading issues. This is a
general problem with functions that execute asynchronously and that
capture variables that aren't meant to escape their
context. [Spores](http://docs.scala-lang.org/sips/pending/spores.html)
is a proposal for macros-enabled closures that are supposed to make
this safer, but until then just be careful.

First of all, see the rule about using `context.become` for mutating
state, which is already a step in the right direction. And then you
need to deal with this by sending another message to our actor when
our future is done:

```scala
import akka.pattern.pipeTo

class MyActor extends Actor {
  val isInSet = mutable.Set.empty[String]

  def receive = {
    case Add(key) =>
      val f = for (isValid <- validate(key))
        yield Validated(key, isValid)
        
      // sending the result as a message back to our actor
      f pipeTo self

    case Validated(key, isValid) =>
      if (isValid) isInSet += key
              
    // ...
  }
  
  def validate(key: String): Future[Boolean] = ???
}

// Messages
case class Add(key: String)
case class Validated(key: String, isValid: Boolean)
```

And of course, we could be modeling a state-machine that doesn't
accept any more requests until the last one is done. Let us also get
rid of that mutable collection and also introduce back-pressure
(i.e. we need to tell the sender when it can send the next item):

```scala
import akka.pattern.pipeTo

class MyActor extends Actor {
  def receive = idle(Set.empty)

  def idle(isInSet: Set[String]): Receive = {
    case Add(key) =>
      // sending the result as a message back to our actor
      validate(key).map(Validated(key, _)).pipeTo(self)
      
      // waiting for validation
      context.become(waitForValidation(isInSet, sender()))
  }

  def waitForValidation(set: Set[String], source: ActorRef): Receive = {
    case Validated(key, isValid) =>
      val newSet = if (isValid) set + key else set
      // sending acknowledgement of completion
      source ! Continue
      // go back to idle, accepting new requests
      context.become(idle(newSet))

    case Add(key) =>
      sender() ! Rejected
  }

  def validate(key: String): Future[Boolean] = ???
}

// Messages

case class Add(key: String)
case class Validated(key: String, isValid: Boolean)
case object Continue
case object Rejected
```

Yeap, actor-based designs can get tricky.

### 5.4. SHOULD do back-pressure

Say you've got an actor that produces values - like reading items from
a RabbitMQ or your own half-assed queue stored in a MySQL table, or
files that have to be observed and processed as soon as the actor sees
them popping up in a certain directory and so on. This producer needs
to push work into a number of variable actors.

Problems:

1. if the queue of messages is unbounded, with slow consumers that
   queue can blow up
2. distribution can be inefficient, as a worker could end up with
   multiple pending items whereas another worker could be standing
   still

A correct, worry-free design does this:

- workers must signal demand (i.e. when they are ready for processing more items)
- the producer must produce items only when there is demand from workers

Here's a detailed sample with comments:

```scala
/**
 * Message signifying acknowledgement that upstream can send the next
 * item.
 */
case object Continue

/**
 * Message used by the producer for continuously polling the
 * data-source, while in the polling state.
 */
case object PollTick

/**
 * State machine with 2 states:
 *
 *  - Standby, which means there probably is a pending queue of items waiting to
 *    be sent downstream, but the actor is waiting for demand to be signaled
 * 
 *  - Polling, which means that there is demand from downstream, but the
 *    actor is waiting for items to happen
 *
 * IMPORTANT: as a matter of protocol, this actor must not receive multiple
 *            Continue events - downstream Router should wait for an item
 *            to be delivered before sending the next Continue event to this
 *            actor.
 */
class Producer(source: DataSource, router: ActorRef) extends Actor {
  import Producer.PollTick

  override def preStart(): Unit = {
    super.preStart()
    // this is ignoring another rule I care about (actors should evolve
    // only in response to external messages), but we'll let that be
    // for didactical purposes
    context.system.scheduler.schedule(1.second, 1.second, self, PollTick)
  }

  // actor starts in standby state
  def receive = standby

  def standby: Receive = {
    case PollTick =>
      // ignore

    case Continue =>
      // demand signaled, so try to send the next item
      source.next() match {
        case None =>
          // no items available, go in polling mode
          context.become(polling)
          
        case Some(item) =>
          // item available, send it downstream,
          // and stay in standby state
          router ! item
      }
  }

  def polling: Receive = {
    case PollTick =>
      source.next() match {
        case None =>
          () // ignore - stays in polling
        case Some(item) =>
          // item available, demand available
          router ! item
          // go in standby
          context.become(standby)
      }
  }
}

/**
 * The Router is the middleman between the upstream Producer and
 * the Workers, keeping track of demand (to keep the producer simpler).
 *
 * NOTE: the protocol of Producer needs to be respected - so
 *       we are signaling a Continue to the upstream Producer
 *       after and only after a item has been sent downstream
 *       for processing to a worker. 
 */
class Router(producer: ActorRef) extends Actor {
  var upstreamQueue = Queue.empty[Item]
  var downstreamQueue = Queue.empty[ActorRef]

  override def preStart(): Unit = {
    super.preStart()
    // signals initial demand to upstream
    producer ! Continue
  }

  def receive = {
    case Continue =>
      // demand signaled from downstream, if we have items to send
      // then send, otherwise enqueue the downstream consumer
      if (upstreamQueue.isEmpty) {
        downstreamQueue = downstreamQueue.enqueue(sender)
      }
      else {
        // no need to signal demand upstream, since we've got queued
        // items, just send them downstream
        val (item, newQueue) = upstreamQueue.dequeue
        upstreamQueue = newQueue
        sender ! item

        // signal demand upstream for another item
        producer ! Continue
      }

    case item: Item =>
      // item signaled from upstream, if we have queued consumers
      // then signal it downstream, otherwise enqueue it
      if (downstreamQueue.isEmpty) {
        upstreamQueue = upstreamQueue.enqueue(item)
      }
      else {
        val (consumer, newQueue) = downstreamQueue.dequeue
        downstreamQueue = newQueue
        consumer ! item

        // signal demand upstream for another item
        producer ! Continue
      }
  }
}

class Worker(router: ActorRef) extends Actor {
  override def preStart(): Unit = {
    super.preStart()
    // signals initial demand to upstream
    router ! Continue
  }
  
  def receive = {
    case item: Item =>
      process(item)
      router ! Continue
  }
}
```