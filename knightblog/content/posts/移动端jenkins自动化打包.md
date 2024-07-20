+++
title = 'Jenkins打包'
date = 2020-08-08
draft = false
+++

这是我的第一篇博客文章！欢迎阅读。

#移动端CICD自动化打包jenkins部署

android:
#!/bin/bash
export PATH="$PATH:/path/to/flutter/bin"
flutter clean
flutter pub get
flutter build apk --release
if [ ! -d "$WORKSPACE/apk" ]; then
  mkdir -p "$WORKSPACE/apk"
fi
timestamp=$(date +"%Y%m%d-%H:%M:%S")
mv build/app/outputs/flutter-apk/app-release.apk $WORKSPACE/apk/release-$timestamp.apk
echo "安卓包地址:~/apk/release-$timestamp.apk"

ios:
#!/bin/bash
export LANG=en_US.UTF-8
export PATH="$PATH:/Users/user/Documents/flutter/bin"
export PATH="$PATH:/usr/local/bin:/usr/local/sbin"
export PATH="$PATH:/opt/homebrew/bin"
flutter clean
flutter pub get
cd ios 
pod install
fastlane adhoc
echo "iOS包地址:~/ios/ipa/release-时间戳.ipa"

##fastlane部署

default_platform(:ios)

platform :ios do

  desc "Build an Ad-Hoc distribution"
  
  lane :adhoc do
    
    #清理 Fastlane 和 Xcode 缓存
    
    sh "rm -rf ~/Library/Developer/Xcode/DerivedData/*"
    
    sh "rm -rf ~/Library/Logs/gym/*"
    

    cert                       # 获取证书
    sigh(
      app_identifier: "com.xx.xx.xx",
      adhoc: true              # 明确指定下载 Ad-hoc 配置文件
    )

    increment_build_number(xcodeproj: "Runner.xcodeproj")

    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_options: {
        method: "ad-hoc",
        provisioningProfiles: {
          "com.xx.xx.xx" => "com.xx.xx.xx AdHoc"
        }
      },
      clean: true,  # 清理构建
      output_directory: ".",  # 输出目录
      output_name: "Runner",  # 输出名称
      buildlog_path: "~/Library/Logs/gym",  # 构建日志路径
      destination: "generic/platform=iOS",  # 构建目标
      xcargs: "OTHER_SWIFT_FLAGS='-D ADHOC'"  # 自定义 Xcode 参数
    )
  end
end



