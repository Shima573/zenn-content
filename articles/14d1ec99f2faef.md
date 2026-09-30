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
# user.rb
has_many :plans, dependent: :destroy

# plan.rb
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
