# ImageJ Plugin 番外編 vimで書いてコンパイル  

非情報科学研究者 (特に生物系研究者) が ImageJ plugin を作るために超えるべき壁やTipsをまとめます。  

今回は番外編で、ImageJ Plugin を vim で書いてシェル上でコンパイルする方法をまとめてみました。  

## はじめに
本編では ImageJ デフォルトのエディタで書くことを推奨してはいるのですが、私自身は vim で書いてます。エディタは好きなものを使えば良いと思うんですが、まあ普通の大人は vim ですよね。  
大規模な開発はしていなくても、vim で書いた後にImageJ で開いてcompileして...となると面倒なのでシェル上で完結できると便利なので、その方法をまとめました。  

## まずは vim で書く
とりあえず何でも良いので vim でサンプルコードを書きます。  
hello world! とかを適当に。  
![helloworld](https://raw.github.com/wiki/KojoKei/lp-tech/images/figB1-1.png)  

## コンパイルはどうしよう？
このまま不思議な力を使って javac とかでコンパイルできれば良いんですが、無理なので、ImageJで一度開いてコンパイルしていました。しかし、いちいちImageJを開いてマウスでポチポチ作業することが面倒でたまりません。なので、mavenを利用してImageJを介さずにコンパイルします。  

## 開発環境は？
とりあえずvimで出来ることはvimでやります。Eclipseを使った場合のやり方は、秀潤社 「IamgeJ ではじめる生物画像解析」に詳しいです。  


## maven
プロジェクト管理ツールですね。ビルドに必要なライブラリが自動的にダウンロードされます。IamgeJ関連のライブラリも maven を使ってダウンロードしてくるんですね。  

mavenを利用してIamgeJ Plugin を作成するのに必要なファイルがとりあえず3種類あります。  

- ソースコード
- POM.xml ファイル
- plugin.config ファイル

## POM.xml に必要な情報を書く
mavenの設定に必要な情報を書き込みます。XML言語というやつです。データ構造を扱いやるい言語らしいですが、筆者は知りませんでした。  

「IamgeJ ではじめる生物画像解析」にもありましたが、github上でImageJ用の[**POM.xml テンプレート**](https://github.com/imagej/minimal-ij1-plugin) が公開されています。  

テンプレートにそって必要な箇所を書き変えたり、削除したりすれば良いんですが、最低限書くべきなのは下記情報です。  
![minimalPOM](https://raw.github.com/wiki/KojoKei/lp-tech/images/figB1-2.png)  

parentの箇所に ImageJ 用の親POM を指定すれば、あとは  

- groupID
- artifactID
- version

は自分のPlugin用なので、好きに書いても大丈夫です。  

テンプレートの POM.xml だと開発者情報やgithubとの連携設定を記入していましたが、ここらへんは書いても書かなくても大丈夫な項目です。  
ちなみに、pom-imagej は2016年12月現在の最新はver16.6.0でした。  

## plugin.config
ImageJ がjarファイルを読み込む時、どのメニューバーに追加してどのファイルを実行するかを参照するファイルです。書いている内容は3つだけ。  

- menu
- "menu label"
- class name

```
Plugins, "hello world", hello_world
```
とだけ1行かけば完了です。  

ImageJ の Pluginsメニューバーに hello world って項目が追加され、hello_world.class が実行されることを示しています。  
ちなみに、Plugins 意外のメニューバーに自作pluginを表示させることもできますよ。というか、ImageJ自体がいろんな人が書いた plugin を吸収しながらバージョンアップしていってます。  

## mavenでビルドするための準備
まず、ソースコードとPOM.xmlファイル、plugin.configファイルを下記のように配置します。  
![tree](https://raw.github.com/wiki/KojoKei/lp-tech/images/figB1-4.png)  

はじめてmavenを利用する場合は、mavenをインストールしましょう。  
筆者は homebrew から install しました。  

```
% brew install maven
```

あと、mavenは java 1.7 が必要です。
もし古ければ[Oracleさん](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)から新しいバージョンを入手しましょう。  
![jar](https://raw.github.com/wiki/KojoKei/lp-tech/images/figB1-3.png)  

(画像)  

## compile
これで準備が整いました。  
plugin のルートディレクトリ (筆者だと、Desktop/hello_world) に移動して、  

```
% maven package
```

で自動的にビルドされます。  

で、classファイルとか jarファイルが作成されました。  
![helloworld](https://raw.github.com/wiki/KojoKei/lp-tech/images/figB1-5.png)  

作成された hello_world.class とか、hello_world-version-.jar を ImageJ Plugin フォルダに移せば完了です。  

## まとめ
長々と書きましたが、とりあえずシェル内でIamgeJ plugin の作成を完結させるやり方でした。準備するフィアルが多いので、githubとの連携を前提にした方が苦労が報われるかもしれません。  
一方で、terminal アプリ内で作業を完結できることに大きな精神衛生上のメリットがあると思います。  

