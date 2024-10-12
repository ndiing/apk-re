## Memodifikasi dan Membangun Ulang APK untuk Debugging Jaringan

Pastikan PC Anda sudah terinstall [JDK](https://builds.openlogic.com/downloadJDK/openlogic-openjdk/21.0.4+7/openlogic-openjdk-21.0.4+7-windows-x64.msi) dan [JRE](https://builds.openlogic.com/downloadJDK/openlogic-openjdk-jre/21.0.4+7/openlogic-openjdk-jre-21.0.4+7-windows-x64.zip). Setelah itu, download dan install [apktool](https://apktool.org/).

### Decoding
Untuk mendekode APK, jalankan perintah berikut:

```bash
apktool d --force YOUR_APK_NAME.apk
```

### Modding
Masuk ke folder hasil dekode APK. Cari file `AndroidManifest.xml`, dan periksa apakah atribut `network_security_config` sudah ada di tag `<application>`. Jika belum, tambahkan seperti ini:

```xml
<application android:networkSecurityConfig="@xml/network_security_config">
</application>
```

Kemudian, buka folder `res/xml/` dan cek apakah file `network_security_config.xml` ada. Jika belum, buat file tersebut dengan isi sebagai berikut:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

### Building
Setelah modifikasi selesai, bangun ulang APK dengan perintah berikut:

```bash
apktool b --debug --force-all -no-crunch --use-aapt2 --output YOUR_APK_NAME.debug.apk YOUR_APK_NAME

```

### Signing APK
Jika Anda belum memiliki `debug.keystore`, buat dengan perintah berikut:

```bash
keytool -genkey -v -keystore debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000 -dname "C=US, O=Android, CN=Android Debug"
```

Kemudian, sign APK dengan perintah ini:

```bash
jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore debug.keystore -storepass android YOUR_APK_NAME.debug.apk androiddebugkey

```

### Instalasi
Setelah APK siap, lanjutkan dengan instalasi APK ke perangkat.

## Zipalign (Opsional)
Jika diperlukan, lakukan `zipalign` dengan urutan sebagai berikut:

1. Lakukan dekode APK seperti sebelumnya:
   ```bash
   apktool d --force YOUR_APK_NAME.apk
   ```

2. Tambahkan atribut `network_security_config` seperti langkah sebelumnya:
   ```xml
   <application android:networkSecurityConfig="@xml/network_security_config">
   </application>
   ```

3. Lakukan modifikasi dan bangun ulang APK seperti langkah sebelumnya:
   ```bash
   apktool b --debug --force-all -no-crunch --use-aapt2 --output YOUR_APK_NAME.debug.apk YOUR_APK_NAME
   
   ```

4. Sebelum signing, lakukan `zipalign` dengan perintah berikut:
   ```bash
   zipalign -p -f -v 4 YOUR_APK_NAME.debug.apk YOUR_APK_NAME.zipalign.apk
   ```

5. Sign APK hasil `zipalign` dengan perintah berikut:
   ```bash
   apksigner.jar sign --ks debug.keystore --ks-pass pass:android --ks-key-alias androiddebugkey --key-pass pass:android YOUR_APK_NAME.zipalign.apk
   
   ```

### Instalasi
Setelah APK siap, lanjutkan dengan instalasi APK ke perangkat.

