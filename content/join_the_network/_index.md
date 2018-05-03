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
ここでハッシュ値`QmT78zSu...`のコンテンツ（すなわち”hello world”）は既に多数のピアが保持しているのが分かります[^2]。これは世界中の多くのIPFSユーザーが最初に"hello world"を自身のノードのリポジトリに登録するためと考えられます。IPFSはコンテンツ指向のプロトコルのために、誰がどのようなタイミングでどのピアのリポジトリに登録したかは関係なく同じコンテンツであれば同一のものと扱います。

誰かが`ipfs cat QmT78zSu...`で"hello world"を取得しようとした場合には、IPFSネットワーク全体に「誰か`QmT78zSu...`のコンテンツを持っていないか？」という問い合わせを投げ、その結果返事が帰ってきた上記のような`QmT78zSu...`を保持するピアのうちもっとも自身に近いピアからコンテンツをダウンロードします。またコンテンツを取得したピアは取得元と同様に他のピアとそのコンテンツを共有することになります。このファイル共有の仕組みにより情報が永続的に保持され続けることになります。この動作はまるで「バカの壁」の書籍を読みたいと思った時に「誰か持っていない？」と周りに問い合わせて、偶然持っていた近くの友人から借りるといった動作と似ていることに気づくと思います。これがコンテンツ指向での情報のやり取りの流れになります。

この動作を少し違った角度で見るために、まだ他のピアには登録されていない（であろう）コンテンツでIPFSの動作を見て見ましょう。他のピアには登録されていないようなコンテンツとして`ifconfig`コマンドの結果をハッシュ化した情報をIPFSネットワーク内で共有してみます[^3]。適当なワーキングディレクトリ以下で
```
$ ifconfig |sha256sum > ifconfig_hash.txt
$ cat ifconfig_hash.txt
01a0e60650c96bca05be0ed61c2ed0e79773f924eba38893d7c5527e0771b6ea  -
$ ipfs add ifconfig_hash.txt
added QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1 ifconfig_hash.txt
```
を実行しIPFSネットワークに登録します。
```
$ ipfs dht findprovs QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
```
今回のコンテンツは世界に一つしかないコンテンツのため、このコンテンツを保持しているピアは自分自身だけです。さて、この状態でパブリックゲートウェイを経由してブラウザからこのコンテンツを取得してみましょう。chromeなどのアドレスバーに
```
https://ipfs-gateway.decentralized-web.jp/ipfs/QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
```
を入力してアクセスします。ここでURL末尾の`QmX5HrsX...`は自身の環境でのハッシュ値に置き換えてください。ブラウザ上でコンテンツが表示されるはずです。その後もう一度このコンテンツを保持しているピアを表示させるコマンドを実行すると
```
$ ipfs dht findprovs QmX5HrsXKMwBPG6gcHyXnZ9TvjpUTxSAGJPczQQWptqbo1
QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
QmVU5iBx5a8tKJimkPzn3mzeGfEW8LjwgZV1xV4yu4bjaD
```
のようにパブリックゲートウェイのピア`QmVU5iBx...`にもコンテンツが保持されているのがみて取れます。これはブラウザからコンテンツ取得を依頼されたパブリックゲートウェイのピアからIPFSネットワークにコンテンツの保持者を問い合わせ、保持している`ipfs add`をしたピアからコンテンツを取得しブラウザに返却。コンテンツをパブリックゲートウェイのピアに保持し続けている状態になります。

{{% notice note %}}
ここで`https://ipfs-gateway.decentralized-web.jp/`は筆者が開設しているパブリックゲートウェイです。Protocol Labsの開設している本家のパブリックゲートウェイ（ https://ipfs.io/ipfs ）は、保持期間を極端に短くしている（またはコンテンツの保持をしないようしている？）ため、上記のようなIPFSの仕様通りの標準的な動作をしません。そのためここでは筆者が開設しているパブリックゲートウェイを指定して標準的な動作を見るようにしています。
{{% /notice %}}


### 【参考】パブリックゲートウェイを構築する
IPFSネットワークにアクセスするもっとも簡単な方法は、WebブラウザからIPFSパブリックゲートウェイにアクセスする方法です。一般に常時公開されているゲートウェイはいくつかあり、先の節で出てきた`https://ipfs.io/ipfs`や`https://ipfs-gateway.decentralized-web.jp/`がそれに当たります。もちろんこれらのゲートウェイ経由でアクセスをしても構いませんし、あなたが技術的に興味があれば自身でゲートウェイを構築し公開することも可能です。ここではパブリックゲートウェイ構築方法の例を示します。（以降、go-ipfsが稼働するサーバがグローバルIPアドレスとドメインを持っておりDNS設定されている前提で進めます。）

{{% notice note %}}
パブリックゲートウェイを構築する最も簡単な方法は、go-ipfsの設定ファイル[^1]内の`Addresses`項目で`"Gateway": "/ip4/127.0.0.1/tcp/8080"`とローカルホストのみが許可されているところを、全てのIPアドレスからのアクセスを許可するように`"Gateway": "/ip4/0.0.0.0/tcp/8080"`に変更することです。この設定状態で`ipfs daemon`コマンドを実行すれば、任意のクライアントのブラウザから、たとえばURL`http://[当該サーバIPアドレス]:8080/ipfs/QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o`にアクセスすることが可能になります。しかし、後述の通りセキュリティや運用の側面からこの方法はあまり推奨されません。
{{% /notice %}}

