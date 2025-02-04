<!--
## `RefCell<T>` and the Interior Mutability Pattern
-->

## `RefCell<T>`と内部可変性パターン

<!--
*Interior mutability* is a design pattern in Rust that allows you to mutate
data even when there are immutable references to that data; normally, this
action is disallowed by the borrowing rules. To mutate data, the pattern uses
`unsafe` code inside a data structure to bend Rust’s usual rules that govern
mutation and borrowing. Unsafe code indicates to the compiler that we’re
checking the rules manually instead of relying on the compiler to check them
for us; we will discuss unsafe code more in Chapter 19.
-->

内部可変性は、そのデータへの不変参照がある時でさえもデータを可変化できるRustでのデザインパターンです:
普通、この行動は借用規則により許可されません。データを可変化するために、このパターンは、データ構造内で`unsafe`コードを使用して、
可変性と借用を支配するRustの通常の規則を捻じ曲げています。
unsafeコードは、コンパイラが規則をチェックしてくれることに頼らず、自分でチェックしていることをコンパイラに表明します;
unsafeコードについては第19章で詳しく議論します。

<!--
We can use types that use the interior mutability pattern only when we can
ensure that the borrowing rules will be followed at runtime, even though the
compiler can’t guarantee that. The `unsafe` code involved is then wrapped in a
safe API, and the outer type is still immutable.
-->

たとえ、コンパイラが保証できなくても、借用規則に実行時に従うことが保証できる時に限り、
内部可変性パターンを使用した型を使用できます。関係する`unsafe`コードはそうしたら、安全なAPIにラップされ、
外側の型は、それでも不変です。

<!--
Let’s explore this concept by looking at the `RefCell<T>` type that follows the
interior mutability pattern.
-->

内部可変性パターンに従う`RefCell<T>`型を眺めてこの概念を探究しましょう。

<!--
### Enforcing Borrowing Rules at Runtime with `RefCell<T>`
-->

### `RefCell<T>`で実行時に借用規則を強制する

<!--
Unlike `Rc<T>`, the `RefCell<T>` type represents single ownership over the data
it holds. So, what makes `RefCell<T>` different from a type like `Box<T>`?
Recall the borrowing rules you learned in Chapter 4:
-->

`Rc<T>`と異なり、`RefCell<T>`型は、保持するデータに対して単独の所有権を表します。では、
どうして`RefCell<T>`が`Box<T>`のような型と異なるのでしょうか？第4章で学んだ借用規則を思い出してください:

<!--
* At any given time, you can have *either* (but not both) one mutable reference
  or any number of immutable references.
* References must always be valid.
-->

* いかなる時点においても、1個の可変参照または任意個の不変参照の*どちらか*(両方ではない)が存在できる。
* 参照は常に有効でなければならない。

<!--
With references and `Box<T>`, the borrowing rules’ invariants are enforced at
compile time. With `RefCell<T>`, these invariants are enforced *at runtime*.
With references, if you break these rules, you’ll get a compiler error. With
`RefCell<T>`, if you break these rules, your program will panic and exit.
-->

参照と`Box<T>`では、借用規則の不変条件は、コンパイル時に強制されています。`RefCell<T>`では、
これらの不変条件は、*実行時に*強制されます。参照でこれらの規則を破ったら、コンパイルエラーになりました。
`RefCell<T>`でこれらの規則を破ったら、プログラムはパニックし、終了します。

<!--
1行目、are that がどこまでかかるか不明だが、3行目最後、For those reasonsとあるので、最後までかかるように訳す
-->

<!--
The advantages of checking the borrowing rules at compile time are that errors
will be caught sooner in the development process, and there is no impact on
runtime performance because all the analysis is completed beforehand. For those
reasons, checking the borrowing rules at compile time is the best choice in the
majority of cases, which is why this is Rust’s default.
-->

コンパイル時に借用規則を精査することの利点は、エラーが開発過程の早い段階で捕捉されることと、
あらかじめ全ての分析が終わるので、実行パフォーマンスへの影響がないことです。それらの理由により、
多くの場合でコンパイル時に借用規則を精査することが最善の選択肢であり、これがRustの既定になっているのです。

