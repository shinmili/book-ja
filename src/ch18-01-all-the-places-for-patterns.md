<!--
## All the Places Patterns Can Be Used
-->

## パターンが使用されることのある箇所全部

<!--
Patterns pop up in a number of places in Rust, and you’ve been using them a lot
without realizing it! This section discusses all the places where patterns are
valid.
-->

Rustにおいて、パターンはいろんな箇所に出現し、そうと気づかないうちにたくさん使用してきました！
この節は、パターンが合法な箇所全部を議論します。

<!--
### `match` Arms
-->

### `match`アーム

<!--
As discussed in Chapter 6, we use patterns in the arms of `match` expressions.
Formally, `match` expressions are defined as the keyword `match`, a value to
match on, and one or more match arms that consist of a pattern and an
expression to run if the value matches that arm’s pattern, like this:
-->

第6章で議論したように、パターンを`match`式のアームで使います。正式には、`match`式はキーワード`match`、
マッチ対象の値、パターンとそのアームのパターンに値が合致したら実行される式からなる1つ以上のマッチアームとして定義されます。
以下のように:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

<!--
For example, here's the `match` expression from Listing 6-5 that matches on an
`Option<i32>` value in the variable `x`:
-->

例えば、以下はリスト6-5の変数`x`内の`Option<i32>`値に対してマッチする`match`式です:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

<!--
The patterns in this `match` expression are the `None` and `Some(i)` on the
left of each arrow.
-->

この`match`式に含まれるパターンは、各矢印の左側にある`Node`と`Some(i)`です。

<!--
One requirement for `match` expressions is that they need to be *exhaustive* in
the sense that all possibilities for the value in the `match` expression must
be accounted for. One way to ensure you’ve covered every possibility is to have
a catchall pattern for the last arm: for example, a variable name matching any
value can never fail and thus covers every remaining case.
-->

`match`式の必須事項の1つは、`match`式の値の可能性全てが考慮されなければならないという意味で*網羅的*である必要があることです。
全可能性をカバーしていると保証する1つの手段は、最後のアームに包括的なパターンを入れることです:
例えば、どんな値にも合致する変数名は失敗することがあり得ないので、故に残りの全ケースをカバーできます。

<!--
The particular pattern `_` will match anything, but it never binds to a
variable, so it’s often used in the last match arm. The `_` pattern can be
useful when you want to ignore any value not specified, for example. We’ll
cover the `_` pattern in more detail in the [“Ignoring Values in a
Pattern”][ignoring-values-in-a-pattern] section later in this
chapter.
-->

`_`という特定のパターンは何にでもマッチしますが、変数には束縛されないので、よく最後のマッチアームに使用されます。
例えば、`_`パターンは、指定されていないあらゆる値を無視したい時に有用です。
`_`パターンについて詳しくは、この章の後ほど、[「パターンの値を無視する」][ignoring-values-in-a-pattern]節で講義します。

<!--
### Conditional `if let` Expressions
-->

### 条件分岐`if let`式

<!--
In Chapter 6 we discussed how to use `if let` expressions mainly as a shorter
way to write the equivalent of a `match` that only matches one case.
Optionally, `if let` can have a corresponding `else` containing code to run if
the pattern in the `if let` doesn’t match.
-->

第6章で主に`if let`式を1つの場合にしか合致しない`match`と同様のものを書く省略法として使用する方法を議論しました。
オプションとして、`if let`には`if let`のパターンが合致しない時に走るコードを含む対応する`else`も用意できます。

<!--
Listing 18-1 shows that it’s also possible to mix and match `if let`, `else
if`, and `else if let` expressions. Doing so gives us more flexibility than a
`match` expression in which we can express only one value to compare with the
patterns. Also, Rust doesn't require that the conditions in a series of `if
let`, `else if`, `else if let` arms relate to each other.
-->

リスト18-1は、`if let`、`else if`、`else if let`式を混ぜてマッチさせることもできることを示しています。
そうすると、パターンと1つの値しか比較することを表現できない`match`式よりも柔軟性が高くなります。
また、一連の`if let`、`else if`、`else if let`アームの条件は、お互いに関連している必要はありません。

<!--
The code in Listing 18-1 determines what color to make your background based on
a series of checks for several conditions. For this example, we’ve created
variables with hardcoded values that a real program might receive from user
input.
-->

リスト18-1のコードは、いくつかの条件を連続してチェックし、それに応じて背景を何色にするかを決定しています。
この例では、実際のプログラムではユーザ入力を受け付ける可能性のある変数をハードコードされた値で生成しています。

<!--
<span class="filename">Filename: src/main.rs</span>
-->

<span class="filename">ファイル名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-01/src/main.rs}}
```

