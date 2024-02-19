+++
title = "在 LazyVim 中使用系统 clangd"
date = "2024-02-19"
[taxonomies]
categories = ["LazyVim"]
tags = ["clangd"]
+++

Mason LSP 使用系统已安装的 clangd。

<!-- more -->

新建 `~/.config/nvim/lua/plugins/mason.lua` 文件，写入如下内容：

```lua
return {
  "neovim/nvim-lspconfig",
  opts = {
    servers = {
      clangd = { mason = false },
    },
  },
}
```

## 参考资料

- [https://github.com/LazyVim/LazyVim/discussions/1351](https://github.com/LazyVim/LazyVim/discussions/1351)