<!--
The advantage of checking the borrowing rules at runtime instead is that
certain memory-safe scenarios are then allowed, where they would’ve been
disallowed by the compile-time checks. Static analysis, like the Rust compiler,
is inherently conservative. Some properties of code are impossible to detect by
analyzing the code: the most famous example is the Halting Problem, which is
beyond the scope of this book but is an interesting topic to research.
-->

借用規則を実行時に代わりに精査する利点は、コンパイル時の精査では許容されなかった特定のメモリ安全な筋書きが許容されることです。
Rustコンパイラのような静的解析は、本質的に保守的です。コードの特性には、コードを解析するだけでは検知できないものもあります:
最も有名な例は停止性問題であり、この本の範疇を超えていますが、調べると面白い話題です。

<!--
Because some analysis is impossible, if the Rust compiler can’t be sure the
code complies with the ownership rules, it might reject a correct program; in
this way, it’s conservative. If Rust accepted an incorrect program, users
wouldn’t be able to trust in the guarantees Rust makes. However, if Rust
rejects a correct program, the programmer will be inconvenienced, but nothing
catastrophic can occur. The `RefCell<T>` type is useful when you’re sure your
code follows the borrowing rules but the compiler is unable to understand and
guarantee that.
-->

不可能な分析もあるので、Rustのコンパイラが、コードが所有権規則に応じていると確証を得られない場合、
正しいプログラムを拒否する可能性があります; このように、保守的なのです。コンパイラが不正なプログラムを受け入れたら、
ユーザは、コンパイラが行う保証を信じることはできなくなるでしょう。しかしながら、
コンパイラが正当なプログラムを拒否するのなら、プログラマは不便に思うでしょうが、悲劇的なことは何も起こり得ません。
コードが借用規則に従っているとプログラマは確証を得ているが、コンパイラがそれを理解し保証することができない時に
`RefCell<T>`型は有用です。

<!--
Similar to `Rc<T>`, `RefCell<T>` is only for use in single-threaded scenarios
and will give you a compile-time error if you try using it in a multithreaded
context. We’ll talk about how to get the functionality of `RefCell<T>` in a
multithreaded program in Chapter 16.
-->

`Rc<T>`と類似して、`RefCell<T>`もシングルスレッドの筋書きで使用するためのものであり、
試しにマルチスレッドの文脈で使ってみようとすると、コンパイルエラーを出します。
`RefCell<T>`の機能をマルチスレッドのプログラムで得る方法については、第16章で語ります。

<!--
Here is a recap of the reasons to choose `Box<T>`, `Rc<T>`, or `RefCell<T>`:
-->

こちらに`Box<T>`, `Rc<T>`, `RefCell<T>`を選択する理由を要約しておきます:

<!--
* `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>`
  have single owners.
* `Box<T>` allows immutable or mutable borrows checked at compile time; `Rc<T>`
  allows only immutable borrows checked at compile time; `RefCell<T>` allows
  immutable or mutable borrows checked at runtime.
* Because `RefCell<T>` allows mutable borrows checked at runtime, you can
  mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is
  immutable.
-->

* `Rc<T>`は、同じデータに複数の所有者を持たせてくれる; `Box<T>`と`RefCell<T>`は単独の所有者。
* `Box<T>`では、不変借用も可変借用もコンパイル時に精査できる; `Rc<T>`では不変借用のみがコンパイル時に精査できる;
  `RefCell<T>`では、不変借用も可変借用も実行時に精査される。
* `RefCell<T>`は実行時に精査される可変借用を許可するので、`RefCell<T>`が不変でも、
  `RefCell<T>`内の値を可変化できる。

<!--
Mutating the value inside an immutable value is the *interior mutability*
pattern. Let’s look at a situation in which interior mutability is useful and
examine how it’s possible.
-->

不変な値の中の値を可変化することは、*内部可変性*パターンです。内部可変性が有用になる場面を見て、
それが可能になる方法を調査しましょう。

<!--
### Interior Mutability: A Mutable Borrow to an Immutable Value
-->

### 内部可変性: 不変値への可変借用

<!--
A consequence of the borrowing rules is that when you have an immutable value,
you can’t borrow it mutably. For example, this code won’t compile:
-->

