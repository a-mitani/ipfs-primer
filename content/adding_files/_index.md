---
title: IPFSへのファイルの追加
weight: 3
pre: "<b>3. </b>"
---

既存の多くのファイルシステムやインターネットの仕組みは、ある情報を取得する場合にその情報のある「場所（サーバやディレクトリ）」を指定して、そこにある情報を取得する方式をとっています（ロケーション指向）。一方IPFSでは、取得したい情報がどこにあるかは意識せず、直接コンテンツ自体に紐づく識別子（アドレス）を元に情報にアクセスとする「コンテンツ指向」型の仕組みを採用しています。本節ではgo-ipfsを操作しながら、IPFSではファイルやその内容（=コンテンツ）がどのように扱われアクセスされるのかを具体的に見ていきます。<!--また本節の最後にコンテンツ指向のアーキテクチャによって得られる多くのメリットについて記載します。-->


### リポジトリにコンテンツを追加する

下記のコマンドを実行して、"hello world"という内容のtest1.txtという名前のファイルを作成してください。

```
$ echo "hello world" > test1.txt
$ cat test1.txt
hello world
```

次に下記のコマンドを実行します。

```
$ ipfs add test1.txt
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o test1.txt
```

ここで`ipfs add`は指定したファイルの内容を「IPFSオブジェクト」としてIPFSのリポジトリに登録するコマンドです。結果で表示された`QmT78zSu....`は今回のファイルの内容である「hello world」をハッシュ化した値であり、これがこのコンテンツの識別子（アドレス）になります。このコンテンツを参照するには`ipfs cat <アドレス>`のコマンドを実行します。そこで今回登録したアドレスの内容を参照して見ましょう。結果として「hello world」が表示されるはずです。

```
$ ipfs cat QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
hello world
```

### 同じコンテンツなら一意のアドレス
次に、全く同じ内容のもう一つのファイルtest2.txtというファイルを作成してIPFSに追加します。

```
$ echo "hello world" > test2.txt
$ cat test2.txt
hello world
$ ipfs add test2.txt
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o test2.txt
```
するとtest1.txtを`ipfs add`した時と全く同じハッシュ値`QmT78zSu....`が表示されることに気づいたはずです。そのためこれを参照するのもtest1.txtの時と全く同じアドレス`QmT78zSu....`を指定してアクセスすることになります。
```
$ ipfs cat QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
hello world
```
このようにIPFSにおいて、異なるファイルで`ipfs add`してもその内容が同一であれば、インプット元のファイルの違いは全く区別されません。これはすなわち`ipfs add`で追加するものは「ファイル」ではなく「コンテンツ自身」つまり今回の例では「hello world」というコンテンツ自体を追加していることを意味しています。またそのコンテンツを参照する場合も、これまでのファイルシステムやインターネットとは異なり、`test1.txt`や`test2.txt`のそのコンテンツがある場所（今回の場合はファイル）を指定するのではなく、`QmT78zSu....`というコンテンツ自体の識別子（アドレス）を示すハッシュ値を指定します。これがコンテンツ指向の思想です。

{{% notice tip %}}
下のコードのようにファイルではなくパイプを利用して直接「hello world」を`add`することも可能であり、またその参照ハッシュ値もファイルでの追加時と同一であることからも、`ipfs add`で追加するものは「ファイル」ではなく「コンテンツ自身」であることが実感できるでしょう。

```
$ echo "hello world" | ipfs add
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
$ ipfs cat QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
hello world
```

{{% /notice %}}

### 巨大なコンテンツは分割されて保存される
IPFSでは、コンテンツのサイズが256KBを超える場合、256KB以下の部品（チャンク）に分割して管理されます。これは主にP2Pのネットワーク上でコンテンツの転送を効率化する目的で行われています。複数の細切れのコンテンツに分割することで、一つのノードから全てを取得するのではなく、コンテンツのパーツを複数のノードから同時に取得してそれらを一つに組み合わせることで、より素早くコンテンツを取得することが可能になります。

実際に大きなファイルがチャンクに分割されて管理する様子を見ていきます。ここでは猫の画像 cat.jpg [^1] を使います。

```
$ wget https://github.com/a-mitani/ipfs-primer/raw/master/static/images/cat.jpg
$ ll cat.jpg # ファイルサイズ確認 約610KB > 256KB
-rw-rw-r-- 1 ubuntu ubuntu 624861 Mar 19 11:13 cat.jpg
$ ipfs add cat.jpg # cat.jpgをリポジトリに登録
added QmZZrDCuCV5A3WsxbbC6UCtrHtNs2eVyfJwF7JcJJoJGwV cat.jpg
```
これで、`QmZZrDCu...`のアドレスにcat.jpgのコンテンツのIPFSオブジェクトが登録されました。ここでこのオブジェクトを少し詳しく見ていきましょう。

