---
title: 参考：whitepaper 日本語訳
type: docs
---

# 参考：whitepaper 日本語訳
本章は参考情報として Juan Benet によって書かれたIPFSの[whitepaper](https://ipfs.io/ipfs/QmR7GSQM93Cx5eAg6a6yRzNde1FQv7uL6X1o4k7zrJa3LX/ipfs.draft3.pdf)「IPFS-Content Addressed, Versioned, P2P File System (DRAFT 3)」の日本語訳を記します。

{{< hint info >}}
本翻訳は直訳を避け、原文の内容を損ねない範囲で（大幅に）意訳しています。そのため必要な場合は[原文](https://ipfs.io/ipfs/QmR7GSQM93Cx5eAg6a6yRzNde1FQv7uL6X1o4k7zrJa3LX/ipfs.draft3.pdf)を参照ください。また、翻訳への協力（翻訳・修正）歓迎です。協力いただける場合は[GitHub](https://github.com/a-mitani/ipfs-primer)にてIssueの発行、またはPull requestをお願いいたします。
{{< /hint >}}

## 概要
Inter Planetary File System(IPFS)はピアツーピア（P2P）型の分散型ファイルシステムであり、全てのデバイスを一つのファイルシステムに結合することを目的としたものです。いくつもの点でIPFSはWebと同様のもの言えますが、一方で、一つのGitリポジトリ上でオブジェクト交換を行うBitTorrentのスウォームと考えることもできます。またはコンテンツアドレス型のハイパーリンク機能をもつストレージとも考えられます。IPFSはMerkle DAGのデータ構造を持ちます。IPFS上にバージョン管理可能なファイルシステムやブロックチェーン、または永続的なWebを構築することが可能になります。IPFSは分散ハッシュテーブル技術、インセンティブが与えられたブロック交換技術、自己証明型名前空間といった要素技術を組み合わせて実現されており、単一障害点がなくピア同士がお互いの信頼を必要とせずに実現できるシステムです。

## 1. 序論
これまで多くのグローバルな分散ファイルシステムを構築する試みがなされてきました。いくつかは顕著な成果を見せましたが、その他は完全に失敗に終わりました。例えば学術的な取り組みとしては、AFS[6]が成功事例として挙げられ現在も活用されている一方で、その他[7,?]の試みは失敗に終わっています。学術的な試み以外でのP2P型の分散ファイルシステムは大容量コンテンツ（音楽や動画など）の共有用途に発展してきました。特にNapsterやKaZaA、BitTorrent[2]といったアプリケーションは1億人以上のユーザーによって支えられる巨大な分散システムでした。今日でもBitTorrentは1,000万ものノードが動作している巨大なファイルシステムとして動作し続けています[16]。これらのアプリケーションは、学術的な取り組みの中でのファイルシステムよりも多くのユーザーを惹きつけてきました。しかしこれらは別目的で利用される事例[^1]はあるものの、あくまでアプリケーションであり、その上でシステムやアプリケーションが動作するインフラ基盤としてデザインされているものではありません。その点で、一般的なファイルシステムとして利用可能な、グローバル規模で低レイテンシー分散ファイルシステムというものはこれまで現れてきていないといえます。

この理由は既に十分良いシステムが存在していたからと考えられます。「HTTP」です。HTTPはこれまで最も成功した分散ファイルシステムプロトコルです。ブラウザと結びつくことでHTTPは技術的にも社会的にも多くのインパクトを与え、インターネット上でのファイル転送のデファクトスタンダードとなっています。しかしHTTPはそれが世に出て以降の15年の間に生み出されてきた新たな技術を取り込むことには失敗しています。多くの後方互換性の確保やステークホルダーの複雑なしがらみから、Webのインフラ基盤を進化させることは事実上不可能に見える一方で、実際にはHTTPの出現以降も様々な新しいプロトコルが生み出され広範囲で利用されている事実もあります。ユーザーエクスペリエンスを損なうことなく新たな機能を導入するようなHTTP-Webのグレードアップを行なっていく必要があります。

HTTPプロトコルは小容量のデータのやりとりに関して比較的安価で行うことを実現する一方で我々はデータ転送に関して新たな時代に突入しつつあります。それは(a)ペタバイト級データのホストと配信 (b)組織横断の大量データ処理 (c)大容量高精細なオンデマンドのコンテンツストリーミング(d)大規模データのバージョン管理およびリンク処理(e)重要データの消失防止、というようなものが要求される時代です。このような状況のもと、我々はHTTPを捨てて新たなデータ転送プロトコルを開発しました。次のステップはこれをWebそのものにすることが必要です。

データ転送と同等に、バージョン管理技術はデータ操作の協調作業に重要な役割を果たしてきました。ソースコード分散バージョン管理システムであるGitは分散されたデータ操作の実装モデルとして非常に重要であり、Git関連技術は従来の大容量ファイルシステムにはなかったバージョン管理機能の実現を可能にします。Camlistore[?]やDat[?]といったGitにインスパイアされたシステムも出現しています。またGitは「コンテンツアドレスMerkle DAG」形式のデータモデルを採用しており、この方式は既に分散ファイルシステムに影響を与えています。

本論文ではバージョン管理機能を備えた新しいファイルシステムであるIPFSについて解説します。IPFSは「全てのデータを統一されたMerkle DAGにモデリングする」という基本原則のもとに設計されています。

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

1. BitTorrentでのデータ交換プロトコルは「擬似しっぺ返し(tit-for-tat)戦略」と呼ばれる、貢献するノードにはより多くの褒賞を与えて、単に他者のリソースを取得するだけのノードには罰則を与えるという仕組みを採用しています。
2. BitTorrentではファイル断片の希少性をトラッキングし、最も希少なファイル断片から優先してダウンロードするように設計されています。これによりシーダー（ファイルの完全なデータを持つピア）の負荷を下げ、シーダーでないピアもデータ交換に参加出来るようにしています。
3. BitTorrent標準のしっぺ返し戦略は、略奪的帯域幅共有戦略（exploitative bandwidth sharing strategies）に対して脆弱性を持ちます。PropShare[8]戦略はこの種の脆弱性を克服し、かつスウォームのパフォーマンスを改善する戦略として知られます。

### 2.3 バージョン管理システム - Git
バージョン管理システムはファイルシステムに対する変更履歴を管理し、かつその各バージョンを効率的に配布することを容易にしてくれます。バージョン管理システムとして普及しているGitは、ファイルシステム内の変更履歴をMerkle DAGのオブジェクトにモデル化することで分散システムに親和性の高いシステムになっています。

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
5. **Objects** - コンテンツアドレス型の不可変（イミュータブル）でリンクをもつMerkle DAGです。ファイルシステムの階層やコミュニケーションシステムなど、任意のデータ構造を表現することが可能です。3.5節に詳述します。
6. **Files** - Gitに影響を受けたバージョン管理ファイルシステムです。3.6節に詳述します。
7. **Naming** - 自己証明可能かつ変更可能な名前空間システムです。3.7節に詳述します。
これらのサブシステムはそれぞれ独立している訳ではなく、互いに影響し合っています。しかし分かりやすさのため、これらを切り離されたプロトコルスタックとして記述します。

注：下記にデータ構造や機能はGo言語の文法に従い記述します。

### 3.1 Identities
各ノードは`NodeId`によって識別され、これはS/Kademliaで交換される公開鍵を暗号学的ハッシュ関数によりハッシュ化したものとなります[1]。各ノードは自身の公開鍵と（パスワードにより暗号化された）秘密鍵を保持します。ノードの管理者はノードを起動する度に新しいIDを発行することも可能ですが未払いのネットワーク報酬を放棄することとなるため、それが同じIDを使い続けるインセンティブとなります。

```
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


```
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

#### 3.4.2 BitSwap戦略
BitSwapのピアは多種多様なブロックの交換戦略を取りうり、しかもどの戦略をとるかはネットワーク全体のブロック交換効率を左右します。BitTorrentでは標準の戦略(しっぺ返し戦略)が指定されているものの、様々な非正規な戦略も実装されており、例えば BitTyrant[8]（最も少ない物をシェアする）、BitThief[8]（脆弱性を突き何もシェアしない）、PropShare [8]（比例的にシェアする）というような戦略があります。このような様々な戦略は、それが良い戦略か悪意ある戦略かに関わらずBitSwapのピアでも実装されるでしょう。そこでBitSwapプロトコルでのブロック交換戦略は下記の目的を満たしておく必要があります。

1. 自身のノードの交換効率、そしてネットワーク全体の交換効率を最大化すること。
2. 悪意あるノードにより脆弱性を突かれることや、ネットワーク全体の交換効率低下を防ぐこと。
3. 未知の交換戦略に対して効率的かつ耐性を持つこと。
4. 信用あるノードには寛容であること。

交換戦略についての探索は今後の仕事としてここでは多くは触れません。一つの実用的な戦略関数は負債比率によってスケールされたシグモイド関数でしょう。ここで、あるノードに関してそのピアとの負債比率 *r* を、

{{< katex display>}}
r = \frac{{\rm bytes＿sent}}{{\rm bytes＿recv + 1}}
{{< /katex >}}

とした時、そのノードがピアにブロックを送信する確率を

{{< katex display>}}
P(send|r) = 1 - \frac{1}{1 + exp(6 − 3r)}
{{< /katex >}}

とします。
Figure 1 に示されるようにこの関数はそのノードの負債比率が２を超えるあたりから急速に減少するものとなっています。 (※訳者注：レンダリング不具合からか[原文](https://ipfs.io/ipfs/QmR7GSQM93Cx5eAg6a6yRzNde1FQv7uL6X1o4k7zrJa3LX/ipfs.draft3.pdf)でもここに表示しているように関数が描画されていない空のグラフになっています。訳者が[当該関数を描画したものをGithub上に置いた](https://github.com/a-mitani/data-for-blog/blob/master/broccoli/ipython_notebooks/ipfs-whitepaper-fig1.ipynb)ので興味があれば参照ください。)

![Fig.1](images/fig_1.png)

負債比率は信用の一つの指標として働きます。つまり多くのデータを交換してきた実績のあるノードに対しては寛容に、一方で未知で信用ができないノードについては厳しい送信確率になるというものです。これにより（a）多くの新しいノードを作るようなシビルアタックを狙う悪意ある攻撃者に対して耐性を持つことを可能にする（b）一時的にブロックを提供できない状態になった場合にも過去に十分交換実績があるノードは保護する（c）修復されないノードに対して最終的に交換を停止するという動作が可能になります。

#### 3.4.3 BitSwap Ledger
BitSwapのノードは他のノードとのブロック転送収支の台帳を持ちます。BitSwapノード間で接続を開始する際はこの台帳を互いに交換します。この台帳に齟齬がある場合はこの台帳の信用と負債の記録を消し初期化することも可能です。悪意あるノードが負債の歴史を隠すために消すことがありえます。ノードが十分に負債を蓄積することはほぼないでしょう。（訳者注：意味不明）接続先のノードはブロックの取引を中断することが可能です。

```
type Ledger struct {
  owner NodeId
  partner NodeId
  bytes_sent int
  bytes_recv int
  timestamp Timestamp
}
```

ノードは台帳の履歴を残すこともできます。ただしブロック取引を行うのには現時点での台帳の値のみが必要で履歴は必須ではありません。ノードは古い台帳（すでに存在しないノードとの取引実績）などを自由に捨てる（Garbage Collect）ことも可能です。

#### 3.4.4 BetSwapの仕様
BitSwapのノードはシンプルなプロトコルに従います。

```
// Additional state kept
type BitSwap struct {
  ledgers map[NodeId]Ledger
  // 既知のノードに関する台帳リスト（未接続のノードも含む）

  active map[NodeId]Peer
  // 現在接続しているノードリスト

  need_list []Multihash
  // 自分が必要としているブロックのチェックサムのリスト

  have_list []Multihash
  //　自分が持っているブロックのチェックサムのリスト
}

type Peer struct {
  nodeid NodeId
  ledger Ledger
  // このピアとの取引台帳

  last_seen Timestamp
  // 最後にメッセージを受信したタイムスタンプ

  want_list []Multihash
  // このピア（またはこのピアのピア）が必要としているブロックのチェックサムのリスト
}

//プロトコルのインターフェース:
interface Peer {
  open (nodeid :NodeId, ledger :Ledger);
  send_want_list (want_list :WantList);
  send_block (block :Block) -> (complete :Bool);
  close (final :Bool);
}
```

ピアとの接続を行う際の流れは以下のようなものになります。

1. Open: ピアが`ledger`を同意するまで送信。
2. Sending: ピア同士で`want_list`と`blocks`を交換します。
3. Close: ピアが接続を閉じます。
4. Ignored: （特別）ノードがブロック送信をしない場合はそのノードは一定期間無視されます。

##### ◆ Peer.open(NodeId, Ledger)
他ノードと接続を行おうとする際、まずノードはその相手のノードとのブロック交換履歴を保存した（または新しい）ブロック交換履歴台帳である`Ledger`を送信し、 その後`Open`メッセージを相手側に送信します。

`Open`メッセージを受け取ったノードはその接続を受け入れるか拒否するかを判断します。例えば受信側で保存している`Leadger`の記録からそのノードが信用できない場合（例えばブロック交換の負債が大きいなど）に接続を拒否します。この接続拒否については`ignore_cooldown`のタイムアウト設定を考慮しつつ確率的に行われるべきものです。

一方で接続を開始する場合、受信側は`Peer`オブジェクトの`Ledger`を受信側のローカルに保持しているオブジェクトに設定し`last_seen`タイムスタンプを更新します。そして受け取った`Leadger`と内容を比較して、それらが正確に一致している場合はそのまま接続を開始します。またもし一致しない場合は新たにゼロに初期化された台帳を生成し相手に送信します。

##### ◆ Peer.send_want_list(WantList)
ノード間の接続が開いている間、ノードは自身の`want_list`を接続している全ノードに周知します。この周知は（a）接続が開始された際、（b）ランダムに設定されたタイムアウト時間を超えた場合、（c）`want_list`の内容が変化した場合、（d）新しいブロックを受信した場合、の４つケースをトリガに行われます。

`want_list`を受信した側は、そこに記述されたブロックを自身が保有しているかをチェックし、もし保有している場合は上述のBitSwap戦略に沿ってブロックを送信します。

##### ◆ Peer.send_block(Block)
ブロックの送信方法は比較的わかりやすいものとなっています。ノードは単純にデータブロックを送信します。受信側はブロックを全て受け取った後、それが望んだものと一致するかをマルチハッシュのチェックサムを計算して確認し、確認結果を返信します。

ブロック交換が完了する際に、受け手側は受け取ったブロックについての記述を`need_list`（訳者注：`want_list`の間違いと思われます）から`have_list`に移動させます。また送受信側の両ノードで、今回行ったブロック交換について台帳に記録し更新します。

仮にチェックサムが一致しない場合は、送信側のノードが正常に動作していないか悪意あるノードであるかのどちらかになります。受け手側が今後もこのノードと取引を続けるかは受信ノード自身の判断に任されます。BitSwapは信頼性のあるネットワークプロトコルで動作していることを想定して動作しているため、仮にネットワークで問題に起因してブロック送受信に異常がある場合はそのネットワーク上で既に異常検知されているはずです。


##### ◆ Peer.close(Bool)
`final`パラメーターは`close`による接続遮断が送信側の意図か否かを示すものです。もしこのパラメータが`false`の場合は受信側で接続を再開することが可能です。これは時期尚早な接続遮断を防ぐためです。

接続遮断は以下の２つのケースで行われます。

* `silence_wait`タイムアウト時間（デフォルト30秒）を超えて接続先のノードから何もメッセージが送られてこない場合にノードは`Peer.close(false)`を発行します。
* ノードがIPFSネットワークから離脱する場合、BitSwapの取引をやめる場合、そのノードは`Peer.close(true)`メッセージを発行します。

`close`メッセージを受信後は送信側と受信側共に、全ての状態を解放し接続を遮断します。また、必要に応じて`Ledger`をローカルに保存します。

### 3.5 Merkle DAGオブジェクト
分散ハッシュテーブル(DHT)やBitSwapプロトコルにより、IPFSはP2Pのシステム上で高効率で堅牢なデータブロックの保存や交換を実現することができます。IPFSではこれらの基盤上でMerkle DAG（有向非巡回グラフ）を構築します。これはオブジェクト間を暗号学的ハッシュ値によるリンクで結びつけたものであり、Gitのデータ構造を一般化したものです。Mekle Dagを採用することにより次のようなメリットが生まれます。

1. コンテンツアドレッシング: 全てのコンテンツがそのコンテンツ自身のマルチハッシュのチェックサムをもとに同定されリンクされます。
2. 耐改ざん性: 全てのコンテンツがそのチェックサムによりリンクされているため、コンテンツの改ざんを行う場合はそのコンテンツに向けたリンクも同時に変更するすることが必要になります。その結果耐改ざん性が増加します。
3. 重複除去: 同一のコンテンツは同一オブジェクトとして扱われるため、同じコンテンツが別ブロックとして保存されることがありません。

IPFSのオブジェクトのフォーマットは下記のものです。

```
type IPFSLink struct {
  Name string
  // 本リンクの名前またはエイリアス

  Hash Multihash
  // リンク先オブジェクトの暗号学的ハッシュ値

  Size int
  // リンク先オブジェクトのサイズ
}

type IPFSObject struct {
  links []IPFSLink
  // リンクのリスト

  data []byte
  // コンテンツ領域
}
```

IPFSのMerkle DAGは非常に柔軟性の高いデータ保存方法です。(a)コンテンツアドレス形式であること、(b)上記のフォーマットに従っている、という２点のみが要求されます。IPFSではデータフィールドのフォーマットは上位のアプリケーションに完全に委ねられ、IPFS自身が理解できないカスタムフォーマットも利用可能です。IPFSオブジェクト内のリンクテーブルは下記の役割を果たします。

- オブジェクトが参照する全てのオブジェクトのリストを保持する。例えば
```
> ipfs ls /XLZ1625Jjn7SubMDgEyeaynFuR84ginqvzb
XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x 189458 less
XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5 19441 script
XLF4hwVHsVuZ78FZK6fozf8Jj9WEURMbCX4 5286 template
<object multihash> <object size> <link name>
```

- foo/bar/bazといったパスの解決を行う。オブジェクトが与えられた際、IPFSはオブジェクトのリンクを参照し、そのリンク先のオブジェクトのリンクを参照するというように辿りパス文字列をフェッチしていきます。

- 再帰的に参照オブジェクトを解決する。
```
> ipfs refs --recursive \
/XLZ1625Jjn7SubMDgEyeaynFuR84ginqvzb
XLLxhdgJcXzLbtsLRL1twCHA2NrURp4H38s
XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x
XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5
XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z
...
```

#### 3.5.1 パス

IPFSのオブジェクトは従来のUNIXシステムやWebのパスと同様にパス（PATH）を指定してオブジェクトにアクセスすることが可能です。
IPFSでの絶対パスは以下のようなフォーマットで表されます。
```
# フォーマット
/ipfs/<hash-of-object>/<name-path-to-object>

# 例
/ipfs/XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x/foo.txt
```
冒頭の`/ipfs`は既存のシステムにマウントする際のマウントポイント名であり、これにより既存のファイルシステムとの名前衝突を防ぎます（もちろんマウントポイント名は別の名前に設定することも可能です）。IPFSでは分散システムの中で多くのオブジェクトを保持するファイルシステムのアドレス一貫性を保つために、グローバルなルートディレクトリは持ちません。そのためパスの２階層目は必ずオブジェクトのハッシュ値が入ります。

例えば<foo>/bar/bazのパスにあるオブジェクトbazは以下の３とおりのパス指定方法でアクセス可能です。
```
/ipfs/<hash-of-foo>/bar/baz
/ipfs/<hash-of-bar>/baz
/ipfs/<hash-of-baz>
```

#### 3.5.2 ローカルオブジェクト

IPFSで管理されるオブジェクトの保存や取り出しを行うために、各IPFSクライアントはそれぞれローカルストレージを持つ必要があります。ほとんどの場合、このローカルストレージはクライアント上のディスクスペースの一部（ネイティブのファイルシステムに管理されたものや、leveldb［4］のようなKVSにより管理されたもの、IPFSにより直接管理されたもの含む）が利用されますが、揮発性の媒体（RAM）を利用するキャッシュ上に保持することも可能です。

IPFS上で管理されるブロックは全て、どこかのノードのローカルストレージ上に保存されています。あるユーザーがそのブロックをリクエストした場合、IPFSの管理のもとでオブジェクトが探し出され、ダウンロードされ、そのユーザーのクライアントのローカルストレージ上に（少なくとも一時的には）保存されます。これは、以後そのブロックを取り出す場合に高速に取り出されるようするためです。

（以降追記予定）

### 3.7 IPNS: Naming and Mutable state
ここまででIPFSはコンテンツアドレスでのDAGを構成することによるP2Pのブロック交換を実現しました。これにより不可変なオブジェクトの公開や取得が可能であり、またその改変も追うことが可能になります。しかし１つ足りない要素があります。それは変更可能な名前付けです。もしそれがなければコンテンツを新しく更新するとその度にIPFSのリンクを送る必要があります。そのため何らかコンテンツの更新を行っても同じパスで取得できるようにする仕組みが必要になります。
ここで、結局変更可能なデータである必要があるのに、なぜ不可変なMerkle DAGを利用したのか？それはオブジェクトを(a)そのハッシュ値で取り出すことを可能にし、(b)データの完全性を検証し、(c)他のデータにリンクし、(d)無制限にキャッシュが可能にできるからです。

つまり一言でいうと、オブジェクトは**恒久的なもの**であるということです。

不可変なコンテンツアドレスのオブジェクトであるMerkle DAGと、Merkle DAGを指すポインターとしての名前付け機構が多くの優れた分散システムの両輪になっています。例えばGitのバージョンコントロールも不可変なオブジェクトと変更可能な参照により成り立ち、Unixの後継である分散型OSのPlan9[?]は不可変なファイルシステムであるFossilと可変なファイルシステムVentiの組合せであり、LBFS[?]も可変なインデックスと不可変なチャンクにより構成されていました。

（以降追記予定）

#### 
## 4. 今後
IPFSを構成する多くのアイデアは学術界での研究とオープンソースのコミュニティの中で数十年にわたって育まれたものになります。独自プロトコルであるBitSwapを除き、IPFSの主な貢献はそれらの数々のアイデアを統合したことです。IPFSは非中央集権なインターネットのインフラを提供するという野心的なプロジェクトであり、そのインフラ上で今後様々なアプリケーションが動作することが可能です。控えめに述べてもこのIPFSはグローバルに広がるバージョン管理可能なファイルシステムや名前空間として、そして次世代ファイル共有システムとして利用可能です。さらにはIPFSはWebの新たな地平を広げるものになるでしょう。それは、価値ある情報を提供者自身がホストすることなくそれを必要とする人自身がホストする、そして情報を受け取る側はホストの信用に関わらず改変や逸失の心配なく必要な情報が入手できるようになります。そのような永続的Webの世界に、IPFSは我々を導いてくれるでしょう。

## 5. 謝辞
IPFSは多くの既存の素晴らしいアイデアやシステムの複合体であり、それら無くしてIPFSという壮大な目標を打ち立てることは出来なかったでしょう。また、David Dalrymple、Joe Zimmerman、そして Ali Yahyaに感謝します。 彼らはそれぞれ、一般マークルDAG（David, Joe）、ハッシュブロック(David), s/Kademliaでのシビル攻撃への防御（David, Ali）のアイデアについて長いディスカッションに付き合ってくれました。また、David Mazieresの素晴らしいアイデアに特別に感謝します。

## 6. Reference TODO
（訳者注：本節は原文でも空白になっています）

## 7. Reference

［1］ I. Baumgart and S. Mies. S/kademlia: A practicable approach towards secure key-based routing. In Parallel and Distributed Systems, 2007 International Conference on, volume 2, pages 1–8. IEEE, 2007.

[2] I. BitTorrent. Bittorrent and A¸ttorrent software surpass 150 million user milestone, Jan. 2012.

[3] B. Cohen. Incentives build robustness in bittorrent. In Workshop on Economics of Peer-to-Peer systems, volume 6, pages 68–72, 2003.

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
