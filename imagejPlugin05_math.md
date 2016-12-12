# ImageJ Plugin で数値計算をしてみる

非情報科学研究者 (特に生物系研究者) が ImageJ plugin を作るために超えるべき壁やTipsをまとめます。  

前回までで大量の画像に対して自動的に2値化・面積計測を施すコードを紹介しました。今回は、面積の平均値や中央値、分散を出力するためのコードを自作してみましょう。  

## はじめに
何度も同じ解析を手作業でする場合は自動化を検討しますね。同様に、何度も同じコードをくり返し書くのは手間ですね。自動化しちゃいましょう。  

自動化の手法としては、自分で作ってしまう **自作関数 (ユーザー定義関数というやつ)** と他人様が作って公開している関数をありがたく拝借する **外部ライブラリ** があります。  
特に外部ライブラリは便利なのでどんどん使っていきましょう。覚えた後から振り返ると、ライブラリを知らなかった頃は極寒の地で全裸で凍えながらなぜ辛いのかわかっていないようなものだったなぁと思うハズです。  


## 自作関数をつくってみる
とりあえず hello, world あたりで何か書いてみましょう。  
ささっと。  

```
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class Area_math implements PlugIn {

	public void run(String arg) {

		helloWorld(10);
		
	}//end of run

	private void helloWorld(int iteration) {
		
		for (int i=0; i < iteration; i++) {
			IJ.log("Hello, world!");
		}

	}//end of helloWorld

}//end of this plugin
```

このコード、何かおかしいですね? いつもいろいろ書いてる所に、**helloWorld (10)** としか書いてません?  
あと、その下に **private** ほげほげ。  

### helloWorld(10) って何さ?  
何かの関数っぽいんですが、Java にも ImageJ にも helloWordl() って関数はありません (多分) 。というわけで、この helloWorld() が自作関数ですね。  
自作関数の説明というか定義はコードの後半でやっています。そのため、本文は短かくなっています (なんたって1行) 。良く使う機能は自作の関数としてまとめて外に書いておけば、タイプミスも減りますし、コードの内容がスッキリして読みやすくなります。  

### private ほげほげって何さ?
このシリーズではあまりjavaについて真面目に解説してこなかったツケがまわってきた気がします。  
メインのコードは、

```
	}//end of run
```

この部分で終わっています。その後の  

```
	private void helloWorld (int iteration) {
```

から

```
	}//end of helloworld
```

までが自作のhelloWorld関数の説明をしています。  
ちなみに、**//end of hogehoge** は私の手癖で、ここで本文が終わるよ、とかここまでが関数の説明だよってコメントを残しているだけです。  

### public と private って何が違うの?
おっと困りました。javaでメソッドを書く時の修飾子なんですが、筆者みたいにjavaをよく知らない生物学者なら、**本文は public、自分で定義した自作関数は private** あたりで区別しておけば不自由は無いと思います (本当かな?) 。  

### helloWorldの後の (int iteration) って何さ?
この説明は逃れられませんね。  
helloWorld関数に渡す引数 (ひきすう) が1つあり、それはint型だってことを定義しています。  

引数って何かって言うと、例えば  

```
ImagePlus imp = IJ.getImage();
ImageProcessor ip = imp.getProcessor();
int intensity = ip.get(10,10);
ip.set(10, 10, intensity + 10);
```

上のコードだと、xy座標(10, 10) の輝度が int型の変数intensity に代入されますね。そして、その後、座標(10, 10) の輝度を10プラスしています。  

```
ip.get(10, 10);
```

上記の例だと、get関数は引数を2つ要求していて、1つ目がx座標、2つ目がy座標です。  

```
ip.set(10, 10, intensity + 10);
```

上記の例だと、set関数は引数を3つ要求していて、3つ目は起き変わる輝度ですね。  

```
helloWorld(10);
```

helloWorld関数の説明に戻ると、引数を1つ、int型の変数 (っていうか10って数値を直接) 渡しています。なので、関数を定義している部分を確認すると、  

```
int intensity = 10;
for (int i=0; i < iteration; i++) {
	IJ.log("Hello, world!");
}
```