<!--
<span class="caption">Listing 18-1: Mixing `if let`, `else if`, `else if let`,
and `else`</span>
-->

<span class="caption">リスト18-1: `if let`、`else if`、`else if let`、`else`を混ぜる</span>

<!--
If the user specifies a favorite color, that color is used as the background.
If no favorite color is specified and today is Tuesday, the background color is
green. Otherwise, if the user specifies their age as a string and we can parse
it as a number successfully, the color is either purple or orange depending on
the value of the number. If none of these conditions apply, the background
color is blue.
-->

ユーザがお気に入りの色を指定したら、その色が背景として使われます。
お気に入りの色が指定されておらず、今日が火曜日なら、背景色は緑です。
もしそうではなく、ユーザが年齢を文字列で指定し、数値として解析することができたら、
背景色は、その数値によって紫かオレンジになります。
どの条件も適用できなければ、背景色は青になります。

<!--
This conditional structure lets us support complex requirements. With the
hardcoded values we have here, this example will print `Using purple as the
background color`.
-->

この条件分岐構造により、複雑な要件をサポートさせてくれます。ここにあるハードコードされた値では、
この例は`Using purple as the background color`と出力するでしょう。

<!--
You can see that `if let` can also introduce shadowed variables in the same way
that `match` arms can: the line `if let Ok(age) = age` introduces a new
shadowed `age` variable that contains the value inside the `Ok` variant. This
means we need to place the `if age > 30` condition within that block: we can’t
combine these two conditions into `if let Ok(age) = age && age > 30`. The
shadowed `age` we want to compare to 30 isn’t valid until the new scope starts
with the curly bracket.
-->

`match`アームのように`if let`もシャドーイングされた変数を導入できることがわかります:
`if let Ok(age) = age`の行は、`Ok`列挙子の中の値を含むシャドーイングされた新しい`age`変数を導入します。
つまり、`if age > 30`という条件は、そのブロック内に配置する必要があります: これら2つの条件を組み合わせて、
`if let Ok(age) = age && age > 30`とすることはできません。30と比較したいシャドーイングされた`age`は、
波括弧で新しいスコープが始まるまで有効にならないのです。

<!--
The downside of using `if let` expressions is that the compiler doesn’t check
for exhaustiveness, whereas with `match` expressions it does. If we omitted the
last `else` block and therefore missed handling some cases, the compiler would
not alert us to the possible logic bug.
-->

`if let`式を使うことの欠点は、コンパイラが網羅性を確認してくれないことです。一方で`match`式ではしてくれます。
最後の`else`ブロックを省略して故に、扱い忘れたケースがあっても、コンパイラは、ロジックバグの可能性を指摘してくれないでしょう。

<!--
### `while let` Conditional Loops
-->

### `while let`条件分岐ループ

<!--
Similar in construction to `if let`, the `while let` conditional loop allows a
`while` loop to run for as long as a pattern continues to match. In Listing
18-2 we code a `while let` loop that uses a vector as a stack and prints the
values in the vector in the opposite order in which they were pushed.
-->

`if let`と構成が似て、`while let`条件分岐ループは、パターンが合致し続ける限り、`while`ループを走らせます。
リスト18-2では、ベクタをスタックとして使用する`while let`ループを書き、
ベクタの値をプッシュしたのとは逆順に出力します。

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-02/src/main.rs:here}}
```

<!--
<span class="caption">Listing 18-2: Using a `while let` loop to print values
for as long as `stack.pop()` returns `Some`</span>
-->

<span class="caption">リスト18-2: `while let`ループを使って`stack.pop()`が`Some`を返す限り値を出力する</span>

<!--
This example prints 3, 2, and then 1. The `pop` method takes the last element
out of the vector and returns `Some(value)`. If the vector is empty, `pop`
returns `None`. The `while` loop continues running the code in its block as
long as `pop` returns `Some`. When `pop` returns `None`, the loop stops. We can
use `while let` to pop every element off our stack.
-->

この例は、3, 2, そして1と出力します。`pop`メソッドはベクタの最後の要素を取り出して`Some(value)`を返します。
ベクタが空なら、`pop`は`None`を返します。`while`ループは`pop`が`Some`を返す限り、ブロックのコードを実行し続けます。
`pop`が`None`を返すと、ループは停止します。`while let`を使用してスタックから全ての要素を取り出せるのです。

<!--
### `for` Loops
-->

### `for`ループ

<!--
In a `for` loop, the value that directly follows the keyword `for` is a
pattern. For example, in `for x in y` the `x` is the pattern. Listing 18-3
demonstrates how to use a pattern in a `for` loop to destructure, or break
apart, a tuple as part of the `for` loop.
-->

`for`ループにおいて、キーワード`for`の直後に続く値はパターンです。
例えば、`for x in y`では、`x`がパターンになります。
リスト18-3は`for`ループでパターンを使用して`for`ループの一部としてタプルを分配あるいは、分解する方法をデモしています。

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-03/src/main.rs:here}}
```

