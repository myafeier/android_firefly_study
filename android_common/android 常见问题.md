- JNI Function 红色的解决办法（cannot resolve corresponding JNI function）
  - 修改配置文件，搜索"JNI"，并取消选中 "Missing JNI function"
    - Preferences > Editor > Inspections

- Run -> Run App 时加入签名的方法

  ###### 参考链接：<https://blog.csdn.net/u013654125/article/details/79755157>

  - 打开“Build——>Signing“  选项中配置自定义的签名文件
  - 打开“Build——>Edit Build Types”，默认Build Types提供两种构建模式：debug、release。在“Signing Config”选项中选中Signing配置的config名称（这一步很关键），即指定debug模式下使用的是自定义的签名文件。最后会发现在module的build.gradle的文件中添加如下内容：

