## 分类
- 标准参数`-`：所有的JVM实现都必须实现这些参数的功能，而且向后兼容
  - `-client` `-server`
  - `-classpath / -cp`多路径用分号';'隔开，使用后，JVM不再搜索环境变量中定义的CLASSPATH，JVM搜索路径的顺序为：
    - 先搜索JVM自带的jar或zip包（Bootstrat，搜索路径可以用System.getProperty("sun.boot.class.path")获得）
    - 搜索JRE_HOME/lib/ext下的jar包（Extension，搜索路径可以用System.getProperty("java.ext.dirs")获得）
    - 搜索用户自定义目录，顺序为：当前目录（.），CLASSPATH，-cp；（搜索路径用System.getProperty("java.class.path")获得）
- 非标准参数`-X`：默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容
- 非Stable参数`-XX`：此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用（但是，这些参数往往是非常有用的）
- `-D`设置环境变量，和JVM无关
