#MSDK 토큰

##개요

이 모듈은 MSDK 모든 인증 관련 모듈을 정리하고 인증 로그인, 자동 로그인, 빠른 로그인, 토큰 갱신, 읽기 등 모듈에 대한 자세한 설명 포함. 게임은 먼저 이 모듈을 참조하여 MSDK의 모든 인증 관련 모듈을 이해한 후 게임 자체 수요에 따라 상응한 인터페이스를 사용하여 인증 등 기능 구현 가능

##용어 해석, 인터페이스 설명
	
###로그인 관련 용어 해석:
| 명칭| 설명 |지원 플랫폼| 호출 인터페이스 |
|: ------------- :|
| 인증 로그인 | 플랫폼의 인증 화면을 불러와 유저가 게임에 인증하고 로그인에 필요한 토큰을 받도록 인도| 모바일QQ/Wechat | WGLogin |
| 빠른 로그인 | 유저의 조작은 플랫폼에서 게임 실행시 로그인 관련 토큰 정보를 투과 전송하여 게임에 로그인하게 함| 모바일QQ| 없음 |
| 자동 로그인 | 게임 시작시 유저의 최근 게임 로그인 토큰 정보를 직접 사용하여 게임에 로그인| MSDK가 기능 제공| WGLoginWithLocalInfo |
| 자동 갱신 | MSDK가 Wechat 토큰 자동 갱신 인터페이스 제공| MSDK가 기능 제공 | 없음 |
| 다른 계정 | 현재 게임에 로그인한 계정과 플랫폼에 로그인한 계정 불일치 | 플랫폼/MSDK 전부 지원| WGSwitchUser |

### 로그인 관련 인터페이스 개요

로그인과 관련된 호출 인터페이스 중 `WGGetLoginRecord`, `WGLogout`는 동기적 인터페이스이고 기타 인터페이스는 전부 비동기적으로 구현되기에 callback 형식을 이용하여 OnLoginNotify(LoginRet)를 통해 최종 결과를 게임에 콜백. 다른계정과 관련된 인터페이스는 MSDK의 다른계정 모듈에서 별도로 설명. 구체 내용:

| 명칭| 설명 |비고 |
|: ------------- :|
| WGGetLoginRecord | 로컬에 저장된 현재 유저의 로그인 토큰 획득 |  |
| WGSetPermission | 게임이 유저 인증을 통해 획득할 플랫폼 정보 설정 | |
| WGLogin | 플랫폼을 실행하여 인증 로그인 진행 |  |
| WGLogout | 현재 로그인한 계정의 로그인 정보 전부 제거 |  |
| WGLoginWithLocalInfo | 로컬에 저장된 로그인 토큰으로 로그인 시도|  |
| handleCallback | 각 플랫폼 호출 처리 |  |
| WGRefreshWXToken | WechatrefreshToken으로 갱신하여 accessToken 획득 |  MSDK 2.0부터 게임이 스스로 Wechat 토큰을 갱신하는 방식을 권장하지 않음 |

### 추천 로그인 프로세스

![login](./recommend_login.png)

**스텝 설명：**

* 스텝1：게임 시작한 후에 우선 MSDK를 초기화한다.
* 스텝2：자동 로그인 인터페이스 `WGLoginWithLocalInfo()`호출하여, 해당 인터페이스는 MSDK서버단에서 로컬 토큰이 유효여부를 체크할 것이다.
* 스텝3：자동 로그인 인터페이스는 `OnLoginNotify(LoginRet ret)`를 통해 로그인 결과를 게임에 콜백.`ret.flag`를 통해 로그인 결과를 판단한다.`eFlag_Succ`(0)혹은 `eFlag_WX_RefreshTokenSucc`(2005)일 경우는 성공이며 기타 경우는 실패.
* 스텝4：로그인 인증 성공하면 스텝9로 이동.
* 스텝5：자동 로그인 실패하면,` WGLogin(EPlatform platform)`를 콜하여, QQ 혹은 위챗 플랫폼 로그인 인증을 호출.
* 스텝6：로그인 결과는 `OnLoginNotify(LoginRet ret)`를 통해 콜백.`ret.flag`를 통하여 결과 판단.`eFlag_Succ`(0)일 경우는 성공이고 기타 경우는 실패.
* 스텝7：플랫폼 호출하여 로그인 인증 실패하면, 유저로 하여금 로그인 재시도를 안내하여 스텝5로 이동.
* 스텝8：로그인 인증 성공.
* 스텝9：클라이언트에 있는 최신 토큰을 게임 서버에게 동기화한다.서버가 로그인 토큰 사용 필요할 경우，**반드시 로그인 성공의 콜백을 받은 후에야 최신 토큰을 동기화 할 수 있다** 해서 서버에서 실효된 토큰을 사용하는 상황을 피할 수 있다. 

