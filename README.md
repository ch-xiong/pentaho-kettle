# webSpoon

web Spoon是一个基于kettle的web图形设计器，用于Pentaho数据集成，外观和感觉与Kettle相同。

## 特性

- 数据安全
- 远程使用
- 易于管理
- 云

# 如何使用

请参考 [wiki](https://github.com/zhangrenhua/pentaho-kettle/wiki) and [issues](https://github.com/zhangrenhua/pentaho-kettle/issues).

# 如何配置（可选）

## Carte

如果 `$CATALINA_HOME/system/kettle/slave-server-config.xml` 存在，嵌入式CARTE servlet可以相应地配置。
参考 [here](https://wiki.pentaho.com/display/EAI/Carte+Configuration) 文件配置格式。
配置xml示例如下：

```xml
<slave_config>
  <max_log_lines>10000</max_log_lines>
  <max_log_timeout_minutes>2880</max_log_timeout_minutes>
  <object_timeout_minutes>240</object_timeout_minutes>
  <repository>
    <name>test</name>
    <username>username</username>
    <password>password</password>
  </repository>
</slave_config>
```

注意，只能使用`$HOME/.kettle/repositories.xml`中定义的存储库。

## 第三方插件和JDBC驱动程序

将第三方插件放入`$CATALINA_HOME/plugins`，将JDBC驱动程序放入`$CATALINA_HOME/lib`，如下所示：

```
$CATALINA_HOME
├── system
├── plugins
│   ├── YourPlugin
│   │   └── YourPlugin.jar
│   ├── ...
├── lib
│   ├── YourJDBC.jar
│   ├── ...
├── webapps
│   ├── spoon
│   ├── ...
```

## 禁用UI组件

Spoon使用XUL（XML用户界面语言）定义其用户界面的某些功能（请参见[此处](https://wiki.pentaho.com/display/ServerDoc2x/the+pentaho+XUL+Framework+Developer%27s+Guide)了解详细信息）。

```xml
<menu id="file" label="${Spoon.Menu.File}" accesskey="alt-f">
  <menuitem id="file-open" label="${Spoon.Menu.File.Open}" />
  <menuitem id="file-save-as" label="${Spoon.Menu.File.SaveAs}" />
</menu>
```

为了限制用户的能力，可能需要禁用一些UI组件。
为此，将`disable="true"`添加到要禁用的组件中，如下所示。

```xml
<menu id="file" label="${Spoon.Menu.File}" accesskey="alt-f">
  <menuitem id="file-open" label="${Spoon.Menu.File.Open}" />
  <menuitem id="file-save-as" label="${Spoon.Menu.File.SaveAs}" disabled="true" />
</menu>
```

没有`disabled`属性的效果与`disabled="false"`相同。

# 如何发展

Spoon依赖SWT来实现UI小部件，这对于操作系统不可知非常好，但它只作为桌面应用程序运行。
RAP/RWT为web用户界面提供了SWT API，因此用RAP/RWT替换SWT允许Spoon作为一个web应用程序运行，只需稍作代码更改。
尽管如此，一些api并没有实现；因此，需要比听起来更多的代码更改。

## 设计理念

1. 尽量减少与原来kettle spoon的差异。
2. 将webSpoon优化为web应用程序。

## 分支和版本控制

我从webspoon分支开始这个项目，从6.1.0.5-R到6.1.0.6-R之间的分支6.1分支。
很快我意识到我应该从一个发布的版本中分离出来。
所以我决定做两个分支：webspoon-6.1和webspoon-7.0，每一个分支分别被重新定位到6.1.0.1-R和7.0.0.0-R。

webSpoon使用4位数字版本控制，规则如下：

-第1位数字始终为0（绝不作为单独的软件发布）。
-第2位和第3位代表基本水壶版本，例如6.1、7.0。
-最后一个数字表示修补程序版本。

因此，下一个（预）版本将是0.6.1.4，这意味着它是基于Kettle版本6.1的第4个补丁。
可能有一个0.7.0.4版本，它基于Kettle 7.0版本，有（基本上）相同的补丁。

## 构建和本地发布依赖库

请生成并本地发布以下依赖库。

- pentaho-xul-swt
- org.eclipse.rap.rwt
- org.eclipse.rap.jface
- org.eclipse.rap.fileupload
- org.eclipse.rap.filedialog
- org.eclipse.rap.rwt.testfixture
- pentaho-vfs-browser

### pentaho-xul-swt

```bash
$ git clone -b webspoon-8.3 https://github.com/zhangrenhua/pentaho-commons-xul.git
$ cd pentaho-commons-xul
$ mvn clean install -pl swt
```

### rap

```bash
$ git clone -b webspoon-3.7.0 https://github.com/zhangrenhua/rap.git
$ cd rap
$ mvn clean install
```

### pentaho-vfs-browser

```bash
$ git clone -b webspoon-8.3 https://github.com/zhangrenhua/apache-vfs-browser.git
$ cd apache-vfs-browser
$ mvn clean install
```

## 编译 webSpoon

**确保上面修补的依赖库已在本地发布**

```bash
$ git clone -b webspoon-8.3 https://github.com/zhangrenhua/pentaho-kettle.git
$ cd pentaho-kettle
$ mvn clean install -Dmaven.test.skip=true
```

解压`assemblies/pdi-server/target/pdi-server-8.3.0.0-371.tar.gz`，启动命令：
linux/unix OS：`sh start-pentaho.sh`
Windows：`start-pentaho.bat`

启动之后浏览器访问：
`http://localhost:8080/spoon/spoon`

## 使用Selenium进行UI测试

目前，只有Google Chrome浏览器在运行UI测试用例时经过了测试。
除非通过参数`-Dheadless.unittest=false`，否则测试以无头模式运行。
要在non-headless模式下运行测试，Chrome的版本应该高于59。

默认访问地址 `http://localhost:8080/spoon`.
如果webSpoon部署到不同的url，则传递如下参数。

以下命令以non-headless模式运行所有单元测试用例，包括UI。

```bash
$ cd integration
$ mvn clean test -Dtest.baseurl=http://localhost:8080/spoon/spoon -Dheadless.unittest=false
```

# 通知

- 此版本是基于HiromuHot开源的增强版本，解决了很多RAP框架的重要bug。
