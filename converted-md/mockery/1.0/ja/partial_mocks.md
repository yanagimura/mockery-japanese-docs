::: {.index}
single: パーシャルモック
:::

パーシャルモックの生成
======================

パーシャル（部分）モックは、あるオブジェクトのいくつかのメソッドのみをモックし、残りは通常の呼び出し通り（つまり、実装されている通り）にレスポンスさせたい必要がある場合に便利です。Mockeryはパーシャル生成の明確な３つの戦略を実装しています。それぞれは特定の利点と欠点があるため、モックの必要に応じて、好みとソースコードに基づいて使用する戦略を選びます。

前に[パーシャルテストダブル](creating_test_doubles.htm#パーシャルテストダブル)
で少し取り扱っていますが、ここでもう少しこの主題を取り上げましょう。

1.  ランタイムパーシャルテストダブル
2.  生成パーシャルテストダブル
3.  プロキシパーシャルモック

ランタイムパーシャルテストダブル
--------------------------------

ランタイムパーシャルテストダブルは、パッシブパーシャルモックとしても知られていますが、これはモック対象のオブジェクトを一種のデフォルト状態として扱います。

``` {.php}
$mock = \Mockery::mock('MyClass')->makePartial();
```

パーシャルを生成する場合、エクスペクションと一致するメッソッド呼び出しでない限り、親のクラス(`MyClass`)のオリジナルメソッドへ実行が渡されます。指定したメソッドコールと一致するエクスペクションがない場合、その呼び出しはモックしているクラスへ渡されます。呼び出しがモックされるか、されないかの境は、定義したエクスペクションに完全に依存するため、前もってモックするメソッドを定義する必要がありません。

ランタイムパーシャルテストダブルの使用例は、クックブックの[大きな親クラス](big_parent_class.html)にあるエントリを参照してください。

> {note}
> `makePartial()`メソッドは、最初にパーシャルモックのタイプとして導入された、オリジナルの`shouldDeferMissing()`メソッドと同じものです。`shouldDeferMissing()`メソッドについては、[振る舞いモディファイヤー](creating_test_doubles.html#振る舞いモディファイヤー)の章をご覧ください。（訳注：リンク先に情報は記述されていません。この注記は古い可能性があります。）

生成パーシャルテストダブル
--------------------------

生成パーシャルテストダブルは、伝統的なパーシャルモックとしても知られており、前もってモックするクラスのメソッドを定義しておき、残りはモックしません。伝統的なモックを作成する記法は以下のとおりです。

``` {.php}
$mock = \Mockery::mock('MyClass[foo,bar]');
```

上記の例で、MyClassの`foo()`と`bar()`メソッドはモックされますが、MyClassのその他のクラスではしません。`foo()`と`bar()`メソッドのモックでの振る舞いを指示するため、エクスペクションを定義する必要があります。

モックしないメソッドが関連する可能性があるため、コンストラクターの引数を渡せることを忘れないでください。

``` {.php}
$mock = \Mockery::mock('MyNamespace\MyClass[foo]', array($arg1, $arg2));
```

詳細は、[コンストラクターの引数](creating_test_doubles.html#コンストラクターの引数)セクションで確認してください。

> {note}
> 生成パーシャルテストダブルをサポートしていますが、使用を推奨していません。

プロキシパーシャルモック
------------------------

パーシャルの最後の種類は、プロキシパーシャルモックです。finalがマークされているため、あるクラスがモックできない事態に出会うことがあります。同様にあるクラスのメソッドで、finalがマークされているのを見つけることがあります。このようなシナリオでは、クラスをシンプルに拡張し、メソッドをモックするためにオーバーライドできません。解決するにはクリエイティブになる必要があります。

``` {.php}
$mock = \Mockery::mock(new MyClass);
```

そうです。新しいモックはプロキシです。呼び出しを横取りし、エクスペクションに合わないメソッドは（生成し、引数で渡した）オブジェクトへ引き渡します。プロキシは制限を受けていないため、これにより間接的にfinalがついているメソッドをモックできるのです。トレードオフは明らかです。プロキシパーシャルは、モックしているクラスのタイプヒントのチェックに失敗します。クラスは拡張できないからです。

特別な内部クラス
----------------

プロキシパーシャルを除く、全てのモックオブジェクトで、`passthru()`呼び出しエクスペクションを使用し、背後の本当のクラスメソッドを呼び出せます。戻り値は実際のクラスから返され、モックされている戻り値のキューはバイパスされます。（ただ無視するわけです。）

内部使用のための、第４のパーシャルモックが存在しています。finalがマークされているメソッドを含むクラスをモックしようとすると、自動的に生成されます。なぜなら、そうしたメソッドはオーバーライドできないため、モックされずにただ残されてしまうからです。通常、これを意識する必要はありませんが、本当に、本当に、finalメソッドをモックする必要がある場合、唯一の可能性はプロキシパーシャルモックを通じて行います。たとえば、SplFileInfoがこの自動内部パーシャルが使用される一般的なクラスです。なぜなら、内部にpublicなfinalメソッドを含んでいるからです。