###로그인 관련 인터페이스 권장 사용법

1. 인증 로그인: 직접 `WGLogin`을 호출하여 상응한 플랫폼의 인증 획득
- 게임 시작 또는 게임이 백그라운드에서 포어그라운드로 전환시 토큰 유효성 검사: `WGLoginWithLocalInfo`를 호출하여 토큰 유효성 검사 진행
- 토큰 획득: 직접 `WGGetLoginRecord`를 호출하여 로컬에서 읽기
- 로그아웃: 직접 `WGLogout`를 호출하여 현재 유저의 로그인 정보 제거

## 로그인 액세스 구체 작업(개발자 필독)

**게임 개발자는 아래 제공한 절차에 따라 MSDK로그인 모듈의 액세스를 진행하여 액세스 비용과 누락된 처리 로직을 줄일 수 있다. 이 부분 내용을 숙지할 것을 강력히 권장!!**

1. 유저 인증이 필요한 권한 설정:
	- 게임은 MSDK 초기화 이후 모바일QQ 권한 설정 인터페이스를 호출하여 유저 인증이 필요한 게임 플랫폼 권한을 설정해야 한다. 구체 방법[클릭하여 보기](#유저 인증이 필요한 권한 설정 WGSetPermission).
- **인증 로그인 처리**：
	1. 로그인 버튼의 클릭 이벤트의 처리 함수에서 `WGLogin`을 호출하여 인증 로그인 진행. 구체 방법[클릭하여 보기](#인증 로그인 처리 WGLogin)
- **자동 로그인 처리**：
	1. 게임이 포그라운드로 이동한 후 `WGLoginWithLocalInfo`를 호출하여 게임 실행시 자동 로그인 진행. 구체 방법[클릭하여 보기](#자동 로그인 처리 WGLoginWithLocalInfo)
	- AppDelegate의 applicationDidEnterBackground에서 게임이 백그라운드로 전환하는 시간 판단. 30분을 초과하면 자동으로 `WGLoginWithLocalInfo`를 호출하여 자동 로그인 진행
		- 게임이 백그라운드로 전환하는 시간의 판단에 있어, 게임은 MSDK의 demo 방식을 참조하여 전환시 타임 스탬프를 1개 기록하고 반환하여 시간차 계산 가능
- **유저 로그아웃 처리**:
	- 로그아웃 버튼의 클릭 이벤트의 처리 함수에서 WGLogout를 호출하여 인증 로그인 진행. 구체방법[클릭하여 보기](#유저 로그아웃 처리 WGLogout)
- **MSDK의 로그인 콜백 처리**:
	- 게임의 MSDK 콜백 처리 로직에 onLoginNotify에 대한 처리 추가. 구체방법[클릭하여 보기](#MSDK의 로그인 콜백 처리)
- **MSDK의 실행 콜백 처리**:
	- 게임의 MSDK 콜백 처리 onWakeUpNotify에서 플랫폼 실행에 대한 처리 추가. 구체방법[클릭하여 보기](#MSDK의 실행 콜백 처리)
- **다른계정 처리 로직**:
	- 게임이 다른계정에 대한 처리 로직, 구체 내용은 [MSDK 다른계정 액세스](diff-account.md#다른계정 처리 로직(개발자 주목)) 참조
- **기타 특수 로직 처리**:
	- 저메모리 설비에서 인증 진행시 게임이 강제 종료된 후 로그인 방안. 구체 방법[클릭하여 보기](#모바일QQ가 저메모리 설비에서 인증 진행시 게임이 강제 종료된 후 로그인 방안)
	- MSDKWechat 토큰 기한 만료시 자동 갱신 메커니즘. 구체방법[클릭하여 보기](#Wechat 토큰 자동 갱신)
	- 로그인 데이터 업로드 인터페이스 호출 요구. 구체방법[클릭하여 보기](#로그인 데이터 업로드)

##유저 인증이 필요한 권한 설정 WGSetPermission

####개요

게임은 MSDK 초기화 이후 모바일QQ 권한 설정 인터페이스를 호출하여 유저 인증이 필요한 게임 플랫폼 권한을 설정해야 한다.

####인터페이스 선언

	/**
	 * @param permissions ePermission 열거값 또는 연산 결과, 필요한 인증 항 표시
	 * @return void
	 */
	void WGSetPermission(int permissions);

#### 인터페이스 호출:

	// QQ 실행시 필요한 유저 인증 항 설정
	WGPlatform.WGSetPermission(WGQZonePermissions.eOPEN_ALL); 

#### 주의사항:

1. 게임은 MSDK 초기화 이후에 이 인터페이스를 호출해야 하며 인터페이스 파라미터는 **`eOPEN_ALL`**을 입력할 것을 제안한다. 이 내용이 부족하면 게임이 일부 인터페이스 호출 시 권한이 없다고 통지할 수 있다.


##인증 로그인 처리 WGLogin

#### 개요:

**모바일QQ/Wechat 클라이언트 또는 web페이지(모바일QQ 미설치)를 실행하여 인증을 진행하고 유저가 인증한 후 onLoginNotify를 통해 openID, accessToken, payToken, pf, pfkey 등 로그인 정보를 획득했다고 게임에 통지**


####인터페이스 선언:

	/**
	 * @param platform 게임이 전송한 플랫폼 유형, 가능한 값: ePlatform_QQ, ePlatform_Weixin
	 * @return void
	 *   게임이 설정한 전역 콜백의 OnLoginNotify(LoginRet& loginRet) 메소드를 통해 데이트를 게임에 반환
	 */
	void WGLogin(ePlatform platform);

#### 인터페이스 호출:

	WGPlatform::GetInstance()->WGLogin(ePlatform_QQ); 

#### 주의사항:

- **통용**:
	- **OnLoginNotify/OnWakeupNotify콜백을 받지 못한다：
		- AppDelegate의 didFinishLaunchingWithOptions함수가 YES로 return를 확보해야 한다;
		- info.plist에 QQ와 위챗 appid/appkey설정이 정확하다.
		- Url Schemes는 wiki 요구에 따라 설정
	
##자동 로그인 처리 WGLoginWithLocalInfo

#### 개요:

이 인터페이스는 과거에 로그인했던 게임에 사용된다. 유저가 게임에 다시 들어갈 때 게임은 이 인터페이스를 먼저 호출하여 백그라운드에서 토큰을 검사한 후 OnLoginNotify를 통해 결과를 게임에 콜백한다. 게임은 더 이상 위챗 토큰 새로고침과  QQ/위챗 AccessToken검증 등을 처리할 필요없다.

####인터페이스 선언:

	/**
	  *  @since 2.0.0
	  *  이 인터페이스는 과거에 로그인했던 게임에 사용된다. 유저가 게임에 다시 들어갈 때 게임은 이 인터페이스를 먼저 호출하여 백그라운드에서 토큰 검사를 시도한다
	   *  이 인터페이스는 OnLoginNotify를 통해 결과를 게임에 콜백하고 eFlag_Local_Invalid와 eFlag_Succ 등 2개 flag만 반환한다.
	  *  로컬에 토큰이 없거나 로컬 토큰 검사 실패시 반환하는 flag는 eFlag_Local_Invalid이다. 게임이 이 flag를 받으면 유저를 인증 페이지로 안내하여 인증을 진행하면 된다.
	  *  로컬에 토큰이 있고 검사에 성공하면 flag는 eFlag_Succ이다. 게임이 이 flag를 받으면 재차 검사할 필요가 없이 sdk가 제공한 토큰을 직접 사용할 수 있다.
	  *  @return void
	  *   Callback: 검사 결과는 OnLoginNotify를 통해 반환
	  */
 	void WGLoginWithLocalInfo();

####주의사항:

1. 게임이 `WGLoginWithLocalInfo`를 사용하여 로그인시 획득한 토큰은 게임 백그라운드에 전송하여 유효성 검사를 진행할 필요가 없이 MSDK가 검사한 후 게임에 콜백한다

##유저 로그아웃 처리 WGLogout

#### 개요:

이 인터페이스를 호출하면 현재 로그인 계정의 로그인 정보를 제거할 수 있다.

####인터페이스 선언:

	/**
	 * @return bool 리턴값을 이미 폐기, 전부 true 반환
	 */
	bool WGLogout();

####호출 예:

    WGPlatform.WGLogout();

####주의사항:

1. 게임이 **로그아웃 버튼을 클릭하거나 로그인창이 팝업되는 로직에서 반드시 WGLogout를 호출하여 로컬 로그인 정보를 전부 제거해야 한다**. 그렇지 않으면 인증 실패 등 문제를 초래할 수 있다

##MSDK의 로그인 콜백 처리

#### 개요:

MSDK의 로그인 콜백은 아래 몇개 시나리오에서 생긴다:

- WGLogin 인증에서 돌아옴
- WGLoginWithLocalInfo 로그인에서 돌아옴
- 플랫폼 실행을 처리한 후(토큰을 갖고 실행하면)

#### 구체적 처리:

	OnLoginNotify(LoginRet ret) {
        Logger.d("called");
        switch (ret.flag) {
            case CallbackFlag.eFlag_Succ:
				 CallbackFlag.eFlag_WX_RefreshTokenSucc
            	//인증 성공의 처리 로직
				break;
            case CallbackFlag.eFlag_WX_UserCancel:
				 CallbackFlag.eFlag_QQ_UserCancel
				//유저가 인증을 취소하는 로직
				break;
			case CallbackFlag.eFlag_WX_UserDeny
				//유저가 Wechat인증을 거절하는 로직
				break;
            case CallbackFlag.eFlag_WX_NotInstall:
				//유저 설비에 Wechat 클라이언트가 설치되지 않은 로직
				break;
			case CallbackFlag.eFlag_QQ_NotInstall:
				//유저 설비에 QQ 클라이언트가 설치되지 않은 로직
				break;
            case CallbackFlag.eFlag_WX_NotSupportApi:
				//유저 Wechat 클라이언트가 이 인터페이스 로직을 지원하지 않음
				break;
            case CallbackFlag.eFlag_QQ_NotSupportApi:
				//유저 모바일QQ 클라이언트가 이 인터페이스 로직을 지원하지 않음
				break;
            case CallbackFlag.eFlag_NotInWhiteList
				//유저 계정이 화이트리스트에 없는 로직
				break;
            default:
                // 기타 로그인 실패 로직
                break;
        }
    }

#### 주의사항:
**이곳에서는 중요한 loginNotify의 로직만 처리했을 뿐이고, 완전한 콜백 flag 정보는 [콜백표시eFlag](const.md#콜백표시eFlag)를 클릭하여 확인할 수 있다. 게임은 자체 수요에 따라 처리할 수 있다**

##MSDK의 실행 콜백 처리

#### 개요

게임이 플랫폼 실행에 대한 처리는 주로 다른계정과 관련된 로직을 처리하는 것이다. 구체적 처리는 다음과 같다

#### 구체적 처리：

        if (CallbackFlag.eFlag_Succ == ret.flag
                || CallbackFlag.eFlag_AccountRefresh == ret.flag) {
            //실행 후 로컬 계정을 통해 게임에 로그인하는 것을 표시. 처리 로직은 onLoginNotify와 일치
            
        } else if (CallbackFlag.eFlag_UrlLogin == ret.flag) {
            // MSDK은 계정을 불러와 토큰을 갖고 인증 로그인 시도. 결과는 OnLoginNotify에서 콜백. 이때 게임은 onLoginNotify 콜백 대기

        } else if (ret.flag == CallbackFlag.eFlag_NeedSelectAccount) {
            // 현재 게임에 다른계정이 존재하기에 게임은 대화창을 팝업하여 유저에게 로그인할 계정을 선택하게 해야 한다

        } else if (ret.flag == CallbackFlag.eFlag_NeedLogin) {
            // 유효 토큰이 없어 게임 로그인 실패. 이때 게임은 WGLogout를 호출하여 로그아웃하여 유저가 다시 로그인하게 한다

        } else {
            //기본 처리 로직은 게임이 WGLogout를 호출하여 게임에서 로그아웃한 후 유저가 다시 로그인할 것을 제안한다
        }

##다른계정 처리 로직

다른계정 관련 모듈은 [MSDK 다른계정 액세스](diff-account.md#다른계정 처리 로직(개발자 주목）)을 참조할 수 있다

##기타 특수 로직 처리

### 모바일QQ가 저메모리 설비에서 인증 진행시 게임이 강제 종료된 후 로그인 방안

시중 대부분 게임이 메모리를 많이 점용하기에 인증 과정에서 모바일QQ를 불러와 인증 진행 시 iOS 메모리 복구 메커니즘이 백그라운드의 게임 프로세스를 강제로 종료시켜 게임 모바일QQ 인증이 게임에 들어가지 못하는 문제를 초래할 수 있다. 게임은 AppDelegate에서 아래 코드를 추가하여 프로세스가 종료된 후에도 토큰을 갖고 게임에 들어갈 수 있게 해야 한다.
```ruby
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    NSLog(@"!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!url == %@",url);
    WGPlatform* plat = WGPlatform::GetInstance();
    WGPlatformObserver *ob = plat->GetObserver();
    if (!ob) {
        MyObserver* ob = new MyObserver();
        ob->setViewcontroller(self.viewController);
        plat->WGSetObserver(ob);
    }
	//광고 기능을 구현되지 않을 경우 WGADObserver 설정하지 않아도 된다
    WGADObserver *adOb = plat->GetADObserver();
    if (!adOb) {
        MyAdObserver *adObserver = new MyAdObserver();
        plat->WGSetADObserver(adObserver);
    }
     return  [WGInterface  HandleOpenURL:url];
}
```
### Wechat 토큰 자동 갱신

1. MSDK2.0.0부터 게임 운행 기간에 주기적으로 Wechat 토큰을 검사하고 갱신한다. 갱신이 필요할 경우 MSDK가 자동으로 갱신을 진행하고 OnLoginNotify를 통해 게임에 통지한다. flag는 eFlag_WX_RefreshTokenSucc와 eFlag_WX_RefreshTokenFail이다(이미 onLoginNotify의 콜백에 포함됨).
- **게임이 새로운 토큰을 받으면 게임 클라이언트에 저장된 토큰과 서버의 토큰을 동기적으로 갱신하여 새로운 토큰으로 후속적인 절차를 진행할 수 있게 해야 한다.**
- 게임에 Wechat 토큰 자동 갱신 기능이 필요하지 않으면 `info.plist`에서 `AutoRefreshToken`를 `NO`로 설정하면 된다.

## 기타 인터페이스 리스트

###WGGetLoginRecord

#### 개요:

이 인터페이스를 호출하면 현재 계정의 로그인 정보를 획득할 수 있다.

####인터페이스 선언:

	/**
	 * @param loginRet 반환한 기록
	 * @return 반환 값은 플랫폼 id, 유형은 ePlatform, ePlatform_None을 반환하면 로그인 기록이 없음을 표시한다
	 *   loginRet.platform(유형 ePlatform) 플랫폼id 표시, 가능한 값은 ePlatform_QQ, ePlatform_Weixin, ePlatform_None.
	 *   loginRet.flag(유형 eFlag) 현재 로컬 토큰의 상태 표시, 가능한 값과 설명은 다음과 같다:
	 *     eFlag_Succ: 인증 토큰 유효
	 *     eFlag_QQ_AccessTokenExpired: 모바일QQ accessToken 기한 만료, 인증 인터페이스 표시, 유저가 다시 인증하도록 안내
	 *     eFlag_WX_AccessTokenExpired: WechataccessToken 토큰 기한 만료, WGRefreshWXToken을 호출하여 갱신해야 한다
	 *     eFlag_WX_RefreshTokenExpired: WechatrefreshToken, 인증 인터페이스 표시, 유저가 다시 인증하도록 안내
	 *   ret.token은 하나의 Vector<TokenRet>이고 그중에 저장된 TokenRet에 type와 value가 있다. Vector 순회를 통해 type를 판단하여 필요한 토큰을 읽는다.
     *
	 */
	int WGGetLoginRecord(LoginRet& loginRet);

####호출 예:

    LoginRet ret = new LoginRet();
    WGPlatform.WGGetLoginRecord(ret);

획득한 LoginRet 중 flag가 eFlag_Succ이면 로그인이 유효하다고 인정할 수 있으며 유효한 토큰 정보를 읽을 수 있다. 그중 token은 아래 방식으로 획득할 수 있다.

Wechat 플랫폼:

   NSString *accessToken = "";
    NSString *refreshToken = "";
    for (int i = 0; i < loginRet.token.size(); i++) {
             if (loginRet.token.at(i).type == eToken_WX_Access) {
                 accessToken = [NSString stringWithCString:loginRet.token.at(i).value.c_str() encoding:NSUTF8StringEncoding];
             } else if (loginRet.token.at(i).type == eToken_WX_Refresh) {
                 payToken = [NSString stringWithCString:loginRet.token.at(i).value.c_str() encoding:NSUTF8StringEncoding];
             }
    }

QQ 플랫폼:

    NSString *accessToken = "";
    NSString *payToken = "";
    for (int i = 0; i < loginRet.token.size(); i++) {
        if (loginRet.token.at(i).type == eToken_QQ_Access) {
            accessToken = [NSString stringWithCString:loginRet.token.at(i).value.c_str() encoding:NSUTF8StringEncoding];
        } else if (loginRet.token.at(i).type == eToken_QQ_Pay) {
            payToken = [NSString stringWithCString:loginRet.token.at(i).value.c_str() encoding:NSUTF8StringEncoding];
        }
    }

#### 주의사항:

없음

##자주 발생하는 문제

1. 결제시 paytoken 기한 만료를 제시하면 로그인 인터페이스를 불러와서 다시 인증해야 결제할 수 있다. paytoken 기한이 만료되면 반드시 다시 인증해야 한다.
