# 1. AppLovin MAX Android SDK 연동 가이드

## 시작하기

### ID 발급

매니저로부터 다음을 발급 받으세요.

- _AppLovin SDK key_
- _AppLovin MAX ad unit ID_ (앱의 광고 형식마다 고유한 값)
- _AdMob app ID_ (앱마다 고유한 값)
- _PointBerry inventory ID_ (앱의 광고 단위마다 고유한 값)

### build.gradle

AppLovin MAX SDK와 [AdMob 어댑터](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/Google), [Unity Ads 어댑터](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/UnityAds), [Android Advertising ID](https://developers.google.com/android/guides/setup#list-dependencies), [PointBerry Event Tracker](https://github.com/connect-n/pointberry-event-tracker-android-guide)를 다운로드하세요.

```groovy
repositories {
    google()
    mavenCentral()
    maven { url 'https://jitpack.io' }
}

dependencies {
    // AppLovin MAX SDK
    implementation 'com.applovin:applovin-sdk:11.3.1'
    
    // Meta SDK
    implementation 'com.applovin.mediation:facebook-adapter:6.11.0.2'    
    implementation 'com.facebook.android:audience-network-sdk:6.11.0'

    // Vungle SDK
    implementation 'com.applovin.mediation:vungle-adapter:6.11.0.1'
    
    // AdMob-to-MAX adapter
    implementation 'com.applovin.mediation:google-adapter:20.6.0.3'

    // Unity-Ads-to-MAX adapter
    implementation 'com.applovin.mediation:unityads-adapter:4.3.0.0'

    // Android Advertising ID (AAID)
    implementation 'com.google.android.gms:play-services-ads-identifier:18.0.1'

    // PointBerry Event Tracker
    implementation 'com.github.connect-n:pointberry-event-tracker-android:1.0.3'
}
```

### AndroidManifest.xml

매니저로부터 발급 받은 *AppLovin SDK key*와 *AdMob app ID*를 추가하세요.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest … >
    <application … >
        <meta-data android:name="applovin.sdk.key"
           android:value="YOUR_APPLOVIN_SDK_KEY"/>
        <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="YOUR_ADMOB_APP_ID"/>
        ⋮
    </application>
</manifest>
```

### AppLovin MAX SDK 초기화

앱이 시작된 후 가능한 한 빨리 SDK를 초기화하세요.

```java
public class FirstActivity extends Activity {
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        AppLovinSdk.getInstance(this).setMediationProvider(AppLovinMediationProvider.MAX);
        AppLovinSdk.getInstance(this).initializeSdk(config -> {
            // SDK가 초기화됐습니다.
            // 이제 광고를 로드할 수 있습니다.
        });
    }
}
```

## 배너 광고

### 레이아웃 XML에 배너 정의

기본 배너 높이를 50dp로 설정하세요.

```xml
<!-- res/values/attrs.xml -->

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <dimen name="banner_height">50dp</dimen>
</resources>
```

태블릿 배너 높이를 90dp로 설정하세요.

```xml
<!-- res/values-sw600dp/attrs.xml -->

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <dimen name="banner_height">90dp</dimen>
</resources>
```

레이아웃 XML에 `MaxAdView`를 정의하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

```xml
<!-- res/layout/activity_banner_ad.xml -->

<com.applovin.mediation.ads.MaxAdView
    xmlns:maxads="http://schemas.applovin.com/android/1.0"
    maxads:adUnitId="YOUR_MAX_AD_UNIT_ID"
    android:background="@color/banner_background_color" <!-- Set background or background color for banners to be fully functional -->
    android:layout_width="match_parent"                 <!-- Stretch to the width of the screen for banners to be fully functional -->
    android:layout_height="@dimen/banner_height"
