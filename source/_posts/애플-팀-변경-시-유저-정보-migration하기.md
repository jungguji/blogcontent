---
layout: posts
title: 애플 팀 변경 시 유저 정보 migration하기
date: 2021-10-31 20:24:57
tags:
    - apple
---

## 서론

현재 운영 중인 서비스가 어른들의 사정(?)으로 기존에 앱스토어에 올라와있던 어플의 팀을 이전해야 하는 경우가 생겼는데 이 때 팀 변경 후(정확히는 변경 후, Service Ids를 새로 생성한 후) 기존 애플 유저들의 계정이 로그인 되지 않고 가입 프로세스를 타게 되었는데, 이를 해결 하기 위해 이전 된 팀으로 기존 유저 정보를 마이그레이션 하는 방법을 정리 해둔다.

* * *

## 본론

### 1. 기존 팀 Key file로 client_secret 생성

```ruby
require 'jwt'

key_file = '기존 키파일명'
team_id = '기존 팀 아이디'
client_id = '클라이언트 아이디'
key_id = '기존 키 아이디'

ecdsa_key = OpenSSL::PKey::EC.new IO.read key_file

headers = {
  'kid' => key_id
}

claims = {
    'iss' => team_id,
    'iat' => Time.now.to_i,
    'exp' => Time.now.to_i + 86400*180,
    'aud' => 'https://appleid.apple.com',
    'sub' => client_id,
}

token = JWT.encode claims, ecdsa_key, 'ES256', headers

puts token
```

1. 위 코드를 복사해서 ruby script 파일(.rb)로 만든다.
2. 키 파일과 작성한 script 파일을 같은 디렉토리 내에 위치 시킨다.
3. 터미널에서 .rb 스크립트를 실행 시킨다.

```ruby
# ruby 스크립트와 키파일이 존재하는 디렉토리로 이동
cd /Users/user/Documents/apple_client

# ruby 스크립트 실행
ruby client_secret.rb
```

4. 스크립트가 정상적으로 실행됐으면 client_secret이 생성됨

```txt
Ignoring ffi-1.15.0 because its extensions are not built. Try: gem pristine ffi --version 1.15.0
# 아래
eyJraWQiOiJHMzg2UlM3MlY3IiwiYWxnIjoiRVMyNTYifQ.eyJpc3MiXXXXXXXXXxxxTJSOVM1IiwiaWF0IjoxNjM1MzAwOTY0LCJleHAiOjE2NTA4NTI5NjQsImxxxxxxXXXBzOi8vYXBwbGVpZC5hcHBsZS5jb20iLCJzdWIiOiJjb20uampsZWUuV2VkUXVlZW4ifQ.Zw18dKVQxxxxxXXXXXXeSVMT8B_aTkVHULNPOal7W_n_KTba3WtYwE-fQh8Ru6GhmNdVbx2VD1TnPBjTZmKB6PaZ58w
```

{% asset_img 1.png 애플 팀 변경 시 유저 정보 migration하기 %}

* * *

### 2. access_token 생성

