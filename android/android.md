Android ����
=======

#### ���ṹ˵��
MSDK�ķ�����(zip)��Ҫ����������Ҫ����, MSDKLibrary��MSDKSample, ǰ��ΪMSDK��, ����Ӧ��ǰ��, ��MSDK��ʹ��ʵ��. ��ant����MSDKSample���ɻ�ÿ�ִ�е�apk������MSDKSample/bin��.
MSDK������Android Library Project����ʽ�ṩ, ��Ϸ����MSDKLibrary����ʹ��, MSDKLibrary�а�����: ����Ҫ��jar��, so�ļ�, ��Դ�ļ�. ����Ŀ¼�ṹ����ͼ:
![library_progect](/library_progect.png "library_progect")

#### ����MSDK��
��Eclipse������MSDKLibrary��Ŀ, �һ���Ϸ��Ŀ->����->Android->���(��)->ѡ��MSDKLibrary, ��ɶ�MSDKLibrary������. ����ʹ��Android Library Project����Ϸ, ��Ҫ����libs, res����Ŀ¼����Ϸ������ӦĿ¼.
	����MSDKLibrary/jniĿ¼�µ�jni��.cpp��.h�ļ��ӵ���Ϸ����, ����ӵ�makefile.
	ע������:
	����MSDKLibrary�Ժ���뷢������ͻ(�ظ�), ��ΪMSDK�����Ѿ������� ΢��SDK(libammsdk.jar), QQ����sdk(open_sdk.jar), MTA(mta-xxx.jar), ����SDK(beacon-xxx.jar), ������sdk�����������ȶ���, ��Ϸ�����ǰ�е���������ЩSDK, ��ɾ��֮ǰ���ɵ�jar��.

#### ����˵��
a)Ȩ������

	<!-- TODO SDK�������Ȩ��ģ�� START -->
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
	<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
	<uses-permission android:name="android.permission.GET_TASKS" />
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
	<uses-permission android:name="android.permission.READ_PHONE_STATE" />
	<uses-permission android:name="android.permission.RESTART_PACKAGES" />
	<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

	<!-- ��¼�ϱ�ʱ��Ҫ���豸����, ͨ������ģ������ȡ�豸���� -->
	<uses-permission android:name="android.permission.BLUETOOTH" />
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />

	<!-- ��ѡ��Ȩ�ޣ��쳣�ϱ�ϵͳlog,XGҲ��Ҫ -->
	<uses-permission android:name="android.permission.READ_LOGS" />

	<!-- ���α� permission start -->
	<uses-permission android:name="android.permission.CHANGE_CONFIGURATION" />
	<uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES" />
	<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
	<uses-permission android:name="android.permission.RECORD_AUDIO" />
	<uses-permission android:name="android.permission.VIBRATE" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
	<uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
	<!-- ���α� permission end -->

	<!-- �����Ÿ���Ҫ������Ȩ�� -->
	<uses-permission android:name="android.permission.BROADCAST_STICKY" />
	<uses-permission android:name="android.permission.WRITE_SETTINGS" />
	<uses-permission android:name="android.permission.RECEIVE_USER_PRESENT" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
	<uses-permission android:name="android.permission.VIBRATE" />

	<!-- TODO SDK���� ����Ȩ��ģ�� END -->

��assets/msdkconfig.ini�����÷��ʵ���Ҫ���ʵ�MSDK��̨����(���Ի���/��ʽ����), �����׶���Ϸ��Ҫʹ�ò��Ի���, ����ʱ��Ҫ�л�����ʽ����(��˾�ڲ�, ����, ��ʽ����), �л�����ʱ����Ҫע��, ��̨ҲҪͬʱ�л��ɶ�Ӧ�Ļ���. ����������ǰ�˷���MSDK��̨���Ի���. 

	;������Ϸ��Ʒ������Ϸ��ʹ�ô�������, ʹ������һ������
	;��msdktestΪ���Ի�������, msdkΪ��ʽ��������
	;MSDK_URL=http://msdk.qq.com
	;MSDK_URL=http://msdkdev.qq.com
	MSDK_URL=http://msdktest.qq.com
	
PS: Ϊ�˷�ֹ��Ϸ�ò��Ի�������, SDK�ڼ�⵽��Ϸʹ�ò��Ի������߿�������ʱ, ��Toast������: ��You are using http://msdktest.qq.com�� ��������ʾ, ��Ϸ�л�����ʽ���������Ժ����ʾ�Զ���ʧ.

