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
</dl>

### modelのテスト


### request（controller）のテスト

## getのテストの場合

```ruby:video_spec.rb
require 'rails_helper'

RSpec.describe "Video", type: :request do
  describe "#index" #メソッド名を書く
    describe 'GET /api/videos/:id.json' do

      #ここでのデータをまとめる
      before do
        @video = FactoryBot.create(:video)

        #TODO 共通処理として切り出す
          @headers = { 'Content-Type' => 'application/json', 'ACCEPT' => 'application/json' }
          auth_header = @user.create_new_auth_token
          @headers.merge! auth_header
      end

      #HTTPリクエストに関しては、
      subject do
        get "/api/videos/#{@video.id}.json"
      end

      it '200 OK が返ってくる' do
        subject
        expect(response).to be_success
        expect(response.status).to eq(200)
      end

      it '動画情報を取得できる' do
        subject
        json = JSON.parse(response.body)
        expect(json['title']).to eq(@video.title)
        expect(json['status']).to eq(@video.status)
      end
    end
  end
end

```
このようにHTTPステータスコードが200(OK)になることのテストを記述した上で、
json形式で送られた値が想定するものと一致しているのかのテストも記述します。
