# AppLovin MAX Android SDK 연동 가이드

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

    // AdMob-to-MAX adapter
    implementation 'com.applovin.mediation:google-adapter:20.6.0.3'

    // Unity-Ads-to-MAX adapter
    implementation 'com.applovin.mediation:unityads-adapter:4.0.1.1'

    // Android Advertising ID (AAID)
    implementation 'com.google.android.gms:play-services-ads-identifier:18.0.1'

    // PointBerry Event Tracker
    implementation 'com.github.connect-n:pointberry-event-tracker-android:1.0.2'
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

## 데모 앱

AppLovin에서 제공하는 데모 앱을 참고하세요.

- [Java 데모 앱](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/AppLovin%20MAX%20Demo%20App%20-%20Java)
- [Kotlin 데모 앱](https://github.com/AppLovin/AppLovin-MAX-SDK-Android/tree/master/AppLovin%20MAX%20Demo%20App%20-%20Kotlin)
