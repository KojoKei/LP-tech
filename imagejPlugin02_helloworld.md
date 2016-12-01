# ImageJ plugin で Hello world!
非情報科学研究者 (特に生物系研究者) が ImageJ plugin を作るために超えるべき壁やTipsをまとめます。  

前回は最低限の Java についてまとめましたが、今回から早速 ImageJ plugin を作成していきます。  

## はじめに
使うソフトウェアは **ImageJ あるいは Fiji のみ**。  
筆者は Mac 環境ですが、Windows環境でも基本的に同じです。  
ImageJ Plugin を作成・保存することろまでまとめます。  

## Plugins - New - Plugin
![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-1.png)  
ImageJ あるいは Fiji 起動して、**メニューバーのPlugins - New - Plugin** を選択します。  

### Plugin と plugin Filter と Plugin Frameがあるけど?  
特に何も考えずに plugin を選んでしまって問題ありません。 Pluginは他と比べて自由度が高くて、自分で書くコード量が多いです。 書くコード量が多いなら Plugin Filter 等を使えば良いかもしれません。でも、ImageJ のデフォルト機能が使えるようになりますし、その代償はせいぜい数行コードが長くなったりするだけです。  
ここは何も考えずに Plugin で始めてしまいましょう。  

## 開発環境はどうするの?
どうにもしません。  
ImageJ あるいは Fiji に**デフォルトで搭載されているエディタ**で書きましょう。  
便利な統合開発環境 (IDE) である Ecripse や高機能エディタの Vim や emacs 等を使えば開発速度は (精神的にも肉体的にも) 格段に向上します。  
それゆえ、開発環境は沼です。vimmer であれば vimrc を眺めて1日を過した経験がきっとあると思います。  
ここでは何も考えずに標準のエディタで始めてしましましょう。  

## sample code
plugin を開くと、そこにあらかじめ下記のサンプルコードが書かれています。  

```java
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;

public class My_Plugin implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		IJ.run(imp, "Invert", "");
		IJ.wait(1000);
		IJ.run(imp, "Invert", "");
	}

}

```

このうち、下記コードは便利な機能を自動的に読み込んでいます。とりあえずおまじないだと思って読み飛ばしましょう。  

```java
//ここはおまじないなので読み飛ばす
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
```

### おまじないって何さ?

プログラミング関係の文章を読むと結構な頻度で出てきます。こういうちょっとした単語が通じないのが、情報生命科学と (古典的) な生物学の溝だったりするんじゃないかと思います。  

おまじないって単語の意味は**「意味はあるんだけど、説明すると混乱するだけで理解を妨げるので、ちょっと今は説明したくない」**くらいに思って下さい。  

ちなみに、数値計算する時にココに書き足すことになるので、その時また思い出してもらえれば充分です。

### まずは plugin 名を書いてみる  

```java
//変更前
public class My_Plugin implements PlugIn {

//変更後
public class HelloWorld_ implements PlugIn {
```

上記コードの **My_Plugin** が plugin名です。**My_Plugin** から **HelloWorld_** に変更しました。  
ちなみに、plugin名のどこかにアンダースコア必須です。**Hello_World** でも **Hello_world** でも良いですよ。  

### コード本文を書いてみる  

```java
//変更前
	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		IJ.run(imp, "Invert", "");
		IJ.wait(1000);
		IJ.run(imp, "Invert", "");
	}
	
//変更後
	public void run(String arg) {
		IJ.log("Hello, world!");
	}
```

### Hello world! って何さ?  
とりあえず何でも良いからコードを書いて、出力させたい時によく選ばれる単語です。大抵のプログラミング本の1ページ目あたりのサンプルコードによくあります。  
特に意味はないので、深く考えないでおきましょう。  

### sample code の完成
```java
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;

public class HelloWorld_ implements PlugIn {

	public void run(String arg) {
		IJ.log("Hello, world!");
	}

}
```

![sample_code](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-2.png)  
上記のようなコードが書けたら完成です。  **おめでとうございます。**

## コンパイル
完成だと思いました?  
残念、コンパイルが残ってるんでした。  

### Compile and Run
コンパイルっていういのは、上で書いたコードを機械語に翻訳することを言います。R や python は不思議な力で動いているのでコンパイルする必要がありません。とにかく、**Java はコンパイルしないと動かない**くらいで覚えてもらえれば問題ありません。  

![compileAndRun](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-3.png)  

Compile and Run ボタンを押すと、まずはコードの保存を求められます。**コードの保存名は plugin名と一致させる**ことに気をつけて下さい。  
Plugin名は  

```
public class HellowWorld_ implements PlugIn {
```

上記コードの **HelloWorld_** の部分ですね。**アンダースコア**を忘れずに。  

![save](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-4.png)  

### Hello world! が表示された
出来た! 出来ました! **おめでとうございます。**  


## plugin の加筆・修正
Hello, world! ではなく別のメッセージを表示させたいとか、続きを書きたいとか、とにかく plugi を編集したい場合は **Plugins - Macros - Edit...** からコードを編集することができます。  

![save](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-5.png)  

### plugin なのに Macros から編集するの? おかしくない?
筆者もそう思います。  


## ソースコードと実行ファイル
一度 Compile and Run した自作の plugin は、(ImageJの再起動後に) Plugins メニューバーからいつでも選択できるようになっています。  

![save](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-7.png)  

### plugin はどこにあるの?  
コードは **ImageJ フォルダ - plugins フォルダ** の中に保存されています。**HelloWorld_.java** がコードですね。  

同じ場所に **HeloWorld_.class** も保存されていますね。これは何でしょうか...?  

実はこれが plugin 本体です。コードを機械語に翻訳して出来た例のアレです。  

![save](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig2-6.png)  

### 共同研究者に plugin を渡すには  
plugin本体の **HelloWorld_.class** だけ渡せば ok です。共同研究者の **ImageJ フォルダ - plugins フォルダにコピー**すれば Plugins メニューバーから選択できます。  

共同研究者にコードを自由に改変・チューニングすることを許可する場合は **HelloWorld_.java** も一緒に渡しましょう。編集した後に ImageJ で Compile and Run すれば plugin 本体を作り直すことが出来ます。  

## まとめ  
Java と ちょっとした ImageJ の操作を覚えるだけで、あっという間に ImageJ plugin を自作することが出来ました。ここまでくれば、生物画像の解析を自動化する plugin まであと一歩です。  

次回は画像から面積を自動的に計測する plugin を作ってみましょう。  