```
$ ipfs ls QmZZrDCuCV5A3WsxbbC6UCtrHtNs2eVyfJwF7JcJJoJGwV
QmT97cgDGaFBYJYZfAA3WFa9h6sFhKNkPTPH81S9x3JY3o 262158
QmNaE7A7VszmF2TaBKmkU9Z4HQo347wiXb9ZGch4rPt4aG 262158
Qmd6h47LMqE2oGoFy15aoDcWjSGpj3KzNYuBvkgAaSEarU 100587
```
ここで`ipfs ls`コマンドは指定されたアドレスのIPFSオブジェクトからリンクされているIPFSオブジェクトのアドレスとそのサイズを列挙するコマンドです。`QmZZrDCu...`のオブジェクトから３つのオブジェクトが参照されているのが分かります。そしてこれらの３つのオブジェクトが元のcat.jpgを分割したものになります。


`ipfs cat`を利用することでipfsは３つの分割されたファイルを組み合わせて、元ファイル（cat.jpg）と同一のcat_output.jpgが出力できます。

```
$ ipfs cat QmZZrDCuCV5A3WsxbbC6UCtrHtNs2eVyfJwF7JcJJoJGwV > cat_output.jpg
$ ll cat*jpg   #元ファイルと出力ファイルでのファイルの比較 → 同じ
-rw-rw-r-- 1 ubuntu ubuntu 624861 Mar 19 11:13 cat.jpg
-rw-rw-r-- 1 ubuntu ubuntu 624861 Mar 19 11:52 cat_output.jpg
$ cmp cat.jpg cat_output.jpg    #念のため２つのファイルをバイナリ比較。何も出力されず一致することがわかる。

```

### ディレクトリ
これまでIPFSではコンテンツ自身のハッシュ値がアドレスとなり、それをキーにコンテンツにアクセスする旨を書いてきました。一方でIPFSはさらにコンテンツへの参照を持つディレクトリのオブジェクトの作成が可能です。ディレクトリからコンテンツを指すリンクを辿って構造的にコンテンツにアクセスすることを可能にしています。先ほど作成した`test1.txt`と`test2.txt`ファイルを利用して実際の動きを見ていきましょう。

まずは以下のようにテスト用のディレクトリ`testdir`を作成しその中にtest1.txtとtest2.txtを格納します。

```
$ cat test1.txt
hello world
$ cat test2.txt
hello world
$ mkdir testdir
$ mv test*.txt testdir/ ## test1.txtとtest2.txtをtestdir内に移動
$ ls testdir/
test1.txt  test2.txt
```

そしてディレクトリ自体を`ipfs add`してみます。ここで`-r`オプションは指定したディレクトリ`testdir`以下の全てのディレクトリやファイルを再帰的にリポジトリに追加していくオプションです。

```
$ ipfs add -r testdir
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o testdir/test1.txt
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o testdir/test2.txt
added QmWpxMfLZcDVeMcBJXoZbqpaeXdzXBxoBScYSSb8ctzu8W testdir
```

ここで出力の１行目と２行目はtestdir以下に配置したそれぞれのコンテンツがリポジトリに追加されたことを示すものです。先述と同じくコンテンツ自身を示すためアドレスは共に`QmT78zSu...`となります。またこのアドレスは先ほどのディレクトリ以下に配置していない場合に`add`した場合とも同じであることに注意してください。繰り返しですがこれは、アドレスはコンテンツに一意に決定され、そのコンテンツがどの場所（ディレクトリ）やどのファイル内に存在するかはコンテンツ指向のIPFSのアドレスには関係が無いからです。

3行目の出力はディレクトリがリポジトリに`add`されたことを示す出力です。IPFSにおいてディレクトリの実体は、他のIPFSオブジェクトへの名前付きリンクを持つIPFSオブジェクトです。ディレクトリ内に含まれるリンク情報を`ipfs ls`で見てみましょう。

```
$ ipfs ls QmWpxMfLZcDVeMcBJXoZbqpaeXdzXBxoBScYSSb8ctzu8W
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o 20 test1.txt
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o 20 test2.txt
```
この出力は、今回登録したディレクトリには`test1.txt`と`test2.txt`という「名前付きのリンク」が含まれて、それらは同じ`QmT78zSu...`を参照していることを示しています。

また、コンテンツはディレクトリから「リンク名」を指定して辿るかたちで参照することが可能です。
```
$ ipfs cat QmWpxMfLZcDVeMcBJXoZbqpaeXdzXBxoBScYSSb8ctzu8W/test1.txt #ディレクトリオブジェクトからリンク名を辿ってアクセス
hello world
```
ロケーション指向のインターネット等とは異なり、ここで`test1.txt`を指定しているのはファイル名を指定しているのではなく、あくまでディレクトリの`QmT78zSu...`のコンテンツを指す「リンク名」を指定していることに注意してください。

