---
title: 【参考】whitepaper 日本語訳
weight: 7
pre: "<b>7. </b>"
---
本章は参考情報として Juan Benet によって書かれたIPFSの[whitepaper](https://ipfs.io/ipfs/QmR7GSQM93Cx5eAg6a6yRzNde1FQv7uL6X1o4k7zrJa3LX/ipfs.draft3.pdf)「IPFS-Content Addressed, Versioned, P2P File System (DRAFT 3)」の日本語訳を記します。

{{% notice note %}}
本翻訳は直訳を避け、原文の内容を損ねない範囲で（大幅に）意訳しています。そのため必要な場合は[原文](https://ipfs.io/ipfs/QmR7GSQM93Cx5eAg6a6yRzNde1FQv7uL6X1o4k7zrJa3LX/ipfs.draft3.pdf)を参照ください。また、翻訳への協力（翻訳・修正）歓迎です。協力いただける場合は[GitHub](https://github.com/a-mitani/ipfs-primer)にてIssueの発行、またはPull requestをお願いいたします。

{{% /notice %}}

## 概要
Inter Planetary File System(IPFS)はピアツーピア（P2P）型の分散型ファイルシステムであり、全てのデバイスを一つのファイルシステムに結合することを目的としたものです。いくつもの点でIPFSはWebと同様のもの言えますが、一方で、一つのGitリポジトリ上でオブジェクト交換を行うBitTorrentのスウォームと考えることもできます。またはコンテンツアドレス型のハイパーリンク機能をもつストレージとも考えられます。IPFSはMerkle-DAGのデータ構造を持ちます。IPFS上にバージョン管理可能なファイルシステムやブロックチェーン、または永続的なWebを構築することが可能になります。IPFSは分散ハッシュテーブル技術、インセンティブが与えられたブロック交換技術、自己証明型名前空間といった要素技術を組み合わせて実現されており、単一障害点がなくピア同士がお互いの信頼を必要とせずに実現できるシステムです。

## 1. 序論
これまで多くのグローバルな分散ファイルシステムを構築する試みがなされてきました。いくつかは顕著な成果を見せましたが、その他は完全に失敗に終わりました。例えば学術的な取り組みとしては、AFS[6]が成功事例として挙げられ現在も活用されている一方で、その他[7,?]の試みは失敗に終わっています。学術的な試み以外でのP2P型の分散ファイルシステムは大容量コンテンツ（音楽や動画など）の共有用途に発展してきました。特にNapsterやKaZaA、BitTorrent[2]といったアプリケーションは1億人以上のユーザーによって支えられる巨大な分散システムでした。今日でもBitTorrentは1,000万ものノードが動作している巨大なファイルシステムとして動作し続けています[16]。これらのアプリケーションは、学術的な取り組みの中でのファイルシステムよりも多くのユーザーを惹きつけてきました。しかしこれらは別目的で利用される事例[^1]はあるものの、あくまでアプリケーションであり、その上でシステムやアプリケーションが動作するインフラ基盤としてデザインされているものではありません。その点で、一般的なファイルシステムとして利用可能な、グローバル規模で低レイテンシー分散ファイルシステムというものはこれまで現れてきていないといえます。

この理由は既に十分良いシステムが存在していたからと考えられます。「HTTP」です。HTTPはこれまで最も成功した分散ファイルシステムプロトコルです。ブラウザと結びつくことでHTTPは技術的にも社会的にも多くのインパクトを与え、インターネット上でのファイル転送のデファクトスタンダードとなっています。しかしHTTPはそれが世に出て以降の15年の間に生み出されてきた新たな技術を取り込むことには失敗しています。多くの後方互換性の確保やステークホルダーの複雑なしがらみから、Webのインフラ基盤を進化させることは事実上不可能に見える一方で、実際にはHTTPの出現以降も様々な新しいプロトコルが生み出され広範囲で利用されている事実もあります。ユーザーエクスペリエンスを損なうことなく新たな機能を導入するようなHTTP-Webのグレードアップを行なっていく必要があります。

HTTPプロトコルは小容量のデータのやりとりに関して比較的安価で行うことを実現する一方で我々はデータ転送に関して新たな時代に突入しつつあります。それは(a)ペタバイト級データのホストと配信 (b)組織横断の大量データ処理 (c)大容量高精細なオンデマンドのコンテンツストリーミング(d)大規模データのバージョン管理およびリンク処理(e)重要データの消失防止、というようなものが要求される時代です。このような状況のもと、我々はHTTPを捨てて新たなデータ転送プロトコルを開発しました。次のステップはこれをWebそのものにすることが必要です。

データ転送と同等に、バージョン管理技術はデータ操作の協調作業に重要な役割を果たしてきました。ソースコード分散バージョン管理システムであるGitは分散されたデータ操作の実装モデルとして非常に重要であり、Git関連技術は従来の大容量ファイルシステムにはなかったバージョン管理機能の実現を可能にします。Camlistore[?]やDat[?]といったGitにインスパイアされたシステムも出現しています。またGitは「コンテンツアドレスMerkle-DAG」形式のデータモデルを採用しており、この方式は既に分散ファイルシステムに影響を与えています。

本論文ではバージョン管理機能を備えた新しいファイルシステムであるIPFSについて解説します。IPFSは「全てのデータを統一されたMerkle-DAGにモデリングする」という基本原則のもとに設計されています。

## 2. 背景
本節では、IPFSが融合しようとしているP2Pシステムの重要な技術について振り返ります。

### 2.1 分散ハッシュテーブル
分散ハッシュテーブル（Distributed Hash Table: DHT）は、P2Pシステムにおいてメタデータを統合し管理するために広く用いられているものです。例えば MainlineDHTはBitTorrentでピアのトラッキングに利用されています。

#### 2.1.1 Kademlia DHT
Kademliaは広く知られたDHTであり、下記のような特徴があります。

1. 高効率のデータ探索：システムのノード数をnとしたとき、問い合わせ先のノード数は平均 log2(n)のオーダーとなります。（例えば、10,000,000ノードであれば20ホップでデータ探索が可能です。）
2. コーディネーションの低いオーバーヘッド：ノード間でやりとりされるコントロールメッセージ数が最適化されます。
3. より長く接続されているノードを重視することで耐攻撃性を実現しています。
4. GnutellaやBitTorrentなど多くのP2Pシステムで実際に利用されており、2,000万ノードのネットワークを形成している実績があります。

#### 2.1.2 Coral DSHT
いくつかのP2Pファイルシステムはデータブロックを直接DHTに格納する手法を取っていますが、この方法は格納する必要のないノードにもデータを格納する必要性を発生させるため、ストレージとネットワーク帯域を必要以上に消費する欠点があります[5]。Coral DSHTはKademliaを以下の３つの重要な点で拡張した方式です。

1. Kademliaはデータブロックを格納するノードをデータのキーと最も距離（XOR距離）が近いIDをもつノードに決定します。この時、たとえ離れていてもより頻繁にデータを必要とするノードなどという、アプリケーション側のデータの局所性については考慮されません。そのため、ストレージとネットワーク帯域を必要以上に消費する欠点があります。代わりにCoralではデータブロックを格納する「アドレス」を最も距離が近いノードに格納します。
2. Coralは分散ハッシュテーブルのAPIを`get_value(key)`から`get_any_values(key)`と緩和（Distributed Sloppy Hash TableのSloppyの由来）しています。これによりkeyへのアクセスが増加した際に特定のノードに負荷が集中しホットスポットになることを防いでいます。
3. Coralはサイズや地域に応じてDSHTのクラスターの階層を形成します。これにより距離の遠いノードへの問い合わせ数を減らし[5]、探索のレイテンシーを軽減しています。

#### 2.1.3 S/Kademlia DHT
S/Kademlia[1]は、悪意の攻撃を防ぐために、以下の２点についてKademliaを拡張しています。

1. S/KademliaはPKI鍵ペアを利用したセキュアなノードのID生成スキームを導入することによりシビル攻撃を防ぎます。
2. S/Kademliaでは連結されていないパスも対象にデータを探索します。これにより多くの悪意あるノードが含まれるネットワークにおいても善意のノード同士が接続可能にします。実際に半数の悪意あるノードが含まれるネットワーク内においても0.85という（訳者注：データ探索の？）成功率を実現しています。

### 2.2 ブロック交換 - BitTorrent
BitTorrent[3]はファイル共有システムとして大きく成功しているシステムです。信頼性の低いピア（スウォーム）が存在するP2Pネットワークにおいても各ピアが連携してファイルピース（ファイル断片）の共有を行う仕組みを作ることに成功しています。IPFSのデザインに影響を与えたBitTorrentの特徴は以下のものです。

1. BitTorrentでのデータ交換プロトコルは「擬似しっぺ返し戦略」と呼ばれる、より貢献するノードには褒賞を与え、単に他者のリソースを取得するだけのノードには罰則を与える仕組みを採用しています。
2. BitTorrentではファイル断片の希少性をトラッキングし、最も希少なファイル断片から優先してダウンロードするように設計されています。これによりシーダー（ファイルの完全なデータを持つピア）の負荷を下げ、シーダーでないピアもデータ交換に参加出来るようにしています。
3. BitTorrent標準のしっぺ返し戦略は、略奪的帯域幅共有戦略（exploitative bandwidth sharing strategies）に対して脆弱性を持ちます。PropShare[8]戦略はこの種の脆弱性を克服し、かつスウォームのパフォーマンスを改善する戦略として知られます。

### 2.3 バージョン管理システム - Git
バージョン管理システムはファイルシステムに対する変更履歴を管理し、かつその各バージョンを効率的に配布することを容易にしてくれます。バージョン管理システムとして普及しているGitは、ファイルシステム内の変更履歴をMerkle-DAGのオブジェクトにモデル化することで分散システムに親和性の高いシステムになっています。

1. ファイル (blob)、ディレクトリ (tree)、そして変更 (commit)は、それぞれ不可変（イミュータブル）なオブジェクトとして表現されます。
2. それらのオブジェクトはコンテンツから生成されたハッシュ値そのものをアドレスとして参照されます（コンテンツアドレス）。
3. オブジェクト間がリンクにより参照されることで Merkle DAGを構成します。
4. ブランチやタグ等のバージョン管理用のメタデータは単なるポインターとして管理されるため、それらの生成や更新にはほとんどコストはかかりません。
5. バージョンの変更はそれらポインターの変更と差分のオブジェクトの追加として管理されます。
6. バージョン変更の他ユーザーへの配布はオブジェクトの転送とリモート参照の更新です。（＊訳者注：この部分は意味不明です。）

### 2.4 自己証明型ファイルシステム - SFS
自己証明型ファイルシステム（SFS） [12, 11] は、(a) 分散トラストチェーン、(b) 平等でグローバルな名前空間　の２つを実現するための実装です。SFSはこれらを実現するためにリモートファイルシステムのアドレスを
```
/sfs/<Location>:<HostID>
```
というスキームに定義しています。ここで、`Location`はサーバのネットワークアドレスであり、`HostID`は
```
HostID = hash(public_key || Location)
```
で定義されます。このことからSFSファイルシステムでの名前自体がそのサーバ（の正当性）を証明することになります。ユーザーはサーバによって提供される公開鍵を検証し共有鍵を交換することで全ての通信を秘匿化します。全てのSFSのインスタンスはグローバルで暗号学的な名前空間を共有するため、何らかの中央機関による管理が必要としません。

## 3. IPFSの設計
IPFSは先述したDHT、BitTorrent、Git、SFSのアイデアを融合して実現する分散ファイルシステムです。IPFSはこれらの技術を単純化し進化させ一つの技術として融合しています。IPFSは、そこにアプリケーションをデプロイして動作させる新たな基盤として、または新たな大容量データのバージョン管理や配布を行うシステム基盤として活用を可能にするものです。IPFSはwebそのものを進化させる可能性もあります。IPFSはP2Pであり特別なノードを必要としません。 IPFSノードはそれぞれののローカルストレージにIPFSオブジェクトを格納し、ノード同士が繋がり合いこれらのオブジェクト（つまりデータ）を交換します。 IPFSプロトコルは下記の各々個別の役割をもつサブプロトコルに分離することができます。

1. **Identities** - ノードIDの生成と検証を管理します。3.1節に詳述します。
2. **Network** - 下位の様々な通信プロトコルを利用して他ピアとの接続を管理します。3.2節に詳述します。
3. **Routing** - 特定のピアやオブジェクトのロケーション情報を管理し、ローカルやリモートからの問い合わせに対して情報を返します。デフォルトではDHTが利用されますが、変更可能です。3.3節に詳述します。
4. **Exchange** - 効率的な新型ブロック交換プロトコルである`BitSwap`です。ブロックの市場としてモデリングされ、データの複製を緩やかに誘引します。ブロック交換の戦略は変更可能です。3.4節に詳述します。
5. **Objects** - コンテンツアドレス型の不可変（イミュータブル）でリンクをもつMerkle-DAGです。ファイルシステムの階層やコミュニケーションシステムなど、任意のデータ構造を表現することが可能です。3.5節に詳述します。
6. **Files** - Gitに影響を受けたバージョン管理ファイルシステムです。3.6節に詳述します。
7. **Naming** - 自己証明可能かつ変更可能な名前空間システムです。3.7節に詳述します。
これらのサブシステムはそれぞれ独立している訳ではなく、互いに影響し合っています。しかし分かりやすさのため、これらを切り離されたプロトコルスタックとして記述します。

注：下記にデータ構造や機能はGo言語の文法に従い記述します。

### 3.1 Identities
各ノードは`NodeId`によって識別され、これはS/Kademliaで交換される公開鍵を暗号学的ハッシュ関数によりハッシュ化したものとなります[1]。各ノードは自身の公開鍵と（パスワードにより暗号化された）秘密鍵を保持します。ノードの管理者はノードを起動する度に新しいIDを発行することも可能ですが未払いのネットワーク報酬を放棄することとなるため、それが同じIDを使い続けるインセンティブとなります。

```go
type NodeId Multihash
type Multihash []byte
// self-describing cryptographic hash digest
type PublicKey []byte

type PrivateKey []byte
// self-describing keys
type Node struct {
  NodeId NodeID
  PubKey PublicKey
  PriKey PrivateKey
}
```


```go
difficulty = <integer parameter>
n = Node{}
do {
  n.PubKey, n.PrivKey = PKI.genKeyPair()
  n.NodeId = hash(n.PubKey)
  p = count_preceding_zero_bits(hash(n.NodeId))
} while (p < difficulty)
```
IPFSではS/Kademlia方式を基礎にしたID生成を行います。最初に他ノードと接続する際には公開鍵を公開し`hash(other.PublicKey)`が`other.NodeId`と一致するかを確認します。もし異なる場合は接続を遮断します。

#### *暗号学的関数について*
IPFSでの暗号学的ハッシュ値は、何らか特定の関数を使用することをに限定せず、自己表示的な値をもつようにしています。つまり下記のようにマルチハッシュのフォーマット、すなわち、値の先頭部分に使用されたハッシュ関数やダイジェストのバイト数を示すヘッダを持つようにしています。
```
<function code><digest length><digest bytes>
```
これはシステムに対して (a)ユースケース（例えば強固なセキュリティが必要か？それとも速度が重要か？）に合わせてベストな関数を選択する可能にし、(b)暗号の進化に合わせてシステムも進化可能にします。そしてどの関数を利用したものかを自己表示することにより異なる関数の場合でも互換性を確保しています。

### 3.2 Network
IPFSのノードはインターネットを横断して常に100以上もの他ノードと通信しています。IPFSのネットワークスタックには次のような特徴があります。

* トランスポート: IPFSは任意のトランスポート層のプロトコルが利用可能です。特にWebRTC DataChannels[?] (ブラウザ間接続)やuTP(LEDBAT [14])のようなプロトコルに親和性が高いものとなっています。
* 信頼性: IPFSはuTP (LEDBAT [14])やSCTP [15]を利用することで、下層のネットワークプロトコルが信頼性を持たない場合でも信頼性を確保することが可能です。
* 接続性: IPFSはICE-NAT-traversal[13]を利用しています。
* 整合性: IPFSでは任意でハッシュ関数のチェックサムを利用してメッセージ整合性をチェックすることが可能です。
* メッセージ認証: IPFSでは任意で送り主の公開鍵とHMACを利用して、メッセージ認証を行うことがß可能です。

#### 3.2.1 ノードアドレス指定
IPFSでは任意のネットワークが利用可能です。それは即ち、必ずしもインターネットプロトコル（IP）にアクセスすることを想定しておらず、また依存していることもありません。そのためIPFSはオーバーレイネットワークとして利用が可能です。IPFSは`multiaddr`にバイト文字列としてアドレスを保持します。`multiaddr`では下記の例のようにプロトコルの種類とそのアドレスが格納され下層のネットワークに伝えられます。

```
# an SCTP/IPv4 connection
/ip4/10.20.30.40/sctp/1234/
# an SCTP/IPv4 connection proxied over TCP/IPv4
/ip4/5.6.7.8/tcp/5678/ip4/1.2.3.4/sctp/1234/
```

### 3.3 Routing
IPFSのノードは２つのルーティングを行う必要があります。つまり(a)他のノードのネットワークアドレスへのルーティング、(b)オブジェクトのホストする場所へのルーティング、の２つです。IPFSではこれらのルーティングをS/KademliaとCoralを基礎としたDSHTを用いることで実現しています。IPFSでは、Coral[5]やMainline[16]と似たオブジェクトのサイズと利用方法を採用しています。つまりIPFSでは保存するオブジェクトのサイズにより保存方法を切り分けます。1KB以下の小さいサイズのオブジェクトの場合そのオブジェクトを直接DHT上に保存し、それを超えるサイズのオブジェクトの場合はそのオブジェクトへの参照（＝保持しているノードのID）をDHT上に保存します。 このDSHTのインターフェースは次のようなものです。

```
type IPFSRouting interface {
FindPeer(node NodeId)
// gets a particular peer’s network address
SetValue(key []bytes, value []bytes)
// stores a small metadata value in DHT
GetValue(key []bytes)
// retrieves small metadata value from DHT
ProvideValue(key Multihash)
// announces this node can serve a large value
FindValuePeers(key Multihash, min int)
// gets a number of peers serving a large value
}
```

注意: IPFSではユースケースに応じて異なる実装がコールされます。（例えば広域ネットワークではDHTが、ローカルネットワークではstatic-HTがコールされます。）そのためIPFSでは上記のインターフェースに準拠する限りユーザーのニーズに応じてルーティングシステムが切り替えることが可能です。

### 3.4 ブロック交換 - BitSwapプロトコル

IPFSでのデータ配布はBitSwapプロトコルに準拠したピア間のブロック交換によって行われます。BitSwapプロトコルはBitTorrentに強く影響されたプロトコルであり、BitTorrentと同様に各ピアは自分が取得したいブロックのリスト(want_list)と、自分が提供できるブロックのリスト(have_list)を保持します。一方でBitTorrentと異なりBitSwapは１つのTorrentに限定せず、そのブロックがどのファイル（Torrent）の一部かに関係なくブロック交換を行う一種の永続的なブロックの交換市場のように振舞います。ノードはそのブロック交換市場に寄り合ってブロックを交換することになります。交換市場が形成されることは即ち仮想通貨とその取引を登録する台帳が必要になることを意味しますがこれについては今後の論文にて議論することにします。

最も基本的なブロック交換のケースは２つのノード間で直接ブロックを交換しあうことです。しかしこれはお互いに保持しているブロックが相補的、つまり他者が必要なブロックを自身が持っていることが必要となります。しかしこのようなケースはまれであるため、多くの場合で自分が必要なブロックを取得するために他者が必要とするブロックを探し出し保持することでブロック交換が可能になります。これは各ノードが自分自身が必要でないブロックも保持することのインセンティブとして働きます。

#### 3.4.1 BitSwapにおいての信用

BitSwapプロトコルにおいてノードは貸しが帰ってくることを期待して楽観的にブロックを他者ノードに送ります。しかしリーチノード（ブロックを他者に一切シェアすることなくブロックを受け取るのみのノード） からは守る必要があります。そのためBitSwapプロトコルでは単純な信用システムを構築しそれを解決します。

1. ノードは（他者ノードと送受信したブロックのバイト単位での）収支をトラッキングします。
2. ノードは債務ノード（受信ブロックのバイト数が送信ブロックのそれよりも大きいノード）にはその債務が増えるに従い小さくなる確率関数にしたがい、確率的にブロックの送信を行います。

上記の仕組みでノードが特定の他者ノードに送信を行わないことになった場合、その後`ignore_cooldown`の時間だけ経過するまではその他者ノードを無視し続けることに注意してください。これは確率が低くても試行回数が多くなると送信のチャンスが増えてしまうことを防ぐためです。（BitSwapプロトコルではデフォルトで10秒に設定されています。）

（以降追記予定）

## 4. 今後
IPFSを構成する多くのアイデアは学術界での研究とオープンソースのコミュニティの中で数十年にわたって育まれたものになります。独自プロトコルであるBitSwapを除き、IPFSの主な貢献はそれらの数々のアイデアを統合したことです。IPFSは非中央集権なインターネットのインフラを提供するという野心的なプロジェクトであり、そのインフラ上で今後様々なアプリケーションが動作することが可能です。控えめに述べてもこのIPFSはグローバルに広がるバージョン管理可能なファイルシステムや名前空間として、そして次世代ファイル共有システムとして利用可能です。さらにはIPFSはWebの新たな地平を広げるものになるでしょう。それは、価値ある情報を提供者自身がホストすることなくそれを必要とする人自身がホストする、そして情報を受け取る側はホストの信用に関わらず改変や逸失の心配なく必要な情報が入手できるようになります。そのような永続的Webの世界に、IPFSは我々を導いてくれるでしょう。

## 5. 謝辞
IPFSは多くの既存の素晴らしいアイデアやシステムの複合体であり、それら無くしてIPFSという壮大な目標を打ち立てることは出来なかったでしょう。また、David Dalrymple、Joe Zimmerman、そして Ali Yahyaに感謝します。 彼らはそれぞれ、一般マークルDAG（David, Joe）、ハッシュブロック(David), s/Kademliaでのシビル攻撃への防御（David, Ali）のアイデアについて長いディスカッションに付き合ってくれました。また、David Mazieresの素晴らしいアイデアに特別に感謝します。

## 6. Reference TODO

## 7. Reference

［1］ I. Baumgart and S. Mies. S/kademlia:
A practicable approach towards secure key-based routing. In Parallel and Distributed Systems, 2007 International Conference on, volume 2, pages 1–8. IEEE, 2007.

[2] I. BitTorrent. Bittorrent and A¸ttorrent software ˆ
surpass 150 million user milestone, Jan. 2012.

[3] B. Cohen. Incentives build robustness in bittorrent. In
Workshop on Economics of Peer-to-Peer systems,
volume 6, pages 68–72, 2003.

[4] J. Dean and S. Ghemawat. leveldb–a fast and
lightweight key/value database library by google, 2011.

[5] M. J. Freedman, E. Freudenthal, and D. Mazieres.
Democratizing content publication with coral. In
NSDI, volume 4, pages 18–18, 2004.

[6] J. H. Howard, M. L. Kazar, S. G. Menees, D. A.
Nichols, M. Satyanarayanan, R. N. Sidebotham, and
M. J. West. Scale and performance in a distributed file
system. ACM Transactions on Computer Systems
(TOCS), 6(1):51–81, 1988.

[7] J. Kubiatowicz, D. Bindel, Y. Chen, S. Czerwinski,
P. Eaton, D. Geels, R. Gummadi, S. Rhea,
H. Weatherspoon, W. Weimer, et al. Oceanstore: An
architecture for global-scale persistent storage. ACM
Sigplan Notices, 35(11):190–201, 2000.

[8] D. Levin, K. LaCurts, N. Spring, and
B. Bhattacharjee. Bittorrent is an auction: analyzing
and improving bittorrent’s incentives. In ACM
SIGCOMM Computer Communication Review,
volume 38, pages 243–254. ACM, 2008.

[9] A. J. Mashtizadeh, A. Bittau, Y. F. Huang, and
D. Mazieres. Replication, history, and grafting in the
ori file system. In Proceedings of the Twenty-Fourth
ACM Symposium on Operating Systems Principles,
pages 151–166. ACM, 2013.

[10] P. Maymounkov and D. Mazieres. Kademlia: A
peer-to-peer information system based on the xor
metric. In Peer-to-Peer Systems, pages 53–65.
Springer, 2002.

[11] D. Mazieres and F. Kaashoek. Self-certifying file
system. 2000.

[12] D. Mazieres and M. F. Kaashoek. Escaping the evils
of centralized control with self-certifying pathnames.
In Proceedings of the 8th ACM SIGOPS European
workshop on Support for composing distributed
applications, pages 118–125. ACM, 1998.

[13] J. Rosenberg and A. Keranen. Interactive connectivity
establishment (ice): A protocol for network address
translator (nat) traversal for offer/answer protocols.
2013.

[14] S. Shalunov, G. Hazel, J. Iyengar, and M. Kuehlewind.
Low extra delay background transport (ledbat).
draft-ietf-ledbat-congestion-04. txt, 2010.

[15] R. R. Stewart and Q. Xie. Stream control transmission
protocol (SCTP): a reference guide. Addison-Wesley
Longman Publishing Co., Inc., 2001.

[16] L. Wang and J. Kangasharju. Measuring large-scale
distributed systems: case of bittorrent mainline dht. In
Peer-to-Peer Computing (P2P), 2013 IEEE Thirteenth
International Conference on, pages 1–10. IEEE, 2013.
ß

[^1]: 例えば、Linuxはディスクイメージの配布のためにBitTorrentを利用したり、Blizzard,Inc.はビデオゲームの配布のために利用したりしています。
