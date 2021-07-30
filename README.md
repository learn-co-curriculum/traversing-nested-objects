# Traversing Nested Objects

## Learning Goals

1. Explain what a nested object looks like
2. Describe how to access inner properties
3. Find an element in a nested array

## Introduction

When we're looking for occurrences of a word or concept in a book, we often turn
to the index. The index tells us where we can find more information on that
concept by giving us a list (or page numbers) that we can use to look up more
information in the book. Additionally, it might include information that is
related to the heading that we looked up in a _sublist_ (e.g. Animals >
Vertebrate; Animals > Invertebrate). We link the connections between these lists
in our heads, and it doesn't cause any issues to think of one list containing
other lists. The index itself is, after all, a kind of list.

## Explain What a Nested Object Looks Like

### Objects in Objects

Remember when we said that the values in an object can be _anything_? Well, like
the lists in the index in the example above, the values in an object **can also
be other objects**.

![mind blown](http://i.giphy.com/5aLrlDiJPMPFS.gif)

Type (don't just copy!) the following into your console to see what we mean:

```javascript
const person = {
  name: "Terrance Roberts",
  occupation: {
    title: "District Manager",
    yearsHeld: 2
  },
  pets: [{
    kind: "dog",
    name: "Fiona"
  }, {
    kind: "cat",
    name: "Ralph"
  }]
}
```

If you look closely, you can see that we've kind of seen this before when we
looped over an array containing objects. So it should be familiar!

## Describe How to Access Inner Properties

How would you imagine we'd access the `yearsHeld` field?
Let's try:

```javascript
person.yearsHeld
```

We should see `undefined`. However, we can see that `yearsHeld` is a property of
`occupation`, which in turn is a property of `person`. If we try
`occupation.yearsHeld`, that'll throw an error because `occupation` is not
defined globally. It's only a property of `person`.

Let's try:

```javascript
person.occupation
```

We should see this printed to the console:

```javascript
{ title: "District Manager", yearsHeld: 2 }
```

That suggests that if we type:

```javascript
person.occupation.yearsHeld
```

We should now see the `yearsHeld` value:

```javascript
2
```

Sweet!

### Arrays in Arrays

Let's get a little bit more abstract. In the above example, we had a name
for each field that we wanted to access (`person`, `occupation`, and
`yearsHeld`). If we had wanted to access the second pet's name, we could have
done `person.pets[1].name`. Notice that we need to specify the _index_ in the
`pets` array of the pet that we want (`[1]`).

Working with arrays isn't all that different. It's just that instead of named
properties of objects (and sub-objects), we have indexes of arrays (and
sub-arrays). And, of course, JavaScript is flexible enough that we can mix the
two:

```javascript
const numberCollections = [1, [2, [4, [5, [6]], 3]]]
```

Given the above nested array, how would we get the number `6`?

* First, we'd need the second element of `numberCollections`:

```javascript
numberCollections[1]
//=> [2, [4, [5, [6]], 3]]
```

* Then we'd need the second element of that element:

```javascript
numberCollections[1][1]
//=> [4, [5, [6]], 3]
```

* Then get the second element of _that_ element:

```javascript
numberCollections[1][1][1]
//=> [5, [6]]
```

* And again:

```javascript
numberCollections[1][1][1][1]
//=> [6]
```

* Finally, the first element of that element to return
the number, instead of an `Array`:

```javascript
numberCollections[1][1][1][1][0]
//=> 6
```

That can be a lot to keep track of. Just remember that each lookup (square
brackets) selects a different array for each subsequent lookup. To recap, what
we're really doing is:

```javascript
[1, [2, [4, [5, [6]], 3]]] // numberCollections
[2, [4, [5, [6]], 3]]      // numberCollections[1]
[4, [5, [6]], 3]           // numberCollections[1][1]
[5, [6]]                   // numberCollections[1][1][1]
[6]                        // numberCollections[1][1][1][1]
6                          // numberCollections[1][1][1][1][0]
```

**Note:** JavaScript is flexible enough that we can mix the objects and arrays
together.

## Find an Element in a Nested Array

### Use `for`

What if we have criteria for finding an element that we know is in a nested data
structure? Let's implement a simple `find` function that takes two arguments: an
array (which can contain sub-arrays) and a function that returns `true` for the
thing that we're looking for.

```javascript
function find(array, criteriaFn) {
  for (let i = 0; i < array.length; i++) {
    if (criteriaFn(array[i])) {
      return array[i]
    }
  }
}
```

> **REMEMBER** In JavaScript, functions are "first class data" just like
> `Number`s. It's probably not surprising to imagine writing: `find( [1,2,["a",
> "b"]], 3.14)` or `find( [1,2,["a", "b"]], "Byron the Poodle")`. Since
> functions are first-class data, just like `Number`s, it means that the
> following code shouldn't stay foreign to you: `find( [1,2,["a", "b"]],
> function(someNumber){ return someNumber % 2 === 0} )`. In fact, writing
> functions that test some condition (a "criteria function") is ***so*** common,
> JavaScript even has an advanced syntax for writing functions that do one
> evaluation and return the value: _the arrow function syntax_. Thus we could
> write: `find( [1,2,["a", "b"]], someNumber => someNumber % 2 === 0 )`. This
> code would do something like find all the members in the array that are even.

The above will work for a flat array — but what if `array` looks like the
`numberCollections` nested array and we want to find the first element that's
`> 5`? We'll need some way to move down the levels of the array (like we described
above).

Follow along with the code below — we know it's a little tricky, but be sure to
read the comments!

```javascript
function find(array, criteriaFn) {
  // initialize two variables, `current`, and `next`
  // `current` keeps track of the element that we're
  // currently on, just like we did when unpacking the
  // array above; `next` is itself an array that keeps
  // track of the elements (which might be arrays!) that
  // we haven't looked at yet
  let current = array
  let next = []

  // hey, a `while` loop! this loop will only
  // trigger if `current` is truthy — so when
  // we exhaust `next` and, below, attempt to
  // `shift()` `undefined` (when `next` is empty)
  // onto `current`, we'll exit the loop
  //
  // Note that we had to add on this || statement
  // if current is the number 0 it won't be run in the
  // loop. Recall your truthy / falsey rules! This is
  // a subtle bug that went unnoticed in this code for
  // many years!
  while (current || current === 0) {
    // if `current` satisfies the `criteriaFn`, then
    // return it — recall that `return` will exit the
    // entire function!
    if (criteriaFn(current)) {
      return current
    }

    // if `current` is an array, we want to push all of
    // its elements (which might be arrays) onto `next`
    if (Array.isArray(current)) {
      for (let i = 0; i < current.length; i++) {
        next.push(current[i])
      }
    }

    // after pushing any children (if there
    // are any) of `current` onto `next`, we want to take
    // the first element of `next` and make it the
    // new `current` for the next pass of the `while`
    // loop
    current = next.shift()
  }

  // if we haven't
  return null
}
```

Type the code (you can exclude the comments) above into your console and run it
a few times.

Try it with the `numberCollections` nested array and the function
`number => number > 5`.

```javascript
find(numberCollections, number => number > 5)
```

Does it return the result you'd expect? What about if we pass this into
`criteriaFn`?

```javascript
number => (typeof number === 'number' && number > 5)
```

Without knowing it, you've just implemented your first **[breadth-first
search](https://en.wikipedia.org/wiki/Breadth-first_search)**! Congratulations!

Breadth-first search is one of the main algorithms (that's right, you've
conquered an algorithm) used to search through nested objects. It earned its
name because it looks at the siblings of an object (the elements that are on the
same level) before looking at the children (the elements that are one or more
levels down).

### A Challenge, Should You Choose to Accept It

Can you modify the breadth-first search algorithm in such a way that it will
traverse both nested objects and nested arrays (or even a mix of both)?

## Resources

* [breadth-first search](https://en.wikipedia.org/wiki/Breadth-first_search)
