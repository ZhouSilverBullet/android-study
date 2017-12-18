1.spareArray可以解决相同类型但是不能同时控制的时候使用，用spareArray来分别存储这些情况，然后对应的取出来，放回去。通常可以在想到使用map的时候使用。

2.context == null ，一般在写Sdk的时候要进行检查

3.SdkSession，中的静态资源需要释放
   
     public static void clear() {
         synchronized(SdkSession.class) {
		   sdkSession = null;
         }
         sCtx = null; //静态资源释放
         sPlatformId = 0; //静态资源释放
     }
 在进行初始化的时候进行调用clear方法就可以把上次的静态的SdkSession的静态进行释放，从而是新的SdkSession的对象
   
     public static void init(Context context, int id) {
         clear();
         sCtx = context;
		 sPlatformId = id;

		 sdkSession = new SdkSession();
     }
4.readMetaData(context)方法的实现(一般SDK的在manifestest定义的都用这个方法去获得app_key和app_id)
 
     //这个方法一般用一次，所以一般在那个类里面私有化就OK
     private static void readMeta(Context context) {
		 int id = 0 ;
		 String key = null;
         try {
			ApplicationInfo info = context.getPackageManager().getApplicationInfo
            (context.getPackageName(), PackageManager.GET_META_DATA);
			id = Integer.valueOf(info.metaData.get("MWAPP_ID"))
            key = String.valueOf(info.metaData.get("MWAPP_KEY"));
         }catch(Exception e) {
         } 
		 if(TextUtils.isEmpty(key) || id == 0) {
			throws new RuntimeException("不能为空");
         }
         
	     setAppId(id);
		 setAppKey(key);
     }
5.android中的定时任务(AlarmManager)。AlarmManager是一个系统级的，不能直接new，可以通过context.getSystemService()来进行获取。

注意：一般跟一个广播接收器一起使用，然后广播再进行注册注册定时任务，当广播注册后然后再进行定时任务注册，在广播取消注册时，进行定时任务的取消，这样防止下次启动可能还能接收到上次注册的定时任务

    //注册定时任务并进行启动
	private static void setAlarm(Context context) {
		 AlarmManager alarm = context.getSystemService(Context.ALARM_SEVICE);
		 Intent intent = new Intent("自定义action");
		 PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 20, intent, PendingIntent.FLAG_CANCEL_CURRENT);
		 alarm.set(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() * count, pendingIntent);	 
    }

    //进行注册一个Broadcast来进行接收AlarmManager的action
    private static void registerAlarm(Context context) {
         IntentFilter filter = new IntentFilter();
         //监听自定义的action
         filter.addAction("自定义action"); 
         filter.addAciton(ConnectivityManager.CONNECTIVITY_ACTION);
         //注册broadcast
         context.registReceiver(RefreshHelper.getInstance, filter);
    }
6.定时任务定时去扫描本地缓存。</br>

    在定时任务扫描时要注意一些细节问题，比如，在初始化和注册广播的
    时候都可以去读缓存，这时目的都是进行缓存读取的，**一次就
    够了**，所以直接用个时间间隔来进行处理，**在5分钟内访问的都是无效的，不进行缓存读取**。如果访问读写操作的时候，都是比必须要
	的，可能操作同一个文件，这样就要考虑线程同步问题，分别用同一把
	锁进行，然后只要有一个线程在操作，其他线程是无法进入的，这样达
    到同步的作用。【同步建议在子线程中完成，因为会产生堵塞状态】

7.DeviceUtil的设计
	
    public static String getDeviceNo(Context context) {
		context.getSystemService();
    }
    TelephonyManager tm = (TelephonyManager) this.getSystemService(TELEPHONY_SERVICE);