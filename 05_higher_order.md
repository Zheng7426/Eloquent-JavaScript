{{meta {load_files: ["code/scripts.js", "code/chapter/05_higher_order.js", "code/intro.js"], zip: "node/html"}}}

# Higher-Order Functions

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

李子和苏子正吹嘘他们最新程序的大小。“20万行”，李子说，“不包括注释！”苏子回答到，“切，我的已有大约百万行了”。师傅马原说，“我最好的程序有 500行字。”听到这个，李子和苏子豁然开朗。

quote}}

if}}

{{quote {author: "C.A.R. Hoare", title: "1980 ACM Turing Award Lecture", chapter: true}

{{index "Hoare, C.A.R."}}

构建软件设计有两种方法：一种是让程序非常简洁、一目了然，却仍找不到任何明显的瑕疵。另一种是让程序及其复杂，所以找不到任何明显的瑕疵。

quote}}

{{figure {url: "img/chapter_picture_5.jpg", alt: "Letters from different scripts", chapter: true}}}

{{index "program size"}}

一个大程序是非常昂贵的，而且不只因为其所花费的开发时间。程序的大小和((复杂度))密切相关，然而复杂度只会迷惑程序员。于是被迷惑的程序员写出错误（_((bug))s_）的代码。所以一个大程序是一个给 bugs 提供了足够隐藏空间的平台，让它们难以被找到。

{{index "summing example"}}

让我们回顾下开导篇的两个例题程序。第一个是个6行的独立程序。

```
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
```

第二个需要两个额外的函数，但本身只有一行。

```
console.log(sum(range(1, 10)));
```

哪个程序更容易出错？

{{index "program size"}}

如果我们算上 `sum` 和 `range` 的大小，那么第二个程序也很大——甚至比第一个还大。尽管如此，我依旧相信第一个程序更容易出错。

{{index [abstraction, "with higher-order functions"], "domain-specific language"}}

第二个程序更大可能正确是因为它的答案是以符合原题的((单词))的形式表达的。一个范围内的（range）数字的和（sum）其实和循环、计步器无关。它完全是关于范围（range）以及和（sum）的。

尽管这个单词的定义（`sum` 和 `range` 函数）依旧需要循环、计步器、和其他附加细节。但因为它们以更简单的形式，而不是整体，展现出来，所以更容易正确。

## Abstraction

在编程的世界，这些单词通常被叫做_((抽象化))_。抽象化把细节隐藏起来，使我们可以在更高层面（更抽象化的）探讨问题。

{{index "recipe analogy", "pea soup"}}

打个比方，对比下面两个豌豆汤的食谱。第一个如下：

{{quote

每人一杯干豌豆放入容器中。加水没至豌豆，泡至少12小时。从水中取出豌豆，放入锅中。每人四杯水加入锅中。盖上锅盖，炖2小时。每人半个洋葱。用刀切块，加入豆中。每人一个芹菜茎，切块，加入豆中。每人一个胡萝卜，用刀切块，加入豆中。再煮10分钟。

quote}}

第二个食谱如下：

{{quote

每人：一杯干豌豆，半个切块的圆葱，一个芹菜茎，一个胡萝卜。

豌豆浸泡12小时。在4杯水（每人）中炖2小时。加入切块的蔬菜。在煮10分钟。

quote}}

{{index vocabulary}}

第二个更简短易懂。但你需要理解一些做饭相关的单词：_浸泡_，_炖_，_切块_，以及_蔬菜_。

编程时，我们不能依赖所有所需单词都在我们的字典中。因此，我们可能会陷入第一个菜谱——把电脑需要运行的每一步逐次写下，再混入它们代表的高层概念中。

{{index abstraction}}

编程中，发现你何时徘徊在低层抽象化中的技巧很有用。

## Abstracting repetition

{{index [array, iteration]}}

我们上面看到的纯函数是一种不错的抽象化方法。然而它们不是万能的。

{{index "for loop"}}

一个程序经常重复着做一件事情若干次。比如我们熟知的 `for`((循环))：

```
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

那我们可以将“做一件事情 _N_ 次”抽象成一个函数么？如果仅是一个重复 _N_ 次的 `console.log` 的话，是轻而易举的：

```
function repeatLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