借用規則の結果は、不変値がある時、可変で借用することはできないということです。
例えば、このコードはコンパイルできません:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

<!--
If you tried to compile this code, you’d get the following error:
-->

このコードをコンパイルしようとしたら、以下のようなエラーが出るでしょう:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

<!--
However, there are situations in which it would be useful for a value to mutate
itself in its methods but appear immutable to other code. Code outside the
value’s methods would not be able to mutate the value. Using `RefCell<T>` is
one way to get the ability to have interior mutability, but `RefCell<T>`
doesn’t get around the borrowing rules completely: the borrow checker in the
compiler allows this interior mutability, and the borrowing rules are checked
at runtime instead. If you violate the rules, you’ll get a `panic!` instead of
a compiler error.
-->

ですが、メソッド内で値が自身を可変化するけれども、他のコードにとっては、
不変に見えることが有用な場面もあります。その値のメソッドの外のコードは、その値を可変化することはできないでしょう。
`RefCell<T>`を使うことは、内部可変性を取得する能力を得る1つの方法です。しかし、
`RefCell<T>`は借用規則を完全に回避するものではありません: コンパイラの借用チェッカーは、内部可変性を許可し、
借用規則は代わりに実行時に精査されます。この規則を侵害したら、コンパイルエラーではなく`panic!`になるでしょう。

<!--
Let’s work through a practical example where we can use `RefCell<T>` to mutate
an immutable value and see why that is useful.
-->

`RefCell<T>`を使用して不変値を可変化する実践的な例に取り組み、それが役に立つ理由を確認しましょう。

<!--
#### A Use Case for Interior Mutability: Mock Objects
-->

#### 内部可変性のユースケース: モックオブジェクト

<!--
Sometimes during testing a programmer will use a type in place of another type,
in order to observe particular behavior and assert it’s implemented correctly.
This placeholder type is called a *test double*. Think of it in the sense of a
“stunt double” in filmmaking, where a person steps in and substitutes for an
actor to do a particular tricky scene. Test doubles stand in for other types
when we’re running tests. *Mock objects* are specific types of test doubles
that record what happens during a test so you can assert that the correct
actions took place.
-->

プログラマはテスト中に、特定の振る舞いを観測して、それが適切に実装されていることを確認するために、
ある型の代わりに他の型を使用することがあるでしょう。このプレースホルダ型は*テストダブル*と呼ばれます。
映画撮影において特定のトリッキーなシーンを演じるために、役者の代わりとして参加する人を指す、
「スタントダブル」と同じような意味で考えてください。テストダブルはテストの実行時に、他の型の代役を務めます。
*モックオブジェクト*は、テスト中に起きることを記録するテストダブルの特定の型なので、
正しい動作が起きたことをアサートできます。

<!--
Rust doesn’t have objects in the same sense as other languages have objects,
and Rust doesn’t have mock object functionality built into the standard library
as some other languages do. However, you can definitely create a struct that
will serve the same purposes as a mock object.
-->

Rustには、他の言語でいうオブジェクトは存在せず、また、他の言語のように標準ライブラリにモックオブジェクトの機能が組み込まれてもいません。
ですが、同じ目的をモックオブジェクトとして提供する構造体を作成することは確実にできます。

<!--
Here’s the scenario we’ll test: we’ll create a library that tracks a value
against a maximum value and sends messages based on how close to the maximum
value the current value is. This library could be used to keep track of a
user’s quota for the number of API calls they’re allowed to make, for example.
-->

以下が、テストを行う筋書きです: 値を最大値に対して追跡し、現在値がどれくらい最大値に近いかに基づいてメッセージを送信するライブラリを作成します。
このライブラリは、ユーザが行うことのできるAPIコールの数の割り当てを追跡するのに使用することができるでしょう。

<!--
Our library will only provide the functionality of tracking how close to the
maximum a value is and what the messages should be at what times. Applications
that use our library will be expected to provide the mechanism for sending the
messages: the application could put a message in the application, send an
email, send a text message, or something else. The library doesn’t need to know
that detail. All it needs is something that implements a trait we’ll provide
called `Messenger`. Listing 15-20 shows the library code:
-->

