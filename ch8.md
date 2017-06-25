# JavaScript 轻量级函数式编程
# Chapter 8: List Operations
# 第 8 章：列表操作

Did you have fun down our little closures/objects rabbit hole in the previous chapter? Welcome back!
在上一章中对 闭包／对象 玩的还开心吗？欢迎回来！
> If you can do something awesome, keep doing it repeatedly.
> 如果你能把一件事情做的非常好，那就坚持做下去。
We've already seen several brief references earlier in the text to some utilities that we now want to take a very close look at, namely `map(..)`, `filter(..)`, and `reduce(..)`. In JavaScript, these utilities are typically used as methods on the array (aka, "list") prototype, so we would naturally refer to them as array or list operations.
我们在之前的文章中就已经看到了一些简短的参考，现在，我们要来仔细的看一下这些实用函数，即 `map(..)`, `filter(..)`, 和 `reduce(..)`。在 JavaScript 中，这些实用函数通常被应用在数组（又称为"list"）的原型上，所以，我们称他们为数组或者列表操作。

Before we talk about the specific array methods, we want to examine conceptually what these operations are used for. It's equally important in this chapter that you understand *why* list operations are important as it is to understand *how* list operations work. Make sure you approach this chapter with that detail in mind. 

在我们谈论特定的数组方法之前，我们想要从概念上检查这些操作的用途。在本章中和它同样重要的是，明白 *为什么* 列表操作是重要的，因为它可以让我们理解列表操作是 *如何* 工作的。请确保在这一章中你注意到了这些细节。

The vast majority of common illustrations of these operations, both outside of this book and here in this chapter, depict trivial tasks performed on lists of values (like doubling each number in an array); it's a cheap and easy way to get the point across.

无论是本书外部还是本章中的关于这些操作的大量插图，都描述了在值列表上操作的微小任务（就像将数组的每个数字加倍）；这是一个获得关键点的简单的方法。

But don't just gloss over these simple examples and miss the deeper point. Some of the most important FP value in understanding list operations comes from being able to model a sequence of tasks -- a series of statements that wouldn't otherwise *look* like a list -- as a list operation instead of performing them individually.

但是不要无视这些简单的例子而忽略深层次的要点。 FP 更重要的一些价值在于理解列表操作，这些列表操作来源于可以构建一系列任务 —— 这一系列声明 *看起来* 不像列表 —— 就如同列表操作不是单个执行的。

This isn't just a trick to write more terse code. What we're after is to move from imperative to declarative style, to make the code patterns more readily recognizable and thus more readable.

这不仅仅是编写更简洁的代码的技巧。我们追求的是从命令式转换为声明式风格，使代码模式更简明，从而更有可读性。

But there's something **even more important to grasp**. With imperative code, each intermediate result in a set of calculations is stored in variable(s) through assignment. The more of these imperative patterns your code relies on, the harder it is to verify that there aren't mistakes -- in the logic, accidental mutation of values, or hidden side causes/effects lurking.

但是有些东西要 **更加重要的去理解** 。 使用命令式代码，一组计算的每个中间结果都通过赋值给变量进行存储。你的代码越依赖这些命令式模式，就越难去验证它们的错误 —— 逻辑上说，是值的意外变化或者是隐藏的副作用／潜在的影响。

By chaining and/or composing list operations together, the intermediate results are tracked implicitly and largely protected from these hazards.

通过链式组合列表操作，中间结果会被隐式的跟踪，这在很大程度上避免了风险。

**Note:** More than previous chapters, to keep the many following code snippets as brief as possible, we'll rely heavily on the ES6 `=>` form. However, my advice on `=>` from Chapter 2 still applies for general coding.

**注意：** 不只是前几章，尽可能简短的保留代码段，我们将会依赖 ES6 的 `=>` 语法。然而，我建议第 2 章的 `=>` 仍然适用于大部分编码。

## Non-FP List Processing

## 非 FP 列表处理

As a quick preamble to our discussion in this chapter, I want to call out a few operations which may seem related to JavaScript arrays and FP list operations, but which aren't. These operations will not be covered here, because they are not consistent with general FP best practices:

作为在本章中讨论的一个简短的序言，我想要调用一些可能与 JavaScript 数组和 FP 列表操作有关的操作，但是并不能这样。这些操作不会被包含在里面，因为它们和一般的 FP 最佳实践不一致：

* `forEach(..)`
* `some(..)`
* `every(..)`

`forEach(..)` is an iteration helper, but it's designed for each function call to operate with side effects; you can probably guess why that's not an endorsed FP list operation for our discussion!

`forEach(..)` 是一个辅助迭代的方法，但是它是为每个调用有副作用的函数设计的；你或许会想为什么在我们的讨论中它不是一个被支持的 FP 列表操作。

`some(..)` and `every(..)` do encourage the use of pure functions (specifically, predicate functions like `filter(..)` does), but they inevitably reduce a list to a `true` / `false` result, essentially like a search or matching. These two utilities don't really fit the mold of how we want to model our code with FP, so we're going to skip covering them here.

`some(..)` 和 `every(..)` 鼓励使用纯函数（特别是像 `filter(..)` 这样的谓词函数），但是他们不可避免的 reduce 一个列表并返回 `true` / `false`，本质上就像搜索或者匹配。这两个实用函数并不真的适合用 FP 来构建代码的模式，所以我们将略过这个内容。

## Map

We'll start our exploration of FP list operations with one of the most basic and fundamental: `map(..)`.

我们将开始 FP 列表操作的探索，其中最基础的就是： `map(..)` 。

A mapping is a transformation from one value to another value. For example, if you start with the number `2` and you multiply it by `3`, you have mapped it to `6`. It's important to note that we're not talking about mapping transformation as implying *in-place* mutation or reassignment; rather mapping transformation projects a new value from one location to the other.

映射是把一个值转换成另一个值。例如，如果你用 `2` 乘以 `3` ，就可以映射出 `6` 。需要注意的是我们不是在讨论把映射转换暗示为 *in-place* 转换或者重新赋值；而是把原值转换成另一个新的值。

In other words:

换句话说：

```js
var x = 2, y;

// transformation / projection
// 转换 / 变换
y = x * 3;

// mutation / reassignment
// 转换 / 重新赋值
x = x * 3;
```

If we define a function for this multiplying by `3`, that function acts as a mapping (transformer) function:

如果我们定义一个函数，它的功能是一个数乘以 `3` ，那么这个函数就可以做为映射（转换器）函数：

```js
var multipleBy3 = v => v * 3;

var x = 2, y;

// transformation / projection
// 转换 / 变换
y = multiplyBy3( x );
```

We can naturally extend mapping from a single value transformation to a collection of values. `map(..)` is an operation that transforms all the values of a list as it projects them to a new list:

我们能够自然地将单一值转换成值的集合。 `map(..)` 是将一个列表的所有值转变成一个新的列表的操作。

<p align="center">
	<img src="fig9.png" width="400">
</p>

实现 `map(..)`:

```js
function map(mapperFn,arr) {
	var newList = [];

	for (let idx = 0; idx < arr.length; idx++) {
		newList.push(
			mapperFn( arr[idx], idx, arr )
		);
	}

	return newList;
}
```

**Note:** The parameter order `mapperFn, arr` may feel backwards at first, but this convention is much more common in FP libraries because it makes these utilities easier to compose (with currying).

**注意：**  `mapperFn, arr` 这种参数的顺序我们刚开始会感觉这是反向的，但是这种约定在 FP 的函数库中是很常见的，因为它使实用函数更容易（和柯里化）组合。

The `mapperFn(..)` is naturally passed the list item to map/transform, but also an `idx` and `arr`. We're doing that to keep consistency with the built-in array `map(..)`. These extra pieces of information can be very useful in some cases.

`mapperFn(..)` 不仅自然的传递列表项到映射或者转换(todo)，而且也有 `idx` 和 `arr` 。我们这样做是为了与内置的数组 `map(..)` 保持一致性。这些额外的信息在一些情况下是非常有用的。

But in other cases, you may want to use a `mapperFn(..)` that only the list item should be passed to, because the extra arguments might change its behavior. In "All For One" in Chapter 3, we introduced `unary(..)`, which limits a function to only accept a single argument (no matter how many are passed).

但是另外一种情况，你可能想要用 `mapperFn(..)` ，这仅仅是一个列表项应该传递的，因为这个额外参数可能会改变它的行为。在第 3 章的 "All For One"(todo 名称未定) 我们引入了 `unary(..)` ，它限制了一个函数仅仅接受一个参数(不管有多少会被传递过来)。

Recall the example from Chapter 3 about limiting `parseInt(..)` to a single argument to be used safely as a `mapperFn(..)`:

回顾第三章的例子，我们讲到通过限制 `parseInt` 函数接受单一参数，并将其作为安全的 `mapperFn(..)` 函数：

```js
map( ["1","2","3"], unary( parseInt ) );
// [1,2,3]
```

JavaScript provides the `map(..)` utility built-in on arrays, making it very convenient to use as part of a chain of operations on a list.

JavaScript 提供了 `map(..)` 实用函数内置数组，使它非常方便的作为链式操作的一部分来使用。