<!--
<span class="caption">Listing 18-3: Using a pattern in a `for` loop to
destructure a tuple</span>
-->

<span class="caption">リスト18-3: `for`ループでパターンを使用してタプルを分配する</span>

<!--
The code in Listing 18-3 will print the following:
-->

リスト18-3のコードは、以下のように出力するでしょう:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-03/output.txt}}
```

<!--
We adapt an iterator using the `enumerate` method so it produces a value and
the index for that value, placed into a tuple. The first value produced is the
tuple `(0, 'a')`. When this value is matched to the pattern `(index, value)`,
`index` will be `0` and `value` will be `'a'`, printing the first line of the
output.
-->

`enumerate`メソッドを使用して、値とその値の添え字をタプルに配置して生成するようにイテレータを調整しています。
最初に生成される値はタプル`(0, 'a')`です。この値がパターン`(index, value)`とマッチさせられると、
`index`は`0`、`value`は`'a'`になり、出力の最初の行を出力するのです。

<!--
### `let` Statements
-->

### `let`文

<!--
Prior to this chapter, we had only explicitly discussed using patterns with
`match` and `if let`, but in fact, we’ve used patterns in other places as well,
including in `let` statements. For example, consider this straightforward
variable assignment with `let`:
-->

この章に先駆けて、`match`と`if let`でパターンを使用することだけ明示的に議論してきましたが、
実は`let`文を含む他の箇所でもパターンを使用してきたのです。例として、この`let`での率直な変数代入を考えてください:

```rust
let x = 5;
```

<!--
Every time you've used a `let` statement like this you've been using patterns,
although you might not have realized it! More formally, a `let` statement looks
like this:
-->

お気付きではなかったかもしれませんが、このように`let`文を使うときは毎回、パターンを使っていたのです！
より正式には、`let`文はこんな見た目をしています:

```text
let PATTERN = EXPRESSION;
```

<!--
In statements like `let x = 5;` with a variable name in the `PATTERN` slot, the
variable name is just a particularly simple form of a pattern. Rust compares
the expression against the pattern and assigns any names it finds. So in the
`let x = 5;` example, `x` is a pattern that means “bind what matches here to
the variable `x`.” Because the name `x` is the whole pattern, this pattern
effectively means “bind everything to the variable `x`, whatever the value is.”
-->

`let x = 5;`のような変数名が`PATTERN`スロットにある文で、変数名は、ただ特に単純な形態のパターンなのです。
Rustは式をパターンと比較し、見つかったあらゆる名前を代入します。故に、`let x = 5;`の例では、
`x`は「ここでマッチしたものを変数`x`に束縛する」ことを意味するパターンです。
名前`x`がパターンの全容なので、このパターンは実質的に「値が何であれ、全てを変数`x`に束縛しろ」を意味します。

<!--
To see the pattern matching aspect of `let` more clearly, consider Listing
18-4, which uses a pattern with `let` to destructure a tuple.
-->

`let`のパターンマッチングの観点をよりはっきり確認するためにリスト18-4を考えてください。
これは`let`でパターンを使用し、タプルを分配します。

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-04/src/main.rs:here}}
```

<!--
<span class="caption">Listing 18-4: Using a pattern to destructure a tuple and
create three variables at once</span>
-->

<span class="caption">リスト18-4: パターンを使用してタプルを分配し、3つの変数を一度に生成する</span>

<!--
Here, we match a tuple against a pattern. Rust compares the value `(1, 2, 3)`
to the pattern `(x, y, z)` and sees that the value matches the pattern, so Rust
binds `1` to `x`, `2` to `y`, and `3` to `z`. You can think of this tuple
pattern as nesting three individual variable patterns inside it.
-->

ここでタプルに対してパターンをマッチさせています。Rustは値`(1, 2, 3)`をパターン`(x, y, z)`と比較し、
値がパターンに合致すると確認するので、`1`を`x`に、`2`を`y`に、`3`を`z`に束縛します。
このタプルパターンを個別の3つの変数パターンが内部にネストされていると考えることもできます。

