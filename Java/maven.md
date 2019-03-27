参考 maven runoob.com

1. Maven 基础概述

基于项目对象模型（POM）概念，Maven 利用一个中央信息片段管理一个项目的构建、报告和文档等步骤。

Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

Maven 能够帮助开发者完成以下工作：

- 构建
- 文档生成
- 报告
- 依赖
- SCMs
- 发布
- 分发
- 邮件列表

Maven 的约定配置

| 目录                               | 目的                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| ${basedir}                         | 存放 pom.xml 和所有的子目录                                  |
| ${basedir}/src/main/java           | java source code                                             |
| ${basedir}/src/main/resources      | 项目的资源，比如说 property 文件                             |
| ${basedir}/src/test/java           | 项目的测试类，比如 Junit                                     |
| ${basedir}/src/test/resources      | 测试用的资源                                                 |
| ${basedir}/src/main/webapp/WEB-INF | web 应用文件目录，web 项目的信息，比如存放 web.xml、本地图片、jsp 视图页面 |
| ${basedir}/target                  | 打包输出目录                                                 |
| ${basedir}/target/classes          | 编译输出目录                                                 |
| ${basedir}/target/test-classes     | 测试编译输出目录                                             |
| Test.java                          | Maven 只会自动运行符合该命名规则的测试类                     |
| ~/.m2/repository                   | Maven 默认的本地仓库目录位置                                 |

2. Maven 环境配置
3. Maven POM