</com.applovin.mediation.ads.MaxAdView>
```

### 배너 광고 로드

`MaxAdView`의 인스턴스를 생성하세요.

`MaxAdView`의 `loadAd()`를 호출해서 광고를 로드하세요.

`MaxAdViewAdListener`를 구현해서 광고 이벤트를 수신하세요.

`onAdLoaded()` 콜백에서 `PointBerryImpressionTracker`의 `logImpression()`을 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```java
public class BannerAdActivity extends Activity implements MaxAdViewAdListener {

    private MaxAdView bannerAd;

    private PointBerryImpressionTracker impTracker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_banner_ad);

        bannerAd = findViewById(R.id.banner_ad);
        bannerAd.setListener(this);
        bannerAd.setPlacement("YOUR_POINTBERRY_INVENTORY_ID");

        // 첫 광고를 로드하세요.
        bannerAd.loadAd();

        impTracker = new PointBerryImpressionTracker(getApplicationContext());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        bannerAd.destroy();
    }

    // 광고 이벤트

    @Override
    public void onAdLoaded(final MaxAd maxAd) {
        // 광고가 로드됐습니다.

        // 광고 노출을 로깅하세요.
        impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID");

        // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
        // impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
    }

    @Override
    public void onAdLoadFailed(final String adUnitId, final MaxError error) {
        // 광고가 로드에 실패했습니다.
    }

    @Override
    public void onAdClicked(final MaxAd maxAd) {
        // 광고가 클릭됐습니다.
    }

    @Override
    public void onAdExpanded(MaxAd ad) {
    }

    @Override
    public void onAdCollapsed(MaxAd ad) {
    }

    @Override
    public void onAdDisplayed(final MaxAd maxAd) {
        // 사용하지 마세요!
    }

    @Override
    public void onAdDisplayFailed(final MaxAd maxAd, final MaxError error) {
        // 사용하지 마세요!
    }

    @Override
    public void onAdHidden(final MaxAd maxAd) {
        // 사용하지 마세요!
    }
}
```

## 전면 광고

### 전면 광고 로드

`MaxInterstitialAd`의 인스턴스를 생성하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

`MaxInterstitialAd`의 `loadAd()`를 호출해서 광고를 로드하세요.

`MaxAdListener`를 구현해서 광고 이벤트를 수신하세요.

`onAdDisplayed()` 콜백에서 `PointBerryImpressionTracker`의 `logImpression()`을 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```java
public class InterstitialAdActivity extends Activity implements MaxAdListener {

    private MaxInterstitialAd interstitialAd;
    private int retryAttempt;

    private PointBerryImpressionTracker impTracker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        interstitialAd = new MaxInterstitialAd("YOUR_MAX_AD_UNIT_ID", this);
        interstitialAd.setListener(this);

        // 첫 광고를 로드하세요.
        interstitialAd.loadAd();

