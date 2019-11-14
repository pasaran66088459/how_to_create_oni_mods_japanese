# Kleiから提供されているMOD機能

1. [ModUtilライブラリ](#mod_util)
1. [Harmonyのパッチに加えて記述できるMOD準備・後始末用メソッド](#mod_patch)

<a name="mod_util"></a>
## ModUtilライブラリ

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

<a name="mod_patch"></a>
## Harmonyのパッチに加えて記述できるMOD準備・後始末用メソッド

　次のメソッドを含むクラスを定義しておくと、ONIはMODロード時に解釈して自動的に実行する。

- `public static void PrePatch(object harmonyInstance)`
- `public static void PostPatch(object harmonyInstance)`
- `public static void OnLoad()`
  - または
  - `public static void OnLoad(object path)`

### PrePatch(Object harmonyInstance)

　Harmonyによるパッチングが実行される **前に** まとめて実行されるメソッド。HarmonyのHarmonyInstanceオブジェクトが引数で渡される。

### PostPatch(Object harmonyInstance)

　Harmonyによるパッチングが実行された **後に** まとめて実行されるメソッド。HarmonyのHarmonyInstanceオブジェクトが引数で渡される。

### OnLoad()

　MODのローディングの **最後に** まとめて実行されるメソッド。引数を指定すれば、MODのディレクトリ情報が文字列で渡される。

### 使用例：

```C#
public static class Patches
{
    // MODの準備や後始末等
    public static class ModInitializer
    {
        public static void PrePatch(object harmonyInstance)
        {
            // ...
        }

        public static void PostPatch(object harmonyInstance)
        {
            // ...
        }

        public static void OnLoad()
        {
            // ...
        }
        // または
        public static void OnLoad(object path)
        {
            // ...
        }
    }

    // ここからパッチ本体
    [HarmonyPatch(typeof(Xxxx))]
    [HarmonyPatch(nameof(Xxxx.XxxxMethod))]
    public class Xxxx_XxxxMethod_Patch
    {
        public static void Prefix()
        {
            // ...
        }
        // ...
    }
}
```
