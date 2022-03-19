# Flutterについてアプリエンジニアが勉強してみた。

## Flutterとは

Googleがオープンソースで開発しているUI SDKです。

UI SDKとは、色んなプラットフォームで動作するアプリケーションのUI（User Interface）を開発できるSDK（Software Development Kit）のことです。

Flutterはクロスプラットフォームであるため、1つのコードからiOS、Android、Web、Windows、macOS、Linuxなど複数のプラットフォームで動作するアプリケーションを開発することができます。

そのため多くの企業がiOS、Androidアプリの開発にFlutterを採用し始めています。

私もアプリエンジニアの端くれ、そろそろ勉強せねばならんのです。

## Flutterの特徴

### 独自のレンダリングシステム

一般的なクロスプラットフォームでは、WebViewを使用してHTML/CSSで描画するものや、プラットフォームごとのUIライブラリを呼び出して描画するものがあります。

Flutterでは『Skia』という描画ライブラリが組み込まれた独自のレンダリングシステムによって、ネイティブで実装した場合と比較しても遜色ないほどのアプリを実現できます。

### 各プラットフォームに対応している

先述の通り、色んなプラットフォームに対応しています。

現在、macOSとLinuxはベータ版ですが、今後安定版としてサポートされるとなると、よりクロスプラットフォームとしての地位を確立するのではないかと思います。

### 開発言語はDart（ダート）

Flutterでは「Dart」というプログラミング言語を使用して開発します。

DartもGoogleが開発しているオープンソースです。

静的型付け言語（コンパイル時に変数の型が決まる）で、JavaやJavascriptと似た文法と言われています（まだ使ったことないから知らない）。

## Dartの特徴

ここでは、開発言語として使われるDartの言語特徴についてみていきます。

### Null安全

DartはNull安全をサポートしているため、NullPointerExceptionを防ぐことができます。

Kotlinのように、Null許容、非Null許容の2つの型が明確に区別されています。

おさらいしておくと、Null安全がサポートされている言語では、Nullが入る恐れがある箇所をそのままにコンパイルするとコンパイルエラーが起きます。

ちゃんとNullじゃないかどうか確認するif文を挟んで！という意味でエラー出してくるんです。いい子ですよね。

### async/await

スマホアプリ開発において、サーバとの通信で非同期処理が必要になる場面が多々あります。

Dartでは「async/await」を用いて非同期処理をシンプルに記述することができます。

```dart
final url = Uri.parse('httos://api.github.com/repos/flutter/flutter/issues');
// await＝実行結果を待つ
final response = await http.get(url);
print(response.body);
```

### Extension methods

Extension methods（拡張メソッド）は、定義元のコードを変更することなく拡張メソッドを定義できる機能のことです。

Swiftにもありますね。

Int型に独自の拡張メソッドを書いて使うとか。

```dart
// 組み込み型であるString型に自身の値をint型に変換して値を返却する拡張メソッド

extension on String {
    int parsedToInt() {
        return int.parse(this);
    }
}

void main() {
    final originalValue = '1';
    // 本来String型にはこのようなメソッドはないが、拡張メソッドによって自由に定義・使用できる
    final parsed = originalValue.parseToInt();
    print(parsed * 100) // output: 100
}
```

## クロスプラットフォームの利点

ここで話の方向性を変えて、クロスプラットフォームの利点について解説していきます。

クロスプラットフォーム開発を採用することで、どのようなメリットがあるのでしょうか。

例えばiOSとAndroidの開発では、次のようなメリットが挙げられます。

- 少人数での開発が可能になる
- 仕様変更やバグへの対応コストを低減できる
- プラットフォーム間での実装差異が生まれにくい
- OS間での区別がなくなり、コミュニケーションが円滑・連携が取りやすくなる

このような恩恵は、さまざまなクロスプラットフォーム技術において共通しています。

ではなぜFlutterが選ばれているのか。それを深掘りしていきましょう。

## なぜ『Flutter』が選ばれているのか

有名なクロスプラットフォーム開発技術として「React Native」「Xamarin」「Cordova」などがあります。

それらと比べ、なぜFlutterがキテるのか。

次の3つがFlutterの特徴であり、選ばれる理由です。

- 生産性の高さ
- 習熟しやすさ
- 様々な要件をカバーできる多機能性

### 生産性の高さ

- ホットリロード
- DevTools

### 習熟しやすさ

- オープンソースかつ読みやすい
- 学習リソースが揃っている（公式YouTubeチャンネルもある！）

### 様々な要件をカバーできる多機能性

- pub.devで高品質なパッケージを探せる
- Pluginでプラットフォーム固有の実装を呼び出せる

## Everything is a widget

UIはWidgetを使って構築する

## Flutterの環境構築

必要なものは以下の通りです。

- Flutter SDK
- Xcode
- Android Studio
- Visual Studio Code

### Flutter SDKのインストール

以下のような流れで行います。

- Flutterの公式HPからSDKをダウンロード
  - https://docs.flutter.dev/get-started/install
- Zipファイルを任意の場所で解凍し、パスを通す

パスを通す際は、~/.bash_profileに以下を追加

```sh
export PATH=$PATH:[解凍したディレクトリパス]/flutter/bin
```

### Android Studioのインストール

### Xcodeのインストール

### Visual Studio Codeのインストール

Flutterのプラグインもインストールしておきましょう。

## Flutterプロジェクトの作成

作成したいディレクトリで以下を実行します。
パスが通っていないと実行できないのでご注意ください。

`$ flutter create [プロジェクト名]`

早速試しに実行してみましょう。

`cd`コマンドでプロジェクト内部に入り、`$ flutter run`を実行します。

するとChromeが起動し、Webアプリケーションとしてデフォルトの画面が表示されます。

## Dartに触れてみよう

入れ子がすんごい。
VSCodeだったらプラグイン必須です。