**Note:** The JavaScript array prototype operations (`map(..)`, `filter(..)`, and `reduce(..)`) all accept an optional last argument to use for `this` binding of the function. As we discussed in "What's This?" in Chapter 2, `this`-based coding should generally be avoided wherever possible in terms of being consistent with the best practices of FP. As such, our example implementations in this chapter do not support such a `this`-binding feature.

**注意：** JavaScript 数组原型操作（`map(..)`, `filter(..)` 和 `reduce(..)`），它们接受的最后一个参数都是可选的，是用 `this` 来和这个函数绑定。正如我们在第 2 章中讨论的 “来说说 This ？” 这一部分， `this` —— 根据 FP 一贯的最佳实践，这种基础编码通常应该尽可能地避免。同样地，本章中的实例项目不支持像 `this` 这样的绑定特性。

Beyond the obvious numeric or string operations you could perform against a list of those respective value types, here's some other examples of mapping operations. We can use `map(..)` to transform a list of functions into a list of their return values:

除了这些明显的数值型或者字符串操作之外，你还能够对它们各自的值类型的列表执行，这是其他的一些映射操作的例子。我们可以使用 `map(..)` 把函数列表转换成返回值列表：

```js
var one = () => 1;
var two = () => 2;
var three = () => 3;

[one,two,three].map( fn => fn() );
// [1,2,3]
```

Or we can first transform a list of functions by composing each of them with another function, and then execute them:

或者我们先转换函数的列表，把它们组成另一个函数，然后执行：
```js
var increment = v => ++v;
var decrement = v => --v;
var square = v => v * v;

var double = v => v * 2;

[increment,decrement,square]
.map( fn => compose( fn, double ) )
.map( fn => fn( 3 ) );
// [7,5,36]
```

Something interesting to observe about `map(..)`: we typically would assume that the list is processed left-to-right, but there's nothing about the concept of `map(..)` that really requires that. Each transformation is supposed to be independent of every other transformation.

一些关于  `map(..)` 的有趣的观察：我们通常会猜想列表是从左到右被处理的，但是这并没有我们真正需要的关于 `map(..)` 的概念。每一个转换都应该独立于其他的转变。

Mapping in a general sense could even been parallelized in an environment that supports that, which for a large list could drastically improve performance. We don't see JavaScript actually doing that because there's nothing that requires you to pass a pure function as `mapperFn(..)`, even though you **really ought to**. If you were to pass an impure function and JS were to run different calls in different orders, it would quickly cause havoc.

一般来说，映射能够在支持的环境中并行，对于一个重的列表来说能够极大的提升性能。实际上，我们并没有看到 JavaScript 做了什么，那是因为你不需要通过纯粹的函数 `mapperFn(..)` ，即使你真的 **真的应该** 。如果你通过一个不纯的函数和 js 在不同的命令下运行不同的回调，将要迅速的遭到严重的破坏。

Even though theoretically, individual mapping operations are independent, JS has to assume that they're not. That's a bummer.

即使从理论上讲，单独的映射操作是独立的，但是 JS 不得不认为并不是这样的。真是无赖。

### Sync vs Async

### 同步 vs 异步

The list operations we're discussing in this chapter all operate synchronously on a list of values that are all already present; `map(..)` as conceived here is an eager operation. But another way of thinking about the mapper function is as an event handler which is invoked for each new value encountered in the list.

我们在本章讨论的列表操作都在已经存在的值列表中同步执行；`map(..)` 和设想的一样是一个立刻执行的操作。但是另一种思考 mapper 函数的方式是作为事件句柄，它被列表中的每个新值调用。

Imagine something fictional like this:

想象一下像这样的虚构：

```js
var newArr = arr.map();

arr.addEventListener( "value", multiplyBy3 );
```

Now, any time a value is added to `arr`, the `multiplyBy3(..)` event handler -- mapper function -- is called with the value, and its transformation is added to `newArr`.

现在，一个值被随时添加到 `arr`，`multiplyBy3(..)` 事件具柄 —— mapper 函数 —— 用这个值调用，然后被它处理过的值被添加到了 `newArr` 中。 

What we're hinting at is that arrays, and the array operations we perform on them, are the eager synchronous versions, whereas these same operations can also be modeled on a "lazy list" (aka, stream) that receives its values over time. We'll dive into this topic in Chapter 10.

我们暗示的是数组，对它们执行的数组操作是热更新的版本，然而这些相同的操作仿照“懒列表”（又称：流）随着时间的推移接收值。然而这些相同的操作被称为“懒列表”接收他的值。我们将会在第 10 章深入探讨这个话题。

### Mapping vs Eaching

Some advocate using `map(..)` as a general form of `forEach(..)`-iteration, where essentially the value received is passed through untouched, but then some side-effect can be performed:

有些人提倡使用 `map(..)` 作为 `forEach(..)` —— 迭代 的通用式，从本质上说，接收的值被处理时是未受影响的，但是仍然会带有一些副作用：

```js
[1,2,3,4,5]
.map( function mapperFn(v){
	console.log( v );			// 副作用！
	return v;
} )
..
```

The reason this technique can seem useful is that the `map(..)` returns the array so you can keep chaining more operations after it; the return value of `forEach(..)` is `undefined`. However, I think you should avoid using `map(..)` in this way, because it's a net confusion to use a core FP operation in a decidedly un-FP way.

这个技术看起来有用的原因是 `map(..)` 返回的是数组，所以你可以在后面添加更多的链式操作；`forEach(..)`的返回值是 `undefined`。然而我认为你应该避免使用 `map(..)` 这个方法，因为在一个明显的非 FP 方法上使用 FP 的核心操作是非常混乱的。

You've heard the old addage about using the right tool for the right job, right? Hammer for a nail, screwdriver for a screw, etc. This is slightly different: it's use the right tool *in the right way*.

听过一个老的谚语，关于使用正确的工具做正确的事情，对吧？用锤子钉钉，用螺丝刀拧螺丝，等等。这里稍微不同的是：以 *正确的方式* 使用正确的工具。

A hammer is meant to be swung in your hand; if you instead hold it in your mouth and try to hammer the nail, you're not gonna be very effective. `map(..)` is intended to map values, not create side effects.

锤子在你手中摇摆；如果你把它放在嘴里，尝试去敲钉子，很可能就不会有什么效果。 `map(..)` 主要是 map 值，而不是产生副作用。

### A Word: Functors
### 一个单词：函子
We've mostly tried to stay away from artificial invented terminology in FP as much as possible in this book. We have used official terms at times, but mostly when we can derive some sense of meaning from them in regular everyday conversation.

在本书中我们尽可能的避免使用 FP 中的人工术语。我们偶尔使用官方术语，但是从他们日常的交谈中我们可以获得一些有意义的思想。

I'm going to very briefly break that pattern and use a word that might be a little intimidating: functor. The reason I want to talk about functors here is because we now already understand what they do, and because that term is used heavily throughout the rest of FP literature; you being at least familiar with  and not scared by it will be beneficial.

我打算非常简短的打破这种模式，使用一个可能有些吓人的单词： functor。我在这里讨论函子的原因是因为我们已经明白了它们是做什么的了，因为这个术语在剩下的 FP 文章中被大量的使用；你至少是熟悉而且不害怕它的。

A functor is a value that has a utility for using an operator function on that value.

函子是一个值，它具有对该值使用运算符函数的实用函数。

If the value in question is compound, meaning it's comprised of individual values -- as is the case with arrays, for example! -- a functor uses the operator function on each individual value. Moreover, the functor utility creates a new compound value holding the results of all the individual operator function calls.

如果讨论的这个值是合成的就意味着它是由独立的值组成的 —— 例如数组！函子对每个独立的值使用运算符函数。此外，函子实用函数创建了一个新的合成值，保存了所有调用独立的运算符函数的结果。

This is all a fancy way of describing what we just looked at with `map(..)`. The `map(..)` function takes its associated value (an array) and a mapping function (the operator function), and executes the mapping function for each individual value in the array. Finally, it returns a new array with all the newly mapped values in it.

这是我们刚刚看到的 `map(..)` 的一种特殊的方式。`map(..)` 函数使值（一个数组）和 mapping 函数（运算符函数）相关联，并对数组的每个独立的值执行 mapping 函数。最后，返回一个新的数组，它包含所有新映射的值。

Another example: a string functor would be a string plus a utility that executes some operator function across all the characters in the string, returning a new string with the processed letters. Consider this highly-contrived example:

另一个例子：字符串函子是将一个字符串加上一个实用函数，并且对字符串中的所有字符执行一些运算符函数，返回一个由处理过的字符组成的新的字符串。 字符串中的所有人物，返回一个新的字符串处理信件。思考一下这个被高度设计的例子：

```js
function uppercaseLetter(c) {
	var code = c.charCodeAt( 0 );

	// 小写字符？
	if (code >= 97 && code <= 122) {
		// 转换成大写！
		code = code - 32;
	}

	return String.fromCharCode( code );
}

function stringMap(mapperFn,str) {
	return [...str].map( mapperFn ).join( "" );
}

stringMap( uppercaseLetter, "Hello World!" );
// HELLO WORLD!
```

`stringMap(..)` allows a string to be a functor. You can define a mapping function for any data structure; as long as the utility follows these rules, the data structure is a functor.

`stringMap(..)` 允许字符串成为函子。你可以为任何构造函数定义 mapping 函数；只要实用函数遵循这些规则，数据结构就是一个函子。

## Filter

