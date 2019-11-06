# Harmonyフレームワークの使い方

1. [Harmonyとは](#what_is_harmony)
1. [Harmonyの使い方](#using_harmony)
1. [パッチクラスに対するアノテーションの記法](#class_annotation)
1. [パッチメソッドの記法](#method_defining)
1. [実行フロー](#execution_flow)
1. [アノテーションのプライオリティ設定](#priority_annotation)
1. [Harmonyの共有ライブラリ](#utilities)
1. [参考](#reference)

<a name="what_is_harmony"></a>
## Harmonyとは

　pardeike氏の開発した、Aspect指向のMOD開発用フレームワーク。

　Harmonyでは、C#(正しくは.NET環境)の **リフレクション機能** を活用して、実行時に動的に実行文をインジェクションするライブラリ・機構を提供している。

　基本的な実現方法は継承とオーバーライトメソッドのイメージが近いだろう。改変対象となるクラスに薄いラッピングクラスを被せ、そちらが元のクラスの代わりに生成・実行されるようになっている。記述されたパッチコードは元のメソッドの前後に実行したり、元のメソッドの代わりに実行したりしている。

　ONIではゲーム画面表示前、**MODのDLL読み込み時** にHarmonyの機構が実行され、各コードの動的追加/すり替えを行なっている。そのため、各MODはMODの反映タイミングを気にする事なしに"パッチを書く"ことにのみ集中できる。

　なお、以下の節ではONIに関係のないBootstrappingやアノテーションの一部については解説の対象外とした。技術的に興味がある、もしくは別のゲームのために調べたい場合は [参考](reference) にある **開発元ドキュメント** を参照する事。

### Harmonyでできない事

- ゲーム側のクラス・構造体定義のなかに新たにメンバ(クラス/インスタンス)変数や定数等を追加する事。
- ゲーム側のクラス・構造体定義に新たにメソッドを追加する事。
- MODがロードされるより前に実行されるソース部分に改修を加えて反映させる事。

　※ パッチを提供する側のクラスでの独自のメソッド定義や変数・定数の定義とは別の事である点に注意。混同して混乱しないように。

<a name="using_harmony"></a>
## Harmonyの使い方

　Harmonyでは、特定の命名規則に従ったメソッド、又は特定のアノテーションを付けたメソッドがインジェクションの対象になる。以下ではそのルールを解説する。

### 1メソッド1クラスの原則

　Harmonyでは、改造対象となる1つのメソッドに対して必ず **クラスを1つ** 定義する。

　定義するクラスは、次のようにアノテーションを加えることで、Harmonyが捕捉できるようにする。

```C#
[HarmonyPatch(typeof(SomeTypeHere/*ターゲットとなるメソッドのあるクラス名*/))]
[HarmonyPatch(nameof(SomeTypeHere.SomeMethodName/*ターゲットとなるメソッド名*/))]
class MyPatchClass
{
  // パッチメソッド(後述)
	static void Postfix(...)
	{
		//...
	}
}
```

<a name="class_annotation"></a>
## パッチクラスに対するアノテーションの記法

　パッチを提供するクラスに対するアノテーションは、次のように記述する。基本的に **クラスのType** 、 **メソッド名** 、 **パラメータタイプ(オプション)** の順に定義していく。

- `[HarmonyPatch(Type)]` - パッチを充てるメソッドのあるクラスの **Type** を指定する。
  - `[HarmonyPatch(Type[])]` - **Type[]** とする事でジェネリクスに対応(オプション)。
- `[HarmonyPatch(String)]` - パッチを充てるメソッド名の **文字列** を指定する。
- `[HarmonyPatch(String, PropertyMethod)]` - パッチを充てるプロパティ名とプロパティメソッド(Setter,Getter)を指定する。
- `[HarmonyPatch(Type[])]` - パッチを充てるメソッドのパラメータタイプを指定する(オーバーロード定義などで同じ名前の複数の名前がある場合に向けたもの)

　次のように省略可して記述する事も可能。

- `[HarmonyPatch(Type, String, Type[])]` - 先頭からクラスType、メソッド名、パラメータタイプの指定

### 記述の例

　**String.ToUpper()** にパッチする場合：

```C#
[HarmonyPatch(typeof(String))]
[HarmonyPatch("ToUpper")]
```

　**MyClass** の **Account** プロパティのset(Setter)にパッチする場合：

```C#
[HarmonyPatch(typeof(MyClass))]
[HarmonyPatch("Account", PropertyMethod.Setter)]
```

　**String.IndexOf(char, int)** にパッチする場合：

```C#
[HarmonyPatch(typeof(String))]
[HarmonyPatch("IndexOf")]
[HarmonyPatch(new Type[] { typeof(char), typeof(int) })]

// または

[HarmonyPatch(typeof(String), "IndexOf", new Type[] { typeof(char), typeof(int) })]
```

### コンストラクタへのパッチ

　コンストラクタへのパッチは内部名である **".ctor"への直接のパッチはしてはいけない** 。代わりに次のように **メソッド名を省略し、パラメータに new クラスType[] を記述** する。

```C#
[HarmonyPatch(typeof(TestClass))]
[HarmonyPatch(new Type[] { })]
```

### ジェネリックメソッドへのパッチ

　ジェネリックメソッドに関しては、 **汎用的・全般的な型指定(T item)のパッチングはできない。** かわりに、 **特定タイプを指定して(string item)のパッチングは可能** なので、その方法で代用する。

　必要十分なタイプを指定したオーバーロードするパッチの提供は、代用に十分足りる実装になる。

### ノート

　パッチクラスは定義におけるpublic/privateもしくはstaticの有無は問わない。

<a name="method_defining"></a>
## パッチメソッドの記法

　パッチメソッドは、次のように命名規則に則った名前にするか、アノテーションを付加することでパッチメソッドとHarmonyが認識できるようになる。

　パッチメソッドには **定義が必須のメソッドは無い** ので、適時必要なメソッドのみ実装する。

### Prepareメソッド(オプション)

　パッチがじっさいに **充てられる前に** 実行される。

　このメソッドを定義したら、必ず戻り値(bool)を戻さなければならない。Harmonyは戻り値を見てじっさいにパッチを充てるべきか否か判断する。

```C#
// 次のメソッド名にする。
static bool Prepare()
static bool Prepare(MethodBase original)
// または次のようにアノテーションする。
[HarmonyPrepare]
static bool MyInitializer(...)
```

### Prefixメソッド(オプション)

　パッチの対象となる **オリジナルメソッドの実行前に** 実行される。

　引数はオリジナルメソッドと同じものを指定する事。

　Prefixメソッドは、boolの戻り値を返すことができる。ここでfalseを返すと、オリジナルのコード(及び後続するPrefixメソッド)の実行をキャンセルできる。詳しくは [実行フロー](execution_flow) を参照する事。

　※ 戻り値をvoidにした場合、C#の仕様によりtrue相当が返されたのと同等の動作を(正しくはfalse以外が返されたと判断して)する。

```C#
// 次のメソッド名にする。
static bool Prefix(...)
// または次のようにアノテーションする。
[HarmonyPrefix]
static bool MyPrefix(...)
```

### Postfixメソッド(オプション)

　パッチの対象となる **オリジナルメソッドの実行後に** 実行される。

　引数はオリジナルメソッドと同じものを指定する事。

　Postfixメソッドはいかなる場合でも実行される。

```C#
// 次のメソッド名にする。
static void Postfix(...)
// または次のようにアノテーションする。
[HarmonyPostfix]
static void MyPostfix(...)
```

### Transpilerメソッド(オプション)

　パッチの対象となる **オリジナルメソッドとすり替わって** 実行される。

　引数の2つ目以降はオリジナルメソッドと同じものを指定する事。

```C#
// 次のメソッド名にする。
static IEnumerable<CodeInstruction> Transpiler(IEnumerable<CodeInstruction> instr, ...)
// または次のようにアノテーションする。
[HarmonyTranspiler]
static IEnumerable<CodeInstruction> MyTranspiler(IEnumerable<CodeInstruction> instr, ...)
```

### Cleanupメソッド(オプション)

　パッチが **充てられた後** に実行される。内部で生成したオブジェクトの廃棄などに利用できる。

```C#
// 次のメソッド名にする。
static bool Cleanup()
static bool Cleanup(MethodBase original)
// または次のようにアノテーションする。
[HarmonyCleanup]
static bool MyInitializer(...)
```

### TargetMethodメソッド(オプション)

　だいたいの場合、HarmonyPatch()もしくは(前述の)クラスアノテーションの組み合わせでパッチを適用するメソッドは定義できる。

　ただ、コード内で **動的にメソッドを切り替える** 必要がある可能性もあるので、そのためにこのTargetMethodが用意されている。

```C#
// 次のメソッド名にする。
static  MethodBase  TargetMethod（）
static MethodBase TargetMethod（HarmonyInstance  instance）
// または次のようにアノテーションする。
// 注：HarmonyInstanceはアトリビュートと一緒に渡さないと、1.2.0.1では動作しない。
[HarmonyTargetMethod]
static MethodBase CalculateMethod（ HarmonyInstance  instance）
```

　このメソッドから返されたMethodInfoが実行される。別の(あなたの定義した)Harmonyメソッド(harmonyInstance)を返すこともできる。

### ノート

　パッチメソッドは定義におけるpublic/privateは問わないが、必ずstaticにする必要がある。ただ、手でパッチメソッドを指定するよう記述することでDynamicMethodとして提供する方法もある。

<a name="execution_flow"></a>
## 実行フロー

　ここから先の話は少々込み入った話なので、上記の記法までで十分な方はとくに気にする必要はない。

　MODが同一メソッドをパッチングする状況になった場合、Harmonyは(パッチを上書きする事なく)全てのパッチを順に実行するようになっている。先に記述したようにPrefixメソッドはbool値を返す事ができるが、明示的にfalseを返す事によりオリジナルのコード(及び後続のパッチング)をスキップさせる事ができる。

　比べて、全てのPostfixメソッドは常に順に実行される。

　実行時イメージは、疑似コードで表現した場合ちょうど下記のようなイメージになる。

```C#
bool run = true;
var result = null;

// Prefixの実行
if (run) run = Prefix1(...);
if (run) run = Prefix2(...);
// ...

// オリジナルコードの実行
if (run) result = Original(...);

// Postfixの実行
Postfix1(...);
Postfix2(...);
// ...

return result;
```

<a href="priority_annotation"></a>
## アノテーションのプライオリティ設定

　このプライオリティの設定は、MODの読込み順序とは別の話なので、混同しない事。

　Harmonyにおいては、通常パッチングの順序はMODの読込み順通りになるが、(MOD同士の設定の衝突などで)実行順序を制御しなければいけない局面にもアノテーションで対応している。

- [HarmonyPriority（int）] - このPrefix/Postfixメソッドの優先度を設定する。デフォルトは400(Priority.Normal）になる。
- [HarmonyBefore（string []）] - このPrefix/Postfixメソッドは、IDが与えられる前に実行される。
- [HarmonyAfter（string []）] - このPrefix/Postfixメソッドは、IDが与えられた後に実行される。

　※ ONI上においては、 **後者2つのIDを指定したアノテーションは役に立たない** (全て同一IDのパッチとして処理されているため)ので、オリジナルの文章における例は省略する。

　ノート："OxygenNotIncluded_v0.1"現在指定されているID

<a name="utilities"></a>
## Harmonyの共有ライブラリ

　Harmonyでは共有ライブラリとして、次のユーティリティを準備してある。

### AccessTools

　リフレクションをより簡易に扱うためのヘルパークラス。

```C#
public static BindingFlags all = ... {...}
public static Type TypeByName(string name) {...}
public static FieldInfo Field(Type type, string name) {...}
public static PropertyInfo Property(Type type, string name) {...}
public static MethodInfo Method(Type type, string name, Type[] parameters = null, Type[] generics = null) {...}
public static ConstructorInfo Constructor(Type type, Type[] parameters = null) {...}
public static Type Inner(Type type, string name) {...}
public static Type FirstInner(Type type, Func<Type, bool> predicate) {...}
```

　これらのメソッドは **全て** BindingFlagsを使用した実装をしているため、public/privateあるいはstaticの区別なく利用できる。

### Traverse
　リフレクションを介して、フィールド・プロパティ・メソッドにアクセスするためのユーティリティ。
　それらにアクセスできるLINQ的存在。


　クラス内のprivateなフィールド・プロパティ・メソッドへも、Traverseは容易にアクセスできる。途中でnullが返された場合、それ以降伝播させてnullを返す保護機構も実装されている。

```C#
// Type/インスタンスから開始
public static Traverse Create(Type type) {...}
public static Traverse Create<T>() {...}
public static Traverse CreateWithType(string name) {...}

// より深い部分を探る
public Traverse Type(string name) {...}
public Traverse Field(string name) {...}
public Traverse Property(string name, object[] index = null) {...}
public Traverse Method(string name, params object[] arguments) {...}
public Traverse Method(string name, Type[] paramTypes, object[] arguments = null) {...}

// メソッド/Getterを呼び出す
public object GetValue() {...}
public T GetValue<T>() {...}
public object GetValue(params object[] arguments) {...}
public T GetValue<T>(params object[] arguments) {...}
public override string ToString() {...}

// Setterを呼び出す
public Traverse SetValue(object value) {...}

// イテレーション
public static void IterateFields(object source, Action<Traverse> action) {...}
public static void IterateFields(object source, object target, Action<Traverse, Traverse> action) {...}
public static void IterateProperties(object source, Action<Traverse> action) {...}
public static void IterateProperties(object source, object target, Action<Traverse, Traverse> action) {...}
```

#### 使用例

```C#
class Foo
{
	struct Bar
	{
		static string secret = "hello";

		public string ModifiedSecret()
		{
			return secret.ToUpper();
		}
	}

	Bar myBar
	{
		get
		{
			return new Bar();
		}
	}

	public string GetSecret()
	{
		return myBar.ModifiedSecret();
	}

	Foo()
	{
	}

	static Foo MakeFoo()
	{
		return new Foo();
	}
}

var foo = Traverse.Create<Foo>().Method("MakeFoo").GetValue<Foo>();
Traverse.Create(foo).Property("myBar").Field("secret").SetValue("world");
Console.WriteLine(foo.GetSecret()); // outputs WORLD
```

### FileLog

　シンプルで素早いロギングのため、3つのメソッドからなるロギングツールを使用及び提供している。

```C#
public static void Log(string str) {...}
// ログファイル"harmony.log.txt"をデスクトップに新規作成(既存ファイルが無い場合)*str*の内容を書き出す。

public static void Reset() {...}
// ログファイルを削除する。

public static unsafe void LogBytes(long ptr, int len) {...}
// Log(string str)とだいたい同じだが、バイナリダンプやmd5ハッシュをログに出力する。
```

<a name="reference"></a>
## 参考

　[開発元ドキュメント](https://github.com/pardeike/Harmony/wiki)

　ノート:[DLLLoader.cs] ONIでHarmonyの起動とMODの読込みをしているライブラリ名
