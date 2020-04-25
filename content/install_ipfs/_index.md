---
title: IPFSのインストール
weight: 2
pre: "<b>2. </b>"
---

先の章でIPFSはP2P型のファイルシステムであることを説明しました。IPFSを利用するためにIPFSクライアントをインストールする必要があります。ここではGo言語で実装されたクライアントであるgo-ipfsを利用します。

### go-ipfsのインストール

go-ipfsのインストールは非常に容易です。ここでは、`Ubuntu 18.04 LTS`環境での例を示しますが他OSでも同様です。基本的には[go-ipfsのダウンロードサイト](https://dist.ipfs.io/#go-ipfs)よりビルド済みバイナリパッケージをダウンロードして解凍したのち、実行モジュールをPATHが通っているディレクトリに配置するスクリプトを動かすだけです。下記にコマンド例を示します。

```
$ wget https://dist.ipfs.io/go-ipfs/v0.4.23/go-ipfs_v0.4.23_linux-amd64.tar.gz    #URLは、環境やバージョンにより適宜変更してください。
$ tar xvzf go-ipfs_v0.4.13_linux-amd64.tar.gz
$ cd go-ipfs/
$ sudo ./install.sh
```

ここで、下記のように`ipfs version`のコマンドを実行しバージョン情報が表示されれば問題なくインストールされています。

```
$ ipfs version
ipfs version 0.4.23
```

##### ■Tips:  go-ipfsのアップデート

新しいバージョンのgo-ipfsにアップデートをする場合も、上記のインストール手順を新しいバージョンのモジュールで行うことでアップデート可能です。ただし、go-ipfsをデーモンとして起動（後述）している場合はそれを停止してから行ってください。

​

### IPFSリポジトリの初期化

go-ipfsのインストールを終えたら、IPFSのP2Pネットワークに参加するために下記のコマンドを実行してローカルのIPFSリポジトリを初期化します。

```
$ ipfs init
initializing IPFS node at /home/ubuntu/.ipfs
generating 2048-bit RSA keypair...done
peer identity: QmP5PY85F8XjH5gF67qXLCmU3BxyAW7NdFJ1h8Vpk8zu7s
to get started, enter:

    ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
```

実行結果に示されるようにユーザーのhomeディレクトリ直下に`.ipfs`ディレクトリが生成される。このディレクトリがIPFSのリポジトリとなります。

実行結果の最後に示された`ipfs cat ...`のコマンドをコンソール上で実行してみましょう。詳細は後述しますがipfs catはIPFSネットワーク上に保存されているファイルの内容を取得するコマンドで、ここではreadmeファイルの内容を取得しています。

```
$ ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
Hello and Welcome to IPFS!

██╗██████╗ ███████╗███████╗
██║██╔══██╗██╔════╝██╔════╝
██║██████╔╝█████╗  ███████╗
██║██╔═══╝ ██╔══╝  ╚════██║
██║██║     ██║     ███████║
╚═╝╚═╝     ╚═╝     ╚══════╝

If you're seeing this, you have successfully installed
IPFS and are now interfacing with the ipfs merkledag!

 -------------------------------------------------------
| Warning:                                              |
|   This is alpha software. Use at your own discretion! |
|   Much is missing or lacking polish. There are bugs.  |
|   Not yet secure. Read the security notes for more.   |
 -------------------------------------------------------

Check out some of the other files in this directory:

  ./about
  ./help
  ./quick-start     <-- usage examples
  ./readme          <-- this file
  ./security-notes
```

先の章で紹介したとおり、IPFSはコンテンツアドレス型の仕組みを取り入れており、通常のファイルシステムとはファイルの取扱い方法が大きく異なっています。次節より実際にgo-ipfsを操作しIPFS上でファイルがどのように扱われるのかを見ていきます。