Imagine I bring an empty basket with me to the grocery store to visit the fruit section; there's a big display of fruit (apples, oranges, and bananas). I'm really hungry so I want to get as much fruit as they have available, but I really only prefer the round fruits (apples and oranges). So I sift through each fruit one-by-one, and I walk away with a basket full of just the apples and oranges.

想象我带着空篮子去到食品杂货店参观水果区域；那儿有很多水果（苹果、橘子和香蕉）。我真的很饿所以我想要得到尽可能多的水果，但是我确实更喜欢圆形的水果（苹果和橘子），所以我就一个个的筛选每个水果，最后带着一篮子的苹果和橘子离开了。

Let's say we call this process *filtering*. Would you more naturally describe my shopping as starting with an empty basket and **filtering in** (selecting, including) only the apples and oranges, or starting with the full display of fruits and **filtering out** (skipping, excluding) the bananas as my basket is filled with fruit?

我们称这次过程为 *filtering*。当我的篮子装满水果的时候，你会自然地描述我的购物是以空篮子开始，只 **过滤** （选择，包括）苹果和橘子，或者以所有展示的水果开始，**过滤掉**（跳过，排除）香蕉？

If you cook spaghetti in a pot of water, and then pour it into a strainer (aka filter) over the sink, are you filtering in the spaghetti or filtering out the water? If you put coffee grounds into a filter and make a cup of coffee, did you filter in the coffee into your cup, or filter out the coffee grounds?

如果你在一壶水里煮意大利面，然后将其倒入水槽的过滤器（又称 filter ）中，那么你是过滤出意大利面还是过滤掉水呢？如果你把咖啡渣放入过滤器中，然后做出一杯咖啡，那么你是把咖啡过滤到你的杯子里，还是过滤掉咖啡渣呢？

Does your view of filtering depend on whether the stuff you want is "kept" in the filter or passes through the filter?

你的过滤视图取决于你想要的内容是“保留”在过滤器中的还是被过滤器滤出的？

What about on airline / hotel websites, when you specify options to "filter your results"? Are you filtering in the results that match your criteria, or are you filtering out everything that doesn't match? Think carefully: this example might have a different semantic than the previous ones.

当你在航空公司／酒店的网站上指定选项“筛选结果”时，你是在符合条件的结果中进行过滤，还是过滤掉不匹配的所有内容？仔细想想：这个例子可能与之前的意思不同。

Depending on your perspective, filter is either exclusionary or inclusionary. This conceptual conflation is unfortunate.

根据你的观点，filter 要么是排他的，要么是包容的。这种概念上的合并是不成功的。

I think the most common interpretation of filtering -- outside of programming, anyway -- is that you filter out unwanted stuff. Unfortunately, in programming, we have essentially flipped this semantic to be more like filtering in wanted stuff.

我认为关于 filter 最通用的解释是 -- 在编程之外，就是过滤掉了不想要的东西。但是，在编程里，这个语义有了实质性的反转，更像是在想要的东西里进行过滤。

The `filter(..)` list operation takes a function to decide if each value in the original array should be in the new array or not. This function needs to return `true` if a value should make it, and `false` if it should be skipped. A function that returns `true` / `false` for this kind of decision making goes by the special name: predicate function.

`filter(..)` 列表操作使用一个函数来确定原数组中的每个值是否应该出现在新数组中。如果出现，这个函数就需要返回 `true`，如果不出现，就需要返回 `false`。返回 `true` / `false`的函数有一个特殊的命名：谓词函数。

If you think of `true` as being as a positive signal, the definition of `filter(..)` is that you are saying "keep" (to filter in) a value rather than saying "discard" (to filter out) a value.

如果你认为 `true` 是一个肯定的标志，那么 `filter(..)` 的定义就是所说的“保留”（过滤出）而不是“丢弃”（过滤掉）一个值。

To use `filter(..)` as an exclusionary action, you have to twist your brain to think of positively signaling an exclusion by returning `false`, and passively letting a value pass through by returning `true`.

使用 `filter(..)` 这个排除性的方法的时候，你必须扭转自己的思维，想到通过返回 `false` 来暗示这是一个排除的信号，通过返回 `true` 来传递一个值。

The reason this semantic mismatch matters is because of how you will likely name the function used as `predicateFn(..)`, and what that means for the readability of code. We'll come back to this point shortly.

语意不匹配的原因是因为你可能命名这个函数为 `predicateFn(..)`以及这对于代码的可读性意味着什么。我们很快就会讲到这一点。

Here's how to visualize a `filter(..)` operation across a list of values:

以下是如何通过值列表来可视化 `filter(..)` 操作

<p align="center">
	<img src="fig10.png" width="400">
</p>

To implement `filter(..)`:

执行 `filter(..)`：

```js
function filter(predicateFn,arr) {
	var newList = [];

	for (let idx = 0; idx < arr.length; idx++) {
		if (predicateFn( arr[idx], idx, arr )) {
			newList.push( arr[idx] );
		}
	}

	return newList;
}
```

Notice that just like `mapperFn(..)` before, `predicateFn(..)` is passed not only the value but also the `idx` and `arr`. Use `unary(..)` to limit its arguments as necessary.

要注意的是在 `mapperFn(..)` 之前， `predicateFn(..)` 不仅传值还传了 `idx` 和 `arr`。这时就要根据需要来使用 `unary(..)` 限制其参数。

Just as with `map(..)`, `filter(..)` is provided as a built-in utility on JS arrays.

就像 `map(..)` 一样，在 JS 的数组里 `filter(..)` 是作为一个内置实用函数被提供的。

Let's consider a predicate function like this:

让我们来看一个像这样的谓词函数：

```js
var whatToCallIt = v => v % 2 == 1;
```

This function uses `v % 2 == 1` to return `true` or `false`. The effect here is that an odd number will return `true`, and an even number will return `false`. So, what should we call this function? A natural name might be:

该函数使用 `v % 2 == 1` 来返回一个 `true` 或者 `false`。效果是奇数会返回 `true`，偶数会返回 `false`。所以，我们应该如何来称呼这个函数呢？自然的命名可能是：

```js
var isOdd = v => v % 2 == 1;
```

Consider how you might use `isOdd(..)` with a simple value check somewhere in your code:

思考一下如何使用 `isOdd(..)` 在代码的某个地方做一个简单的值检查：

```js
var midIdx;

if (isOdd( list.length )) {
	midIdx = (list.length + 1) / 2;
}
else {
	midIdx = list.length / 2;
}
```

Makes sense, right? But, let's consider using it with the built-in array `filter(..)` to filter a list of values:

没毛病，对吧？但是，我们来使用内置数组 `filter(..)` 来筛选值列表：

```js
[1,2,3,4,5].filter( isOdd );
// [1,3,5]
```

If you described the `[1,3,5]` result, would you say, "I filtered out the even numbers", or would you say "I filtered in the odd numbers"? I think the former is a more natural way of describing it. But the code reads the opposite. The code reads, almost literally, that we "filtered (in) each number that is odd".

如果描述结果 `[1,3,5]`，你可能会说“我过滤掉了偶数”，或者会说“我过滤出了奇数”？我认为前者的描述方式更自然。但是代码却是相反的。代码几乎每个字母都在表达我们“过滤出了每个奇数”。

I personally find this semantic confusing. There's no question there's plenty of precedent for experienced developers. But if you just start with a fresh slate, this expression of the logic seems kinda like not speaking without a double negative -- aka, speaking with a double negative.

我个人认为这个语义是混乱的。毫无疑问，对于有经验的开发者来说有很多先例。但是如果你是从一个全新的平板开始，这个逻辑的表达不像说话那样没有双重否定 —— 也就是说，是一个双重否定。

We could make this easier by renaming the function from `isOdd(..)` to `isEven(..)`:

我们通过将函数 `isOdd(..)` 重命名为 `isEven(..)` 来简化这个操作：

```js
var isEven = v => v % 2 == 1;

[1,2,3,4,5].filter( isEven );
// [1,3,5]
```

Yay! But that function makes no sense with its name, in that it returns `false` when it's even:

很棒！但是这个函数的名称是没有任何意义的，因为当传入偶数是它会返回 `false` ：

```js
isEven( 2 );		// false
```

Yuck.

讨厌！

Recall that in "No Points" in Chapter 3, we defined a `not(..)` operator that negates a predicate function. Consider:

回想一下第 3 章的 "No Points" ，我们定义了一个否定谓词函数的 `not(..)` 运算符。

```js
var isEven = not( isOdd );

isEven( 2 );		// true
```

But we can't use *this* `isEven(..)` with `filter(..)` the way it's currently defined, because our logic will be reversed; we'll end up with evens, not odds. We'd need to do:

但是我们不能使用 *this* `isEven(..)` 来 `filter(..)` 当前定义的方式，因为我们的逻辑将会被逆转；最终将会结得到偶数，而不是奇数。我们需要这样做：

```js
[1,2,3,4,5].filter( not( isEven ) );
// [1,3,5]
```

That defeats the whole purpose, though, so let's not do that. We're just going in circles.

然而，这违背了我们的初衷，所以不要不要这样做。我们只是在兜圈子。

### Filtering-Out & Filtering-In

To clear up all this confusion, let's define a `filterOut(..)` that actually **filters out** values by internally negating the predicate check. While we're at it, we'll alias `filterIn(..)` to the existing `filter(..)`:

