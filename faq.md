# 常见问题解答

### 问: 我安装了一个 Magisk 模块，现在我的设备无法正常开机。如何处理？

如果你在开发者模式中启用了 USB 调试，将你的手机连接上电脑。如果你的设备被检测到了（在终端里运行`adb devices` 来检查你的设备是否被检测），使用 `adb shell` 进入 shell ，然后运行 `magisk --remove-modules`。Magisk将移除所有的模块然后自动重启你的设备。

如果你没有启用 USB 调试，请重启到安全模式。大多数现代的 Android 都支持在启动时按下特殊的组合键来进入安全模式作为紧急选项。当 Magisk 在检测到安全模式被激活时会禁用所有模块。然后重启到正常模式（所有模块会保持禁用状态）。你可以在启动后使用 Magisk 应用管理模块。
### 问: 为什么某个应用检测到了 root？

Magisk 不再积极隐藏 root。但有很多 Magisk/Zygisk 模块提供了 root 隐藏的功能。请自行搜索。 😉

### 问：在我隐藏 Magisk 应用后，应用图标不正常。

在隐藏 Magisk 应用时。应用会创建一个空的代理应用。这个应用唯一的功能就是下载完整的 Magisk 应用到内部存储然后动态地加载它。因为这个应用是一个空应用，所以这个应用不包含 Magisk 的图标。

当你打开被隐藏的 Magisk 应用时，应用会提供创建主页快捷方式的选项（包含正确的应用名称和图标）以方便使用。当然，你也可以在应用设置里手动创建图标。
