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

# 事件

事件处理类似于常规属性，通过以`on`开头，接收一个闭包函数。

# 状态

## use_hook

使用纯Rust函数来存储和获取状态，无需其它结构。在组件调用时，会返回存储值`.clone()`。

```rust
fn DogView() -> Element {
    let img_src = use_hook(|| "https://images.dog.ceo/breeds/pitbull/dog-3981540_1280.jpg");

    // ..

    rsx! {
        div { id: "dogview",
            img { src: "{img_src}" }
        }
        // ..
    }
}
```

## use_signal

Signal是将原始的Rust值进行包装，会对其进行读写跟踪。

当Signal的值发生改变时，组件会进行重新渲染。

```rust
let mut img_src = use_signal(|| "https://images.dog.ceo/breeds/pitbull/dog-3981540_1280.jpg");
let save = move |evt| {
    info!("save click");
    img_src.set("https://images.dog.ceo/breeds/terrier-russell/IMG_7714.jpg");
};
```

## Context

当需要在整个App中管理状态，可以使用`Context`和`GlobalSignal`。

Context API可以在不同组件之间传递数据，而不通过额外的属性参数。

在创建数据的组件中，调用`use_context_provider()`将可以Clone的结构体作为数据提供者。在需要数据的组件中，调用`use_context()`获取数据。

```rust
// Create a new wrapper type
#[derive(Clone)]
struct TitleState(String);

fn App() -> Element {
    // Provide that type as a Context
    use_context_provider(|| TitleState("HotDog".to_string()));
    rsx! {
        Title {}
    }
}

fn Title() -> Element {
    // Consume that type as a Context
    let title = use_context::<TitleState>();
    rsx! {
        h1 { "{title.0}" }
    }
}
```

可以将Signal和Context结合，提供可以响应式的状态。`consume_context`可以修改状态。

```rust
#[derive(Clone, Copy)]
struct MusicPlayer {
    song: Signal<String>
}

fn use_music_player_provider() {
    let song = use_signal(|| "Drift Away".to_string());
    use_context_provider(|| MusicPlayer { song });
}

fn Player() -> Element {
    rsx! {
        button {
            onclick: move |_| consume_context::<MusicPlayer>().song.set("Vienna"),
            "Shuffle"
        }
    }
}
```

## GlobalSignal

GlobalSignal是一个简单的全局值，是Context和Signal的结合，无需额外的结构定义和配置。

```rust
// 定义在某个代码中
static SONG: GlobalSignal<String> = Signal::global(|| "Drift Away".to_string());
// 在组件中使用
fn Player() -> Element {
    rsx! {
        h3 { "Now playing {SONG}" }
        button {
            onclick: move |_| *SONG.write() = "Vienna".to_string(),
            "Shuffle"
        }
    }
}
```

# 接口调用

添加serde和reqwest依赖：

```sh
cargo add reqwest --features json
cargo add serde --features derive
```

使用async来直接调用reqwest获取数据。

```rust
let save = move |_| async move {
    let response: DogApi = reqwest::get("https://dog.ceo/api/breeds/image/random")
        .await
        .unwrap()
        .json()
        .await
        .unwrap();

    img_src.set(response.message);
};
```

Dioxus在异步闭包中自动调用`dioxus::spawn`。也可以在闭包中，直接使用`dioxus::spawn`，这样不用声明异步闭包。

```rust
rsx! {
    button {
        onclick: move |_| {
            dioxus::spawn(async move {
                // do some async work...
            });
        }
    }
}
```

直接使用async会在某些情况下出现竞态，更推荐使用`use_resource`来管理异步状态。

```rust
#[component]
fn DogView() -> Element {
    let mut img_src = use_resource(|| async move {
        reqwest::get("https://dog.ceo/api/breeds/image/random")
            .await
            .unwrap()
            .json::<DogApi>()
            .await
            .unwrap()
            .message
    });

    rsx! {
        div { id: "dogview",
            img { src: img_src.cloned().unwrap_or_default() }
        }
        div { id: "buttons",
            button { onclick: move |_| img_src.restart(), id: "skip", "skip" }
            button { onclick: move |_| img_src.restart(), id: "save", "save!" }
        }
    }
}
```

# 后端

Dioxus是一个全栈框架，可以在构建前端的同时，无缝构建后端，提供了一系列工具Server Function，Server Futures，Server State集成到应用中。

开启全栈：

```toml
[dependencies]
dioxus = { version = "0.6.0", features = ["fullstack"] }
```

添加server feature，移除默认的web target。

```toml
[features]
default = [] # <----- remove the default web target
web = ["dioxus/web"]
desktop = ["dioxus/desktop"]
mobile = ["dioxus/mobile"]
server = ["dioxus/server"] # <----- add this additional target
```

运行命令也修改为：`dx serve --platform web`。

server function是async的带`#[server]`的函数，返回`Result<(), ServerFnError>`。

