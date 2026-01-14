+++
title = "BeefBuild"
weight = 60
+++

## BeefBuild 概览

BeefBuild 是 Beef IDE 的命令行对应工具。

```
BeefBuild -workspace=samples\HelloWorld -run -args Argument1 Argument2
```

|参数    |说明      |
|----|------|
|-args|在指定 '-run' 时，该参数后的内容会传给编译后的程序|
|-clean|构建时清理构建缓存|
|-config=&lt;config>|设置构建配置|
|-generate|为一个空项目生成启动代码|
|-new|创建新的工作区和项目|
|-platform=&lt;platform>|设置平台|
|-run=|编译并运行工作区中的启动项目|
|-test=&lt;path>|执行测试脚本|
|-verbosity=&lt;verbosity>|设置详细程度：quiet/minimal/normal/detailed/diagnostic|
|-workspace=&lt;path>|设置工作区路径|
