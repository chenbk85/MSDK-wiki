﻿MSDK 다른계정 문제 
=======

다른계정이란
---

다른계정은: 현재 게임에 로그인한 계정과 플랫폼에 로그인한 계정이 일치하지 않는 경우를 말한다. 다음과 같은 2가지 경우가 있다

1. 플랫폼이 같지만 계정이 다름(예하면, 게임과 모바일QQ가 부동한 QQ계정으로 로그인)
2. 플랫폼이 다름(예하면, 게임은 위챗으로 로그인하고 게임 관련 조작은 모바일QQ에서 진행)

다른계정 scene
---

1. 게임이 메시지를 공유하려고 플랫폼을 실행할 때 계정이 일치하지 않으면 플랫폼이 팝업창으로 다른계정 제시. 현재 모든 플랫폼은 이 기능 지원


2. 유저가 플랫폼을 통해 게임을 실행할 때 계정이 일치하지 않으면 게임이 팝업창으로 다른계정 제시

`현재 게임 런칭 시 플랫폼이 게임에 요구하는 다른계정 처리, MSDK가 구현한 다른계정은 모두 두번째 경우에 속한다.`

다른계정 처리 로직(개발자 주목)
---
MSDK의 다른계정 처리 로직은 다른계정 판단, 유저가 계정 선택, 다른계정 로그인 등 3개 절차를 포함한다(자세한 내용은 MSDK 액세스 문서 참조). 간단히 설명하자면:

####1. 다른계정 판단:

외부 플랫폼에서 게임 실행시 MSDK는 handCallback에서 실행 플랫폼과 게임 로컬 계정에 다른계정 존재 여부를 판단하고 다른계정 판단 결과를 OnWakeupNotify(WakeupRet ret) 방법의 ret.flag를 통해 게임에 콜백한다. 게임은 콜백 결과에 따라 상응한 로그인 처리를 진행할 수 있다. 자세한 ret.flag 및 상응한 처리는 다음과 같다.

	eFlag_Succ: 다른계정이 존재하지 않고 로컬 계정으로 로그인 성공. 게임은 이 flag를 수신하면 직접 LoginRet 구조체 중 토큰을 획득하여 게임 인증 절차를 진행한다.

	eFlag_AccountRefresh: 다른계정이 존재하지 않고 MSDK가 이미 인터페이스 새로고침을 통해 로컬 계정 토큰을 새로 고침. 이 flag를 수신하면 직접 LoginRet 구조체 중 토큰을 획득하여 게임 인증 절차를 진행한다.

	eFlag_UrlLogin：다른계정이 존재하지 않고 게임이 계정을 이용하여 빠른 로그인.게임이 해당 이 flag 수신한 후 onLoginNotify의 콜백을 받아야 처리한다.

	eFlag_NeedLogin：게임 로컬 계정과 실행 계정이 모두 로그인 실패. 게임이 이 flag를 수신하면 로그인창을 팝업하여 유저를 로그인하게 해야 한다.

	eFlag_NeedSelectAccount：게임 로컬 계정과 실행 계정에 다른계정이 존재. 게임이 이 flag를 수신하면 대화창을 팝업하여 유저에게 로그인할 계정을 선택하게 해야 한다.

####2. 유저 선택:

다른계정 판단 ret.flag가 eFlag_NeedSelectAccount일 경우, 게임은 대화창을 팝업하여 유저에게 로컬 계정 또는 다른 계정으로 게임에 로그인할 지 문의한다. 게임은 유저가 선택한 결과에 따라 인터페이스 WGSwitchUser를 호출하여 로그인을 완료해야 한다.

		/**
		 *  외부에서 실행한 URL을 통해 로그인. 이 인터페이스는 다른계정 발생시 유저가 외부에서 계정 실행시 호출한다.
	 	*  로그인 성공 후 onLoginNotify를 통해 콜백
	 	*
	 	*  @param flag 가 YES이면, 유저가 실행 계정으로 바꾸려 한다는 것을 표시한다. 이럴 경우, 이 인터페이스는 지난번에 저장한 실행 계정 로그인 데이터를 이용해 게임에 로그인 시도하고 로그인에 성공한 후 onLoginNotify를 통해 콜백한다. 토큰이 없거나 토큰이 무효 함수이면 NO를 리턴하고 onLoginNotify 콜백이 발생하지 않는다.
	 	*              NO이면, 유저가 원래 로컬계정을 계속 사용한다는 것을 표시한다. 이때 저장된 실행 계정 데이터를 삭제하여 혼동을 방지한다.
	 	*
	 	*  @return 토큰이 없거나 무효하면 NO 리턴; 다른 경우는 YES 리턴
		 */
		bool WGSwitchUser(bool flag);

####3. 로그인 콜백 리턴:

WGSwitchUser의 파라미터가 true일 경우，MSDK는 게임 호출 때의 상태에 따라 로그인을 시도하고 onLoginNotify를 통해 로그인 결과를 게임에 콜백한다. 게임은 콜백 결과에 따라 로그인 결과를 처리한다.

플랫폼에서 게임으로 다른계정 9가지 경우
----

**개발자와 무관하며, 개발자는 MSDK가 게임에 콜백한 flag만 관심하면 된다.**：

|           |완전한 토큰을 갖고 실행|완전한 토큰이 없이 실행|토큰이 없이 실행|
|: ------------- :|: ------------- :|: ------------- :|: ------------- :|
|로컬 토큰 유효|유저에게 다른계정 제시|유저에게 다른계정 제시|로컬 계정으로 로그인|
|로컬 토큰 무효|유저에게 다른계정 제시|유저에게 다른계정 제시|게임은 로그인 페이지에 돌아감|
|로컬에 토큰 없음|계정을 통해 로그인|게임은 로그인 페이지에 돌아감|게임은 로그인 페이지에 돌아감|


다른 계정 버전 지원
-----
####게임에서 플랫폼으로 다른계정:

1. 1.8.0 아래 버전에서, 게임에서 위챗으로 다른계정은 이미 알려진 MSDK BUG가 자동 로그인 시 제시가 없는 문제를 초래할 수 있다. MSDK 버전을 1.8.0 이상으로 업데이트하여 이 문제를 해결하기 바란다.
2. 게임에서 위챗으로 다른계정은 위챗 5.0 및 이상 버전에서만 지원한다.

####플랫폼에서 게임으로 다른계정:
1. MSDK는 1.8.0부터 다른계정을 지원한다. 현재 모바일QQ만 토큰을 갖고 실행할 수 있다.
2. 모바일QQ4.6 아래 버전에서, 모바일QQ에서 게임으로 다른계정은 게임이 이미 실행된 상황에서 존재하지 않는다.


##FAQ

1. 공유한 메시지를 클릭하여 다른 계정이 없다：메시지 보기를 클릭[공유 메시지 클릭 효과](share.md#공유 메시지 클릭 효과)현재 조작이 다른 계정을 유발 여부를 확인.
2. 모바일 QQ 게임 호출하여 로그인 불가：[클릭하여 QQ지원하는 퀵 로그인 조건을 알아보기](qq.md#퀵 로그인)，네용에 의하여 현 게임 QQ 퀵 로그인 지원 여부를 확인.