となって、ログが10回出力される訳です。  
長いこと説明しましたが、ここは結構大事な所なのでJavaだと敬遠せずについてきて欲しい所です。  

## 平均輝度を求める自作関数を作ってみる  
では早速、平均輝度を求める自作の関数を作ってみましょう。各ピクセルの輝度の総和を求めて、ピクセル数で割り算する関数を自作します。  

```
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class Area_math implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		ImageProcessor ip = imp.getProcessor();

		double meanIntensity = mean(ip);
		IJ.log("meanIntensity: " + meanIntensity);
	}//end of void

	private double mean(ImageProcessor ip) {
		int width = ip.getWidth(); //幅を抽出
		int height = ip.getHeight(); //高さを抽出

		int pixelNum = width*height; //ピクセル数を抽出
		int sum = 0;

		//各ピクセルから輝度の総和を求める
		for (int y = 0; y < height; y++) {
			for (int x = 0; x < width; x++) {
				sum += ip.get(x, y);
			}
		}

		//輝度の総和をピクセル数で割る
		double mean = (double)sum/(double)pixelNum; //キャスト
		return mean; //返り値の設定
	}//end of mean

}//end of this plugin
```

これまでの記事に出てきたコードが多いかもしれませんが、新しい概念が2つ登場しています。  
**返り値と型変換**です。  

### 返り値
再びのget関数とset関数に登場してもらいましょう。  

```
int intensity = ip.get(10, 10);
ip.set(10, 10, intensity+10);
```

よく見ると、get関数は = がついてて、set関数は = がついてないですね。  
get関数はxy座標の引数をもらうと、その座標の輝度が出てくる関数です。なので、輝度を格納するために int型の変数に代入しているんですね。  

一方で、set関数は1番目と2番目の引数で指定した座標の輝度を、3番目の輝度で置き変える関数です。自分自身だけで完結しているので、int型の関数に代入とかの必要がありません。  

前者のget関数が返り値を持つ関数で、後者のset関数が返り値のない関数です。  
返り値がある関数は、使う時に返り値を代入する変数を用意しておく必要がありますね。  

![関数の説明](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig5-1.png)  
筆者の机の上はホワイトボードなんですが、適当に書いてみた関数と引数、返り値の関係です。こんな感じかなぁと。  

### 返り値があれば private の後に書く、なければ void を書く
返り値の概念をわかってもらった後は、返り値の書き方です。  
よく見ると private の後もvoid やら double やら書いてますね。  

```
private void helloWorld (int iteraction) {
```

やら、  

```
private double mean(ImageProcessor ip) {
```

やらですね。  
private と関数名の間には、返り値の方を書きます。int型を返すような関数なら int、doubel型なら double、**返り値がなければ void** を書きます。

### 返り値は return で返す
返り値の説明もここで最後です。  
返り値を持つ関数は、説明しているコードの最後に **return** があります。  
これですね。  

```
return mean; //返り値の設定
```

return の後ろに変数を書くと、その変数が返り値になります。  

### 型変換
自作関数の最後は型変換の説明です。  
型変換は結構大事。計算結果に可笑しな数値が表示されていて何のハグだろうと朝まで悩んでいた結果、int型で割り算していたとか初学者ならきっとあるんじゃないでしょうか？私はよくあります。  


割り算の結果はとかく小数点になりがちですね。割る変数とか割られる変数とかにintがあると、正確な計算をしてくれません。なので、どちらもint型からdouble型に変換します。これが型変換。  

型変換は、変数の前にカッコ書きで変換後の型名を書きます。  

```
double mean = (double)sum/(double)pixelNum; //キャスト
```

ここは型変換をしているんですね。ちなみに型変換はキャストって言います。英語かな？  

## まとめ
ここまでで、自作の関数を自由自在に作成できるようになったかと思います。一度関数を作成すると、その後はコードを再利用できるようになるのでバグが少なくできて大変便利です。  
import の説明までしたかったんですが、長くなってしまったので次回に説明します。  
Mathクラスをimportして標準偏差とかの統計量を求める関数を自作してみましょう。  





























