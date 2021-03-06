---
title: watch 和 watchOptions
sort: 14
contributors:
  - sokra
  - skipjack
  - SpaceK33z
  - EugeneHlushko
  - byzyk
  - spicalous
  - Neob91
---

webpack 可以监听文件变化，当它们修改后会重新编译。这个页面介绍了如何启用这个功能，以及当 watch 无法正常运行的时候你可以做的一些调整。


## `watch`

`boolean = false`

启用 Watch 模式。这意味着在初始构建之后，webpack 将继续监听任何已解析文件的更改。

__webpack.config.js__

```javascript
module.exports = {
  //...
  watch: true
};
```

T> [webpack-dev-server](https://github.com/webpack/webpack-dev-server) 和 [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) 里 Watch 模式默认开启。


## `watchOptions`

`object` `number = 200`

一组用来定制 watch 模式的选项：

__webpack.config.js__

```javascript
module.exports = {
  //...
  watchOptions: {
    aggregateTimeout: 200,
    poll: 1000
  }
};
```

直接向 `watchOptions` 设置为一个数字，等同于将 `watchOptions.aggregateTimeout` 设置为给定的数字。

__webpack.config.js__

```javascript
module.exports = {
  // 与 watchOptions.aggregateTimeout = 300 相同
  watchOptions:  300
};
```


## `watchOptions.aggregateTimeout`

`number = 200`

当第一个文件更改，会在重新构建前增加延迟。这个选项允许 webpack 将这段时间内进行的任何其他更改都聚合到一次重新构建里。以毫秒为单位：

```javascript
module.exports = {
  //...
  watchOptions: {
    aggregateTimeout: 600
  }
};
```


## `watchOptions.ignored`

`RegExp` [`anymatch`](https://github.com/micromatch/anymatch)

对于某些系统，监听大量文件系统会导致大量的 CPU 或内存占用。这个选项可以排除一些巨大的文件夹，例如 `node_modules`：

__webpack.config.js__

```javascript
module.exports = {
  //...
  watchOptions: {
    ignored: /node_modules/
  }
};
```

也可以使用多种 [anymatch](https://github.com/micromatch/anymatch) 模式：

__webpack.config.js__

```javascript
module.exports = {
  //...
  watchOptions: {
    ignored: ['files/**/*.js', 'node_modules/**']
  }
};
```

T> 如果你使用 `require.context`，webpack 会观察你的整个目录。你应该忽略一些文件和/或(and/or)目录，以便那些不需要监听的文件修改，不会触发重新构建。


## `watchOptions.poll`

`boolean = false` `number`

通过传递 `true` 开启 [polling](https://whatis.techtarget.com/definition/polling)，或者指定毫秒为单位进行轮询。

__webpack.config.js__

```javascript
module.exports = {
  //...
  watchOptions: {
    poll: 1000 // 每秒检查一次变动
  }
};
```

T> 如果监听没生效，试试这个选项吧。Watch 在 NFS 和 VirtualBox 机器上不适用。


## `info-verbosity`

`string = 'info': 'none' | 'info' | 'verbose'`

控制生命周期消息的详细程度，例如 `Started watching files(开始监听文件)...` 日志。将 `info-verbosity` 设置为 `verbose`，还会额外在增量构建的开始和结束时，向控制台发送消息。

```bash
webpack --watch --info-verbosity verbose
```


## 故障排除

如果您遇到任何问题，请查看以下注意事项。对于 webpack 为何会忽略文件修改，这里有多种原因。

### 发现修改，但并未做处理

在运行 webpack 时，通过使用 --progress 标志，来验证文件修改后，是否没有通知 webpack。如果进度显示保存，但没有输出文件，则可能是配置问题，而不是文件监视问题。

```bash
webpack --watch --progress
```

### 没有足够的文件观察者

确认系统中有足够多的文件观察者。如果这个值太低，webpack 中的文件观察者将无法识别修改：

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

Arch 用户，请将 `fs.inotify.max_user_watches=524288` 添加到 `/etc/sysctl.d/99-sysctl.conf` 中，然后执行 `sysctl --system`。 Ubuntu 用户（可能还有其他用户）请执行：`echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`。

### macOS fsevents Bug

在 macOS 中，某些情况下文件夹可能会损坏。请参阅[这篇文章](https://github.com/livereload/livereload-site/blob/master/livereload.com/_articles/troubleshooting/os-x-fsevents-bug-may-prevent-monitoring-of-certain-folders.md)。

### Windows Paths

因为 webpack 期望获得多个配置选项的绝对路径（如 `__dirname + '/app/folder'`），所以 Windows 的路径分隔符 `\` 可能会破坏某些功能。

使用正确的分隔符。即 `path.resolve(__dirname, 'app/folder')` 或 `path.join(__dirname, 'app', 'folder')`。

### Vim

在某些机器上，Vim 预先将 [backupcopy 选项](http://vimdoc.sourceforge.net/htmldoc/options.html#'backupcopy') 设置为 `auto`。这可能会导致系统的文件监视机制出现问题。将此选项设置为 `yes` 可以确保创建文件的副本，并在保存时覆盖原始文件。

`:set backupcopy=yes`

### 在 WebStorm 中保存

使用 JetBrains WebStorm IDE 时，你可能会发现保存修改过的文件，并不会按照预期触发观察者。尝试在设置中禁用`安全写入(safe write)`选项，该选项确定在原文件被覆盖之前，文件是否先保存到临时位置：取消选中 `File > {Settings|Preferences} > Appearance & Behavior > System Settings > Use "safe write" (save changes to a temporary file first)`。