作成するライブラリは、値がどれくらい最大に近いかと、いつどんなメッセージになるべきかを追いかける機能を提供するだけです。
このライブラリを使用するアプリケーションは、メッセージを送信する機構を提供すると期待されるでしょう:
アプリケーションは、アプリケーションにメッセージを置いたり、メールを送ったり、テキストメッセージを送るなどできるでしょう。
ライブラリはその詳細を知る必要はありません。必要なのは、提供する`Messenger`と呼ばれるトレイトを実装している何かなのです。
リスト15-20は、ライブラリのコードを示しています:

<!--
<span class="filename">Filename: src/lib.rs</span>
-->

<span class="filename">ファイル名: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

<!--
<span class="caption">Listing 15-20: A library to keep track of how close a
value is to a maximum value and warn when the value is at certain levels</span>
-->

<span class="caption">リスト15-20: 値が最大値にどれくらい近いかを追跡し、特定のレベルの時に警告するライブラリ</span>

<!--
One important part of this code is that the `Messenger` trait has one method
called `send` that takes an immutable reference to `self` and the text of the
message. This trait is the interface our mock object needs to implement so that
the mock can be used in the same way a real object is. The other important part
is that we want to test the behavior of the `set_value` method on the
`LimitTracker`. We can change what we pass in for the `value` parameter, but
`set_value` doesn’t return anything for us to make assertions on. We want to be
able to say that if we create a `LimitTracker` with something that implements
the `Messenger` trait and a particular value for `max`, when we pass different
numbers for `value`, the messenger is told to send the appropriate messages.
-->

このコードの重要な部分の1つは、`Messenger`トレイトには、`self`への不変参照とメッセージのテキストを取る`send`というメソッドが1つあることです。
このトレイトが、モックが実際のオブジェクトと同じように使用できるために、モックオブジェクトが実装する必要のあるインターフェイスなのです。
もう1つの重要な部分は、`LimitTracker`の`set_value`メソッドの振る舞いをテストしたいということです。`value`引数に渡すものを変えることができますが、
`set_value`はアサートを行えるものは何も返してくれません。`LimitTracker`を`Messenger`トレイトを実装する何かと、
`max`の特定の値で生成したら、`value`に異なる数値を渡した時にメッセンジャーは適切なメッセージを送ると指示されると言えるようになりたいです。

<!--
We need a mock object that, instead of sending an email or text message when we
call `send`, will only keep track of the messages it’s told to send. We can
create a new instance of the mock object, create a `LimitTracker` that uses the
mock object, call the `set_value` method on `LimitTracker`, and then check that
the mock object has the messages we expect. Listing 15-21 shows an attempt to
implement a mock object to do just that, but the borrow checker won’t allow it:
-->

`send`を呼び出す時にメールやテキストメッセージを送る代わりに送ると指示されたメッセージを追跡するだけのモックオブジェクトが必要です。
モックオブジェクトの新規インスタンスを生成し、モックオブジェクトを使用する`LimitTracker`を生成し、
`LimitTracker`の`set_value`を呼び出し、それからモックオブジェクトに期待しているメッセージがあることを確認できます。
リスト15-21は、それだけをするモックオブジェクトを実装しようとするところを示しますが、借用チェッカーが許可してくれません:

<!--
<span class="filename">Filename: src/lib.rs</span>
-->

<span class="filename">ファイル名: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

<!--
<span class="caption">Listing 15-21: An attempt to implement a `MockMessenger`
that isn’t allowed by the borrow checker</span>
-->

<span class="caption">リスト15-21: 借用チェッカーが許可してくれない`MockMessenger`を実装しようとする</span>

<!--
This test code defines a `MockMessenger` struct that has a `sent_messages`
field with a `Vec` of `String` values to keep track of the messages it’s told
to send. We also define an associated function `new` to make it convenient to
create new `MockMessenger` values that start with an empty list of messages. We
then implement the `Messenger` trait for `MockMessenger` so we can give a
`MockMessenger` to a `LimitTracker`. In the definition of the `send` method, we
take the message passed in as a parameter and store it in the `MockMessenger`
list of `sent_messages`.
-->

