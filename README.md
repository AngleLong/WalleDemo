>在项目中，你有没有遇到过这样的问题。每次要上线的时候，都会自己在开发的分支上进行测试，然后打个测试的环境的包，交到测试手中去进行测试！然后测试说没有问题了，之后在到生产上进行相应的测试！这样周而复始，往往需要切换相应的网络地址，各种相应的配置，有的时候忘记点什么简直就是灾难性的打击！线上出现各种问题，最近在看了美团开源的wall多渠道打包方案，受了比较大的启发，所以才有了今天的博客！

# 本文知识点
- 多版本的配置和打包
- walle的配置和打包
- 关于我碰到的一些问题


## 多版本的配置和打包
> 每次我写博客的时候，都会先从小白的角度去思考怎么样才能使用这个东西，然后才会去深入的了解怎么去实现的！好了准备开车了。。。

在你对应主module的build.gradle的android标签内添加相应的属性！
```
    productFlavors {
        //开发环境
        develop {
            buildConfigField "String", "ENV_TYPE", "\"1\""
            applicationId 'xxx'
            manifestPlaceholders = [
                    app_name     : "开发",
            ]
        }
        //测试环境
        check {
            buildConfigField "String", "ENV_TYPE", "\"2\""
            applicationId 'xxx'
            manifestPlaceholders = [
                    app_name     : "测试",
            ]
        }
        //生产环境
        product {
            buildConfigField "String", "ENV_TYPE", "\"3\""
            applicationId 'xxx'
            manifestPlaceholders = [
                    app_name     : "xxx",
            ]
        }
    }
```

这里说明几个相关的问题和注意点：
- buildConfigField 这里配置的是相关的一个属性，可以怎么理解呢？就是你之后要根据它去区分相应的状态，还有这里如果是String的话，后面一定要跟着转译字符，因为你不去设置这个转译字符的话，它会生成一个int类型的内容，所以这里注意一下就好了！
- applicationId 有很多人根据这个设置相应的包名，从而使APP能安装多个，让测试能同时安装多个app一起测试，但是我发现有一个问题，如果这样修改的话，像什么激光推送之类的，就会受到相应的影响，所以这里只能辛苦测试一个一个安装了！不怀好意的微笑😅！！！如果哪位看官有更好的方案，可以告诉小弟一声，小弟不胜感谢！！！
- manifestPlaceholders 这个是配置一些基本属性的，
- app_name app显示的名称
- icon 配置app的logo（我这里没写，但是这个属性，可以使用的）


- 配置相应的AndroidManifest.xml
> 上面你配置了app_name 如果你不配置相应的xml是不会生效的！

```
android:label="${app_name}"
```

你需要在label中设置这个，才能让app的名称进行更改，如果你要是设置了icon的话，也是要像上面这样修改的！我就不在多废话了！

![cf55594249d04dc0052d460ae994f0d8.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190703_1.png)

**这个时候，你要做的是Sync Now，然后你会发现在这个文件路径下看到相应的java文件**

![ff12b2ce47ef4c4908dcb4614644d68c.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190703_3.png)

你会发现，这个和你在主module的build.gradle中设置的内容差不多，其实无非就是apt写到项目中的！

这个时候，所有的配置都已经配置完成了，这个时候，你要根据相应的类型去设置对应的处理逻辑了！

- 相应的处理逻辑

```
//获取相应的类型，这里其实就是根据你打包生成的类型进行获取的
String evnType = BuildConfig.ENV_TYPE;
```

这里我建议单写一个类，这个类主要的目的是区分上面写的这些类型的！

```
public class EnvType {
    /**
     * 开发环境
     */
    public static final String DEVELOP = "1";
    /**
     * 测试环境
     */
    public static final String CHECK = "2";
    /**
     * 正式环境
     */
    public static final String PRODUCT = "3";
}
```

这样获取到相应的类型，switch一判断，搞定！

- 打包的细节
> 如果你正常打包的话，你会看到相应的渠道包，但是我毕竟是一个懒的不能再懒的人，这个我都闲麻烦怎么办，当然有办法！

![96f8848a09b5d08de1088343538341b4.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190703_4.png)

点这个，完美的打出相应的渠道包，真的是哪里不会点哪里啊！

