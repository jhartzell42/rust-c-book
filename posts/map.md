+++
title = "A Rust Gem: The Rust Map API"
date = "2022-03-12"
slug = "rust-map-entry"
author = "Jimmy Hartzell"
tags = ["programming", "Rust", "computers", "C++", "Rust vs C++"]
showFullContent = false
draft = false
+++

For my next entry in my [series](/tags/rust-vs-c++/) comparing Rust
to C++, I will be discussing a specific data structure API: the Rust
map API. Maps are often one of the more awkward parts of a collections
library, and the Rust map API is top-notch, especially its
[entry API](https://doc.rust-lang.org/book/ch08-03-hash-maps.html?highlight=entry#only-inserting-a-value-if-the-key-has-no-value) -- I literally squealed
when I first learned about entries.

And as we shall discuss, this isn't just because Rust made better choices
than other standard libraries when designing the maps API. Even more so,
it's because the Rust programming language provides features that better
expresses the concepts involved in querying and mutating maps. Therefore,
this serves as a window into some deep differences between C++ and Rust
that show why Rust is better.

And for this post, specifically, we'll also be discussing Java, so
this will be a three-way comparison, between Java, C++ and Rust.

## Reading from a Map

So, let's talk about map APIs. But before we get to `Entry` and friends,
let's discuss something a little simpler: getting an item from a
map. Let's say we have a sorted map of strings to integers:

* In Java, `TreeMap<String, Integer>`
* In C++, `std::map<std::string, int>`
* In Rust, `BTreeMap<&str, i32>`

Let's also say we have a string `"foo"`, and want to know what integer
corresponds to it. Now, if we're always sure that the string we're
looking up is always in the map, then we know what we want: we want
to get an integer.

But what if we're not sure? There are plenty of situations where we want
to read a value corresponding to the key -- or do something else when
that key is not present. Maybe the value is a count, and an absent key
means 0. Or maybe the absent key means that the user has made a typo,
and needs to be informed. Or maybe the map is a cache, and the absent
key means we need to read a file or query a database. In all of these
cases, we need to know either the value, or the fact that the key
is absent.

Let's see how this is handled in our three programming languages, and
how fundamental design choices in these programming languages lead to
such APIs.

### Java `get` a (Nullable) Reference

A long time ago, Java made an extreme choice in the name of simplicity:
It divided all values into a dichotomy of "primitives" and "objects."
Primitives are passed around by implicit copy, whereas objects are
aliased through many mutable references. Objects always have optionality
built in -- any object reference is automatically "nullable," which
means you can store the special sentinal/invalid value `null` in it,
the interpretation of which varies wildly. Primitives are not optional
in this way.

Also for the sake of simplicity, and very relevantly to the topic at hand,
generics are only supported for object types, not primitives. That means
that map values can only ever be object types. And that means that our
map from strings to integers in Java doesn't use Java's primitive integer
type `int`, but rather this special wrapper/adapter type `Integer`,
which auto-casts to and from `int`, and which, like any object type,
is managed through mutable, *nullable* references. (At this point, I
for one am beginning to suspect they missed the mark on their simplicity).

So what's that mean for our map? How do we find out what value
corresponds to `"foo"` in our map, or else that there is none?
Well, the method for this is called `get`, and that returns the
value in question if there is one. And when there isn't? Well,
Java here leverages nullability, and returns `null` when there
is no value.

So we can write something like this:

```
Integer value = map.get("foo");
if (value == null) {
    System.out.println("No value for foo");
} else {
    int i_value = value;
    System.out.println("Value for foo was: " + i_value);
}
```

So far, so good. But there are problems. And perhaps I'm missing some
-- now is a good time to take a second, look at the code, and try to
imagine in your mind what problems there may be with this system (you
know, besides the fact that I have to use `i_` as improvized Hungarian
notation due to lack of support in Java for shadowing).

You have some? I'll now list what I've got.

*Problem the first:* The signature of `get` doesn't really alert
us to the possibility of a value not being in a map. This is
the sort of "edge case" that programmers regularly forget to handle;
a programmer may know, due to their situation-specific knowledge,
that the key ought to be present, and forget to consider that the
key might not be.

Compilers of strongly typed languages generally work to ensure that
programmers don't miss edge cases like this, don't make simple "thinkos"
(typos but with thought)
or "stupid mistakes." How's Java hold up? Well, remember how we mentioned
that primitives can't be `null`, but these wrapper types like `Integer`
are coercible to primitives? Well, this compiles without a word of
complaint from the compiler:

```
TreeMap<String, Integer> map = new TreeMap<String, Integer>();

map.put("foo", 3);

int foo = map.get("foo");
System.out.println("int foo: " + foo);

int bar = map.get("bar");
System.out.println("int bar: " + bar);
```

And what happens at run-time? Similar behavior to Rust's infamous
`unwrap` function. The conversion from the nullable `Integer`
and the non-nullable `int` crashes when the `Integer` is in
fact `null`:

```
int foo: 3
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "java.util.TreeMap.get(Object)" is null
        at test.main(test.java:12)
```

So you might try to fix this by querying if the key exists first:

```
TreeMap<String, Integer> map = new TreeMap<String, Integer>();

if (map.containsKey("bar")) {
    int bar = map.get("bar");
    System.out.println("int bar: " + bar);
} else {
    System.out.println("bar not present");
}
```

But now we've reached *problem the second*. Unfortunately, even though
this looks like it addresses the issue, this won't prevent the crash
either. There is nothing stopping you from putting a `null` into the map,
so this code also crashes given the right context:

```
        TreeMap<String, Integer> map = new TreeMap<String, Integer>();
        map.put("bar", null);
        if (map.containsKey("bar")) {
            int bar = map.get("bar");
            System.out.println("int bar: " + bar);
        } else {
            System.out.println("bar not present");
        }
```

So for a given key in a Java map, there are actually three possible
situations:

1. The key is absent.
2. The key corresponds to an integer.
3. The key corresponds to one of these special `null`-values.

`get` can distinguish 2 from 1 and 3, but cannot distinguish between
1 and 3. `containsKey` can distinguish 1 from 2 and 3, but cannot
distinguish 2 from 3. To distinguish all 3 scenarios, and handle
all the representable values, you need to call *both* `get`
and `containsKey`:

```
if (map.containsKey("bar")) {
    Integer bar = map.get("bar");
    if (bar == null) {
        System.out.println("bar present and null");
    } else {
        int i_bar = map.get("bar");
        System.out.println("int bar: " + i_bar);
    }
} else {
    System.out.println("bar not present");
}
```

In addition to this precaution not being enforced to the compiler,
it leads to *problem the third*: We are now querying the map twice.
We are walking the tree twice with our `containsKey` followed by
`get`.

At this point, we find ourselves scrolling through the `Map` methods
in [Java's documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html), trying to find a more general solution. `getOrDefault` might
help in some situations -- when there's a value that makes sense as the
default. `compute` might be useful -- if we're OK with modifying
the map in the process.

But in general, nothing clean exists to tidy up these problems.  And the
blame lies squarely on Java's decision to make almost all types --
and all types that can be map values -- nullable.

But wait! -- you might object -- Can't we just maintain an invariant on the
map that it contains no `null` values? If we have a map without `null`
values, all these issues -- well, many of these issues -- dry up.

And this is true. Maintaining such an invariant makes for a much cleaner
situation. Pretend you aren't allowed to put nulls in maps, and arrange
not to do it.

But, first off, maintaining an invariant like this is easier said than
done. Programmers often do this sort of thing implicitly in their
head, but it's much better to comment. Either way, you have to
trust future programmers -- even future versions of the same programmers
-- to know about the invariant, either by intuiting it (all too common)
or by reading the relevant comment (which, even if there is one, might not
happen). And you have to trust them to not intentionally violate the
invariant, and also to not accidentally violate the invariant: Are
they sure that all those values they add to the map can never be null?

And second off, somewhat shockingly, sometimes people do assign special
meanings to `null`. I said before `null` has a wide range of meanings,
and it's not uncommon to use `null` to mean special things. Maybe
"not mapped" means "load from cache," but "null" means "there actually
is no value and we know it." Or maybe the opposite convention applies.
`null` is frustratingly without intrinsic meaning.

For such situations, programmers should probably compose the map with [other
types](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
or better yet, write custom types that make the semantics of these
situations abundantly clear. But let's not put all the blame on
the programmers. If Java had really wanted to protect people from
distinguishing these "not mapped" and "mapped to null" situations, Java
maps shouldn't have made the distinction representable at all.  It's bad
programming language design to put features in a library that can only
be abused, and it's bad understanding of human nature to then solely
blame the programmers for misusing them.

### C++: No Nulls No More

So now we move on to C++.

In C++, fewer types are nullable, and non-nullable types like `int`
*can* be used as the value type of a map.  For our map, of type
`std::map<std::string, int>`, we no longer have the trichotomy of
"key not present, value null, or value non-null," but the much more
reasonable dichotomy of either the key is present and there is an `int`,
or it's absent and there isn't one.

This is, in my mind, the bare minimum a strongly typed language should
be able to provide, but after the context of Java it's worth pointing out.

There are three (3) methods in C++ that look like they might be usable
as a `get` operation, an operation where we either get an `int` value
or learn that the key is absent:

* [`at`](https://en.cppreference.com/w/cpp/container/map/at)
* [`operator[]`](https://en.cppreference.com/w/cpp/container/map/operator%5Fat)
* [`find`](https://en.cppreference.com/w/cpp/container/map/find)

See if you can identify which one is the right one to use.

Spoiler alert! It's `find`, the one whose name superficially looks least like
it'll be the right one. `at` throws an exception if the key is absent,
and `operator[]`, the one with the most appealing name, is an eldritch
abhomination which we'll discuss and condemn later.

But all well-deserved teasing aside, `find` is much better than
Java's `get`. It returns a special object -- an iterator -- that
can be easily tested to see whether we've found an `int`, and easily
probed to extract the `int`.

```
auto it = map.find(key);
if (it == map.end()) {
    std::cout << key << " not present" << std::endl;
} else {
    std::cout << key << " " << it->second << std::endl;
}
```

This is actually pretty good! The `->` operator also serves as a signal
to experienced C++ programmers that we're assuming that `it` is valid:
generally `->` or `*` means that the object being operated on is
"nullable" in some way.

So when a C++ programmer reads something like this, they have a little
bit of warning that they're doing something that might crash:

```
int foo = map.find(key)->second;
```


And certainly, they have more warning than the Java programmer with
the equivalent Java:
```
int foo = map.get(foo);
```

Of course, this is awkward. `find` returns an *iterator*, which isn't
exactly the type we'd expect for this "optional value" situation. And
to determine if the value isn't present, we compare it to `map.end()`,
which is a weird value to compare it to. Nothing about what these things
are named is specifically intuitive, and people would be forgiven for
using the accursed `operator[]`. `map["foo"]` just *looks* like an
expression for doing boring map indexing, doesn't it?

And what does `operator[]` do, if the key isn't present? It inserts the
key, with a default-constructed value. No configuration is possible of
what value gets inserted, short of defining a new type for the object
values. This is sometimes what you want -- like if your value type has a
good default (especially if you defined it yourself), or if you're about
to overwrite the value anyway. But in most cases, you want some other
behavior if the value is not present -- `operator[]` doesn't really tell
you that it inserted the item, so if you need to make a network query
or read a file or print an error, you're out of luck.  `operator[]`,
as innocuous as it looks, has surprising behavior, and that is not good.

But all in all, as far as getting values goes, as far as querying the map
goes, C++ is doing OK. Solid B result on this exam, I think. Decent work,
C++. Especially since we just looked at Java.

### The Rust `Option`

So now on to Rust: we want to query our `BTreeMap<&str, i32>`.

(Or... it might be a `BTreeMap<String, i32>`, depending on whether we
want to own the strings. This is a decision we also have to make in C++
(where we could have used `string_view`s as the keys), but do not have
to make in Java.  At least in Rust, we know that whichever decision we
make, we will not accidentally introduce undefined behavior. But that's
a distraction!)

So let's apply the same test to Rust as we've applied before.
Here, the [method in
question](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.get) is given an obvious name, `get` rather than `find`. So let's
see how it does in our test, of allowing us to read a value if present,
but know if not:

```
if let Some(val) = map.get(key) {
    println!("{key}: {val}");
} else {
    println!("{key} not present");
}
```

See, `get` returns an `Option` type. Therefore, unlike in C++, we can test
for the presence of the value and extract the value inside the same `if`
statement. Unlike in C++, the return value of `get` isn't a map-specific
type, but rather the completely normal way to express a maybe-present
value in Rust. This means that if we want to implement defaulting, we
get that for free by using the `Option` type in Rust, which implements
that already:

```
// Let's say missing keys means the count is 0:
let value = *map.get("foo").unwrap_or(&0);
```

Similarly, calling `is_none()` or pattern-matching against `None` is
much more ergonomic than comparing an iterator to `map.end()`. It requires
some more intimate knowledge -- or some follow-up reading -- to learn that
the concept of "end of collection" and "not found" are for various reasons
combined into one in C++.

So while C++ avoids the problematic elements of Java maps, Rust does so
more ergonomically, because it has a well-established `Option` type. C++
now has one as well, `std::optional`, but it hasn't yet reached its
`map` API, because it was only added very recently, in C++17.

And `Option` integrates even better than `std::optional` with the programming
language, because `Option` is just a garden-variety sum type, a
Rust `enum`, which lets you do things like `if let Some(x) = ...`,
and combine testing and unpacking in the same statement. C++ could
not design a map API this ergonomic, because they lack this fundamental
feature.

Also, unlike with `null` in Java, if you want to use `Option`
as a meaningful distinction in your map, you still can. The `get`
function would then return `Option<Option<...>>` instead of
just `Option` -- the outer one representing presence, the inner one
representing whether the value was `None` or `Some(...)`. `Option`
is composable in a way that `null` is not.

For the record, the Rust equivalent to `operator[]` -- the `Index`
trait implementation on maps -- does the equivalent to C++ `at`, and
panics if the key isn't present. While not as generally useful as `get`,
I think this is a reasonable interpretation of what `map["foo"]` should
mean.

## Mutation Station

So Rust wins, I'd say pretty handily, when comparing how to access a
value from a map, how to query them. But where Rust truly shines is when
*mutating* a map. For mutation, I'm going to approach the discussion
differently. I'm going to start by specifying what use cases might exist,
and then, in that context, we can discuss how an API might be built.

The mutation situation has a similar dilemma to querying: the key in
question might or might not already be in the map. And, for example,
we often want to change the value if the key is present, and insert a
fresh value if the key is absent.

Of course, we could always check if the key is present first, and
then do something different in these two scenarios. But that has
the same problem we already discussed for querying: We then have
to iterate the tree twice, or hash the key twice, or in general
traverse the container twice:

```
auto it = map.find(key); // first traversal
if (it != map.end()) {
    return it->second;
} else {
    int res = load_from_file(key);
    map.insert(std::pair{key, res}); // second traversal
    return res;
}
```

So what should we do for our API for this scenario, where we want to
change the value if the key is present, and insert a fresh value if
the key is absent?

Well, sometimes that fresh value is a default value,
like if we're counting and the key is the thing we're counting -- in that
case, we can always insert 0. In that case, C++'s `operator[]` -- when
combined with an appropriate default constructor -- can actually
work well.

And sometimes, that fresh value depends on the key, like if the value is a
more complicated record of many data points about the item in question.
If the value is a sophisticated OOP-style "object," and the key indexes
one of the fields also contained in the value, C++'s `operator[]` would
not work. The default value is a function of the key.

And sometimes, there isn't a default value *per se*. Sometimes, if the key
is absent, we need to do additional work to find out what value should
be inserted. This is the case if the map is a cache of some database,
accessed via IPC or file or even Internet. In that situation, we only
want to send a query if the key is not present. We would not be able
to accomplish our goals simply provide a default value when sending the
mutation operation.

C++ doesn't have anything for us here. `operator[]`
is pretty much its most sophisticated "query-and-mutate"
operation. Java, somewhat surprisingly, does have something relevant,
[`compute`](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#compute-K-java.util.function.BiFunction-).
This handles all of these situations, with a relatively unergonomic
callback function -- and as long as your map never contains `null`s.

Rust's solution, however, is to create a value that encapsulates
being at a key in the map that *might or might not* have a value
associated with it, a value of the
[`Entry`](https://doc.rust-lang.org/std/collections/btree_map/enum.Entry.html) type.

As long as you have that value, the borrow checker prevents you from
modifying the map and potentially invalidating it. And as long
as you have it, you can query which situation you're in -- the
missing key or the present key. You can update a present key. You can
compute a default for the missing key, either by providing the value or
providing a function to generate it. There are many options, and you can
read all of them in the `Entry` documentation; the world is your oyster.

So the C++ code above can be ergonomically expressed as something like
this in Rust:

```
let entry = map.entry(key.to_string());
*entry.or_insert_with(|| load_from_file(key))
```

And the idiom where we're counting something could be expressed
something like:

```
map.entry(string)
    .and_modify(|v| *v += 1)
    .or_insert(1);
```

So we get this nice little program that counts how many times
we use different command line arguments:

```
use std::collections::BTreeMap;
use std::env;

fn count_strings(strings: Vec<String>) -> BTreeMap<String, u32> {
    let mut map = BTreeMap::new();
    for string in strings {
        map.entry(string)
            .and_modify(|v| *v += 1)
            .or_insert(1);
    }
    map
}

fn main() {
    for (string, count) in count_strings(env::args().collect()) {
        println!("{string} shows up {count} times");
    }
}
```

## Conclusion

So first off, `Entry`s are super nice, and neither Java nor C++ has
anything anywhere near as nice. Even when it comes to just querying,
Rust's `get` is much better than Java's `get`, and a little more ergonomic
than C++'s `find`.

But this isn't an accident. This isn't just about Rust's map API having
a nice touch. When we look at the definition of [`Entry`](https://doc.rust-lang.org/std/collections/btree_map/enum.Entry.html),
we see things that Java and C++ can't do:

```
pub enum Entry<'a, K, V> 
where
    K: 'a,
    V: 'a, 
 {
    Vacant(VacantEntry<'a, K, V>),
    Occupied(OccupiedEntry<'a, K, V>),
}
```

First, this is an `enum`: There's two options, and in both option,
there's additional information. Of course, Java and C++ can express
a dichotomy between two options, but it's a lot clumsier. Either you'd
have to use a class hierarchy, or `std::variant`, or something else. In
Rust, this is as easy as pie, and since it does it the easy way, you can
not only use the various combinator methods in Rust, you can also use
`Entry`s with a good old-fashioned `match` or `if let` to distinguish
between the `Vacant` and `Occupied` situation.

Second, there's a little lifetime annotation there: `'a`. This is
an indication that while you have an `Entry` into a map, Rust won't
let you change it. Now, in Java and C++, there's also iterators,
which you may not change a map while you're holding, but in both
those languages, you have to enforce that constraint yourself.
In Rust, the compiler can enforce it for you, making `Entry`s
impossible to use wrong in this way.

Without both of these features, `Entry` would not have been an obvious API
to create. It would've been barely possible. But Rust's feature set encourages
things like `Entry`, which is yet another reason to prefer Rust over C++
(and Java): Rust has `enum`s (and lifetimes) and uses them to good effect.

## Addendum

I wanted to address a few points that people have raised in comments
since I posted this.

Some people have pointed out that C++ has `insert_or_assign`,
but in spite of the promising name, it just unconditionally sets a key
to be associated with a value, whether or not it previously
was. This is not the same as behaving differently based on
whether a value previously existed, and it is therefore not
relevant to our discussion.

More interestingly, it has been pointed out to me that with
the return value of `insert`, you can tell whether the `insert`
actually `insert`ed anything, and also get an iterator to the entry
that existed before if it didn't. This allows implementing some, but not
all, of the patterns of `Entry` without traversing the map twice.

For example, counting:

```
int main(int argc, char **argv) {
    std::vector<std::string> args{argv, argv + argc};
    std::map<std::string, int> counts;

    for (const auto &arg : args) {
        counts.insert(std::pair{arg, 0}).first->second += 1;
    }

    for (const auto &pair : counts) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

This works, but is much less clear and ergonomic than the `Entry`-based
API. But perhaps more importantly, this functionality is much more
constrained than `Entry`, and is equivalent to using `Entry` with just
`or_insert`, and never using any of the other methods. As another
commentator pointed out, counting is possible with just `or_insert`:

```
*map.entry(key).or_insert(0) += 1
```

But counting is just one example. C++'s `insert` is still deeply
limited. Using C++'s `insert` means you have to know *a priori* what
value you would be inserting.  You can't use it to notice that a key is
missing and then go off and do other work to figure out what the value
should be. So you can't do my `load_from_file` example.

In order to do the `load_from_file` example in C++, even with this use of
`insert`, you would have to temporarily insert some sentinal value in the
map -- and that goes against how strongly typed languages ought to work,
in addition to breaking the C++ concept of exception safety.

This is, as was pointed out in another comment, exactly what C++
programmers sometimes have to do, to meet performance goals, at the
expense of clarity and simplicity, and therefore, especially in C++,
at the expense of confidence in safety and correctness.
