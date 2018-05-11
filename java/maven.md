## 作用
- 项目管理工具，几乎管理项目开发过程中所有的东西

## 组成
- 1.源代码管理
- 2.单元测试用例
- 3.各种maven插件
- 可独立管理这几个部分，包括各种外部依赖关系

##　生命周期
- clean
  - pre-clean
  - clean
  - post-clean
- default
  - validate
  - initialize
  - generate-sources
  - process-resources
  - **compile**
  - process-classes
  - generate-test-sources
  - process-test-sources
  - test-compile
  - **test**
  - prepare-package
  - **package**
  - pre-integration-test
  - integration-test
  - post-integration
  - verify
  - **install**
  - **deploy**
- site
  - pre-site
  - site
  - post-site
  - site-deploy
- 生命周期由phase组成，phase可绑定多个goal

## 插件
- 执行真正的动作
- `<plugin>/<groupId><artifactId><version>`
- `<plugin>/<executions>/<execution><id><phase><goals><goal>`

## 版本规范
- groupId:artifactId:packaging:version
- version组成 `<major version>.<minor version>.<incremental version>-<qualifier>`
  - qualifier用字符串比较，注意用`alpha-01`代替`alpha-1`
  - SNAPSHOT 展开为一个日期和时间值并转为UTC时间，比如`1.0-SNAPSHOT`变为`1.0-20180511-143100-1`
  - LATEST 最新发布的发布版或者SNAPSHOT版
  - RELEASE 最后一个发布版

## 依赖管理
- groupdId artifactId version packaging type(一般用于pom) scope
- version 可用区间来表示，比如`(2.0,)` `[2.0, 3.0)`，多个区间可用逗号隔开
- scope
  - compile
  - test
  - provided
  - runtime
  - system
- 冲突的解决
  - 短路径优先
  - 声明优先
- 排除依赖 `<exclusions>/<execlution>/<groupId><artifactId>`
  
## 多项目
- 继承关系
  - 同一个工程所有子项目放到同一个目录下，即总项目文件夹下面pom.xml + 各子项目文件夹
  - 总项目 聚合`<modules>/<module>`，module是目录名，而不是子工程的artifactId
  - 子项目 继承`<parent>/<groupId><artifactId><version>`
  - 如果不用层次递进的目录结构来实现继承，可在parent中加入`<relativePath>`
- 引用关系 `<dependency>/...<type>pom</type>`

## profile 多个profile用于不同场景，类似于spring中的profile

## 属性
- env ${env.M2_HOME} ${java.home}
- setting.xml ${settings.localRepository}
- 内置 ${basedir} ${version}
- 自定义 `<project>/<properties>/<xxname>/<xxvalue>`

## profile
- 定义： `<profiles>/<profile>/<id><properties>/<profiles.activation>`
- 使用： `${profiles.activation}`
- 激活： `mvn install -P{profile}`

## 命令行
- `mvn eclipse:eclipse` | `mvn idea:idea`
- `mvn install` 将打包结果放入本地库中
- `mvn build` eclipse插件创造的概念，可自定义生命周期
- `mvn package` 打包，默认打jar包

##　搭建maven仓库
- nexus服务器
- 安装库文件到maven库中
  - 本地仓库 `mvn install:install-file -DgroupId= -DartifactId -Dversion= -Dpackaging=jar -Dfile=`
  - 公司的仓库 `mvn deploy:deploy-file -DgroupId= -DartifactId -Dversion= -Dpackaging=jar -Dfile= -Durl=http://xxx/sss.xxx -DrepositoryId=release-repo`
