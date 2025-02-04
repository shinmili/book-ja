<!--
## Customizing Builds with Release Profiles
-->

## リリースプロファイルでビルドをカスタマイズする

<!--
In Rust, *release profiles* are predefined and customizable profiles with
different configurations that allow a programmer to have more control over
various options for compiling code. Each profile is configured independently of
the others.
-->

Rustにおいて、*リリースプロファイル*とは、プログラマがコードのコンパイルオプションについてより制御可能にしてくれる、
定義済みのカスタマイズ可能なプロファイルです。各プロファイルは、それぞれ独立して設定されます。

<!--
Cargo has two main profiles: the `dev` profile Cargo uses when you run `cargo
build` and the `release` profile Cargo uses when you run `cargo build
--release`. The `dev` profile is defined with good defaults for development,
and the `release` profile has good defaults for release builds.
-->

Cargoには2つの主なプロファイルが存在します: `dev`プロファイルは、`cargo build`コマンドを実行したときに使用され、
`release`プロファイルは、`cargo build --release`コマンドを実行したときに使用されます。
`dev`プロファイルは、開発中に役に立つデフォルト設定がなされており、`release`プロファイルは、
リリース用の設定がなされています。

<!--
These profile names might be familiar from the output of your builds:
-->

これらのプロファイル名は、ビルドの出力で馴染みのある可能性があります:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

<!--
The `dev` and `release` are these different profiles used by the compiler.
-->

`dev`と`release`が、コンパイラによって使用される異なるプロファイルです。

<!--
Cargo has default settings for each of the profiles that apply when you haven't
explicitly added any `[profile.*]` sections in the project’s *Cargo.toml* file.
By adding `[profile.*]` sections for any profile you want to customize, you
override any subset of the default settings. For example, here are the default
values for the `opt-level` setting for the `dev` and `release` profiles:
-->

Cargoには、プロジェクトの*Cargo.toml*ファイルに`[profile.*]`セクションが明示的に追加されていない際に適用される、
各プロファイル用のデフォルト設定が存在します。カスタマイズしたいプロファイル用の`[profile.*]`セクションを追加することで、
デフォルト設定の一部が上書きされます。例えば、こちらが`dev`と`release`プロファイルの`opt-level`設定のデフォルト値です:

<!--
<span class="filename">Filename: Cargo.toml</span>
-->

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

<!--
The `opt-level` setting controls the number of optimizations Rust will apply to
your code, with a range of 0 to 3. Applying more optimizations extends
compiling time, so if you’re in development and compiling your code often,
you’ll want fewer optimizations to compile faster even if the resulting code
runs slower. The default `opt-level` for `dev` is therefore `0`. When you’re
ready to release your code, it’s best to spend more time compiling. You’ll only
compile in release mode once, but you’ll run the compiled program many times,
so release mode trades longer compile time for code that runs faster. That is
why the default `opt-level` for the `release` profile is `3`.
-->

`opt-level`設定は、0から3の範囲でコンパイラがコードに適用する最適化の度合いを制御します。
最適化を多くかけるとコンパイル時間が延びるので、開発中に頻繁にコードをコンパイルするのなら、
たとえ出力結果のコードの動作速度が遅くなっても早くコンパイルできるように、より少ない最適化で済ませてほしいですよね。
そのため、`dev`の`opt-level`のデフォルト設定は`0`になっています。
コードのリリース準備ができたら、より長い時間をコンパイルにかけるのが最善の策です。
リリースモードでコンパイルするのはたった1回ですが、コンパイル結果のプログラムは何度も実行するので、
リリースモードでは、長いコンパイル時間と引き換えに、生成したコードが速く動作します。
そのため、`release`の`opt-level`のデフォルト設定が`3`になっているのです。

<!--
You can override a default setting by adding a different value for it in
*Cargo.toml*. For example, if we want to use optimization level 1 in the
development profile, we can add these two lines to our project’s *Cargo.toml*
file:
-->

デフォルト設定に対して`Cargo.toml`で異なる値を追加すれば、上書きすることができます。
例として、開発用プロファイルで最適化レベル1を使用したければ、以下の2行をプロジェクトの*Cargo.toml*ファイルに追加できます:

<!--
<span class="filename">Filename: Cargo.toml</span>
-->

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

<!--
This code overrides the default setting of `0`. Now when we run `cargo build`,
Cargo will use the defaults for the `dev` profile plus our customization to
`opt-level`. Because we set `opt-level` to `1`, Cargo will apply more
optimizations than the default, but not as many as in a release build.
-->

このコードは、デフォルト設定の`0`を上書きします。こうすると、`cargo build`を実行したときに、
`dev`プロファイル用のデフォルト設定に加えて、Cargoは`opt-level`の変更を適用します。
`opt-level`を`1`に設定したので、Cargoはデフォルトよりは最適化を行いますが、リリースビルドほどではありません。

<!--
For the full list of configuration options and defaults for each profile, see
[Cargo’s documentation](https://doc.rust-lang.org/cargo/reference/profiles.html).
-->

設定の選択肢と各プロファイルのデフォルト設定の一覧は、[Cargoのドキュメンテーション](https://doc.rust-lang.org/cargo/reference/profiles.html)を参照されたし。
