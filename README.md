Agregar react-native a proyecto Android existente
=================================================

Antes de usar:


Levantar el packager


```
cd /tmp
git clone https://github.com/hanspoo/AndroidMasReactNativo
cd AndroidMasReactNativo/proyectoreact
yarn install
yarn start
```
En otro terminal levantar android. También lo pueden importar y lanzar desde android estudio o hacer:

```
cd /tmp/AndroidMasReactNativo
cd MyApplication
adb reverse tcp:8081 tcp:8081
./gradlew installDebug
adb shell am start -n com.demo.myapplication/.MyReactActivity
```

## Como Agregar React Native a tu proyecto Android

A continuación se explican los pasos y modificaciones mínimos necesarios para agregar react-native a un proyecto android preexistente. Para este demo hemos los dos proyectos al mismo nivel uno con android studio el otro con expo-cli.

Para el proyecto react-native hemos usado el flujo “bare workflow” con typescript. Para el proyecto Android hemos usado el template “Empty Activity” con Java.

En el proyecto react-native recien creado
---------------------------------------------

Sólo hay que instalar javascript runtime jsc-android.
 
```
expo init proyectoreact -t expo-template-bare-typescript

cd proyectoreact/

yarn add jsc-android
```

En el proyecto Android
----------------------

### build.gradle

En este archivo. Sólo se agregan dos repositorios de tipo maven. Deben referencian el proyecto RN recién creado.

```
allprojects {
    repositories {
    ...
        maven {
            url "$rootDir/../proyectoreact/node_modules/react-native/android"
        }
        maven {
            url("$rootDir/../proyectoreact/node_modules/jsc-android/dist")
        }
    }


```

### app/build.gradle
Sólo se agregan 3 dependencias.
 
```
dependencies {
...
    implementation "org.webkit:android-jsc:r241213"
    implementation 'com.facebook.react:react-native:+' //Added Dependency
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0-alpha02'
}
```

### La vista/aplicación react-native

El nombre ***proyectoreact*** debe ser el mismo registrado por el app react native, normalmente la propiedad name en ***proyectoreact/app.json***. Esta actividad la creamos en la carpeta de código fuente donde estan quedando las actividades de Android, normalmente algo como ***src/main/xxx/yyy***. Hemos omitido la declaracion package para que salga más fácil de copiar y pegar.


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
        mReactRootView.startReactApplication(mReactInstanceManager, "proyectoreact", null);
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

Para todos los efectos la actividad recién creada MyReactActivity es igual a cualquiera del proyecto android y como tal se puede agregar en el manifiesto, incluso de partida como es el caso. Se han agregado:

Permisos
```    <uses-permission android:name="android.permission.INTERNET" />```

Poder acceder al bundle JS en modo dev
```        android:usesCleartextTraffic="true"```

La actividad de devSettings con shake o Ctrl-M.
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
Si dejamos la de react como de partida, debemos quitarle el intent-filter a la predeterminada.


