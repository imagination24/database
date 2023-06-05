# Flutter-Android 打包及相关设置

## Flutter 支持的平台 platformName(architecture)
* android-arm64(arm64-v8a)
* android-arm(armeabi-v7a)
* android-x64(x86_64)
* android-x86(x86)
## Apk打包命令

* fatApk(包含全部架构平台)
```
flutter build apk --obfuscate --split-debug-info=./symbols
```
如果只需要支持部分架构平台，可以在build.gradle中配置splits
```
···
android {
    ···
    splits {
        abi {
            enable true
            reset()
            include 'x86_64', 'armeabi-v7a', 'arm64-v8a'
            universalApk false
        }
    }
}
···
```
* abiApk(每个平台分离)
```
flutter build apk --obfuscate --split-debug-info=./symbols --split-per-abi

flutter build apk --obfuscate --split-debug-info=./symbols --target-platform=platformName --split-per-abi
```

## aab打包命令
```
flutter build appbundle --obfuscate --split-debug-info=./symbols
```
如果只需要支持某些平台可以在build.gradle中配置ndk

需要注意的是，ndk和splits不兼容，不能同时配置。
```
···
android {
    ···
    buildTypes {
        release {
            ···
            ndk {
               abiFilters 'x86_64', 'armeabi-v7a', 'arm64-v8a'
           }
        }
    }
    ···
}
···
```

*需要注意的是，splits abi 和 fatapk/aab 打包后的buildnumber是有差别的。splits abi 打包出的apk每个架构的buildnumber会递增1000，比如armeabi-v7a的buildnumber为1001，则arm64-v8a的buildnumber则为2001。fatapk和aab的buildnumber则为1，如果功能有需要用到buildnumber需要特别注意这点。*

## 打包apk-outputName自定义
```
···
android {
    ···
    android.applicationVariants.all {
        variant ->
            variant.outputs.all { output ->
                def buildType = variant.name
                ///SkyCastle-v2.0.0-arm64-v8a-release.apk
                outputFileName = "SkyCastle-v${defaultConfig.versionName}-${output.getFilter(com.android.build.OutputFile.ABI)}-${buildType}.apk"
            }
    }
    ···
}
···
```

## 为应用签名

*签名犹如应用的身份证，当你需要升级应用时，只有签名相同的应用可以升级，当签名不同时，会当成两个不同的应用，签名对应用是一种保护，起到对盗版应用的防范作用*

*如果上架GooglePlay，GooglePlay会自动帮助签名，不需要自己维护签名*

* keytool方式
```
//  -keystore：设置生成的文件名称，包含后缀；
//  -alias：设置别名
//  -storepass：设置文件的密码
//  -keypass：设置key的密码
//  -keyalg：设置使用的加密算法，一般写RSA
//  -keysize：指定密钥长度（默认 1024）
//  -validity：设置有效期，尽可能长啦（天）
keytool -genkey -v -alias 密钥库名称 -keyalg RSA -validity 有效时间 -keystore 存储路径 -storetype pkcs12
```
所有信息请自行替换
```
您的名字与姓氏是什么?
[Unknown]: *
您的组织单位名称是什么?
[Unknown]: *
您的组织名称是什么?
[Unknown]: *
您所在的城市或区域名称是什么?
[Unknown]: *
您所在的省/市/自治区名称是什么?
[Unknown]: *
该单位的双字母国家/地区代码是什么?
[Unknown]: *
CN=izpan, OU=izpan, O=izpan, L=SZ, ST=GD, C=CN是否正确?
[否]: y
```

* Android Studio生成

*生成签名文件*

*在 Build 中选择 Generate Signed Bundle / APK…*

*勾选 APK，点击Next，再选择Create new… 创建密钥库*

*填写密钥库相关信息，然后点击OK*

* 生成签名后在build.gradle中配置signingConfigs
*填写生成签名时填入的信息，确保一致*
```
···
android {
    ···

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    ···
}

···
```