上記Noteのようにgo-ipfsを直接インターネットに公開することで手軽にパブリックゲートウェイを構築することはできます。しかしセキュリティの懸念があることや`http`のアクセスのみで`https`でのアクセスを許可することができないこと、またアクセスが増えた場合の負荷分散が難しいというようにデメリットが少なくありません。そのためにWebサーバ（ここではnginxを想定します）をリバースプロキシとして利用しgo-ipfsへのアクセスを中継する方法を取るのが一般的です（下の構成図参照）。以下に`Ubuntu 16.04 LTS`環境での具体的な手順例を示します。

#### 1. go-ipfsの自動起動設定
先にインストールしているgo-ipfsをサーバ再起動時などに自動起動する設定を行います。
```
$ sudo sudo vi /lib/systemd/system/ipfs-daemon.service
```
で下記の内容が記されたファイルを生成します。ここではコマンド`/usr/local/bin/ipfs daemon`が`ubuntu`ユーザー＆グループで実行されることを指定しています。（その他の設定の詳細については[ここ](http://enakai00.hatenablog.com/entry/20130917/1379374797)が参考になります。）
```
[Unit]
Description=ipfs daemon

[Service]
ExecStart=/usr/local/bin/ipfs daemon
Restart=always
User=ubuntu
Group=ubuntu

[Install]
WantedBy=multi-user.target
```

用意したファイルでの自動起動を有効化し、実際にサーバ再起動時にgo-ipfsが自動起動されるかを確認します。
```
$ sudo systemctl enable ipfs-daemon
Created symlink from /etc/systemd/system/multi-user.target.wants/ipfs-daemon.service to /lib/systemd/system/ipfs-daemon.service.
$ sudo shutdown -r now #自動起動されるか実際にサーバ再起動し確認。
$ ps aux |grep ipfs #プロセス確認
ubuntu     639     1  4 11:21 ?        00:00:05 /usr/local/bin/ipfs daemon
ubuntu     942   929  0 11:23 pts/0    00:00:00 grep --color=auto ipfs

```

#### 2.リバースプロキシに用いるnginxのインストールと設定
webサーバはapacheなどもありmod_proxyを導入することでリバースプロキシとして利用することも可能ですが、ここではwebサーバとしてnginxを採用したいと思います。nginxをまずサーバにインストールします。
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install nginx

```
下記ファイル内容のようにnginx設定を`vi`などを用いて当該ファイル内容を変更します。
```
$ cat /etc/nginx/sites-available/default

# http(80番ポート)アクセスをhttps(443番ポート)にリダイレクトする設定
server {
    listen 80;
    server_name ipfs-gateway.decentralized-web.jp;
    return 301 https://$host$request_uri;
}

# httpsへのアクセスを受けた場合の設定
server {
  listen 443 ssl default_server;
 	index index.html index.htm index.nginx-debian.html;

  # SSL証明書関連の設定。
  # 証明書はLet's encrypt（https://letsencrypt.org/）を利用して用意。
  # 設定内のドメインは各環境に合わせること。
	server_name pfs-gateway.decentralized-web.jp;
	ssl_certificate	/etc/letsencrypt/live/ipfs-gateway.decentralized-web.jp/cert.pem;
	ssl_certificate_key /etc/letsencrypt/live/ipfs-gateway.decentralized-web.jp/privkey.pem;

  # バックエンドのgo-ipfsとの連携設定。
  # ドメイン以下のアクセス全て（/）をgo-ipfsへ。
	location / {
	proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
	}
}
```
上記nginxの設定ファイル内のコメントにも書いたようにHTTPS通信する際の証明書は[Let's encrypt](https://letsencrypt.org/)を利用して証明書を発行しています。証明書発行は自動で行われ容易にかつ手早く発行が可能です。Let's encryptについては[ここ](https://knowledge.sakura.ad.jp/5573/)で詳しく解説してくれているので参考にしてください[^4]。

最後にもう一度サーバ再起動を行いnginxとgo-ipfsが自動起動されることや所定のアドレスにブラウザからアクセスしコンテンツが表示されることを確認して完了です。


{{% notice warning %}}
上記の手順はFirewallの設定などセキュリティ関連の手順は割愛しています。不特定多数のアクセスを許可するサーバとしての十分なセキュリティ対策は別途必ず行うようにしてください。
{{% /notice %}}


[^1]: 通常は`$HOME/.ipfs/config`。ここで`$HOME`は（IPFSを起動した）ユーザーのホームディレクトリ。

[^2]: 実際はより多いピアでコンテンツが共有されていますがコマンドの仕様上20件に限定されて表示されています。

[^3]: `ifconfig`コマンドはIPアドレスなどのネットワーク情報の他にネットワークインターフェースの累積トラフィック量などが出力されるため、同じサーバ内でも出力の内容が変わり世界で唯一のコンテンツを作成するのに便利です。

[^4]: もちろん[公式ドキュメント](https://letsencrypt.org/docs/)も充実しています。
