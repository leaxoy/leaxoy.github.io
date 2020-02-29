# Flutter Startup

&emsp;&emsp;之前一直想独立完成一个产品，奈何每次都死在了客户端上，iOS/Android两个主要的平台各自为战，学习两套对于后端来说几乎是不太可能的，虽然有React-Native等跨平台技术，但是性能上却广为人诟病。
直到[Flutter](https://flutter.dev/)的出现，才见的一缕曙光，媲美原生的性能，一致的开发体验，简单的入门成本，简直完美拯救了我们这些后端猿们。

## 环境配置
本文采用的环境：
- 系统：MacOS 10.15
- 版本：Flutter 1.12/master
- IDE：Vs Code

### Flutter
打开[官网](https://flutter.dev/), 点击右上角`Get Started`, 系统选择`Mac OS`, 按照指示的步骤下载并配置。

### Vs Code
- 去官网[Vs Code](https://code.visualstudio.com/)下载对应的安装包，我本地安装是1.41版本。
- 安装Flutter插件。

### XCode(只支持MacOS)
为了保证可以打包，测试iOS的应用，需要下载Xcode，点开App Store，搜索XCode，下载即可

### Android Studio(Intellij IDEA)
Android的开发环境，下载下来后需要安装Flutter插件。

### 升级
执行`flutter upgrade`，master分支的代码，几乎每天都会有更新，会修复一些bug，加入一些feature，当然也会引入一些新的bug，如果是想要稳定的话，还是使用stable版本比较合适。

> 最后执行`flutter doctor -v`，如果上面的步骤都完成了，应该是不会报错的。

## 创建项目
&emsp;&emsp;执行`flutter create --org com.example my_project`, 经过一小会儿等待后，便可以看到创建了一个新的项目，同时当前目录下出现了`my_project`的目录，这里就是我们的代码目录了。
### 目录结构

## Widget
万物皆组件，刚开始听到这句话的时候是懵逼的，虽然现在还是有点儿晕
Widget可以从一下几个角度进行划分。
- 有没有状态。StatefulWidget和StatelessWidget.
- 布局组件和展示组件。
- 一个子Widget和多个子Widget

