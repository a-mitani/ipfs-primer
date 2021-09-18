---
title: コンテンツ共有
type: docs
---

# コンテンツ共有
前節では単一のノード内で、IPFSの特徴であるコンテンツ指向のプロトコルにおいて種々のファイル（コンテンツ）がどのように扱われるかを見てきました。本節では自身のノードをIPFSネットワークに参加させることで、IPFSがP2Pのネットワークの中で様々なコンテンツがどのように共有されていくのかを見ていきます。

{{< hint danger >}}
自身のIPFSノードを接続することで、ノードのリポジトリに登録されたコンテンツはネットワーク上に公開されることになります。そのため公開したくない秘密のコンテンツは接続前に[リポジトリを再初期化する](/faq/#q-ipfsリポジトリを再度初期化するには)などして削除するようにしてください。
{{< /hint >}}


## IPFSネットワークへ接続する

自身のノードをIPFSのP2Pネットワークに追加するには`go-ipfs`をデーモンとして起動します。これにより`go-ipfs`は自動でネットワークに接続された他のノード（ピア）を探索し接続する動作が行われます。（もちろん自身のノードがインターネットに接続されている必要があることに注意してください。）

```bash
$ ipfs daemon   #go-ipfsをデーモンとして起動
Initializing daemon...
go-ipfs version: 0.5.0
Repo version: 9
System version: amd64/linux
Golang version: go1.13.10
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/160.16.52.199/tcp/4001
Swarm listening on /p2p-circuit
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/160.16.52.199/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
WebUI: http://127.0.0.1:5001/webui
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
```
{{< hint info >}}
IPFSノードは他のピアと通信するためにデフォルトではTCPポート4001を利用しています。そのためFirewallなどでポートを閉じている場合は4001を開放してからデーモンを起動してください。
{{< /hint >}}

この状態でもう１つ別のターミナルウィンドウを開き、自身のノードの接続状態を見てみましょう。現在自身のノードが接続している他のピアは`ipfs swarm peers`コマンドで確認できます。
```bash
ipfs swarm peers |head
/ip4/104.236.179.241/tcp/4001/ipfs/QmSoLPppuBtQSGwKDZT2M73ULpjvfd3aZ6ha4oFGL1KrGM
/ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
/ip4/178.62.158.247/tcp/4001/ipfs/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd
（中略）
/ip4/98.109.32.211/tcp/4001/ipfs/QmdcDLV5czbEBB1RoYvMxsYUt8KRp5g8GWKsg86cHb2ByD
/ip4/98.114.215.177/tcp/36383/ipfs/QmWABRCqWoYCGyGV2PZ2T3D9upV39yMgkmFCprXVsKvAZr
/ip4/98.217.220.122/tcp/16006/ipfs/QmVdY2H8ZyWKae7X4HoCkmLeGCw6P9ucqSPSAS9KfeohFf
/ip4/99.53.221.81/tcp/4001/ipfs/QmcRw8uibrQmRsbeaCRmFRkxTZG91Jy2WN8kuythaZiopz
```
出力されている各行が自身のノードと接続しているピアの情報を示しています。非常に多くのピアと接続されているのがみて取れます。筆者の環境では約700のピアと接続されていました。他ピアとの接続数の下限および上限はそれぞれ、IPFSの設定ファイル`config`内[^1]の`Swarm.ConnMgr`セクションの`LowWater`と`HighWater`で設定されており、この部分を編集することで変更が可能です。より多くのピアとの接続することでP2Pネットワークの中でアクセスしたいコンテンツが早く見つけることができるため、システムのリソースが許容できる範囲で接続数は多くしておくことが推奨されます。

## コンテンツ共有の動作を見る
ここまでで自身のノードがIPFSのネットワークに接続されました。これにより自身のノードのリポジトリに登録されているコンテンツもネットワーク上で共有されることになります。実際に前節で登録した"hello world"（ハッシュ値`QmT78zSu...`）のコンテンツの共有状態を見てみましょう。`ipfs dht findprovs`コマンドで当該ハッシュ値を指定することで、IPFSネットワーク上でこのコンテンツを保持しているピアの情報が取得できます。
```bash
$ ipfs dht findprovs QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ122Vpk8zu7s
QmNybVkF2E6j1spYuMDxxpLEUeARB3Fya1w1PLwbLsUBKi
QmNkPXCDWZ21tzLhtRqSA2pyAs3CdHkU2txT2JP3f7UrD3
（中略）
QmQSsV89WQDv4mQWGUeosYRddDSmGjhpLXPWmJcmGUegKJ
QmQ9N2TJCjdX69Z6eRyxkjichXsxHk6zTubHumpSkxPBwR
```
ここでハッシュ値`QmT78zSu...`のコンテンツ（すなわち”hello world”）は既に多数のピアが保持しているのが分かります[^2]。これは世界中の多くのIPFSユーザーが最初に"hello world"を自身のノードのリポジトリに登録するためと考えられます。IPFSはコンテンツ指向のプロトコルのために、誰がどのようなタイミングでどのピアのリポジトリに登録したかは関係なく同じコンテンツであれば同一のものと扱います。

誰かが`ipfs cat QmT78zSu...`で"hello world"を取得しようとした場合には、IPFSネットワーク全体に「誰か`QmT78zSu...`のコンテンツを持っていないか？」という問い合わせを投げ、その結果返事が帰ってきた上記のような`QmT78zSu...`を保持するピアのうちもっとも自身に近いピアからコンテンツをダウンロードします。またコンテンツを取得したピアは取得元と同様に他のピアとそのコンテンツを共有することになります。このファイル共有の仕組みにより情報が永続的に保持され続けることになります。この動作はまるで「バカの壁」の書籍を読みたいと思った時に「誰か持っていない？」と周りに問い合わせて、偶然持っていた近くの友人から借りるといった動作と似ていることに気づくと思います。これがコンテンツ指向での情報のやり取りの流れになります。

この動作を少し違った角度で見るために、まだ他のピアには登録されていない（であろう）コンテンツでIPFSの動作を見て見ましょう。他のピアには登録されていないようなコンテンツの例として、ここでは`ip addr`コマンドの結果をハッシュ化した情報をIPFSネットワーク内で共有してみます[^3]。適当なワーキングディレクトリ以下で
```bash
$ ip addr |sha256sum > ip_hash.txt
$ cat ip_hash.txt
01a0e60650c96bca05be0ed61c2ed0e79773f924eba38893d7c5527e0771b6ea  -
$ ipfs add ip_hash.txt
added QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1 ip_hash.txt
```
を実行しIPFSネットワークに登録します。
```bash
$ ipfs dht findprovs QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
```
今回のコンテンツは世界に一つしかないコンテンツのため、このコンテンツを保持しているピアは自分自身だけです。さて、この状態でパブリックゲートウェイを経由してブラウザからこのコンテンツを取得してみましょう。chromeなどのアドレスバーに
```bash
https://ipfs-gateway.decentralized-web.jp/ipfs/QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
```
を入力してアクセスします。ここでURL末尾の`QmX5HrsX...`は自身の環境でのハッシュ値に置き換えてください。ブラウザ上でコンテンツが表示されるはずです。その後もう一度このコンテンツを保持しているピアを表示させるコマンドを実行すると
```bash
$ ipfs dht findprovs QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
QmVU5iBx5a8tKJimkPzn3mzeGfEW8LjwgZV1xV4yu4bjaD
```
のようにパブリックゲートウェイのピア`QmVU5iBx...`にもコンテンツが保持されているのがみて取れます。これはブラウザからコンテンツ取得を依頼されたパブリックゲートウェイのピアからIPFSネットワークにコンテンツの保持者を問い合わせ、保持している`ipfs add`をしたピアからコンテンツを取得しブラウザに返却。コンテンツをパブリックゲートウェイのピアに保持し続けている状態になります。

{{< hint info >}}
ここで[https://ipfs-gateway.decentralized-web.jp/](https://ipfs-gateway.decentralized-web.jp/)は筆者が開設しているパブリックゲートウェイです。Protocol Labsの開設している本家のパブリックゲートウェイ（ https://ipfs.io/ipfs ）は、保持期間を極端に短くしている（またはコンテンツの保持をしないようしている？）ため、上記のようなIPFSの仕様通りの標準的な動作をしません。そのためここでは筆者が開設しているパブリックゲートウェイを指定して標準的な動作を見るようにしています。
{{< /hint >}}



[^1]: 通常は`$HOME/.ipfs/config`。ここで`$HOME`は（IPFSを起動した）ユーザーのホームディレクトリ。

[^2]: 実際はより多いピアでコンテンツが共有されていますがコマンドの仕様上20件に限定されて表示されています。

[^3]: `ifconfig`コマンドはIPアドレスなどのネットワーク情報の他にネットワークインターフェースの累積トラフィック量などが出力されるため、同じサーバ内でも出力の内容が変わり世界で唯一のコンテンツを作成するのに便利です。
