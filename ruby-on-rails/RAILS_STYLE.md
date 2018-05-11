# Railsスタイルガイド
このスタイルガイドは、個人の裁量になってしまいそうな部分の判断基準を明確にするためのものである。

## API設計の基準
基本的には、RESTfulの思想に基づいてURIの設計を行う。

## コントローラ分割の基準


### 原則
#### CRUD以外のアクションは作らない
基本的には、CRUD以外のアクションを実装してはならない。
理由は、一つのコントローラが肥大化してしまうのを防ぐため。

## Rspecによるテストの書き方の基準
### modelとrequestのテストに共通する書き方

<dl>
  <dt>describe</dt>
  <dd>何についてのテストであるのかをまとめるために使う</dd>
  <dd>model  scopesやvalidatesなどの名前を付け、テストする対象をまとめます</dd>
  <dd>request  'GET /users/:user_id #show' のようにどこにリクエストを送っているのかとアクション名がわかるように記述する</dd>
  <dt>it</dt>
  <dd>期待する処理について具体的に書く</dd>
  <dd>'Gameが1つ作成される'のようにそのテストするメソッドなどがそういう挙動を示せば良いのかをわかりやすく記述する</dd>
  <dd>_ステータスコード200(OKの意味)のテストに関しては、 'ステータスコード２００を返す' で統一する_</dd>
  <dd>_scopeなどの想定する値を返せるかに関するテストは '(期待する値を分かりやすく)を返す'で統一する_</dd>
  <dd>_request内のメソッドなど、クライアント側にデータを渡せているのかのテストに関しては、'(期待する値を分かりやすく)を送る'で統一する_</dd>
  <dd>_create後にデータベースに値が保存されたかに関するテストは、'(作成されるのを期待するモデル名など)が作成される'で統一する_</dd>
  <dt>before</dt>
  <dd>そのspecのテスト全てに共通して使うテストデータに関する処理を中に記述する</dd>
  <dd>_itの中などで使う変数は　＠を付けて @user のようにする。それ以外は game のように書く_</dd>
  <dt>expect</dt>
  <dd>itのなかに記述し、expect(期待するもの).to matcher という記述の仕方で、実際の値と期待する値を比較検証する</dd>
  <dd><a href="https://relishapp.com/rspec/rspec-expectations/v/3-4/docs/built-in-matchers">matcher一覧</dd>
</dl>


### アンチパターン集
* privateメソッドをテストすること
  * privateメソッドは普通のアクション内で使われるはずなので間接的にテストを行うべきです。
* 必要のないテストデータを作成する
  * そのテストに使わないテストデータは作っても遅くなるだけなので作らないべきです。

### modelのテスト
modelのテストに関しては主に、scopeのテストおよびバリデーションのテストを行います。ただし、private内のscopeに関しては直接テストせずに、publicのscopeで間接的にテストを行います。

#### scopeのテスト

##### app/models/score.rb

```ruby

      scope :of_not_user_units, ->(user) {
        joins(:users)
          .where.not(
            users: {
              id: user.id
            }
          )
      }
```
##### spec/model/score_spec.rb

```ruby
describe "scopes" do

    describe "of_not_user_units" do
      it "userが含まれていないunitの検索" do
        expect(Score.of_not_user_units(Unit.first.users.first).pluck(:id)).to eq Unit.second.scores.pluck(:id)
      end
    end

end

```
scopeは、describeで全てまとめておくようにします。scopeそれぞれで期待する値が取れているのかどうかを確認します。この際、返ってくるハッシュの中身は同じですが、形式的なズレがあるため、 eqで確認する場合はidにそれぞれ変換させて検証する必要があります。


## request（controller）のテスト
requestのテストをすることによって、コントローラーの処理自体もHTTPリクエストに関するテストも同時にできるので、requestのテストを使用します。

### getのテストの場合
##### games_spec.rb
```ruby
    require 'rails_helper'

RSpec.describe "Games", type: :request do

  describe "GET /games" do

            .
            .
            .

    subject do
      get "/api/v1/games", headers: @headers
    end

    it "ステータスコード200を返す" do
      subject
      expect(response).to have_http_status(200)
    end

    it "現在のユーザから取れるgameを送る" do
      subject
      expect(json['games'].length).to eq 1
    end
  end
```
このようにHTTPステータスコードが200(OK)になることのテストを記述した上で、json形式で送られた値が想定するものと一致しているのかのテストも記述します。


### postのテストの場合
##### games_spec.rb
```ruby
            .
            .
            .
#paramsはletで作るようにする(必要な値のみを渡すようにする)
let(:params) do
      {
        units: {
          left: {users: [{id: 1}], count: 1},
          right: {users: [{id: 2}], count: 1},
          ids: [1, 2]
        },
        scores: [
          {unit: 0, dropped_at: 1, shot_type: 1, miss_type: 0, side: 0},
          {unit: 0, dropped_at: 2, shot_type: 2, miss_type: 0, side: 0},
          {unit: 1, dropped_at: 3, shot_type: 3, miss_type: 0, side: 1},
          {unit: 1, dropped_at: 4, shot_type: 4, miss_type: 0, side: 1},
          {unit: 1, dropped_at: 5, shot_type: 5, miss_type: 0, side: 1}
        ],
        game: {name: "トレーニングマッチ"} ,
      }
    end

    subject do
      post "/api/v1/games", params: params, as: :json, headers: @headers
    end

    it 'ステータスコード200を返す' do
      subject
      expect(response.status).to eq 200
    end

    it 'Gameが1つ作成される' do
      expect{ subject }.to change(Game, :count).by(1)
    end
    it 'Unitが2つ作成される' do
      expect{ subject }.to change(Unit, :count).by(2)
    end
    it 'Scoreが5つ作成される' do
      expect{ subject }.to change(Score, :count).by(5)
    end
```
postリクエストに関するテストでは、HTTPステータスコードが200(OK)になることのテストを記述した上で実際にレコードが作成されたのかを確認するテストを記述します。
paramsはletを用いて初期化します。


### put(patch)のテストの場合
##### users_spec.rb
```ruby
describe 'PUT #update' do

          .
          .
          .

    it 'ステータスコード200を返す' do
      subject
      expect(response.status).to eq 200
    end

    it '更新されたユーザー情報を送る' do
      subject
      expect(json['user']['name']).to eq(params[:name])
      expect(json['user']['sport_id']).to eq(params[:sport_id])
    end
  end
```

put のテストの場合は、本当に値が更新されているのかについても確認します。
userの場合はemail、パスワードの変更に関しては、認証メールを送り、認証リンクを踏んでもらわないといけないので変更確認ができないため、ここではnameとsport_idが変化しているのかを検証しています。
