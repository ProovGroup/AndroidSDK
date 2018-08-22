## Installation

Coller le WeProovSdk.aar dans app/libs

### build.gradle
Changer la version et activer le dataBinding
```
android {
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    dataBinding {
        enabled = true
    }
    ...
}
```

```
dependencies {
    ...

    implementation files('libs/WeProovSdk.aar')
    implementation 'com.android.support:design:27.1.1'

    implementation 'com.android.support:appcompat-v7:27.1.1'
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    implementation 'com.android.support:design:27.1.1'
    implementation 'com.android.support:multidex:1.0.3'

    implementation 'com.android.support:support-v4:27.1.1'
    implementation 'com.android.support:recyclerview-v7:27.1.1'
    implementation 'com.android.support:gridlayout-v7:27.1.1'
    implementation 'com.android.support:exifinterface:27.1.1'

    implementation 'com.google.android.gms:play-services-location:15.0.1'
    implementation 'com.google.android.gms:play-services-gcm:15.0.1'
    implementation 'com.google.code.gson:gson:2.8.0'

    implementation 'com.github.bumptech.glide:glide:3.8.0'


    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'

    implementation "android.arch.lifecycle:runtime:1.1.0"
    implementation "android.arch.lifecycle:common-java8:1.1.0"
    implementation "android.arch.lifecycle:extensions:1.1.0"
    implementation "android.arch.lifecycle:reactivestreams:1.1.0"
    annotationProcessor "android.arch.lifecycle:compiler:1.1.0"

    api 'me.dm7.barcodescanner:zxing:1.9.8'

    implementation 'com.squareup.okhttp3:okhttp:3.8.1'
    implementation 'io.github.lizhangqu:coreprogress:1.0.2'
}
```
## Permission
AndroidManifest.xml ajouter les drois
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="...">
    ...
	<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application>
        ...
        <service
            android:name="com.weproov.sdk.WPReportUploadService"
            android:exported="true"
            android:permission="com.google.android.gms.permission.BIND_NETWORK_TASK_SERVICE">
            <intent-filter>
                <action android:name="com.google.android.gms.gcm.ACTION_TASK_READY" />
            </intent-filter>
        </service>
    </application>
</manifest>
```


## Initialisation 
Au lancement de l'app appeler ``WPConfig.init("fr");``
```
WPConfig.init("fr");
```

## colors.xml
Changer les couleur du theme.

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    ...

    <color name="wprv_report_initial">#67B0F9</color> <!-- Main tint of an initial report -->
    <color name="wprv_report_final">#67B0F9</color> <!-- Main tint of a final report -->
    ...
</resources>
```

pour connecter au backend
```
    WPConfig.connect(this, "<token>", "<secret>", new WPConfig.ConnectionListener() {
        @Override
        public void onError(Exception e) {
            Log.e("WeProovSDK", "Error on connect", e);

        }       
        @Override
        public void onConnected() {
        	// SDK is connected to WeProov
        }
    });
```

pour ouvire un rapport dans une vue manager

```
startActivity(WPLoadingActivity.getIntent(MainActivity.this, proovCode, params));
```


