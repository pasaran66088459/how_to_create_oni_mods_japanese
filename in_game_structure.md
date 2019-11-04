# ゲームソースの構造

1. [最も基本的な事](#the_basic)

<a name="the_basic"></a>
## 最も基本的な事

### 全ての建築物のprefabはIBuildingConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内アイテムのprefabは **IBuildingConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **BuildingConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### 全てのアイテムのprefabはIEntityConfigを継承したクラスとして定義する

　MODも含め、ONIの全ゲーム内アイテムのprefabは **IEntityConfig** インタフェースを継承する必要がある。同インタフェースを継承したprefab(候補)クラスは **EntityConfigManager** クラスが全て自動的にゲーム内にGameObjectとして展開する。

### ModUtilライブラリを使用する

　**ModUtil** クラスは、まさしくMODで使用できるようにKlei側で用意してくれたユーティリティクラスで、内容はStaticなメソッドの集合体になる。
　ModUtilで提供されている機能（メソッド）は次のリストの通り。

- `void AddBuildingToPlanScreen(HashedString category, string building_id)`
  - 建造物を(ゲーム画面でいう画面左下の)建造計画欄に追加する。
- `void AddBuildingToHotkeyBuildMenu(HashedString category, string building_id, global::Action hotkey)`
  - 建造物をホットキーに登録する。
- `KAnimFile AddKAnimMod(string name, KAnimFile.Mod anim_mod)`
  - アニメーションの追加。
  - 流儀に従っている限りアセットが自動で読み込まれるので用途不明。
  - 同名メソッドのオーバーロードが複数存在する。
    - `KAnimFile AddKAnim(string name, TextAsset anim_file, TextAsset build_file, IList<Texture2D> textures)`
    - `KAnimFile AddKAnim(string name, TextAsset anim_file, TextAsset build_file, Texture2D texture)`
- `Substance CreateSubstance(string name, Element.State state, KAnimFile kanim, Material material, Color32 colour, Color32 ui_colour, Color32 conduit_colour)`
  - Substanceクラスのインスタンスを生成する。
  - 何に使うのか要調査。
- `void RegisterForTranslation(Type locstring_tree_root)`
  - LocString.CreateLocStringKeysと何が違うのか謎？
