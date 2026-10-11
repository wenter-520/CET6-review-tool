# CET6-review-tool

一个浏览器端的 CET6 词汇复习工具。读取 CET6 目录下的 Excel 词库，按间隔重复曲线安排复习，记录每次反馈。

---

## 目录结构

```
CET6/
├── start.html              # 跳转页
├── 启动.bat                # 一键启动
├── .userdata/              # 学习记录（自动生成）
├── synonyms.json           # 近义词典（可选）
├── typo_hints.json         # 拼写提示（可选）
├── 第一天词汇学习.xlsx
├── 第二天词汇学习.xlsx
├── ...
├── 第N天词汇学习.xlsx
└── 词汇复习程序/
    └── index.html          # 主程序
```

**三条硬性约定：**

1. `index.html` 与 xlsx 词库之间隔一层 `词汇复习程序/` 目录。
2. CET6 根目录不能有 `index.html`。有的话 Python 会用它替换目录列表，程序就枚举不到 xlsx。跳转页因此叫 `start.html`。
3. `.bat` 里不能出现中文 URL 和 `%`，cmd 会把 `%XX` 当环境变量展开。

---

## 如何使用

### 1. 准备两个文件

在 CET6 根目录（与 xlsx 同级）创建 `start.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>正在进入…</title>
<meta http-equiv="refresh" content="0; url=词汇复习程序/index.html">
</head>
<body>
<p>正在进入 <a href="词汇复习程序/index.html">词汇复习程序</a>…</p>
</body>
</html>
```

在 CET6 根目录创建 `启动.bat`：

```bat
@echo off
chcp 65001 >nul
cd /d "%~dp0"
start "" http://localhost:8000/start.html
python -m http.server 8000
```

> `python` 找不到命令的话，把 `python` 换成 `py`。

### 2. 启动

双击 **`启动.bat`**。黑窗口弹出，浏览器自动进入复习程序。

**黑窗口不要关**，它就是服务器。用完按 `Ctrl+C` 停止，或直接关窗口。

### 3. 使用

- **开始本轮**：选好模式（复习优先 / 弱项强化 / 全量抽题）和分组，点「▶ 开始本轮」
- **翻面**：`空格`
- **评分**：`1` 熟悉 / `2` 模糊 / `3` 陌生 / `4` 混淆
- **上一张**：`←` 或 `Backspace`（会撤销上次评分）
- **跳过**：`S` 或 `→`

每次评分自动保存，关掉浏览器也不丢。

### 手动启动（不用 .bat）

```bash
cd path/to/CET6
python -m http.server 8000
```

浏览器访问 `http://localhost:8000/start.html`。

端口被占用就换 `8080`（`.bat` 和访问地址两处都要改）。

---

## 词库格式

每个 `第N天词汇学习.xlsx` 对应一天。文件名要能被识别出天号（`第3天` / `第三天` 都行）。

默认读前三列：

| 列 | 内容 |
|----|------|
| 1 | 单词 |
| 2 | 释义 |
| 3 | 分组标签 |

分组标签取值：`基础词汇` / `过考词汇` / `核心`。在表格里插入一行只写分组名，该行以下归入这一组，直到遇到下一个分组名。分组影响抽题优先级：核心 > 过考 > 基础。

列位置可在「数据」页调整。

---

## 数据

学习记录实时保存在浏览器本地（IndexedDB），换浏览器或清缓存会丢。

- **备份**：「数据」页 → 导出全部数据（JSON）
- **恢复**：「数据」页 → 从 JSON 恢复，上传备份文件
- **从 .userdata 导入**：如果 CET6 目录下已有 `.userdata/` 里的历史记录，点这个按钮覆盖导入

换电脑时，要么导出 JSON 再导入，要么整个 CET6 目录拷走。

---

## 常见问题

**双击 .bat 后浏览器 404**

- `.bat` 是不是放在 CET6 根目录
- `.bat` 里 URL 是不是 `http://localhost:8000/start.html`

**页面开了但词库为空**

- CET6 根目录有没有误放的 `index.html`，有就删掉或改名

**ERR_CONNECTION_REFUSED**

- 服务器窗口被关了。重新双击 `启动.bat`

**Address already in use**

- 端口被占。把 `8000` 改成 `8080`

**双击 index.html 打不开**

- 浏览器不允许 `file://` 读同级文件。用 `启动.bat`

**改了 synonyms.json 不生效**

- 「数据」页 → 重新加载辅助数据

---

## License

MIT