{{index [function, "higher-order"], loop, [function, "as value"]}}

{{indexsee "higher-order function", "function, higher-order"}}

但如果我们想打印其他非数字的东西呢？“做件事”可以用函数表示，而函数也是值，所以我们可以将想做的事情当成函数值。

```{includeCode: "top_lines: 5"}
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log);
// → 0
// → 1
// → 2
```

我们不需要给 `repeat` 提供一个已定义的函数。通常，当场创建一个函数值更简单。

```
let labels = [];
repeat(5, i => {
  labels.push(`Unit ${i + 1}`);
});
console.log(labels);
// → ["Unit 1", "Unit 2", "Unit 3", "Unit 4", "Unit 5"]
```

{{index "loop body", [braces, body], [parentheses, arguments]}}

这个的结构类似于 `for` 循环——先定义循环的类别，在提供块。不过，这里的块是调用 `repeat` 的括号包围住的函数值。所以它需要两个右括号（分别为 `}` 和 `)`）。这种以一句简介的表达式为块的函数也可以去掉大括号，简化为一行。

## Higher-order functions

{{index [function, "higher-order"], [function, "as value"]}}

在其他函数上运行的函数，不论是作为参数还是返回的值，又叫_高阶函数_。函数实际上就是普通的值，因此高阶函数的存在并无令人诧异之处。它的命名来自对函数和其他值间的区别更为重视的((数学))。

{{index abstraction}}

高阶函数使我们不仅可以抽象化值，还可以抽象化_操作_。它们有几种形式。比如，我们可以用函数创建新的函数。

```
function greaterThan(n) {
  return m => m > n;
}
let greaterThan10 = greaterThan(10);
console.log(greaterThan10(11));
// → true
```

我们也可以用函数改变其他函数。

```
function noisy(f) {
  return (...args) => {
    console.log("calling with", args);
    let result = f(...args);
    console.log("called with", args, ", returned", result);
    return result;
  };
}
noisy(Math.min)(3, 2, 1);
// → calling with [3, 2, 1]
// → called with [3, 2, 1] , returned 1
```

我们甚至可以用函数提供新的((控制流))。

```
function unless(test, then) {
  if (!test) then();
}

repeat(3, n => {
  unless(n % 2 == 1, () => {
    console.log(n, "is even");
  });
});
// → 0 is even
// → 2 is even
```

{{index [array, methods], [array, iteration], "forEach method"}}

有一个内置的数组方法 `forEach` 提供类似 `for`/`of` 循环的高阶函数。

```
["A", "B"].forEach(l => console.log(l));
// → A
// → B
```

## Script data set

高阶函数大放异彩的领域之一是数据处理。首先我们需要一些数据。本章会用一个关于文字的((数据集))——比如拉丁文、西里尔文、或者阿拉伯文。

