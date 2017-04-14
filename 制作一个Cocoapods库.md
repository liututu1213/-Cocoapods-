## 一.注册Cocoapods账户信息
- pod trunk register 邮箱地址 '用户名' --verbose
- 注册成功之后执行 pod trunk me
## 二.创建共享库文件并上传到共有仓库
共享库三个必不可少的部分
A:共享文件
B:LICENSE文件(默认一般选择==MIT==);
C:库描述文件 .podspec

### 1.Spec Repo
     它是所有Pods的索一个索引 就是一个容器,所有公开的Pods都在里面,它实际上是一个Git仓库remote端GitHub上,
     当你使用了Cocoapods 后它会被clone到本地的 ~/.cocoapods/repos   
     创建私有库:
     a.执行pod repo add Cathyliu-repo git@git.coding.net:cathyliu/spec_repo_demo.git
       cd ~/.cocoapods/Cathyliu-repo 
       pod repo lint . //检查是否安装成功
     b.执行pod repo list 查看repo列表
     c.cd ~/.cocoapods/repos/  查看 repository目录下的文件内容
     d.想新创建spec repo 添加podspec 文件
     e.创建项目天剑podspec
     f.验证podspec
     g.pod repo push Cathyliu-repo podSepcTest.podspec //将podspec文件push新建的repo库里面
     
     
     
     
### 2.编辑.podspec文件
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
  pod lib lint podSepcTest.podspec --allow-warnings

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
    
    
### 二.使用fastlane自动打包
#### 1.创建fastlane 文件

      # Customise this file, documentation can be found here:
      # https://github.com/fastlane/fastlane/tree/master/fastlane/docs
      # All available actions: https://docs.fastlane.tools/actions
      # can also be listed using the `fastlane actions` command
      # Change the syntax highlighting to Ruby
      # All lines starting with a # are ignored when running `fastlane`
      # If you want to automatically update fastlane if a new version is available:
      # update_fastlane
      # This is the minimum version number required.
      # Update this, if you use features of a newer version
      
      fastlane_version "2.9.0"
      CURRENT_POD = "spec-repo-demo"

      default_platform :ios

      before_all do
      # ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
      end

      desc "Realse Pod"
      lane :release do |options|
      UI.message('begin release')
      perform_lane(:release, options)
      end

     lane :beta do |options|
     perform_lane(:beta, options)
     end

    # You can define as many lanes as you want

    after_all do |lane|
    # This block is called, only if the executed lane was successful

    # slack(
    #   message: "Successfully deployed new App Update."
    # )
    end

    error do |lane, exception|
    UI.message('end Lane release')

    # slack(
    #   message: exception.message,
    #   success: false
    # )
    end

    SOURCES = ['git@git.coding.net:cathyliu/spec_repo_demo.git']


    def perform_lane(env = :release, options)
    target_version = options[:version]
    
    UI.confirm("confirm push repo version is #{target_version}, pod is #{CURRENT_POD}")

    perform_scan = options[:perform_scan]

    check_current_work_status(env, options)
    perform_lint_lib
    if perform_scan
      perform_scan       
    end
    update_version(options)
    tag_git_and_repo_push(env, options)
    spec_push(env, options)
    end

    def check_current_work_status(env = :release, options)
    target_version = options[:version]
    if target_version.nil?
      UI.user_error!("target_version miss")
    end
    ensure_git_status_clean
    ensure_git_branch(branch: 'master') if env == :release
    end

    def perform_lint_lib
    pod_lib_lint(verbose: true, allow_warnings: true, sources: SOURCES, use_bundle_exec: true, fail_fast: false, use_libraries: true)
    end

    def perform_scan
      scan(
        workspace: "Example/#{CURRENT_POD}.xcworkspace",
        scheme: "#{CURRENT_POD}Demo",
        devices: ["iPhone 6s", "iPhone 7"],
        output_directory: "~/Downloads"
      )
    end

    def update_version(options)
      target_version = options[:version]
      #sync_build_number_to_git
      increment_version_number(version_number: target_version)

      version_bump_podspec(path: "#{CURRENT_POD}.podspec",
                version_number: target_version)
     end

     def tag_git_and_repo_push(env, options)
      target_version = options[:version]
      begin
        git_commit_all(message: "Bump version to #{target_version}")
      rescue # rescue, because this raises an exception if it can't be found at all
      end
      add_git_tag(tag: target_version)
      push_to_git_remote
     end

     def spec_push(env, options)
      repo = case env
          when :beta
           'iOS_SPEC_Beta'
          when :release
           'Cathyliu-repo'
          else
           return
         end
      pod_push(
        path: "#{CURRENT_POD}.podspec",
        repo: repo,
        sources: SOURCES,
        allow_warnings: true,
        use_libraries: true
      )
    end

    # More information about multiple platforms in fastlane: https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Platforms.md
    # All available actions: https://docs.fastlane.tools/actions

    # fastlane reports which actions are used
    # No personal data is sent or shared. Learn more at https://github.com/fastlane/enhancer
#### 2.执行fastlane命令
       fastlane release version:0.0.3
    
    


