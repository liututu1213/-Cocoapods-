## 一.使用Cocoapods制作私有库步骤
    制作一个私有仓库主要分两个部分:
    1.创建一个私有的spec repo 
    2.创建pod所需要的工程文件
    
## 二.创建一个私有的spec repo
    一).什么是私有spec repo
        就是所有pods的一个索引,即一个容器,所有公开的Pods都在这个里面,
        可以使用 $pod repo 查看本地的spec repo
![image](https://raw.githubusercontent.com/CathyLy/imageForSource/master/CocoaPods/pod-repo.png)

    二).在github上创建一个git仓库,并clone到本地
        $pod repo add me_spec git@github.com:CathyLy/me_specs.git
        如果创建成功,可以进入到~/.cocoapods/repos目录下查看创建的私有spec repo

## 三.创建Pod项目工程文件
### 1.创建项目文件
    使用Cocoapods官方提供的模板进行创建
       $pod lib create CustomPod 
       (pod lib projectName --template-url=https://github.com/CocoaPods/pod-template.git)
       执行命令时，其实是下载了一个pod模板，然后通过在内部更改 .podspec文件的配置定制化自己的pod
![image](https://raw.githubusercontent.com/CathyLy/imageForSource/master/CocoaPods/customeExmaple.png)

### 2.打tag,发布一个release版本
##### A.提交需要共享文件
    创建一个github仓库存放pod项目文件,将pod地址clone到本地
    $git clone git@github.com:CathyLy/CustomPod-Example.git用模板创建的pod项目移动到这个git仓库里
     git add .
     git commit -m '提交信息'
     git push 
##### B.打tag
    git tag -m 'first release' '0.1.0'
    git push --tag #推送他会到origin
### 3.编辑.podspec文件
    Pod::Spec.new do |s|
    s.name             = 'CustomPod'
    s.version          = '0.1.0'
    s.summary          = 'A short description of CustomPod.'

    s.description      = <<-DESC
    TODO: Add long description of the pod here.
                       DESC

    s.homepage         = 'https://github.com/CathyLy/CustomPod-Example'
    # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
    s.license          = { :type => 'MIT', :file => 'LICENSE' }
    s.author           = { 'xxx' => 'xxx' }
    s.source           = { :git => 'https://github.com/CathyLy/CustomPod-Example.git', :tag => s.version.to_s }

    # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

    s.ios.deployment_target = '8.0'
    s.source_files = 'CustomPod/Classes/**/*'
  
    # s.resource_bundles = {
    #   'CustomPod' => ['CustomPod/Assets/*.png']
    # }

    # s.public_header_files = 'Pod/Classes/**/*.h'
    # s.frameworks = 'UIKit', 'MapKit'
    # s.dependency 'AFNetworking', '~> 2.3'
    end

### 4.验证podspec文件的合法性
     pod lib lint CustomPod.podspec


### 5.向自己的私有spec repo提交podspec文件
   pod repo me_spec CustomPod.podspec  #前面是本地repo的名字 后面是podspec的名字
    完成之后就可以在~/.cocoapods/repos/me_spec中查看
### 6.关于查找和使用新建的库
    a.在podspec发布之后,pod setup
    b.pod search CustomPod
    c.若不能找到自己的库文件,在pod setup成功之后,rm ~/Library/Caches/CocoaPods/search_index.json 删除这个文件
    这个是用来查找索引的文件
    d.在执行 rm 之后 pod search 会提示'Creating search index for spec repo 'master'.. Done!
    e.将自己的私有pod引用到其他项目时,记得在podfile文件中加入
      source 'https://github.com/CocoaPods/Specs.git'
      source 'git@github.com:CathyLy/me_specs.git'
    
    
