## Installation
### ajouter le repo weproov
```
allprojects {
    repositories {
        ...
        maven{
            url "https://proovgroup.github.io/AndroidSDK/"
        }
    }
}
```

### Dependence
```
implementation 'com.ProovGroup.AndroidSDK:AndroidSDK:0.0.19'
```

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
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

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

Pour ouvrir un rapport dans une vue manager

```
WPParameters params = new WPParameters();
params.partOptions.put(0, new WPPartOption(false,true));
startActivity(WPLoadingActivity.getIntent(MainActivity.this, proovCode, params));
```

Pour écouter les résultats des Activities du SDK, au sein de l'Activity contenant un WPReportFragment ou de l'Activity initialisant une WPReportActivity.

```
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if(requestCode == WPReportFragment.REQ_UPLOAD){ 
        // ... La vue d'upload vient de se fermer, le processus est terminée
    }
    else if(requestCode == WPReportFragment.REQ_SIGNATURE){ //La vue de Signature vient de se fermer
        if (resultCode == RESULT_OK){
            // ... Signature correctement terminée, l'upload est sur le point de démarrer
        }
        else{
            // ... Retour sans avoir terminé la signature, le rapport peut continuer
        }
    }
    super.onActivityResult(requestCode, resultCode, data); //Important de toujours faire cet appel pour que WPReportFragment reçoive la callback
}
```

