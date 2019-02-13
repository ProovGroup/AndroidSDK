# WeProovSDK

## Requirements

### Dependency installation
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

```
implementation 'com.ProovGroup.AndroidSDK:AndroidSDK:$sdk_version'
```

### Gradle parameters
Change java version to 1.8 and activate android's dataBinding
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

### Permissions and service declaration
In `AndroidManifest.xml` add the required permissions and the WeProov upload service
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

## Usage

### SDK initialisation
At the app startup, call to ``WPConfig.init("fr");`` to initialise the SDK.
```
WPConfig.init("fr");
```

### Setup your theme
Change the colors of the theme through the Android resources by overriding the keys used by the SDK. 

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    ...

    <color name="wprv_report_initial">#67B0F9</color> <!-- Main tint of an initial report -->
    <color name="wprv_report_final">#67B0F9</color> <!-- Main tint of a final report -->
    ...
</resources>
```
More information on the existing fields and their function later. 

### Backend connection
To connect to the backend, call `WPConfig.connect`
```
    WPConfig.connect(this, "<token>", "<secret>", new WPConfig.ConnectionListener() {
        @Override
        public void onError(Exception e) {
            Log.e(TAG", "Error on connect", e);

        }       
        @Override
        public void onConnected() {
        	// SDK is connected to WeProov
        }
    });
```
### Parameters
The SDK takes some parameters.
```
WPParameters params = new WPParameters();
params.partOptions.put(0, new WPPartOption(false,true));
```

### To load and start a report in an Activity

Before you start displaying a report in the WeProov SDK, you need to load it from its code.

The easiest way is to do that is to start a WPLoadingActivity with the proper parameters. This Activty will take care of loading the report, displaying the UI of the load, then opening an Activity displaying the report.

```
startActivity(WPLoadingActivity.getIntent(MainActivity.this, proovCode, new WPParameters()));
```

But if you wish to make your own UI for displaying the progress of the load, or if you wish to display the report in a fragment instead of an activity (see below), you will have to take care of loading the report by yourself.

For this, we provide you with the `WPReportDownloader` class.
This class offers method to start loading report or template, and livedata to observe the progress of this load. 
Look at the implementation of the `WPReportDownloadingController` for an example of how to use the `WPReportDownloader`.

Once the report is completely loaded, you can start a `WPReportActivity`:
```
WPParameters parameters =  new WPParameters(array);
startActivity(WPReportActivity.getStartingIntent(MainActivity.this, parameters));
```

### To display the report in a Fragment
If you wish to display the WeProov report inside of your own Activity instead of the `WPReportActivity`, you can use the `WPReportFragment`.
To instantiate it, you can use for example a FragmentTransaction:

```
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
transaction.replace(R.id.container, WPReportFragment.newInstance(new WPParameters());
transaction.commit();
```

#### Activity duty
It is the job of the Activity containing the WPReportFragment to reset the configuration when the user navigates back out of the report by calling `WPConfig.reset()`.
Typically:
```
@Override
public void onBackPressed(){
    WPConfig.reset();
    super.onBackPressed();
}
```

#### WPReportViewModel
The WeProov SDK for Android uses the android architecture components such as the `ViewModel` and `LiveData` components.
If you are not familiar with these components yets, you can find information on the Android documentation website: https://developer.android.com/topic/libraries/architecture/viewmodel

You can get information and control the `WPReportFragment` through the `WPReportViewModel`.
To get a reference to the WPReportViewModel:
```
ViewModelProviders.of(this).get(WPReportViewModel.class);
```
Where `this` is a reference to the Activity containing the `WPReportFragment`.

Some methods of interests in the WPReportViewModel:
- `WPReportViewModel.reportLiveData`: a livedata of the current report model. Be careful to not do long operation in an observer of this value as this is call very often, at every character change of any field of the report
- `WPReportViewModel.titleLiveData`: a livedata of the title of the current page being displayed
- `WPReportViewModel.setCurrentPage(int index)`: set the page to display
- `WPReportViewModel.getCurrentPage()`: returns the current index of the page being displayed
- `WPReportViewModel.goToNextStep()`: display the next page of the report, if any
- `WPReportViewModel.getNavigationIndexLiveData()`: returns a live data of the current index of the page being displayed
- `WPReportViewModel.getPageCount()`: returns the number of page in the report
- `WPReportViewModel.getInvalidCountsLiveData()`: returns the number of invalid fields in each page of the report, and weither or not they should be displayed 
- `WPReportViewModel.signAndFinish()`: if the report is completed, start the signature and upload processes

### Receive results from the SDK
To receive results from the SDK, listen to the `onActivityResult` inside the Activity containing a `WPReportFragment` or the Activity starting a `WPReportActivity`.

```
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if(requestCode == WPReportFragment.REQ_UPLOAD){ 
        // ... The upload activity was closed, the process is complete
    }
    else if(requestCode == WPReportFragment.REQ_SIGNATURE){ //La vue de Signature vient de se fermer
        if (resultCode == RESULT_OK){
            // ... Signature was completed correctly, the upload is about to start
        }
        else{
            // ... Back from the signature from a back or a cancel, the user might want to continue editing the report
        }
    }
    super.onActivityResult(requestCode, resultCode, data); //Never forget to call this method so that fragments inside the main activity can receive the results too
}
```

## Theme colors
Every colors define in the WeProov SDK can be overriden by the main project to customize the theme. Some of the most important colors are:
- `wprv_primary`, defines the main color of widget outside of specific fields of a report. Defaults to #17BBA9
- `wprv_report_initial`, defines the main color of fields in a report in initial state. Defaults to #8066E0
- `wprv_report_final`, defines the main color of fields in a report in final state. Defaults to #8066E0
- `wprv_report_archived`, defines the main color of fields in a report in archived state. Defaults to `wprv_primary`

If you have more specific customization needs, contact us.