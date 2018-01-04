# XiaoXiHotFixDemo

热更新火了这么久，还没亲自玩过，一直很惭愧，现在有空了，就花点时间学习下。<br>
详细流程建议参考：

1.buggly热更新接入文档：
<a href="https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20171109131920#_2">https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20171109131920#_2</a>

2.buggly热更新接入视频教程：
<a href="http://v.qq.com/boke/gplay/9f3b4b1232819f453becd2356a3493c4_bme000301803d13_5_k0385qx3tk2.html">http://v.qq.com/boke/gplay/9f3b4b1232819f453becd2356a3493c4_bme000301803d13_5_k0385qx3tk2.html</a>

3.buggly热更新接入Demo：
<a href="https://github.com/BuglyDevTeam/Bugly-Android-Demo/blob/master/README.md">https://github.com/BuglyDevTeam/Bugly-Android-Demo/blob/master/README.md</a>
(下载的文件里面包含四个Demo。我参考的是：BuglyHotfixDemo)

反正三个我都看了（不止一遍）。

<font color="#FF0000">**## 注意1：在接入的过程中，不要想着instant run运行项目，等接入完成后，我会告诉你怎么使用instant run安装apk。**</font> 

**准备工作**：

* 1.用自己的QQ账号在Buggly里面创建一个产品，方便热更新的测试。(直接用公司的账号，会造成垃圾数据)
![](https://user-gold-cdn.xitu.io/2018/1/4/160bffadef89782e?w=3082&h=588&f=jpeg&s=86455)

* 2.新建一个demo工程，不要直接在项目里面接入你还没接触过的代码。不然出现什么问题，项目不能正常打包。到时候你就懵逼了。


![](https://user-gold-cdn.xitu.io/2018/1/4/160c02f4ccf8dd5a?w=3352&h=2054&f=png&s=472520)

切换到Project结构：


![](https://user-gold-cdn.xitu.io/2018/1/4/160c0309ddd72f34?w=3352&h=2054&f=png&s=489811)

## 第一步：在app目录下，创建tinker-support.gradle并打开，将如下内容复制进去：

```
apply plugin: 'com.tencent.bugly.tinker-support'

def bakPath = file("${buildDir}/bakApk/")

/**
 * 此处填写每次构建生成的基准包目录
 */
//todo 1.配置基准包目录[常改项]
def baseApkDir = "app-0104-18-04-33"

/**
 * 对于插件各参数的详细解析请参考
 */
tinkerSupport {

    // 开启tinker-support插件，默认值true
    //todo instant run
//    enable = false

    //todo 打补丁包时
    enable = true

    // 指定归档目录，默认值当前module的子目录tinker
    autoBackupApkDir = "${bakPath}"

    autoGenerateTinkerId = true

    // 是否启用覆盖tinkerPatch配置功能，默认值false
    // 开启后tinkerPatch配置不生效，即无需添加tinkerPatch

    //todo instant run
//    overrideTinkerPatchConfiguration = false

    //todo 打补丁包时
    overrideTinkerPatchConfiguration = true

    // 编译补丁包时，必需指定基线版本的apk，默认值为空
    // 如果为空，则表示不是进行补丁包的编译
    // @{link tinkerPatch.oldApk }
    baseApk = "${bakPath}/${baseApkDir}/app-release.apk"

    // 对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"

    // 对应tinker插件applyResourceMapping
    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-release-R.txt"

    // 构建基准包和补丁包都要指定不同的tinkerId，并且必须保证唯一性
    //todo 2.tinkerId[常改项]

    //todo 基线版本的tinkerId
    tinkerId = "1.0.1-base"
    //todo 补丁包的tinkerId
//    tinkerId = "1.0.1-patch"

    // 构建多渠道补丁时使用
    // buildAllFlavorsDir = "${bakPath}/${baseApkDir}"

    // 是否开启加固模式，默认为false
    isProtectedApp = true

    // 是否开启反射Application模式
    enableProxyApplication = false

    // 是否支持新增非export的Activity（注意：设置为true才能修改AndroidManifest文件）
    supportHotplugComponent = true
}

/**
 * 一般来说,我们无需对下面的参数做任何的修改
 * 对于各参数的详细介绍请参考:
 * https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97
 */
tinkerPatch {
    //oldApk ="${bakPath}/${appName}/app-release.apk"
    ignoreWarning = false
    useSign = true
    dex {
        dexMode = "jar"
        pattern = ["classes*.dex"]
        loader = []
    }
    lib {
        pattern = ["lib/*/*.so"]
    }

    res {
        pattern = ["res/*", "r/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
        ignoreChange = []
        largeModSize = 100
    }

    packageConfig {
    }
    sevenZip {
        zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
//        path = "/usr/local/bin/7za"
    }
    buildConfig {
        keepDexApply = false
//        tinkerId = "1.0.1-patch"
//        applyMapping = "${bakPath}/${appName}/app-release-mapping.txt" //  可选，设置mapping文件，建议保持旧apk的proguard混淆方式
//        applyResourceMapping = "${bakPath}/${appName}/app-release-R.txt" // 可选，设置R.txt文件，通过旧apk文件保持ResId的分配
    }
}
```

## 第二步：添加插件依赖
Project目录下“build.gradle”文件中添加：

```

buildscript {
    
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath "com.tencent.bugly:tinker-support:1.1.1"
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

## 第三步：集成SDK
app module的“build.gradle”文件中添加：


```
apply plugin: 'com.android.application'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'

    // 多dex配置
    compile "com.android.support:multidex:1.0.1"
    // 远程仓库集成方式（推荐）
    compile 'com.tencent.bugly:crashreport_upgrade:1.3.4'
}

android {
    compileSdkVersion 26

    // recommend
    dexOptions {
        jumboMode = true
    }

    // 签名配置
    signingConfigs {
        release {
            try {
                storeFile file("./keystore/release.keystore")
                storePassword "testres"
                keyAlias "testres"
                keyPassword "testres"
            } catch (ex) {
                throw new InvalidUserDataException(ex.toString())
            }
        }

        debug {
            storeFile file("./keystore/debug.keystore")
        }
    }

    // 默认配置
    defaultConfig {
        applicationId "demo.xiaoxi"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"

        // 开启multidex
        multiDexEnabled true
    }

    // 构建类型
    buildTypes {
        release {
            minifyEnabled true
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            debuggable true
            minifyEnabled false
            signingConfig signingConfigs.debug
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

    repositories {
        flatDir {
            dirs 'libs'
        }
    }

    // 多渠道配置
    /*flavorDimensions "tier"


     productFlavors {
         xiaomi {
             dimension "tier"
         }
         yyb {
             dimension "tier"
         }

         wdj {
             dimension "tier"
         }
     }*/

    lintOptions {
        checkReleaseBuilds false
        abortOnError false
    }
}

// 依赖插件脚本
apply from: 'tinker-support.gradle'


```
<font color="#FF0000">**## 注意2：applicationId "demo.xiaoxi"(你的包名)**</font> 


这时候，你会发现项目报错了，不要紧张，底部有Demo，按着我的步骤来解决它们。

将Demo的资源拷进来，具体内容：


![](https://user-gold-cdn.xitu.io/2018/1/4/160c01f9b15a2178?w=3352&h=2054&f=jpeg&s=772269)
三个红框的内容覆盖到本地文件，复制完后需要修改如下内容：

<font color="#FF0000">**## 注意3：如下修改**</font> 

<font color="#FF0000">**修改1：AndroidManifest.xml的package="com.xiaoxi"(你的包名)，还需要把application的name指向自定义的地址【 android:name=".SampleApplication"】**</font> 

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.xiaoxi">

    <application
        android:allowBackup="true"
        android:name=".SampleApplication"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>
```
<font color="#FF0000">**修改2：布局activity_main.xml的tools:context="com.xiaoxi.MainActivity"(你的包名.MainActivity)**</font> 

<font color="#FF0000">**修改3：SampleApplication的super里面的第二个参数修改成SampleApplicationLike的完整地址【com.xiaoxi.SampleApplicationLike】**</font> 

```
public SampleApplication() {
        super(ShareConstants.TINKER_ENABLE_ALL, "com.xiaoxi.SampleApplicationLike",
                "com.tencent.tinker.loader.TinkerLoader", false);
    }
```
<font color="#FF0000">**修改4：SampleApplicationLike的如下方法的第二个参数改成你在Buggly申请的产品的appid**</font> 

```
Bugly.init(getApplication(), "27a9fcf105", true);
```

<font color="#FF0000">**修改5：Rebuild Project一下**</font> 


![](https://user-gold-cdn.xitu.io/2018/1/4/160c04d5b9cb3830?w=3352&h=2054&f=png&s=525701)

## 第四步：打包发布，打补丁包修复bug

<font color="#9400D3">简单说明一下热更新打包流程：
* 第一步：编译打包发布，一天后，收到bug反馈
* 第二步：修复bug，打出补丁包
* 第三步：发布补丁包，用户收到补丁包后，自动修复bug
</font> 

我们按步骤来操作一下，由于我们对bakApk这个目录还不是很熟悉，所以先Clean Project一下。

<font color="#FF0000">**## 注意4：打包和打补丁包的区别是：**</font> 

<font color="#FF0000">**1.打包是【assembleRelease】**</font> 

<font color="#FF0000">**2.打补丁包是【buildTinkerPatchRelease】**</font> 
<br><br>
* 1.编译打包发布：【assembleRelease】
  
<p style="text-indent:2em">Android Studio右侧Gradle-->app-->Tasks-->build-->assembleRelease：鼠标右键运行
</p>

![](https://user-gold-cdn.xitu.io/2018/1/4/160c067c64515525?w=3336&h=828&f=png&s=405027)

鼠标在app-release.apk上右键，Reveal in Finder 打开apk所在文件夹。然后打开终端，执行 adb install 把这个apk拖进终端，回车

![](https://user-gold-cdn.xitu.io/2018/1/4/160c06c88fbf2433?w=1170&h=732&f=png&s=98060)

<font color="#FF0000">**## 注意5：你需要运行该app，将tinkerId上报到Buggly后台**</font> 

点击显示结果，出现如下界面：

![](https://user-gold-cdn.xitu.io/2018/1/4/160c06eac9521090?w=540&h=960&f=jpeg&s=30810)

* 2.修复bug，打出补丁包
<p style="text-indent:2em">修改代码：BugClass的bug方法里面的最后一行注释解开，然后其它代码注释，如下：
</p>

![](https://user-gold-cdn.xitu.io/2018/1/4/160c07239c8740c6?w=3352&h=2054&f=png&s=788599)

<p style="text-indent:2em">修改tinker-support.gradle，如下：
</p>
def baseApkDir = "app-0104-12-32-46"【将这个值改为刚才assembleRelease生成的目录】

![](https://user-gold-cdn.xitu.io/2018/1/4/160c080911315ef5?w=2204&h=1138&f=jpeg&s=283662)

每个apk的tinkerId是唯一的，那么我们的补丁包也应该有一个不一样的tinkerId，为了便于区分，暂时设为：tinkerId = "1.0.2-patch"

![](https://user-gold-cdn.xitu.io/2018/1/4/160c08329dfd8f9b?w=1780&h=1114&f=jpeg&s=257467)


接下来打补丁包：
 <p style="text-indent:2em">Android Studio右侧Gradle-->app-->tinker-support-->buildTinkerPatchRelease：鼠标右键运行
</p>

![](https://user-gold-cdn.xitu.io/2018/1/4/160c0877de55d77b?w=3352&h=2054&f=png&s=776869)

补丁包：patch_signed_7zip.apk，其它的都是妖魔鬼怪，不要用。

* 3.发布补丁包：【buildTinkerPatchRelease】

鼠标在patch_signed_7zip.apk上右键，Reveal in Finder 打开apk所在文件夹，基线版本发布后，用户安装了，点击按钮闪退，然后我们根据经验，找到了bug，并且也修复了，然后一系列的配置有都完成了，补丁包也打出来了，做了这么多，就以为了一个也许是几kb的小文件，心酸的泪啊，控制不住了，产品那边在催了，先完成工作在哭吧。我们登录Buggly的后台，找到发布补丁包的地方：

![](https://user-gold-cdn.xitu.io/2018/1/4/160c08f3f75280b2)

然后手机app界面等着下发结果：


![](https://user-gold-cdn.xitu.io/2018/1/4/160c0a6e5d6d7601?w=1080&h=1920&f=jpeg&s=71045)

重启应用后再点，【显示结果】至此bug修复。

![](https://user-gold-cdn.xitu.io/2018/1/4/160c0a716ab5faa8?w=1080&h=1920&f=jpeg&s=63336)


![](https://user-gold-cdn.xitu.io/2018/1/4/160c0a826d04726e?w=2506&h=1140&f=png&s=159473)

热修复完成了。

然而我们在调试的情况下，发现instant run安装apk的时候报错了。【这就是我刚才说的，接入Buggly的过程，不要用instant run方式来安装apk】


![](https://user-gold-cdn.xitu.io/2018/1/4/160c0b42117c64a3?w=3352&h=2054&f=png&s=549273)
<font color="#FF0000">**## 注意6：使用instant run方式安装apk需要做如下修改**</font> 

* tinker-support.gradle的tinkerSupport的【enable = true】改为：【enable = false】
* tinker-support.gradle的tinkerSupport的【overrideTinkerPatchConfiguration = true】改为：【overrideTinkerPatchConfiguration = false】
* tinker-support.gradle的tinkerPatch的【//        tinkerId = "1.0.1-patch"】注释解开

```
apply plugin: 'com.tencent.bugly.tinker-support'

def bakPath = file("${buildDir}/bakApk/")

/**
 * 此处填写每次构建生成的基准包目录
 */
//todo 1.配置基准包目录[常改项]
def baseApkDir = "app-0104-18-04-33"

/**
 * 对于插件各参数的详细解析请参考
 */
tinkerSupport {

    // 开启tinker-support插件，默认值true
    //todo instant run
    enable = false

    //todo 打补丁包时
//    enable = true

    // 指定归档目录，默认值当前module的子目录tinker
    autoBackupApkDir = "${bakPath}"

    autoGenerateTinkerId = true

    // 是否启用覆盖tinkerPatch配置功能，默认值false
    // 开启后tinkerPatch配置不生效，即无需添加tinkerPatch

    //todo instant run
    overrideTinkerPatchConfiguration = false

    //todo 打补丁包时
//    overrideTinkerPatchConfiguration = true

    // 编译补丁包时，必需指定基线版本的apk，默认值为空
    // 如果为空，则表示不是进行补丁包的编译
    // @{link tinkerPatch.oldApk }
    baseApk = "${bakPath}/${baseApkDir}/app-release.apk"

    // 对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"

    // 对应tinker插件applyResourceMapping
    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-release-R.txt"

    // 构建基准包和补丁包都要指定不同的tinkerId，并且必须保证唯一性
    //todo 2.tinkerId[常改项]

    //todo 基线版本的tinkerId
    tinkerId = "1.0.1-base"
    //todo 补丁包的tinkerId
//    tinkerId = "1.0.1-patch"

    // 构建多渠道补丁时使用
    // buildAllFlavorsDir = "${bakPath}/${baseApkDir}"

    // 是否开启加固模式，默认为false
    isProtectedApp = true

    // 是否开启反射Application模式
    enableProxyApplication = false

    // 是否支持新增非export的Activity（注意：设置为true才能修改AndroidManifest文件）
    supportHotplugComponent = true
}

/**
 * 一般来说,我们无需对下面的参数做任何的修改
 * 对于各参数的详细介绍请参考:
 * https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97
 */
tinkerPatch {
    //oldApk ="${bakPath}/${appName}/app-release.apk"
    ignoreWarning = false
    useSign = true
    dex {
        dexMode = "jar"
        pattern = ["classes*.dex"]
        loader = []
    }
    lib {
        pattern = ["lib/*/*.so"]
    }

    res {
        pattern = ["res/*", "r/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
        ignoreChange = []
        largeModSize = 100
    }

    packageConfig {
    }
    sevenZip {
        zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
//        path = "/usr/local/bin/7za"
    }
    buildConfig {
        keepDexApply = false
        tinkerId = "1.0.1-patch"
//        applyMapping = "${bakPath}/${appName}/app-release-mapping.txt" //  可选，设置mapping文件，建议保持旧apk的proguard混淆方式
//        applyResourceMapping = "${bakPath}/${appName}/app-release-R.txt" // 可选，设置R.txt文件，通过旧apk文件保持ResId的分配
    }
}
```
*  tinker不支持instant run模式，你需要找到File->Settings->Build,Execution,Deployment->instant run并关闭，日常调试可以tinker关闭来使用instant run。

![](https://user-gold-cdn.xitu.io/2018/1/4/160c0c1eb2c12c2d?w=2044&h=1352&f=png&s=280628)

配置完成后，就可以instant run了：

![](https://user-gold-cdn.xitu.io/2018/1/4/160c0e4031511c48?w=3352&h=2054&f=png&s=534359)

Demo：<a href="https://github.com/zhouhui520w/XiaoXiHotFixDemo">https://github.com/zhouhui520w/XiaoXiHotFixDemo</a>
