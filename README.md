# OmniAuth2 Kakao

This is the OmniAuth strategy for authenticating to [Kakao](http://www.kakao.com/). To
use it, you'll need to sign up for an REST API Key on the [Kakao Developers Page](http://developers.kakao.com). For more information, please refer to [Create New Application](https://developers.kakao.com/docs/restapi#시작하기-앱-생성) page.

이 저장소는 본래 omniauth-kakao gem을 만드신 * [Shayne Sung-Hee Kang](https://github.com/shaynekang)님의 레포에서 클론을 통해 만들어졌습니다. 기존의 Gem 파일에서 kakao 사이트의 omniauth 방식이 변경됨에 따라 오류가 발생하는 부분을 해결하기 위해 제작되었습니다. 공부 및 테스트 용도로 만들어졌으므로 저작권이나 기타 문제가 될 소지가 있는 내용이 있다면 알려주시기 바랍니다.

[카카오](http://www.kakao.com/) 인증을 위한 OmniAuth strategy 입니다. [카카오 개발자 페이지](http://developers.kakao.com)에서 REST API 키를 생성한 뒤 이용해 주세요. 자세한 사항은 [시작하기 - 앱 생성](https://developers.kakao.com/docs/restapi#시작하기-앱-생성) 페이지를 참고하시기 바랍니다.

## Installing

Add to your `Gemfile`:

`Gemfile`에 다음의 코드를 넣어주세요.

```ruby
gem 'omniauth2-kakao'
```

Then `bundle install`.

이후 `bundle install` 을 실행해 주세요.

## Usage

다음은 간단한 예제입니다. `config/initializers/omniauth.rb`에서 미들웨어(Middleware)를 레일즈 어플리케이션에 넣어주세요.


```ruby
  # devise.rb 파일에 omniauth를 위한 환경을 설정해줍니다. ENV["KAKAO_REST_API_KEY"]는 local_env.yml에서 관리합니다.
  # 참고 . https://qiita.com/alokrawat050/items/0d7791b3915579f95791
config.omniauth :kakao, ENV["KAKAO_REST_API_KEY"]
```



그리고 [내 어플리케이션](https://developers.kakao.com/apps)에서 현재 어플리케이션을 선택하고, '설정 - 플랫폼 - 웹 - 사이트 도메인'에  도메인 주소(예: http://localhost:3000/)를 넣어주세요.

그리고 Redirect Path에는 /users/auth/kakao/callback이라고 적어주세요.

Ruby on rails를 활용하고 계신 분의 경우, 하단의 링크를 보고 하나씩 해보시면 됩니다.
https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview

```ruby
  # kakao login page link
<%= link_to "Sign in with Kakaotalk", user_kakaotalk_omniauth_authorize_path %>
```

`config/routes.rb`
```ruby

devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
```


`app/controllers/users/omniauth_callback_controller.rb`
```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def kakaotalk
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, event: :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, kind: "Facebook") if is_navigational_format?
    else
      session["devise.kakaotalk_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
```

`app/model/user.rb`
```ruby
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0, 20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
    # If you are using confirmable and the provider(s) you use validate emails, 
    # uncomment the line below to skip the confirmation emails.
    # user.skip_confirmation!
  end
end
```


## Auth Hash

Here's an example *Auth Hash* available in `request.env['omniauth.auth']`:

`request.env['omniauth.auth']` 안에 들어있는 *Auth Hash* 는 다음과 같습니다.

```ruby
{
  :provider => 'kakao',
  :uid => '123456789',
  :info => {
    :name => 'Hong Gil-Dong',
    :image => 'http://xxx.kakao.com/.../aaa.jpg',
  },
  :credentials => {
    :token => 'ABCDEF...', # OAuth 2.0 access_token, which you may wish to store.
    :refresh_token => 'OPQRST...', # OAuth 2.0 refresh_token.
    :expires_at => 1321747205, # when the access token expires (it always will)
    :expires => true # this will always be true
  },
  :extra => {
    :properties => {
      :nickname => 'Hong Gil-Dong',
      :thumbnail_image => 'http://xxx.kakao.com/.../aaa.jpg'
      :profile_image => 'http://xxx.kakao.com/.../bbb.jpg'
    }
  }
}
```

## Contributors
* [Shayne Sung-Hee Kang](https://github.com/shaynekang)
* [Hong Chulju](https://github.com/fegs)
* [leekorea(hans-hk)](https://github.com/hans-hk)
* [yesjin](https://github.com/yesjin-git)

## Contribute

1. Fork the repository.
1. Create a new branch for each feature or improvement.
1. Add tests for it. This is important!
1. Send a pull request from each feature branch.

***

1. 저장소를 Fork해 주세요.
1. 새로운 기능(또는 개선할 부분)마다 브랜치를 만들어 주세요.
1. 테스트를 작성해주세요. 이는 매우 중요합니다!
1. 브랜치를 pull request로 보내주세요.


## License

Copyright (c) 2019 [yesjin](http://blog.naver.com/yesjin_nav).

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
