## Memodifikasi dan Membangun Ulang APK untuk Debugging Jaringan

Pastikan PC Anda sudah terinstall [JDK](https://builds.openlogic.com/downloadJDK/openlogic-openjdk/21.0.4+7/openlogic-openjdk-21.0.4+7-windows-x64.msi) dan [JRE](https://builds.openlogic.com/downloadJDK/openlogic-openjdk-jre/21.0.4+7/openlogic-openjdk-jre-21.0.4+7-windows-x64.zip). Setelah itu, download dan install [apktool](https://apktool.org/).

### Decoding
Untuk mendekode APK, jalankan perintah berikut:

<pre>
apktool d --force sf.apk
</pre>

### Modding
Masuk ke folder hasil dekode APK. Cari file `AndroidManifest.xml`, dan periksa apakah atribut `network_security_config` sudah ada di tag `<application>`. Jika belum, tambahkan seperti ini:

<pre>
<application android:networkSecurityConfig="@xml/network_security_config">
</application>
</pre>

Kemudian, buka folder `res/xml/` dan cek apakah file `network_security_config.xml` ada. Jika belum, buat file tersebut dengan isi sebagai berikut:

<pre>
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
</pre>

### Building
Setelah modifikasi selesai, bangun ulang APK dengan perintah berikut:

<pre>
apktool b --debug --force-all -no-crunch --use-aapt2 --output sf.debug.apk sf
</pre>

### Signing APK
Jika Anda belum memiliki `debug.keystore`, buat dengan perintah berikut:

<pre>
keytool -genkey -v -keystore debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000 -dname "C=US, O=Android, CN=Android Debug"
</pre>

Kemudian, sign APK dengan perintah ini:

<pre>
jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore debug.keystore -storepass android sf.debug.apk androiddebugkey
</pre>

### Instalasi
Setelah APK siap, lanjutkan dengan instalasi APK ke perangkat.

## Zipalign (Opsional)
Jika diperlukan, lakukan `zipalign` dengan urutan sebagai berikut:

1. Lakukan dekode APK seperti sebelumnya:
   <pre>
   apktool d --force sf.apk
   </pre>

2. Tambahkan atribut `network_security_config` seperti langkah sebelumnya:
   <pre>
   <application android:networkSecurityConfig="@xml/network_security_config">
   </application>
   </pre>

3. Lakukan modifikasi dan bangun ulang APK seperti langkah sebelumnya:
   <pre>
   apktool b --debug --force-all -no-crunch --use-aapt2 --output sf.debug.apk sf
   </pre>

4. Sebelum signing, lakukan `zipalign` dengan perintah berikut:
   <pre>
   zipalign -p -f -v 4 sf.debug.apk sf.zipalign.apk
   </pre>

5. Sign APK hasil `zipalign` dengan perintah berikut:
   <pre>
   apksigner.jar sign --ks debug.keystore --ks-pass pass:android --ks-key-alias androiddebugkey --key-pass pass:android sf.zipalign.apk
   </pre>

### Instalasi
Setelah APK siap, lanjutkan dengan instalasi APK ke perangkat.
