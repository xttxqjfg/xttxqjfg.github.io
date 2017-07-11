---
layout:     post
title:      "利用xcodeproj插件修改Xcode工程文件"
subtitle:   ""
date:       2017-07-11 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

实际开发中，我们都是用Xcode来修改工程文件，如果你这么觉得，那只是因为还没有遇到需要用到这个插件的地方。在利用jenkins构建自动化打包系统的时候，Xcode工程的部分配置，可能需要因不同的需要而实现自动修改。比如说修改Bundle ID以及描述文件等，这个时候xcodeproj插件就派上用场了。其实利用xcodeproj修改工程虽然大家没有直接操作过，但是肯定遇到过，我们常用的cocoapods就是用xcodeproj插件来修改xcode工程文件的

#### 安装

1、终端输入下面命令，目的是安装rvm，然后通过rvm来安装和管理ruby

```
curl -L get.rvm.io | bash -s stable
````

2、安装完成之后使用下面的命令载入rvm

```
source ~/.rvm/scripts/rvm
```

3、使用下面命令可以查看安装好的版本

```
rvm -v
```

4、Mac OS都自带ruby，但是版本基本在2.0，后面安装xcodeproj时需要更高版本，用下面命令再安一个高版本ruby，版本号可以自己选择

```
rvm install 2.1.4
```

接下来就是安装xcodeproj插件，输入下面的命令即可安装，之后可以在终端中输入xcodeproj来检查是否安装成功

```
[sudo] gem install xcodeproj
```

到这里，插件的安装基本已经结束了，接下来的就是介绍如何用代码来修改xcodeproj文件。下面给出的是添加文件引用、删除文件引用和修改证书和描述文件三个操作的ruby脚本代码

#### 示例脚本

```
require 'xcodeproj'  
project_path = ''    # 工程的全路径  
project = Xcodeproj::Project.open(project_path)  

# 1、显示所有的target  
project.targets.each do |target|  
puts target.name  
end  

# 增加新的文件到工程中  
target = project.targets.first  
group = project.main_group.find_subpath(File.join('testXcodeproj','newGroup'), true)  
group.set_source_tree('SOURCE_ROOT')  

# 获取全部的文件引用  
file_ref_list = target.source_build_phase.files_references  

# 设置文件引用是否存在标识  
file_ref_mark = false  

# 检测需要添加的文件是否存在  
for file_ref_temp in file_ref_list  
    puts file_ref_temp.path.to_s  
    if file_ref_temp.path.to_s.end_with?('ViewController1.m') then  
        file_ref_mark = true  
    end  
end  

if !file_ref_mark then  
    file_ref = group.new_reference('ViewController1.h文件路径')  
    target.add_file_references([file_ref])  
else  
    puts '文件引用已存在'  
end  

if !file_ref_mark then  
    file_ref = group.new_reference('ViewController1.m文件路径')  
    target.add_file_references([file_ref])  
else  
    puts '文件引用已存在'  
end 

project.save  
puts '文件添加完成'
```

```
require 'xcodeproj'  
project_path = ''    # 工程的全路径  
project = Xcodeproj::Project.open(project_path)  

# 1、显示所有的target  
project.targets.each do |target|  
puts target.name  
end  

# 从工程中删除文件  
target = project.targets.first  
group = project.main_group.find_subpath(File.join('testXcodeproj','newGroup'), true)  
group.set_source_tree('SOURCE_ROOT')  

def removeBuildPhaseFilesRecursively(aTarget, aGroup)  
aGroup.files.each do |file|  
    if file.real_path.to_s.end_with?(".m", ".mm", ".cpp") then  
        aTarget.source_build_phase.remove_file_reference(file)  
    elsif file.real_path.to_s.end_with?(".plist") then  
        aTarget.resources_build_phase.remove_file_reference(file)  
    end  
end  

aGroup.groups.each do |group|  
    removeBuildPhaseFilesRecursively(aTarget, group)  
    end  
end  

if !group.empty? then  
    removeBuildPhaseFilesRecursively(target, group)  
    group.clear()  
    group.remove_from_project  
end  

project.save
```

```
require 'xcodeproj'

project_path = ''    # 工程的全路径
project = Xcodeproj::Project.open(project_path)

puts 'ruby开始修改证书id和描述文件...'

project.targets.each do |target|
    target.build_configurations.each do |config|
        # 修改描述文件的id，值为id而不是名称
        config.build_settings['PROVISIONING_PROFILE'] = '5d6e268e-831b-4992-9b80-7595fe49380a'

        # 修改证书签名标识，一般无需修改
        # config.build_settings['CODE_SIGN_IDENTITY']

        # 修改工程的标识，即Bundle ID
        config.build_settings['PRODUCT_BUNDLE_IDENTIFIER'] = ''
    end
end

project.save
puts '修改完成...'

project.targets.each do |target|
    target.build_configurations.each do |config|
        puts '修改之后描述文件的id'
        puts config.build_settings['PROVISIONING_PROFILE']

        # 证书签名标识
        # puts '修改之后证书签名标识'
        # puts config.build_settings['CODE_SIGN_IDENTITY']

        # 工程的标识
        puts '修改之后工程的id'
        puts config.build_settings['PRODUCT_BUNDLE_IDENTIFIER']
    end
end
```
