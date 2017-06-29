# JavaScript 轻量级函数式编程
# 附录 B: 谦虚的 Monad

让我从这个附录开始承认：在开始写以下内容之前我并不太了解 Monad 是什么。为了得到一些合乎情理的东西，我们犯了很多错误。如果你不相信我，去看看 [这本书 Git 仓库](https://github.com/getify/Functional-Light-JS) 中关于本章的提交历史吧！

本书中包含了关于 Monad 的话题。就像我在本书中写的一样，这是每个开发者在学习 FP 的时候都会遇到的一部分旅程，。

我们基本上以对 Monad 的简单介绍结束了本书，而很多其他的 FP 函数库几乎都是从 Monad 开始的！在 "functional light" 编程中我没有遇到太多需要考虑 Monad 的问题，所以这就是为什么这个资料比主要的核心更有价值的原因。但是并不是说 Monad 是没用的或者是不普遍的 —— 相反，它们非常重要。

在 JavaScript 中的 FP 界有一个小笑话，几乎每个人都会在他们的文章或者博客里写 Monad 是什么，就像写它是一个仪式。在过去的几年里，Monad 被描述为卷饼、洋葱和各种各样古怪的抽闲概念。我希望在这里没有这样愚蠢的事情发生！

> Monad 在 endofunctor 范畴中仅仅是一个 Monad。

我们将这个引用作为序言，所以回到这里似乎很合适。但是，我们不会讨论 Monad 、endofunctor 或者范畴论。这句话真的是句废话。

我唯一希望你可以从这个讨论中得到的是不要害怕 Monad 这个术语或者这个概念 —— 但是我已经这样很多年了！ —— 当你看到它们的时候你会认出来。你可能，也只是可能会使用到它们。

## Type

在 FP 中有一个巨大的兴趣领域：类型论，本书基本上完全远离了它们。我不会深入到类型类，坦白的说，我没有资格这样做。即使我做了，你也不会感激的。

但是我要说的是，Monad 基本上是一个值类型。

数字 `42` 有一个值类型（number），它带有我们依赖的特征和功能。字符串 `"42"` 可能看起来像一些，但是在编程里它有不同的用途。

在面向对象编程中，当你有一组数据（甚至是一个单独的离散值），并且想要给它绑上一些行为，那么你将创建一个对象／类来表示 "type"。实例是这个 type 的一员。这种做法通常被称为 “数据结构”。

我将会非常宽松的使用数据结构这个概念，并且断言，在程序中我们会发现它很有用，它可以定义一组行为和对某个值的限制，并且把它们和该值打包在一个单独的抽象中。这样，当我们在编程中使用一个或多个这种值的时候，它们的行为会自然的出现，并且会使它们更方便的工作。方便的是，对你的代码的读者来说，是更有描述性和声明性的。

Monad 是一种数据结构。是一种类型。它有一组特定的行为，旨在使一个值是可预测的。

Recall in Chapter 8 that we talked about functors: a value along with a map-like utility to perform an operation on all its constitute data members. A monad is a functor that includes some additional behavior.

回顾第 8 章，我们谈到了 functor：一个值以及一个类似 map 的实用函数对其所有的成员执行一个运算。Monad 是一个包含一些额外行为的 functors。

## Loose Interface

实际上，Monad 并不是单一的数据类型，它更像是相关联的数据类型集合。它是一个接口，根据不同的值的不同需要来实现。每个实现都是一种不同类型的 Monad。

例如，你可以阅读 "Identity Monad"、"IO Monad"、"Maybe Monad"、"Either Monad" 或者其他种类的内容。他们中的每一个都有基本的 Monad 行为定义，但是它根据每个不同类型的 Monad 用例来继承或者重写交互。

它不仅仅是一个接口，因为它不只是使一个对象成为 Monad 的某些 API 方法的实现。对这些方法的交互的保障是必须的，是 monadic 的。这些众所周知的常量对于使用 Monad 提高可读性是至关重要的；另外，它是一个特殊的数据结构，读者必须全部阅读才能明白。

事实上，这些 monadic 方法的名字甚至没有一个统一的标准，一个真正的接口应该是被命名的；Monad 更像是一个 loose interface。有些人称这些方法为 `bind(..)`,有些称它为 `chain(..)`，有些称它为 `flatMap(..)`，等等。

所以，Monad 是一个对象数据类型，并且有充足的方法（几乎任何名称或排序），至少满足了 Monad 定义的主要行为需求。每一种 Monad 在最小值上都有不同程度的延伸。但是，因为它们在行为上都有重叠，所以一起使用两种不同的 Monad 仍然是可控的。

从某种意义上说，Monad 更像是接口。

## Maybe

在 FP 的资料中，像 Maybe 这样涵盖 Monad 是很普遍的。事实上，Maybe monad 真的是另外两个更简单的 Monad 的配对：Just 和 Nothing。 

由于 Monad 是一个类型，你可能认为我们应该定义 `Maybe` 作为类的实例。这是一种有效的方法，但是它在我不想玩的方法中引入了 `this` 绑定问题；相反，我打算只使用一个简单的函数／对象方法。

以下是 Maybe 的最简单的实现：

```js
var Maybe = { Just, Nothing, of/* aka: unit, pure */: Just };

function Just(val) {
	return { map, chain, ap, inspect };

	// *********************

	function map(fn) { return Just( fn( val ) ); }
	// 又叫：bind, flatMap
	function chain(fn) { return fn( val ); }
	function ap(anotherMonad) { return anotherMonad.map( val ); }

	function inspect() {
		return `Just(${ val })`;
	}
}

function Nothing() {
	return { map: Nothing, chain: Nothing, ap: Nothing, inspect };

	// *********************

	function inspect() {
		return "Nothing";
	}
}
```

**注意：** `inspect(..)` 方法只用于我们的示例中。它在 monadic sense 中不起直接作用。

如果现在大部分都没有意义的话，不要担心。我们不会过多的深入 Monad 背后设计的细节／理论。相反，我们将会更专注的说明我们可以用它做什么。

任何含有 `Just(..)` 和 `Nothing()` 的 Monad 实例和所有的 Monad 方法一样，都有  `map(..)`、`chain(..)`（也叫 `bind(..)` 或者 `flatMap(..)`）和 `ap(..)` 方法。这些方法及其行为的目的在于提供多个 Monad 实例一起工作的标准化方法。你将会注意到，`val` 值是 `Just(..)` 实例的保留，它从来没有改变过。所有的方法会创建一个新的 Monad 实例而不是改变它。

Maybe 是这两个 Monad 的配对。如果一个值是非空的，它由 `Just(..)` 的实例来表示；如果该值是空的，它由 `Nothing()` 的实例来表示。注意，这里没有加强"空"的意思 —— 由你的代码决定。下一节会详细介绍这一点。

但是 Monad 的价值在于不论我们有 `Just(..)` 实例还是 `Nothing()` 实例，我们都会同样的使用它们。`Nothing()` 实例对所有的方法都有非操作的定义。所以如果 Monad 实例在 monadic 操作中暴露出来，它会有 short-circuting 的效果来忽略行为。

Maybe 抽象的力量是含蓄地封装了行为／无操作对偶性。（译者注：这句没看懂）

### 与众不同的 Maybe

JavaScript Maybe monad 的许多实现包含一个检查（通常在 `map(..)`中），就是检查该值是 `null` / `undefined`，如果是的话，就跳过。事实上，Maybe 被声称是有价值的，因为自动地封装了空值检查得以在某种程度上 short-circuting 了它的行为。

这是 Maybe 的代表性说明：

```js
// 代替不稳定的 `console.log( someObj.something.else.entirely )`：

Maybe.of( someObj )
.map( prop( "something" ) )
.map( prop( "else" ) )
.map( prop( "entirely" ) )
.map( console.log );
```

换句话说，如果我们在链式操作中的任何一环得到一个 `null` / `undefined` 值，Maybe 会神奇的切换到 no-op 模式 —— 它现在是一个 `Nothing()` monad 实例！ —— 把剩余的链式操作都停止掉。如果一些属性丢失或者是空的话，嵌套的属性访问能安全的抛出 JS 异常。这是非常酷的～ 这肯定是一个很有用的抽象！

但是，Maybe 不是一个纯 monad。

Monad 的核心精神说，它必须对所有的值都是有效的，不能对值做任何检查 —— 甚至是空值检查。所以为了方便，这些其他的实现都是走的捷径。这不是什么大不了的事情，但是当学习一些东西的时候，你应该先学习它的最纯粹的形式，然后再学习更复杂的规则。

我提供的 Maybe monad 的早期的实现不同于其他的 Maybe，就是它没有空置检查。所以，我们将 `Maybe` 作为 `Just(..)` / `Nothing()` 的松散配对。

等一下，如果我们没有得到自动 short-circuting，为什么 Maybe 有用呢？！？这似乎就是它的全部意义。

不要害怕，我们可以简单的提供空值检查，Maybe monad 剩下的 short-circuting 行为也还是可以很好的工作的。你可以在之前做一些 `someObj.something.else.entirely` 属性嵌套，但是我们可以做的更“得体”：

```js
function isEmpty(val) {
	return val === null || val === undefined;
}

var safeProp = curry( function safeProp(prop,obj){
	if (isEmpty( obj[prop] )) return Maybe.Nothing();
	return Maybe.of( obj[prop] );
} );

Maybe.of( someObj )
.chain( safeProp( "something" ) )
.chain( safeProp( "else" ) )
.chain( safeProp( "entirely" ) )
.map( console.log );
```

我们做了一个用于空值检查的 `safeProp(..)` 方法，并选择了一个 `Nothing()` Monad 实例。或者把值封装在 `Just(..)` 实例中（通过 `Maybe.of(..)`）。然后替代 `map(..)`，我们使用 `chain(..)`，它知道如何“展开” `safeProp(..)` 返回的 monad。

当遇到空值的时候，我们得到了相同的链式 short-circuiting。我们只是不把这个逻辑嵌入到 Maybe 中。

具体来说，Monad 的好处是，无论返回哪种类型 Monad， 我们的 `map(..)` 和 `chain(..)` 方法都有一个始终如一的和可预测的交互作用。

## Humble

现在我们对 Maybe 有了更多的了解以及它的作用是什么，我将会在它上面加一些小的改动 —— 在我们的讨论里加一些诙谐的元素 —— 通过发明 Maybe+Humble monad。从技术上来说，`Humble(..)` 并不是一个 Monad，而是一个产生 Maybe monad 实例的工厂函数。

Humble 是一个使用 Maybe 来跟踪 `egoLevel` 数字状态的数据结构包装器。具体来说，`Humble(..)` 只有在他们自身的水平值足够低（少于 `42`！）到被认为是 Humble 的时候才会执行生成的 Monad 实例；否则，它是一个无操作 `Nothing()`。这听起来和 Maybe 很像；真的非常相似！

这是一个 Maybe+Humble monad 工厂函数：

```js
function Humble(egoLevel) {
	// 接收任何大于等于 42 的数字
	return !(Number( egoLevel ) >= 42) ?
		Maybe.of( egoLevel ) :
		Maybe.Nothing();
}
```

你可能会注意到，这个工厂函数有点像 `safeProp(..)`，因为，它使用一个条件来决定是选择 `Just(..)` 还是 Maybe 的 `Nothing()`。

让我们来看一个基础用法的例子：


```js
var bob = Humble( 45 );
var alice = Humble( 39 );

bob.inspect();							// Nothing
alice.inspect();						// Just(39)
```

如果 Alice 赢得了一个大奖，现在是不是在为自己感到自豪呢？

```js
function winAward(ego) {
	return Humble( ego + 3 );
}

alice = alice.chain( winAward );
alice.inspect();						// Nothing
```

`Humble( 39 + 3 )` 创建并调用了一个从 `chain(..)` 返回的 `Nothing()` monad 实例，所以现在 Alice 不再有 Humble 的资格了。

现在，让我们一起使用一些 Monad ：

```js
var bob = Humble( 41 );
var alice = Humble( 39 );

var teamMembers = curry( function teamMembers(ego1,ego2){
	console.log( `Our humble team's egos: ${ego1} ${ego2}` );
} );