为了消除这些混乱，我们来定义一个 `filterOut(..)`，就是通过内部取消谓词检查来 **过滤掉** 值。同时，对现存的 `filter(..)`取一个别名 `filterIn(..)`：

```js
var filterIn = filter;

function filterOut(predicateFn,arr) {
	return filterIn( not( predicateFn ), arr );
}
```

Now we can use whichever filtering makes most sense at any point in our code:

现在我们可以在代码中使用任意的过滤器：

```js
isOdd( 3 );								// true
isEven( 2 );							// true

filterIn( isOdd, [1,2,3,4,5] );			// [1,3,5]
filterOut( isEven, [1,2,3,4,5] );		// [1,3,5]
```

I think using `filterIn(..)` and `filterOut(..)` (known as `reject(..)` in Ramda) will make your code a lot more readable than just using `filter(..)` and leaving the semantics conflated and confusing for the reader.

我认为比起使用 `filter(..)` 使读者对语义困惑来说，使用 `filterIn(..)` 和 `filterOut(..)`（在 Ramda 中被称为 `reject(..)`）会使代码的可读性更高。

## Reduce

While `map(..)` and `filter(..)` produce new lists, typically this third operator (`reduce(..)`) combines (aka "reduces") the values of a list down to a single finite (non-list) value, like a number or string. However, later in this chapter, we'll look at how you can push `reduce(..)` to use it in more advanced ways. `reduce(..)` is one of the most important FP tools; it's like a swiss army all-in-one knife with all its usefulness.

当 `map(..)` 和 `filter(..)` 产生新的列表时，第三个运算符（`reduce(..)`）把列表的值组合成（又称 “reduces”）一个单一有限（非列表）的值，就像数字或者字符串。然而，在本章的后面，将会看到用更炫酷的方式来使用 `reduce(..)`。`reduce(..)` 是最重要的 FP 工具之一；它就像瑞士军刀一样非常有用。

A combination/reduction is abstractly defined as taking two values and making them into one value. Some FP contexts refer to this as "folding", as if you're folding two values together into on value. That's a helpful visualization, I think.

combination/reduction 被抽象的定义为将两个值合并成一个值。在 FP 中被称之为 "folding"，就像你将两个值折叠成一个值一样。我认为这是一个非常帮助的可视化方法。

Just like with mapping and filtering, the manner of the combination is entirely up to you, and generally dependent on the types of values in the list. For example, numbers will typically be combined through arithmetic, strings through concatenation, and functions through composition.

就像 mapping 和 filtering 一样，组合的方式完全取决于你，它通常依赖于列表中值的类型。例如，数字通过算数来结合，字符串通过连接来结合，函数通过合成来组合。

Sometimes a reduction will specify an `initialValue` and start its work by combining it with the first value in the list, cascading down through each of the rest of the values in the list. That looks like this:

有时 reduce 将指定一个 `initialValue`，通过和列表的第一个值结合来启动它的工作，然后和列表中剩下的值逐个级联。就像这样：

<p align="center">
	<img src="fig11.png" width="400">
</p>

Alternately, you can omit the `initialValue` in which case the first value of the list will act in place of the `initialValue` and the combining will start with the second value in the list, like this:

或者，你可以省略 `initialValue`， 在这种情况下，列表中的第一个值将会取代 `initialValue`，结合将会从列表的第二个值开始，就像这样：

<p align="center">
	<img src="fig12.png" width="400">
</p>

**Warning:** In JavaScript, if there's not at least one value in the reduction (either in the array or specified as `initialValue`), an error is thrown. Be careful not to omit the `initialValue` if the list for the reduction could possibly be empty under any circumstance.

**警告：** 在 JavaScript 中，如果 reduce 中没有值（在数组中或者被指定为 `initialValue`）就会抛错。需要注意的是，在任何情况下如果列表可能会空，请不要忽略 `initialValue`。

The function you pass to `reduce(..)` to perform the reduction is typically called a reducer. A reducer has a different signature from the mapper and predicate functions we looked at earlier. Reducers primarily receive the current reduction result as well as the next value to reduce it with. The current result at each step of the reduction is often referred to as the accumulator.

传给 `reduce(..)` 执行 reduction 的函数称为 reducer。reducer 和我们之前看到的 mapper 和谓词函数有不同的签名。首先 reducer 接收一个当前的 reduction 结果以及下一个值去 reduce。每一步被提交之后的结果被称为累加器。

For example, consider the steps involved in multiply-reducing the numbers `5`, `10`, and `15`, with an `initialValue` of `3`:

例如：观察乘法所涉及的步骤 —— 减少这些数字：`5`、`10` 和 `15`，`initialValue` 为 `3`：

1. `3` * `5` = `15`
2. `15` * `10` = `150`
3. `150` * `15` = `2250`

Expressed in JavaScript using the built-in `reduce(..)` method on arrays:

在 JavaScript 中数组是使用内置的 `reduce(..)` 方法被表达的：

```js
[5,10,15].reduce( (product,v) => product * v, 3 );
// 2250
```

But a standalone implementation of `reduce(..)` might look like this:

但是一个独立的 `reduce(..)` 实现可能像这样：

```js
function reduce(reducerFn,initialValue,arr) {
	var acc, startIdx;

	if (arguments.length == 3) {
		acc = initialValue;
		startIdx = 0;
	}
	else if (arr.length > 0) {
		acc = arr[0];
		startIdx = 1;
	}
	else {
		throw new Error( "Must provide at least one value." );
	}

	for (let idx = startIdx; idx < arr.length; idx++) {
		acc = reducerFn( acc, arr[idx], idx, arr );
	}

	return acc;
}
```

Just as with `map(..)` and `filter(..)`, the reducer function is also passed the lesser-common `idx` and `arr` arguments in case that's useful to the reduction. I would say I don't typically use these, but I guess it's nice to have them available.

就像 `map(..)` 和 `filter(..)` 一样，reducer 函数也传入了可能对 reduction 有用的 `idx` 和 `arr` 参数。我通常是不使用它们的，但是我觉得可以取得它们是极好的。

Recall in Chapter 4, we discussed the `compose(..)` utility and showed an implementation with `reduce(..)`:

回顾第 4 章，我们讨论了 `compose(..)` 实用函数，并展示了 `reduce(..)` 的实现：

```js
function compose(...fns) {
	return function composed(result){
		return fns.reverse().reduce( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```

To illustrate `reduce(..)`-based composition differently, consider a reducer that will compose functions left-to-right (like `pipe(..)` does), to use in an array chain:

为了说明 `reduce(..)` 的不同组成，可以考虑一个从左向右组成的函数（如 `pipe(..)`）用于链式数组：

```js
var pipeReducer = (composedFn,fn) => pipe( composedFn, fn );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```

`pipeReducer(..)` is unfortunately not point-free (see "No Points" in Chapter 3), but we can't just pass `pipe(..)` as the reducer itself, because it's variadic; the extra arguments (`idx` and `arr`) that `reduce(..)` passes to its reducer function would be problematic.

todo
不幸的是 `pipeReducer(..)` 是没有 point-free （参见第 3 章的 “No Points”），但是我们不能只通过 `pipe(..)` 作为 reducer 本身，因为它是可变的；`reduce(..)` 传给它的 reducer 函数的额外参数（`idx` 和 `arr`）是不明确的。

Earlier we talked about using `unary(..)` to limit a `mapperFn(..)` or `predicateFn(..)` to just a single argument. It might be handy to have a `binary(..)` that does something similar but limits to two arguments, for a `reducerFn(..)` function:
todo
之前我们讨论过使用 `unary(..)` 来限制 `mapperFn(..)` 或者 `predicateFn(..)` 只有一个参数。类似但是限制只有两个参数的 `binary(..)` 

```js
var binary =
	fn =>
		(arg1,arg2) =>
			fn( arg1, arg2 );
```

Using `binary(..)`, our previous example is a little cleaner:

使用 `binary(..)` 让我们之前的例子更简洁：

```js
var pipeReducer = binary( pipe );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```

Unlike `map(..)` and `filter(..)` whose order of passing through the array wouldn't actually matter, `reduce(..)` definitely uses left-to-right processing. If you want to reduce right-to-left, JavaScript provides a `reduceRight(..)`, with all other behaviors the same as `reduce(..)`:

与 `map(..)` 和 `filter(..)` 不同的是，传递数组的顺序并不重要，`reduce(..)` 是从左到右处理。如果想要从右到左，JavaScript 提供了 `reduceRight(..)`方法，其他的行为都和 `reduce(..)` 一样：

```js
var hyphenate = (str,char) => str + "-" + char;

["a","b","c"].reduce( hyphenate );
// "a-b-c"

["a","b","c"].reduceRight( hyphenate );
// "c-b-a"
```

Where `reduce(..)` works left-to-right and thus acts naturally like `pipe(..)` in composing functions, `reduceRight(..)`'s right-to-left ordering is natural for performing a `compose(..)`-like operation. So, let's revisit `compose(..)` using `reduceRight(..)`:

`reduce(..)` 从左到右工作，在组合函数中更像 `pipe(..)`，`reduceRight(..)` 从右到左的顺序对于执行 `compose(..)` 是很自然的。所以，我们来重温一下 `compose(..)` 使用 `reduceRight(..)`：

```js
function compose(...fns) {
	return function composed(result){
		return fns.reduceRight( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```

