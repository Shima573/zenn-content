---
title: "Railsの関連付け（Association）を自作アプリのコードから改めて理解する"
emoji: "🔗"
type: "tech"
topics: [rails, ruby, activerecord, association]
published: false
---

## Association（関連付け）とは

Association（関連付け）とは、モデル同士の関係を定義する仕組みです。

Railsで使用する主なAssociationには、以下のようなものがあります。

- `belongs_to`
  - 他のモデルに属する関係を表す
  - 基本的に外部キーを持つ側のモデルに記述する
- `has_many`
  - 1つのデータが複数のデータを持つ関係を表す
- `has_one`
  - 1つのデータが1つのデータを持つ関係を表す

今回は、自作アプリで実際に使用している`has_many`と`belongs_to`を中心に確認していきます。

### UserとPlanの関連付け

自作アプリでは、「1人のユーザーが複数の登山プランを持つ」という関係があります。

UserとPlanには、以下のようにAssociationを設定しています。

```ruby
# app/models/user.rb
has_many :plans, dependent: :destroy

# app/models/plan.rb
belongs_to :user
```

User側の、

```ruby
has_many :plans
```

は、1人のUserが複数のPlanを持つことを表しています。

一方、Plan側の、

```ruby
belongs_to :user
```

は、1つのPlanが1人のUserに属することを表しています。

つまり、UserとPlanは「1対多」の関係になっています。

![UserとPlanのAssociationの関係](/images/user-plan-association.png)

*▲ UserとPlanに設定しているAssociationとデータベース上の関係*

## Associationと外部キーの関係

UserとPlanの関連付けについて確認しましたが、ここで「PlanがどのUserに属しているのかを、どのように判断しているのか？」という疑問が出てきました。

例えば、`users`テーブルと`plans`テーブルに以下のようなデータがあるとします。

### usersテーブル

| id | name |
| --- | --- |
| 5 | User A |
| 32 | User B |

### plansテーブル

| id | user_id | title |
| --- | --- | --- |
| 1 | 32 | 登山プランA |
| 2 | 32 | 登山プランB |
| 3 | 5 | 登山プランC |

`plans`テーブルの`user_id`には、そのPlanがどのUserに属しているのかを示すUserの`id`が保存されています。

例えば、登山プランAの`user_id`は`32`です。

この`32`は`users`テーブルの`id = 32`を指しているため、登山プランAは`id = 32`のUserに属していることが分かります。

同じように登山プランBも`user_id = 32`なので、同じUserに属しています。

一方、登山プランCは`user_id = 5`なので、`id = 5`のUserに属しています。

このように、別のテーブルのデータと関連付けるために使用するカラムを**外部キー（Foreign Key）**と呼びます。

今回の場合は、`plans`テーブルの`user_id`が外部キーにあたります。

## Associationを設定すると何ができるのか

Associationを設定すると、モデル同士の関係をRailsに定義し、SQLを直接書かなくても関連するデータを扱えるようになります。

例えば、UserとPlanにAssociationを設定している場合、

```ruby
user.plans
```

のように書くことで、そのUserに紐づいているPlanを取得できます。

では、なぜ`user.plans`のような書き方で関連するデータを取得できるのでしょうか。

## `current_user.plans`はなぜ使えるのか

まず、Userモデルには以下のAssociationを設定しています。

```ruby
# app/models/user.rb
has_many :plans
```

`has_many :plans`を設定することで、Userオブジェクトから、そのUserに関連するPlanを取得するための`plans`メソッドを使えるようになります。

そのため、

```ruby
current_user.plans
```

という書き方ができます。

それぞれの役割を分けると、次のように考えられます。

```text
current_user
↓
現在ログインしているUserオブジェクト

.plans
↓
そのUserに関連付いているPlanを取得
```

`current_user`は、Deviseが提供している現在ログインしているUserを取得するためのメソッドです。

一方、`plans`は`has_many :plans`を設定したことで利用できるようになったメソッドです。

では、Railsはどのようにして「そのUserに紐づくPlan」を判断しているのでしょうか。

例えば、現在ログインしているUserの`id`が`32`だったとします。

```ruby
current_user.id
# => 32
```

この場合、Railsは`plans`テーブルから`user_id`が`32`のPlanを探します。

```text
current_user.id
↓
32

plansテーブルのuser_id
↓
user_id = 32のPlanを探す

↓
現在のUserに紐づくPlanを取得
```

つまり、`current_user.plans`では、`current_user`の`id`と`plans`テーブルの外部キーである`user_id`をもとに、現在のUserに紐づくPlanを取得しています。

Associationによって`user.plans`という形で関連データを扱えるようになり、実際にどのデータが関連しているのかは外部キー`user_id`によって判断されます。

## 実際のコードを見てみる

自作アプリでは、以下のようなコードを使用しています。

```ruby
# app/controllers/plans_controller.rb
@plan = current_user.plans.find(params[:id])
```

これを順番に見ていくと、

```text
current_user
↓
現在ログインしているUser

.plans
↓
そのUserに紐づくPlanに対象を絞る

.find(params[:id])
↓
その中から指定されたidのPlanを探す
```

