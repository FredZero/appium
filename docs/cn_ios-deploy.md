部署ios app 到手机上
=====================================
准备在真机上执行appium测试, 需要如下准备:

1. 用特殊的设备参数来构建app
1. 使用 [fruitstrap](https://github.com/ghughes/fruitstrap), 一个第三方程序，来部署你构建的app到手机上

## Xcodebuild 命令的参数:
新的参数运行指定设置. 参考 [developer.apple.com](https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html):

```
xcodebuild [-project projectname] [-target targetname ...]
             [-configuration configurationname] [-sdk [sdkfullpath | sdkname]]
             [buildaction ...] [setting=value ...] [-userdefault=value ...]
```

这有一个资料来参考可用的[设置](https://developer.apple.com/library/mac/#documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html#//apple_ref/doc/uid/TP40003931-CH3-DontLinkElementID_10)

```
CODE_SIGN_IDENTITY (Code Signing Identity)
    介绍: 标识符，指定一个签名.
    例如: iPhone Developer
```

PROVISIONING_PROFILE is missing from the index of available commands, but may be necessary.

Specify "CODE_SIGN_IDENTITY" & "PROVISIONING_PROFILE" settings in the xcodebuild command:

```
xcodebuild -sdk <iphoneos> -target <target_name> -configuration <Debug> CODE_SIGN_IDENTITY="iPhone Developer: Mister Smith" PROVISIONING_PROFILE="XXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX"
```
On success, the app will be built to your ```<app_dir>/build/<configuration>-iphoneos/<app_name>.app```

## Deploy using Fruitstrap
Go clone a forked version of fruitstrap as the [ghughes version](https://github.com/ghughes/fruitstrap) is no longer maintained. Success has been confirmed with the [unprompted fork](https://github.com/unprompted/fruitstrap), but others are reportedly functional.

Once cloned, run ``make fruitstrap``
Now, copy the resulting ``fruitstrap`` executable to your app's project or a parent directory.

Execute fruitstrap after a clean build by running (commands available depend on your fork of fruitstrap):
```
./fruitstrap -d -b <PATH_TO_APP> -i <Device_UDID>
```
If you are aiming to use continuous integration in this setup, you may find it useful to want to log the output of fruitstrap to both command line and log, like so:
```
./fruitstrap -d -b <PATH_TO_APP> -i <Device_UDID> 2>&1 | tee fruit.out
```
Since fruitstrap will need to be killed before the node server can be launched, an option is to scan the output of the fruitstrap launch for some telling sign that the app has completed launching. This may prove useful if you are doing this via a Rakefile and a ``go_device.sh`` script:
```
bundle exec rake ci:fruit_deploy_app | while read line ; do 
   echo "$line" | grep "text to identify successful launch" 
   if [ $? = 0 ] 
   then 
   # Actions 
       echo "App finished launching: $line" 
       sleep 5 
       kill -9 `ps -aef | grep fruitstrap | grep -v grep | awk '{print $2}'` 
   fi
 done
```
Once fruitstrap is killed, node server can be launched and Appium tests can run!

Next:
[Running Appium on Real Devices](https://github.com/appium/appium/wiki/Running-Appium-on-Real-Devices)
