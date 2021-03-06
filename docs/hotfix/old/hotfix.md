# Hotfix [项目地址](https://github.com/tyaqing/hotfix)

热更新能够不提交应用商店审核而修改 APP 内容,使得 app 的迭代变的非常灵活.

`MogoH5+`在`1.3.0`后加入了`热更新/安装更新`的集成.同时,我们的`热更新服务器`也在内测中,将在不久公测使用.您无需后端就可以完成整个软件的`热更新/安装更新`服务.

| iphone 热更新 效果如下                                           | 安卓热更新效果如下                                                    | 安卓安装更新如下                                                      |
| ---------------------------------------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------- |
| <img   width="400"  :src="$withBase('/IMG_0040.PNG')" alt="foo"> | <img   width="400"  :src="$withBase('/S80805-211149.jpg')" alt="foo"> | <img   width="400"  :src="$withBase('/S80806-112100.jpg')" alt="foo"> |

## 介绍

在使用热更新前,请看完这份介绍再使用.

### 热更新

对于安卓来说,热更新是没有任何问题的,但是苹果您可以看下面这段话.

#### App store 应用更新说明(引自 dcloud)

> 应用资源更新肯定是违反 apple 政策的，但目前看起来它也不管。你在官网案例那里下载 Appstore 版本的那些 app，大多启动后都会提示更新，反正也都没下架。如果你不是很大的公司，apple 不会理睬你。如果你是大公司，建议不要做整体更新，每次更新几个页面，也不要提示更新后需要重启，这样会安全点。

作者本人的实践中也经常使用热更新,名声不大,不频繁基本上可以自由使用.
同样,我们加入了`静默更新`这种比较无脑的更新.

### 安装更新

安装更新在这里指的就是更新大版本,比如添加了模块,必须重新安装.

#### 在安卓的安装更新

1.  可以让 APP 提示,然后用户选择自己下载安装
2.  应用商店更新

#### 在苹果的安装更新

1.  必须提交 AppStore 提交审核

在这里顺便提一下,苹果用户的习惯一般是自己主动更新.
在苹果应用商店的大厂 app (微信,淘宝,京东) 更新几乎不会受到 app 的更新推送,也没有所谓`检查更新`的功能,也就是说,更新是用户主动操作的.我们的`迭代`同样也遵循这条`行规`.

## 引入方式

可以在`main.js`中添加`checkUpdate(URL);`,打开 app 就会自动检测.还可以放在`检查更新`的按钮上触发.

### ES6 Module 引入

首先在`page.json`把用到`checkUpdate`的页面加上管道`|plusReady`.

然后加载使用.

```js
import { checkUpdate } from "./utils/hotfix";
checkUpdate(URL); // 填入您检查api的url地址
```

### `<script>`方式引入

这种用于没有使用脚手架的开发者

```html
<title>APP</title>
<script src="html5plus://ready"></script>  // 这段必须加载title底下
....
<script src="path/hotfix-bs.js"></script>
<script>
  checkUpdate('https://api.hotfix.femirror.com/public/app/checkUpdate?bundleId=你的appId'); // 填入您检查api的url地址
</script>
```

## 热更新配置

热更新分为两种,在 ios 和安卓都可以使用:

1.  静默更新 : 使用`checkUpdate`方法后检查更新,如果有更新会自动下载 app,整个过程没有任何提示,用户第二次打开 app 就会出现更新后的效果.
2.  提示更新 : 使用`checkUpdate`方法后检查更新,如果有则会弹出确认框,用户确认后会提示正在下载,正在安装,安装完成重启,更新完成等提示.

如果您更新后不想立即重启,可以注释掉`hotfix.js`中的函数`plus.runtime.restart();`,这样,app 会在用户第二打开的时候自动完成更新.

## 后端接收的数据格式

`checkUpdate(url)` 是以 `POST` 去访问 URL 的.数据类型为`application/json`

| 名称    | 描述                            | 类型   |
| ------- | ------------------------------- | ------ |
| version | app 当前的版本 ,如 2.1.0        | string |
| os      | app 设备的系统信息 比如系统版本 | json   |
| device  | app 设备的设备信息 比如 uuid    | json   |

## 后端返回的数据格式

下面是一个完整的数据返回 返回格式为`application/json` ,返回状态码为`200`.

> 注意 : 返回`200`以外的状态码将不被处理,如果想自行处理可以改写代码

| 名称           | 描述                                                            | 参数                     |
| -------------- | --------------------------------------------------------------- | ------------------------ |
| name           | 版本号 如果检测的版本大于本地版本就会更新                       | string                   |
| title          | 升级提示标题                                                    | string                   |
| description    | 升级提示内容                                                    | string                   |
| platform       | 符合升级的平台 ios:升级苹果 android:升级安卓 both:都更新        | `ios` ,`android`, `both` |
| type           | 安装类型 wgt:热更新 apk:安装更新                                | `wgt`,`apk`              |
| hotupdate_type | 更新提示 silence:静默更新 tip:提示更新 (apk 的安装是需要提示的) | `silence`,`tip`          |
| android_url    | 更新下载地址 以`.apk`,`.wgt`结尾的 http 地址                    | url                      |

下面这段`json`可以表达为:

1.  更新平台为 `全部平台`
2.  `热更新`
3.  `提示更新`

```json
{
  "name": "2.2.1",
  "title": "发现新版本,快来使用吧!",
  "description": "1.增加了....\n 2.修复了... 3.杀了一个程序员祭天",
  "platform": "both",
  "type": "wgt",
  "hotupdate_type": "tip",
  "android_url": "http://cdn.femirror.com/H5FD926B1.wgt"
}
```
