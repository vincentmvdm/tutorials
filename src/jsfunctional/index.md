Despite it's name, the JavaScript language was based more on [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) than it was on Java. Scheme is functional programming language, which is a very different programming paradigm than object-oriented programming. This tutorial will introduce you to what functional programming is all about, and how you can unlock JavaScript's functional programming features.

> **WARNING:** this tutorial assumes you already read the [Introduction to JavaScript](../javascript/) tutorial. If you didn't, you should read it before continuing.

## What Is Functional Programming?

Functional programming is a style of programming that differs from the object-oriented style you might have learned in your introductory computer science courses. In object-oriented programming, we construct programs by modeling the data we will manipulate as a series of interconnected objects, each of which manages some piece of the overall program state. These objects then send messages to each other (i.e., invoke methods) to mutate the state held by each object. The inputs to the program define the initial state (e.g., a file to edit), and the final outputs are the mutated state that is often persisted somewhere (e.g., written back to a file).

In functional programming, we construct programs by combining small, reusable, "pure" functions that transform data. These pure functions have the following qualities:

- they operate only on their inputs, and make no reference to other data (e.g., variables at a higher scope)
- they never modify their inputs—instead, they always return new data or a reference to unmodified inputs
- they have no side effects outside of their outputs (e.g., they never modify variables at a higher scope)
- because of these previous rules, they always return the same outputs for the same inputs

A functional program sends its initial input state through a series of these pure functions, much like a plumbing system sends water through a series of pipes, filters, valves, splitters, heaters, coolers, and pumps. The outputs of the final function become the program's outputs, which are often passed on to another program or persisted.

Advocates of functional programming note that pure functions are easier to test and reason about. Since they have no side-effects, you can simply test all possible classes of inputs and verify that you get the correct outputs. If all of your pure functions are well-tested, you can then combine them together to create highly-predictable and reliable programs.