        impTracker = new PointBerryImpressionTracker(getApplicationContext());
    }

    // 광고 이벤트

    @Override
    public void onAdLoaded(final MaxAd maxAd) {
        // 광고가 로드됐고 이제 게재될 수 있습니다.
        // interstitialAd.isReady()가 true를 리턴합니다.

        // 로드 재시도 횟수를 리셋하세요.
        retryAttempt = 0;
    }

    @Override
    public void onAdLoadFailed(final String adUnitId, final MaxError error) {
        // 광고가 로드에 실패했습니다.

        // 딜레이를 기하급수적으로 증가시키며 광고 로드를 재시도하세요.
        // 딜레이를 1초부터 64초까지 2배씩 증가시키는 예시입니다.
        retryAttempt++;
        long delayMillis = TimeUnit.SECONDS.toMillis((long) Math.pow(2, Math.min(6, retryAttempt)));
        new Handler().postDelayed(() -> interstitialAd.loadAd(), delayMillis);
    }

    @Override
    public void onAdDisplayFailed(final MaxAd maxAd, final MaxError error) {
        // 광고가 게재에 실패했습니다.

        // 다음 광고를 로드하세요.
        interstitialAd.loadAd();
    }

    @Override
    public void onAdDisplayed(final MaxAd maxAd) {
        // 광고가 게재됐습니다.

        // 광고 노출을 로깅하세요.
        impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID");

        // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
        // impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
    }

    @Override
    public void onAdClicked(final MaxAd maxAd) {
        // 광고가 클릭됐습니다.
    }

    @Override
    public void onAdHidden(final MaxAd maxAd) {
        // 광고가 닫혔습니다.

        // 다음 광고를 미리 로드하세요.
        interstitialAd.loadAd();
    }
}
```

### 전면 광고 게재

`MaxInterstitialAd`의 `showAd()`를 호출해서 광고를 게재하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```java
if (interstitialAd.isReady()) {
    interstitialAd.showAd("YOUR_POINTBERRY_INVENTORY_ID");
}
```

## 보상형 광고

### 보상형 광고 로드

`MaxRewaredAd`의 인스턴스를 생성하세요. 이때 매니저로부터 발급 받은 *MAX ad unit ID*를 사용하세요.

`MaxRewaredAd`의 `loadAd()`를 호출해서 광고를 로드하세요.

`MaxRewardedAdListener`를 구현해서 광고 이벤트를 수신하세요.

`onUserRewarded()` 콜백에서 `PointBerryImpressionTracker`의 `logImpression()`을 호출해서 광고 노출을 로깅하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```java
public class RewardedAdActivity extends Activity implements MaxRewardedAdListener {

    private MaxRewardedAd rewardedAd;
    private int retryAttempt;

    private PointBerryImpressionTracker impTracker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        rewardedAd = MaxRewardedAd.getInstance("YOUR_MAX_AD_UNIT_ID", this);
        rewardedAd.setListener(this);

        // 첫 광고를 로드하세요.
        rewardedAd.loadAd();

        impTracker = new PointBerryImpressionTracker(getApplicationContext());
    }

    // 광고 이벤트

    @Override
    public void onAdLoaded(final MaxAd maxAd) {
        // 광고가 로드됐고 이제 게재될 수 있습니다.
        // rewardedAd.isReady()가 true를 리턴합니다.

        // 로드 재시도 횟수를 리셋하세요.
        retryAttempt = 0;
    }

    @Override
    public void onAdLoadFailed(final String adUnitId, final MaxError error) {
        // 광고가 로드에 실패했습니다.

        // 딜레이를 기하급수적으로 증가시키며 광고 로드를 재시도하세요.
        // 딜레이를 1초부터 64초까지 2배씩 증가시키는 예시입니다.
        retryAttempt++;
        long delayMillis = TimeUnit.SECONDS.toMillis((long) Math.pow(2, Math.min(6, retryAttempt)));
        new Handler().postDelayed(() -> rewardedAd.loadAd(), delayMillis);
    }

    @Override
    public void onAdDisplayFailed(final MaxAd maxAd, final MaxError error) {
        // 광고가 게재에 실패했습니다.

        // 다음 광고를 로드하세요.
        rewardedAd.loadAd();
    }

    @Override
    public void onAdDisplayed(final MaxAd maxAd) {
        // 광고가 게재됐습니다.
    }

    @Override
    public void onAdClicked(final MaxAd maxAd) {
        // 광고가 클릭됐습니다.
    }

    @Override
    public void onRewardedVideoStarted(MaxAd ad) {
        // 광고가 시작됐습니다.
    }

    @Override
    public void onRewardedVideoCompleted(MaxAd ad) {
        // 광고가 종료됐습니다.
    }

    @Override
    public void onUserRewarded(MaxAd ad, MaxReward reward) {
        // 유저가 보상을 받아야 합니다.

        // 광고 노출을 로깅하세요.
        impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID");

        // 개발 환경에서는 development 파라미터에 true를 전달해서 로그 메시지를 출력하세요.
        // impTracker.logImpression("YOUR_POINTBERRY_INVENTORY_ID", true);
    }

    @Override
    public void onAdHidden(final MaxAd maxAd) {
        // 광고가 닫혔습니다.

        // 다음 광고를 미리 로드하세요.
        rewardedAd.loadAd();
    }
}
```

### 보상형 광고 게재

`MaxRewaredAd`의 `showAd()`를 호출해서 광고를 게재하세요. 이때 매니저로부터 발급 받은 *PointBerry inventory ID*를 사용하세요.

```java
if (rewardedAd.isReady()) {
    rewardedAd.showAd("YOUR_POINTBERRY_INVENTORY_ID");
}
```




# 2. PUBMATIC MAX Android SDK 연동 가이드

## 시작하기

PUBMATIC SDK 연동은 Prebid를 기반으로 한 광고 효율 최적화 방식으로 2023.11월(예정) 실 서비스 시작시 앱 별 집행 코드를 별도 전달할 예정이며, 이를 위해 사전 앱 내 설정을 권장합니다.

### 가. PUBMATIC SDK Github 주소

https://github.com/PubMatic/android-openwrap-sdk-samples

### 가. 그래들 안에 추가할 내용 ( build.gradle 에 추가. )

    // To integrate OpenWrap SDK
    implementation 'com.pubmatic.sdk:openwrap:3.0.0'
    // To integrate GAM event handler
    implementation 'com.pubmatic.sdk:openwrap-eventhandler-dfp:3.0.0'


### 나. 매니페스트 추가


```xml