Now, we don't need to do `fns.reverse()`; we just reduce from the other direction!

现在，我们不需要 `fns.reverse()`；我们只是从另一个方向 reduce！

### Map As Reduce

The `map(..)` operation is iterative in its nature, so it can also be represented as a reduction (`reduce(..)`). The trick is to realize that the `initialValue` of `reduce(..)` can be itself an (empty) array, in which case the result of a reduction can be another list!

`map(..)` 操作在本质上迭代的，因此它也可以表示为 reduction（`reduce(..)`）。诀窍是要意识到 `reduce(..)` 的 `initialValue` 可以是一个（空）数组，在这种情况下 reduction 的结果是是一个新的列表！

```js
var double = v => v * 2;

[1,2,3,4,5].map( double );
// [2,4,6,8,10]

[1,2,3,4,5].reduce(
	(list,v) => (
		list.push( double( v ) ),
		list
	), []
);
// [2,4,6,8,10]
```

**Note:** We're cheating with this reducer and allowing a side effect by allowing `list.push(..)` to mutate the list that was passed in. In general, that's not a good idea, obviously, but since we know the `[]` list is being created and passed in, it's less dangerous. You could be more formal -- yet less performant! -- by creating a new list with the val `concat(..)`d onto the end. We'll come back to this cheat in Appendix A.

todo
**注意：** 我们正在欺骗这个 reducer，允许 `list.push(..)` 改变已经传入的列表带来的副作用。总的来说，这不是一个好主意，显然，我们知道 `[]` 列表正在创建和传入，并不是很危险。你可以更正规一些，但是性能不是很好！最后用 `concat(..)` 创建一个新的列表。我们将在附录 A 中详细讨论这个欺骗方法。

Implementing `map(..)` with `reduce(..)` is not on its surface an obvious step or even an improvement. However, this ability will be a crucial recognition for more advanced techniques like those we'll cover in Appendix A "Transducing".

用 `reduce(..)` 实现 `map(..)` 在表面上并不是明显的一步或者是提高，这种能力是对更高技术的重要认可，就像我们将要在附录 A 中介绍的 “Transducing”。todo

### Filter As Reduce

Just as `map(..)` can be done with `reduce(..)`, so can `filter(..)`:

正如 `map(..)` 可以用 `reduce(..)` 实现一样，也可以用 `filter(..)`：

```js
var isOdd = v => v % 2 == 1;

[1,2,3,4,5].filter( isOdd );
// [1,3,5]

[1,2,3,4,5].reduce(
	(list,v) => (
		isOdd( v ) ? list.push( v ) : undefined,
		list
	), []
);
// [1,3,5]
```

**Note:** More impure reducer cheating here. Instead of `list.push(..)`, we could have done `list.concat(..)` and returned the new list. We'll come back to this cheat in Appendix A.
todo
**注意：** 更不纯的 reducer 在作弊。我们可以结束 `list.concat(..)` 而不是 `list.push(..)`并返回一个新的值。我们将在附录 A 中详细讨论这个欺骗方法。

## Advanced List Operations

## 高级的列表操作

Now that we feel somewhat comfortable with the foundational list operations `map(..)`, `filter(..)`, and `reduce(..)`, let's look at a few more-sophisticated operations you may find useful in various situations. These are generally utilities you'll find in various FP libraries.

现在我们对基础的列表操作 `map(..)`、`filter(..)`和 `reduce(..)`有了基本的认识，接下来我们来看一些在各种情况都都很有用的更复杂的操作。你将会发现在各种 FP 函数库中这些是非常普遍的实用函数。

### Unique

Filtering a list to include only unique values, based on `indexOf(..)` searching ( which uses `===` strict equality comparision):

基于 `indexOf(..)` 过滤仅包含一个值的列表（使用  `===` 严格的相等比较）。

```js
var unique =
	arr =>
		arr.filter(
			(v,idx) =>
				arr.indexOf( v ) == idx
		);
```

This technique works by observing that we should only include the first occurrence of an item from `arr` into the new list; when running left-to-right, this will only be true if its `idx` position is the same as the `indexOf(..)` found position.
todo
这个方法的工作原理是：我们只需要包含在项目中第一次发生的从 `arr` 到新的列表；当从左到右的运行，只有当 `idx` 和 `indexOf(..)` 的值一样时才会是真。

Another way to implement `unique(..)` is to run through `arr` and include an item into a new (initially empty) list if that item cannot already be found in the new list. For that processing, we use `reduce(..)`:

实现 `unique(..)` 的另一种方式是通过 `arr` ，如果在新的列表中找不到这个项目，则将该项目存储到一个新的（最初是空的）列表中去。对于这种处理，我们使用 `reduce(..)`：

```js
var unique =
	arr =>
		arr.reduce(
			(list,v) =>
				list.indexOf( v ) == -1 ?
					( list.push( v ), list ) : list
		, [] );
```

**Note:** There are many other ways to implement this algorithm using more imperative approaches like loops, and many of them are likely "more efficient" performance-wise. However, the advantage of either of these presented approaches is that they use existing built-in list operations, which makes them easier to chain/compose alongside other list operations. We'll talk more about those concerns later in this chapter.

**注意：** 有许多其他的方法来实现这个算法，比如像 loops 这样的命令式方法，它们中的大多数可能有“更高效”的性能。然而，这两种方法的优点是它们使用现有的内置列表操作，是它们更容易和其他的列表操作一起使用。本章稍后将会详细讨论这些问题。

`unique(..)` nicely produces a new list with no duplicates:

`unique(..)` 很好的生成了一个没有重复的新列表：

```js
unique( [1,4,7,1,3,1,7,9,2,6,4,0,5,3] );
// [1, 4, 7, 3, 9, 2, 6, 0, 5]
```

### Flatten

From time to time, you may have (or produce through some other operations) an array that's not just a flat list of values, but with nested arrays, such as:

有时，你可能有（或者通过其他的方法生成）一个数组，它不仅仅是一个简单的值的列表，还包含潜逃的数组，比如：

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
```

What if you'd like to transform it into:

如果你想把它转换成下面这样：

```js
[ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

The operation we're looking for is typically called `flatten(..)`, and it could be implemented like this using our swiss army knife `reduce(..)`:

我们寻找的操作被称为 `flatten(..)`，它可以用我们的瑞士军刀 `reduce(..)` 来实现：

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```

**Note:** This implementation choice relies on recursion to handle the nesting of lists. More on recursion in a later chapter.

**注意：** 它依赖于递归来处理列表的嵌套，更多关于递归的内容在后面的章节会提到。

To use `flatten(..)` with an array of arrays (of any nested depth):

对多维数组（深层嵌套）使用 `flatten(..)` 。

```js
flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]] );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

You might like to limit the recursive flattening to a certain depth. We can handle this by adding an optional `depth` limit argument to the implementaiton:

你可能希望限制递归展开到一定的程度。我们可以通过在实现中添加一个可选的 `depth` 限制参数来处理这个问题。

```js
var flatten =
	(arr,depth = Infinity) =>
		arr.reduce(
			(list,v) =>
				list.concat(
					depth > 0 ?
						(depth > 1 && Array.isArray( v ) ?
							flatten( v, depth - 1 ) :
							v
						) :
						[v]
				)
		, [] );
```

Illustrating the results with different flattening depths:

用不同的展开程度来说明结果：

```js
flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 0 );
// [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 1 );
// [0,1,2,3,4,[5,6,7],[8,[9,[10,[11,12],13]]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 2 );
// [0,1,2,3,4,5,6,7,8,[9,[10,[11,12],13]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 3 );
// [0,1,2,3,4,5,6,7,8,9,[10,[11,12],13]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 4 );
// [0,1,2,3,4,5,6,7,8,9,10,[11,12],13]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 5 );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

#### Mapping, Then Flattening

One of the most common usages of `flatten(..)` behavior is when you've mapped a list of elements where each transformed value from the original list is now itself a list of values. For example:

最常见的方法之一`flatten(..)` 的行为是：当你 map 元素列表的时候，每个原始列表中被转换的值，现在本身就是一个值列表。例如：

```js
var firstNames = [
	{ name: "Jonathan", variations: [ "John", "Jon", "Jonny" ] },
	{ name: "Stephanie", variations: [ "Steph", "Stephy" ] },
	{ name: "Frederick", variations: [ "Fred", "Freddy" ] }
];

firstNames
.map( entry => [entry.name].concat( entry.variations ) );
// [ ["Jonathan","John","Jon","Jonny"], ["Stephanie","Steph","Stephy"],
//   ["Frederick","Fred","Freddy"] ]
```

The return value is an array of arrays, which might be more awkward to work with. If we want a single dimension list with all the names, we can then `flatten(..)` that result:

返回的值是一个多维数组，这可能会比较尴尬。如果我们想要一个具有所有名称的单维数组，那就可以用 `flatten(..)` 来处理返回值：

```js
flatten(
	firstNames
	.map( entry => [entry.name].concat( entry.variations ) )
);
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

Besides being slightly more verbose, the disadvantage of doing the `map(..)` and `flatten(..)` as separate steps is primarily around performance; this approach processes the list twice.

稍微啰嗦一下， `map(..)` 和 `flatten(..)` 的缺点是作为单独的步骤它主要是围绕性能的；此方法处理两次列表。