bob.map( teamMembers ).ap( alice );
// Humble 队列：41 39
```

由于 `teamMembers(..)` 是柯里化的，`bob.map(..)` 通过在 `bob` 自身级别（`41`）中调用，并且创建了一个封装着其他函数的 Monad 实例。在 *这个* Monad 中调用的 `ap(alice)` 调用`alice.map(..)`，并且传递给来自 Monad 的函数。这样做的效果是，Monad 的值已经提供给了 `teamMembers(..)` 函数，并且把显示的结果给打印了出来。

然而，如果一个 Monad 或者全部 Monad 实际上是 `Nothing()` 实例（因为它们本身的水平值太高了）：

```js
var frank = Humble( 45 );

bob.map( teamMembers ).ap( frank );

frank.map( teamMembers ).ap( bob );
```

`teamMembers(..)` 永远不会被调用（也没有信息被打印出来），因为，`frank` 是一个 `Nothing()` 实例。这就是 Maybe monad 的威力，我们的 `Humble(..)` 工厂函数允许我们根据自身的水平来选择。好酷！

### Humility

再来一个例子来说明 Maybe + Humble 数据结构的行为：

```js
function introduction() {
	console.log( "I'm just a learner like you! :)" );
}

var egoChange = curry( function egoChange(amount,concept,egoLevel) {
	console.log( `${amount > 0 ? "Learned" : "Shared"} ${concept}.` );
	return Humble( egoLevel + amount );
} );