[第一章](values#unicode)中我们讲过((Unicode))，一个为每个字符分配一个数字的系统。大部分的字符都和一个特定的文字有关。标准的包括 140个不同文字，其中 81个今天仍在使用，59个已成为历史。

尽管我经常只读拉丁文字，我依旧感激其他80余个仍被世人使用的文字，尽管大部分我根本无法识别。比如，下面是用((泰米尔语))手写的一段文字：

{{figure {url: "img/tamil.png", alt: "Tamil handwriting"}}}

{{index "SCRIPTS data set"}}

例题中的((数据集))包括了一些关于 Unicode 中定义的 140个文字。可以在本章[([_https://eloquentjavascript.net/code#5_](https://eloquentjavascript.net/code#5))]{if book} [沙盒](https://eloquentjavascript.net/code#5)中的 `SCRIPTS` 变量中找到。该变量是一个对象数组，每个对象描述了一个文字。


```{lang: "application/json"}
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

这种对象告诉我们这个文字的名字，对应的 Unicdoe 范围，书写的方向，（大体）出现时间，如今是否仍在使用中，以及一个可以获取更多信息的链接。书写方向可以是 `"ltr"`，即从左到右；`"rtl"`，从右到左（比如阿拉伯文或希伯来文）；或`"ttb"`，从上到下（如蒙古文）。

{{index "slice method"}}

`ranges`属性是一个 Unicode 范围的数组，其中每个都是一个二元素的数组，分别代表下界和上界。每个在这个范围内的字符码都属于该文字。该范围包括下((界))（994 是一个科普特字母），但不包括上界（1008 不是）。

## Filtering arrays

{{index [array, methods], [array, filtering], "filter method", [function, "higher-order"], "predicate function"}}

下面的函数可帮助你找寻数据集中仍在使用的文字。它会过滤掉那些不符合要求的数组元素。

```
function filter(array, test) {
  let passed = [];
  for (let element of array) {
    if (test(element)) {
      passed.push(element);
    }
  }
  return passed;
}

console.log(filter(SCRIPTS, script => script.living));
// → [{name: "Adlam", …}, …]
```

{{index [function, "as value"], [function, application]}}

该元素用一个名为 `test` 的参数，是一个函数值，来完成计算中的空白——决定保留哪个元素的过程。

{{index "filter method", "pure function", "side effect"}}

请注意 `filter` 函数，与其从原数组中删除元素，它会创建一个新的数组储存原数组中符合要求的元素。因此它是一个_纯_函数。它不会更改参数。

`forEach` 和 `filter` 都是((标准))数组方法。上面的例子中将它们的实际定义写了出来。从现在起，我们会以下面的方法调用它们：

```
console.log(SCRIPTS.filter(s => s.direction == "ttb"));
// → [{name: "Mongolian", …}, …]
```

{{id map}}

## Transforming with map

{{index [array, methods], "map method"}}

假设我们有一个从 `SCRIPTS` 变量中过滤出来的文字对象的数组。但为了方便，我们只想要一个名字数组。

{{index [function, "higher-order"]}}

`map` 方法可以对所有元素调用同一个函数，并返回一个新的数组。返回的新数组和原数组的长度相同，但其内容是被函数所_映射过_的新值。

```
function map(array, transform) {
  let mapped = [];
  for (let element of array) {
    mapped.push(transform(element));
  }
  return mapped;
}

let rtlScripts = SCRIPTS.filter(s => s.direction == "rtl");
console.log(map(rtlScripts, s => s.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", …]
```

同 `forEach` 以及 `filter` 一样，`map` 也是一个标准的数组方法。

## Summarizing with reduce

{{index [array, methods], "summing example", "reduce method"}}

另一个常用的数组操作是将它们变成一个值。我们的老朋友，一个数字集的和，就是一个例子。另一个例子是找到字符最多的文字。

{{indexsee "fold", "reduce method"}}

{{index [function, "higher-order"], "reduce method"}}

表示这种高级操作的函数叫 _缩减_（有些时候也叫 _fold_）。它通过反复从数组中提取一个元素，并将其于现有的值结和，从而得到一个新值。在计算数字和时，我们从第一个数字开始，逐一添加数字至和。

`reduce` 的参数包括，除了数组外，一个组合函数和一个起始值。这个函数比 `filter` 和 `map` 略微复杂，让我们细看一下：

```
function reduce(array, combine, start) {
  let current = start;
  for (let element of array) {
    current = combine(current, element);
  }
  return current;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

{{index "reduce method", "SCRIPTS data set"}}

上面就是标准的数组方法 `reduce`，它有一个附赠的便利。如果你的数组包含至少一个元素，你就可以不提供 `start` 参数。这个方法会将数组中的一个元素作为它的起始值，并从第二个元素开始缩减。

```
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

{{index maximum, "characterCount function"}}

我们可以用下面的方法，使用 `reduce`（两次）来找寻最多字符的文字：

```
function characterCount(script) {
  return script.ranges.reduce((count, [from, to]) => {
    return count + (to - from);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return characterCount(a) < characterCount(b) ? b : a;
}));
// → {name: "Han", …}
```

函数 `characterCount` 负责通过缩减范围计算出每个文字的字符总和。注意改缩减函数的参数中解构赋值的使用。第二个 `reduce` 调用则通过反复比较两个文字的字符总和来找寻字符数量最多的文字。最后返回最大的那个。

在 Unicode 中，汉字有超过 89000 个文字，所以它是该数据集中拥有最多字符的文字。汉字是个（有时）用于中文、日文、和韩文的文字。这些语言分享很多字符，尽管它们的书写方式不一样。（位于美国的）Unicode 集团决定将它们归纳于同一个书写系统，从而节省字符码。这个叫做_汉字统一_，尽管很多人对此很不满意。

## Composability

{{index loop, maximum}}

试想一下，在没有高阶函数的情况下，我们如何编写上面的例题（找寻最大的文字）。其代码其实没有那么夸张：

```{test: no}
let biggest = null;
for (let script of SCRIPTS) {
  if (biggest == null ||
      characterCount(biggest) < characterCount(script)) {
    biggest = script;
  }
}
console.log(biggest);
// → {name: "Han", …}
```

尽管我们需要更多的变量，且该程序也多了四行。但总体而言还是简洁明了的。

{{index "average function", composability, [function, "higher-order"], "filter method", "map method", "reduce method"}}

{{id average_function}}

高阶函数在你需要_编写_操作时大放异彩。比如，让我们写一个找寻数据集中仍在使用的和停止使用的文字的平均起源年份。

```
function average(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(average(
  SCRIPTS.filter(s => s.living).map(s => s.year))));
// → 1165
console.log(Math.round(average(
  SCRIPTS.filter(s => !s.living).map(s => s.year))));
// → 204
```

平均来讲，已停止使用的文字的寿命比仍在使用的文字寿命要长。当然这并不是一个有意义或令人惊讶的数据。但我希望你也认同上面的运算代码并不复杂。你可以把它想象成一个传递途径：我们从所有文字开始，过滤掉正在使用（或者已停止使用）的，记录这些文字的年份，算它们的平均数，四舍五入最后的结果。

你当然也可以把它写成一个庞大的((循环))。

```
let total = 0, count = 0;
for (let script of SCRIPTS) {
  if (script.living) {
    total += script.year;
    count += 1;
  }
}
console.log(Math.round(total / count));
// → 1165
```

但这个相对而言较难看出它在做什么。而且因为中间的结果并不是连贯的值，反而需要更多工作将 `average` 提取到独立的函数中。

{{index efficiency, [array, creation]}}

至于电脑的实际运行情况而言，这两个方法也截然不同。第一个会在运行 `filter` 和 `map` 时创建新的数组，而第二个则减少工作量只会计算一些数字。通常我们可以选择简而易懂的方法，但如果反复处理大量数组，那么第二种方法的效率更高。

## Strings and character codes

{{index "SCRIPTS data set"}}

该数据集的一个用途是判断一篇文章用了什么文字。让我们一起谱写该程序。

还记得每个文字都有个字符码范围的数组么？所以已知一个字符码，我们可以通过下面的函数找到它所对应的文字（如果存在的话）：

{{index "some method", "predicate function", [array, methods]}}

```{includeCode: strip_log}
function characterScript(code) {
  for (let script of SCRIPTS) {
    if (script.ranges.some(([from, to]) => {
      return code >= from && code < to;
    })) {
      return script;
    }
  }
  return null;
}

console.log(characterScript(121));
// → {name: "Latin", …}
```

`some` 方法也是一个高阶函数。它的参数是一个判断函数，并告诉你数组中的哪个元素符合该函数的判断条件。

{{id code_units}}

但我们如何从字符串中得到字符码？

在[第一章](values)中，我说过 JavaScript 的((字符串))是一个16位数字序列。它们叫做_((码单位))_。一个 ((Unicode)) ((字符))码单位最开始应该在这个单位中（差不多 65000 个字符）。当事实证明这些字符远不够用时，有些人在是否需要对每个字符使用额外内存中犹豫不决。为了解决这些问题，发明了 ((UTF-16))，JavaScript 字符串的格式。它使用单个 16位码单位描述最常见的字符，但用一对码单位描述其他字符。

{{index error}}

如今普遍认为 UTF-16 是个坏主意。它几乎是故意为引起错误而设计的。我们很容易写一个代码将一个码单位等同于一个字符。如果遇到一个不使用一对码单位的文字，这个程序不会遇到任何问题。一旦将其用于双码单位的((汉字))中，它将不会工作。好在，随着((表情符合))的诞生，所有人都开始使用双码单位，因此处理此类问题的负担较为平摊了。

{{index [string, length], [string, indexing], "charCodeAt method"}}

不幸的是，JavaScript 字符串的普遍操作，比如通过 `length` 属性得到它们的长度，或者通过中括号得到它们的内容，只对单码单位有效。

```{test: no}
// 两个表情符合：马和鞋
let horseShoe = "🐴👟";
console.log(horseShoe.length);
// → 4
console.log(horseShoe[0]);
// → (无效的半字符)
console.log(horseShoe.charCodeAt(0));
// → 55357 (这个半字符所对应的字符码)
console.log(horseShoe.codePointAt(0));
// → 128052 (马的表情符号实际对应的字符码)
```

{{index "codePointAt method"}}

JavaScript 的 `charCodeAt` 方法返回一个码单位，而不是一个完整的字符码。而后加入的 `codePointAt` 方法则返回一个完整的 Unicode 字符。所以我们可以通过它读取字符串中的字符。但 `codePointAt` 的参数依旧是一个码单位序列的索引。因此我们依旧需要解决一个字符是单码还是双码的问题，才能逐一读取单个字符。

{{index "for/of loop", character}}

[上一章](data#for_of_loop)中，我说过 `for`/`of` 循环可以用于字符串。类似 `codePointAt`，这类循环也是在大家发现 UTF-16 的问题后添加的。因此用它遍历一个字符串，它会读取真正的字符，而不是单个的码单位。

```
let roseDragon = "🌹🐉";
for (let char of roseDragon) {
  console.log(char);
}
// → 🌹
// → 🐉
```

如果你有一个字符（一个单码或双码的字符串），你可以通过 `codePointAt(0)` 得到其对于的编码。

## Recognizing text

{{index "SCRIPTS data set", "countBy function", [array, counting]}}

我们有一个 `characterScript` 函数和一个可以正确读取字符的方法。下一步就是统计出现在每个文字中的字符了。下面的抽象化的统计函数会帮助到你：

```{includeCode: strip_log}
function countBy(items, groupName) {
  let counts = [];
  for (let item of items) {
    let name = groupName(item);
    let known = counts.findIndex(c => c.name == name);
    if (known == -1) {
      counts.push({name, count: 1});
    } else {
      counts[known].count++;
    }
  }
  return counts;
}

console.log(countBy([1, 2, 3, 4, 5], n => n > 2));
// → [{name: false, count: 2}, {name: true, count: 3}]
```

`countBy` 函数的参数是一个集（任何可以被 `for`/`of` 遍历的数据结构）和一个为给定元素计算组名的函数。它返回一个对象数组，每个都有一个组名和该组中存在的元素数量。

{{index "findIndex method", "indexOf method"}}

它调用另一个数组方法：`findIndex`。该方法类似 `indexOf`，但与其找一个特定的值，它找第一个符合给定函数要求的值。和 `indexOf` 一样，如果该元素不存在，它会返回 -1。

{{index "textScripts function", "Chinese characters"}}

通过 `countBy`，我们可以编写一个告诉我们哪个文字出现在一篇文章中的函数。

```{includeCode: strip_log, startCode: true}
function textScripts(text) {
  let scripts = countBy(text, char => {
    let script = characterScript(char.codePointAt(0));
    return script ? script.name : "none";
  }).filter(({name}) => name != "none");

  let total = scripts.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No scripts found";

  return scripts.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(textScripts('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

{{index "characterScript function", "filter method"}}

该函数首先用 `characterScript` 给每个字符一个名字或者 `"none"` 如果它不属于任何文字，在根据其名字进行统计。通过 `filter` 排除所以名为 `"none"` 的文字，因为我们对这些字符没兴趣。

{{index "reduce method", "map method", "join method", [array, methods]}}

为了计算((百分比))，我们首先需要通过 `reduce` 计算出一个文字中所有字符的数量。如果这个字符不存在，该函数返回一个特殊的字符串。否则，它通过 `map` 将统计的结果变成可读的字符串，在通过 `join` 将他们组合在一起。

## Summary

将函数值传递给其他函数是 JavaScript 中非常重要的存在。它允许我们编写带有“空隙”的计算模型。而这些空隙则由调用该函数的代码提供。

数组提供许多高阶方法。你可以用 `forEach` 遍历一个数组中的元素。`filter` 方法返回一个只包括符合((判断函数))的新数组。你可以通过 `map` 改变一个数组中的每个元素。也可以用 `reduce` 将数组中所有元素结合成一个值。而 `some` 则测试该数组中有没有符合判断函数条件的元素。最后，`findIndex` 会找到第一个符合判断条件的元素的索引。

## Exercises

### Flattening

{{index "flattening (exercise)", "reduce method", "concat method", [array, flattening]}}

结合 `reduce` 和 `concat` 将一个嵌套数组“展平”成一个含有所有元素的单个数组。

{{if interactive

```{test: no}
let arrays = [[1, 2, 3], [4, 5], [6]];
// 你的代码
// → [1, 2, 3, 4, 5, 6]
```
if}}

### Your own loop

{{index "your own loop (example)", "for loop"}}

写一个类似 `for` 的高阶函数 `loop`。它的参数包括一个值，一个判断函数，一个更新函数，和一个块函数。每次迭代，它首先对现有值运行判断函数，并停止如果其结果是 false。之后，它会调用块函数，提供现有值。最后它调用更新函数去创建一个新的值，并重新来过。

在定义该函数时，你可以利用一个正常的循环来进行遍历。

{{if interactive

```{test: no}
// 你的代码

loop(3, n => n > 0, n => n - 1, console.log);
// → 3
// → 2
// → 1
```

if}}

### Everything

{{index "predicate function", "everything (exercise)", "every method", "some method", [array, methods], "&& operator", "|| operator"}}

类似 `some` 方法，数组也有一个 `every` 方法。该方法返回 true。该函数只有在数组中_所有_元素都符合判断是才会返回 true。换句话说，`some` 类似于数组中的 `||`，而 `every` 则是 `&&`。

设计一个 `every` 函数，参数为一个数组和一个判断函数。写两个版本，第一个用循环，第二个用 `some` 方法。

{{if interactive

```{test: no}
function every(array, test) {
  // 你的代码
}

console.log(every([1, 3, 5], n => n < 10));
// → true
console.log(every([2, 4, 16], n => n < 10));
// → false
console.log(every([], n => n < 10));
// → true
```

if}}

{{hint

{{index "everything (exercise)", "short-circuit evaluation", "return keyword"}}

类似 `&&`，`every` 可以在遇到第一个不符合要求的元素时就停止运行。所以循环的方法，可以在发现不符合判断条件的元素的第一时间用 `break` 或者 `return` 跳出循环。如果遍历的每一个元素后，依旧没有找到不符合要求的。那么我们可以返回 `true`。

在 `some` 的基础上编写 `every`，我们可以利用_((德摩根定律))_：`a && b` 等于 `!(!a || !b)`。这个可以推广到数组：数组中所有元素都符合要求，如果没有元素不符合要求。

hint}}

### Dominant writing direction

{{index "SCRIPTS data set", "direction (writing)", "groupBy function", "dominant direction (exercise)"}}

写一个程序计算一篇文字中的主要书写方向。每个文字对象有一个 `direction` 属性，它的值只能是 `"ltr"`（从左到右）、`"rtl"`（从右到左）、和 `"ttb"`（从上到下）。

{{index "characterScript function", "countBy function"}}

主要书写方向是有文章中大部分字符的方向所决定的。前面提到的 `characterScript` 和 `countBy` 可以帮助你。

{{if interactive

```{test: no}
function dominantDirection(text) {
  // 你的代码
}

console.log(dominantDirection("Hello!"));
// → ltr
console.log(dominantDirection("Hey, مساء الخير"));
// → rtl
```
if}}

{{hint

{{index "dominant direction (exercise)", "textScripts function", "filter method", "characterScript function"}}

你的答案应该和 `textScript` 的前半部相似。你依旧需要利用 `characterScript` 统计文章中出现的字符，在过滤掉没有兴趣的（不符合任何文字的）字符。

{{index "reduce method"}}

可以通过 `reduce` 找寻最多字符的书写方法。可以参考本章的例题，看看我们是如何利用 `reduce` 找寻最多字符的文字的。

hint}}
