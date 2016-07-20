HOOK系统服务的机制 <=> Binder Hook;

系统的各个远程 service 对象都是以 Binder 的形式存在的, 而这些 Binder 有一个管理者,
那就是 ServiceManager.

// 应用层获取服务对象;
	// public static final String ACTIVITY_SERVICE = "activity"; (Context.java)
	ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);

// 从 SYSTEM_SERVICE_FETCHERS 里面获取服务对象;
	@Override
    public Object ContextImpl::getSystemService(String name) {
        return SystemServiceRegistry.getSystemService(this, name);
    }

    /**
     * Gets a system service from a given context.
     */
    public static Object SystemServiceRegistry::getSystemService(ContextImpl ctx, String name) {
        ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
        return fetcher != null ? fetcher.getService(ctx) : null;
    }

// SYSTEM_SERVICE_FETCHERS 对象的初始化追踪;
// SystemServiceRegistry.java 中注册的 Service:
static {
        registerService(Context.ACCESSIBILITY_SERVICE, AccessibilityManager.class,
                new CachedServiceFetcher<AccessibilityManager>() {
            @Override
            public AccessibilityManager createService(ContextImpl ctx) {
                return AccessibilityManager.getInstance(ctx);
            }});

        registerService(Context.CAPTIONING_SERVICE, CaptioningManager.class,
                new CachedServiceFetcher<CaptioningManager>() {
            @Override
            public CaptioningManager createService(ContextImpl ctx) {
                return new CaptioningManager(ctx);
            }});

        registerService(Context.ACCOUNT_SERVICE, AccountManager.class,
                new CachedServiceFetcher<AccountManager>() {
            @Override
            public AccountManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.ACCOUNT_SERVICE);
                IAccountManager service = IAccountManager.Stub.asInterface(b);
                return new AccountManager(ctx, service);
            }});

        registerService(Context.ACTIVITY_SERVICE, ActivityManager.class,
                new CachedServiceFetcher<ActivityManager>() {
            @Override
            public ActivityManager createService(ContextImpl ctx) {
                return new ActivityManager(ctx.getOuterContext(), ctx.mMainThread.getHandler());
            }});

        registerService(Context.ALARM_SERVICE, AlarmManager.class,
                new CachedServiceFetcher<AlarmManager>() {
            @Override
            public AlarmManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.ALARM_SERVICE);
                IAlarmManager service = IAlarmManager.Stub.asInterface(b);
                return new AlarmManager(service, ctx);
            }});

        registerService(Context.AUDIO_SERVICE, AudioManager.class,
                new CachedServiceFetcher<AudioManager>() {
            @Override
            public AudioManager createService(ContextImpl ctx) {
                return new AudioManager(ctx);
            }});

        registerService(Context.MEDIA_ROUTER_SERVICE, MediaRouter.class,
                new CachedServiceFetcher<MediaRouter>() {
            @Override
            public MediaRouter createService(ContextImpl ctx) {
                return new MediaRouter(ctx);
            }});

        registerService(Context.BLUETOOTH_SERVICE, BluetoothManager.class,
                new CachedServiceFetcher<BluetoothManager>() {
            @Override
            public BluetoothManager createService(ContextImpl ctx) {
                return new BluetoothManager(ctx);
            }});

        registerService(Context.HDMI_CONTROL_SERVICE, HdmiControlManager.class,
                new StaticServiceFetcher<HdmiControlManager>() {
            @Override
            public HdmiControlManager createService() {
                IBinder b = ServiceManager.getService(Context.HDMI_CONTROL_SERVICE);
                return new HdmiControlManager(IHdmiControlService.Stub.asInterface(b));
            }});

        registerService(Context.CLIPBOARD_SERVICE, ClipboardManager.class,
                new CachedServiceFetcher<ClipboardManager>() {
            @Override
            public ClipboardManager createService(ContextImpl ctx) {
                return new ClipboardManager(ctx.getOuterContext(),
                        ctx.mMainThread.getHandler());
            }});

        // The clipboard service moved to a new package.  If someone asks for the old
        // interface by class then we want to redirect over to the new interface instead
        // (which extends it).
        SYSTEM_SERVICE_NAMES.put(android.text.ClipboardManager.class, Context.CLIPBOARD_SERVICE);

        registerService(Context.CONNECTIVITY_SERVICE, ConnectivityManager.class,
                new StaticOuterContextServiceFetcher<ConnectivityManager>() {
            @Override
            public ConnectivityManager createService(Context context) {
                IBinder b = ServiceManager.getService(Context.CONNECTIVITY_SERVICE);
                IConnectivityManager service = IConnectivityManager.Stub.asInterface(b);
                return new ConnectivityManager(context, service);
            }});

        registerService(Context.COUNTRY_DETECTOR, CountryDetector.class,
                new StaticServiceFetcher<CountryDetector>() {
            @Override
            public CountryDetector createService() {
                IBinder b = ServiceManager.getService(Context.COUNTRY_DETECTOR);
                return new CountryDetector(ICountryDetector.Stub.asInterface(b));
            }});

        registerService(Context.DEVICE_POLICY_SERVICE, DevicePolicyManager.class,
                new CachedServiceFetcher<DevicePolicyManager>() {
            @Override
            public DevicePolicyManager createService(ContextImpl ctx) {
                return DevicePolicyManager.create(ctx, ctx.mMainThread.getHandler());
            }});

        registerService(Context.DOWNLOAD_SERVICE, DownloadManager.class,
                new CachedServiceFetcher<DownloadManager>() {
            @Override
            public DownloadManager createService(ContextImpl ctx) {
                return new DownloadManager(ctx.getContentResolver(), ctx.getPackageName());
            }});

        registerService(Context.BATTERY_SERVICE, BatteryManager.class,
                new StaticServiceFetcher<BatteryManager>() {
            @Override
            public BatteryManager createService() {
                return new BatteryManager();
            }});

        registerService(Context.NFC_SERVICE, NfcManager.class,
                new CachedServiceFetcher<NfcManager>() {
            @Override
            public NfcManager createService(ContextImpl ctx) {
                return new NfcManager(ctx);
            }});

        registerService(Context.DROPBOX_SERVICE, DropBoxManager.class,
                new StaticServiceFetcher<DropBoxManager>() {
            @Override
            public DropBoxManager createService() {
                IBinder b = ServiceManager.getService(Context.DROPBOX_SERVICE);
                IDropBoxManagerService service = IDropBoxManagerService.Stub.asInterface(b);
                if (service == null) {
                    // Don't return a DropBoxManager that will NPE upon use.
                    // This also avoids caching a broken DropBoxManager in
                    // getDropBoxManager during early boot, before the
                    // DROPBOX_SERVICE is registered.
                    return null;
                }
                return new DropBoxManager(service);
            }});

        registerService(Context.INPUT_SERVICE, InputManager.class,
                new StaticServiceFetcher<InputManager>() {
            @Override
            public InputManager createService() {
                return InputManager.getInstance();
            }});

        registerService(Context.DISPLAY_SERVICE, DisplayManager.class,
                new CachedServiceFetcher<DisplayManager>() {
            @Override
            public DisplayManager createService(ContextImpl ctx) {
                return new DisplayManager(ctx.getOuterContext());
            }});

        registerService(Context.INPUT_METHOD_SERVICE, InputMethodManager.class,
                new StaticServiceFetcher<InputMethodManager>() {
            @Override
            public InputMethodManager createService() {
                return InputMethodManager.getInstance();
            }});

        registerService(Context.TEXT_SERVICES_MANAGER_SERVICE, TextServicesManager.class,
                new StaticServiceFetcher<TextServicesManager>() {
            @Override
            public TextServicesManager createService() {
                return TextServicesManager.getInstance();
            }});

        registerService(Context.KEYGUARD_SERVICE, KeyguardManager.class,
                new StaticServiceFetcher<KeyguardManager>() {
            @Override
            public KeyguardManager createService() {
                return new KeyguardManager();
            }});

        registerService(Context.LAYOUT_INFLATER_SERVICE, LayoutInflater.class,
                new CachedServiceFetcher<LayoutInflater>() {
            @Override
            public LayoutInflater createService(ContextImpl ctx) {
                return new PhoneLayoutInflater(ctx.getOuterContext());
            }});

        registerService(Context.LOCATION_SERVICE, LocationManager.class,
                new CachedServiceFetcher<LocationManager>() {
            @Override
            public LocationManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.LOCATION_SERVICE);
                return new LocationManager(ctx, ILocationManager.Stub.asInterface(b));
            }});

        registerService(Context.NETWORK_POLICY_SERVICE, NetworkPolicyManager.class,
                new CachedServiceFetcher<NetworkPolicyManager>() {
            @Override
            public NetworkPolicyManager createService(ContextImpl ctx) {
                return new NetworkPolicyManager(ctx, INetworkPolicyManager.Stub.asInterface(
                        ServiceManager.getService(Context.NETWORK_POLICY_SERVICE)));
            }});

        registerService(Context.NOTIFICATION_SERVICE, NotificationManager.class,
                new CachedServiceFetcher<NotificationManager>() {
            @Override
            public NotificationManager createService(ContextImpl ctx) {
                final Context outerContext = ctx.getOuterContext();
                return new NotificationManager(
                    new ContextThemeWrapper(outerContext,
                            Resources.selectSystemTheme(0,
                                    outerContext.getApplicationInfo().targetSdkVersion,
                                    com.android.internal.R.style.Theme_Dialog,
                                    com.android.internal.R.style.Theme_Holo_Dialog,
                                    com.android.internal.R.style.Theme_DeviceDefault_Dialog,
                                    com.android.internal.R.style.Theme_DeviceDefault_Light_Dialog)),
                    ctx.mMainThread.getHandler());
            }});

        registerService(Context.NSD_SERVICE, NsdManager.class,
                new CachedServiceFetcher<NsdManager>() {
            @Override
            public NsdManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.NSD_SERVICE);
                INsdManager service = INsdManager.Stub.asInterface(b);
                return new NsdManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.POWER_SERVICE, PowerManager.class,
                new CachedServiceFetcher<PowerManager>() {
            @Override
            public PowerManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.POWER_SERVICE);
                IPowerManager service = IPowerManager.Stub.asInterface(b);
                if (service == null) {
                    Log.wtf(TAG, "Failed to get power manager service.");
                }
                return new PowerManager(ctx.getOuterContext(),
                        service, ctx.mMainThread.getHandler());
            }});

        registerService(Context.SEARCH_SERVICE, SearchManager.class,
                new CachedServiceFetcher<SearchManager>() {
            @Override
            public SearchManager createService(ContextImpl ctx) {
                return new SearchManager(ctx.getOuterContext(),
                        ctx.mMainThread.getHandler());
            }});

        registerService(Context.SENSOR_SERVICE, SensorManager.class,
                new CachedServiceFetcher<SensorManager>() {
            @Override
            public SensorManager createService(ContextImpl ctx) {
                return new SystemSensorManager(ctx.getOuterContext(),
                  ctx.mMainThread.getHandler().getLooper());
            }});

        registerService(Context.STATUS_BAR_SERVICE, StatusBarManager.class,
                new CachedServiceFetcher<StatusBarManager>() {
            @Override
            public StatusBarManager createService(ContextImpl ctx) {
                return new StatusBarManager(ctx.getOuterContext());
            }});

        registerService(Context.STORAGE_SERVICE, StorageManager.class,
                new CachedServiceFetcher<StorageManager>() {
            @Override
            public StorageManager createService(ContextImpl ctx) {
                return new StorageManager(ctx, ctx.mMainThread.getHandler().getLooper());
            }});

        registerService(Context.TELEPHONY_SERVICE, TelephonyManager.class,
                new CachedServiceFetcher<TelephonyManager>() {
            @Override
            public TelephonyManager createService(ContextImpl ctx) {
                return new TelephonyManager(ctx.getOuterContext());
            }});

        registerService(Context.TELEPHONY_SUBSCRIPTION_SERVICE, SubscriptionManager.class,
                new CachedServiceFetcher<SubscriptionManager>() {
            @Override
            public SubscriptionManager createService(ContextImpl ctx) {
                return new SubscriptionManager(ctx.getOuterContext());
            }});

        registerService(Context.CARRIER_CONFIG_SERVICE, CarrierConfigManager.class,
                new CachedServiceFetcher<CarrierConfigManager>() {
            @Override
            public CarrierConfigManager createService(ContextImpl ctx) {
                return new CarrierConfigManager();
            }});

        registerService(Context.TELECOM_SERVICE, TelecomManager.class,
                new CachedServiceFetcher<TelecomManager>() {
            @Override
            public TelecomManager createService(ContextImpl ctx) {
                return new TelecomManager(ctx.getOuterContext());
            }});

        registerService(Context.UI_MODE_SERVICE, UiModeManager.class,
                new CachedServiceFetcher<UiModeManager>() {
            @Override
            public UiModeManager createService(ContextImpl ctx) {
                return new UiModeManager();
            }});

        registerService(Context.USB_SERVICE, UsbManager.class,
                new CachedServiceFetcher<UsbManager>() {
            @Override
            public UsbManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.USB_SERVICE);
                return new UsbManager(ctx, IUsbManager.Stub.asInterface(b));
            }});

        registerService(Context.SERIAL_SERVICE, SerialManager.class,
                new CachedServiceFetcher<SerialManager>() {
            @Override
            public SerialManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.SERIAL_SERVICE);
                return new SerialManager(ctx, ISerialManager.Stub.asInterface(b));
            }});

        registerService(Context.VIBRATOR_SERVICE, Vibrator.class,
                new CachedServiceFetcher<Vibrator>() {
            @Override
            public Vibrator createService(ContextImpl ctx) {
                return new SystemVibrator(ctx);
            }});

        registerService(Context.WALLPAPER_SERVICE, WallpaperManager.class,
                new CachedServiceFetcher<WallpaperManager>() {
            @Override
            public WallpaperManager createService(ContextImpl ctx) {
                return new WallpaperManager(ctx.getOuterContext(),
                        ctx.mMainThread.getHandler());
            }});

        registerService(Context.WIFI_SERVICE, WifiManager.class,
                new CachedServiceFetcher<WifiManager>() {
            @Override
            public WifiManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.WIFI_SERVICE);
                IWifiManager service = IWifiManager.Stub.asInterface(b);
                return new WifiManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.WIFI_PASSPOINT_SERVICE, WifiPasspointManager.class,
                new CachedServiceFetcher<WifiPasspointManager>() {
            @Override
            public WifiPasspointManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.WIFI_PASSPOINT_SERVICE);
                IWifiPasspointManager service = IWifiPasspointManager.Stub.asInterface(b);
                return new WifiPasspointManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.WIFI_P2P_SERVICE, WifiP2pManager.class,
                new StaticServiceFetcher<WifiP2pManager>() {
            @Override
            public WifiP2pManager createService() {
                IBinder b = ServiceManager.getService(Context.WIFI_P2P_SERVICE);
                IWifiP2pManager service = IWifiP2pManager.Stub.asInterface(b);
                return new WifiP2pManager(service);
            }});

        registerService(Context.WIFI_SCANNING_SERVICE, WifiScanner.class,
                new CachedServiceFetcher<WifiScanner>() {
            @Override
            public WifiScanner createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.WIFI_SCANNING_SERVICE);
                IWifiScanner service = IWifiScanner.Stub.asInterface(b);
                return new WifiScanner(ctx.getOuterContext(), service);
            }});

        registerService(Context.WIFI_RTT_SERVICE, RttManager.class,
                new CachedServiceFetcher<RttManager>() {
            @Override
            public RttManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.WIFI_RTT_SERVICE);
                IRttManager service = IRttManager.Stub.asInterface(b);
                return new RttManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.ETHERNET_SERVICE, EthernetManager.class,
                new CachedServiceFetcher<EthernetManager>() {
            @Override
            public EthernetManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.ETHERNET_SERVICE);
                IEthernetManager service = IEthernetManager.Stub.asInterface(b);
                return new EthernetManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.WINDOW_SERVICE, WindowManager.class,
                new CachedServiceFetcher<WindowManager>() {
            @Override
            public WindowManager createService(ContextImpl ctx) {
                return new WindowManagerImpl(ctx.getDisplay());
            }});

        registerService(Context.USER_SERVICE, UserManager.class,
                new CachedServiceFetcher<UserManager>() {
            @Override
            public UserManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.USER_SERVICE);
                IUserManager service = IUserManager.Stub.asInterface(b);
                return new UserManager(ctx, service);
            }});

        registerService(Context.APP_OPS_SERVICE, AppOpsManager.class,
                new CachedServiceFetcher<AppOpsManager>() {
            @Override
            public AppOpsManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.APP_OPS_SERVICE);
                IAppOpsService service = IAppOpsService.Stub.asInterface(b);
                return new AppOpsManager(ctx, service);
            }});

        registerService(Context.CAMERA_SERVICE, CameraManager.class,
                new CachedServiceFetcher<CameraManager>() {
            @Override
            public CameraManager createService(ContextImpl ctx) {
                return new CameraManager(ctx);
            }});

        registerService(Context.LAUNCHER_APPS_SERVICE, LauncherApps.class,
                new CachedServiceFetcher<LauncherApps>() {
            @Override
            public LauncherApps createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.LAUNCHER_APPS_SERVICE);
                ILauncherApps service = ILauncherApps.Stub.asInterface(b);
                return new LauncherApps(ctx, service);
            }});

        registerService(Context.RESTRICTIONS_SERVICE, RestrictionsManager.class,
                new CachedServiceFetcher<RestrictionsManager>() {
            @Override
            public RestrictionsManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.RESTRICTIONS_SERVICE);
                IRestrictionsManager service = IRestrictionsManager.Stub.asInterface(b);
                return new RestrictionsManager(ctx, service);
            }});

        registerService(Context.PRINT_SERVICE, PrintManager.class,
                new CachedServiceFetcher<PrintManager>() {
            @Override
            public PrintManager createService(ContextImpl ctx) {
                IBinder iBinder = ServiceManager.getService(Context.PRINT_SERVICE);
                IPrintManager service = IPrintManager.Stub.asInterface(iBinder);
                return new PrintManager(ctx.getOuterContext(), service, UserHandle.myUserId(),
                        UserHandle.getAppId(Process.myUid()));
            }});

        registerService(Context.CONSUMER_IR_SERVICE, ConsumerIrManager.class,
                new CachedServiceFetcher<ConsumerIrManager>() {
            @Override
            public ConsumerIrManager createService(ContextImpl ctx) {
                return new ConsumerIrManager(ctx);
            }});

        registerService(Context.MEDIA_SESSION_SERVICE, MediaSessionManager.class,
                new CachedServiceFetcher<MediaSessionManager>() {
            @Override
            public MediaSessionManager createService(ContextImpl ctx) {
                return new MediaSessionManager(ctx);
            }});

        registerService(Context.TRUST_SERVICE, TrustManager.class,
                new StaticServiceFetcher<TrustManager>() {
            @Override
            public TrustManager createService() {
                IBinder b = ServiceManager.getService(Context.TRUST_SERVICE);
                return new TrustManager(b);
            }});

        registerService(Context.FINGERPRINT_SERVICE, FingerprintManager.class,
                new CachedServiceFetcher<FingerprintManager>() {
            @Override
            public FingerprintManager createService(ContextImpl ctx) {
                IBinder binder = ServiceManager.getService(Context.FINGERPRINT_SERVICE);
                IFingerprintService service = IFingerprintService.Stub.asInterface(binder);
                return new FingerprintManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.TV_INPUT_SERVICE, TvInputManager.class,
                new StaticServiceFetcher<TvInputManager>() {
            @Override
            public TvInputManager createService() {
                IBinder iBinder = ServiceManager.getService(Context.TV_INPUT_SERVICE);
                ITvInputManager service = ITvInputManager.Stub.asInterface(iBinder);
                return new TvInputManager(service, UserHandle.myUserId());
            }});

        registerService(Context.NETWORK_SCORE_SERVICE, NetworkScoreManager.class,
                new CachedServiceFetcher<NetworkScoreManager>() {
            @Override
            public NetworkScoreManager createService(ContextImpl ctx) {
                return new NetworkScoreManager(ctx);
            }});

        registerService(Context.USAGE_STATS_SERVICE, UsageStatsManager.class,
                new CachedServiceFetcher<UsageStatsManager>() {
            @Override
            public UsageStatsManager createService(ContextImpl ctx) {
                IBinder iBinder = ServiceManager.getService(Context.USAGE_STATS_SERVICE);
                IUsageStatsManager service = IUsageStatsManager.Stub.asInterface(iBinder);
                return new UsageStatsManager(ctx.getOuterContext(), service);
            }});

        registerService(Context.NETWORK_STATS_SERVICE, NetworkStatsManager.class,
                new CachedServiceFetcher<NetworkStatsManager>() {
            @Override
            public NetworkStatsManager createService(ContextImpl ctx) {
                return new NetworkStatsManager(ctx.getOuterContext());
            }});

        registerService(Context.JOB_SCHEDULER_SERVICE, JobScheduler.class,
                new StaticServiceFetcher<JobScheduler>() {
            @Override
            public JobScheduler createService() {
                IBinder b = ServiceManager.getService(Context.JOB_SCHEDULER_SERVICE);
                return new JobSchedulerImpl(IJobScheduler.Stub.asInterface(b));
            }});

        registerService(Context.PERSISTENT_DATA_BLOCK_SERVICE, PersistentDataBlockManager.class,
                new StaticServiceFetcher<PersistentDataBlockManager>() {
            @Override
            public PersistentDataBlockManager createService() {
                IBinder b = ServiceManager.getService(Context.PERSISTENT_DATA_BLOCK_SERVICE);
                IPersistentDataBlockService persistentDataBlockService =
                        IPersistentDataBlockService.Stub.asInterface(b);
                if (persistentDataBlockService != null) {
                    return new PersistentDataBlockManager(persistentDataBlockService);
                } else {
                    // not supported
                    return null;
                }
            }});

        registerService(Context.MEDIA_PROJECTION_SERVICE, MediaProjectionManager.class,
                new CachedServiceFetcher<MediaProjectionManager>() {
            @Override
            public MediaProjectionManager createService(ContextImpl ctx) {
                return new MediaProjectionManager(ctx);
            }});

        registerService(Context.APPWIDGET_SERVICE, AppWidgetManager.class,
                new CachedServiceFetcher<AppWidgetManager>() {
            @Override
            public AppWidgetManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.APPWIDGET_SERVICE);
                return new AppWidgetManager(ctx, IAppWidgetService.Stub.asInterface(b));
            }});

        registerService(Context.MIDI_SERVICE, MidiManager.class,
                new CachedServiceFetcher<MidiManager>() {
            @Override
            public MidiManager createService(ContextImpl ctx) {
                IBinder b = ServiceManager.getService(Context.MIDI_SERVICE);
                if (b == null) {
                    return null;
                }
                return new MidiManager(IMidiManager.Stub.asInterface(b));
            }});

        registerService(Context.RADIO_SERVICE, RadioManager.class,
                new CachedServiceFetcher<RadioManager>() {
            @Override
            public RadioManager createService(ContextImpl ctx) {
                return new RadioManager(ctx);
            }});
    }

    ActivityManager.java 中的核心操作都是 ActivityManagerNative 完成的, 在 ActivityManagerNative.java 中:
    private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity");
            if (false) {
                Log.v("ActivityManager", "default service binder = " + b);
            }
            IActivityManager am = asInterface(b);
            if (false) {
                Log.v("ActivityManager", "default service = " + am);
            }
            return am;
        }
    };

    /////////////////////////////////////////////////////////////////////////////
    通过上面的分析, 系统 Service 的使用分为两步:
    1. IBinder b = ServiceManager.getService("service_name"); // 获取原始的IBinder对象
    2. IXXInterface in = IXXInterface.Stub.asInterface(b); // 转换为Service接口


