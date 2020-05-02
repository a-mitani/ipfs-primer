---
title: FAQ
weight: 7
pre: "<b>7. </b>"
---

### Q: IPFSリポジトリを再度初期化するには？
IPFSのノードのリポジトリ情報は全て`${HOME}/.ipfs`ディレクトリ内に保存されています（ここで`${HOME}`はユーザーのHOMEディレクトリです）。そのため、本ディレクトリとその内容を全て削除し、再度`ipfs init`コマンドを実行することで再初期化が可能になります。以下、ubuntu環境でのコマンド例です。

※ 事前に`ipfs daemon`でipfsが動いている状態の場合は事前に`kill`しておく必要があります。

```
$ cd ~ #HOMEディレクトリへの移動
$ ls .ipfs
blocks  config  datastore  datastore_spec  keystore  version
$ rm -rf .ipfs
$ ipfs init
initializing IPFS node at /home/ubuntu/.ipfs
generating 2048-bit RSA keypair...done
peer identity: QmTfTzbaoo4K98VsfG7efr3A4FErsEXAc5utcWjnsCaiz1
to get started, enter:

	ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
```
