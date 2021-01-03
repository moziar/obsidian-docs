Obsidian 支持使用自定义网址协议 `obsidian://` 来触发软件里的各种不同的功能，这在 macOS 的跨软件工作流中十分常见。

## 安装 Obsidian URI

为了确保你的系统能够将 `obsidian://` 重定向到 Obsidian 软件里，你需要先进行以下的操作。

- 对于 Windows，第一次软件时，系统会自动将 `obsidian://` 注册到系统的注册表中。
- 对于 macOS，只需保证软件版本大于 0.8.12，并打开过一次软件即可。
- 对于 Linux，则需要分几步进行：
	- 首先，确保你创建了一个 `obsidian.desktop` 文件。[点击这里查看更多](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en)
	- 确保你的 desktop 文件指定了 `Exec` 为 `Exec=executable %u`。其中 `%u` 即为指向 `obsidian://` URI 到软件。
	- 如果你使用的是 AppImage 安装包，你或许需要使用 `Obsidian-x.y.z.AppImage --appimage-extract` 来解压。然后确保 `Exec` 指向的是解压后的可执行文件。

## 使用 Obsidian URIs

典型的 Obsidian URIs 格式如下：

```
obsidian://action?param1=value&param2=value
```

- `action` 字段通常是你希望软件进行的操作。

### 编码

==重要==

确保你的变量在采用的是 URI 的编码方式。举个例子，斜杆 `/` 必须替换为 `%2F`，而空格必须替换为 `%20`。这非常重要，因为不正确的编码会导致字符转义并进一步导致 URI 解析错误。[点击这里查看更多](https://en.wikipedia.org/wiki/Percent-encoding)。

### 支持的操作

####`open`

支持的参数：

- `vault` 字段可以是库名，也可以是库 ID。
	- 库名就是库文件夹的名字。
	- 库 ID 是一串由十六位字符组成的随机序列。这个 ID 对于你电脑上的每一个文件夹都是独有的。举个例子：`ef6ca3e3b524d22f`。目前而言，要找到这个 ID 并不容易，但我们会在晚些时候通过库切换器来提供这个功能。对于 Windows 而言，这个 ID 可以在 `%appdata%/obsidian/obsidian.json` 找到。对于 macOS 而言，将 `%appdata%` 替换为 `~/Library/Application Support/` 即可。对于 Linux，将 `%appdata%` 替换为 `~/.config/`。
	- `file` 字段既可以是文件名，也可以是一段从库的根目录开始的文件路径。
		- 在库内，Obsidian 使用统一的 `[[wikilink]]` 来解析到对应的目标文件。
		- 如果文件的扩展名是 `md`，那么可以不写扩展名。
	- `path` 字段为该文件在文件系统中的绝对路径。
		- 使用该字段将会自动覆盖 `vault` 字段跟 `file` 字段。
		- 软件会自动自动搜索该绝对路径对应的库。
		- 路径中除库路径，余下部分将会被填充为 `file` 字段参数。

举个例子：

- `obsidian://open?vault=my%20vault`
	这会打开 `my vault` 这个库。如果这个库已经打开了，软件会自动聚焦到对应窗口。

- `obsidian://open?vault=ef6ca3e3b524d22f`
	这会打开 ID 为 `ef6ca3e3b524d22f` 的库。

- `obsidian://open?vault=my%20vault&file=my%20note`
	这会打开 `my vault` 库中的 `my note` 笔记，假如 `my note` 存在并且文件名为 `my note.md`。

- `obsidian://open?vault=my%20vault&file=my%20note.md`
	这同样会打开 `my vault` 库中的  `my note` 笔记。

- `obsidian://open?vault=my%20vault&file=path%2Fto%2Fmy%20note`
	这将会打开 `my vault` 库中相对路径为 `path/to/my note` 的笔记。

- `obsidian://open?path=%2Fhome%2Fuser%2Fmy%20vault%2Fpath%2Fto%2Fmy%20note`
	这将会搜索所有包含 `/home/user/my vault/path/to/my note` 路径的库。之后剩下的路径即为 `file` 字段对应参数。假设现在这个库位于 `/home/user/my vault`，那么它的 `file` 参数将为 `path/to/my note`。

- `obsidian://open?path=D%3A%5CDocuments%5CMy%20vault%5CMy%20note`
	这将会搜索所有包含 `D:\Documents\My vault\My note`路径的库。T之后剩下的路径即为 `file` 字段对应参数。假设现在这个库位于 `D:\Documents\My vault`，那么它的 `file` 参数将为 `My note`.

## 速记格式

除了上面提到的格式以外，你还可以用下面这两种更为简便的方式来打开一个库或者一个文件：

- `obsidian://vault/my vault/my note` 等价于 `obsidian://open?vault=my%20vault&file=my%20note`
- `obsidian:///absolute/path/to/my note` 等价于 `obsidian://open?path=%2Fabsolute%2Fpath%2Fto%2Fmy%20note`
