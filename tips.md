# tips

　ちょっとした小技、小ネタ

## jsonファイルを扱いたい

　ONI本体では、NewtonsoftのJsonライブラリを使用している。なので同じライブラリをNuGetから導入するか、`OxygenNotIncluded_Data\Managed` にある **Newtonsoft.Json.dll** を一緒に参照に組み込めば使える。

## ユニットテスト環境

　NuGetからNUnitを導入することをお勧め。Unityのライブラリにまで食い込んだテストならUnity Test Runnerを使わないとエラーで動かないが、そのUnityでは **NUnit** を採用している。Unityのライブラリから完全に独立した部分なら割と何でも使えるが、どうせならNUnitで統一した方が見通しも良さそう。
