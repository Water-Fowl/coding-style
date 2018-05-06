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
  <dt>it</dt>
  <dd>期待する処理について具体的に書く</dd>
  <dt>before</dt>
  <dd>そのspecのテスト全てに共通して使うテストデータに関する処理を中に記述する</dd>
  <dt>expect</dt>
  <dd>expect(期待するもの).to matcher という記述の仕方で、実際の値と期待する値を比較検証する</dd>
</dl>

###アンチパターン集
* privateメソッドをテストすること
  * privateメソッドは普通のアクション内で使われるはずなので間接的にテストを行うべきです。
* 必要のないテストデータを作成する
  * そのテストに使わないテストデータは作っても遅くなるだけなので作らないべきです。
### modelのテスト


### request（controller）のテスト
requestのテストをすることによって、コントローラーの処理自体もHTTPリクエストに関するテストも同時にできるので、requestのテストを使用します。

## getのテストの場合

```ruby:video_spec.rb
require 'rails_helper'

RSpec.describe "Video", type: :request do
  describe "#index" #メソッド名を書く
    describe 'GET /api/videos/:id' do

      #ここでデータをまとめる
      before do
        @video = FactoryBot.create(:video)

        #TODO 共通処理として切り出す
          @headers = { 'Content-Type' => 'application/json', 'ACCEPT' => 'application/json' }
          auth_header = @user.create_new_auth_token
          @headers.merge! auth_header
      end

      #HTTPリクエストに関しては、
      subject(:index_action) do
        get "/api/videos/#{@video.id}.json", headers: @headers
      end

      it '200 OK が返ってくる' do
        index_action
        expect(response.status).to eq(200)
      end

      it '動画情報を取得できる' do
        index_action
        json = JSON.parse(response.body)
        expect(json['title']).to eq(@video.title)
        expect(json['status']).to eq(@video.status)
      end
    end
  end
end

```
#####このようにHTTPステータスコードが200(OK)になることのテストを記述した上で、json形式で送られた値が想定するものと一致しているのかのテストも記述します。


#####subjectに関しては suject だと何の処理を表しているのかわからないので、アクション名をつけてあげる