[寻找HOOK点]
由于系统服务的使用者都是对第二步获取到的 IXXInterface 进行操作, 因此如果我们要 Hook 掉某个系统服务,
只需要把第二步的 asInterface 方法返回的对象修改为我们 HOOK 过的对象就可以了.

[asInterface过程]
想办法把这个方法的返回值修改为我们 HOOK 过的系统服务对象.
以剪切板为例:
源码位置: frameworks/base/core/java/android/content/IClipboard.aidl
aidl 会编译生成 java 代码如下:

public static android.content.IClipboard asInterface(android.os.IBinder obj) {
    if ((obj == null)) {
        return null; 
    }
    android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR); // Hook点
    if (((iin != null) && (iin instanceof android.content.IClipboard))) {
        return ((android.content.IClipboard) iin);
    }
    return new android.content.IClipboard.Stub.Proxy(obj);
}

我们要尝试修改这个 obj 对象的 queryLocalInterface 方法的返回值, 并保证这个返回值符合接下来的 if 条件检测,
那么就达到了修改 asInterface 方法返回值的目的.

	// ClipboardManager.java
    static private IClipboard ClipboardManager::getService() {
        synchronized (sStaticLock) {
            if (sService != null) {
                return sService;
            }
            IBinder b = ServiceManager.getService("clipboard");
            sService = IClipboard.Stub.asInterface(b);
            return sService;
        }
    }
