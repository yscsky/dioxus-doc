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

# 组件

在Dioxus中，应用是用组件函数构成，函数通过`#[component]`宏声明。

组件属性的结构体通过`#[derive(Props, PartialEq, Clone)]`宏声明，属性是不可变。

组件可以包含组件。

```rust
#[derive(Props, PartialEq, Clone)]
struct DogAppProps {
    breed: String
}

#[component]
fn DogApp(props: DogAppProps) -> Element {
    // ...
}
```

Dioxus通过`rsx! {}`描述界面UI。rsx类似html：

```rust
rsx! {
    div { class: "bg-red-100",
        button {
            onclick: move |_| info!("Clicked"),
            "Click me!"
        }
    }
}
```

所有双引号中的字符串在RSX中会自动应用`format!()`，所以可以在外部定义变量，在表达式中直接使用不需要再调用format。

```rust
rsx! {
    div { "Breed: {breed}" }
}
```

任何表达式都渲染成字符串，同样支持`Option<Element>`和迭代元素。

```rust
rsx! {
    // Anything that's `Display`
    {"Something"}

    // Optionals
    {show_title.and_then(|| rsx! { "title!" } )}

    // And iterators
    ul {
        {(0..5).map(|i| rsx! { "{i}" })}
    }
}
```

Dioxus提供两种语法糖，if和for。

```rust
rsx! {
    if show_title {
        "title!"
    }

    ul {
        for item in 0..5 {
            "{item}"
        }
    }
}
```

对于list，可以使用key属性唯一化每一项，防止循环出错。

```rust
rsx! {
    for user in users {
        div {
            key: "{user.id}",
            "{user.name}"
        }
    }
}
```

# 样式

在项目的assets文件夹下，放置css和其它静态资源。在rust中通过`asset!()`引入。

```rust
static CSS: Asset = asset!("/assets/main.css");

fn App() -> Element {
    rsx! {
        document::Stylesheet { href: CSS }
        // rest of the app
    }
}
```