var learn = egoChange( 3 );

var learner = Humble( 35 );

learner
.chain( learn( "closures" ) )
.chain( learn( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( learn( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Learned side effects.
// Learned recursion.
```

不幸的是，学习过程看起来已经缩短了。我发现学习一大堆东西而不和别人分享，会使自我太膨胀，这对你的技术是不利的。

让我们尝试一个更好的方法：

```js
var share = egoChange( -2 );

learner
.chain( learn( "closures" ) )
.chain( share( "closures" ) )
.chain( learn( "side effects" ) )
.chain( share( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( share( "recursion" ) )
.chain( learn( "map/reduce" ) )
.chain( share( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Shared closures.
// Learned side effects.
// Shared side effects.
// Learned recursion.
// Shared recursion.
// Learned map/reduce.
// Shared map/reduce.
// I'm just a learner like you! :)
```

在学习中分享。是学习学习更多并且能够学的更好的最佳方法。

## 摘要

总之，什么是 Monad ？

Monad 是一个值类型，一个接口，一个有封装行为的对象数据结构。

但是这些定义中没有一个是有用的。这是一个很好的尝试：Monad 是你怎么以一个更具有声明式的方式围绕一个值来组织行为。

和这本书中的其他一切一样，他们在有用的地方使用 Monad，但是不要因为每个人都在 FP 中讨论他们而使用他们。Monad 不是一个万能的银弹，但是他们确实提供了一些有用的实用函数。
