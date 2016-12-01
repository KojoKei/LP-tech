# ImageJ plugin で面積計測を自動化してみた  
非情報科学研究者 (特に生物系研究者) が ImageJ plugin を作るために超えるべき壁やTipsをまとめます。  

前回までで Java の基礎知識や ImageJ plugin 作成・編集を紹介してきました。今回は解析を自動化する plugin を紹介します。  

## はじめに
ImageJ plugin をどんどん作っていくと、Java の知識と ImageJ plugin の機能 (ImageJ API とか言ったりします) の両方が必要になります。  

Java を書いて、ImageJ API も書く。両方やらなければならないってのが ImageJ plugin のつらいところです。覚悟はいいでしょうか? 筆者はできています。  

## 2値化して面積を計測する

ある輝度を閾値として2値化し、自動的に面積を計測する plugin を作ってみます。名前は AreaCounter あたりにしておきましょう。  

という訳で下記のコードをさくっと。  
画像タイトルと幅、高さを表示するコードです。  

```java
//おまじない
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class AreaCounter_ implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		String title = imp.getTitle();
		IJ.log("file name: " + title);

		ImageProcessor ip = imp.getProcessor();
		int width = ip.getWidth();
		int height = ip.getHeight();

		IJ.log("width: " + width);
		IJ.log("height: " + height);
	}

}
```

![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig3-1.png) 

### うわあああ新しいことがたくさん
ImageJ Plugin 上で画像を扱うためのコードがぞくぞくと出てきましたが、1つずつ解説していきます。  

### ImagePlus型  
ImagePlus型の変数を作ります。int型とかdouble型と同じ、型ってやつですね。特に決まりはありませんが、imp って変数がよく使われます。  

### IJ.getImage()  
IJ型のクラス関数の1つですね。何を言っているかわからなくても心配ありません。ImagePlus と併わせて以下のコードをまるっと覚えてしまいましょう。  

```
// 開いてる画像を読み込む
ImagePlus imp = IJ.getImage();
```

### imp.getTitle()  
ImagePlus型には画像タイトルが格納されています。なので、getTitle() でタイトルを抽出できます。  
第1回目で説明できませんでしたが、String型は文字列のための型です。  

```
//開いてる画像を読み込んで、画像タイトルを表示
ImagePlus imp = IJ.getImage();
String title = imp.getTitle();
IJ.log("file name: " + title);
```

これで、開いてる画像へアクセスできましたね。では次は、画像の輝度情報へアクセスしましょう。  

### ImageProcessor型  
ImagePlus型と似てますね。混乱しないように気をつけましょう。グレースケール画像の輝度を持っている型です。  

特に決まりはありませんが、ip って変数がよく使われます。  

ImagePlus型から抽出できます。つまり、ImageJ で扱う画像 (ImagePlus) は ImageProcessor型 と String型の画像タイトルで構成されています。  

何を言っているかわからなくても心配いりません。下記のコードをまるっと覚えてしまいましょう。  

```
//画像を読み込んで、画像の幅と高さ (単位は pixel) を取得
ImagePlus imp = IJ.getImage();
ImageProcessor ip = imp.getProcessor();
int width = ip.getWidth();  //getWidth関数で幅を抽出
int height = ip.getHeight();  //getHeight関数で高さを抽出
```

### タイトルと幅と高さが混乱してきた
気持ちは非常にわかります。  
画像タイトルは ImagePlus に格納されていて、  
幅と高さ (単位はpixel) はIamgeProcessor に格納されています。  

ややこしいですが、覚えてしまいましょう。  

### 輝度情報へアクセス
画像への輝度情報へアクセスできれば、出来ることがどんどん増えます。  

下記のコードをさくっと。  

```java
//おまじない
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class AreaCounter_ implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		String title = imp.getTitle();
		IJ.log("file name: " + title);

		ImageProcessor ip = imp.getProcessor();
		int width = ip.getWidth();
		int height = ip.getHeight();

		IJ.log("width: " + width);
		IJ.log("height: " + height);

		//輝度を全部表示してみよう
		for (int y = 0; y < height; y++) {
			for (int x = 0; x < width; x++) {
				int intensity = ip.get(x,y);
				IJ.log("Intensity of (" + x + ", " + y + "): " + intensity);
			}
		}
	}

}
```

### for文と ip.get(x,y) が追加された
ip.get(x座標, y座標) で、各座標の輝度が抽出できます。  
下記みたいに書きます。  

```java
//(0,0) の輝度を表示するのはこう
int intensity = ip.get(0,0);
IJ.log("Intensity of (0,0): " + intensity);

//(10, 10) の輝度はこう
int intensity = ip.get(10,10);
IJ.log("Intensity of (10,10): " + intensity);
```

2重の for文で 画像全体をラスタスキャンしています。  
for文はy座標から書いているのがちょっとしたコツでしょうか。  

### さあ2値化しましょう
ある輝度を閾値として、閾値よりも輝度が高ければ輝度255、低ければ輝度0に変更する処理が2値化でしたね。  
そして、輝度255の数を数えて面積を計測します。log出力とかを省いて、下記のコードをさくっと。  

```java
//おまじない
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class AreaCounter_ implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		ImageProcessor ip = imp.getProcessor();
		int width = ip.getWidth();
		int height = ip.getHeight();

		int Thr = 128;   //閾値は128に設定
		int Area = 0;   //面積となる変数を設定

		//画像全体をラスタスキャン
		for (int y = 0; y < height; y++) {
			for (int x = 0; x < width; x++) {
				int intensity = ip.get(x,y);
				if ( intensity < Thr) {
					ip.set(x, y, 0);  //閾値未満を輝度0に
				} else {
					ip.set(x, y, 255);  //閾値以上を輝度255に
					Area++;
				}
			}
		}
		imp.updateAndDraw();
		IJ.log("Area (pixel): " + Area);
	}

}
```

### あとは ip.set() と imp.updateAndDraw()だけ
for文とif文さえなんとか読めれば、上記コードは理解できます。  
変数Thr で閾値を設定して (適当に128としています) 、if文を駆使して 閾値 (Thr) 未満であれば輝度0に。閾値以上であれば輝度255にしつつそのピクセル数を数えます。  
数えたピクセル数が面積ってことになりますね。  

輝度の書き変えは **ip.set(x座標, y座標, 書き変える輝度)** で書きます。  
座標 (10, 10) を輝度200 に変更したい場合はこうですね。  

```java
ImagePlus imp = IJ.getImage(); //画像の読み込み
ImageProcessor ip = imp.getProcessor();  //輝度へアクセスするためにipを設定
ip.set(10, 10, 200);  //輝度の書き変え
```

あと、ImageJ はPlugin内で輝度情報を書き変えても、即座に表示している画像に反映してくれる訳ではありません。一度、マウスでクリックして再描写する必要があります。それだと面倒なので、再描写するためのコードが **imp.updateAndDraw()** です。  

![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig3-2.png) 

## まとめ
1枚のグレースケール画像に対して、自動的に2値化と面積測定を行なう ImageJ Plugin を書くことが出来ました。  
でもここまでは基本的なコードなので、手作業でも出来そうなことを頑張ってコード化しているなぁという印象かもしれませんね。  

さて、じゃあ次回は大量の画像に対する自動処理を行ないましょう。手作業だと果てしない時間のかかる処理であっても、計算機を使えば一瞬で終わる (しかも客観的!) 事例を紹介します。  