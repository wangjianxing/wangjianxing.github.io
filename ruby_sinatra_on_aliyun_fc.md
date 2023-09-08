# 在阿里云函数计算上部署sinatra

## 阿里云的函数计算是什么
* 阿里云的函数计算类似于aws上的 lambda，按CPU、内存、调用次数等等的使用量付费
* 支持的部署方式：控制台、Serverless Devs、SDK
* ruby的话目前的话是通过Custom Runtime来支持，内置的ruby版本是2.7

### 部署步骤（控制台方式）
1. 阿里云函数计算FC控制台->服务及函数->创建服务
2. 创建成功点击进入服务，开始创建函数
  
    选择 *使用自定义运行时创建*，
  
    请求处理程序类型选择 *处理HTTP请求*，
  
    运行环境选择 *Ruby 自定义运行时 Debian 9*，
  
    代码上传方式选择 *使用示例代码*，
  
    启动命令留空，我们用bootstrap启动，端口写 *4567*，
  
    这样点击创建后，就跳转到函数的web IDE(像vscode)里了
3. Terminal菜单上new terminal

4. 代码部分

```bundle init``` 生成个Gemfile

  Gemfile加上sinatra，我试图用gem sources添加ruby-china的源，但bundle install没有成功，原因不明，所以直接把Gemfile里的source改了.
  ```ruby
  # Gemfile
  # frozen_string_literal: true

source "https://gems.ruby-china.com"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

ruby "2.7.0"

# gem "rails"
gem 'sinatra'
# gem 'puma'
  ```
接着开始bundle install，非常重要，这里必须将用到的依赖库gem也一并打包了(另一个方法就创建层，这样可以使用公用gem，我没试过，这里就不说了)
```ruby
bundle install --path vendor/bundle
```
安装完成后再建个bootstrap文件（注意：没有后缀名）用于启动，执行chmod 755 bootstrap、chmod 777 bootstrap或chmod +x bootstrap赋予文件的可执行权限。
```shell
# bootstrap
#!/bin/bash

bundle exec ruby server.rb -o 0.0.0.0

```

再把server.rb文件改改
```ruby
# server.rb
require 'sinatra'

get '/' do
  'Hello world!'
end

```
然后点击按钮 *部署代码* 就可以成功部署了

最后再检查下函数的 *触发器管理* 有没有创建了HTTP类型的触发器，看看设置是否没问题。
这样就可以 访问触发器的公网访问地址，看看是否返回 hello world

```ruby
curl https://sendxxx-service-fkhamynmsx.cn-guangzhou.fcapp.run
```
这样整个部署过程就完成了，一个 hello world 小服务就可以访问了。

## 注意要点

- HTTP触发器认证方式选择 *无需认证*
- 如果要使用HTTP触发器的其他认证方式中，签名验证没有官方sdk，使用JWT（gem 'jwt'）会比较简单
- 其他ruby版本目前我也不知道怎么弄
- gem sources设置无效，选择在Gemfile把源 直接 改成 gems.ruby-china.com
- bootstrap里记得加 -o 0.0.0.0 如果host不设置 0.0.0.0 的话，请求会超时，port设置里也有说明；并且记得执行 chmod 755 bootstrap赋予文件的可执行权限，否则会有权限问题

## 参考
- https://qiita.com/poruruba/items/004db74b8d9942a248df
- https://github.com/jwt/ruby-jwt
- https://help.aliyun.com/zh/fc/user-guide/configure-jwt-authentication-for-an-http-trigger
- https://stackoverflow.com/questions/67695411/storing-pem-and-key-files-as-strings-in-database
- https://help.aliyun.com/zh/fc/product-overview/?spm=a2c4g.11174283.0.0.11bc11528wt7qZ