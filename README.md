# iphone-mac-picture-backup

记录将 iPhone 照片备份到 macOS，并按月份归档到 `$HOME/archives/YYYY-MM` 的操作步骤。

## 前置条件

- Mac 已连接 iPhone，并在 iPhone 上选择“信任此电脑”。
- macOS 可以打开“图像捕捉”（Image Capture）。
- 已安装 `exiftool`。如果未安装，可使用 Homebrew 安装：

```sh
brew install exiftool
```

- 建议在开始前确认 Mac 磁盘空间充足，尤其是首次从 iPhone 导入大量照片和视频时。

## 1. 从 iPhone 导入照片到 Mac

1. 打开 macOS 自带的“图像捕捉”。
2. 在左侧设备列表中选择 iPhone。
3. 将导入位置设置为 `$HOME/Pictures`。
4. 将 iPhone 中的照片和视频导入到 `$HOME/Pictures`。
5. 如果希望达到“剪切”效果，请先确认文件已经完整出现在 Mac 上，再从 iPhone 中删除对应照片。

> 注意：不要在导入未完成或未检查前删除 iPhone 上的原文件。

## 2. 按月份归档照片和视频

归档目标目录为：

```sh
$HOME/archives/%Y-%m
```

例如，文件修改时间为 2026 年 5 月的文件会被放入：

```sh
$HOME/archives/2026-05
```

### 推荐：先复制归档

首次执行建议使用带 `-o .` 的命令。它会复制文件到归档目录，保留 `$HOME/Pictures` 中的原文件，适合先验证归档结果。

```sh
exiftool -m -r -P -o . \
-d "$HOME/archives/%Y-%m" \
'-Directory<FileModifyDate' \
-ext heic -ext heif -ext jpg -ext jpeg -ext png -ext mov -ext mp4 -ext aae \
-if '$FilePath !~ m#/archives/#' \
"$HOME/Pictures"
```

### 确认无误后：移动归档

确认复制归档结果正确后，可以改用不带 `-o .` 的命令。它会将匹配到的文件移动到归档目录，适合减少 `$HOME/Pictures` 中的重复占用。

```sh
exiftool -m -r -P \
-d "$HOME/archives/%Y-%m" \
'-Directory<FileModifyDate' \
-ext heic -ext heif -ext jpg -ext jpeg -ext png -ext mov -ext mp4 -ext aae \
-if '$FilePath !~ m#/archives/#' \
"$HOME/Pictures"
```

## 3. 命令参数说明

- `-m`：忽略较小警告，避免因个别文件元数据问题中断整体处理。
- `-r`：递归扫描 `$HOME/Pictures` 下的子目录。
- `-P`：尽量保留文件系统时间戳。
- `-o .`：复制文件到目标目录，同时保留源文件；不加该参数时会移动文件。
- `-d "$HOME/archives/%Y-%m"`：按文件时间格式化目标目录。
- `'-Directory<FileModifyDate'`：使用文件修改时间决定归档目录。
- `-ext ...`：只处理指定扩展名的文件，包括常见图片、视频和 iPhone `.aae` 辅助文件。
- `-if '$FilePath !~ m#/archives/#'`：跳过路径中已经包含 `/archives/` 的文件，避免重复处理归档目录中的内容。

## 4. 检查归档结果

统计 `$HOME/archives` 下所有文件数量：

```sh
find "$HOME/archives" -type f | wc -l
```

只统计常见照片和视频文件数量：

```sh
find "$HOME/archives" \( -iname "*.heic" -o -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.png" -o -iname "*.mov" -o -iname "*.mp4" \) -type f | wc -l
```

也可以查看各月份目录是否已经生成：

```sh
find "$HOME/archives" -maxdepth 1 -type d | sort
```

## 建议流程

1. 使用“图像捕捉”将 iPhone 文件导入 `$HOME/Pictures`。
2. 先执行“复制归档”命令。
3. 用检查命令确认 `$HOME/archives/YYYY-MM` 下的文件数量和目录结构。
4. 抽查几个归档目录中的照片和视频是否可以正常打开。
5. 确认无误后，再决定是否删除 `$HOME/Pictures` 中的临时导入文件，或改用“移动归档”命令。

## 注意事项

- 这些命令使用 `FileModifyDate` 归档，目录月份取决于文件修改时间，不一定等同于照片拍摄时间。
- `.aae` 文件是 iPhone 照片编辑信息文件，建议与照片一起保留。
- 如果导入后发现同名文件，`exiftool` 可能会自动处理或提示冲突；首次执行建议先使用复制归档方式观察结果。
- 删除 iPhone 或 Mac 上的原文件前，请确认归档目录中的文件数量、目录结构和抽样打开结果都正常。