FP libraries typically define a `flatMap(..)` (often also called `chain(..)`) that does the mapping-then-flattening combined. For consistency and ease of composition (via currying), the `flatMap(..)` / `chain(..)` utility typically matches the `mapperFn, arr` parameter order that we saw earlier with the standalone `map(..)`, `filter(..)`, and `reduce(..)` utilities.

FP 函数库定义了 `flatMap(..)`（经常被称作 `chain(..)`），它做了映射，然后平坦的整合在一起。为了一致性且易于组合（通过柯里化），`flatMap(..)` / `chain(..)` 实用函数匹配 `mapperFn, arr` 的参数规则，可以很轻松的看到独立的 `map(..)`、`filter(..)`和`reduce(..)` 实用函数。

```js
flatMap( entry => [entry.name].concat( entry.variations ), firstNames );
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

The naive implementation of `flatMap(..)` with both steps done separately:

`flatMap(..)` 的实现分为两个步骤：

```js
var flatMap =
	(mapperFn,arr) =>
		flatten( arr.map( mapperFn ), 1 );
```

**Note:** We use `1` for the flattening-depth because the typical definition of `flatMap(..)` is that the flattening is shallow on just the first level.

**注意：**我们使用 `1` 来表示展开的深度，因为 `flatMap(..)` 的特有定义是仅有第一级是很浅的。todo

Since this approach still processes the list twice resulting in worse performance, we can combine the operations manually, using `reduce(..)`:

由于这种方法仍然处理两次列表，导致性能比较差，所以我们可以使用 `reduce(..)` 来手动合并操作：

```js
var flatMap =
	(mapperFn,arr) =>
		arr.reduce(
			(list,v) =>
				list.concat( mapperFn( v ) )
		, [] );
