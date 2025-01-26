# 工具安装

rust安装：

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装转wasm32：

```sh
rustup target add wasm32-unknown-unknown
```

dx工具安装：

```sh
cargo install dioxus-cli
```

验证工具：

```sh
dx --version
```

# 创建新项目

执行：`dx new` 项目名称，来创建项目。

首先会出现三个选项：

- Bare-bones：最小组成，只有 main.rs 和 assets 文件夹。
- Jumpstart：一个包含组件、视图等相对完整结构的应用。
- Workspace：一个包含各个平台的完整cargo workspace的项目集合。

其它参数可以按照需求选择。

当项目生成时，进入项目执行：`dx serve` ，启动项目。首次启动推荐设置代理。