<!--
If the number of elements in the pattern doesn’t match the number of elements
in the tuple, the overall type won’t match and we’ll get a compiler error. For
example, Listing 18-5 shows an attempt to destructure a tuple with three
elements into two variables, which won’t work.
-->

パターンの要素数がタプルの要素数と一致しない場合、全体の型が一致せず、コンパイルエラーになるでしょう。
例えば、リスト18-5は、3要素のタプルを2つの変数に分配しようとしているところを表示していて、動きません。

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-05/src/main.rs:here}}
```

<!--
<span class="caption">Listing 18-5: Incorrectly constructing a pattern whose
variables don’t match the number of elements in the tuple</span>
-->

<span class="caption">リスト18-5: 変数がタプルの要素数と一致しないパターンを間違って構成する</span>

<!--
Attempting to compile this code results in this type error:
-->

このコードのコンパイルを試みると、このような型エラーに落ち着きます:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-05/output.txt}}
```

<!--
To fix the error, we could ignore one or more of the values in the tuple using
`_` or `..`, as you’ll see in the [“Ignoring Values in a
Pattern”][ignoring-values-in-a-pattern] section. If the problem
is that we have too many variables in the pattern, the solution is to make the
types match by removing variables so the number of variables equals the number
of elements in the tuple.
-->

このエラーを修正するには、[「パターンの値を無視する」][ignoring-values-in-a-pattern]節で見ることになりますが、
`_`か`..`を使用してタプルの値のうち1つ以上を無視することができます。
パターンに変数が多すぎるというのが問題なら、変数の数がタプルの要素数と一致するように変数を減らすことで、
型を一致させることが解決策です。

<!--
### Function Parameters
-->

### 関数の引数

<!--
Function parameters can also be patterns. The code in Listing 18-6, which
declares a function named `foo` that takes one parameter named `x` of type
`i32`, should by now look familiar.
-->

関数の引数もパターンにできます。リスト18-6のコードは、型`i32`の`x`という引数1つを取る`foo`という関数を宣言していますが、
これまでに馴染み深くなっているはずです。

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-06/src/main.rs:here}}
```

<!--
<span class="caption">Listing 18-6: A function signature uses patterns in the
parameters</span>
-->

<span class="caption">リスト18-6: 関数シグニチャが引数にパターンを使用している</span>

<!--
The `x` part is a pattern! As we did with `let`, we could match a tuple in a
function’s arguments to the pattern. Listing 18-7 splits the values in a tuple
as we pass it to a function.
-->

`x`の部分がパターンです！`let`のように、関数の引数でパターンにタプルを合致させられるでしょう。
リスト18-7では、タプルを関数に渡したのでその中の値を分離しています。

<!--
<span class="filename">Filename: src/main.rs</span>
-->

<span class="filename">ファイル名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-07/src/main.rs}}
```

<!--
<span class="caption">Listing 18-7: A function with parameters that destructure
a tuple</span>
-->

<span class="caption">リスト18-7: タプルを分配する引数を伴う関数</span>

<!--
This code prints `Current location: (3, 5)`. The values `&(3, 5)` match the
pattern `&(x, y)`, so `x` is the value `3` and `y` is the value `5`.
-->

このコードは`Current location: (3, 5)`と出力します。値`&(3, 5)`はパターン`&(x, y)`と合致するので、
`x`は値`3`、`y`は値`5`になります。

<!--
We can also use patterns in closure parameter lists in the same way as in
function parameter lists, because closures are similar to functions, as
discussed in Chapter 13.
-->

また、クロージャの引数リストでも、関数の引数リストのようにパターンを使用することができます。
第13章で議論したように、クロージャは関数に似ているからです。

<!--
At this point, you’ve seen several ways of using patterns, but patterns don’t
work the same in every place we can use them. In some places, the patterns must
be irrefutable; in other circumstances, they can be refutable. We’ll discuss
these two concepts next.
-->

この時点で、パターンを使用する方法をいくつか見てきましたが、パターンを使用できる箇所全部で同じ動作をするわけではありません。
パターンが論駁不可能でなければならない箇所もあります。他の状況では、論駁可能にもなり得ます。この2つの概念を次に議論します。

<!--
[ignoring-values-in-a-pattern]:
ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern
-->

[ignoring-values-in-a-pattern]:
ch18-03-pattern-syntax.html#パターンの値を無視する
