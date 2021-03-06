﻿MSDK마이앱 관련 모듈
==================


# 마이앱 계정 가챠

## 계정 가챠 스위치 설정

계정 가챠 스위치는 두곳에 있다. 하나는 **assets/msdkconfig.ini**에서 클라이언트 계정 가챠 스위치를 설정할 수 있고 다른 하나는 마이앱에서 계정 가챠 스위치를 설정해야 한다. 프런트엔드에서 계정 가챠 기능을 이용하려면 다음과 같이 설정하면 된다.

	; MSDK 모듈 기능 on/off 선택 가능
	; 마이앱 계정 가챠 스위치
	BETA=true

## 계정 가챠 액세스 절차

- **1단계: **게임의 AndroidMenifest.xml에 Service 선언 추가

```
<service 
	android:name="com.tencent.tmassistantsdk.downloadservice.TMAssistantDownloadSDKService"
    android:exported="false"
    android:process=":TMAssistantDownloadSDKService" >
</service>
```

- **2단계: **게임 메인 Acitivity의 onResume과 onDestroy에 각각 MSDK의 대응한 방법을 호출

```
@Override
protected void onResume() {
    super.onResume();
    WGPlatform.onResume();
}
@Override
protected void onDestroy() {
    super.onDestroy();
    WGPlatform.onDestroy(this);
}
```

- **3단계: **계정 가챠 문자를 게임명에 대응하는 파일 경로로 수정: **MSDKLibrary\res\values\com_tencent_tmassistant_sdk_strings.xml**수정 방법: 아래**“Speed Up”**을 대응하는 게임명으로 수정하면 된다.
	
```
<string name="white_list_dlg_body">Speed U의 소규모 CBT에 참가하고 싶나요? 어서 마이앱 계정 가챠를 사용해요!</string>
```

플랫폼 로그인이 완료된 후 MSDK에 돌아가면 MSDK는 화이트리스트 확인 요청을 시작한다. 이 요청이 리턴될 때 MSDK는 OnLoginNotify를 통해 게임에 통지한다. 유저가 화이트리스트에 있으면 이 flag는 eFlag_Succ이며 이런 경우는 정상적인 로그인과 일치하다. 유저가 화이트리스트에 없으면 flag는 eFlag_NotInWhiteList이며 게임에 리턴하는 동시에 MSDK는 팝업창을 통해 유저를 마이앱 계정 가챠로 안내한다.

**PS: 계정 가챠 기능의 액세스 성공 여부를 검증하는 방법**
임의의 새로운 QQ 또는 위챗 계정으로 게임에 로그인할 때 다음과 같은 대화창이 팝업되면 마이앱 계정 가챠 기능이 액세스되었음을 표시한다 

<div align=center> <img src="./myapp_beta_success.jpg" alt="계정 가챠 액세스 완료" height=640 weight=360> </div>


# 트래픽 절약 업데이트

## 트래픽 절약 업데이트 스위치 설정

업데이트 기능을 사용하려면 **assets/msdkconfig.ini**에서 `SAVE_UPDATE` 스위치를 설정해야 한다 
	
	; SAVE_UPDATE
	SAVE_UPDATE=true

## 트래픽 절약 업데이트 액세스 설정

`AndroidManifest.xml` 설정
	
	<service 
		android:name="com.tencent.tmassistantsdk.downloadservice.TMAssistantDownloadSDKService"
        android:exported="false"
        android:process=":TMAssistantDownloadSDKService" >
    </service>

마이앱 sdk를 통해 게임을 업데이트하는 데는 두가지 방법이 있다. 

- 일반 업데이트, 직접 게임내 마이앱 백그라운드에서 업데이트 패키지를 내려받는다
- 트래픽 절약 업데이트, 업데이트 버전이라고도 부른다. 이런 업데이트 방식은 앱 패키지 클라이언트가 필요하다. 트래픽 절약 업데이트는 파일을 대조 비교한 후 변동된 부분만 업데이트하기에 업데이트 패키지 용량을 줄이고 업데이트 성공률을 향상시킬 수 있다.

게임이 마이앱 트래픽 절약 업데이트를 액세스하는 흐름도는 다음과 같다

![myapp_update](./myapp_update.jpg "마이앱 업데이트 흐름도")

