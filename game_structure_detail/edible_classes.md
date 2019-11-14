# 食料関係のクラス構成

## おさえておく必要のあるクラス

### TUNING.FOOD

　食料に関する定数定義群。

### ComplexRecipe

　ゲーム全体で使用されている、複数アイテムを合成する設備のためのレシピクラス。

### ComplexRecipeManager

　ComplexRecipeをリストで管理するsingletonクラス。とはいえ登録はComplexRecipe生成時に自動処理化されるので、利用としてはMakeRecipeIDメソッド(static)を利用するくらい。

### EdiblesManager

　FOOD.FOOD_TYPES_LIST(後述するFoodInfoのリスト)という食料(種別)リストを管理するクラス。何らかのFoodInfoが欲しい場合、ここからGetFoodInfoでIDを指定して取り出す事ができる。しかし既存のFoodInfoの場合、FOOD.FOOD_TYPESで定数定義されている方を参照した方が速度的問題を起こさないはず。

### EdiblesManager.FoodInfo

　EdiblesManagerのインナークラス化しているが、重要なクラス。食料のデフォルトステータスを管理している。全食料はこの情報を作らなければならない。

　生成時にFOOD.FOOD_TYPES_LISTへの登録まで自動化されている。

## 新規食料を追加する場合に必要な事

### 定義

　IEntityConfigの導出クラスを定義する。

```C#
public GameObject CreatePrefab()
{
  // 必ず生成したGameObjectのインスタンスを返す。
}

public void OnPrefabInit(GameObject inst)
{
  // 処理は空でもいい。
}

public void OnSpawn(GameObject inst)
{
  // 処理は空でもいい。
}
```

### 処理

1. EntityTemplates.CreateLooseEntityを使用してGameObjectのインスタンスを生成する。
1. EdiblesManager.FoodInfoのインスタンスを生成する。
1. EntityTemplates.ExtendEntityToFoodでGameObjectのインスタンスとFoodInfoのインスタンスを関連付けたエンティティとして登録する。

## 食料生産設備

　各ConfigクラスのConfigureRecipesメソッドで生産できる食料のComplexRecipeを一括設定しているので、ComplexRecipeを設定する債の参考になる。

クラス | イベントハンドラクラス | 要約
-------|--------------------|--------------------
MicrobeMusherConfig | MicrobeMusher | 微生物粉砕機
CookingStationConfig | CookingStation | 電気グリル
GourmetCookingStationConfig | GourmetCookingStation | ガスレンジ

## 食料

クラス | 要約
-------|--------------------
CookedEggConfig | オムレツ
MushroomWrapConfig | キノコのレタス巻き
GrilledPrickleFruitConfig | グリルベリー
SurfAndTurfConfig | サーフ・アンド・ターフ
ColdWheatConfig | スリートウィートの穀粒
FruitCakeConfig | どろどろベリー
BeanPlantConfig | ノッシュ豆
FishMeatConfig | パクーの切り身
CookedMeatConfig | バーベキュー
PickledMealConfig | ピクルス
SpicyTofuConfig | ピリ辛豆腐
SpiceNutConfig | ピンチャペッパー
ColdWheatBreadConfig | ひんやりパン
BurgerConfig | ひんやりバーガー
PrickleFruitConfig | ブリッスルベリー
SpiceBreadConfig | ペッパーブレッド
SalsaConfig | ベリーの香辛料詰め
MushBarConfig | マッシュバー
FriedMushBarConfig | マッシュフライ
BasicPlantFoodConfig | ミールライス
BasicPlantBarConfig | ライスローフ
LettuceConfig | レタス
FriedMushroomConfig | 焼きキノコ
CookedFishConfig | 焼き魚
TofuConfig | 豆腐
RawEggConfig | 卵の中身
