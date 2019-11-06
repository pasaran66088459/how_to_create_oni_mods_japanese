# ゲームソースの構造

1. [最も基本的な事](#the_basic)

※ ModUtilライブラリほか、MODのためにKleiが用意した機能については、[HarmonyにプラスしてKleiから提供される機能](klei_mod_flamework_guide.md)を参照。

<a name="the_basic"></a>
## 最も基本的な事

### 全ての建築物のprefabはIBuildingConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内建造物(構造物)のprefabは **IBuildingConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **BuildingConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### 全てのアイテムのprefabはIEntityConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内アイテムのprefabは **IEntityConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **EntityConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### 定数が集まっているディレクトリ

#### STRINGS

　ゲーム内で画面表示する文字列(NAME,DESC 等)が列記されている。定数は全て多言語対応のためにLocString型になっている。各国語に向けた.poファイルに対応した内容になっている。

#### TUNING

　ゲームの各種定数が定義されている。のだが、圧倒的に足りず、ほとんどが各所でハードコーディングされていたりする。

　*このディレクトリはどういう方針で運用しているのだろうか？*