このテストコードは`String`の`Vec`で送信すると指示されたメッセージを追跡する`sent_messages`フィールドのある`MockMessenger`構造体を定義しています。
また、空のメッセージリストから始まる新しい`MockMessenger`値を作るのを便利にしてくれる関連関数の`new`も定義しています。
それから`MockMessenger`に`Messenger`トレイトを実装しているので、`LimitTracker`に`MockMessenger`を与えられます。
`send`メソッドの定義で引数として渡されたメッセージを取り、`sent_messages`の`MockMessenger`リストに格納しています。

<!--
In the test, we’re testing what happens when the `LimitTracker` is told to set
`value` to something that is more than 75 percent of the `max` value. First, we
create a new `MockMessenger`, which will start with an empty list of messages.
Then we create a new `LimitTracker` and give it a reference to the new
`MockMessenger` and a `max` value of 100. We call the `set_value` method on the
`LimitTracker` with a value of 80, which is more than 75 percent of 100. Then
we assert that the list of messages that the `MockMessenger` is keeping track
of should now have one message in it.
-->

テストでは、`max`値の75%以上になる何かに`value`をセットしろと`LimitTracker`が指示される時に起きることをテストしています。
まず、新しい`MockMessenger`を生成し、空のメッセージリストから始まります。そして、
新しい`LimitTracker`を生成し、新しい`MockMessenger`の参照と100という`max`値を与えます。
`LimitTracker`の`set_value`メソッドは80という値で呼び出し、これは100の75%を上回っています。
そして、`MockMessenger`が追いかけているメッセージのリストが、今は1つのメッセージを含んでいるはずとアサートします。

<!--
However, there’s one problem with this test, as shown here:
-->

ところが、以下のようにこのテストには1つ問題があります:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

<!--
We can’t modify the `MockMessenger` to keep track of the messages, because the
`send` method takes an immutable reference to `self`. We also can’t take the
suggestion from the error text to use `&mut self` instead, because then the
signature of `send` wouldn’t match the signature in the `Messenger` trait
definition (feel free to try and see what error message you get).
-->

`send`メソッドは`self`への不変参照を取るので、`MockMessenger`を変更してメッセージを追跡できないのです。
代わりに`&mut self`を使用するというエラーテキストからの提言を選ぶこともできないのです。
そうしたら、`send`のシグニチャが、`Messenger`トレイト定義のシグニチャと一致しなくなるからです(気軽に試してエラーメッセージを確認してください)。

<!--
This is a situation in which interior mutability can help! We’ll store the
`sent_messages` within a `RefCell<T>`, and then the `send` method will be
able to modify `sent_messages` to store the messages we’ve seen. Listing 15-22
shows what that looks like:
-->

これは、内部可変性が役に立つ場面なのです！`sent_messages`を`RefCell<T>`内部に格納し、
そうしたら`send`メソッドは、`sent_messages`を変更して見かけたメッセージを格納できるようになるでしょう。
リスト15-22は、それがどんな感じかを示しています:

<!--
<span class="filename">Filename: src/lib.rs</span>
-->

<span class="filename">ファイル名: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

<!--
<span class="caption">Listing 15-22: Using `RefCell<T>` to mutate an inner
value while the outer value is considered immutable</span>
-->

<span class="caption">リスト15-22: 外側の値は不変と考えられる一方で`RefCell<T>`で内部の値を可変化する</span>

<!--
The `sent_messages` field is now of type `RefCell<Vec<String>>` instead of
`Vec<String>`. In the `new` function, we create a new `RefCell<Vec<String>>`
instance around the empty vector.
-->

さて、`sent_messages`フィールドは、`Vec<String>`ではなく、型`RefCell<Vec<String>>`になりました。
`new`関数で、空のベクタの周りに`RefCell<Vec<String>>`を新しく作成しています。

<!--
For the implementation of the `send` method, the first parameter is still an
immutable borrow of `self`, which matches the trait definition. We call
`borrow_mut` on the `RefCell<Vec<String>>` in `self.sent_messages` to get a
mutable reference to the value inside the `RefCell<Vec<String>>`, which is the
vector. Then we can call `push` on the mutable reference to the vector to keep
track of the messages sent during the test.
-->