我们看看这个函数, IBinder 对象的 queryLocalInterface 方法的参数,来自 
IBinder b = ServiceManager.getService("clipboard");
因此,我们希望修改这个 getService 方法的返回值, 让这个方法返回一个我们伪造过的 IBinder 对象,
进而使得 asInterface 方法返回在 queryLocalInterface 方法里面处理过的值,最终实现 hook 系统服务的目的.

系统服务是一系列的远程Service, 它们的本体, 也就是 Binder 本地对象一般都存在于某个单独的进程,在这个进程之外的其它进程
存在的都是这些 Binder 本地对象的代理.
(此处如果不懂,请先看我的另一个分支代码: https://github.com/pinggle/Fake_AIDL )


接下来,我们看看 ServiceManager.getService 函数的实现:
    /**
     * Returns a reference to a service with the given name.
     * 
     * @param name the name of the service to get
     * @return a reference to the service, or <code>null</code> if the service doesn't exist
     */
    public static IBinder ServiceManager::getService(String name) {
        try {
            IBinder service = sCache.get(name);
            if (service != null) {
                return service;
            } else {
                return getIServiceManager().getService(name);
            }
        } catch (RemoteException e) {
            Log.e(TAG, "error in getService", e);
        }
        return null;
    } 
ServiceManager 为了避免每次都进行跨进程通信, 把这些 Binder 代理对象缓存在一张 map (sCache) 里面;
private static HashMap<String, IBinder> sCache = new HashMap<String, IBinder>();

我们可以替换这个 sCache 里面指定位置的指定内容, 将 IBinder 替换为我们伪造过的 IBinder 即可.
由于系统在 getService 的时候每次都优先查找缓存,因此返回给使用者的都是被我们修改过的对象.


==============================================================================================

总结一下, 要达到修改系统服务的目的, 我们需要如下两步:
1.首先需要伪造一个系统服务对象, 接下来就要想办法让 asInterface 能够返回我们的这个伪造对象而不是原始的系统服务对象;
2.要让 getService 返回 IBinder 对象的 queryLocalInterface 方法直接返回我们伪造过的系统服务对象;
  所以, 我们需要伪造一个 IBinder 对象, 主要是修改它的 queryLocalInterface 方法, 让它返回我们伪造的系统服务对象;
  然后把这个伪造对象放置在 ServiceManager 的缓存 map 里面即可.

  由于我们接管了 asInterface 这个方法的全部, 我们伪造过的这个系统服务对象不能是只拥有本地 Binder 对象的能力,还要有
  Binder 代理对象操控驱动的能力;

==============================================================================================


HOOK 系统剪切版服务

1.伪造剪切版服务对象:
使用代理的方式 HOOK 掉以下两个函数:
// frameworks/base/core/java/android/content/IClipboard.aidl
ClipData getPrimaryClip(String pkg);
boolean hasPrimaryClip(String callingPackage);

public class BinderHookHandler implements InvocationHandler {

    private static final String TAG = "BinderHookHandler";

    // 原始的Service对象 (IInterface)
    Object base;

    public BinderHookHandler(IBinder base, Class<?> stubClass) {
        try {
            Method asInterfaceMethod = stubClass.getDeclaredMethod("asInterface", IBinder.class);
            // IClipboard.Stub.asInterface(base);
            this.base = asInterfaceMethod.invoke(null, base);
        } catch (Exception e) {
            throw new RuntimeException("hooked failed!");
        }
    }

    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        // 把剪切版的内容替换为 "you are hooked"
        if ("getPrimaryClip".equals(method.getName())) {
            Log.d(TAG, "hook getPrimaryClip");
            return ClipData.newPlainText(null, "you are hooked");
            //method.invoke(base, args);
        }

        // 欺骗系统,使之认为剪切版上一直有内容
        if ("hasPrimaryClip".equals(method.getName())) {
            return true;
        }

        return method.invoke(base, args);
    }
}

