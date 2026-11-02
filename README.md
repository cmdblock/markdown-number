# markdown-number

一个纯 Lua 的 Neovim Markdown 标题编号插件。支持当前缓冲区和当前工作目录（`cwd`）下递归批量处理 `.md` 文件。

## 功能

- 为 ATX 标题（`#` 到 `######`）添加层级编号：`1`、`1.1`、`1.1.1`。
- 删除标题开头已有的数字编号。
- 批量递归处理当前工作目录中的 Markdown 文件。
- 跳过围栏代码块（` ``` ` 和 `~~~`）中的伪标题。
- 添加前先清理已有编号，因此重复执行不会产生 `1 1 标题`。
- 当前缓冲区操作可通过 Neovim 的撤销命令 `u` 撤销。

## 安装（lazy.nvim）

```lua
{
 "cmdblock/markdown-number",
  ft = "markdown",
  config = function()
    require("markdown-number").setup({
      exclude_dirs = {
        [".git"] = true,
        ["node_modules"] = true,
        -- ["assets"] = true,
      },
      extension = "md",
      notify = true,
    })

    vim.keymap.set("n", "<leader>ma", "<cmd>MarkdownNumberAdd<cr>",
      { desc = "Markdown: add heading numbers" })
    vim.keymap.set("n", "<leader>mr", "<cmd>MarkdownNumberRemove<cr>",
      { desc = "Markdown: remove heading numbers" })
    vim.keymap.set("n", "<leader>mA", "<cmd>MarkdownNumberAddAll<cr>",
      { desc = "Markdown: add numbers to all Markdown files in cwd" })
    vim.keymap.set("n", "<leader>mR", "<cmd>MarkdownNumberRemoveAll<cr>",
      { desc = "Markdown: remove numbers from all Markdown files in cwd" })
  end,
}
```

## 命令

| 命令                       | 作用                                       |
| -------------------------- | ------------------------------------------ |
| `:MarkdownNumberAdd`       | 当前缓冲区标题添加编号                     |
| `:MarkdownNumberRemove`    | 当前缓冲区标题删除编号                     |
| `:MarkdownNumberAddAll`    | 递归处理 `cwd` 下所有 `.md` 文件并添加编号 |
| `:MarkdownNumberRemoveAll` | 递归处理 `cwd` 下所有 `.md` 文件并删除编号 |

批量命令的根目录是执行命令时 `:pwd` 显示的 Neovim 当前工作目录，不是当前文件所在目录。可先用 `:cd` 切换目录。

## 编号规则

```markdown
# 第一章

## 第一节

### 第一小节

## 第二节

# 第二章
```

添加后：

```markdown
# 1 第一章

## 1.1 第一节

### 1.1.1 第一小节

## 1.2 第二节

# 2 第二章
```

如果跳过标题层级，例如 H1 后直接出现 H3，中间缺少的层级会显示为 `0`，例如 `1.0.1`。建议标题层级按顺序组织。

## 注意事项

1. 批量操作会直接写入磁盘文件，不会自动创建备份；建议先提交 Git 或备份目录。
2. 默认跳过 `.git` 和 `node_modules` 目录。可通过 `exclude_dirs` 修改。
3. 只处理标准 Markdown ATX 标题（行首 `#` 至 `######`，后面至少一个空格）；不处理 Setext 标题（下一行使用 `===` 或 `---`）。
4. 代码围栏中的内容不会被处理。
5. 删除编号会识别标题开头类似 `1 `、`1.2 `、` `1.2.3 `、`1. 标题`、`1) 标题`、`1、标题` 的编号。若标题本身以类似格式的数字开头，也可能被当作编号删除。
6. 批量命令可能处理当前已在其他窗口打开的文件；建议执行前保存相关缓冲区，避免磁盘内容与未保存缓冲区不一致。
