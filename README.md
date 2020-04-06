Add react native to existing Android Project
=============================================
Below are the minimal steps and modifications necessary to add react-native to a pre-existing android project. For this demo we have the two projects at the same level.
For the react-native project we have used the “bare workflow” with typescript. For the Android project we have used the “Empty Activity” template with Java.

React native side
---------------------------------------------
Just install javascript core librarie jsc-android.
 
```

mkdir /tmp/integration1
cd /tmp/integration1

expo init reactproject -t expo-template-bare-typescript

cd reactproject/

yarn add jsc-android
```

Android Side
----------------------
### build.gradle

In this project level file we add two maven repositores. They point below node_modules in the RN (react native) project.
Create the project under in /tmp/integration1 folder.

```
allprojects {
    repositories {
    ...
        maven {
            url "$rootDir/../reactproject/node_modules/react-native/android"
        }
        maven {
            url("$rootDir/../reactproject/node_modules/jsc-android/dist")
        }
    }


```

### app/build.gradle
In the app level build.gradle just add 3 dependencies.
 
```
dependencies {
...
    implementation "org.webkit:android-jsc:r241213"
    implementation 'com.facebook.react:react-native:+' //Added Dependency
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0-alpha02'
}
```


### The RN view

The parameter to startReactApplication should be the same of the RN project, usually the name property in ***reactproject/app.json***. We create this activity besides the other activities in the java source code folder, usually something like ***src/main/xxx/yyy***. We have omitted the package declaration to make it easier to copy and paste.

With expo version 36 as of Abril 6th expo modules doesn't register as the app name, but with a generic name: "main".

so
        mReactRootView.startReactApplication(mReactInstanceManager, "reactproject", null);
should be
        mReactRootView.startReactApplication(mReactInstanceManager, "main", null);

It's consistent, the names must match, so the error is loading the bundle.

```
import android.app.Activity;
import android.os.Bundle;
import android.view.KeyEvent;
import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;
import com.facebook.react.shell.MainReactPackage;
import com.facebook.soloader.SoLoader;

public class MyReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SoLoader.init(this, /* native exopackage */ false);
        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setCurrentActivity(this)
                .setBundleAssetName("index.android.bundle")
                .setJSMainModulePath("index")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        // The string here (e.g. "MyReactNativeApp") has to match
        // the string in AppRegistry.registerComponent() in index.js
        mReactRootView.startReactApplication(mReactInstanceManager, "reactproject", null);
        setContentView(mReactRootView);
    }
    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }
    @Override
    protected void onResume() {
        super.onResume();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
        if (mReactRootView != null) {
            mReactRootView.unmountReactApplication();
        }
    }
    @Override
    public void onBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }
    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyUp(keyCode, event);
    }
}
```

### AndroidManifest.xml

For all purposes, the newly created MyReactActivity activity is like any other. and as such can be added to the manifest, even as a starting activity. The following are the modifications:

Permissions
```    <uses-permission android:name="android.permission.INTERNET" />```

To be able to load JS bundle from port 8081.
```        android:usesCleartextTraffic="true"```

Development menu Activity, shake or Ctrl-M.
```        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />            ```

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.demo.androidreactdemo">
    <uses-permission android:name="android.permission.INTERNET" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:usesCleartextTraffic="true">

        <activity
            android:name=".MyReactActivity"
            android:label="@string/app_name"
            android:theme="@style/Theme.AppCompat.Light.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
        <activity android:name=".MainActivity"></activity>
    </application>
</manifest>
```
We are starting with the react activity, so we have put the intent filter in it.

## Notes to execute

Before running the android project, remember to start the packager in the react project:

```yarn start ```

Don't forget to reverse proxy port 8081

```adb reverse tcp:8081 tcp:8081 ```
 

## Making release apk

### Create React JS Bundle and put it in Android Project

```
cd reactproject
mkdir ../MyApplication/app/src/main/assets
react-native bundle --platform android --dev false --entry-file index.js \
--bundle-output ../MyApplication/app/src/main/assets/index.android.bundle \
--assets-dest ../MyApplication/app/src/main/res
```

Create signed apk using android studio, with next command install apk in the device.

```
cd  ../MyApplication
adb install ./app/release/app-release.apk 
```