2.伪造 IBinder 对象;
再回顾一个获取服务对象的获取步骤:
IBinder b = ServiceManager.getService("clipboard");
sService = IClipboard.Stub.asInterface(b);
我们目前是要想办法 让 asInterface 方法返回我们伪造的对象;
那么我们需要伪造一个 IBinder, 并且 IBinder 支持 queryLocalInterface 返回我们伪造的 服务对象;

public class BinderProxyHookHandler implements InvocationHandler {

    private static final String TAG = "BinderProxyHookHandler";

    // 绝大部分情况下,这是一个BinderProxy对象
    // 只有当Service和我们在同一个进程的时候才是Binder本地对象
    // 这个基本不可能
    IBinder base;

    Class<?> stub;

    Class<?> iinterface;

    public BinderProxyHookHandler(IBinder base) {
        this.base = base;
        try {
            this.stub = Class.forName("android.content.IClipboard$Stub");
            this.iinterface = Class.forName("android.content.IClipboard");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        if ("queryLocalInterface".equals(method.getName())) {

            Log.d(TAG, "hook queryLocalInterface");

            // 这里直接返回真正被Hook掉的Service接口
            // 这里的 queryLocalInterface 就不是原本的意思了
            // 我们肯定不会真的返回一个本地接口, 因为我们接管了 asInterface方法的作用
            // 因此必须是一个完整的 asInterface 过的 IInterface对象, 既要处理本地对象,也要处理代理对象
            // 这只是一个Hook点而已, 它原始的含义已经被我们重定义了; 因为我们会永远确保这个方法不返回null
            // 让 IClipboard.Stub.asInterface 永远走到if语句的else分支里面
            return Proxy.newProxyInstance(proxy.getClass().getClassLoader(),

                    // asInterface 的时候会检测是否是特定类型的接口然后进行强制转换
                    // 因此这里的动态代理生成的类型信息的类型必须是正确的

                    // 这里面Hook的是一个BinderProxy对象(Binder代理) (代理Binder的queryLocalInterface正常情况下是返回null)
                    // 因此, 正常情况下 在asInterface里面会由于BinderProxy的queryLocalInterface返回null导致系统创建一个匿名的代理对象, 这样我们就无法控制了
                    // 所以我们要伪造一个对象, 瞒过这个if检测, 使得系统把这个queryLocalInterface返回的对象透传给asInterface的返回值;
                    // 检测有两个要求, 其一: 非空, 其二, IXXInterface类型。
                    // 所以, 其实返回的对象不需要是Binder对象, 我们把它当作普通的对象Hook掉就ok(拦截这个对象里面对于IXXInterface相关方法的调用)
                    // tks  jeremyhe_cn@qq.com
                    new Class[] { this.iinterface },
                    new BinderHookHandler(base, stub));
        }

        Log.d(TAG, "method:" + method.getName());
        return method.invoke(base, args);
    }
}