`send`メソッドの実装については、最初の引数はそれでも`self`への不変借用で、トレイト定義と合致しています。
`RefCell<Vec<String>>`の`borrow_mut`を`self.sent_messages`に呼び出し、
`RefCell<Vec<String>>`の中の値への可変参照を得て、これはベクタになります。
それからベクタへの可変参照に`push`を呼び出して、テスト中に送られるメッセージを追跡しています。

<!--
The last change we have to make is in the assertion: to see how many items are
in the inner vector, we call `borrow` on the `RefCell<Vec<String>>` to get an
immutable reference to the vector.
-->

行わなければならない最後の変更は、アサート内部にあります: 内部のベクタにある要素の数を確認するため、
`RefCell<Vec<String>>`に`borrow`を呼び出し、ベクタへの不変参照を得ています。

<!--
Now that you’ve seen how to use `RefCell<T>`, let’s dig into how it works!
-->

`RefCell<T>`の使用法を見かけたので、動作の仕方を深掘りしましょう！

<!--
#### Keeping Track of Borrows at Runtime with `RefCell<T>`
-->

#### `RefCell<T>`で実行時に借用を追いかける

<!--
When creating immutable and mutable references, we use the `&` and `&mut`
syntax, respectively. With `RefCell<T>`, we use the `borrow` and `borrow_mut`
methods, which are part of the safe API that belongs to `RefCell<T>`. The
`borrow` method returns the smart pointer type `Ref<T>`, and `borrow_mut`
returns the smart pointer type `RefMut<T>`. Both types implement `Deref`, so we
can treat them like regular references.
-->

不変および可変参照を作成する時、それぞれ`&`と`&mut`記法を使用します。`RefCell<T>`では、
`borrow`と`borrow_mut`メソッドを使用し、これらは`RefCell<T>`に所属する安全なAPIの一部です。
`borrow`メソッドは、スマートポインタ型の`Ref<T>`を返し、`borrow_mut`はスマートポインタ型の`RefMut<T>`を返します。
どちらの型も`Deref`を実装しているので、普通の参照のように扱うことができます。

<!--
The `RefCell<T>` keeps track of how many `Ref<T>` and `RefMut<T>` smart
pointers are currently active. Every time we call `borrow`, the `RefCell<T>`
increases its count of how many immutable borrows are active. When a `Ref<T>`
value goes out of scope, the count of immutable borrows goes down by one. Just
like the compile-time borrowing rules, `RefCell<T>` lets us have many immutable
borrows or one mutable borrow at any point in time.
-->

`RefCell<T>`は、現在活動中の`Ref<T>`と`RefMut<T>`スマートポインタの数を追いかけます。
`borrow`を呼び出す度に、`RefCell<T>`は活動中の不変参照の数を増やします。`Ref<T>`の値がスコープを抜けたら、
不変参照の数は1下がります。コンパイル時の借用規則と全く同じように、`RefCell<T>`はいかなる時も、
複数の不変借用または1つの可変借用を持たせてくれるのです。

<!--
If we try to violate these rules, rather than getting a compiler error as we
would with references, the implementation of `RefCell<T>` will panic at
runtime. Listing 15-23 shows a modification of the implementation of `send` in
Listing 15-22. We’re deliberately trying to create two mutable borrows active
for the same scope to illustrate that `RefCell<T>` prevents us from doing this
at runtime.
-->

これらの規則を侵害しようとすれば、参照のようにコンパイルエラーになるのではなく、
`RefCell<T>`の実装は実行時にパニックするでしょう。リスト15-23は、リスト15-22の`send`実装に対する変更を示しています。
同じスコープで2つの可変借用が活動するようわざと生成し、`RefCell<T>`が実行時にこれをすることを阻止してくれるところを説明しています。

<!--
<span class="filename">Filename: src/lib.rs</span>
-->

<span class="filename">ファイル名: src/lib.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

<!--
<span class="caption">Listing 15-23: Creating two mutable references in the
same scope to see that `RefCell<T>` will panic</span>
-->

<span class="caption">リスト15-23: 同じスコープで2つの可変参照を生成して`RefCell<T>`がパニックすることを確かめる</span>