![bc7b8a37246c15257c6f0f5393d3f59a.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190703_6.png)

然后你会在相应的路径看到对应的渠道包，不用感谢我！！！哈哈。。。

## walle的配置和打包
在写本文的时候，github上最新的版本是1.1.6，所以这里就拿它祭我的5米大刀吧！

- 在项目的build.gradle中添加配置相应的插件！
![82ce0685a8304b971931ac2f479a52bc.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190703_7.png)
上面注释已经写上了，所以我也就不多逼逼了
- 在相应主module中添加依赖的插件以及相应的引用
```
//添加相应渠道的aar
apply plugin: 'walle'
```
```
/*多渠道的库*/
implementation 'com.meituan.android.walle:library:1.1.6'
```
其实就是在app的build.gradle中添加这个，其实和“butterknife”差不多的配置，都是apt的一些思想而已！
- 配置相应的输出
> 这个才是重点，为什么这么说呢？因为这个关乎到你配置的内容是怎么输出的！
```
walle {
    // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels");
    // 定制渠道包的APK的文件名称
    apkFileNameFormat = '${appName}-${packageName}-${channel}-${buildType}-v${versionName}-${versionCode}-${buildTime}.apk';
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```

这里面做一下配置项的说明：
- apkOutputFolder：指定渠道包的输出路径， 默认值为new File("${project.buildDir}/outputs/apk")
- apkFileNameFormat：定制渠道包的APK的文件名称, 默认值为'${appName}-${buildType}-${channel}.apk'
- 可使用以下变量:
     projectName - 项目名字
     appName - App模块名字
     packageName - applicationId (App包名packageName)
     buildType - buildType (release/debug等)
     channel - channel名称 (对应渠道打包中的渠道名字)
     versionName - versionName (显示用的版本号)
     versionCode - versionCode (内部版本号)
     buildTime - buildTime (编译构建日期时间)
     fileSHA1 - fileSHA1 (最终APK文件的SHA1哈希值)
     flavorName - 编译构建 productFlavors 名
- channelFile：包含渠道配置信息的文件路径。 具体内容格式详见：渠道配置文件示例，支持使用#号添加注释。

这里只要注意几点问题就好了：
- 相应的输出路径
- 渠道名称（其实这里可以不用这么复杂的，这里写这么多，是因为官方文档上就写这么多，其实你可以删减一下的）
- channel文件 这个应该放在主module的下面，要不你要改地址的！
```
anzhi #安智
baidu #百度
...
yyb #应用宝
```

配置基本上就这么多了，基本上就能满足你的需求了！官方上面还有添加额外信息的地方，但是我觉得没有什么卵用😂，所以我就没去弄！

关于这个打包其实你应该可以使用命令行，但是我觉得这样很喽，所以呢？你这样点一个按钮，全部搞定！

![e0d2e67af4e71fd8a1eeb652e73e552a.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190704_8.png)

注意我圈起来的这个东西，这个是正式版的构建，你要拿正式版的构建才好使，测试版的你没有必要打渠道包！你觉得呢？（☺️😒☺️夹在产品中的我）

点一下就好了，紧接着你会啊在这个路径下看到你想要的东西！

![2cc87acb06fefe53748376913959d1eb.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190704_9.png)

## 关于我碰到的一些问题
- 错误：All flavors must now belong to a named flavor dimension.
关于这个错误网上有很多办法，基本上就是在下图的位置添加我圈起来的一句话
![309a59641a87c362854c6049c8ac8438.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190704_10.png)

- 错误：Plugin requires 'APK Signature Scheme v2 Enabled' for ProductRelease.
关于这个错误是你没有在项目中配置相应的jks文件配置上就好了！
![0042820608a5e215d5894ea327ec96cc.png](https://github.com/AngleLong/WalleDemo/blob/master/img/Snip20190704_11.png)

- 命令行无法成功打包
> 其实这个问题我也纠结了好久，为什么呢？我按照github的说法敲了命令行，但是怎么也不好使。最后我发现了一件事情，其实多渠道的时候，已经生成了相应的assemble，所以按照github上就无法完成打包了。换成上面截图的assembleXXXReleaseChannels就可以了！注意一下相应的明明就可以了！

---
好了今天要分享的就这么多了！希望对您能有帮助！感谢你抽出宝贵的时间阅读！！！