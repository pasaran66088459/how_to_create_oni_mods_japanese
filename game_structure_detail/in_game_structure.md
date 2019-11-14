# ゲームソースの構造

1. [最も基本的な事](#the_basic)
1. [Kleiから提供されているMOD機能](klei_mod_flamework_guide.md)
1. [食料関係のクラス構成](edible_classes.md)

※ ModUtilライブラリほか、MODのためにKleiが用意した機能については、[HarmonyにプラスしてKleiから提供される機能](klei_mod_flamework_guide.md)を参照。

<a name="the_basic"></a>
## 最も基本的な事

- M-V-CやM-V-VMなどのデザインパターン指向はないので、いったん頭から捨てる事。
- 全体的にUnityフレームワークと不可分に繋がっており、View-TriggerであるUnity層との独立性はほぼ無いと見た方が良い。
  - ※ MOD開発のためにUnityまで入れる必要は無い。
- 速度を稼ぐため、全体的にstaticなメソッドを多用している。基本的に引数渡しでデータをやり取りしている等、コード構造はC相当で考えた方がいい。一度しか走りそうもない処理などもそうなっているので、過剰な気はするが…
- 文字列を含めかなりの固定値がハードコーディングされている。一応、それなりに固定値を局所的に納める努力は見られる。

### MonoBehaviorの代替クラスKMonoBehaviour

　Unityのイベントハンドラの既定クラスであるMonoBehaviorに一枚被せた、Klei独自のKMonoBehaviorクラスを既定クラスにしている。
- [MonoBehavior リファレンス(Unity)](https://docs.unity3d.com/jp/current/ScriptReference/MonoBehaviour.html)

　※ 内部的にKMonoBehavior2というクラスが(ご丁寧にExampleクラスまで)用意されているが、使用はしていない様子。

### 全ての建築物のprefabはIBuildingConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内建造物(構造物)のprefabは **IBuildingConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **BuildingConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### 全てのアイテムのprefabはIEntityConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内アイテムのprefabは **IEntityConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **EntityConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### 定数が集まっているディレクトリ

#### STRINGS

　ゲーム内で画面表示する文字列(NAME,DESC 等)が列記されている。定数は全て多言語対応のためにLocString型になっている。各国語に向けた.poファイルに対応した内容になっている。

#### TUNING

　ゲームの各種定数が定義されている。のだが、圧倒的に足りず。

　*このディレクトリはどういう方針で運用しているのだろうか？*
