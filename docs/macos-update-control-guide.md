# macOS 系统更新失败问题总结与手动控制教程

## 1. 本次问题总结

本机 macOS 系统更新失败的根因是：系统更新源被手动改成了一个无效的自定义地址。

检查到的配置：

```bash
CatalogURL = "http://su.example.com:8088/index.sucatalog"
```

这个地址不是 Apple 官方更新服务器，且无法提供有效更新目录，所以 `softwareupdate --list` 会报错，系统设置里的软件更新也无法正常获取更新。

表现类似：

```text
不能从软件更新服务器（(null)）载入数据。
Software Update Tool
Finding available software
```

同时，系统里还保留过一次历史更新失败记录：

```text
MSU_ERR_STAGE_SPLAT_FAILED(56)
FullReplacementRequired
```

这表示某次增量补丁失败，需要完整替换更新。但本次真正阻止系统获取更新的主因是 `CatalogURL` 指向了假服务器。

## 2. 关键原理

macOS 的软件更新可以通过 `/Library/Preferences/com.apple.SoftwareUpdate` 里的 `CatalogURL` 指定更新目录地址。

- 不设置 `CatalogURL`：使用 Apple 官方默认更新源。
- 设置为有效内部更新源：从指定源获取更新。
- 设置为无效地址：系统无法获取更新，达到屏蔽更新效果。

所以，手动禁止更新的本质是：把 `CatalogURL` 指向一个无效地址。

手动恢复更新的本质是：删除 `CatalogURL`，让系统回到 Apple 默认更新源。

## 3. 手动禁止系统更新

执行：

```bash
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate CatalogURL "http://127.0.0.1:8088/index.sucatalog"
```

说明：

- `127.0.0.1` 是本机地址。
- 本机通常没有运行这个更新服务。
- macOS 会尝试从这个地址获取更新目录，但获取不到。
- 结果：系统无法发现、下载、安装更新。

执行后可验证：

```bash
sudo softwareupdate --list
```

如果看到类似无法载入更新服务器数据的提示，说明屏蔽生效。

## 4. 手动恢复系统更新

执行：

```bash
sudo defaults delete /Library/Preferences/com.apple.SoftwareUpdate CatalogURL
```

说明：

- 删除自定义更新源。
- macOS 自动恢复 Apple 官方默认更新源。
- 之后系统设置里的“软件更新”应可正常检查更新。

执行后验证：

```bash
sudo softwareupdate --list
```

或打开：

```text
系统设置 → 通用 → 软件更新
```

如果能正常显示可用更新，说明恢复成功。

## 5. 查看当前更新源配置

执行：

```bash
defaults read /Library/Preferences/com.apple.SoftwareUpdate CatalogURL
```

可能结果：

### 情况 A：显示一个 URL

例如：

```text
http://127.0.0.1:8088/index.sucatalog
```

说明当前使用了自定义更新源。

### 情况 B：显示不存在

例如：

```text
The domain/default pair of (/Library/Preferences/com.apple.SoftwareUpdate, CatalogURL) does not exist
```

说明没有自定义更新源，系统使用 Apple 默认更新源。

## 6. 推荐操作流程

### 需要禁止更新时

```bash
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate CatalogURL "http://127.0.0.1:8088/index.sucatalog"
sudo softwareupdate --list
```

### 需要恢复更新时

```bash
sudo defaults delete /Library/Preferences/com.apple.SoftwareUpdate CatalogURL
sudo softwareupdate --list
```

## 7. 注意事项

1. 禁止更新后，系统安全更新也会被屏蔽。
2. 长期禁用更新可能带来安全风险。
3. 建议只在确实需要避免自动升级时临时使用。
4. 需要更新系统前，先删除 `CatalogURL`。
5. 如果恢复后仍更新失败，可以重启 Mac 后再检查。

## 8. 本机本次修复结论

本机之前无法更新，是因为：

```bash
CatalogURL = "http://su.example.com:8088/index.sucatalog"
```

执行以下命令后恢复：

```bash
sudo defaults delete /Library/Preferences/com.apple.SoftwareUpdate CatalogURL
```

用户确认：系统已经可以正常更新。