- 기존 provider_id를 transfer 하기 위해선 apple api로 통신해야하는데 이 때 필요한 access_token을 생성한다.
- POST 요청을 전송하기 위해 postman([https://www.postman.com/](https://www.postman.com/))을 사용
- 참고 document ([https://developer.apple.com/documentation/sign_in_with_apple/transferring_your_apps_and_users_to_another_team](https://developer.apple.com/documentation/sign_in_with_apple/transferring_your_apps_and_users_to_another_team))

1. request 작성

{% asset_img 2.png 애플 팀 변경 시 유저 정보 migration하기 %}

```txt
URL : https://appleid.apple.com/auth/token
HTTP Method : POST
Content-Type : x-www-form-urlencoded
Body : 
{
    grant_type : client_credentials
    scope : user.migration
    client_id : 클라이언트 아이디
    client_secret : 위에서 생성한 client_secret
}
```

2. 요청 후 결과 확인

- 정상적으로 응답이 온다면 아래처럼 1시간 짜리 access_token이 발급된다.
- 1 시간이 경과하면 재발급을 받아야 한다.

{% asset_img 3.png 애플 팀 변경 시 유저 정보 migration하기 %}

* * *

### 3. 기존 유저 provider Id transfer

- 변경된 팀의 provider Id로 변경하기 전 단계
- 여기서 응답받은 transfer_sub 으로 다시 변경 요청을 보내야 변경된 팀 provider id를 받을 수 있음

1. request 작성

{% asset_img 4.png 애플 팀 변경 시 유저 정보 migration하기 'access_token 설정 token_type이 bearer였으므로, bearer token으로 설정' "'access_token 설정 token_type이 bearer였으므로, bearer token으로 설정'"%}

{% asset_img 5.png 애플 팀 변경 시 유저 정보 migration하기 %}

```txt
URL : https://appleid.apple.com/auth/usermigrationinfo
HTTP Method : POST
Content-Type : x-www-form-urlencoded
Authorization: Bearer {access_token}
Body : 
{
    sub : 기존 유저 Provider Id
    target : 새로 만든 팀 ID
    client_id : 클라이언트 아이디
    client_secret : 위에서 생성한 client_secret
}
```

2. 요청 후 결과 확인

{% asset_img 6.png 애플 팀 변경 시 유저 정보 migration하기 %}

* * *

### 4. 변경된 팀 Provider Id로 transfer

- 참고 document
  - [https://developer.apple.com/documentation/sign_in_with_apple/bringing_new_apps_and_users_into_your_team](https://developer.apple.com/documentation/sign_in_with_apple/bringing_new_apps_and_users_into_your_team)
  - [https://developer.apple.com/forums/thread/666734](https://developer.apple.com/forums/thread/666734)
- 필자는 이 단계가 존재한다는 걸 몰라서 문제 해결에 많은 시간이 소요 되었다...

1. requset 생성
    - 이 때 전송되는 client_secret은 새로 만든 팀의 키 파일로 생성된 client_secret이다.
    - 이 때 전송되는 client_secret은 새로 만든 팀의 키 파일로 생성된 client_secret이다.
    - 이 때 전송되는 client_secret은 새로 만든 팀의 키 파일로 생성된 client_secret이다.
    - client_secret을 만드는 방법은 1번과 같다.
    - bearer token은 기존에 만든 것을 사용하면 된다.

{% asset_img 7.png 애플 팀 변경 시 유저 정보 migration하기 %}

```txt
URL : https://appleid.apple.com/auth/usermigrationinfo
HTTP Method : POST
Content-Type : x-www-form-urlencoded
Authorization: Bearer {access_token}
Body : 
{
    transfer_sub : 이전 단계에서 transfer된 sub
    client_id : 클라이언트 아이디
    client_secret : 새로 만든 팀의 client_secret
}
```

2. 요청 후 결과 확인

{% asset_img 8.png 애플 팀 변경 시 유저 정보 migration하기 %}

- 000000.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.0000
- 형식으로 생성되는데 이 때 맨 앞에 00000. 부분은 변경되지 않으므로
- 이 부분을 확인

* * *

### 5. DB에 저장된 유저 provider Id 변경

- 필자의 서비스는 provider ID로 유니크한 KEY를 만들어 사용하고 있었으므로 이 부분을 업데이트 해주었다.

## 참고 사이트

- [https://developer.apple.com/documentation/sign_in_with_apple/transferring_your_apps_and_users_to_another_team](https://developer.apple.com/documentation/sign_in_with_apple/transferring_your_apps_and_users_to_another_team)
- [https://developer.apple.com/documentation/sign_in_with_apple/bringing_new_apps_and_users_into_your_team](https://developer.apple.com/documentation/sign_in_with_apple/bringing_new_apps_and_users_into_your_team)
- [https://developer.apple.com/forums/thread/666734](https://developer.apple.com/forums/thread/666734)