# ImageJ Plugin で大量の画像に対する自動処理

非情報科学研究者 (特に生物系研究者) が ImageJ plugin を作るために超えるべき壁やTipsをまとめます。  

前回までで、2値化や面積測定に関する基本的な ImageJ Plugin を作ってきました。今回はスタック画像に対する ImageJ Plugin の作成方法を紹介します。  

## はじめに
"諦めなければ夢は必ずかなう" はこの世の真理ですね。夢は「かなえる」か「諦める」かの2択です。人類に対して夢を諦めることを強いる最も大きな要因は寿命だと思います。私はこの世界の面白い漫画を出来るだけ多く読みたいのです。人類の寿命が限られる現代科学では、何十枚の画像であっても手動で面積を測定する時間なんて完全な無駄だと悟りましょう。  

## スタック画像の読み込み
ImageJ はスタック画像として大量の画像を扱います。いきなりですが、スタック画像に対する基本的なコードは下記です。  

```java
//おまじない
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class Stack_AreaCounter implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		int width = imp.getWidth();
		int height = imp.getHeight();
		
		ImageStack ist = imp.getStack();
		int sliceNum = ist.getSize();
		
		for (int i = 1; i <= sliceNum; i++) {
			ImageProcessor ip = ist.getProcessor(i);
		}
	}

}

```

![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig4-1.png =200) 

とりあえず、スライス番号をログ出力するだけのコードです。  
たった数行にたくさんの初めてがつまってますね!  


### あれ、imp.getWidth()だと...
スタック画像を ImagePlus型で読み込んだ場合、幅と高さは imp.getWidth(), imp.getHeight() から抽出します。静止画像の場合は ip.getWidth() でしたね。    

というのも、スタック画像の場合は輝度にアクセスする前に ImageStack型を経由する必要があるのがちょっと注意が必要です。  

### ImageStack型が出てきた
スタック画像は大量に画像が格納されているので、1枚1枚の画像を示している ImageProcessor もたくさんあります。それが ImageStack の中に格納されています。  

例えば、スライス10番目の画像の座標 (10, 10) の輝度にアクセスする場合は以下のコードになります。  

```
ImagePlus imp = IJ.getImage();  //スタック画像の読み込み
ImageStack ist = imp.getStack();  //ImageStack型へ

ImageProcessor ip = ist.getProcessor(10);  //ImaegStack から 10枚目の ImageProcessor を抽出
int Intensity = ip.get(10,10);  //輝度へアクセス
```

ist.getProcessor()のカッコ内に抽出したいスライス番号を渡せば、そのスライス番号の輝度へアクセルできます。  
よくあるのは、for文で各スライスの画像に対応する ImageProcessor をくり返し抽出するコードかと思います。  

### スライスの枚数ってどうやってわかるの?
```
ImagePlus imp = IJ.getImage();
ImageStack ist = imp.getStack();

int sliceNum = ist.getSize();  //スライス枚数の抽出
```

こうやって、ist.getSize() でスライス枚数を抽出できます。  

### for文、1から始めてるけど?
そうなんです。スライス番号って1から始まるんです。  

座標なんかだと0から始まります。例えば、幅100, 高さ100の画像だと、画像左上が (0, 0) で、画像右下が (99, 99) ですね。  

ところが、スライス枚数100枚のスタック画像は、スライス番号1から100まであるんです。そのかわり、0番がありません。なので  

```
//エラーになる
ImageProcessor ip = ist.getProcessro(0);
```

上記コードはエラーになります。注意しましょう。  
for文が1から始まっているので、終わりが   

```
i < sliceNum;
```

ではなく、  

```
i <= sliceNum；
```
なんですね。  

## スタック画像に対する2値化処理と面積の計測
スタック画像の読み込みが出来るようになりました。前回紹介した2値化と面積計測のコードを組み合せせて、各スライスに対する2値化と面積測定を自動的に施してみましょう。  

という訳で下記のコードをさくっと。  

```
//おまじない
import ij.*;
import ij.process.*;
import ij.gui.*;
import java.awt.*;
import ij.plugin.*;
import ij.plugin.frame.*;
//おまじない終わり

public class Stack_AreaCounter implements PlugIn {

	public void run(String arg) {
		ImagePlus imp = IJ.getImage();
		int width = imp.getWidth();
		int height = imp.getHeight();
		
		ImageStack ist = imp.getStack();
		int sliceNum = ist.getSize();

		int Thr = 128;   //閾値は128に設定
		int Area = 0;   //面積となる変数を設定

		for (int i = 1; i <= sliceNum; i++) {
			ImageProcessor ip = ist.getProcessor(i);
			for (int y = 0; y < height; y++) {
				for (int x = 0; x < width; x++) {
					int intensity = ip.get(x,y);
					if ( intensity < Thr) {
						ip.set(x, y, 0);
					} else {
						ip.set(x, y, 255);
						Area++;
					}
				}
			}
			//面積の出力
			IJ.log("Area of " + i  + "slice: " + Area);
			Area = 0; //次のスライスで面積を計測するためにAreaをリセット
		}
		imp.updateAndDraw();  //画像を再描写
	}
}
```

![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig4-2.png)  

前回までのコードと、今回のコードを組み合わせると、もう結構高度な解析が出来てしまいますね。  

### 数値解析のために、タブ区切りで面積を出力
上記のコードでもう充分に完成なんですが、ログ出力の結果をRとか (エクセルとか) で読みこませると、文字情報と数値情報が混ざっていて少し面倒ですね。  
なので文字と数値をタブ区切りで出力してみましょう。  

タブ区切りは **\t** と書きます (Macだとバックスラッシュなんですが、Windowsだと ￥t だったりするんでしかっけか) 。  

```
//変更前
IJ.log("Area of " + i  + "slice: " + Area);

//タブ区切りに変更後
IJ.log("Slice: \t" + i + "\tArea: \t" + Area);
```

これで統計解析に耐え得るデータ出力が可能となりました。  

![plugins_new](https://raw.github.com/wiki/KojoKei/lp-tech/images/fig4-3.png)  

## まとめ
大量の画像で自動的かつ定量的に面積を測定するために、スタック画像に対する自動処理のコードと、統計解析ソフトウェアへ渡せるデータ出力を紹介しました。
ここまで出来るだけで、データ解析の効率はすごく向上されると思います。  

じゃあ次は、平均や分散、中央値などの統計量を ImageJ Plugin の出力にしてみましょう。ライブラリのインポートとか自作関数の作り方を中心に解説していきます。  