#### Java���ʼ��
	public void onCreate(Bundle savedInstanceState) {
	...
	//��Ϸ����ʹ���Լ���QQ AppId����
		baseInfo.qqAppId = "1007033***";
		baseInfo.qqAppKey = "4578e54fb3a1bd18e0681bc1c7345***";

	//��Ϸ����ʹ���Լ���΢��AppId����
		baseInfo.wxAppId = "wxcde873f99466f***"; 
		baseInfo.wxAppKey = "bc0994f30c0a12a9908e353cf05d4***";

	//��Ϸ����ʹ���Լ���֧��offerId����
		baseInfo.offerId = "100703***";
	WGPlatform.Initialized(this, baseInfo);
	// ���뱣֤handleCallback��Initialized֮��
	WGPlatform.handleCallback(getIntent());
	...
	}
	@Override
	protected void onNewIntent(Intent intent) {
	super.onNewIntent(intent);
	// �����ⲿ������Ϸ, ����Ȩ���ص�����
	WGPlatform.handleCallback(intent); 
	}

	// Ϊ�˱�֤��¼�����ϱ�׼ȷ, ��Ϸ�����������������
	@Override
	protected void onPause() {
	    super.onPause();
	    WGPlatform.onPause();
	}
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

���⣬��Ҫ��MainActivty�м��ر�Ҫ�Ķ�̬�⣬ʾ���������£�

	// TODO GAME Ҫ���ر�Ҫ�Ķ�̬��
	static {
	// ��Ϸ��Ҫ���ش˶�̬��, �����ϱ���
	System.loadLibrary("NativeRQD"); 
	// ��Ϸ����Ҫ���, ����MSDKSample���õ�
	System.loadLibrary("WeGameSample");
	}

#### C++ ���ʼ��(ֻʹ��Java API����Ϸ����������˶�)
����WG��ͷ�Ľӿھ��ṩ��C++���Java��ӿ�. Java��ͨ��WGPlatform����, C++��ͨ��WGPlatform::GetInstance()����. ������÷�ʽ���Բο�jni/PlatformTest.cpp. SDK��Ҫ����һ��ȫ�ֵĻص�, ����Ȩ����, ������ʱ���������Ӧ�Ļص�. ���õĻص�����ʵ����WGPlatformObserver, �ڳ�ʼ��ʱ����˶���. ʾ����������:

	GlobalCallback g_Test;
	JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
	    WGPlatform::GetInstance()->init(vm);// C++���ʼ��SDK
	    WGPlatform::GetInstance()->WGSetObserver(&g_Test);
	    return JNI_VERSION_1_4;
	}
   
#### ����ȫ�ֻص�
MSDKͨ��WGPlatformObserver�������еķ�������Ȩ��������ѯ����ص�����Ϸ����Ϸ���ݻص��������UI�ȡ�ֻ�����ûص�����Ϸ�����յ�MSDK����Ӧ��
����Java �ص���

	WGPlatform.WGSetObserver(new WGPlatformObserver() {
		@Override
		public void OnWakeupNotify(WakeupRet ret) { }
		@Override
		public void OnShareNotify(ShareRet ret) { }
		@Override
		public void OnRelationNotify(RelationRet relationRet) { }
		@Override
		public void OnLoginNotify(LoginRet ret) { }
	});

����C++ �ص�(������Java��ص������ȵ���Java��ص�, ���Ҫʹ��C++��ص���������Java��ص�)��

	class GlobalCallback: public WGPlatformObserver {
	public:
	    virtual void OnLoginNotify(LoginRet& loginRet) { }
	    virtual void OnShareNotify(ShareRet& shareRet) { }
	    virtual void OnWakeupNotify(WakeupRet& wakeupRet) { }
	    virtual void OnRelationNotify(RelationRet& relationRet) { }
	    virtual ~GlobalCallback() { }
	};
	GlobalCallback g_Test;

	JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
		// C++���ʼ��, ��������Ϸ��Activity��onCreate֮ǰ������
		WGPlatform::GetInstance()->init(vm);
		WGPlatform::GetInstance()->WGSetObserver(&g_Test);
		return JNI_VERSION_1_4;
	}

ע: �����Ϸʹ��C++ API, ��Ҫ������Java���ȫ�ֻص�, SDK�����ȵ���Java��Ļص�, ��Java��ص�������C++��ص�.
	
����, ��Ϸ���Կ�ʼ����MSDK API�ĵ����ᵽ������API.