と分解できます。

このように、Associationを設定することで、Userに紐づくPlanを起点としてデータを取得できます。

## Associationの裏側で行われていること

ここまで、Associationを設定することで、

```ruby
current_user.plans
```

のように、Userに紐づくPlanを取得できることを確認しました。

では、このコードを実行したとき、裏側ではどのような処理が行われているのでしょうか。

大まかな流れは次のようになります。

```text
current_user.plans
↓
Active Record
↓
SQLを生成
↓
データベースに問い合わせる
↓
plansテーブルから該当するデータを取得
```

Railsでは、Active RecordがAssociationの情報を利用してSQLを組み立て、データベースへの問い合わせを行います。

例えば、`current_user`の`id`が`32`の場合、イメージとしては次のようなSQLが実行されます。

```sql
SELECT "plans".*
FROM "plans"
WHERE "plans"."user_id" = 32;
```

これは、`plans`テーブルから`user_id`が`32`のPlanの全カラムを取得する、という意味です。

そのため、自分でSQLを直接書かなくても、

```ruby
current_user.plans
```

と書くことで、現在のUserに紐づくPlanを取得できます。

つまり、AssociationがSQLの代わりになるのではなく、Associationで定義したモデル同士の関係を利用して、Active Recordが必要なSQLを組み立てているということです。

## `dependent: :destroy`とは

Userモデルでは、以下のようにAssociationを設定しています。

```ruby
# app/models/user.rb
has_many :plans, dependent: :destroy
```

ここで設定している`dependent: :destroy`は、Userを削除したときに、そのUserに紐づいているPlanも一緒に削除するための設定です。

例えば、1人のUserに3つのPlanが紐づいている場合、

```text
User
├── Plan A
├── Plan B
└── Plan C
```

このUserを削除すると、

```text
Userを削除
↓
Userに紐づくPlanも削除
↓
Plan A、Plan B、Plan Cを削除
```

という処理が行われます。

`dependent: :destroy`を設定することで、Userが存在しなくなったにもかかわらず、そのUserに紐づくPlanだけがデータベースに残ってしまうことを防ぐことができます。

つまり、`dependent: :destroy`は、関連元のデータを削除したときに、それに紐づく関連データも削除するための設定です。

## 自作アプリの他のAssociation

ここまでUserとPlanの関係を中心にAssociationについて確認してきましたが、自作アプリでは他のモデルにもAssociationを設定しています。

### PlanとActivityRecord

```ruby
# app/models/plan.rb
has_one :activity_record

# app/models/activity_record.rb
belongs_to :plan, optional: true
```

Planから見ると、1つのPlanに対して1つのActivityRecordを関連付ける「1対1」の関係を設定しています。

また、`belongs_to :plan`には`optional: true`を設定しています。

通常、`belongs_to`で関連付けたデータは必須になりますが、`optional: true`を設定することで、Planに紐づいていないActivityRecordも保存できるようになります。

### UserとFavoriteとMountain

お気に入り機能では、UserとMountainの間にFavoriteモデルが存在しています。

```ruby
# app/models/user.rb
has_many :favorites, dependent: :destroy
has_many :favorite_mountains, through: :favorites, source: :mountain

# app/models/favorite.rb
belongs_to :user
belongs_to :mountain

# app/models/mountain.rb
has_many :favorites, dependent: :destroy
has_many :favorited_users, through: :favorites, source: :user
```

この関係は次のようになっています。

```text
User ── Favorite ── Mountain
          ↑
       中間モデル
```

Favoriteを中間のモデルとして、UserとMountainを関連付けています。

`through: :favorites`を設定することで、Favoriteを経由して関連するMountainを取得できます。

例えばUser側では、

```ruby
user.favorite_mountains
```

とすることで、そのUserがお気に入りに登録しているMountainを取得できます。

このように、自作アプリの中でもデータ同士の関係に合わせて、`has_many`、`belongs_to`、`has_one`などのAssociationを使い分けています。

## まとめ

今回は、自作アプリで使用しているコードをもとに、RailsのAssociationについて改めて整理しました。

Associationを設定することで、モデル同士の関係をRailsに定義し、関連するデータを扱いやすくできます。

今回確認した内容は以下の通りです。

- `has_many`は、1つのデータが複数のデータを持つ関係を表す
- `belongs_to`は、他のモデルに属する関係を表す
- 外部キーによって、どのデータ同士が関連しているのかを判断できる
- `has_many :plans`を設定することで、`user.plans`のように関連するデータを取得できる
- 裏側ではActive RecordがAssociationの情報を利用してSQLを組み立て、データベースへ問い合わせている
- `dependent: :destroy`を設定すると、関連元のデータを削除したときに紐づくデータも削除できる
- `has_one`や`has_many :through`など、データ同士の関係に合わせてAssociationを使い分ける

これまでは`has_many`や`belongs_to`を「モデル同士を関連付けるもの」として使っていましたが、今回改めて確認したことで、Association・外部キー・Active Record・SQLがどのようにつながっているのかを整理することができました。
