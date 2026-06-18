# Tolol Launcher - Source Code

## app/src/main/java/com/tolollauncher/MainActivity.java
```java
package com.tolollauncher;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.widget.*;
import java.io.*;
import java.util.zip.*;

public class MainActivity extends Activity {

    private TextView statusText;
    private TextView driverNameText;
    private String selectedDriverPath = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        statusText = findViewById(R.id.statusText);
        driverNameText = findViewById(R.id.driverNameText);
        Button pickZipBtn = findViewById(R.id.pickZipBtn);
        Button installBtn = findViewById(R.id.installBtn);
        Button clearBtn = findViewById(R.id.clearBtn);

        updateDriverStatus();

        pickZipBtn.setOnClickListener(v -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/zip");
            startActivityForResult(intent, 1);
        });

        installBtn.setOnClickListener(v -> {
            if (selectedDriverPath == null) {
                toast("Pilih ZIP driver dulu!");
                return;
            }
            installDriver(selectedDriverPath);
        });

        clearBtn.setOnClickListener(v -> clearDriver());
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == 1 && resultCode == RESULT_OK && data != null) {
            Uri uri = data.getData();
            try {
                File zipFile = copyUriToCache(uri);
                selectedDriverPath = zipFile.getAbsolutePath();
                driverNameText.setText("ZIP: " + zipFile.getName());
                statusText.setText("ZIP siap diinstall!");
            } catch (Exception e) {
                toast("Gagal baca ZIP: " + e.getMessage());
            }
        }
    }

    private File copyUriToCache(Uri uri) throws IOException {
        File cacheFile = new File(getCacheDir(), "driver_temp.zip");
        InputStream in = getContentResolver().openInputStream(uri);
        FileOutputStream out = new FileOutputStream(cacheFile);
        byte[] buf = new byte[4096];
        int len;
        while ((len = in.read(buf)) > 0) out.write(buf, 0, len);
        in.close();
        out.close();
        return cacheFile;
    }

    private void installDriver(String zipPath) {
        File driverDir = getDriverDir();
        if (!driverDir.exists()) driverDir.mkdirs();
        for (File f : driverDir.listFiles()) f.delete();

        try {
            ZipInputStream zis = new ZipInputStream(new FileInputStream(zipPath));
            ZipEntry entry;
            boolean foundSo = false;

            while ((entry = zis.getNextEntry()) != null) {
                String name = new File(entry.getName()).getName();
                if (name.endsWith(".so")) {
                    File outFile = new File(driverDir, name);
                    FileOutputStream fos = new FileOutputStream(outFile);
                    byte[] buf = new byte[4096];
                    int len;
                    while ((len = zis.read(buf)) > 0) fos.write(buf, 0, len);
                    fos.close();
                    foundSo = true;
                }
                zis.closeEntry();
            }
            zis.close();

            if (foundSo) {
                saveDriverPath(driverDir.getAbsolutePath());
                statusText.setText("✅ Driver berhasil diinstall!\nPath: " + driverDir.getAbsolutePath());
                toast("Driver diinstall!");
            } else {
                statusText.setText("❌ Tidak ada file .so di ZIP!");
            }
        } catch (Exception e) {
            statusText.setText("❌ Error: " + e.getMessage());
        }
    }

    private void clearDriver() {
        File driverDir = getDriverDir();
        if (driverDir.exists()) {
            for (File f : driverDir.listFiles()) f.delete();
        }
        clearDriverPath();
        selectedDriverPath = null;
        driverNameText.setText("Belum ada driver dipilih");
        statusText.setText("Driver dihapus!");
        toast("Driver dihapus!");
    }

    private void updateDriverStatus() {
        String path = getSavedDriverPath();
        if (path != null) {
            statusText.setText("✅ Driver aktif:\n" + path);
        } else {
            statusText.setText("Belum ada driver diinstall");
        }
    }

    private File getDriverDir() {
        return new File(getExternalFilesDir(null), "custom_driver");
    }

    private void saveDriverPath(String path) {
        getSharedPreferences("driver", MODE_PRIVATE).edit().putString("path", path).apply();
    }

    private String getSavedDriverPath() {
        return getSharedPreferences("driver", MODE_PRIVATE).getString("path", null);
    }

    private void clearDriverPath() {
        getSharedPreferences("driver", MODE_PRIVATE).edit().remove("path").apply();
    }

    private void toast(String msg) {
        Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
    }
}
```

## app/src/main/res/layout/activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:background="#1a1a2e">

    <TextView
        android:text="🎮 Tolol Launcher"
        android:textSize="28sp"
        android:textColor="#e94560"
        android:textStyle="bold"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:text="Custom Driver Manager"
        android:textSize="14sp"
        android:textColor="#888888"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="24dp"/>

    <TextView
        android:id="@+id/statusText"
        android:text="Belum ada driver diinstall"
        android:textSize="13sp"
        android:textColor="#aaaaaa"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#16213e"
        android:padding="12dp"
        android:layout_marginBottom="24dp"/>

    <TextView
        android:id="@+id/driverNameText"
        android:text="Belum ada driver dipilih"
        android:textSize="13sp"
        android:textColor="#aaaaaa"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#16213e"
        android:padding="12dp"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/pickZipBtn"
        android:text="📂 Pilih ZIP Driver"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:backgroundTint="#0f3460"
        android:textColor="#ffffff"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/installBtn"
        android:text="⚡ Install Driver"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:backgroundTint="#e94560"
        android:textColor="#ffffff"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/clearBtn"
        android:text="🗑️ Hapus Driver"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:backgroundTint="#333333"
        android:textColor="#ffffff"/>

</LinearLayout>
```

## app/src/main/res/values/strings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Tolol Launcher</string>
</resources>
```

## app/src/main/AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.tolollauncher">

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:theme="@android:style/Theme.Material.NoActionBar">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

## build.gradle (root)
```groovy
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.4.2'
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

## app/build.gradle
```groovy
plugins {
    id 'com.android.application'
}

android {
    compileSdk 33
    defaultConfig {
        applicationId "com.tolollauncher"
        minSdk 26
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
        }
    }
}
```

## settings.gradle
```groovy
include ':app'
```

## .github/workflows/build.yml
```yaml
name: Build Tolol Launcher

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Setup Android SDK
        run: |
          wget -q -O cmdtools.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
          mkdir -p $ANDROID_HOME/cmdline-tools
          unzip -q cmdtools.zip -d $ANDROID_HOME/cmdline-tools
          mv $ANDROID_HOME/cmdline-tools/cmdline-tools $ANDROID_HOME/cmdline-tools/latest
          echo "y" | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager --licenses > /dev/null
          echo "y" | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager "build-tools;33.0.2" "platforms;android-33" > /dev/null

      - uses: gradle/actions/setup-gradle@v4

      - name: Build APK
        run: gradle assembleDebug

      - uses: actions/upload-artifact@v4
        with:
          name: tolol-launcher-apk
          path: app/build/outputs/apk/debug/*.apk
```