 // Hook 掉这个Binder代理对象的 queryLocalInterface 方法
// 然后在 queryLocalInterface 返回一个IInterface对象, hook掉我们感兴趣的方法即可.
IBinder hookedBinder = (IBinder) Proxy.newProxyInstance(serviceManager.getClassLoader(),
        new Class<?>[]{IBinder.class},
        new BinderProxyHookHandler(rawBinder));

我们伪造了一个跟原始 IBinder 一模一样的对象, 然后在这个伪造的 IBinder 对象的 queryLocalInterface 方法里面返回了我们
之前创建的 伪造过的系统服务对象;

3.替换 ServiceManager 的 IBinder 对象;
我们使用反射的方式修改 ServiceManager 类里面缓存的 Binder 对象, 使得 getService 方法返回我们伪造的 IBinder 对象,
进而 asInterface 方法使用伪造 IBinder 对象的 queryLocalInterface 方法返回了我们伪造的系统服务对象, 代码如下:

public static void hookClipboardService() throws Exception {

    final String CLIPBOARD_SERVICE = "clipboard";

    // 下面这一段的意思实际就是: ServiceManager.getService("clipboard");
    // 只不过 ServiceManager这个类是@hide的
    Class<?> serviceManager = Class.forName("android.os.ServiceManager");
    Method getService = serviceManager.getDeclaredMethod("getService", String.class);

    // ServiceManager里面管理的原始的Clipboard Binder对象
    // 一般来说这是一个Binder代理对象
    IBinder rawBinder = (IBinder) getService.invoke(null, CLIPBOARD_SERVICE);

    // Hook 掉这个Binder代理对象的 queryLocalInterface 方法
    // 然后在 queryLocalInterface 返回一个IInterface对象, hook掉我们感兴趣的方法即可.
    IBinder hookedBinder = (IBinder) Proxy.newProxyInstance(serviceManager.getClassLoader(),
            new Class<?>[]{IBinder.class},
            new BinderProxyHookHandler(rawBinder));

    // 把这个hook过的Binder代理对象放进ServiceManager的cache里面
    // 以后查询的时候 会优先查询缓存里面的Binder, 这样就会使用被我们修改过的Binder了
    Field cacheField = serviceManager.getDeclaredField("sCache");
    cacheField.setAccessible(true);
    Map<String, IBinder> cache = (Map) cacheField.get(null);
    cache.put(CLIPBOARD_SERVICE, hookedBinder);
}

接下来, 在 APP 里面使用剪切版, 剪切版的内容永远都是: you are hooked;

摘自: http://weishu.me/2016/02/16/understand-plugin-framework-binder-hook




