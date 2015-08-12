# LambdaOmega
A simple wrapper API to make usage of Java collections, lambdas and CompletableFuture more simple, concise and enjoyable.

## What’s in the box?
LambdaOmega consists of only a few classes. For brevity reasons, most of their names consist of a single letter.
*	`L` (“List”) is a wrapper for List.
*	`M` (“Map”) is a wrapper for Map.
*	`S` (“Set”) is a wrapper for Set.
*	`R` (“Range”) is a wrapper for an IntStream range.
*	`V2` (“Vector 2D”) represents a 2D vector (= a 2-ary tuple). It can be converted into a Map.Entry.
*	`F` (“Function”) is a wrapper for functional interfaces (lambda expressions) which also provides helper methods to convert functions.
*	`C` (“Collection”) is the base class for L and M and provides additional helper methods to convert collections.
*	`U` (“Utils”) provides additional miscellaneous helper methods.
*	`Promise` is a wrapper and a drop-in replacement for CompletableFuture, providing several simplifications and fixes for the API. It can be used independently of all the other classes. It’s discussed in a separate section.

## How to use it
LambdaOmega is not released yet; however, if you feel adventurous, can use [JitPack](https://jitpack.io/) to add its SNAPSHOT dependency to your Maven project:
```
<dependency>
    <groupId>com.github.codebulb</groupId>
    <artifactId>LambdaOmega</artifactId>
    <version>aeb4090f62</version>
</dependency>
```
Replace the `version` by the current commit hash to get the newest SNAPSHOT. Visit [JitPack's docs](https://jitpack.io/docs/) for more information.

## Getting started with Collections
The heart of LambdaOmega are the wrapper classes `L` and `M` which wrap around vanilla Java Collection `List` or `Map`, respectively (decorator pattern) to provide a more concise and more enjoyable API to the underlying collection.
You can wrap an L around everything which can be turned into a List: a List, a `Stream`, varargs:
```
import static ch.codebulb.lambdaomega.L.*;

L<Integer> myL = l(0, 1, 2);
```
The wrapper function invokes the L constructor and returns the L. You can’t invoke that constructor explicitly, always use the convenience wrapper function instead.

It’s best practice to statically import the `l()` method so you can write just `l()` instead of `L.l()`.

In the most simple case, you just want to return the underlying List:
```
List<Integer> mylist = l(0, 1, 2).l;
```
There’s a shorthand method for this simple case:
```
List<Integer> list = list(0, 1, 2);
```
Note that `l(Collection)` returns a one-element L with the collection provided being the one element. To create an L containing all the elements of the collection, use
```
L<Integer> flatList = L(list(0, 1, 2));
```
Some wrapper methods return another wrapper (e.g. of type L) or `this`; these are “intermediate” functions. You can chain another function call after them (builder pattern). Other wrapper methods return the underlying Java collection (e.g. of type List) or any other type; these are “terminal” functions. The wrapper API adheres to a simple method naming schema:
* methods with names starting with an `UpperCase` letter are **intermediate** functions
* methods with names consisting of a `single letter` are **intermediate** functions as well. These are typically shorthand aliases for another intermediate function.
* methods with names starting with a `lowerCase` letter are **terminal** functions

There are no exceptions to these rules. (Although they don’t apply to the special `Promise` class.)

With these rules in mind, we can start working with a L, e.g. add a new element:
```
List<Integer> outcome = l(0, 1, 2).add(3);
```
This is a terminal function. Using the equivalent intermediate function, we can chain method calls:
```
L<Integer> modifiedList = l(0, 1, 2).Add(3).Add(4, 5);
```
There’s a shorthand for the `Add()` method:
```
L<Integer> modifiedList2 = l(0, 1, 2).a(3).a(4, 5);
```
When constructing a Map, you make intense use of method chaining:
```
Map<String, Integer> map = m("a", 0).Insert("b", 1).i("c", 2).m;
```
(we cover Map' `insert()` method in a bit.)

There’s a two-arg constructor for empty Maps of an explicit key / value type. If the “key” class is omitted, it’s assumed to be String:
```
M<String, Integer> emptyM = m(Integer.class);
```
## Collection methods
None of the LambdaOmega collections implements Java’s Collection or Map interface, for two reasons:
* We don’t want to inherit this bloated, flawed API.
* It encourages us to use LambdaOmega collections locally only and unwrap them into Java collections for external method calls.

However, L and M implement a set of methods closely inspired by Collection and Map; most methods are actually the same, others are augmented to e.g. accept varargs or to provide a more reasonable return type.

Some of these methods exist both as intermediate and terminal function. For the most common methods, a shorthand one-letter variant exists.

We’ve already met the `add` / `Add` / `a` method on L. There’s also `addAll` / `AddAll` / `A`; all of these also works with varargs:
```
List<Integer> zeroToSix = l(0, 1, 2).A(list(3, 4)).addAll(l(5, 6));
```
For M, there’s an add method variant named `insert` / `Insert` / `I` / `insertAll` / `InsertAll` / `I` which will invoke `put()`, but only after a check preventing you from inserting the same key twice; otherwise, a `MapEntryKeyAlreadyPresentException` is thrown and the map is not modified. This method comes in handy e.g. in unit tests when you explicitly build a map and you want to make sure that you don’t accidentally insert the same key twice. Note that this check costs significantly more performance than just performing put().
```
m("a", 0).i("b", 1).I(m("c", 2).i("d", 3), m("c", 9)); // exception because of "c"!
```
For L or M, you can set a single element with `set` / `Set` / `s` / `put` / `Put` / `p` or set all elements included in a Map data structure with `setAll` / `SetAll` / `S` / `putAll` / `PutAll` / `P`.
```
m("a", 0).p("b", 1).p("c", 2).P(m("d", 3).i("e", 4));
```
For L or M, you can get a single element or multiple elements with `get` / `Get` / `g`:
```
String a = l("a", "b", "c").g(0);
L<String> aAndC = l("a", "b", "c").g(0, 2);
```
For L, you can remove a single element or a collection with `remove` / `Remove` / `r` / `removeAll` / `RemoveAll` / `R`:
```
L<String> aAndB = l("a", "b", "c", "d", "e").r("c").R(list("d", "e"));
```
For M, you can remove a single element or a collection with `deleteKey` / `DeleteKey` / `d` / `deleteAllKeys` / `DeleteAllKeys` / `D` or `deleteValue` / `DeleteValue` / `deleteAllValues` / `DeleteAllValues` (it’s called “delete” instead of “remove” to avoid naming clashes for the one-letter abbreviation):
```
M<String, Integer> bAndC = m("a", 0).i("b", 1).i("c", 2).d("a");
```
Also, you can convert an L / M into almost any collection with a corresponding `to…(…)` method:
```
Set<Integer> set = l(0, 1, 2).toSet();
```
These conversion methods internally use the `C.to…(…)` static helper methods. You can call them directly to convert collection without the need to create intermediate L / M instances.

There are a lot of additional methods for L and M. For more information, visit the [API docs](http://codebulb.github.io/pages/2015/08/LambdaOmega/doc/).


## A List is a Map and a Map is a List
Now comes the fun part. Because the LambdaOmega API lives independently of vanilla Java Collection / Map API, it features its own API which is more simple, consistent and powerful at the same time, whilst keeping it as close to the original Java APIs as possible.

Most importantly, a List is also a Map from int to T, and a Map is also a List of entries. More precisely, both L and M implement the `SequencedI` (Collection-like access) interface as well as the `IndexedI` (Map-like access) interface.
Thus, you can call L methods such as `indexOf()` on a M:
```
String c = m("a", 0).i("b", 1).i("c", 2).indexOf(2);
```
and M methods such as `insert()` on a L:
```
L<String> abcd = l("a", "b").I(m(2, "c").i(3, "d"));
```
Finally, indexed access using get() is actually provided by the `ReadonlyIndexedI` interface which is implemented by L, M, and F, thus you can use “one interface to rule them all”:
```
ReadonlyIndexedI<Integer, String> indexed = m(0, "a").i(1, "b");
String b = indexed.g(1);
indexed = l("a", "b");
b = indexed.g(1);
indexed = f((Integer it) -> it == 0 ? "a" : "b");
b = indexed.g(1);
```
## Collection methods with Lambdas
One of the main reasons to use LambdaOmega are the simplifications to use collections with lambda expressions in functional programming. When compared with vanilla Java lambda use, the API is much more simple and concise. Many of these modifications are inspired by the equivalent Groovy semantics.

Most importantly, functions are invoked on the collection object itself, without the need to create an intermediate stream.

To execute a forEach loop:
```
l("a", "b", "c").forEach(it -> println(it));
```
(`U.println()` is a shorthand for `System.out.println()`.)

Keep in mind that a L is also a Map, thus there’s a variant of the forEach loop which takes both a key and value argument whereby in a L, the key is the index (this method is thus equivalent to a Groovy `forEachWithIndex()` loop).
```
l("a", "b", "c").forEach((index, it) -> println(index + ": " + it));
```
Again note how intermediary functions are marked by a name starting with an UpperCase letter. Here we invoke two consecutive  map operations:
```
List<Integer> list246 = l(0, 1, 2).Map(it -> it + 1).map(it -> it * 2);
```
Some functions incorporate syntax changes (when compared to their vanilla Java couterpart) to facilitate their usage. With mapEntries(), you can map list elements to a Map. Note that the lambda returns a `M.E` Map Entry which is much simpler than having two separate return values as in the corresponding vanilla Java method.
```
Map<String, Integer> stringsToSize = l("a", "ab", "abc").mapEntries(it -> e(it, it.length()));
```
(`e()` is a shorthand to create a M.E with key, value.)

`find()` returns null or a result instead of an `Option`:
```
Integer negative = l(1, 2, -1, 3).find(it -> it < 0);
```
There are sort operations which work with consecutive sort key mappers:
```
l("John Smith", "Bob Miller", "Tim Miller", "Anna Smith").sortAscBy(it -> it.split(" ")[1], it -> it.split(" ")[0]);
```
There are also additional functional operations such as `flatten()`, `flattenDeep()`, `join()`, `partition()`, and more.
```
List<Integer> list123456 = l(0, 1, l(2, 3, l(4)), list(5, 6)).<Integer> flattenDeep();
```
There’s also a `WithDefault()` method which allows you for any ReadonlyIndexedI to register a function the return value of which is returned if a get() access would return null or get out of bounds (as inspired by Groovy). You can use this e.g. to easily create a Map of Lists:
```
M<String, L> withDefault = m(L.class).WithDefault(it -> l());
withDefault.g("a").a(1);
println(withDefault); // prints M{a=L[1]}
```
Take a look at the [API docs](http://codebulb.github.io/pages/2015/08/LambdaOmega/doc/) to see all available functional operations.

## Ranges
The `R` class represents an int range. It’s basically syntactic sugar to create ranges using the Java 8 stream API.

Create a range and convert it into a List like this:
```
List<Integer> list012 = r(0).to(3).list;
```
Support for range access has been added to L:
```
List<String> listBcd = l("a", "b", "c", "d", "e").get(r(1).to(3));
```
## 2D vectors
The V2 class represents a 2-dimensional vector = a 2-ary tuple:
```
V2<String, Integer> vector = v("a", 0);
```
By simply adding another value pair, the whole structure turns into a L<V2>:
```
V2.LV2<String, Integer> vector2 = v("a", 0).a("a", 1);
```
## Promises
This library also features a drop-in-replacement for CompletableFuture, simplifying and fixing its partially flawed API. I went into some more details in the [accompanying blog post](http://www.codebulb.ch/2015/08/lambdaomega-java-collections-lambdas-promises-simplified.html#drop-in-replacement).
## Project status and future plans
**The current status of the library is unstable / experimental / request for feedback.**

Even though its core functionality is perfectly working already, there’s no stable release yet. I decided to share it on GitHub nonetheless as I’m hoping to receive feedback before fixing the API.

Please let me know what you think about this library. Do you think it is useful, useless, harmful, crazy, funny? I’m open to any kind of improvement proposals. I’ve started this project for my own pleasure and usage and to learn more about Java 8’s collection API. I will try to work on a complete, stable implementation in the near future, but I may as well leave it as it is for now. If you’re really enthusiastic about this library and you’d like to team up to get it finished, please let me know.

## For more information
Please visit the **[accompanying blog post](http://www.codebulb.ch/2015/08/lambdaomega-java-collections-lambdas-promises-simplified.html)** to learn more about why I created this library or check out the **[API docs](http://codebulb.github.io/pages/2015/08/LambdaOmega/doc/)**.
