
## 参考文档

- [Jetbrains Idea插件开发文档](https://plugins.jetbrains.com/docs/intellij/welcome.html)
- 代码地址：[https://github.com/yangfeng20/selected-camel-words](https://github.com/yangfeng20/selected-camel-words)
- 整体流程文档：[2024idea插件开发指南](https://github.com/yangfeng20/idea-plugin-dev-guide/)

## 背景介绍

- 痛点：在idea开发过程中，希望按需驼峰选中文本。
- 现在默认是一整个单词选中，只有在设置-->智能按键 中开启了`使用"CamelHumps单词"`时能够驼峰选中。但是这种情况比较粗暴，直接全局开启了。但是在日常开发中，其实选中大部分是整个单词一起选中。只有少部分情况是按照驼峰选中。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718587278502-014951c0-37a6-45ca-880b-6f6ad4362d67.png#averageHue=%232d3035&clientId=u82f56fd2-2963-4&from=paste&height=489&id=ub5710b47&originHeight=734&originWidth=982&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=78949&status=done&style=none&taskId=u744323cf-7a02-426d-8c84-93174281e1f&title=&width=654.6666666666666)

- 所以就需要一块插件，能够快捷开关驼峰选中的能力，仅在需要的时候开启。

## 安装Plugin DevKit插件

- Idea 2023.3 以下版本可以不用安装当前插件。idea自带。2023.3版本及之后的版本没有当前查询，需要自己安装。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718547512708-62beeeb4-08f4-4d8e-a2b5-0ded6a1c194b.png#averageHue=%233f2a1f&clientId=u82f56fd2-2963-4&from=paste&height=351&id=u4c646a37&originHeight=527&originWidth=1188&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=99482&status=done&style=none&taskId=uc3badae3-5aaf-4249-8a41-b3cb80e4759&title=&width=792)

- 插件商店搜索`Plugin DevKit`用于安装插件，安装完成之后建议先重启一下，不然可能开发工具可能出不来。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718547670477-39627652-c472-48d1-9306-b3f6951ad1aa.png#averageHue=%232d3036&clientId=u82f56fd2-2963-4&from=paste&height=733&id=u72c3db4d&originHeight=1100&originWidth=1472&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=119533&status=done&style=none&taskId=uc3e8d4b6-2255-47c9-8d2c-1c944b86977&title=&width=981.3333333333334)


## 开启IDEA内部模式（使用UI检查器定位代码）

- 主菜单栏-->帮助(Help)-->编辑自定义属性

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718547904472-acb07240-5f27-4c6b-9ea9-8121371b4b6d.png#averageHue=%232c2f33&clientId=u82f56fd2-2963-4&from=paste&height=711&id=uab8a08aa&originHeight=1067&originWidth=1006&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=79772&status=done&style=none&taskId=u410fa5bb-e622-409f-8d7a-19f6bde2643&title=&width=670.6666666666666)

- 输入`idea.is.internal=true`开启内部模式，重启后生效。
- 参考链接 [https://plugins.jetbrains.com/docs/intellij/enabling-internal.html](https://plugins.jetbrains.com/docs/intellij/enabling-internal.html)
- 后面介绍如何使用UI检查器来定位代码位置

## 新建插件项目

- 新建项目选择Idea插件(如果没有`Plugin DevKit`插件，这里就没有当前选项)。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718548362008-214cfcf3-34f3-4b73-ae88-806c4397d79d.png#averageHue=%232c2f34&clientId=u82f56fd2-2963-4&from=paste&height=717&id=u8003bd34&originHeight=1075&originWidth=1178&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=84429&status=done&style=none&taskId=uc73c37e4-dd59-4d42-94ca-72baf73a8d1&title=&width=785.3333333333334)

- 同时注意jdk版本需要和idea对应。参考：[https://plugins.jetbrains.com/docs/intellij/creating-plugin-project.html#creating-a-plugin-with-new-project-wizard](https://plugins.jetbrains.com/docs/intellij/creating-plugin-project.html#creating-a-plugin-with-new-project-wizard)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718548749686-b3357c9f-bd8b-42e4-a516-2ecbc756e701.png#averageHue=%238e7d42&clientId=u82f56fd2-2963-4&from=paste&height=419&id=u5af8dfbd&originHeight=629&originWidth=1048&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=127510&status=done&style=none&taskId=u5b036d50-f2aa-4d94-a607-544dee07b3d&title=&width=698.6666666666666)
## 修改Gradle相关配置

### 取消gradle构建

- 先取消默认的gradle构建。先修改一些配置在重新构建。

### 修改依赖远程仓库地址

- 修改`build.gradle.kts`中的repositories，注释中央仓库（mavenCentral）。配置阿里云地址。
```kotlin
repositories {
    //    mavenCentral()
    maven {
        url = uri("https://maven.aliyun.com/repository/public")
    }
}
```

- 如果文件名不是kts结尾，而是gradle结尾，配置可能有些区别。

### 修改使用本地Idea调试

- 注释intellij中的version和type（如果指定，需要下载），使用本地安装的idea进行调试。
- 使用`localPath`设置本地idea的安装路径。参考：[https://plugins.jetbrains.com/docs/intellij/tools-gradle-intellij-plugin.html#configuration-intellij-extension](https://plugins.jetbrains.com/docs/intellij/tools-gradle-intellij-plugin.html#configuration-intellij-extension)
```kotlin
intellij {
    localPath.set("D:\\Program Files\\JetBrains\\IntelliJ IDEA 2024.1.2")
    //    version.set("2023.2.6")
    //    type.set("IC") // Target IDE Platform

    plugins.set(listOf(/* Plugin Dependencies */))
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718550133488-f7c8aab6-5427-4bd3-ba5e-1b86117bacc5.png#averageHue=%232c241d&clientId=u82f56fd2-2963-4&from=paste&height=776&id=u44ae2f78&originHeight=1164&originWidth=1298&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=185252&status=done&style=none&taskId=uc3d654d7-2dcf-46a5-ae27-a63bc190589&title=&width=865.3333333333334)
### 重新加载Gradle变更

- 点击Gradle图标重新加载变更。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718550376741-7a0881ef-e7bb-40f7-9b25-798cb91047d1.png#averageHue=%2325272b&clientId=u82f56fd2-2963-4&from=paste&height=169&id=uc0b3dda9&originHeight=253&originWidth=646&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=13856&status=done&style=none&taskId=ub06d2f6b-6c86-4873-acd1-30ad25ca8b9&title=&width=430.6666666666667)

- 等待构建完成，初次可能比较慢，需要下载依赖等。构建完成之后，文件也就有高亮显示了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718550509902-04294ef4-94d1-454b-81d8-7766e8fa231a.png#averageHue=%2324262a&clientId=u82f56fd2-2963-4&from=paste&height=1019&id=u97ebd297&originHeight=1528&originWidth=2560&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=284171&status=done&style=none&taskId=uc43557a0-663a-443d-a83b-1dd78d1d5d1&title=&width=1706.6666666666667)

### 其他build.gradle相关配置说明

- [https://plugins.jetbrains.com/docs/intellij/tools-gradle-intellij-plugin.html#ide-configuration](https://plugins.jetbrains.com/docs/intellij/tools-gradle-intellij-plugin.html#ide-configuration)

## plugin.xml解读

- 参考：[https://plugins.jetbrains.com/docs/intellij/plugin-configuration-file.html#configuration-structure-overview](https://plugins.jetbrains.com/docs/intellij/plugin-configuration-file.html#configuration-structure-overview)
- plugin.xml文件可以理解是当前插件的相关配置。是插件开发中非常重要的一个文件。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718550754318-92400b24-675a-4e22-a22d-2a2d7f18b135.png#averageHue=%23202225&clientId=u82f56fd2-2963-4&from=paste&height=712&id=uf6c5b27c&originHeight=1068&originWidth=1617&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=176952&status=done&style=none&taskId=uee806d90-9577-4c73-aac0-02c613ffc94&title=&width=1078)

- 构建完成之后，它会报错，是因为不能使用默认的模版字符。我们改掉就可以了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718552517018-138531ff-a610-43ef-9dac-dbae4e2f39ac.png#averageHue=%23202125&clientId=u82f56fd2-2963-4&from=paste&height=705&id=uc8755d1b&originHeight=1058&originWidth=1701&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=213092&status=done&style=none&taskId=uc0949c5c-b305-499d-adc0-961528be1be&title=&width=1134)
## 防止报错

- `kotlin`目录名修改为`java`。选中kotlin目录，按shift+f6修改目录名为java。不修改后面可能会报错。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718553531881-6e03337d-3f01-44fd-ab0b-ec7f5f79366e.png#averageHue=%23282b2f&clientId=u82f56fd2-2963-4&from=paste&height=607&id=u7ddb7c02&originHeight=911&originWidth=1491&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=163585&status=done&style=none&taskId=u8420db3a-295f-4076-b0fa-dae5cccf163&title=&width=994)


## 创建Action

- idea中的action概念可以理解为idea中的任意一个动作，点击了某个按钮，就会执行它绑定的Action类的代码。
- 使用Plugin DevKit 创建一个Action。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718553733269-446b5c17-537c-485a-b03e-43f7f1f09f41.png#averageHue=%23242629&clientId=u82f56fd2-2963-4&from=paste&height=1019&id=ua56391b3&originHeight=1528&originWidth=2560&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=373231&status=done&style=none&taskId=u93875180-5bc0-41e5-8121-65cf622b9f5&title=&width=1706.6666666666667)

- 填写actionId，类名，功能名称，功能描述，添加到目标位置。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718553891605-aedad20f-5839-47fc-9cdd-71ecd5523414.png#averageHue=%232d3035&clientId=u82f56fd2-2963-4&from=paste&height=701&id=u40c9bd57&originHeight=1052&originWidth=1442&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=115476&status=done&style=none&taskId=u7898169d-5fb4-44db-bc92-eca757c8f6f&title=&width=961.3333333333334)

- 创建成功之后就会生成一个Action类和在plugin中注册一个action，表说明他的位置。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718554136406-5a21f9f9-08bd-471e-89ea-938376a06300.png#averageHue=%23232529&clientId=u82f56fd2-2963-4&from=paste&height=1019&id=u8c3b70ff&originHeight=1528&originWidth=2560&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=382548&status=done&style=none&taskId=u0febc9cd-8abe-4ae4-af2e-7d952707677&title=&width=1706.6666666666667)

## 运行插件

- 点击gradle运行idea插件，它会重新启动一个idea环境，来运行idea插件。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718554273995-189bb165-3462-46fc-9d4d-3228a906eef4.png#averageHue=%232a2d32&clientId=u82f56fd2-2963-4&from=paste&height=347&id=ua427ffda&originHeight=520&originWidth=1191&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=46442&status=done&style=none&taskId=ue764d062-e5a8-4d51-a912-03e0bb027e6&title=&width=794)

- 第一次运行可能较慢，需要等待一会儿。运行成功之后，在工具栏中显示了我们的插件Action。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718554430625-a3b01d5c-73c9-4f8e-8233-dd38280e3720.png#averageHue=%235a8558&clientId=u82f56fd2-2963-4&from=paste&height=941&id=u62d3185f&originHeight=1412&originWidth=2078&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=292847&status=done&style=none&taskId=u0b9e3005-e554-4bc5-bb13-67c2c2ce66a&title=&width=1385.3333333333333)

- 点击插件Action，后台也打印了相应的结果。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718554512220-a67127f2-48b1-4f77-aa6e-c4de7eb904f9.png#averageHue=%2327282c&clientId=u82f56fd2-2963-4&from=paste&height=1019&id=u68e0512a&originHeight=1528&originWidth=2560&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=348452&status=done&style=none&taskId=u31b91a52-6c44-42f4-ac52-7f2893fdd95&title=&width=1706.6666666666667)
### 找不到你的插件注册的Action，并且控制台报错ClassNotFoundException？

- 需要将上面说的kotlin目录修改为java。

### 控制台中文乱码，报错？

- 修改`build.gradle.kts`，解决编译中文报错和控制台中文乱码问题。
```kotlin
tasks {
    // Set the JVM compatibility versions
    withType<JavaCompile> {
        sourceCompatibility = "17"
        targetCompatibility = "17"
        // 解决编译时中文报错
        options.encoding = "UTF-8"
    }

    // 添加以下内容，解决运行时控制台中文乱码
    withType<JavaExec> {
        jvmArgs = listOf("-Dfile.encoding=UTF-8", "-Dsun.stdout.encoding=UTF-8", "-Dsun.stderr.encoding=UTF-8")
    }
    ...
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718590182258-31e3f8a6-e765-462b-9f4e-f9697e1e519f.png#averageHue=%23202125&clientId=u82f56fd2-2963-4&from=paste&height=265&id=u513f96fd&originHeight=397&originWidth=1075&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=49835&status=done&style=none&taskId=u2223af60-a8e9-4bc0-9a68-4f8d6eaaf2d&title=&width=716.6666666666666)

## 定位Action插入位置

- 我是需要将Action插入到主菜单栏中的Tools菜单，要如何定位呢。这就只要使用我们之前说的内部工具，ui检查器了。
- 开启内部模式之后，按`ctrl+alt+鼠标左键`点击ui，就能显示当前ui的详细信息。找到他的`Group Id`然后创建action时搜索选择就好了。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718586983454-e462e2f0-bf09-4153-aeb0-6fcede3e0f1c.png#averageHue=%23303337&clientId=u82f56fd2-2963-4&from=paste&height=688&id=u86ee6a3e&originHeight=1032&originWidth=1920&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=172646&status=done&style=none&taskId=ufe59706f-f9ac-4957-a827-5e68c217a1e&title=&width=1280)

## 定位Idea相关代码

- 在前面提到，idea设置中的智能按键有提供全局的驼峰选中能力，开启之后的就能够实现驼峰选中了。
- 我们插件的逻辑也很简单，就是在编辑器中能够快捷开启这个开关。
- 那我们要如何实现快捷开关呢，其实也很简单，我们只需要去看idea是怎么实现的就好了。
### 使用ui检查器定位智能按键逻辑

- 在设置中使用ui检查器定位智能按键的配置类。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718587906529-7b04c456-c451-4e50-a26b-e7b293b646b0.png#averageHue=%23303338&clientId=u82f56fd2-2963-4&from=paste&height=688&id=u7888d3a7&originHeight=1032&originWidth=1920&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=336610&status=done&style=none&taskId=u957228ef-f4ba-468c-9f37-df4b670afed&title=&width=1280)

- 打开idea源代码，搜索类`EditorSmartKeysConfigurable`

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718587967799-3d921f7f-baa9-4fed-887e-e742f8c8d690.png#averageHue=%232d3034&clientId=u82f56fd2-2963-4&from=paste&height=688&id=ub74710ac&originHeight=1032&originWidth=1920&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=153201&status=done&style=none&taskId=ud3dc31d1-97ba-441c-90aa-f4c69aa3230&title=&width=1280)

- 然后搜索关键字camel，定位到配置，可以看到当前配置项有一个获取配置的方法，和更新配置的方法。然后查看这个`editorSettings`的来源。可以看到``editorSettings``的数据源是`EditorSettingsExternalizable.getInstance()`

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718588619291-22dba771-f11a-44b2-b5ef-72e8eb28654e.png#averageHue=%2327292e&clientId=u82f56fd2-2963-4&from=paste&height=688&id=uabf4537b&originHeight=1032&originWidth=1920&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=412026&status=done&style=none&taskId=ud9888140-93a9-4ca5-8e5b-3774ea14a88&title=&width=1280)

- 那这样就可以在我的项目中使用当前类的方法去设置快捷开关了。

## 编写插件逻辑（驼峰选中快捷开关）

- 在我们的Action中获取到刚刚的编辑器设置，然后对配置项`CamelWords`的状态取反就可以了。
```java
public class SelectCamelWordsAction extends AnAction {

    @Override
    public void actionPerformed(AnActionEvent e) {
        // 获取到编辑器设置
        EditorSettingsExternalizable editorSettingsExternalizable = EditorSettingsExternalizable.getInstance();
        // 对之前的状态取反
        editorSettingsExternalizable.setCamelWords(!editorSettingsExternalizable.isCamelWords());
    }
}
```

- 运行插件，通过ToolMenu中的`SelectCamelWords`就能够快捷开启关闭驼峰选中的功能。
- 但是这还是很麻烦啊，不能够快捷启动。接下来就要使用监听器来监听按键实现对应的快捷启动。

## 使用监听器监听键盘按键

- 参考文档：[https://plugins.jetbrains.com/docs/intellij/plugin-listeners.html#defining-application-level-listeners](https://plugins.jetbrains.com/docs/intellij/plugin-listeners.html#defining-application-level-listeners)
- 在plugin.xml中注册应用激活时监听，并绑定到我们的监听器中。
```xml
<applicationListeners>
  <listener class="com.maple.plugindemo.MyKeyListener" topic="com.intellij.openapi.application.ApplicationActivationListener"/>
</applicationListeners>
```

- 添加我们的自定义分发器
```java
public class MyKeyListener implements ApplicationActivationListener {
    private final KeyEventDispatcher dispatcher = new MyKeyEventDispatcher();

    @Override
    public void applicationActivated(@NotNull IdeFrame ideFrame) {
        KeyboardFocusManager.getCurrentKeyboardFocusManager().addKeyEventDispatcher(dispatcher);
    }

    @Override
    public void applicationDeactivated(@NotNull IdeFrame ideFrame) {
        KeyboardFocusManager.getCurrentKeyboardFocusManager().removeKeyEventDispatcher(dispatcher);
    }

    private static class MyKeyEventDispatcher implements KeyEventDispatcher {
        // 分发器逻辑
    }
}
```

- 在自定义的事件分发器中监听shift+ctrl+win按下，开启驼峰选中。win键抬起，关闭驼峰选中。
```java
private static class MyKeyEventDispatcher implements KeyEventDispatcher {
    private boolean shiftPressed = false;
    private boolean ctrlPressed = false;
    private boolean winPressed = false;

    private final EditorSettingsExternalizable editorSettingsExternalizable = EditorSettingsExternalizable.getInstance();

    @Override
    public boolean dispatchKeyEvent(KeyEvent e) {
        int keyCode = e.getKeyCode();
        int id = e.getID();

        if (id == KeyEvent.KEY_PRESSED) {
            switch (keyCode) {
                case KeyEvent.VK_SHIFT:
                    shiftPressed = true;
                    break;
                case KeyEvent.VK_CONTROL:
                    ctrlPressed = true;
                    break;
                case KeyEvent.VK_WINDOWS:
                    winPressed = true;
                    break;
            }

            if (shiftPressed && ctrlPressed && winPressed) {
                // 开启驼峰选中
                System.out.println("Shift + Ctrl + Win 按下!");
                editorSettingsExternalizable.setCamelWords(true);
            }
        } else if (id == KeyEvent.KEY_RELEASED) {
            switch (keyCode) {
                case KeyEvent.VK_WINDOWS:
                    if (shiftPressed && ctrlPressed) {
                        // 关闭驼峰选中
                        System.out.println("Win 键抬起!");
                        editorSettingsExternalizable.setCamelWords(false);
                    }
                    winPressed = false;
                    break;
                case KeyEvent.VK_SHIFT:
                    shiftPressed = false;
                    break;
                case KeyEvent.VK_CONTROL:
                    ctrlPressed = false;
                    break;
            }
        }
        return false; // 继续分发事件
    }
}
```

- 至此，插件逻辑就已经大功告成了。由于我没有mac，不清楚mac的快捷键，就没有适配mac快捷键。大家也可以去我的仓库提交pr。

## 打包插件

- 通过gradle的task打包插件。构建jar包，最终会输出到build-->libs下

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718605402182-6892efd6-d869-4e9e-bd5d-0df89b33203a.png#averageHue=%23272a2d&clientId=u82f56fd2-2963-4&from=paste&height=688&id=uc59316a2&originHeight=1032&originWidth=1920&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=241228&status=done&style=none&taskId=u78be52ff-dfed-4bf6-ba87-45f8b752e64&title=&width=1280)

## 本地安装验证

- 打开插件仓库，选择本地安装，找到打包之后的jar

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718605154760-763f0770-ff35-4c84-9648-75ecafd51dc3.png#averageHue=%232f3339&clientId=u82f56fd2-2963-4&from=paste&height=489&id=u35784b9e&originHeight=734&originWidth=982&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=121788&status=done&style=none&taskId=uc07d450f-1a03-4973-9261-2420ab0b2cd&title=&width=654.6666666666666)

- 安装成功，确认，然后测试插件

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718605193203-04254703-0174-439d-9e0d-f5a734068e16.png#averageHue=%232d3238&clientId=u82f56fd2-2963-4&from=paste&height=489&id=udf46665b&originHeight=734&originWidth=982&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=89994&status=done&style=none&taskId=u9197d542-1d33-4e9b-b07f-fbbfddf7971&title=&width=654.6666666666666)

## 发布插件

- 进入jetbrains插件开发平台：[https://plugins.jetbrains.com/developers/intellij-platform](https://plugins.jetbrains.com/developers/intellij-platform)
- 点击登录，或者注册账号

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718596132349-2ba08bb0-8719-4cf7-9c8d-249b1f418745.png#averageHue=%23d3d3d2&clientId=u82f56fd2-2963-4&from=paste&height=912&id=ufc118511&originHeight=1368&originWidth=2526&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=455672&status=done&style=none&taskId=u2ec7fb4a-be85-4ad0-9046-b5a2aee953d&title=&width=1684)

- 点击Upload Plugin上传插件，上传插件jar包，填写相关信息，点击upload Plugin.

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718596787371-f61a020b-3fcc-4110-8ecc-06b83c44a084.png#averageHue=%23fbede0&clientId=u82f56fd2-2963-4&from=paste&height=888&id=u2e0b6b2d&originHeight=1332&originWidth=1248&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=143534&status=done&style=none&taskId=u95ffb854-2642-405e-900d-c2c30bd19fb&title=&width=832)

- 上传成功，等待审核。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718603578791-d48bb06a-0cf6-4215-a172-20b41b970aa8.png#averageHue=%23faece0&clientId=u82f56fd2-2963-4&from=paste&height=843&id=ue76ce533&originHeight=1264&originWidth=2524&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=161156&status=done&style=none&taskId=u1dc23142-0a3c-41a2-a13e-5d435b71a7d&title=&width=1682.6666666666667)

## 兼容不同的idea版本

- idea-version参考: [https://plugins.jetbrains.com/docs/intellij/plugin-configuration-file.html#idea-plugin__idea-version](https://plugins.jetbrains.com/docs/intellij/plugin-configuration-file.html#idea-plugin__idea-version)
- 内部版本号映射参考：[https://plugins.jetbrains.com/docs/intellij/build-number-ranges.html#earlier-versions](https://plugins.jetbrains.com/docs/intellij/build-number-ranges.html#earlier-versions)
- 修改`build.gradle.kts`中的`patchPluginXml`，它用来不是兼容的idea版本。该值会覆盖`plugin.xml`中的`idea-versin`标签，所有可以直接把兼容版本写在这里。
```kotlin
patchPluginXml {
    // 最低版本
    sinceBuild.set("232")
    // 最高版本
    untilBuild.set("242.*")
}
```
```xml
<idea-version since-build="232" until-build="242.*"/>
```

- 要做到多版本兼容，就是要将最低版本设置的尽可能的小，但是同时需要你设置的版本有你插件当前使用的api。例如`applicationListeners`就在193.0之后发布，我的最低版本就只能设置193.0。最大版本不设置。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718604713486-ac067487-f505-435b-bdf0-410f6366e969.png#averageHue=%231f2024&clientId=u82f56fd2-2963-4&from=paste&height=626&id=u556c05cd&originHeight=939&originWidth=1033&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=120890&status=done&style=none&taskId=u3b3a9b4c-7041-468e-bb9b-bc50b17c8f0&title=&width=688.6666666666666)

- 发布前jetbrains会校验当前插件的兼容性，我这边就是校验通过，显示可以使用的版本。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/32453917/1718605579056-9618caf9-16ef-421d-8728-78b1e0136679.png#averageHue=%23fbeddf&clientId=u82f56fd2-2963-4&from=paste&height=850&id=ue34b9c73&originHeight=1275&originWidth=2527&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=276534&status=done&style=none&taskId=u6d856e51-7835-4fad-9516-3361aa2daaf&title=&width=1684.6666666666667)


## 后记

- 插件比较简单，主要是解决开发中的痛点，然后监听按键按下和释放的逻辑是直接让ai写的，可能有一些小bug。
- 然后mac端我不太清楚选中的快捷键是什么，也就没有做兼容。
- 感兴趣的话，可以点个在github 点个star。欢迎大家把项目拉下来，然后提交pr。



