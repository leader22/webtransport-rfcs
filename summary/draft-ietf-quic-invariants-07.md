> Read [original](https://tools.ietf.org/html/draft-ietf-quic-invariants-07) / [markdown](../markdown/draft-ietf-quic-invariants-07.md)

---

# Version-Independent Properties of QUIC

## Abstract

- この文書では、QUICプロトコルにおける不変の要素について定義する
- 新しいバージョンになっても変わらない原則

## 1. Introduction

- セキュアで多重化できることに加えて、QUICはバージョンを切り替えできる
- この機能を備えることで、新たな要件を満たすことができるはず

## 2. Conventions and Definitions

- いつもの

## 3. An Extremely Abstract Description of QUIC

- QUICについておさらい
- 2つのエンドポイントを結ぶコネクションこそがQUIC
- エンドポイントはUDPでやり取りする
  - UDP上でQUICのパケットを送受信する
- QUICパケットでコネクションを確立するし、状態もシェアする

## 4. QUIC Packet Headers

- QUICパケットには2種類のヘッダが定義されてる
  - LongとShort
- 先頭の1バイトの1ビット目（MSB）で見分けられる
  - Long: 1 | Short: 0
- その他のペイロードは、バージョンによって長さも内容も異なる

### 4.1. Long Header

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+
|1|X X X X X X X|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Version (32)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| DCID Len (8)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Destination Connection ID (0..2040)           ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| SCID Len (8)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Source Connection ID (0..2040)              ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|X X X X X X X X X X X X X X X X X X X X X X X X X X X X X X  ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 先頭1バイトの1ビット目が`1`
  - その他のビットはバージョンによって用途が異なる
- そのあとの4バイトは、32bitで`Version`を表す
- そのあと1バイトは、8bitで`DCID` = `Destination Connection ID`の長さ
- そのあとは、長さに従って、0-255バイトのDCID本体が続く
- そのあと1バイトは、8bitの`SCID` = `Source Connection ID`の長さ
- そのあとは、長さに従って、0-255バイトのSCID本体が続く

### 4.2. Short Header

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+
|0|X X X X X X X|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Destination Connection ID (*)               ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|X X X X X X X X X X X X X X X X X X X X X X X X X X X X X X  ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 先頭1バイトの1ビット目が`0`
- そのあとは`DCID`本体が続く
  - それ以外のフィールドは存在しない
  - 長さはパケットに含まれないし、この文書でも定義しない

### 4.3. Connection ID

- 任意の長さを持つフィールド
- 決まったエンドポイントに対してパケットを配送するためのID
  - 下層のプロトコルでアドレスが変わったとしても届けられるように
  - エンドポイントとしてもこのIDで、QUICコネクションを識別する
- IDは、バージョンごとに決められた方法で生成される

### 4.4. Version

- QUICのバージョンは、32bitの整数で、ネットワークバイトオーダー
  - `0`は、後述するバージョンネゴシエーションのために割当られてる
- この文書の内容は、いかなるバージョンのQUICになっても適当される

## 5. Version Negotiation

- Longヘッダから成るパケットを受け取ったエンドポイントは、バージョンを確認する
- 解釈できない・サポートしないバージョンだった場合は、バージョンネゴシエーションを行う
  - そのためのパケットをレスポンスする
- Shortヘッダから成るパケットだった場合は、ネゴシエーションを行わない

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+
|1|X X X X X X X|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Version (32) = 0                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| DCID Len (8)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Destination Connection ID (0..2040)           ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| SCID Len (8)  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                 Source Connection ID (0..2040)              ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Supported Version 1 (32)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   [Supported Version 2 (32)]                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                              ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   [Supported Version N (32)]                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- バージョンネゴシエーションパケットは、`Version`フィールドが`0`かどうかで見分けられる
- Longヘッダの後には、サポートしているバージョンが連なる
- これらの情報のないバージョンネゴシエーションパケットは無視しなければならない
- バージョンネゴシエーションパケットは、Integrityなどで保護されない
  - 特定のバージョンでは認証フローが入るかもしれないが
- 受け取った`SCID`を、`DCID`フィールドに入れてレスポンスする必要がある
- バージョンネゴシエーションパケットを受け取ったエンドポイントは、使用するバージョンを変更する
  - バージョン変更の条件は、QUICのバージョンによる

## 6. Security and Privacy Considerations

- すべてのQUICパケットがバージョン番号を含むわけではない
  - ミドルボックスで何かしたい場合は、最初からトラッキングする必要がある
- Integrityなどの仕組みは、各バージョンのQUICが定義する必要がある

## 7. IANA Considerations

- IANAへの要求はない

## 8. References

### 8.1. Normative References

### 8.2. Informative References

### 8.3. URIs

## Appendix A. Incorrect Assumptions

## Author's Address