<!--
We create a variable `one_borrow` for the `RefMut<T>` smart pointer returned
from `borrow_mut`. Then we create another mutable borrow in the same way in the
variable `two_borrow`. This makes two mutable references in the same scope,
which isn’t allowed. When we run the tests for our library, the code in Listing
15-23 will compile without any errors, but the test will fail:
-->

`borrow_mut`から返ってきた`RefMut<T>`スマートポインタに対して変数`one_borrow`を生成しています。
そして、同様にして変数`two_borrow`にも別の可変借用を生成しています。これにより同じスコープで2つの可変参照ができ、
これは許可されないことです。このテストを自分のライブラリ用に走らせると、リスト15-23のコードはエラーなくコンパイルできますが、
テストは失敗するでしょう:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

<!--
Notice that the code panicked with the message `already borrowed:
BorrowMutError`. This is how `RefCell<T>` handles violations of the borrowing
rules at runtime.
-->

コードは、`already borrowed: BorrowMutError`というメッセージとともにパニックしたことに注目してください。
このようにして`RefCell<T>`は実行時に借用規則の侵害を扱うのです。

<!--
Choosing to catch borrowing errors at runtime rather than compile time, as
we’ve done here, means you’d potentially be finding mistakes in your code later
in the development process: possibly not until your code was deployed to
production. Also, your code would incur a small runtime performance penalty as
a result of keeping track of the borrows at runtime rather than compile time.
However, using `RefCell<T>` makes it possible to write a mock object that can
modify itself to keep track of the messages it has seen while you’re using it
in a context where only immutable values are allowed. You can use `RefCell<T>`
despite its trade-offs to get more functionality than regular references
provide.
-->

ここで行ったように、コンパイル時ではなく実行時に借用エラーをキャッチすることを選択するということは、
開発過程のより遅い段階でコードのミスを発見することになる可能性があることを意味します:
コードをプロダクションにデプロイする時までに発見できないかもしれません。また、
コンパイル時ではなく、実行時に借用を追いかける結果として、少し実行時にパフォーマンスを犠牲にするでしょう。
しかしながら、`RefCell<T>`を使うことで、不変値のみが許可される文脈で使用しつつ、
自身を変更して見かけたメッセージを追跡するモックオブジェクトを書くことが可能になります。
代償はありますが、`RefCell<T>`を使用すれば、普通の参照よりも多くの機能を得ることができるわけです。

<!--
### Having Multiple Owners of Mutable Data by Combining `Rc<T>` and `RefCell<T>`
-->

### `Rc<T>`と`RefCell<T>`を組み合わせることで可変なデータに複数の所有者を持たせる

<!--
A common way to use `RefCell<T>` is in combination with `Rc<T>`. Recall that
`Rc<T>` lets you have multiple owners of some data, but it only gives immutable
access to that data. If you have an `Rc<T>` that holds a `RefCell<T>`, you can
get a value that can have multiple owners *and* that you can mutate!
-->

`RefCell<T>`の一般的な使用法は、`Rc<T>`と組み合わせることにあります。`Rc<T>`は何らかのデータに複数の所有者を持たせてくれるけれども、
そのデータに不変のアクセスしかさせてくれないことを思い出してください。`RefCell<T>`を抱える`Rc<T>`があれば、
複数の所有者を持ち*そして*、可変化できる値を得ることができるのです。

<!--
For example, recall the cons list example in Listing 15-18 where we used
`Rc<T>` to allow multiple lists to share ownership of another list. Because
`Rc<T>` holds only immutable values, we can’t change any of the values in the
list once we’ve created them. Let’s add in `RefCell<T>` to gain the ability to
change the values in the lists. Listing 15-24 shows that by using a
`RefCell<T>` in the `Cons` definition, we can modify the value stored in all
the lists:
-->

例を挙げれば、`Rc<T>`を使用して複数のリストに別のリストの所有権を共有させたリスト15-18のコンスリストの例を思い出してください。
`Rc<T>`は不変値だけを抱えるので、一旦生成したら、リストの値はどれも変更できません。`RefCell<T>`を含めて、
リストの値を変更する能力を得ましょう。`RefCell<T>`を`Cons`定義で使用することで、
リスト全てに格納されている値を変更できることをリスト15-24は示しています:

