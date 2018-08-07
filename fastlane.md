---
layout: page
title: 通过fastlane持续交付
description: 如何通过Fastlane自动连续构建和发布Flutter应用程序。

permalink: /fastlane-cd/
---

Follow continuous delivery best practices with Flutter to make sure your
application is delivered to your beta testers and validated on a frequent basis
without resorting to manual workflows.

Flutter遵循持续交付最佳实践能确保您的应用在验收测试时能快速生效并且不需要手动工作流，



This guide shows how to integrate [Fastlane](https://docs.fastlane.tools/), an
open-source tool suite, with your existing testing and continuous integration
(CI) workflows (for example, Travis or Cirrus).

本指南介绍如何集成[Fastlane]（https://docs.fastlane.tools/）,一个开源工具套件，具有您现有的测试和持续集成 （CI）工作流（类似的还有 Travis和 Cirrus）。



* TOC Placeholder
{:toc}

## Local setup 本地设置

It's recommended that you test the build and deployment process locally before
migrating to a cloud-based system. You could also choose to perform continuous
delivery from a local machine.

建议你在迁移到云端系统之前先在本地测试一遍构建和部署流程。您也可以设置持续交付在本地执行。



1. Install Fastlane `gem install fastlane` or `brew cask install fastlane`.

1. 通过`gem install fastlane`或`brew cask install fastlane`命令安装Fastlane。

1. Create your Flutter project, and when ready, make sure that your project builds via
    * ![Android](/images/fastlane-cd/android.png) `flutter build apk --release`; and
    * ![iOS](/images/fastlane-cd/ios.png) `flutter build ios --release --no-codesign`.

1. 创建Flutter项目后，一切准备完毕，执行以下指令确保项目编译通过
    * ![Android](/images/fastlane-cd/android.png) `flutter build apk --release`; 
    * ![iOS](/images/fastlane-cd/ios.png) `flutter build ios --release --no-codesign`.
  


1. Initialize the Fastlane projects for each platform.
    * ![Android](/images/fastlane-cd/android.png) In your `[project]/android`
    directory, run `fastlane init`.
    * ![iOS](/images/fastlane-cd/ios.png) In your `[project]/ios` directory,
    run `fastlane init`.

1. 初始化不同平台的Fastlane工程。
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

编辑Appfile文件，确保他们有提供合适的元素对您的应用。

 * ![Android](/images/fastlane-cd/android.png)检查`[project]/android/Appfile`里的`package_name`和在pubspec.yaml里的包名一致。

* ![iOS](/images/fastlane-cd/ios.png) 检查`[project]/ios/Appfile`和`app_identifier`的一致。填写个人账户信息`apple_id`, `itc_team_id`,`team_id`。


1. Set up your local login credentials for the stores.

    * ![Android](/images/fastlane-cd/android.png) Follow the [Supply setup steps](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply)
    and ensure that `fastlane supply init` successfully syncs data from your
    Play Store console. _Treat the .json file like your password and do not check
    it into any public source control repositories._

    * ![iOS](/images/fastlane-cd/ios.png) Your iTunes Connect username is already
    in your `Appfile`'s `apple_id` field. Set the `FASTLANE_PASSWORD` shell
    environment variable with your iTunes Connect password. Otherwise, you'll be
    prompted when uploading to iTunes/TestFlight.

1. 设置应用商店的登录凭证。

 * ![Android](/images/fastlane-cd/android.png)按照[设置步骤](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply)确保`fastlane supply init`成功同步在您的应用商店控制台。把.json文件像你的密码一样对待，不要上传到任何公共的代码仓库。

* ![iOS](/images/fastlane-cd/ios.png) 你的iTunes链接用户名已经在你的'Appfile的'apple_id字段中。设置`FASTLANE_PASSWORD`shell环境变量与您的iTunes Connect密码。否则，你上传到iTunes / TestFlight会得到提示。



1. Set up code signing.

1.设置代码签名。

    * ![Android](/images/fastlane-cd/android.png) On Android, there are two
    signing keys: deployment and upload. The end-users download the .apk signed
    with the 'deployment key'. An 'upload key' is used to authenticate the .apk
    uploaded by developers onto the Play Store and is re-signed with the
    deployment key once in the Play Store.

* ![Android](/images/fastlane-cd/android.png) 在Android中，有两种签名密钥:部署和上传。最终用户下载的apk签的是“部署密钥”。“上传密钥”用于验证开发人员上传apk到应用商店和在应用商店重新签名。


        * It's highly recommended to use the automatic cloud managed signing for
        the deployment key. For more information, see the [official Play Store documentation](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en).
*强烈建议使用自动云托管签名 部署密钥。更多详细信息，请参阅[官方应用商店文档](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en)。


        * Follow the [key generation steps](https://developer.android.com/studio/publish/app-signing#sign-apk)
        to create your upload key.
* 按照[密钥生成步骤](https://developer.android.com/studio/publish/app-signing#sign-apk)创建上传密钥。


        * Configure gradle to use your upload key when building your app in
        release mode by editing `android.buildTypes.release` in
        `[project]/android/app/build.gradle`.
*在gradle配置在release模式下使用上传密钥构建app，通过编辑`[project]/android/app/build.gradle`里的android.buildTypes.release。


    * ![iOS](/images/fastlane-cd/ios.png) On iOS, create and sign using a
    distribution certificate instead of a development certificate when you're
    ready to test and deploy using TestFlight or App Store.

  * ![iOS](/images/fastlane-cd/ios.png)在iOS上，当你准备通过TestFlight和App Store 测试和部署时应该用发行证书而不是开发证书。

        * Create and download a distribution certificate in your [Apple Developer Account console](https://developer.apple.com/account/ios/certificate/).
创建和下载一个发布证书在你的[Apple Developer Account console](https://developer.apple.com/account/ios/certificate/)。

        * `open [project]/ios/Runner.xcworkspace/` and select the distribution
        certificate in your target's settings pane.

*  `open [project]/ios/Runner.xcworkspace/`和选择发布证书在你setting面板。



1. Create a `Fastfile` script for each platform.

1. 为不同平台创建一个Fastfile脚本。

    * ![Android](/images/fastlane-cd/android.png) On Android, follow the
    [Fastlane Android beta deployment guide](https://docs.fastlane.tools/getting-started/android/beta-deployment/).

* ![Android](/images/fastlane-cd/android.png) 按照 [Fastlane Android beta版部署指南](https://docs.fastlane.tools/getting-started/android/beta-deployment/)。

    Your edit could be as simple as adding a `lane` that calls `upload_to_play_store`.
你的编辑可以像通过调用`upload_to_play_store`添加一个`lane`一样简单。

    Set the `apk` argument to `../build/app/outputs/apk/release/app-release.apk`
    to use the apk `flutter build` already built.

 将apk参数设置到`../build/app/outputs/apk/release/app-release.apk`用`flutter build`来用已经编译好的apk。


    * ![iOS](/images/fastlane-cd/ios.png) On iOS, follow the [Fastlane iOS beta deployment guide](https://docs.fastlane.tools/getting-started/ios/beta-deployment/).
* ![iOS](/images/fastlane-cd/ios.png)在iOS，依照[Fastlane iOS beta版部署指南](https://docs.fastlane.tools/getting-started/ios/beta-deployment/)。


    Your edit could be as simple as adding a `lane` that calls `build_ios_app` with
    `export_method: 'app-store'` and `upload_to_testflight`. On iOS an extra
    build is required since `flutter build` builds an .app rather than archiving
    .ipas for release.

你可以向添加一个`lane`一样简单编辑调用`build_ios_app`




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


1. Run the Fastfile script on each platform.


    * ![Android](/images/fastlane-cd/android.png) `cd android` then
    `fastlane [name of the lane you created]`.


    * ![iOS](/images/fastlane-cd/ios.png) `cd ios` then
    `fastlane [name of the lane you created]`.



## Cloud build and deploy setup



First, follow the local setup section described in 'Local setup' to make sure
the process works before migrating onto a cloud system like Travis.



The main thing to consider is that since cloud instances are ephemeral and
untrusted, you won't be leaving your credentials like your Play Store service
account JSON or your iTunes distribution certificate on the server.



CI systems, such as [Travis](https://docs.travis-ci.com/user/environment-variables/#Encrypting-environment-variables)
or [Cirrus](https://cirrus-ci.org/guide/writing-tasks/#encrypted-variables)
generally support encrypted environment variables to store private data.




**Take precaution not to re-echo those variable values back onto the console in
your test scripts**. Those variables are also not available in pull requests
until they're merged to ensure that malicious actors cannot create a pull
request that prints these secrets out. Be careful with interactions with these
secrets in pull requests that you accept and merge.




1. Make login credentials ephemeral.



    * ![Android](/images/fastlane-cd/android.png) On Android:
        * Remove the `json_key_file` field from `Appfile` and store the string
        content of the JSON in your CI system's encrypted variable. Use the
        `json_key_data` argument in `upload_to_play_store` to read the
        environment variable directly in your `Fastfile`.



        * Serialize your upload key (for example, using base64) and save it as
        an encrypted environment variable. You can deserialize it on your CI
        system during the install phase with


        ```bash
        echo "$PLAY_STORE_UPLOAD_KEY" | base64 --decode > /home/travis/[directory and filename specified in your gradle].keystore
        ```



    * ![iOS](/images/fastlane-cd/ios.png) On iOS:


        * Move the local environment variable `FASTLANE_PASSWORD` to use
        encrypted environment variables on the CI system.


        * The CI system needs access to your distribution certificate. Fastlane's
        [Match](https://docs.fastlane.tools/actions/match/) system is
        recommended to synchronize your certificates across machines.




2. It's recommended to use a Gemfile instead of using an indeterministic
`gem install fastlane` on the CI system each time to ensure the Fastlane
dependencies are stable and reproducible between local and cloud machines. However, this step is optional.


    * In both your `[project]/android` and `[project]/ios` folders, create a
    `Gemfile` containing the following content:


      ```
      source "https://rubygems.org"

      gem "fastlane"
      ```


    * In both directories, run `bundle update` and check both `Gemfile` and
    `Gemfile.lock` into source control.

在两个目录中，运行`bundle update`并检查`Gemfile`和
`Gemfile.lock`两个文件进入源代码管理。



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


         * Make sure the Flutter SDK is available and set in `PATH`.


    * In the script phase of the CI task:


         * Run `flutter build apk --release` or `flutter build ios --release --no-codesign` depending on the platform.


         * `cd android` or `cd ios`.


         * `bundle exec fastlane [name of the lane]`.

## Reference
参照


The [Flutter Gallery in the Flutter repo](https://github.com/flutter/flutter/tree/master/examples/flutter_gallery)
uses Fastlane for continuous deployment. See the source for a working example of
Fastlane in action. The Travis script is [here](https://github.com/flutter/flutter/blob/master/.travis.yml).

[Flutter Gallery in the Flutter repo](https://github.com/flutter/flutter/tree/master/examples/flutter_gallery)通过Fastlane持续部署。参见一个Flutter项目的示例代码，其中travis脚本[在此](https://github.com/flutter/flutter/blob/master/.travis.yml)。


