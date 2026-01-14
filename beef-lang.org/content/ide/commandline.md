+++
title = "命令行参数"
+++

## Beef IDE 命令行参数

Beef IDE 是一款 GUI 应用，本身不需要参数，但支持一些可选参数。

|参数    |说明      |
|----|------|
|-config=&lt;config>|设置配置|
|-launch|在调试器中编译并运行工作区启动项目。'--' 之后的内容会作为参数传入|
|-launch=&lt;path>|在调试器中编译并运行可执行文件 'path'。'--' 之后的内容会作为参数传入|
|-launchDir=&lt;path>|设置启动工作目录|
|-launchPaused|与 '-launch' 一起使用，启动调试器时暂停|
|-new|创建新的工作区和项目|
|-path&lt;Path>|设置目标文件（具体操作取决于文件类型）|
|-platform=&lt;platform>|设置平台|
|-test=&lt;path>|执行测试脚本|
|-verbosity=&lt;verbosity>|设置详细程度：quiet/minimal/normal/detailed/diagnostics|
|-workspace=&lt;path>|设置工作区路径|
