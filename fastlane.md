---
layout: page
title: 通过fastlane实现Flutter应用的持续交付
description: 介绍如何通过Fastlane自动化连续构建和发布Flutter应用程序。

permalink: /fastlane-cd/
---

Follow continuous delivery best practices with Flutter to make sure your
application is delivered to your beta testers and validated on a frequent basis
without resorting to manual workflows.

下面介绍的Flutter持续交付最佳实践能确保您的应用能持续beta提测时能快速生效并且不需要介入手动操作，



This guide shows how to integrate [Fastlane](https://docs.fastlane.tools/), an
open-source tool suite, with your existing testing and continuous integration
(CI) workflows (for example, Travis or Cirrus).

本指南介绍如何集成[Fastlane]（https://docs.fastlane.tools/）,一个开源工具套件，提供现成的的测试和持续集成 （CI）工作流（类似的还有 Travis和 Cirrus）。



* TOC Placeholder
{:toc}

## Local setup 本地设置

It's recommended that you test the build and deployment process locally before
migrating to a cloud-based system. You could also choose to perform continuous
delivery from a local machine.

建议先在本地环境构建和部署流程再迁移到云端系统。您也可以选择本地持续交付。



1. Install Fastlane `gem install fastlane` or `brew cask install fastlane`.
 

1. 通过`gem install fastlane`或`brew cask install fastlane`指令安装Fastlane。



1. Create your Flutter project, and when ready, make sure that your project builds via
    * ![Android](/images/fastlane-cd/android.png) `flutter build apk --release`; and
    * ![iOS](/images/fastlane-cd/ios.png) `flutter build ios --release --no-codesign`.

1. 创建Flutter项目后，开发完成后，执行以下指令确保项目编译通过。
    * ![Android](/images/fastlane-cd/android.png) `flutter build apk --release`; 
    * ![iOS](/images/fastlane-cd/ios.png) `flutter build ios --release --no-codesign`.
  


1. Initialize the Fastlane projects for each platform.
    * ![Android](/images/fastlane-cd/android.png) In your `[project]/android`
    directory, run `fastlane init`.
    * ![iOS](/images/fastlane-cd/ios.png) In your `[project]/ios` directory,
    run `fastlane init`.

1. 初始化Fastlane项目。
    *![Android](/images/fastlane-cd/android.png)在你的`[project]/android`
    目录下，运行`fastlane init`。
    *![iOS](/images/fastlane-cd/ios.png)在你的`[project]/ios`目录下，
    运行`fastlane init`。


1. Edit the `Appfile`s to ensure they have adequate metadata for your app.
    * ![Android](/images/fastlane-cd/android.png) Check that `package_name` in
    `[project]/android/Appfile` matches your package in pubspec.yaml.
    * ![iOS](/images/fastlane-cd/ios.png) Check that `app_identifier` in
    `[project]/ios/Appfile` also matches. Fill in `apple_id`, `itc_team_id`,
    `team_id` with your respective account info.

检查Appfile文件，确保里面有合适字段来描述你的应用。

 * ![Android](/images/fastlane-cd/android.png)`[project]/android/Appfile`里的`package_name`要和pubspec.yaml里的包名相匹配。

* ![iOS](/images/fastlane-cd/ios.png) `[project]/ios/Appfile`里`app_identifier`字段也能匹配得上。并填写和个人账户信息相关的：`apple_id`, `itc_team_id`,`team_id`。


1. Set up your local login credentials for the stores.

    * ![Android](/images/fastlane-cd/android.png) Follow the [Supply setup steps](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply)
    and ensure that `fastlane supply init` successfully syncs data from your
    Play Store console. _Treat the .json file like your password and do not check
    it into any public source control repositories._

    * ![iOS](/images/fastlane-cd/ios.png) Your iTunes Connect username is already
    in your `Appfile`'s `apple_id` field. Set the `FASTLANE_PASSWORD` shell
    environment variable with your iTunes Connect password. Otherwise, you'll be
    prompted when uploading to iTunes/TestFlight.

1. 设置发布到应用商店的本地签名文件。

 * ![Android](/images/fastlane-cd/android.png)按照文章[“设置步骤”](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply)确保`fastlane supply init`指令成功同步了您在应用商店的数据。把.json文件像密码一样对待，不要提交到任何公共代码仓库。

* ![iOS](/images/fastlane-cd/ios.png) 你的连接到iTunes的用户名已经保存在'Appfile里的'apple_id字段。设置`FASTLANE_PASSWORD` shell环境变量为你的iTunes连接密码。否则，当你上传到iTunes / TestFlight时会得到提示。



1. Set up code signing.

1.设置代码签名。

    * ![Android](/images/fastlane-cd/android.png) On Android, there are two
    signing keys: deployment and upload. The end-users download the .apk signed
    with the 'deployment key'. An 'upload key' is used to authenticate the .apk
    uploaded by developers onto the Play Store and is re-signed with the
    deployment key once in the Play Store.

* ![Android](/images/fastlane-cd/android.png) 在Android中，分为两种类型的签名密钥:部署密钥和上传密钥。最终用户下载的apk签的是“部署密钥”。“上传密钥”用作开发人员上传apk在应用商店已经部署过的应用 和在应用商店重新签名时的权限验证。


        * It's highly recommended to use the automatic cloud managed signing for
        the deployment key. For more information, see the [official Play Store documentation](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en).
*强烈建议通过云自动托管签名的部署密钥。更多详细信息，请参阅-[官方应用商店文档](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en)。


        * Follow the [key generation steps](https://developer.android.com/studio/publish/app-signing#sign-apk)
        to create your upload key.
* 按照[密钥生成步骤](https://developer.android.com/studio/publish/app-signing#sign-apk)创建和上传密钥。


* Configure gradle to use your upload key when building your app in
        release mode by editing `android.buildTypes.release` in
        `[project]/android/app/build.gradle`.
*在gradle配置中设置release模式下通过上传密钥构建app，通过编辑`[project]/android/app/build.gradle`里的android.buildTypes.release。


    * ![iOS](/images/fastlane-cd/ios.png) On iOS, create and sign using a
    distribution certificate instead of a development certificate when you're
    ready to test and deploy using TestFlight or App Store.

  * ![iOS](/images/fastlane-cd/ios.png)在iOS上，当你准备用TestFlight或App Store测试和部署应用时用的应该是发行证书而不是开发证书。

        * Create and download a distribution certificate in your [Apple开发者账户控制台](https://developer.apple.com/account/ios/certificate/).

在你的[Apple开发者账户控制台](https://developer.apple.com/account/ios/certificate/)创建和下载发行证书。

        * `open [project]/ios/Runner.xcworkspace/` and select the distribution
        certificate in your target's settings pane.

*  打开 `[project]/ios/Runner.xcworkspace/`选择发布证书在你setting面板。



1. Create a `Fastfile` script for each platform.

1. 为不同平台创建一个Fastfile脚本。

    * ![Android](/images/fastlane-cd/android.png) On Android, follow the
    [Fastlane Android beta deployment guide](https://docs.fastlane.tools/getting-started/android/beta-deployment/).

* ![Android](/images/fastlane-cd/android.png) 按照 [Fastlane Android beta版部署指南](https://docs.fastlane.tools/getting-started/android/beta-deployment/)。

    Your edit could be as simple as adding a `lane` that calls `upload_to_play_store`.
你的编辑可以添加一个`lane`调用`upload_to_play_store`一样简单。

    Set the `apk` argument to `../build/app/outputs/apk/release/app-release.apk`
    to use the apk `flutter build` already built.

 将apk参数设置到`../build/app/outputs/apk/release/app-release.apk`用`flutter build`来用已经编译好的apk。


    * ![iOS](/images/fastlane-cd/ios.png) On iOS, follow the [Fastlane iOS beta deployment guide](https://docs.fastlane.tools/getting-started/ios/beta-deployment/).

* ![iOS](/images/fastlane-cd/ios.png)在iOS，根据[Fastlane iOS beta版部署指南](https://docs.fastlane.tools/getting-started/ios/beta-deployment/)一步步操作。


    Your edit could be as simple as adding a `lane` that calls `build_ios_app` with
    `export_method: 'app-store'` and `upload_to_testflight`. On iOS an extra
    build is required since `flutter build` builds an .app rather than archiving
    .ipas for release.

你可以向添加一个`lane`一样简单编辑调用`build_ios_app`
在iOS一个额外的编译是被需要的自从`flutter build`构建一个.app而不是 存档一个release版的.ipa。



You're now ready to perform deployments locally or migrate the deployment
process to a continuous integration (CI) system.

现在您已准备好在本地执行部署或迁移部署 过程到持续集成（CI）系统。



## Running deployment locally
在本地运行部署



1. Build the release mode app.
1.构建release模式应用程序。

    * ![Android](/images/fastlane-cd/android.png) `flutter build apk --release`.


    * ![iOS](/images/fastlane-cd/ios.png) `flutter build ios --release --no-codesign`.


    No need to sign now since Fastlane will sign when archiving.
	这个阶段不需要签名，Fastlane将在归档时签名。

1. Run the Fastfile script on each platform.

在不同的平台下运行Fastfile脚本


    * ![Android](/images/fastlane-cd/android.png) `cd android` then
    `fastlane [name of the lane you created]`.


    * ![iOS](/images/fastlane-cd/ios.png) `cd ios` then
    `fastlane [name of the lane you created]`.



## Cloud build and deploy setup

配置云构建和云部署


First, follow the local setup section described in 'Local setup' to make sure
the process works before migrating onto a cloud system like Travis.

首先，根据上面”本地配置“部分对”本地配置“的描述确保 在迁移到一个类似Travis云系统之前 能运作起来。

The main thing to consider is that since cloud instances are ephemeral and
untrusted, you won't be leaving your credentials like your Play Store service
account JSON or your iTunes distribution certificate on the server.



CI systems, such as [Travis](https://docs.travis-ci.com/user/environment-variables/#Encrypting-environment-variables)
or [Cirrus](https://cirrus-ci.org/guide/writing-tasks/#encrypted-variables)
generally support encrypted environment variables to store private data.

在CI系统中，像[Travis](https://docs.travis-ci.com/user/environment-variables/#Encrypting-environment-variables)和[Cirrus](https://cirrus-ci.org/guide/writing-tasks/#encrypted-variables)都支持通过加密的环境变量保持私有数据。




**Take precaution not to re-echo those variable values back onto the console in
your test scripts**. Those variables are also not available in pull requests
until they're merged to ensure that malicious actors cannot create a pull
request that prints these secrets out. Be careful with interactions with these
secrets in pull requests that you accept and merge.


**特别注意不要在测试脚本将这些变量重新返回到控制台中。**这些变量在pull requests被合并之前都是中是不可读取的，以防止有人恶意创建一个新的拉取请求打印这些秘密数据。当接收和合并pull requests时记得小心对待这些秘密数据。



1. Make login credentials ephemeral.

使用登录临时证书

    * ![Android](/images/fastlane-cd/android.png) On Android:
        * Remove the `json_key_file` field from `Appfile` and store the string
        content of the JSON in your CI system's encrypted variable. Use the
        `json_key_data` argument in `upload_to_play_store` to read the
        environment variable directly in your `Fastfile`.

Android：
替换`Appfile`中的`json_key_file`字段为在你的CI系统中变量加密后生成的JSON字符串内容。通过
 `upload_to_play_store`的`json_key_data`参数直接读取你的`Fastfile`的环境变量。

        * Serialize your upload key (for example, using base64) and save it as
        an encrypted environment variable. You can deserialize it on your CI
        system during the install phase with

序列化你的上传密钥（比如可以用Base64）并将其保存为加密后的环境变量。你可以在CI系统安装时期反通过以下命令序列化它。

        ```bash
        echo "$PLAY_STORE_UPLOAD_KEY" | base64 --decode > /home/travis/[directory and filename specified in your gradle].keystore
        ```



    * ![iOS](/images/fastlane-cd/ios.png) On iOS:

iOS：
        * Move the local environment variable `FASTLANE_PASSWORD` to use
        encrypted environment variables on the CI system
.
替换`FASTLANE_PASSWORD`本地环境变量，使用CI系统加密后的环境变量。

        * The CI system needs access to your distribution certificate. Fastlane's
        [Match](https://docs.fastlane.tools/actions/match/) system is
        recommended to synchronize your certificates across machines.

CI系统需要你发行证书的权限。推荐使用Fastlane的[Match](https://docs.fastlane.tools/actions/match/)系统在不同机器之间同步你的证书。




2. It's recommended to use a Gemfile instead of using an indeterministic
`gem install fastlane` on the CI system each time to ensure the Fastlane
dependencies are stable and reproducible between local and cloud machines. However, this step is optional.

推荐使用Gemfile来确保Fastlane在本地和在云端机器上的依赖是稳定和可重复的，而不是每次都重新`gem install fastlane`来在不同的CI系统部署。然而，这个步骤是并不是强制的。


    * In both your `[project]/android` and `[project]/ios` folders, create a
    `Gemfile` containing the following content:

无论在Android `[project]/android`的目录下还是 iOS `[project]/ios`的目录下，创建一个`Gemfile`都要包含以下内容：
      ```
      source "https://rubygems.org"

      gem "fastlane"
      ```


    * In both directories, run `bundle update` and check both `Gemfile` and
    `Gemfile.lock` into source control.

在上面的目录下，运行`bundle update`并提交 `Gemfile` 和`Gemfile.lock`到版本控制。

 



    * When running locally, use `bundle exec fastlane` instead of `fastlane`.
在本地运行时，使用`bundle exec fastlane`而不是`fastlane`。


3. Create the CI test script such `.travis.yml` or `.cirrus.yml` in your
repository root.

创建名如`.travis.yml`或`.cirrus.yml`的CI测试脚步在工程仓库的根目录。

    * Shard your script to run on both Linux and OSX platforms.

    这个脚本在Linux和OSX平台都可以运行。

    * Remember to specify a dependency on Xcode for OSX (for example
    `osx_image: xcode9.2`).
    
    记得声明OSX需要Xcode的最低版本（例如：`osx_image: xcode9.2`）。



    * See [Fastlane CI documentation](https://docs.fastlane.tools/best-practices/continuous-integration/)
    for CI specific setup.


    * During the setup phase, depending on the platform, make sure that:


         * Bundler is available using `gem install bundler`.


         * For Android, make sure the Android SDK is available and the `ANDROID_HOME`
         path is set.



         * Run `bundle install` in `[project]/android` or `[project]/ios`.
 android的在`[project]/android`目录下运行`bundle install`

         * Make sure the Flutter SDK is available and set in `PATH`.
确保`PATH`配置了Flutter SDK的地址可用。

    * In the script phase of the CI task:

在执行CI任务脚本解析
         * Run `flutter build apk --release` or `flutter build ios --release --no-codesign` depending on the platform.
根据不同依赖的平台，在Android下运行`flutter build apk --release`，在iOS下运行`flutter build ios --release --no-codesign`。


         * `cd android` or `cd ios`.



         * `bundle exec fastlane [name of the lane]`.

## Reference
参照


The [Flutter Gallery in the Flutter repo](https://github.com/flutter/flutter/tree/master/examples/flutter_gallery)
uses Fastlane for continuous deployment. See the source for a working example of
Fastlane in action. The Travis script is [here](https://github.com/flutter/flutter/blob/master/.travis.yml).

[Flutter Gallery in the Flutter repo](https://github.com/flutter/flutter/tree/master/examples/flutter_gallery)通过Fastlane持续部署。参见一个Flutter项目的示例代码，其中travis脚本[在此](https://github.com/flutter/flutter/blob/master/.travis.yml)。


