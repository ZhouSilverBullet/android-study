1.静态变量释放
	
	CacheRefreshHelper,的registerAlarm,由于RefreshHelper 是
    继承 BroadcastReceiver，且是静态方法，如果在下次进来时不先
    进行取消注册广播，然后释放资源，将自己置为空。然后再进行新的注册，这样才是初始化

    SdkSession,在进行初始化时调用了clear的这个方法
        public static void clear() {
		synchronized (SdkSession.class) {
			sdkSession = null;
		}

			sCtx = null;  //静态
			sPlatformId = 0; //静态
	    }   
  逻辑处理，抽取方法


    
    