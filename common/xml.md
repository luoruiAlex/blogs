## xmlns
- XML Namespace的简写
- 用于区分同名的XML元素，比如`<context:component-scan>`中的component-scan来源于context的XML Namespace
- xml的根元素中需要声明即将使用的xmlns
  - xmlns=""表示默认Namespace，使用时不用加prefix
  - xmlns:xsi=""表示使用xsi作为前缀的Namespace，使用时需要加前缀xsi，xsi是w3规范的固定写法
  - xsi:schemaLocation，定义了XML Namespace和对应的XSD（Xml Schema Definition）文档的位置的关系。它的值由一个或多个URI引用对组成，两个URI之间以空白符分隔（空格和换行均可）。总是成对出现，主要用于无法联网时使用本地的xsd文档来进行校验。
  
## 文档限定
- 使用dtd和schema
- 描述xml文档结构,限定文档的数据类型
- schema比dtd功能更多