<!-- Mandatory permission for OpenWrap SDK -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!--  Ask this permission to user (at runtime from code) only for API 30+ -->

<uses-permission android:name="android.permission.READ_PHONE_STATE" />

```


### 다. 매니페스트 사용을 위한 허용 요청

```xml
private void Permission() {
    // Runtime optional permission list
    List<String> permissionList = new ArrayList<>();
    permissionList.add(Manifest.permission.ACCESS_FINE_LOCATION);
    permissionList.add(Manifest.permission.ACCESS_COARSE_LOCATION);
    permissionList.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
    // Access READ_PHONE_STATE permission if api level 30 and above
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        permissionList.add(Manifest.permission.READ_PHONE_STATE);
    }

    final String[] PERMISSIONS = new String[permissionList.size()];
    permissionList.toArray(PERMISSIONS);
    // Ask permission from user for location and write external storage
    if (!hasPermissions(this, PERMISSIONS)) {
        int MULTIPLE_PERMISSIONS_REQUEST_CODE = 123;
        ActivityCompat.requestPermissions(this, PERMISSIONS, MULTIPLE_PERMISSIONS_REQUEST_CODE);
    }
}


private static boolean hasPermissions(Context context, String... permissions) {
    if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.M && context != null && permissions != null) {
        for (String permission : permissions) {
            if (ActivityCompat.checkSelfPermission(context, permission) != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
    }
    return true;
}

```

### 라. PUBMATIC 배너 뷰를 위한 정의 

```xml

pubmatic Banner View

   <com.pubmatic.sdk.openwrap.banner.POBBannerView
            android:id="@+id/banner"
            android:layout_width="320dp" // android:layout_width="300dp" 
            android:layout_height="50dp" // android:layout_height="250dp"
 />

```

### 마. PUBMATIC 인터스티셜 뷰를 위한 정의 


 // 아래 값들 중 코드 및 유닛 값은 임의 사항으로 실 반영시 별도 제공 예정

```xml

 // Interstitial
    void DFPInterstitial() {

        // Create an interstitial custom event handler for your ad server. Make sure
        // you use separate event handler objects to create each interstitial ad instance.
        // For example, The code below creates an event handler for DFP ad server.
        DFPInterstitialEventHandler eventHandler = new DFPInterstitialEventHandler(this, DFP_AD_UNIT_ID_Interstitial);

        // Create  interstitial instance by passing activity context and
        interstitial = new POBInterstitial(this,
                "YOUR_PUB_ID_Interstitial",                    // publisherId
                "YOUR_PROFILE_ID_Interstitial",                // profileId
                "YOUR_OPENWRAP_AD_UNIT_ONE_Interstitial",      // adUnitId
                eventHandler);

        // Set Optional listener
        interstitial.setListener(new POBInterstitialListener());


    }

// To show interstitial ad call this method
     
    private void showInterstitialAd() {
        // check if the interstitial is ready
        if (null != interstitial && interstitial.isReady()) {
            // Call show on interstitial
            interstitial.show();
        }
    }

// Interstitial Ad listener callbacks
    class POBInterstitialListener extends POBInterstitial.POBInterstitialListener {
        private final String TAG = "tpmn";

        // Callback method notifies that an ad has been received successfully.
        @Override
        public void onAdReceived(@NonNull POBInterstitial ad) {
            Log.d(TAG, "onAdReceived Interstitial");
            //Method gets called when ad gets loaded in container
            //Here, you can show interstitial ad to user
            btnInterstitial.setBackgroundColor(getResources().getColor(R.color.colorPrimary));

        }

        // Callback method notifies an error encountered while loading an ad.
        @Override
        public void onAdFailedToLoad(@NonNull POBInterstitial ad, @NonNull POBError error) {
            Log.e(TAG, "onAdFailedToLoad Interstitial : Ad failed to load with error -" + error.toString());
            //Method gets called when loadAd fails to load ad
            //Here, you can put logger and see why ad failed to load
        }

        // Callback method notifies an error encountered while showing an ad.
        @Override
        public void onAdFailedToShow(@NonNull POBInterstitial ad, @NonNull POBError error) {
            Log.e(TAG, "onAdFailedToShow Interstitial : Ad failed to show with error -" + error.toString());
            //Method gets called when loadAd fails to show ad
            //Here, you can put logger and see why ad failed to show
        }

        // Callback method notifies that a user interaction will open another app (for example, App Store), leaving the current app.
        @Override
        public void onAppLeaving(@NonNull POBInterstitial ad) {
            Log.d(TAG, "onAppLeaving");
        }

        // Callback method notifies that the interstitial ad will be presented as a modal on top of the current view controller
        @Override
        public void onAdOpened(@NonNull POBInterstitial ad) {
            Log.d(TAG, "onAdOpened Interstitial");
        }

        // Callback method notifies that the interstitial ad has been animated off the screen.
        @Override
        public void onAdClosed(@NonNull POBInterstitial ad) {
            Log.d(TAG, "onAdClosed Interstitial");
            btnInterstitial.setBackgroundColor(Color.GRAY);

        }

        // Callback method notifies ad click
        @Override
        public void onAdClicked(@NonNull POBInterstitial ad) {
            Log.d(TAG, "onAdClicked");
        }
    }

```

### 바. PUBMATIC 리워드 뷰를 위한 정의 

```xml

// Rewarded
    void DFPRewarded() {
    

        // Create an rewarded custom event handler for your ad server. Make sure
        // you use separate event handler objects to create each rewarded ad instance.
        // For example, The code below creates an event handler for DFP ad server.
        DFPRewardedEventHandler eventHandler = new DFPRewardedEventHandler(this, "YOUR_DFP_AD_UNIT_ID_Rewarded");

        //Create POBRewardedAd instance by passing activity context and profile parameters
        rewarded = POBRewardedAd.getRewardedAd(this,
                "YOUR_PUB_ID_Rewarded",
                "YOUR_PROFILE_ID_Rewarded",
                "YOUR_OPENWRAP_AD_UNIT_ID_Rewarded",
                eventHandler
        );

        //Set Optional Callback Listener
        rewarded.setListener(new POBRewardedAdListenerImpl());

    }

// To show Rewarded Ad Call this method

    private void showRewardedAd() {
        //Call showAd when Ad is ready
        if (rewarded != null && rewarded.isReady()) {
            rewarded.show();
        }
    }


// Rewarded Ad listener callbacks
    class POBRewardedAdListenerImpl extends POBRewardedAd.POBRewardedAdListener {
        private final String TAG = "tpmn";

        // Callback method notifies that an ad has been received successfully.
        @Override
        public void onAdReceived(@NonNull POBRewardedAd ad) {
            Log.d(TAG, "Rewarded: onAdReceived : " + ad);
            //Method gets called when ad gets loaded in container
            //Here, you can show Rewarded ad to user
            btnRewarded.setBackgroundColor(getResources().getColor(R.color.colorPrimary));
        }

        // Callback method notifies an error encountered while loading an ad.
        @Override
        public void onAdFailedToLoad(@NonNull POBRewardedAd ad, @NonNull POBError error) {
            Log.d(TAG, "Rewarded : onAdFailedToLoad");
            Log.e(TAG, "Ad failed with load error - " + error.toString());
            //Method gets called when sdk fails to load ad
            //Here, you can put logger and see why ad failed to load
        }

        // Callback method notifies an error encountered while rendering an ad.
        @Override
        public void onAdFailedToShow(@NonNull POBRewardedAd ad, @NonNull POBError error) {
            Log.e(TAG, "Rewarded : onAdFailedToShow");
            Log.e(TAG, "Ad failed with show error - " + error.toString());
            //Method gets called when sdk fails to show ad
            //Here, you can put logger and see why ad failed to show
        }


        // Callback method notifies that a user interaction will open another app (for example, App Store), leaving the current app.
        @Override
        public void onAppLeaving(@NonNull POBRewardedAd ad) {
            Log.d(TAG, "Rewarded : onAppLeaving");
        }

        // Callback method notifies that the rewarded ad will be presented as a modal on top of the current view controller
        @Override
        public void onAdOpened(@NonNull POBRewardedAd ad) {
            Log.d(TAG, "Rewarded : onAdOpened");
        }

        // Callback method notifies that the rewarded ad has been animated off the screen.
        @Override
        public void onAdClosed(@NonNull POBRewardedAd ad) {
            Log.d(TAG, "Rewarded : onAdClosed");
            btnRewarded.setBackgroundColor(Color.GRAY);
        }

        // Callback method notifies ad click
        @Override
        public void onAdClicked(@NonNull POBRewardedAd ad) {
            Log.d(TAG, "Rewarded : onAdClicked");
        }

        @Override
        public void onReceiveReward(@NonNull POBRewardedAd ad, @NonNull POBReward reward) {
            // As this is callback method, No action Required
            Log.d(TAG, "Rewarded : Ad should Reward -" + reward.getAmount() + "(" + reward.getCurrencyType() + ")");
            Toast.makeText(getApplicationContext(), "Congratulation! You are rewarded with " + reward.getAmount() + " " + reward.getCurrencyType(), Toast.LENGTH_LONG).show();
        }
    }

```

### 사. PUBMATIC 배너(320x50) 뷰를 위한 정의 

```xml

// Banner
    private void POBView() {

        try {

            // Initialize banner view
            banner = findViewById(R.id.banner);

            // Call init() to set tag information
            // For test IDs see - https://community.pubmatic.com/display/AOPO/Test+and+debug+your+integration#Testanddebugyourintegration-Testprofile/placement

            
            // Create a banner custom event handler for your ad server. Make sure you use
            // separate event handler objects to create each banner view.
            // For example, The code below creates an event handler for DFP ad server.
            DFPBannerEventHandler eventHandler = new DFPBannerEventHandler(this, AdUnit, AdSize.BANNER);

            // Initialise banner view
            banner = findViewById(R.id.banner);
            banner.init("YOUR_PublisherId",
                    "YOUR_ProfileId",
                    "YOUR_AdUnit",
                    eventHandler);

            //optional listener to listen banner events
            banner.setListener(new POBBannerViewListener());

            // Call loadAd() on banner instance
            banner.loadAd();

 
        } catch (Exception e) {
            e.printStackTrace();
            Log.e("tpmn", "Exception  e : " + e.getMessage());
        }
    }

```

### 아. PUBMATIC 배너(MREC:300x250) 뷰를 위한 정의 

```xml

// Banner 
    void BannerVideo() {


        // Create a banner custom event handler for your ad server. Make sure you use
        // separate event handler objects to create each banner view.
        // For example, The code below creates an event handler for DFP ad server with adsize medium rectangle i.e 300x250.
        DFPBannerEventHandler eventHandler = new DFPBannerEventHandler(this, "YOUR_DFP_AD_UNIT_ID_BV", AdSize.MEDIUM_RECTANGLE);

        // Initialise banner view
        bannerVideo = findViewById(R.id.bannerVideo);
        bannerVideo.init("YOUR_PUB_ID_BV",
                "YOUR_PROFILE_ID_BV",
                "YOUR_OPENWRAP_AD_UNIT_ID_BV",
                eventHandler);
 

        //optional listener to listen banner events
        bannerVideo.setListener(new POBBannerViewListener());

        // Call loadAd() on banner instance
        bannerVideo.loadAd();
    }


// Banner Ad listener callbacks

    class POBBannerViewListener extends POBBannerView.POBBannerViewListener {
        private final String TAG = "tpmn";

        // Callback method Notifies that an ad has been successfully loaded and rendered.
        @Override
        public void onAdReceived(@NonNull POBBannerView view) {
            Log.d(TAG, "Ad Received");
        }

        // Callback method Notifies an error encountered while loading or rendering an ad.
        @Override
        public void onAdFailed(@NonNull POBBannerView view, @NonNull POBError error) {
            Log.e(TAG, "onAdFailed : " + error.toString());
        }

        // Callback method Notifies whenever current app goes in the background due to user click
        @Override
        public void onAppLeaving(@NonNull POBBannerView view) {
            Log.d(TAG, "App Leaving");
        }

        // Callback method Notifies that the banner ad view is clicked.
        @Override
        public void onAdClicked(@NonNull POBBannerView view) {
            Log.d(TAG, "Ad Clicked");
        }

        // Callback method Notifies that the banner ad view will launch a dialog on top of the current view
        @Override
        public void onAdOpened(@NonNull POBBannerView view) {
            Log.d(TAG, "Ad Opened");
        }

        // Callback method Notifies that the banner ad view has dismissed the modal on top of the current view
        @Override
        public void onAdClosed(@NonNull POBBannerView view) {
            Log.d(TAG, "Ad Closed");
        }
    }


```



# 체크리스트

- [x] 광고가 노출되나요?

  광고가 노출되지 않는다면 MAX SDK를 점검하세요.

- [x] 광고 노출이 로깅되나요?

  `logImpression("YOUR_POINTBERRY_INVENTORY_ID", true)`을 호출하면 다음과 같은 로그 메시지가 출력되어야 합니다.

  _Successfully logged ad impression for inventory ID P_148 and advertising ID a1312407-a27a-4a2e-a52f-0a0ab89febe1_

  성공을 뜻하는 로그 메시지가 출력되지 않는다면, 즉 광고 노출이 로깅되지 않는다면 PointBerry Event Tracker를 점검하세요.

- [x] 앱 출시 전에, 광고 노출 로깅에 관한 로그 메시지가 출력되지 않도록 했나요?

  출시 버전에서는 `logImpression("YOUR_POINTBERRY_INVENTORY_ID")` 또는 `logImpression("YOUR_POINTBERRY_INVENTORY_ID", false)`을 호출하세요.

## 데모 앱

AppLovin에서 제공하는 데모 앱을 참고하세요.

- [Java 데모 앱](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/AppLovin%20MAX%20Demo%20App%20-%20Java)
- [Kotlin 데모 앱](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/AppLovin%20MAX%20Demo%20App%20-%20Kotlin)