## 트래픽 절약 업데이트 디버깅

마이앱 트래픽 절약 업데이트를 사용하려면 다음과 같은 몇 단계가 있다.

- 1단계: 게임 Activity 라이프사이클 데이터 모니터링

```
@Override
protected void onResume() {
    super.onResume();
    WGPlatform.onResume();
}
@Override
protected void onDestroy() {
	super.onDestroy();
	WGPlatform.onDestory(this);
}
```

- 2단계: 초기화 시 마이앱 트래픽 절약 업데이트의 전역 콜백 객체 설정. 관련된 콜백의 자세한 설명은 이곳 참조:
**MSDKLibrary/jni/CommonFiles/WGSaveUpdateObserver.h**。

마이앱 업데이터 콜백 유형, 게임 자체 구현


    WGPlatform.WGSetSaveUpdateObserver(new SaveUpdateDemoObserver()); 

    class SaveUpdateDemoObserver extends WGSaveUpdateObserver{
        @Override
        public void OnCheckNeedUpdateInfo(long newApkSize, String newFeature, long patchSize,
                final int status, String updateDownloadUrl, final int updateMethod) {
            Logger.d("called");
            String statusDesc = "";
            switch (status) {
                case TMSelfUpdateSDKUpdateInfo.STATUS_OK:
                    // 업데이터 조회 성공
                    statusDesc = "Check success!";
                    break;
                case TMSelfUpdateSDKUpdateInfo.STATUS_CHECKUPDATE_RESPONSE_IS_NULL:
                    // 조회 응답은 null
                    statusDesc = "Response is null!";
                    break;
                case TMSelfUpdateSDKUpdateInfo.STATUS_CHECKUPDATE_FAILURE:
                    // 업데이트 조회 실패
                    statusDesc = "CheckNeedUpdate FAILURE!";
                    break;
            }
            if(status == TMSelfUpdateSDKUpdateInfo.STATUS_OK) {
                switch(updateMethod) {
                    case TMSelfUpdateSDKUpdateInfo.UpdateMethod_NoUpdate:
                        // 업데이트 패키지가 없음
                        statusDesc += "But no update package.";
                        break;
                    case TMSelfUpdateSDKUpdateInfo.UpdateMethod_Normal:
                        // 풀 용량 업데이트 패키지가 있음
                        // WGPlatform.WGStartCommonUpdate(); //게임 업데이트
                        statusDesc += "Common package is available.";
                        break;
                    case TMSelfUpdateSDKUpdateInfo.UpdateMethod_ByPatch:
                        // 트래픽 절약 업데이트 패키지가 있음
                        // WGPlatform.WGStartSaveUpdate(); //게임 업데이트
                        statusDesc += "Save update package is available.";
                        break;
                    default :
                        statusDesc += "Happen error!";
                        break;
                }
            }
            Logger.d(statusDesc);
            MsdkCallback.sendResult(statusDesc);
        }

        @Override
        public void OnDownloadAppProgressChanged(final long receiveDataLen, final long totalDataLen) {
            // 게임 다운로드 진행률은 이곳에서 콜백. 게임은 콜백된 파라미터에 따라 진행률을 표시할 수 있음
            Logger.d("totalData:" + totalDataLen + "receiveData:" + receiveDataLen);
        }

        @Override
        public void OnDownloadAppStateChanged(int state, int errorCode, String errorMsg) {
            // 다운로드 진행률은 이곳에서 콜백
            switch (state) {
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_SUCCEED:
                        // 마이앱내 게임 다운로드 작업 완료. 업데이트 완료 후 계속 플레이
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_DOWNLOADING:
                        // 마이앱내 게임 다운로드 중. 게임은 대기 애니메이션을 출력하거나 OnDownloadAppProgressChanged와 결합하여 다운로드 진행률 표시
                        break;
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_WAITING:
                        // 마이앱내 게임 다운로드 작업 대기 중. 유저에게 대기하라고 안내
                        break;
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_PAUSED:
                        break;
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_FAILED:
                        // 자세한 오류 코드는 errorCode에 존재, 오류 코드 정의는 TMAssistantDownloadSDKErrorCode 중 DownloadSDKErrorCode로 시작되는 속성에 존재
                        break;
                    case TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_DELETE:
                        break;
            } 
            Logger.d(String.format("%d, %d, %s", state, errorCode, errorMsg));
        }
        
        /**
         * 트래픽 절약 업데이트(WGStartSaveUpdate), 마이앱이 설치되지 않았을 경우 마이앱을 먼저 내려받는다. 이는 마이앱 패키지 다운로드 진행률 콜백이다
         * @param url 현재 작업의 url
         * @param receiveDataLen 수신된 데이터 길이
         * @param totalDataLen 수신해야 할 모든 데이터 길이(대상 파일의 총 길이를 획득할 수 없으면 이 파라미터는 －1 리턴)
         */
        @Override
        public void OnDownloadYYBProgressChanged(String url, final long receiveDataLen, final long totalDataLen) {
            // 마이앱 다운로드 진행률은 이곳에서 콜백, 게임은 콜백된 파라미터에 따라 진행률을 표시할 수 있음
            Logger.d("totalData:" + totalDataLen + "receiveData:" + receiveDataLen);
        }
        
        /**
         * @param url 지정된 작업의 url
         * @param state 다운로드 상태: 값 TMAssistantDownloadSDKTaskState.DownloadSDKTaskState_*
         * @param errorCode 오류 코드
         * @param errorMsg 오류 설명, null이 될 수 있음
         */
        @Override
        public void OnDownloadYYBStateChanged(final String url, final int state, final int errorCode, final String errorMsg) {
             Logger.d("OnDownloadYYBStateChanged " + "\nstate:" + state + 
                    "\nerrorCode:" + errorCode + "\nerrorMsg:" + errorMsg);
        }
    }


