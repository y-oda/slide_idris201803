### はじめてのプログラミング with Idris

Yohei Oda

---

### whoami

- Scalaエンジニア（2年半）
- JVMとかGCとか好き
- すごいH本は一通り読んだ
- Coq??😪

---

### 今日話したいこと

- Idrisと依存型の紹介
  - 普通の関数をちょっと安全にしてみる
  - プログラミングするときに意識すること
  - さわってみての個人的な感想

---

#### What's idris?

- Haskellベースの関数型言語
  - 文法はほぼHaskell
  - 実装もHaskellメインで行われている
  - 使いやすいBetterHaskell的な側面
    - 正格評価がデフォルト
    - `:` と `::` が逆
    - ちょっと便利な関数が増えてる etc

---

#### What's idris?

- 資料を書いている時点でバージョン1.1.1
  - 本(※)が出たのでバージョン1を出した
    - 今日の説明もこの本がベース
    - 興味があるかたはぜひ買ってみてください
  - プロダクションで使えるとかそういう話ではない

※ `Type Driven Development with Idris`

---

#### What's idris?

- 依存型を扱うことができる

![](https://upload.wikimedia.org/wikipedia/commons/1/19/Lambda_cube.png)

from [wikipedia](https://upload.wikimedia.org/wikipedia/commons/1/19/Lambda_cube.png)

---

#### 依存型

- 型と値が組み合わさったとても強い型
  - 型が値に依存する
- `types are first class`
  - 型を値や関数と同じように扱える
- 型を使った証明ができる
  - 証明支援ではCoqやAgdaなどが有名
  - Idrisは定理証明系より汎用プログラミング路線

---

#### 依存型の例

- 長さを型に持つリスト Vect 型
  - `Vect n a`
    - n は要素の数、 a は要素の型

---

#### Vect の型とコンストラクタ

- 型

```haskell
Vect : Nat -> Type -> Type
```

- コンストラクタ

```haskell
Nil : Vect 0 elem
    Empty vector

(::) : (x : elem) -> (xs : Vect len elem) -> Vect (S len) elem
    A non-empty vector of length S len, consisting of a head element and the rest of the list, of length len.
    infixr 7
```

- Nat は `Z` と `S a` からなる自然数を表す型

---

Vect の基本的な処理を書いてみます

---

#### List の map

```haskell
my_list_map: (a->b) -> List a -> List b
my_list_map f [] = []
my_list_map f (x :: xs) = f x :: my_list_map f xs
```

---

#### Vect の map

```haskell
my_vect_map: (a->b) -> Vect n a -> Vect n b
my_vect_map f [] = []
my_vect_map f (x :: xs) = f x :: my_vect_map f xs
```

```
> my_vect_map length ["apple", "banana"]
[5, 6] : Vect 2 Nat
```

---

#### List の append

```haskell
my_list_append : List a -> List a -> List a
my_list_append [] ys = ys
my_list_append (x :: xs) ys = x :: my_list_append xs ys
```

---

#### Vect の append

```haskell
my_vect_append : Vect n a -> Vect m a -> Vect (n + m) a
my_vect_append [] ys = ys
my_vect_append (x :: xs) ys = x :: my_vect_append xs ys
```

```
> my_vect_append [1,2,3] [6,7]
[1, 2, 3, 6, 7] : Vect 5 Integer
```

---

どんなうれしいことがあるか

---

#### 関数を作る時に面倒なこと

- 色々なパターンの入力に対処する必要がある
  1. 普通の値
  1. コーナーケース
  1. やってほしくないが型の都合上渡せてしまう値
- 引数が複数あると組み合わせが爆発する

---

#### 関数を作る時に面倒なこと

- 呼び出し側のバリデーションは信用できない
  - 結果バリデーションが二重になってたりする
- ダメな値が渡された時にどうするか？
  - エラーの返し方を考えないといけない
  - 変な値を入れてくる方が悪い😤（ときもある）

---

#### 例 リストの要素を添字で取り出す index 関数 を作る

```haskell
index: List a -> Integer -> a
```

- マイナスの数値が来た時どうする？
- リストのサイズより大きな値がきたらどうする？
  - Maybe 必要

---

#### 型の制約をきつくする

```haskell
index: List a -> Nat -> a
```

- 添字を `Nat` で渡すように変更
  - マイナスの数を渡すことはできない
- リストのサイズより大きな値がきたらどうする？
  - やっぱり Maybe 必要

---

#### 依存型を使う

```haskell
index: Vect n a -> Fin n -> a
```

- `List a` を `Vect n a` に変更
- 添字を `Fin n` (n より小さい自然数を表す型) で渡す
- 要素を必ず取り出せることを型で保証できる
  - 関数側には Maybe 不要

---

#### 他の例

##### Vect の tail

```haskell
my_vect_tail: Vect (S n) a -> Vect n a
my_vect_tail (x :: xs) = xs
```

- `Vect (S n) a` を要求する
  - `Vect 0 a` では型が合わない
    - 空リストではこの関数は呼べない
  - 処理が必ず成功することを保証できる

---

#### Vect の zip

```haskell
my_vect_zip : Vect n a -> Vect n b -> Vect n (a, b)
my_vect_zip [] [] = []
my_vect_zip (x :: xs) (y :: ys) = (x, y) :: my_vect_zip xs ys
```

- 同じ長さの2つのVectにしか使うことができない
- 複数の引数をとる場合でもシグネチャだけで関係性を表現できる

---

#### 関数に依存型を使ってうれしいこと

- 関数が期待する値を型として表現できる
  - 複数の値の組み合わせも型の制約対象にできる
  - 期待する値での振る舞いに注力できる
- バリデーションを呼び出し側に強制できる
  - バリデーション漏れがない

---

#### 関数に依存型を使ってうれしいこと

- 呼び出し側から見ると…
  - 関数が求める値をシグネチャだけで判断できる
    - ドキュメントもすっきり(のはず)
  - 必要なバリデーションが明確
    - 型をあわせられればよい

---

いい感じ😊

---

#### ここで問題

1. `zip: Vect n a -> Vect n b -> Vect n (a, b)` がある
1. `Vect n String` と `Vect m Int` がある
1. `n==m` は `True` である
1. `zip` を呼ぶことはできる？

---

できない😭

---

#### 型チェッカーの観点

- `==` の挙動は型として定義されてない
  - 「`n==m` だから `n と m は同じ` 」と判定できない
- 型チェッカーにもわかるように書かないとダメ
  - とりあえず自分で作ってみる

---

#### Natの値が同じであることを示す型

```haskell
data EqNat : (num1 : Nat) -> (num2 : Nat) -> Type where
     Same : (num : Nat) -> EqNat num num
```

- コンストラクタから以下のことが判断可能
  - `EqNat a b` 型の値が存在していれば、a と b は同じ値である

---

#### 2つの同じ値に1ずつ足してみる

```haskell
sameS : (k: Nat) -> (j: Nat) -> (eq : EqNat k j) -> EqNat (S k) (S j)
sameS j j (Same j) = Same(S j)
```

- `EqNat k j` の値があれば `EqNat (S k) (S j)` の値も存在する

---

#### 2つの値からEqNatを作る

```haskell
checkEqNat : (num1 : Nat) -> (num2 : Nat) -> Maybe (EqNat num1 num2)
checkEqNat Z Z = Just (Same 0)
checkEqNat Z (S k) = Nothing
checkEqNat (S k) Z = Nothing
checkEqNat (S k) (S j) = case checkEqNat k j of
                              Nothing => Nothing
                              (Just eq) => Just (sameS _ _ eq)
```

- 2つの値が…
  - Z と Z であれば `EqNat 0 0` が作れる
  - S k と Z は違う値
  - S k と S j と書けるなら k と j について再帰する
    - `sameS` より`EqNat k j` から `EqNat (S k) (S j)` が導ける

---

#### 型を変換する関数

```haskell
exactLength : (len : Nat) -> (input : Vect m a) -> Maybe (Vect len a)
exactLength {m} len input = case checkEqNat m len of
                                 Nothing => Nothing
                                 Just (Same len) => Just input

```

- `len と m が等しい` 場合は `Vect m a` を `Vect len a` に変換できる
  - `EqNat m len` の存在から型チェックで m と len が同じだと判定している

---

#### zipとあわせてみる

```haskell
-- checkEqNatなどは省略
exactLength : (len : Nat) -> (input : Vect m a) -> Maybe (Vect len a)
exactLength {m} len input = case checkEqNat m len of
                                 Nothing => Nothing
                                 Just (Same len) => Just input

my_zip: Vect n a -> Vect n b -> Vect n (a, b)
my_zip [] [] = []
my_zip (x :: z) (y :: w) = (x, y) :: my_zip z w

maybe_zip: Vect n a -> Vect m b -> Maybe(Vect n (a,b))
maybe_zip {n} x y = case exactLength n y of
                      Nothing => Nothing
                      Just z => Just (my_zip x z)
```

---

#### checkEqNat をもうちょっと一般化

```haskell
checkEqNat' : (num1 : Nat) -> (num2 : Nat) -> Maybe (num1 = num2)
checkEqNat' Z Z = Just Refl
checkEqNat' Z (S k) = Nothing
checkEqNat' (S k) Z = Nothing
checkEqNat' (S k) (S j) = case checkEqNat' k j of
                             Nothing => Nothing
                             (Just proof) => Just (cong proof)
```

---

#### auto-implicit argument

- 同じようなテクニックで依存型を引数に取らない関数も安全にしている

```haskell
Prelude.List.tail : (l : List a) -> {auto ok : NonEmpty l} -> List a
```

- `NonEmpty l` の値の存在をコンパイル時にチェック
  - `NonEmpty` のコンストラクタ

  ```haskell
  IsNonEmpty : NonEmpty (x :: xs)
  ```

  - l が空だと `NonEmpty l` 型の値は存在しないのでコンパイルに失敗する

---

#### まとめ

- Idrisは依存型を扱えるプログラミング言語
- 依存型を使うと、様々な事前条件や事後条件を関数の型で表せる
  - バリデーションが行われることの保証もできる
- 依存型の操作では型チェッカーにわかるようにする必要がある
  - コンストラクタをうまく使う
  - 依存型でない型を使う関数も安全にできる

---

#### 個人的な感想
##### Idris のいいところ

- コンパイルが通ったときの信頼感がすごい
  - 事後条件のチェックが結構うれしい
- 安全かつ柔軟
  - 安全であれば割となんでもあり
  - たまになぜ動いているのかよくわからない😇

---

#### 個人的な感想
##### Idris のいいところ

- 依存型ならではのテクニックやツール
  - 強力な実装のサーチ
  - 返り値の型をパラメータ的に使ったりできる
  - `totality checking`
- 基本的な文法の学習が比較的容易
  - Haskellが読めればだいたい読める

---

#### 個人的な感想
##### Idris のつらいところ

- わかりづらいエラー
  - 言っていることが難しくてよくわからない 😇
    - 厳密さがつらい
  - 型がわからなくなり遭難
  - 色々試した結果インデントのずれだったり…
    - 自動でできることは自動でやるのがよさそう
- 依存型のノウハウ（特に証明）がない
  - Agdaの資料とかが参考になりそうな気がする

---

#### 個人的な感想
##### Idris のつらいところ

- 遅い
  - コンパイルも実行も遅い
- CPUもぶんぶん回る
- 普通のことが普通にできないときがある
  - 型の証明がちゃんと動いていないときがあるような…?
  - `deriving` がなくて面倒
- コンパイラ&ランタイムの改善は頑張ってほしい…

---

#### 参考文献
##### 公式

- [Type Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris)
  - （実質的な）公式ガイドブック
  - 今回の資料はこの本ベースで作っています

- [Idris Tutorial](http://docs.idris-lang.org/en/latest/tutorial/index.html)
  - 公式チュートリアル

---

#### 参考文献
##### 日本語

- [IdrisでWebアプリを書く](https://www.slideshare.net/tanakh/idrisweb)
  - 3年前の資料ですが面白いので最初に読むのにお勧め
  - この時の問題は現在もあまり解決されていない気が…

- [プログラミング言語 idris - wkwkesのやつ](http://wkwkes.hatenablog.com/entry/2016/12/17/000000)
  - tactic を使った証明の様子が紹介されています

---

#### 参考文献
##### Haskellers 向け

- [Idris for Haskellers](https://github.com/idris-lang/Idris-dev/wiki/Idris-for-Haskellers)
  - Haskeller 向け公式ドキュメント

- [10 things Idris improved over Haskell](https://deque.blog/2017/06/14/10-things-idris-improved-over-haskell/)
  - Better Haskell としての Idris を知りたい人にお勧め
