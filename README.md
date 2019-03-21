# Bitnami-Redmine 安装与维护手册

>>Bitnami为众多开源软件——比如Redmine，WordPress等经过优化整理——提供Windows、OS X的一键式安装包
>>
>>，建议先安装[Bitnami-Redmine](https://bitnami.com/stack/redmine)，之后可以添加WordPress模块

**注意**: *安装时填写的是MySQL管理员密码，可用于维护Bitnami服务器；建议使用Redmine时，新注册一个高级用户帐户自己用；这样以后移交升级Redmine时也方便*

**注意：此手册是用于Bitnami_Redmine3.2.3-0——>3.3.0-1——>3.4.6-5升级**

## 一、安装后的常用配置

>安装时可以不配置SMTP邮箱发送

### 1.配置邮箱：异步模式configuration.yml

*redmine-4.0.x版本默认异步，不能更改，保持默认“smtp”配置*
```ruby
#位于C:\Bitnami\redmine-3.4.6-5\apps\redmine\htdocs\config
default:
  # Outgoing emails configuration
  # See the examples below and the Rails guide for more configuration options:
  # http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration
  email_delivery:
    delivery_method: :async_smtp
    async_smtp_settings:
      tls: false
      address: smtp.yeah.net
      port: 25
      domain: yeah.net
      authentication: :login
      enable_starttls_auto: true
      user_name: ******@yeah.net
      password: #不是用户的密码，是一个api_secret_token字符串
```

### 2.更改Redmine附件上传目录路径：configuration.yml

```ruby
  # Examples:
  # attachments_storage_path: /var/redmine/files
  # attachments_storage_path: D:/redmine/files
  attachments_storage_path: E:/BitNami/redmine/files
```

### 3.打开Apache 的 X-sendfile模块

>***在下载时支持大文件、续传、文件大小、高速的等特性***
>***Support large files, resume, file size, high speed and other features during download***

A.修改（增加）C:\Bitnami\redmine-3.4.6-5\apache2\conf\httpd.conf
```xml
LoadModule xsendfile_module modules/mod_xsendfile.so
RequestHeader Set X-Sendfile-Type X-Sendfile
<IfModule xsendfile_module>
XSendFile on
XSendFilePath E:/BitNami/redmine/files
XSendFilePath C:/Bitnami/redmine-3.4.6-5/apps/redmine/htdocs/public
XSendFilePath C:/Bitnami/redmine-3.4.6-5/apps/redmine/htdocs/tmp
</IfModule>
```

B.修改（增加）C:\Bitnami\redmine-3.4.6-5\apps\redmine\htdocs\config\additional_environment.rb
```ruby
# Specifies the header that your server uses for sending files
config.action_dispatch.x_sendfile_header = "X-Sendfile"
```

### 4.修改html主题CSS:application.css

```css
#位于C:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\public\stylesheets\application.css
```

```css
/***** Layout *****/
#top-menu {background: #3E5B76; color: #fff; height:1.8em; 
font-size: 1.2em; padding: 2px 2px 0px 6px;}
```

### 5.增加Html5对MP3、MP4的识别与播放：

>( *redmine-4.0.x版本默认支持* )

```
#位于
C:\Bitnami\redmine-4.0.1-1\apps\redmine\htdocs\public\javascripts\application.js
#在最后添加：
```

```javascript
$(document).ready(function(){
  $(".attachments p:has(a[href$='.mp3'])").after(function () {
    return "<audio controls><source src=\"" +
      $(this).find(" a[href$='.mp3'] ").attr("href")+"\" type=\"audio/mp3\"</audio>" 
  });
});

$(document).ready(function(){
  $(".attachments p:has(a[href$='.mp4'])").after(function () {
    return "<video width=\"640\" height=\"360\" controls><source src=\"" +
      $(this).find(" a[href$='.mp4'] ").attr("href")+"\" type=\"video/mp4\"</video>" 
  });
});
```

## 二、插件安装与刷新Redmine

### 1.简易安装


Put your Redmine plugins here:
redmine/apps/redmine/htdocs/plugins

#### 第一步：

将安装的插件copy到安装目录：D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins .

#### 第二步：

启动use_redmine.bat后，在CMD下将目录切换到D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins下，然后在执行D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins>
```ruby
bundle exec rake redmine:plugins:migrate NAME=redmine_materials RAILS_ENV=production
```
NAME=后接具体插件目录名


### 2.正规安装

**插件：Redmine_People**，Gemfile与主目录：

```ruby
#位于C:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\Gemfile
和
#位于C:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\Gemfile.lock
源地址冲突，所以：都改为http://,把插件中含的源地址删掉
```

    Download plugins and unpack them into redmine/apps/redmine/htdocs/plugins folder.
    Open an explorer and go to the folder where redmine is now installed. Find the script use_redmine.bat and run it with double click. New window with command line will be opened.
    It that new window enter the next commands

```ruby
    cd apps\redmine\htdocs
    gem source -a http:// # (把源由https换成http, 注意各个插件目录的Gemfile文件中，尽量删除重复的指定源链接语句)
    gem insatll bundler # (升级？)
    bundle install --without development test --no-deployment
    bundle exec rake redmine:plugins RAILS_ENV=production
```

Restart bitnami application.

**但是，正规安装后需刷新Redmine与bundle,我是在导入数据库（整个新安装升级Redmine）后，发现普通用户出错，然后百度到需执行：**

```ruby
bundle install --without development test
bundle exec rake generate_secret_token
bundle exec rake db:migrate RAILS_ENV=production
bundle exec rake tmp:cache:clear tmp:sessions:clear RAILS_ENV=production
```



### 3.卸载
第一步：启动use_redmine.bat后，在CMD下将目录切换到D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins下。

第二步：然后在执行D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins>bundle exec rake redmine:plugins:migrate NAME=redmine_contacts_invoices VERSION=0 RAILS_ENV=production。
          （注意migrate NAME后面的名字一定是插件的名字）

第三步：将D:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins下的对应插件文件移走。

第四步：重新启动Bitnami Redmine Stack服务即可。


### 4.插件配置

**redmine_latex_mathjax** 的“init.rb”

```ruby
#位于C:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\plugins\redmine_latex_mathjax
  settings :default => {
    'latex_mathjax_url' => '/mathjax/MathJax.js',
```
这样，需下载解压mathjax到目录 ： C:\Bitnami\redmine-3.3.0-1\apps\redmine\htdocs\public\mathjax

最好把字体在服务器安装一下 ： mathjax\fonts\HTML-CSS\TeX\otf


## 三、升级与更改服务器Bitnami，MySQL

**注意：此手册是用于Bitnami_Redmine3.2.3-0——>3.3.0-1——>3.4.6-5升级**

**再次注意：不要经常升级，不要安装最新（插件兼容差）**

1. 备份旧数据库，卸载
    1. 用Bitnami Redmine Stack关闭Redime服务进程：Thin_Redmine 和 Thin_Redmine2
    1. 打开phpAdmin，输入本地账户密码：root@...
    1. 快速导出库：Bitnami_Redime (所有表)
    1. 卸载Bitnami Redmine软件
    1. 重启电脑
1. 一键安装新的Bitnami Redmine软件，使用便于移交的root密码
	  1. phpAdmin删除库：bitnami_Redime
	  1. phpAdmin新建库：bitnami_Redime uft8_general_ci
    1. 导入备份数据库
    1. Config等设置
    1. 复制插件到/plugins
1. 用bundle升级数据库以兼容新Redmine
    1. ***刷新bundle***
        ```ruby
        cd apps\redmine\htdocs
        bundle install --without development test     --no-deployment
        bundle exec rake generate_secret_token
        bundle exec rake db:migrate RAILS_ENV=production
        bundle exec rake redmine:plugins RAILS_ENV=production
        bundle exec rake db:migrate RAILS_ENV=production
        bundle exec rake tmp:cache:clear tmp:sessions:clear   RAILS_ENV=production
        ```
    1. 重启所有服务进程
	  1. 分别用管理员账户和普通用户登录测试文件上传等

[![diamond2nv.png](https://avatars0.githubusercontent.com/u/20199788?v=3&s=460)](https://github.com/diamond2nv)