またディレクトリを作成しても依然、以下のように「hello world」のコンテンツにはそのハッシュ値`QmT78zSu...`により直接アクセスすることも可能であることは、コンテンツ指向の重要な特徴です。

```
$ ipfs cat QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o #直接アクセス
hello world
```
<!--
また、ディレクトリオブジェクトは下記の例のようにリンクを追加することも削除することも、それらを組み合わせてリンク名をリネームすることも可能です。

```
$ # リンクの追加コマンド → ipfs object patch add-link <対象のディレクトリのハッシュ> <リンク名> <リンク先オブジェクトのハッシュ>
$ ipfs object patch add-link QmXMg3DMLH7ouewbPMnvHRziyxq49XkN1UdMbSALmDRv1c "hogehoge" QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o #コマンドを実行すると新たなディレクトリが生成される
QmVhuCdEBoxtXJCwjMUz6oupmL212tRaocEYuuJ5QeKWWt
$ ipfs ls QmVhuCdEBoxtXJCwjMUz6oupmL212tRaocEYuuJ5QeKWWt #リンク情報を見るとhogehogeが追加されている。
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o 20 hogehoge
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o 20 test1.txt
```

```
$ # リンクの削除コマンド → ipfs object patch rm-link <対象のディレクトリのハッシュ> <削除するリンク名>
$ ipfs object patch rm-link QmVhuCdEBoxtXJCwjMUz6oupmL212tRaocEYuuJ5QeKWWt test1.txt   #コマンドを実行すると新たなディレクトリが生成される
QmRe4beQkVRLUgrBj6cYymCeNJvLuqM23epWsVCNZXRij2
$ ipfs ls QmRe4beQkVRLUgrBj6cYymCeNJvLuqM23epWsVCNZXRij2 #リンク情報を見るとhogehogeのみが残っている。
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o 20 hogehoge
$ ipfs cat QmRe4beQkVRLUgrBj6cYymCeNJvLuqM23epWsVCNZXRij2/hogehoge   #新たなリンク名でアクセスすると「hello world」が表示される。
hello world
```

ディレクトリのリンク情報を変更ることは即ちディレクトリのコンテンツを変更することであり、結果そのハッシュ値も変わるります。そのためディレクトリの編集を行うとディレクトリのアドレス自体が変わることに注意してください。
-->

### IPFSオブジェクト
これまで`ipfs add`で、コンテンツやディレクトリをIPFSオブジェクトとして追加しそれにアクセスする方法を見てきました。IPFSではチャンク化されたものを含むコンテンツやディレクトリなどすべて区別なく「IPFSオブジェクト」として扱われます。ところでIPFSオブジェクトの実体はどのようなモノでしょうか？結論からいうとIPFSオブジェクトの実体は、

* IPFSオブジェクトへのリンク（IPFSリンク）の配列（Links）
* データ（Data）

の２つの要素から構成される構造体です。

実際にIPFSオブジェクトをシリアライズ化して表示する `ipfs object get`コマンドを利用して、これまで作成したIPFSオブジェクトを見てみましょう。

まずは、ディレクトリとして作成されたIPFSオブジェクト`QmXMg3DM...`を調べてみます。
```
$ ipfs object get  QmWpxMfLZcDVeMcBJXoZbqpaeXdzXBxoBScYSSb8ctzu8W
{
  "Links":
    [
      {
        "Name":"test1.txt",
        "Hash":"QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o",
        "Size":20
      },
      {
        "Name":"test2.txt",
        "Hash":"QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o",
        "Size":20
      }
    ],
  "Data":"\u0008\u0001"
}
```
出力（見易さのため整形してます）から確かにIPFSオブジェクトが`"Links"`と`"Data"`の２つの要素が含まれており、`"Links"`には「hello world」のIPFSオブジェクトへの`test1.txt`と`test2.txt`という名前のリンクの配列が格納されています。また`"Data"`領域は空となっています[^2]。

またこのディレクトリのリンク先である`QmT78zSu`のIPFSオブジェクトを同じコマンドで調べると、空の`”Links"`と`hello world`が`"Data"`に格納されているのが見て取れます。

```
$ ipfs object get QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
{"Links":[],"Data":"\u0008\u0002\u0012\u000chello world\n\u0018\u000c"}
```


[^1]: [pixabay](https://pixabay.com/photo-2273598/)より拝借したパブリックドメインライセンスの画像をgithubに上げています。

[^2]: 純粋に空ではなく、"\u0008\u0001"は含まれていますが・・・
