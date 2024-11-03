# blog
多模块的个人博客

## JDK 1.8
## Spring boot mysqbits-plus spring security  CommandLineRunner spring task
### 工程创建
主要包括三个模块，一个前台模块，一个后台模块以及一个公用模块,然后就是maven父类的
基础
```
    <modules>
        <module>sangeng-framework</module>
        <module>sangeng-admin</module>
        <module>sangeng-blog</module>
    </modules>
```
启动模块在sangeng-blog里面，包括配置文件

domain，service，mapper，常量，统一响应格式放在sangeng-framework


采用easycode这个插件来，注意每次更新都要用maven的生命周期更新一下

这里有采用FastJson，这个是时间格式的转换，比默认的转换格式好，把@bean写在webconfig就行了

## 程序启动
对于java代码，直接打开，修改数据库信息即可，以及redis

对于前端工程
```
pnpm i 
当用 npm run dev的时候会显示有一些包没有下载
在这里使用npm下载慢会比较慢，因为npm存储包文件的服务器在国外，会被“墙”，建议切换成cnpm下载
npm install -g cnpm --registry=https://registry.npm.taobao.org

然后用cnpm根据它所包的信息来下载即可，

```


