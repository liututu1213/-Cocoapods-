## 一.注册Cocoapods账户信息
- pod trunk register 邮箱地址 '用户名' --verbose
- 注册成功之后执行 pod trunk me
## 二.创建共享库文件并上传到共有仓库
共享库三个必不可少的部分
A:共享文件
B:LICENSE文件(默认一般选择==MIT==);
C:库描述文件 .podspec
### 1.编辑.podspec文件
    Pod::Spec.new do |s|  
    s.name             = "podSepcTest"  
    s.version          = "0.1.0"  
    s.summary          = "podSpecTest is test firstly for me "  
    s.homepage         = "https://github.com/CathyLy/podSepcTest.git"   
    s.license          = 'MIT'  
    s.author           = { "CathyLy" => "137086637@qq.com" }  
    s.source           = { :git => "https://github.com/CathyLy/podSepcTest.git", :tag => s.version.to_s }    
    s.platform     = :ios, '8.0'  
    s.requires_arc = true  
    s.source_files = 'podSepcTestDemo/*'  
    s.frameworks = 'UIKit'  
    end  
- 验证podspec文件的合法性
  pod lib lint podSepcTest.podspec

### 3.打tag,发布一个release版本
##### A.提交需要共享文件
    git add .
    git commit -m '提交信息'
    git push 
##### B.打tag
    git tag -m 'first release' '0.1.0'
    git push --tag #推送他会到origin
### 4.发布自己的描述文件podspec到Cocoapods
    pod trunk push podSepcTest.podspec
### 5.关于查找和使用新建的库
    a.在podspec发布之后,pod setup
    b.pod search podSepcTest
    c.若不能找到自己的库文件,在pod setup成功之后,rm ~/Library/Caches/CocoaPods/search_index.json 删除这个文件
    这个是用来查找索引的文件
    d.在执行 rm 之后 pod search 会提示'Creating search index for spec repo 'master'.. Done!
    
    

