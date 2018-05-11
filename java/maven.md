## 作用
- 项目管理工具，几乎管理项目开发过程中所有的东西

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
- 特殊字符串: RELEASE LATEST SNAPSHOT(UTC时间）
- version组成 `<major version>.<minor version>.<incremental-version>-<qualifier>`

## 依赖
- groupdId artifactId version packaging type(一般用于pom) scope
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
- 父项目 聚合`<modules>/<module>`
- 子项目 继承`<parent>/<groupId><artifactId><version>`
- 引用 `<dependency>/...<type>pom</type>`

## profile 多个profile用于不同场景，类似于spring中的profile

## 属性
- env ${env.M2_HOME} ${java.home}
- setting.xml ${settings.localRepository}
- 内置 ${basedir} ${version}
- 自定义 `<project>/<properties>/<xxname>/<xxvalue>`

## 命令行
- `mvn eclipse:eclipse` | `mvn idea:idea`
- `mvn install` 将打包结果放入本地库中
- `mvn build` eclipse插件创造的概念，可自定义生命周期
- `mvn package` 打包，默认打jar包
