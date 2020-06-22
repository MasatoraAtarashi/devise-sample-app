deviseを利用してRailsアプリケーションに認証機能を作る

# はじめに
deviseはRailsアプリケーションに簡単に認証機能を追加することができるgemです。deviseはwardenというRackミドルウェアのラッパーで、実際の認証はwardenが行っています(そのため、実装の詳細を知りたい場合はwardenの実装をまず見る必要があるみたいです)。
Railsチュートリアルでは認証機能をスクラッチで作成していますが、実際のRailsアプリケーションではこのdeviseを利用して認証機能を作成することがデファクトスタンダードとなっているようです。

今回は自分の勉強のため、deviseを利用してRailsアプリケーションに認証機能を追加する方法を整理したいと思います。なお内容は[こちらの記事](https://qiita.com/cigalecigales/items/f4274088f20832252374)をほぼ踏襲しています。謝謝

なお以下の実装はすべてこちらのリポジトリにあります。

# 環境
- Ruby on Rails 5.2.4.3
- Ruby 2.6.4

# deviseのインストール
## 1.Gemfileを編集
```Gemfile
```

## 2.bundle install
```sh
$bundle install
```

# deviseの設定
## 1.以下のコマンドを実行
```sh
$ rails g devise:install
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Depending on your application's configuration some manual setup may be required:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

     * Required for all applications. *

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"
     
     * Not required for API-only Applications *

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

     * Not required for API-only Applications *

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views
       
     * Not required *

===============================================================================
```

## デフォルトURLを設定
```config/development.rb
```

## rootページを作成
ここはスキップして、任意のページをrootページに設定しても大丈夫です。
```sh
$rails g controller StaticPages index
```

```route.rb
Rails.application.routes.draw do
  root 'static_pages#index'
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
end
```

## flashメッセージを設定

```app/views/layouts/application.html.erb
<!DOCTYPE html>
<html> 
 <head>
  <title>DeviseRails5</title>
  <%= csrf_meta_tags %>

  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
 </head>
 <body>
  <p class="notice"><%= notice %></p>
  <p class="alert"><%= alert %></p>

  <%= yield %>

 </body> 
</html>
```

## deviseのviewを作成
```sh
$ rails g devise:views
invoke  Devise::Generators::SharedViewsGenerator
create    app/views/devise/shared
create    app/views/devise/shared/_error_messages.html.erb
create    app/views/devise/shared/_links.html.erb
invoke  form_for
create    app/views/devise/confirmations
create    app/views/devise/confirmations/new.html.erb
create    app/views/devise/passwords
create    app/views/devise/passwords/edit.html.erb
create    app/views/devise/passwords/new.html.erb
create    app/views/devise/registrations
create    app/views/devise/registrations/edit.html.erb
create    app/views/devise/registrations/new.html.erb
create    app/views/devise/sessions
create    app/views/devise/sessions/new.html.erb
create    app/views/devise/unlocks
create    app/views/devise/unlocks/new.html.erb
invoke  erb
create    app/views/devise/mailer
create    app/views/devise/mailer/confirmation_instructions.html.erb
create    app/views/devise/mailer/email_changed.html.erb
create    app/views/devise/mailer/password_change.html.erb
create    app/views/devise/mailer/reset_password_instructions.html.erb
create    app/views/devise/mailer/unlock_instructions.html.erb
```

# Userモデルを作成
```sh
rails g devise User
```

上記のコマンドで以下のマイグレーションファイルが生成されます。

```db/migrate/20200622180124_devise_create_users.rb
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[5.2]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.inet     :current_sign_in_ip
      # t.inet     :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

デフォルトではdatabase_authenticatable、registerable、recoverable、rememberable、 validatableがオンになっているようです。

```app/models/user.rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```

今回はデフォルトのまま作成しますが、オプションで10個のモジュールから好きなものを有効化できます。
* [Database Authenticatable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/DatabaseAuthenticatable): サインイン中にユーザーの信頼性を検証するために、パスワードをハッシュしてデータベースに保存します。認証は、POST要求またはHTTP基本認証の両方で実行できます。
* [Omniauthable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Omniauthable): OmniAuth（https://github.com/omniauth/omniauth） サポートを追加します。
* [Confirmable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Confirmable): 確認手順が記載された電子メールを送信し、サインイン時にアカウントが既に確認されているかどうかを確認します。
* [Recoverable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Recoverable): ユーザーのパスワードをリセットし、リセットの指示を送信します。
* [Registerable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Registerable): 登録プロセスを通じてユーザーのサインアップを処理し、ユーザーが自分のアカウントを編集および破棄できるようにします.
* [Rememberable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Rememberable):   保存されたCookieからユーザーを記憶するためのトークンの生成とクリアを管理します.
* [Trackable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Trackable): サインイン数、タイムスタンプ、IPアドレスを追跡します。.
* [Timeoutable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Timeoutable): 指定した期間アクティブでなかったセッションを期限切れにします。
* [Validatable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Validatable): 電子メールとパスワードの検証を提供します。 これはオプションであり、カスタマイズできるため、独自の検証を定義できます。
* [Lockable](http://www.rubydoc.info/github/heartcombo/devise/master/Devise/Models/Lockable): 指定された回数のサインイン試行の失敗後にアカウントをロックします。 メールまたは指定した期間の後にロックを解除できます

マイグレーションを実行します
```sh
$ rake db:migrate
```

# Viewを編集
```app/views/layouts/application.html.erb
<!DOCTYPE html>
<html> 
 <head>
  <title>DeviseSampleApp</title>
  <%= csrf_meta_tags %>

  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
 </head>
 <body>
 <header>
    <nav>
        <% if user_signed_in? %>
        <%= link_to 'プロフィール変更', edit_user_registration_path %>
        <%= link_to 'ログアウト', destroy_user_session_path, method: :delete %>
    <% else %>
        <%= link_to 'サインアップ', new_user_registration_path %>
        <%= link_to 'ログイン', new_user_session_path %>
        <% end %>
    </nav>
  </header>
  <p class="notice"><%= notice %></p>
  <p class="alert"><%= alert %></p>

  <%= yield %>

 </body> 
</html>
```
写真挿入
こんな感じの画面が生成されています。サインアップしてメールアドレスとパスワードを入力すると、見事にログインできました！

# その他
## deviseが提供するヘルパーメソッド
> user_signed_in?


ユーザがログインしているかどうかを確認できます。

> current_user


ログインしているユーザにアクセスできます。

> user_session


セッションにアクセスできます。

## サインアップ後のリダイレクト先を変更
```app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  def after_sign_up_path_for(resource)
    edit_user_registration_path #ここを編集して任意のページに飛ばせる
  end
end
```

## ログイン後のリダイレクト先を変更
```app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  def after_sign_in_path_for(resource)
    edit_user_registration_path #ここを編集して任意のページに飛ばせる
  end
end
```

## コールバック関数
(ここは正確にはよくわかっていませんが)
deviseが呼んでくれるコールバックはafter_fetchだけらしいので(要出典)、その他のコールバックを呼びたいときは[wardenのインターフェイス](https://github.com/wardencommunity/warden/wiki/callbacks)を直接呼び出す必要があります。


# 参考文献
- [heartcombo/devise](https://github.com/heartcombo/devise)
- [wardencommunity/warden](https://github.com/wardencommunity/warden)
- https://qiita.com/cigalecigales/items/f4274088f20832252374
- https://qiita.com/Hal_mai/items/350c400e8763ce0487a3
- https://stackoverflow.com/questions/11409828/does-devise-have-callback





a
a
a


twitter
API key: gEgDaO9HJET7q8OD92olz6NrJ
API secret Key: Mvji6x1omoPoDPlPVFOsZh2es355AMwK8wpJqsca8kQbVQUCpE
Bearer token: AAAAAAAAAAAAAAAAAAAAAJAgFQEAAAAAC0C4xb1Q5p0SlpOsvl9I7HxlX68%3DwUkm1I7SkwyVOIEUoiloHyOdwC7felL9H4m6dcjBuoxziOZRSP

・deviseでapiログイン実装
・omniauth-twitter
facebook
github

・wardenでapi作成
・apiドキュメント作成
# 参考
https://qiita.com/cigalecigales/items/f4274088f20832252374
https://qiita.com/Hal_mai/items/350c400e8763ce0487a3