- 3단계: `WGCheckNeedUpdate`를 호출하고 `OnCheckNeedUpdateInfo` 중 `updateMethod` 콜백에 따라 사용 가능한 업데이트 방식을 선택. 각 인터페이스는 다음과 같다

```
/**
 * @param saveUpdateObserver 트래픽 절약 업데이트 전역 콜백. 업데이트와 관련된 모든 콜백은 이 객체를 통해 콜백
 */
void WGSetSaveUpdateObserver(WGSaveUpdateObserver * saveUpdateObserver);

/**
 * @return void
 *   조회 결과가 WGSetSaveUpdateObserver 인터페이스에서 설정한 콜백 객체에 콜백되는 OnCheckNeedUpdateInfo 방법
 */
void WGCheckNeedUpdate()

/**
 * 일반 업데이트 시작. 이런 업데이트는 마이앱 클라이언트에 의존하지 않고 다운로드 진행률과 상태 변화는 OnDownloadAppProgressChanged와 OnDownloadAppStateChanged를 통해 게임에 콜백된다
 */
void WGStartCommonUpdate();

/**
 * 휴대폰에 마이앱이 설치되지 않았을 경우 이 인터페이스는 자동으로 마이앱을 내려받고 OnDownloadYYBProgressChanged와 OnDownloadYYBStateChanged 인터페이스를 통해 각각 콜백한다
 * 휴대폰에 마이앱이 설치되어 있으면 이 인터페이스는 마이앱을 실행시킨다. 다운로드 진행률과 상태 변화는 OnDownloadAppProgressChanged와 OnDownloadAppStateChanged를 통해 게임에 콜백한다
 */
void WGStartSaveUpdate()

```

## 트래픽 절약 업데이트 체험도

체험도는 트래픽을 절약하는 사용 환경을 모의할 뿐이다. 그중 **UI는 게임이 자체 정의한다**. 마이앱 콜백 함수는 상태와 다운로드 진행률을 업데이트하며 자세한 내용은 트래픽 절약 액세스 장절의 내용을 참조하기 바란다. 트래픽 절약 업데이트 디버깅 시, 디버깅 휴대폰에 설치된 게임 버전은 마이앱에 업로드된 버전보다 낮아야 한다.

![myapp_update](./updatesave1.png "트래픽 절약 업데이트 체험도1")

![myapp_update](./updatesave2.png "트래픽 절약 업데이트 체험도2")

![myapp_update](./updatesave3.png "트래픽 절약 업데이트 체험도3")

![myapp_update](./updatesave4.png "트래픽 절약 업데이트 체험도4")

![myapp_update](./updatesave5.png "트래픽 절약 업데이트 체험도5")
