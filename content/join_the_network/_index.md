---
title: コンテンツ共有
weight: 5
pre: "<b>4. </b>"
---

前節では単一のノード内で、IPFSの特徴であるコンテンツ指向のプロトコルにおいて種々のファイル（コンテンツ）がどのように扱われるかを見てきました。本節では自身のノードをIPFSネットワークに参加させることで、IPFSがP2Pのネットワークの中で様々なコンテンツがどのように共有されていくのかを見ていきます。

{{% notice warning %}}
自身のIPFSノードを接続することで、ノードのリポジトリに登録されたコンテンツはネットワーク上に公開されることになります。そのため公開したくない秘密のコンテンツは接続前に削除するようにしてください。
{{% /notice %}}


### IPFSネットワークへ接続する

自身のノードをIPFSのP2Pネットワークに追加するには`go-ipfs`をデーモンとして起動します。これにより`go-ipfs`は自動でネットワークに接続された他のノード（ピア）を探索し接続する動作が行われます。（もちろん自身のノードがインターネットに接続されている必要があることに注意してください。）

```
$ ipfs daemon   #go-ipfsをデーモンとして起動
Initializing daemon...
Adjusting current ulimit to 2048...
Successfully raised file descriptor limit to 2048.
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/160.16.212.47/tcp/4001
Swarm listening on /p2p-circuit/ipfs/QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/160.16.212.47/tcp/4001
Swarm announcing /ip4/160.16.212.47/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Daemon is ready
```

この状態でもう１つ別のターミナルウィンドウを開き、自身のノードの接続状態を見てみましょう。現在自身のノードが接続している他のピアは`ipfs swarm peers`コマンドで確認できます。
```
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

### コンテンツ共有の動作を見る
ここまでで自身のノードがIPFSのネットワークに接続されました。これにより自身のノードのリポジトリに登録されているコンテンツもネットワーク上で共有されることになります。実際に前節で登録した"hello world"（ハッシュ値`QmT78zSu...`）のコンテンツの共有状態を見てみましょう。`ipfs dht findprovs`コマンドで当該ハッシュ値を指定することで、IPFSネットワーク上でこのコンテンツを保持しているピアの情報が取得できます。
```
$ ipfs dht findprovs QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ122Vpk8zu7s
QmNybVkF2E6j1spYuMDxxpLEUeARB3Fya1w1PLwbLsUBKi
QmNkPXCDWZ21tzLhtRqSA2pyAs3CdHkU2txT2JP3f7UrD3
（中略）
QmQSsV89WQDv4mQWGUeosYRddDSmGjhpLXPWmJcmGUegKJ
QmQ9N2TJCjdX69Z6eRyxkjichXsxHk6zTubHumpSkxPBwR
```
ここでハッシュ値`QmT78zSu...`のコンテンツ（すなわち”hello world”）は既に多数のピアが保持しているのが分かります[^2]。これは世界中の多くのIPFSユーザーが最初に"hello world"を自身のノードのリポジトリに登録するためと考えられます。IPFSはコンテンツ指向のプロトコルのために、誰がどのようなタイミングでどのピアのリポジトリに登録したかは関係なく同じコンテンツであれば同一のものと扱います。誰かが`ipfs cat QmT78zSu...`で"hello world"を取得しようとした場合には、上記で表示されたピアのうちの「どれか」からコンテンツが取得されることになります。そしてコンテンツを取得したピアは取得元と同様に他のピアとそのコンテンツを共有することになります。このコンテンツ指向型のファイル共有の仕組みにより

より詳しくその動作を見るために、まだ他のピアには登録されていない（であろう）コンテンツでIPFSの動作を見て見ましょう。

（以降追記予定）
### パブリックゲートウェイを構築する。


[^1]: 通常は`$HOME/.ipfs/config`。ここで`$HOME`は（IPFSを起動した）ユーザーのホームディレクトリ。

[^2]: 実際はより多いピアでコンテンツが共有されていますがコマンドの仕様上20件に限定されて表示されています。