These advocates also argue that functional programs are easier to read and reason about because they end up looking more [declarative than imperative](../javascript/#secimperativevsdeclarative). A functional program reads like a series of data transformations, the output of each flowing into the next as input. This will become easier to see as I show examples in the next section.

Although some functional programming zealots would argue that all programs should be written in a functional style, it's better to think of functional programming as another tool in your toolbox that is appropriate for some jobs, and not so much for others. Object-oriented programming is often the better choice for long-running, highly-interactive client programs, while functional is a better choice for short-lived programs or servers that handle discrete transactions. It's also possible to combine the two styles: for example, React components can be either object-oriented or functional, and you often use some functional techniques within object-oriented components.

## Functional Programming in JavaScript

This is all a bit abstract so far, so let's see what this really looks like in practice. Functional programs operate on data, so we will use the following data file as input to our program. 

> <https://faculty.washington.edu/dlsinfo/data/babynames_2016.js>

The data includes counts for all distinct [baby names registered with the Social Security Administration (SSA) during 2016](https://www.ssa.gov/oact/babynames/limits.html). The file defines one constant named `BABYNAMES`, which is set to an array of 32,868 objects. Each object in the array has the following properties:

- `name`: a first name
- `sex`: a reported sex (the SSA allows only `M` or `F` for this field)
- `count`: the number of babies registered in 2016 with that name and reported sex

For privacy reasons, baby names with fewer than 5 registrations are omitted from this set. Also note that `sex` refers to biological sex, not gender, and the SSA limits responses to only male or female.

If you include this file in your web page, it will define this constant in the global scope, and your code can then reference it using the name `BABYNAMES`. For example the code `BABYNAMES.length;` will return the length of the array (32,868).

### Filtering

The first thing we might want to do with this array is extract just the objects that meet a particular criteria. For example, we might want to extract the objects where `sex === "M"` or `sex === "F"` into separate arrays so that we can process the male names separately from the female names. To do this, we first need a few functions that test whether the `sex` property of a given record is set to `"M"` or `"F"`.

```javascript
function isMale(record) {
	return record.sex === "M";
}

function isFemale(record) {
	return record.sex === "F";
}
```

Both of these are "pure" functions. They operate only on their inputs, they don't modify those inputs, and they don't have any side-effects. If you hand these functions the same object, they will always return the same results.

We can then use these functions with the built-in `.filter()` method that is available on all JavaScript arrays:

```javascript
let females = BABYNAMES.filter(isFemale);
females.length; // => 18757
let males = BABYNAMES.filter(isMale);
males.length; // => 14111
```

The `.filter()` method takes a function (known as a **predicate function**) as a parameter. It calls that function once for each element in the array, passing that element as the first parameter to the predicate function. If your predicate function returns a [truthy value](../javascript/#secbooleanexpressionsandtruthiness), the element will be included in the output array. If your predicate function returns a falsy value, the element won't be included.

To make this more clear, here is what the body of `.filter()` looks like:

```javascript
//assume `this` is the array upon which .filter() was called
//and `test` is the function passed to .filter() as the first parameter
let output = []; //the array that will be returned
//loop over all elements in the source array
for (let i = 0; i < this.length; i++) {
	let elem = this[i];
	//if the test functions returns something truthy...
	if (test(elem)) {
		//...add the element to the output array
		output.push(elem);
	}
}
//return the output array
return output;
```

As you can see, the `.filter()` method separates the task of iterating over the array from the task of testing whether each element should be included in the output array. The `.filter()` method supplies the first, but delegates the second to the predicate function you pass to it. That predicate function can be as complex as it needs to be, and the `.filter()` method doesn't have to care about its details.

Note that the `.filter()` method is also a pure function: it doesn't modify the array upon which it was called. Instead, it returns a new array containing only the objects that passed the predicate function.

You may have noticed that the two filter predicate functions above (`isMale()` and `isFemale()`) are very similar: the only real difference is what value they compare the `.sex` property to. Whenever you see something like this, you should ask yourself, "is there a way I can do this with just one function?" Indeed there is. Remember that if you declare a function inside another function, it creates a [closure](../javascript/#secclosures), which allows the inner function to reference parameters and local variables defined in the outer function. So we could write one function that takes the value to compare against as a parameter, and returns a new predicate function one could use with the `.filter()` method.

```javascript
//returns a filter predicate function that 
//compares the .sex property to the value 
//passed as the `sex` parameter
function isSex(sex) {
	//return a new filter test function that...
	return function(record) {
		//...compares the .sex property to the 
		//sex parameter value
		return record.sex === sex;
	};
}

//use isSex() to create separate 
//male/female filter predicate functions
let isMale = isSex("M");
let isFemale = isSex("F");

//use those filter functions
let females = BABYNAMES.filter(isFemale);
```

This code looks a bit more complicated than before, but now we've isolated the way we test the `.sex` property to just one function, and we don't have to duplicate that for each distinct sex. If we discover later on that the data contains both upper and lower-case letters for that field, we can make the adjustment to just one line of code and handle all of the cases. For example:

```javascript
//returns a filter predicate function that 
//compares the .sex property to the value 
//passed as the `sex` parameter
function isSex(sex) {
	//convert parameter to lower case for a 
	//case-insensitive comparison
	let sexLower = sex.toLowerCase();
	//return a new filter test function that...
	return function(record) {
		//...compares the lower-cased .sex property
		//to the lower-cased sex parameter value
		return record.sex.toLowerCase() === sexLower
	};
}
```

And if the SSA ever started allowing a value of `"U"` for unknown or unspecified, we could easily create a new filter predicate function without having to duplicate code.

### Combining Functions

So far we are filtering on only one property, but what if wanted to filter for male baby names that have a count under 100? We could write a filter predicate function specifically for that, but if we then want to the same thing for female baby names, we'd have to duplicate that with only a small change. Or we could embrace functional programming and realize that this predicate is a combination of two simpler predicates that are useful on their own: `isMale()` and `countUnder100()`. We could write one function that combines two existing predicate functions, regardless of what those predicate functions happen to test, using AND logic.

```javascript
//returns true if count is less than 100
function countUnder100(record) {
	return record.count < 100;
}

//returns a new filter predicate that combines
//the two predicate function parameters using &&
function and(predicate1, predicate2) {
	return function(record) {
		return predicate1(record) && predicate2(record);
	}
}

//create new predicate functions combining
//existing predicate functions with AND logic
let isMaleUnder100 = and(isMale, countUnder100);
let isFemaleUnder100 = and(isFemale, countUnder100);

//use them
let malesWithLowCounts = BABYNAMES.filter(isMaleUnder100);
```

Now you can create filter predicates that are combinations of _any_ two existing predicate functions, including predicates returned from the `and()` function. You could of course implement an `or()` function as well that used `||` instead of `&&`. You could also extend these to handle more than just two predicates at a time, but we need to learn a few other techniques before we can do that.

> **NOTE:** Although you can implement functions like this yourself, there are several functional programming JavaScript libraries that already implement generic function combiners like these. See the [Functional Programming Libraries section](#secfunctionalprogramminglibraries) at the end for links to these libraries.

Lastly, you might already be thinking that the `countUnder100()` function is too specific, and could easily be converted to a more generic `countUnder()` function that takes the upper threshold number as a parameter, and returns a new predicate function. You'd be right:

```javascript
function countUnder(amount) {
	return function(record) {
		return record.count < amount;
	}
}

let countUnder100 = countUnder(100);
let isMaleUnder100 = and(isMale, countUnder100);
```

This approach to combining functions is central to functional programming. It's like building with Legos: the smallest pieces do just one simple and general thing, and you then combine those together to create more specific and sophisticated structures.

### Sorting

Now that we have the ability to filter arrays, we next might want to sort those filtered sets. We already saw a bit of sorting in the [JavaScript Tutorial](../javascript/#secpassingfunctionstofunctions), but let's quickly review how that works. Every JavaScript array has a `.sort()` method, which takes a function (known as a **comparator**) that compares two of the array elements. The comparator function should return a negative number if the first element is less than the second, a zero if they are equal, and a positive number if the first is greater than the second. For example, if we wanted to sort an array of these baby name objects by their `count` property ascending, the comparator function would look like this:

```javascript
function byCount(record1, record2) {
	return record1.count - record2.count;
}
```

And a comparator that sorts by the `name` property would use the [`.localeCompare()` method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare), which is on every string:

```javascript
function byName(record1, record2) {
	return record1.name.localeCompare(record2.name);
}
```

Now that we have these two functions, we can sort any array of these baby names (filtered or not) by these properties. Remember that `.filter()` returns a new array, and since every array has a `.sort()` method, we can simply chain the call to `.sort()` on the end of the call to `.filter()`:

```javascript
//extract all the male names and sort by count ascending
let sortedMales = BABYNAMES.filter(isMale).sort(byCount);
//same thing for female names
let sortedFemales = BABYNAMES.filter(isFemale).sort(byCount);
```

> **WARNING:** the `.sort()` method isn't pure—it actually sorts the array in place and returns a reference to the original array. This is why you'll often see developers filter or map the array first to create a copy of the original array before sorting it. See the [Mapping section](#secmapping) below for an explanation of mapping.

But what if we want these sorted descending by count instead of ascending? You might be tempted to write more comparator functions, but the combinations would start to explode. Instead, let's embrace functional programming and realize that a descending sort requires a comparator that simply negates the return value of the ascending comparator: if the ascending comparator returns a negative number, our descending comparator needs to flip that to a positive number, and vice-versa. Thankfully `-0 === 0` in JavaScript, so negating a zero won't hurt anything. We can write this descending comparator as a function that takes another comparator as a parameter and returns a new comparator that negates the result:

```javascript
//`comparator` is a sort comparator function
function descending(comparator) {
	//return a new comparator that...
	return function(record1, record2) {
		//...negates the result of `comparator()`
		return -comparator(record1, record2);
	}
}
```

Now we can wrap any existing comparator with `descending()` to get a descending rather than ascending sort:

```javascript
//create a descending version of our byCount comparator
let byCountDescending = descending(byCount);
//extract all the male names and sort by count descending
let sortedMales = BABYNAMES.filter(isMale).sort(byCountDescending);
```

Notice how our program is starting to look more like a declarative series of high-level operations rather than an imperative series of low-level commands. The data flow through these high-level operations and the output of the last one becomes the output of our program.

Now what if we want to sort by name _within_ count? That is, we want the overall array ordered by `count`, but when there are a bunch of records that all have the same value for `count`, we want those ordered by `name`. This is known as a multi-key sort, and we can enable this with just one additional function:

```javascript
//`comparator1` and `comparator2` are both sort comparator functions
function multiKey(comparator1, comparator2) {
	//return a new comparator that...
	return function(record1, record2) {
		//...runs comparator1 and if the result is 0...
		let result = comparator1(record1, record2);
		if (result === 0) {
			//...returns the result of comparator2 instead
			return comparator2(record1, record2);
		} else {
			return result;
		}
	}
}
```

Like the `and()` function earlier, this function returns a new comparator that combines two existing comparators. It runs the first comparator and if the result is `0` (i.e., the two records are considered equal to each other), it returns the results of the second comparator instead. This will cause the array to be sorted by the first comparator overall, but then by the second comparator within common values for the first.

To sort males by name ascending _within_ count descending, the code would look like this:

```javascript
let byCountDescending = descending(byCount);
let byNameWithinCount = multiKey(byCountDescending, byName);
let malesByNameWithinCount = BABYNAMES.filter(isMale).sort(byNameWithinCount);
```

Of course, you could make this all one line without creating intermediate variables, but the readability would start to suffer. Remember that one of the goals of functional programming is to create code that is easier to read and reason about. If each statement solves only one piece of the overall puzzle, it's easy to see how the whole puzzle comes together.

### Slicing

After filtering and sorting, it's common to slice off only the top or bottom elements from the array. Every JavaScript array has a built-in [slice() method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) that does exactly this. For example, say we want just the top 10 female baby names. We already have all the filtering and sorting functions we need, so we just need to add a final `.slice()`:

```javascript
let top10Females = BABYNAMES.filter(isFemale)
	.sort(byCountDescending)
	.slice(0,10);
```

The `.slice()` method takes two parameters: the array index to start at, and the array index to go up to but not include. So `.slice(0,10)` will return a new array containing elements `0` through `9`, which are the first 10 elements in the array. Since we filtered for only female names and then sorted by count descending, those first 10 elements are the female names with the highest counts.

### Mapping

Another common operation that you will want to do on arrays of data is mapping. A mapping operation transforms each element of the array by passing that array element through a function you provide (known as a **transformer**). This is best explained through some concrete examples.

```javascript
function addOne(value) {
	return value + 1;
}

var evens = [0,2,4,6,8,10];
evens.map(addOne); // => [1,3,5,7,9,11]
```

The `.map()` method in this example will create a new array for the outputs that is the same size as the input array, call the `addOne()` function once for each element in the `events` array, and put the returned value into the output array at the same position. After mapping all of the elements, it returns the output array. The body of the `.map()` method looks something like this:

```javascript
//assume `this` is the array upon which .map() is called
//and `transform` is the function passed to .map()

//create an output array of the same length
let output = new Array(this.length);
for (let i = 0; i < this.length; i++) {
	//send each element through the transform
	//function and put the result into the output 
	//array at the same position
	output[i] = transform(this[i]);
}
return output;
```

It's actually pretty simple and straightforward. Each element is passed through the transformer function you pass to `.map()` and whatever that function returns is put into the output array at the same index.

This can be used to transform elements in all sorts of ways. For example, say we have an array of strings and we want to make them all lower-case:

```javascript
function toLower(str) {
	return str.toLowerCase();
}
let values = ["John","Mary","Peter"];
values.map(toLower); // => ["john","mary","peter"]
```

But we can also use this to transform elements in more substantial ways. For example, say we have an array of those baby name objects and we want to extract just the `name` property from each. In other words, we want to end up with an array of strings instead of an array of objects. We can do that with `.map()` as well.

```javascript
//returns just the `name` property
function pluckName(record) {
	return record.name;
}

let top10FemaleNames = BABYNAMES.filter(isFemale)
	.sort(byCountDescending)
	.slice(0,10)
	.map(pluckName);
```

Because we ran each element of the filtered, sorted, and sliced array through the `pluckName()` function, the `top10FemaleNames` variable will be set to an _array of strings_ containing just the top 10 female baby names. We transformed entire objects into just strings using the `pluckName()` function. If you print `top10FemaleNames` to the console, you'll get this:

```json
["Emma", "Olivia", "Ava", "Sophia", "Isabella", "Mia", "Charlotte", "Abigail", "Emily", "Harper"]
```

Those were the top 10 female baby names registered with the SSA during 2016!

As you might expect, we can make `pluckName()` more generic by creating a function that returns a transformer that plucks _any_ property from an object given a property name.

```javascript
//`propName` is a string
function pluck(propName) {
	//return a transformer that...
	return function(record) {
		//...returns just the requested property
		return record[propName];
	}
}

//now we can pluck any property by name!
let pluckName = pluck("name");
let pluckCount = pluck("count");
```

Transformer functions can transform their inputs in any way they want. In fact, a transformer could return an entirely new object, or even a function that makes use of the input. This makes mapping a very powerful technique.

### Reducing

The last functional technique I will discuss in this tutorial is also the hardest to understand: [reducing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). While mapping transforms each element into an array of the same size as the input array, reducing reduces the elements of an array to a single value. This value could be a primitive value (number, string, boolean) or it could be a complex type like an object, a smaller array, or even a function.

A simple example will illustrate what I mean:

```javascript
//returns the sum of `accumulator` and `num`
function sum(accumulator, num) {
	return accumulator + num;
}

//create an array of a few numbers
let nums = [1,2,3,4,5];

//calculate the sum of all the numbers
nums.reduce(sum, 0); // => 15
```

The `.reduce()` method takes two parameters: a function, known as a **reducer**, and a starting value, known as the **accumulator**. The `.reduce()` method calls the reducer once for each element in the array, passing the current value of the accumulator as the first parameter, and the current array element as the second parameter. It resets the accumulator to whatever the reducer function returns, an continues processing the next element in the array.

The body of `.reduce()` would look something like this:

```javascript
//assume `this` is the array upon which .reduce() is called
//and `reducer` is the function passed to .reduce() as the first parameter
//and `accumulator` is the initial value passed as the second parameter
for (let i = 0; i < this.length; i++) {
	accumulator = reducer(accumulator, this[i]);
}
return accumulator;
```

Note how simple, yet elegant this is. Your reducer gets the current accumulator value and the current element. Whatever your reducer returns becomes the new accumulator value. At the end, it returns the final accumulator.

Summing numbers is only one example of how you might use `.reduce()`. Finding the minimum or maximum value in the array is another classic application of this method. We could sort the array and then slice off the first or last element, but it would be much more efficient to use reduce to do only one pass through the array:

```javascript
function max(n1, n2) {
	//ternary conditional--see Intro to JavaScript tutorial for details
	return n2 > n1 ? n2 : n1;
}

nums.reduce(max, nums[0]);
```

Here we use our `max()` function as the reducer, and the first element in the array as the starting accumulator. Each time `.reduce()` calls our `max()` function, it passes the current accumulator value and the current element. Our function returns the greater of the two, which becomes the current accumulator. When it's all done, the accumulator is set to the maximum value.

The term "accumulator" is intentionally vague. This value can be of any type, and you can use it however you want in your reducer functions. For example, the accumulator could be a JavaScript object where the keys are all the distinct strings in an array, and the values associated with those keys are the number of times that distinct string appears in the array. For example:

```javascript
//`nameMap` is a JavaScript object
//`name` is a string array element
function countDistinct(nameMap, name) {
	//if the object doesn't have a key for
	//the name yet, add one with a value of 0
    if (!nameMap.hasOwnProperty(name)) {
        nameMap[name] = 0;
    }
    //increment the count associated with this name
    nameMap[name]++;
    //return the map
    return nameMap;
}

let names = ["Dave", "Mary", "John", "Dave", "Mary"];
names.reduce(countDistinct, {}); // => {Dave: 2, Mary: 2, John: 1}
```

The initial accumulator value we pass here is an empty object `{}`. This gets passed into our `countDistinct()` method as the first parameter, which we called `nameMap` (you can call it whatever you want, it's your parameter after all). If the name map doesn't contain a key matching the current array element, we add it with the value of `0`. We then increment the value associated with that key and return the map as the next accumulator. By the end, we get a JavaScript object with a key for each distinct string in the array, and the value associated with each of those keys is the number of times that string was in the array.

## Functional Programming Libraries

This tutorial has introduced you to a few of the functional programming methods that are built-in to all JavaScript arrays, but these methods only scratch the surface of what a full functional programming library provides. If you're interested in doing more serious functional programming, check out these libraries.

- [Lodash](https://lodash.com/) and the more pure variant [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
- [Ramda](http://ramdajs.com/)
- [Lazy.js](http://danieltao.com/lazy.js/)

Finally, I would remiss if I didn't acknowledge that [not everyone thinks functional programming in JavaScript is a good idea](https://hackernoon.com/functional-programming-in-javascript-is-an-antipattern-58526819f21e). Although JavaScript was based more on Scheme than Java, there are a lot of elements in the language where they break the functional paradigm (e.g., `.sort()` not being pure). Many client-side developers who have gotten serious about functional programming have turned to other languages that were designed as functional languages from the start, but can still compile to JavaScript. Here are some of the currently-popular choices:

- [Clojure](https://clojure.org/) and the [ClojureScript compiler for JavaScript](https://github.com/clojure/clojurescript)
- [Haskell](https://www.haskell.org/) and one of the several [Haskell to JS compilers](https://wiki.haskell.org/The_JavaScript_Problem)
- [Elm](http://elm-lang.org/), which has a built-in compiler to JavaScript
 