<!--
<span class="filename">Filename: src/main.rs</span>
-->

<span class="filename">ファイル名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

<!--
<span class="caption">Listing 15-24: Using `Rc<RefCell<i32>>` to create a
`List` that we can mutate</span>
-->

<span class="caption">リスト15-24: `Rc<RefCell<i32>>`で可変化できる`List`を生成する</span>

<!--
We create a value that is an instance of `Rc<RefCell<i32>>` and store it in a
variable named `value` so we can access it directly later. Then we create a
`List` in `a` with a `Cons` variant that holds `value`. We need to clone
`value` so both `a` and `value` have ownership of the inner `5` value rather
than transferring ownership from `value` to `a` or having `a` borrow from
`value`.
-->

`Rc<RefCell<i32>>`のインスタンスの値を生成し、`value`という名前の変数に格納しているので、
直接後ほどアクセスすることができます。そして、`a`に`value`を持つ`Cons`列挙子で`List`を生成しています。
`value`から`a`に所有権を移したり、`a`が`value`から借用するのではなく、`a`と`value`どちらにも中の`5`の値の所有権を持たせるよう、
`value`をクローンする必要があります。

<!--
We wrap the list `a` in an `Rc<T>` so when we create lists `b` and `c`, they
can both refer to `a`, which is what we did in Listing 15-18.
-->

リスト`a`を`Rc<T>`に包んでいるので、リスト`b`と`c`を生成する時に、どちらも`a`を参照できます。
リスト15-18ではそうしていました。

<!--
After we’ve created the lists in `a`, `b`, and `c`, we want to add 10 to the
value in `value`. We do this by calling `borrow_mut` on `value`, which uses the
automatic dereferencing feature we discussed in Chapter 5 (see the section
[“Where’s the `->` Operator?”][wheres-the---operator]) to
dereference the `Rc<T>` to the inner `RefCell<T>` value. The `borrow_mut`
method returns a `RefMut<T>` smart pointer, and we use the dereference operator
on it and change the inner value.
-->

`a`、`b`、`c`のリストを作成した後、`value`の値に10を足したいとします。これを`value`の`borrow_mut`を呼び出すことで行い、
これは、第5章で議論した自動参照外し機能([「`->`演算子はどこに行ったの？」][wheres-the---operator]節をご覧ください)を使用して、
`Rc<T>`を内部の`RefCell<T>`値に参照外ししています。`borrow_mut`メソッドは、
`RefMut<T>`スマートポインタを返し、それに対して参照外し演算子を使用し、中の値を変更します。

<!--
When we print `a`, `b`, and `c`, we can see that they all have the modified
value of 15 rather than 5:
-->

`a`、`b`、`c`を出力すると、全て5ではなく、変更された15という値になっていることがわかります。

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

<!--
This technique is pretty neat! By using `RefCell<T>`, we have an outwardly
immutable `List` value. But we can use the methods on `RefCell<T>` that provide
access to its interior mutability so we can modify our data when we need to.
The runtime checks of the borrowing rules protect us from data races, and it’s
sometimes worth trading a bit of speed for this flexibility in our data
structures. Note that `RefCell<T>` does not work for multithreaded code!
`Mutex<T>` is the thread-safe version of `RefCell<T>` and we’ll discuss
`Mutex<T>` in Chapter 16.
-->

このテクニックは非常に綺麗です！`RefCell<T>`を使用することで表面上は不変な`List`値を持てます。
しかし、内部可変性へのアクセスを提供する`RefCell<T>`のメソッドを使用できるので、必要な時にはデータを変更できます。
借用規則を実行時に精査することでデータ競合を防ぎ、時としてデータ構造でちょっとのスピードを犠牲にこの柔軟性を得るのは価値があります。
`RefCell<T>`は、マルチスレッドで実行されるコードに対しては機能しないことに注意してください！
`RefCell<T>`のスレッドセーフなバージョンは`Mutex<T>`で、`Mutex<T>`については第16章で議論しましょう。

<!--
[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
-->

[wheres-the---operator]: ch05-03-method-syntax.html#-演算子はどこに行ったの
