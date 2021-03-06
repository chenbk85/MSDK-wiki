﻿MSDK Android 소개
=======

모듈 소개
---

| 모듈명 | 모듈 기능 | 액세스 조건 |
| ------------- |:-------------:|
|데이터 분석 모듈|데이터 보고, 이상 보고 제공	||
|모바일QQ	 |모바일QQ에 로그인 및 공유 기능 제공	|모바일QQ appId와 appKey 필요|
|위챗 |	위챗에 로그인 및 공유 기능 제공	|위챗 appId와 appKey 필요|
|QQ게임로비	| 게임로비의 게임 실행 능력 제공|
|내장브라우저	| 인앱 내장브라우저 기능 제공	|
|공지	| 롤링 공지 팝업 기능 제공	||
|LBS	| LBS 기능 제공	||
|마이앱 계정 가챠|	마이앱 계정 가챠 기능 제공	|액세스 프로세스는 SDK패키지내 “마이앱 계정 가챠 기능 액세스 가이드라인” 참조|
|마이앱 트래픽 절약 업데이트|	마이앱 트래픽 절약 업데이트 기능 제공|
주의사항:
- 결제(Midas)는 MSDK와 별도로 존재하기에 관련 결제 문서를 확인하기 바란다.
- appId, appKey, offerId는 관련 모듈에 액세스하는 증빙이다. 신청 방법은 “MSDK 제품 소개”를 참조하면 된다.

용어 해석
---

| 이름 | 용어 개요 |
| ------------- |:-------------:|
| 플랫폼| 위챗, 모바일QQ를 통털어 플랫폼이라 부른다|
|openId|유저 인증 후 플랫폼이 반환하는 고유 마크|
|accessToken|유저 인증 증표. 이 증표를 획득하면 유저가 인증되었음을 표시한다. 공유/결제 등 기능은 이 증표를 제시해야 한다. 모바일QQ의 accessToken 유효 기간은 90일이다. 위챗의 accessToken 유효 기간은 2시간이다.|
|payToken|결제 증표. 이 증표는 모바일QQ 결제에 사용된다. 모바일QQ 인증은 이 증표를 반환하고 위챗 인증은 이 증표를 반환하지 않는다. 유효 시간은 6일이다.|
|offerId|결제시 사용한다. 안드로이드의 offerid는 모바일QQappid이다|
|refreshToken|위챗 플랫폼 고유 증표, 유효 기간은 30일이다. 위챗accessToken 만료 후 accessToken 갱신에 사용된다.|
|다른 계정|게임에서 인증한 계정과 모바일QQ/위챗에서 인증한 계정이 다르면 이런 경우를 다른 계정이라 부른다. |
|구조화 메시지|메시지를 공유하는 방식. 이런 메시지의 공유 후 표시 형식: 왼쪽은 썸네일, 우측 상단은 메시지 제목, 우측 하단은 메시지 개요. |
|빅이미지 메시지|메시지를 공유하는 방식. 이런 메시지는 이미지 한 장만 포함하고 표시할 때도 이미지 한 장만 표시한다. 빅이미지 공유, 단순 이미지 공유라고 부르기도 한다.|
|게임친구|모바일QQ 또는 위챗 친구 중 같은 게임을 놀아본 친구를 게임친구라고 부른다|
|게임센터|모바일QQ 클라이언트 또는 위챗 클라이언트의 게임센터를 통틀어 게임센터라고 부른다.|
|게임로비|QQ 게임로비를 특별히 지칭한다|
|플랫폼 실행|플랫폼 또는 경로(모바일QQ/위챗/게임로비/마이앱 등)에서 게임 시작|
|관계사슬|유저가 플랫폼에서 가진 친구 관계|
|대화|모바일QQ 또는 위챗의 채팅 메시지|
|설치 경로|게임 출시전 패키징시 부동한 경로(예하면 마이앱, SnapPea, 91 등)에 따라 부동한 경로 번호 apk 패키지를 생성하게 된다. 설치 패키지에 있는 채널 번호를 설치 경로라 부른다.|
|등록 경로|유저 최초 로그인시 게임의 설치 경로는 MSDK 백그라운드에 기록되어 유저 등록 경로로 간주된다.|
|Pf|결제시 필요한 필드로, 데이터 분석에 사용된다. pf 구성: 실행 플랫폼_계정 체계-등록 경로-운영체제-설치 경로-사용자정의 필드.|
|pfKey| 결제용|

버전 역사
---
* [클릭하면 MSDK 버전 변경 역사를 볼 수 있습니다](version.md)
