# Changing macOS Application Icons Programmatically

macOS 系统以美观而闻名。几近每条垂直线和水平线都是完美的......直到有一个应用程序的图标，它像洁白纸面上的一个黑点一样突出。显然，每个人都有不同的审美偏好。手动替换这些图标是可行的。[Apple官方](https://support.apple.com/zh-cn/guide/mac-help/mchlp2313/mac)给出的步骤如下：

1. 找到你想替换的图标
2. 下载它，但它可能是PNG
3. 将它转换为.icns格式
4. 在“访达”中找到要替换图标的应用程序
5. 右键，打开"显示简介"（⌘ + I）
6. 把.icns图标文件拖到应用程序图标上（左上角小图标）

这种方法的缺点（除了完全手动外）是应用程序更新经常覆盖美丽的自定义图标！且其本质会在APP包目录下新建一个`Icon？`文件。对于像我这样的人来说，这是不能接受的。

## 更换图标
事实上，macOS 中的“应用程序”实际上是文件夹。苹果称之为“软件包”。您可以使用`cd`命令进入其中：

```
$ ls /Applications/Atom.app
Contents

$ ls /Applications/Atom.app/Contents
Frameworks     Info.plist     MacOS          PkgInfo        Resources
```

如果我们深入该文件夹，Info.plist中将有一个`CFBundleIconFile`和`CFBundleTypeIconFile`

* `CFBundleIconFile` - 此应用程序显示在“访达”和“程序坞”中的图标
* `CFBundleTypeIconFile` - 此应用程序可以打开的其他类型文件的图标（并非所有应用程序都有此文件）

在 Atom 中，CbundleConfile 被指定为 atom.icns ，但它其实可以被指定为其他任何 `.icon` 文件。该名称所指示的文件存在于 `Contents/Resources` 文件夹下。果不其然，它就在那里：

```
$ ls /Applications/Atom.app/Contents/Resources | grep .icns
atom.icns
electron.icns
file.icns
```

现在，我们只需使用一些基本的unix命令即可替换图标。将你下载的图标文件放在一个安全的地方([macOS Icons](https://macosicons.com/#/)网站中中提供了许多美观的免费图标。)。如 `~/.custom-icons`，您也可以使用任何路径---只需更改下面脚本中的文件路径。

```
cp ~/.custom-icons/atom.icns /Applications/Atom.app/Contents/Resources/atom.icns
```

以上代码可以替换图标，但您必须重新启动计算机才能使更改生效......或者？

## 强制重新加载
默认情况下，应用程序图标会在启动时加载到缓存中。有您可以用以下方法重新加载图标缓存：

```
sudo rm -rfv /Library/Caches/com.apple.iconservices.store; sudo find /private/var/folders/ \( -name com.apple.dock.iconcache -or -name com.apple.iconservices \) -exec rm -rfv {} \; ; sleep 3; killall Dock; killall Finder
```

但实际上有个更容易的方法——只需`touch`该应用程序即可：

```
touch /Applications/Atom.app
```

事实上我也不知道这种方法为何有效，但直觉告诉我，更改文件的lstime会导致缓存失效。现在，您只需重新启动“访达”和“程序坞”即可完成更改：

```
sudo killall Finder
sudo killall Dock
```

现在我们已经清楚整个流程了！完整脚本如下：

```
function replace_icons() {
  cp ~/.custom-icons/atom.icns /Applications/Atom.app/Contents/Resources/atom.icns
  touch /Applications/Atom.app
  sudo killall Finder && sudo killall Finder
}
```

需要注意一点：先停止APP的运行，之后再运行该脚本。我曾用脚本强制退出APP，但事实证明这是个坏主意。

## 总结
希望此帖子能帮助您自动更换图标。同时，这绝不是为了让APP开发人员感到难过。当应用程序更新时，只需重新运行脚本，您的图标将再次完美！