```

While there's some convenience and performance gained with a `flatMap(..)` utility, there may very well be times when you need other operations like `filter(..)`ing mixed in. If that's the case, doing the `map(..)` and `flatten(..)` separately might still be more appropriate.

虽然使用 `flatMap(..)` 实用程序可以获得一些便利和性能，但是当需要像 `filter(..)` 这种其他的操作混合时确实很棒的。如果是那样的话， `map(..)` 和 `flatten(..)` 可能更合适。

### Zip

So far, the list operations we've examined have operated on a single list. But some cases will need to process multiple lists. One well-known operation alternates selection of values from each of two input lists into sub-lists, called `zip(..)`:

到目前为止，我们检查过的操作都是在耽搁列表上执行的。但是一些情况下可能需要处理多个列表。一个众所周知的操作，将两个
```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]
```

Values `1` and `2` were selected into the sub-list `[1,2]`, then `3` and `4` into `[3,4]`, etc. The definition of `zip(..)` requires a value from each of the two lists. If the two lists are of different lengths, the selection of values will continue until the shorter list has been exhausted, with the extra values in the other list ignored.

An implementation of `zip(..)`:

```js
function zip(arr1,arr2) {
	var zipped = [];
	arr1 = arr1.slice();
	arr2 = arr2.slice();

	while (arr1.length > 0 && arr2.length > 0) {
		zipped.push( [ arr1.shift(), arr2.shift() ] );
	}

	return zipped;
}
```

The `arr1.slice()` and `arr2.slice()` calls ensure `zip(..)` is pure by not causing side effects on the received array references.

**Note:** There are some decidedly un-FP things going on in this implementation. There's an imperative `while`-loop and mutations of lists with both `shift()` and `push(..)`. Earlier in the book, I asserted that it's reasonable for pure functions to use impure behavior inside them (usually for performance), as long as the effects are fully self-contained. This implementation is safely pure.

### Merge

Merging two lists by interleaving values from each source looks like this:

```js
mergeLists( [1,3,5,7,9], [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

It may not be obvious, but this result seems similar to what we get if we compose `flatten(..)` and `zip(..)`:

```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]

flatten( [ [1,2], [3,4], [5,6], [7,8], [9,10] ] );
// [1,2,3,4,5,6,7,8,9,10]

// composed:
flatten( zip( [1,3,5,7,9], [2,4,6,8,10] ) );
// [1,2,3,4,5,6,7,8,9,10]
```

However, recall that `zip(..)` only selects values until the shorter of two lists is exhausted, ignoring the leftover values; merging two lists would most naturally retain those extra values. Also, `flatten(..)` works recursively on nested lists, but you might expect list-merging to only work shallowly, keeping nested lists.

So, let's define a `mergeLists(..)` that works more like we'd expect:

```js
function mergeLists(arr1,arr2) {
	var merged = [];
	arr1 = arr1.slice();
	arr2 = arr2.slice();

	while (arr1.length > 0 || arr2.length > 0) {
		if (arr1.length > 0) {
			merged.push( arr1.shift() );
		}
		if (arr2.length > 0) {
			merged.push( arr2.shift() );
		}
	}

	return merged;
}
```

**Note:** Various FP libraries don't define a `mergeLists(..)` but instead define a `merge(..)` that merges properties of two objects; the results of such a `merge(..)` will differ from our `mergeLists(..)`.

Alternatively, here's a couple of options to implement the list merging as a reducer:

```js
// via @rwaldron
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);


// via @WebReflection
var mergeReducer =
	(merged,v,idx) =>
		merged
			.slice( 0, idx * 2 )
			.concat( v, merged.slice( idx * 2 ) );
```

And using a `mergeReducer(..)`:

```js
[1,3,5,7,9]
.reduce( mergeReducer, [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

**Tip:** We'll use the `mergeReducer(..)` trick later in the chapter.

## Method vs. Standalone

A common source of frustration for FPers in JavaScript is unifying their strategy for working with utilities when some of them are provided as standalone functions -- think about the various FP utilities we've derived in previous chapters -- and others are methods of the array prototype -- like the ones we've seen in this chapter.

The pain of this problem becomes more evident when you consider combining multiple operations:

```js
[1,2,3,4,5]
.filter( isOdd )
.map( double )
.reduce( sum, 0 );					// 18

// vs.

reduce(
	map(
		filter( [1,2,3,4,5], isOdd ),
		double
	),
	sum,
	0
);									// 18
```

Both API styles accomplish the same task, but they have very different ergonomics. Many FPers will prefer the latter to the former, but the former is unquestionably more common in JavaScript. One thing specifically that's disliked about the latter is the nesting of the calls. The preference for the method chain style -- typically called a fluent API style, as in jQuery and other tools -- is that it's compact/concise and it reads in declarative top-down order.

The visual order for that manual composition of the standalone style is neither strictly left-to-right (top-to-bottom) nor right-to-left (bottom-to-top); it's inner-to-outer, which harms the readability.

Automatic composition normalizes the reading order as right-to-left (bottom-to-top) for both styles. So, to explore the implications of the style differences, let's examine composition specifically; it seems like it should be straightforward, but it's a little awkward in both cases.

### Composing Method Chains

The array methods receive the implicit `this` argument, so despite their appearance, they can't be treated as unary; that makes composition more awkward. To cope, we'll first need a `this`-aware version of `partial(..)`:

```js
var partialThis =
	(fn,...presetArgs) =>
		// intentionally `function` to allow `this`-binding
		function partiallyApplied(...laterArgs){
			return fn.apply( this, [...presetArgs, ...laterArgs] );
		};
```

We'll also need a version of `compose(..)` that calls each of the partially applied methods in the context of the chain -- the input value it's being "passed" (via implicit `this`) from the previous step:

```js
var composeChainedMethods =
	(...fns) =>
		result =>
			fns.reduceRight(
				(result,fn) =>
					fn.call( result )
				, result
			);
```

And using these two `this`-aware utilities together:

```js
composeChainedMethods(
   partialThis( Array.prototype.reduce, sum, 0 ),
   partialThis( Array.prototype.map, double ),
   partialThis( Array.prototype.filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

**Note:** The three `Array.prototype.XXX`-style references are grabbing references to the built-in `Array.prototype.*` methods so that we can reuse them with our own arrays.

### Composing Standalone Utilities

Standalone `compose(..)`-style composition of these utilities doesn't need all the `this` contortions, which is its most favorable argument. For example, we could define standalones as:

```js
var filter = (arr,predicateFn) => arr.filter( predicateFn );

var map = (arr,mapperFn) => arr.map( mapperFn );

var reduce = (arr,reducerFn,initialValue) =>
	arr.reduce( reducerFn, initialValue );
```

But this particular standalone style suffers from its own awkwardness; the cascading array context is the first argument rather than the last, so we have to use right-partial application to compose them:

```js
compose(
	partialRight( reduce, sum, 0 ),
	partialRight( map, double ),
	partialRight( filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

That's why FP libraries typically define `filter(..)`, `map(..)`, and `reduce(..)` to alternately receive the array last instead of first. They also typically automatically curry the utilities:

```js
var filter = curry(
	(predicateFn,arr) =>
		arr.filter( predicateFn )
);

var map = curry(
	(mapperFn,arr) =>
		arr.map( mapperFn )
);

var reduce = curry(
	(reducerFn,initialValue,arr) =>
		arr.reduce( reducerFn, initialValue );
```

Working with the utilities defined in this way, the composition flow is a bit nicer:

```js
compose(
	reduce( sum )( 0 ),
	map( double ),
	filter( isOdd )
)
( [1,2,3,4,5] );					// 18
```

The cleanliness of this approach is in part why FPers prefer the standalone utility style instead of instance methods. But your mileage may vary.

### Adapting Methods To Standalones

In the previous definition of `filter(..)` / `map(..)` / `reduce(..)`, you might have spotted the common pattern across all three: they all dispatch to the corresponding native array method. So, can we generate these standalone adaptations with a utility? Yes! Let's make a utility called `unboundMethod(..)` to do just that:

```js
var unboundMethod =
	(methodName,argCount = 2) =>
		curry(
			(...args) => {
				var obj = args.pop();
				return obj[methodName]( ...args );
			},
			argCount
		);
```

And to use this utility:

```js
var filter = unboundMethod( "filter", 2 );
var map = unboundMethod( "map", 2 );
var reduce = unboundMethod( "reduce", 3 );

compose(
	reduce( sum )( 0 ),
	map( double ),
	filter( isOdd )
)
( [1,2,3,4,5] );					// 18
```

**Note:** `unboundMethod(..)` is called `invoker(..)` in Ramda.

### Adapting Standalones To Methods

If you prefer to work with only array methods (fluent chain style), you have two choices. You can:

1. Extend the built-in `Array.prototype` with additional methods.
2. Adapt a standalone utility to work as a reducer function and pass it to the `reduce(..)` instance method.

**Don't do (1).** It's never a good idea to extend built-in natives like `Array.prototype` -- unless you define a subclass of `Array`, but that's beyond our discussion scope here. In an effort to discourage bad practices, we won't go any further into this approach.

Let's **focus on (2)** instead. To illustrate this point, we'll convert the recursive `flatten(..)` standalone utility from earlier:

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```

Let's pull out the inner `reducer(..)` function as the standalone utility (and adapt it to work without the outer `flatten(..)`):

```js
// intentionally a function to allow recursion by name
function flattenReducer(list,v) {
	return list.concat(
		Array.isArray( v ) ? v.reduce( flattenReducer, [] ) : v
	);
}
```

Now, we can use this utility in an array method chain via `reduce(..)`:

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
.reduce( flattenReducer, [] )
// ..
```

## Looking For Lists

So far, most of the examples have been rather trivial, based on simple lists of numbers or strings. Let's now talk about where list operations can start to shine: modeling an imperative series of statements declaratively.

Consider this base example:

```js
var getSessionId = partial( prop, "sessId" );
var getUserId = partial( prop, "uId" );

var session, sessionId, user, userId, orders;

session = getCurrentSession();
if (session != null) sessionId = getSessionId( session );
if (sessionId != null) user = lookupUser( sessionId );
if (user != null) userId = getUserId( user );
if (userId != null) orders = lookupOrders( userId );
if (orders != null) processOrders( orders );
```

First, let's observe that the five variable declarations and the running series of `if` conditionals guarding the function calls are effectively one big composition of these six calls `getCurrentSession()`, `getSessionId(..)`, `lookupUser(..)`, `getUserId(..)`, `lookupOrders(..)`, and `processOrders(..)`. Ideally, we'd like to get rid of all these variable declarations and imperative conditionals.

Unfortunately, the `compose(..)` / `pipe(..)` utilties we explored in Chapter 4 don't by themselves offer a convenient way to express the `!= null` conditionals in the composition. Let's define a utility to help:

```js
var guard =
	fn =>
		arg =>
			arg != null ? fn( arg ) : arg;
```

This `guard(..)` utility lets us map the five conditional-guarded functions:

```js
[ getSessionId, lookupUser, getUserId, lookupOrders, processOrders ]
.map( guard )
```

The result of this mapping is an array of functions that are ready to compose (actually, pipe, in this listed order). We could spread this array to `pipe(..)`, but since we're already doing list operations, let's do it with a `reduce(..)`, using the session value from `getCurrentSession()` as the initial value:

```js
.reduce(
	(result,nextFn) => nextFn( result )
	, getCurrentSession()
)
```

Next, let's observe that `getSessionId(..)` and `getUserId(..)` can be expressed as a mapping from the respective values `"sessId"` and `"uId"`:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
```

But to use these, we'll need to interleave them with the other three functions (`lookupUser(..)`, `lookupOrders(..)`, and `processOrders(..)`) to get the array of five functions to guard / compose as discussed above.

To do the interleaving, we can model this as list merging. Recall `mergeReducer(..)` from earlier in the chapter:

```js
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);
```

We can use `reduce(..)` (our swiss army knife, remember!?) to "insert" `lookupUser(..)` in the array between the generated `getSessionId(..)` and `getUserId(..)` functions, by merging two lists:

```js
.reduce( mergeReducer, [ lookupUser ] )
```

Then we'll concatenate `lookupOrders(..)` and `processOrders(..)` onto the end of the running functions array:

```js
.concat( lookupOrders, processOrders )
```

To review, the generated list of five functions is expressed as:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
```

Finally, to put it all together, take this list of functions and tack on the guarding and composition from earlier:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
.map( guard )
.reduce(
	(result,nextFn) => nextFn( result )
	, getCurrentSession()
);
```

Gone are all the imperative variable declarations and conditionals, and in their place we have clean and declarative list operations chained together.

If this version is harder for you read right now than the original, don't worry. The original is unquestionably the imperative form you're probably more familiar with. Part of your evolution to become a functional programmer is to develop a recognition of FP patterns such as list operations. Over time, these will jump out of the code more readily as your sense of code readability shifts to declarative style.

Before we leave this topic, let's take a reality check: the example here is heavily contrived. Not all code segments will be straightforwardly modeled as list operations. The pragmatic take-away is to develop the instinct to look for these opportunities, but not get too hung up on code acrobatics; some improvement is better than none. Always step back and ask if you're **improving or harming** code readability.

## Fusion

As you roll FP list operations into more of your thinking about code, you'll likely start seeing very quickly chains that combine behavior like:

```js
..
.filter(..)
.map(..)
.reduce(..);
```

And more often than not, you're also probably going to end up with chains with multiple adjacent instances of each operation, like:

```js
someList
.filter(..)
.filter(..)
.map(..)
.map(..)
.map(..)
.reduce(..);
```

The good news is the chain-style is declarative and it's easy to read the specific steps that will happen, in order. The downside is that each of these operations loops over the entire list, meaning performance can suffer unnecessarily, especially if the list is longer.

With the alternate standalone style, you might see code like this:

```js
map(
	fn3,
	map(
		fn2,
		map( fn1, someList )
	)
);
```

With this style, the operations are listed from bottom-to-top, and we still loop over the list 3 times.

Fusion deals with combining adjacent operators to reduce the number of times the list is iterated over. We'll focus here on collapsing adjacent `map(..)`s as it's the most straightforward to explain.

Imagine this scenario:

```js
var removeInvalidChars = str => str.replace( /[^\w]*/g, "" );

var upper = str => str.toUpperCase();

var elide = str =>
	str.length > 10 ?
		str.substr( 0, 7 ) + "..." :
		str;

var words = "Mr. Jones isn't responsible for this disaster!"
	.split( /\s/ );

words;
// ["Mr.","Jones","isn't","responsible","for","this","disaster!"]

words
.map( removeInvalidChars )
.map( upper )
.map( elide );
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

Think about each value that goes through this flow of transformations. The first value in the `words` list starts out as `"Mr."`, becomes `"Mr"`, then `"MR"`, and then passes through `elide(..)` unchanged. Another piece of data flows: `"responsible"` -> `"responsible"` -> `"RESPONSIBLE"` -> `"RESPONS..."`.

In other words, you could think of these data transformations like this:

```js
elide( upper( removeInvalidChars( "Mr." ) ) );
// "MR"

elide( upper( removeInvalidChars( "responsible" ) ) );
// "RESPONS..."
```

Did you catch the point? We can express the three separate steps of the adjacent `map(..)` calls as a composition of the transformers, since they are all unary functions and each returns the value that's suitable as input to the next. We can fuse the mapper functions using `compose(..)`, and then pass the composed function to a single `map(..)` call:

```js
words
.map(
	compose( elide, upper, removeInvalidChars )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

This is another case where `pipe(..)` can be a more convenient form of composition, for its ordering readability:

```js
words
.map(
	pipe( removeInvalidChars, upper, elide )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

What about fusing two or more `filter(..)` predicate functions? Typically treated as unary functions, they seem suitable for composition. But the wrinkle is they each return a different kind of value (`boolean`) than the next one would want as input. Fusing adjacent `reduce(..)` calls is also possible, but reducers are not unary so that's a bit more challenging; we need more sophisticated tricks to pull this kind of fusion off. We'll cover these advanced techniques in Appendix A "Transducing".

## Beyond Lists

So far we've been discussing operations in the context of the list (array) data structure; it's by far the most common scenario you encounter them. But in a more general sense, these operations can be performed against any collection of values.

Just as we said earlier that array's `map(..)` adapts a single-value operation to all its values, any data structure can provide a `map(..)` operation to do the same. Likewise, it can implement `filter(..)`, `reduce(..)`, or any other operation that makes sense for working with the data structure's values.

The important part to maintain in the spirit of FP is that these operators must behave according to value immutability, meaning that they must return a new data structure rather than mutating the existing one.

Let's illustrate with a well-known data structure: the binary tree. A binary tree is a node (just an object!) that has two references to other nodes (themselves binary trees), typically referred to as *left* and *right* child trees. Each node in the tree holds one value of the overall data structure.

<p align="center">
	<img src="fig7.png" width="250">
</p>

For ease of illustration, we'll make our binary tree a binary search tree (BST). However, the operations we'll identify work the same for any regular non-BST binary tree.

**Note:** A binary search tree is a general binary tree with a special constraint on the relationship of values in the tree to each other. Each value of nodes on the left side of a tree is less than the value of the node at the root of that tree, which in turn is less than each value of nodes in the right side of the tree. The notion of "less than" is relative to the kind of data stored; it can be numerical for numbers, lexicographic for strings, etc. BSTs are useful because they make searching for a value in the tree straightforward and more efficient, using a recursive binary search algorithm.

To make a binary tree node object, let's use this factory function:

```js
var BinaryTree =
	(value,parent,left,right) => ({ value, parent, left, right });
```

For convenience, we make each node store the `left` and `right` child trees as well as a reference to its own `parent` node.

Let's now define a BST of names of common produce (fruits, vegetables):

```js
var banana = BinaryTree( "banana" );
var apple = banana.left = BinaryTree( "apple", banana );
var cherry = banana.right = BinaryTree( "cherry", banana );
var apricot = apple.right = BinaryTree( "apricot", apple );
var avocado = apricot.right = BinaryTree( "avocado", apricot );
var cantelope = cherry.left = BinaryTree( "cantelope", cherry );
var cucumber = cherry.right = BinaryTree( "cucumber", cherry );
var grape = cucumber.right = BinaryTree( "grape", cucumber );
```

In this particular tree structure, `banana` is the root node; this tree could have been set up with nodes in different locations, but still had a BST with the same traversal.

Our tree looks like:

<p align="center">
	<img src="fig8.png" width="450">
</p>

There are multiple ways to traverse a binary tree to process its values. If it's a BST (our's is!) and we do an *in-order* traversal -- always visit the left child tree first, then the node itself, then the right child tree -- we'll visit the values in ascending (sorted) order.

Since you can't just easily `console.log(..)` a binary tree like you can with an array, let's first define a convenience method, mostly to use for printing. `forEach(..)` will visit the nodes of a binary tree in the same manner as an array:

```js
// in-order traversal
BinaryTree.forEach = function forEach(visitFn,node){
	if (node) {
		if (node.left) {
			forEach( visitFn, node.left );
		}

		visitFn( node );

		if (node.right) {
			forEach( visitFn, node.right );
		}
	}
};
```

**Note:** Working with binary trees lends itself most naturally to recursive processing. Our `forEach(..)` utility recursively calls itself to process both the left and right child trees. We'll cover recursion in more detail in a later chapter, where we'll cover recursion in that chapter on recursion.

Recall `forEach(..)` was described at the beginning of this chapter as only being useful for side effects, which is not very typically desired in FP. In this case, we'll use `forEach(..)` only for the side effect of I/O, so it's perfectly reasonable as a helper.

Use `forEach(..)` to print out values from the tree:

```js
BinaryTree.forEach( node => console.log( node.value ), banana );
// apple apricot avocado banana cantelope cherry cucumber grape

// visit only the `cherry`-rooted subtree
BinaryTree.forEach( node => console.log( node.value ), cherry );
// cantelope cherry cucumber grape
```

To operate on our binary tree data structure using FP patterns, let's start by defining a `map(..)`:

```js
BinaryTree.map = function map(mapperFn,node){
	if (node) {
		let newNode = mapperFn( node );
		newNode.parent = node.parent;
		newNode.left = node.left ?
			map( mapperFn, node.left ) : undefined;
		newNode.right = node.right ?
			map( mapperFn, node.right ): undefined;

		if (newNode.left) {
			newNode.left.parent = newNode;
		}
		if (newNode.right) {
			newNode.right.parent = newNode;
		}

		return newNode;
	}
};
```

You might have assumed we'd `map(..)` only the node `value` properties, but in general we might actually want to map the tree nodes themselves. So, the `mapperFn(..)` is passed the whole node being visited, and it expects to receive a new `BinaryTree(..)` node back, with the transformation applied. If you just return the same node, this operation will mutate your tree and quite possibly cause unexpected results!

Let's map our tree to a list of produce with all uppercase names:

```js
var BANANA = BinaryTree.map(
	node => BinaryTree( node.value.toUpperCase() ),
	banana
);

BinaryTree.forEach( node => console.log( node.value ), BANANA );
// APPLE APRICOT AVOCADO BANANA CANTELOPE CHERRY CUCUMBER GRAPE
```

`BANANA` is a different tree (with all different nodes) than `banana`, just like calling `map(..)` on an array returns a new array. Just like arrays of other objects/arrays, if `node.value` itself references some object/array, you'll also need to handle manually copying it in the mapper function if you want deeper immutability.

How about `reduce(..)`? Same basic process: do an in-order traversal of the tree nodes. One usage would be to `reduce(..)` our tree to an array of its values, which would be useful in further adapting other typical list operations. Or we can `reduce(..)` our tree to a string concatenation of all its produce names.

We'll mimic the behavior of the array `reduce(..)`, which makes passing the `initialValue` argument optional. This algorithm is a little trickier, but still manageable:

```js
BinaryTree.reduce = function reduce(reducerFn,initialValue,node){
	if (arguments.length < 3) {
		// shift the parameters since `initialValue` was omitted
		node = initialValue;
	}

	if (node) {
		let result;

		if (arguments.length < 3) {
			if (node.left) {
				result = reduce( reducerFn, node.left );
			}
			else {
				return node.right ?
					reduce( reducerFn, node, node.right ) :
					node;
			}
		}
		else {
			result = node.left ?
				reduce( reducerFn, initialValue, node.left ) :
				initialValue;
		}

		result = reducerFn( result, node );
		result = node.right ?
			reduce( reducerFn, result, node.right ) : result;
		return result;
	}

	return initialValue;
};
```

Let's use `reduce(..)` to make our shopping list (an array):

```js
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	banana
);
// ["apple","apricot","avocado","banana","cantelope"
//   "cherry","cucumber","grape"]
```

Finally, let's consider `filter(..)` for our tree. This algorithm is trickiest so far because it effectively (not actually) involves removing nodes from the tree, which requires handling several corner cases. Don't get intimiated by the implementation, though. Just skip over it for now, if you prefer, and focus on how we use it instead.

```js
BinaryTree.filter = function filter(predicateFn,node){
	if (node) {
		let newNode;
		let newLeft = node.left ?
			filter( predicateFn, node.left ) : undefined;
		let newRight = node.right ?
			filter( predicateFn, node.right ) : undefined;

		if (predicateFn( node )) {
			newNode = BinaryTree(
				node.value,
				node.parent,
				newLeft,
				newRight
			);
			if (newLeft) {
				newLeft.parent = newNode;
			}
			if (newRight) {
				newRight.parent = newNode;
			}
		}
		else {
			if (newLeft) {
				if (newRight) {
					newNode = BinaryTree(
						undefined,
						node.parent,
						newLeft,
						newRight
					);
					newLeft.parent = newRight.parent = newNode;

					if (newRight.left) {
						let minRightNode = newRight;
						while (minRightNode.left) {
							minRightNode = minRightNode.left;
						}

						newNode.value = minRightNode.value;

						if (minRightNode.right) {
							minRightNode.parent.left =
								minRightNode.right;
							minRightNode.right.parent =
								minRightNode.parent;
						}
						else {
							minRightNode.parent.left = undefined;
						}

						minRightNode.right =
							minRightNode.parent = undefined;
					}
					else {
						newNode.value = newRight.value;
						newNode.right = newRight.right;
						if (newRight.right) {
							newRight.right.parent = newNode;
						}
					}
				}
				else {
					return newLeft;
				}
			}
			else {
				return newRight;
			}
		}

		return newNode;
	}
};
```

The majority of this code listing is dedicated to handling the shifting of a node's parent/child references if it's "removed" (filtered out) of the duplicated tree structure.

As an example to illustrate using `filter(..)`, let's narrow our produce tree down to only vegetables:

```js
var vegetables = [ "asparagus", "avocado", "brocolli", "carrot",
	"celery", "corn", "cucumber", "lettuce", "potato", "squash",
	"zucchini" ];

var whatToBuy = BinaryTree.filter(
	// filter the produce list only for vegetables
	node => vegetables.indexOf( node.value ) != -1,
	banana
);

// shopping list
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	whatToBuy
);
// ["avocado","cucumber"]
```

You will likely use most of the list operations from this chapter in the context of simple arrays. But now we've seen that the concepts apply to whatever data structures and operations you might need. That's a powerful expression of how FP can be widely applied to many different application scenarios!

## Summary

Three common and powerful list operations:

* `map(..)`: transforms values as it projects them to a new list.
* `filter(..)`: selects or excludes values as it projects them to a new list.
* `reduce(..)`: combines values in a list to produce some other (usually but not always non-list) value.

Other more advanced operations that can be very useful in processing lists: `unique(..)`, `flatten(..)`, and `merge(..)`.

Fusion uses function composition techniques to consolidate multiple adjacent `map(..)` calls. This is mostly a performance optimization, but it also improves the declarative nature of your list operations.

Lists are typically visualized as arrays, but can be generalized as any data structure that represents/produces an ordered collection of values. As such, all these "list operations" are actually "data structure operations".
