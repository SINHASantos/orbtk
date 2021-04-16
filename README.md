<img alt="OrbTk" width="380" src="https://gitlab.redox-os.org/redox-os/assets/raw/master/logos/orbtk/logo_dark.png">

[![Build and test](https://github.com/redox-os/orbtk/workflows/CI/badge.svg)](https://github.com/redox-os/orbtk/actions)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![crates.io](https://img.shields.io/badge/crates.io-0.3.1alpha3-orange.svg)](https://crates.io/crates/orbtk/0.3.1-alpha3)
[![docs.rs](https://img.shields.io/badge/docs-0.3.1alpha3-blue.svg)](https://docs.rs/crate/orbtk/0.3.1-alpha3)

The Orbital Widget Toolkit is a cross-platform (G)UI toolkit for building scalable user interfaces with the programming language Rust. It's based
on the [Entity Component System Pattern](https://en.wikipedia.org/wiki/Entity_component_system) and provides a [functional Reactive](https://en.wikipedia.org/wiki/Functional_reactive_programming)-like API.

The main goals of OrbTk are speed, ease of use, and cross-platform compatibility.

## Screenshots

<img alt="showcase" src="https://github.com/redox-os/orbtk-assets/blob/main/screenshots/orbtk-showcase-macos.png">

<p float="left">
<img alt="calculator-macos" height="300" src="https://github.com/redox-os/orbtk-assets/blob/main/screenshots/orbtk-calculator-macos.png">
<img alt="calculator-macos-light" height="300" src="https://github.com/redox-os/orbtk-assets/blob/main/screenshots/orbtk-calculator-light-macos.png">
<img alt="calculator-redox" height="300" src="https://github.com/redox-os/orbtk-assets/blob/main/screenshots/orbtk-calculator-redox.png">
</p>

## Features:

* Modern lightweight API
* Cross platform
* Modular crates
* Based on Entity Component System library [DCES](https://gitlab.redox-os.org/redox-os/dces-rust)
* Flexible event system
* Integrated widget library
* Custom widgets
* Custom theming engine
* Dynamic theme switching
* Integrated debugging tools
* Localization

## Platforms

* Redox OS (native)
* Linux (native | cargo-node)
* macOS (native | cargo-node)
* Windows (native | cargo-node)
* openBSD (not tested, but should work)
* Web (cargo-node)
* Android (native planned | cargo-node)
* iOS (native planned | cargo-node planned)
* Ubuntu Touch (native planned | cargo-node planned)

## Planned features

* Conformable use of async
* More default widgets
* Book
* Animations
* Split application in modules
* 3D context
* More integrated debugging tools

## Usage

To include OrbTk in your project, add this dependency
line to your `Cargo.toml` file:

```text
orbtk = "0.3.1-alpha3"
```

To use the latest development version of OrbTk, add this dependency
line to your `Cargo.toml` file:

```text
orbtk = { git = "https://github.com/redox-os/orbtk.git", branch = "develop" }
```
You can also check out the OrbTk template project to start a new project: https://github.com/redox-os/orbtk-template.

## Minimal Example

```rust
use orbtk::prelude::*;

fn main() {
      Application::new()
        .window(|ctx| {
            Window::new()
                .title("OrbTk - minimal example")
                .position((100.0, 100.0))
                .size(420.0, 730.0)
                .child(TextBlock::new().text("OrbTk").build(ctx))
                .build(ctx)
        })
        .run();
}
```

## Base concepts

### Widget

Widgets are the building blocks of user interfaces in OrbTk. They are things like Buttons, TextBoxes, ListViews, Views (Screens) and Grid(Layout)s. Each widget implements the [Widget trait](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/widget_base/mod.rs) and is generated by the [widget! macro](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/macros.rs). A widget consists of a name like `Button` and a list of its properties like `text: String`, `background: Brush` or `count: u32`. After the `build` method of a widget is called it's added to the Entity Component System where it exists as an `Entity` (index) with `Components`. The struct of a widget serves as a builder using the [builder pattern](https://doc.rust-lang.org/1.0.0/style/ownership/builders.html).

Basic usage of the widget! macro:

```rust
widget!(
    MyWidget {
      background: Brush,
      count: u32,
      text: String,
      ...
    }
);
```

### Widget Templates

Each widget has to implement the [Template trait](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/widget_base/template.rs). The template defines the default values of a widget's properties as well as its structure. A
Button e.g. consists of a `Container` widget, a `StackPanel` widget and a `TextBlock` widget.

Basic usage of the Template trait:

```rust
impl Template for MyWidget {
    fn template(self, id: Entity, ctx: &mut BuildContext) -> Self {
         self.name("MyWidget")
            .style("my_widget_style")
            .background("#000000")
            .count(0)
            .text("Initial text")
            .child(Container::new()
                    // Container references the same background as MyWidget
                    .background(id)
                    .child(TextBlock::new()
                            // TextBlock references the same text as MyWidget
                            .text(id)
                            .build(ctx)
                    )
                    .build(ctx)
            )
    }
}
```

### Widget State

The state of a widget is used to update its inner state. Each state has to implement the [State trait](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/widget_base/state.rs). The inner state of a widget is represented by the 
current values of its properties.

Basic usage of the state trait:

```rust
#[derive(Default, AsAny)]
struct MyState {
    ...
}

impl State for MyState {
    fn update(&mut self, _: &mut Registry, ctx: &mut Context) {
        // update the widget
        ...
    }
}

widget!(
    // Add MyState as state of MyWidget
    MyWidget<MyState> {
        ...
    }
);
```

The [Context parameter](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/widget_base/context.rs) of the update method provides access to the state's widget (entity) and its properties (components). It also provides functions to access the children of the widget, and to manipulate the widget tree.

### Styling widgets and define themes

OrbTk provides a theme engine base on [RON](https://github.com/ron-rs/ron). The engine provides the following features:

* Split theme in different files
* Outsource resources like colors and font stuff
* Derive styles
* Dynamic theme switch
* State styling (pressed | selected | focused | disabled)

Short example: 
```ron
Theme (
    styles: {
        "base": (
            properties: {
                "font_size": "$FONT_SIZE_12",
                "font_family": "$MEDIUM_FONT",
            }
        ),
        "button": (
            base: "base",
            properties: {
                "background": "$BLACK",
            },
            states: [
                (
                    key: "pressed",
                    properties: {
                        "background": "$WHITE",
                    }
                )
            ]
        )
    },
    resource: {
        "BLACK": "#000000",
        "WHITE": "#ffffff",
        "MEDIUM_FONT": "Roboto-Medium",
        "FONT_SIZE_12": 12,
        "FONT_SIZE_16": 16,
    }
)
```

OrbTk will also provide a plain mechanism to style and theme widgets and UIs. 

### Localization

OrbTk provides the possibility to register a application wide localization service. A localization service has to 
implement the [Localization](https://github.com/redox-os/orbtk/blob/develop/crates/localization/src/localization.rs) trait.

*Example*

```rust
pub struct MyLocalization {
    ...
}

impl Localization for MyLocalization {
    /// Gets the current language by language key e.g. `en_US` or `de_DE`.
    fn language(&self) -> &String {
        ...
    }

    /// Sets the current language by key e.g. `en_US` or `de_DE`.
    fn set_language(&mut self, key: &str) {
        ...
    }

    /// Gets the translated text for the given key. If there is no given translation the `key` will be returned as result.
    fn text(&self, key: String) -> String {
        ...
    }
}
```

It is possible to register a localization service on an application. There is also a ready to use [RonLocalization](https://github.com/redox-os/orbtk/blob/develop/crates/localization/src/ron_localization/mod.rs) service, that can read localization dictionaries in the [RON](https://github.com/ron-rs/ron) format.

*Example*

```rust
let de_de = r#"
    Dictionary( 
        words: {
            "hello": "Hallo",
            "world": "Welt",
        }
    )
    "#;

Application::new()
    .localization(
        RonLocalization::create()
            // sets the initial language
            .language("en_US")
            // adds an language dictionary to the localization service. 
            .dictionary("de_DE", de_de)
            .build()
    )
    .window(|ctx| {
        Window::new()
            .title("OrbTk - showcase example")
            .position((100, 100))
            .size(600, 730)
            .resizeable(true)
            .child(TextBlock::new().text("hello").build(ctx))
            .build(ctx)
    })
    .run();
```
In this example the text property with the value `hello` is the key of the localization service. If there is no localization service or no given dictionary for the current language the value of the property will drawn. It is possible to start the development of an complete without localization and add it later.

To switch the language on runtime the `set_language` method of the [Context](https://github.com/redox-os/orbtk/blob/develop/crates/api/src/widget_base/context) struct can be used.

## Run Examples

To build and run the examples you *will* need an C compiler (like `gcc`, `clang`, or MS's own compiler).

On Linux you also nee to install `cmake`. e.g.: 

```shell
sudo apt install cmake
```

If you have trouble to build and run the examples or you don't want to use a C compiler check the backend section please? It contains alternatives.

You can find examples in the `examples/` directory.

You can start the showcase example by executing the following command:

```text
cargo run --example showcase --release
```

OrbTk has integrated `debug` tools. If you want to show the bounds of all widgets (even invisible ones) and want to see a debug print of the whole widget tree, you can run the examples with `--features debug`, like this:

```text
cargo run --example showcase --release --features debug
```

## Run Examples with cargo-node

To run the examples as a browser, electron or cordova app you have to install cargo-node:

```text
cargo install -f cargo-node
```

Before you can use cargo-node you have to install `npm` version 6.9.0, which is included with `Node.js` version 10.16.3. You can download it from https://nodejs.org/dist/v10.16.3/. 

Rust's `cargo` is also required. The rest of cargo-node's dependencies are installed automatically.

### Start examples

You can run the "showcase" example by executing one of the following commands:

* Run as browser app:

```text
cargo node run --target browser --example showcase
```

* Run as electron app:

```text
cargo node run --target electron --example showcase
```

* Run as cordova app on android:

```text
cargo node run --target android --example showcase
```

## OrbTk backends

At the moment will evaluate different backends for OrbTk. A OrbTk backend consists of two different parts, window / events and rendering. OrbTk provides at the moment the following backends:

### orbraq (default)

* default backend for Redox, Linux, macOS and Windows (default feature)
* window and events based on [OrbClient](https://gitlab.redox-os.org/redox-os/orbclient)
* 2D rendering based on [raqote](https://github.com/jrmuizel/raqote)
* Known issues: window does not redraw while resizing

### stdweb

* default backend for web (default feature)
* window, events and 2D rendering based on [stdweb](https://github.com/koute/stdweb)
* Does not yet support all features of `orbraq` e.g. DropEvents, Clipboard access 

## Documentation

### Build and open documentation

You can build and view the latest documentation by executing the following command:

```text
cargo doc --no-deps --open
```

### OrbTk Manual

To build and run the latest version of the OrbTk manual check: [Manual](https://github.com/redox-os/orbtk/tree/develop/manual)

### OrbTk book

There is a (wip) OrbTk book check [OrbTk book](https://github.com/redox-os/orbtk-book)

## Sub Crates

* [api](https://github.com/redox-os/orbtk/tree/develop/crates/api): base api elements of OrbTk e.g. widget and application parts
* [proc_macros](https://github.com/redox-os/orbtk/tree/develop/crates/proc_macros): procedural helper macros
* [render](https://github.com/redox-os/orbtk/tree/develop/crates/render): cross platform 2D/3D render library
* [shell](https://github.com/redox-os/orbtk/tree/develop/crates/shell): cross platform window and event handling
* [theming](https://github.com/redox-os/orbtk/tree/develop/crates/theming): provide mechanism to style OrbTk UI's in rust and ron (replaces css-engine)
* [tree](https://github.com/redox-os/orbtk/tree/develop/crates/tree): tree structure based on DCES
* [utils](https://github.com/redox-os/orbtk/tree/develop/crates/utils): helper structs and traits
* [widgets](https://github.com/redox-os/orbtk/tree/develop/crates/widgets): base widget library

## Inspirations

* [Flutter](https://flutter.io/)
* [React](https://reactjs.org/)
* [Yew](https://github.com/DenisKolodin/yew)

## Showcases

* [Plural Planner](https://codeberg.org/PluralTools/Plural): Task app
* [twin-commander](https://github.com/kivimango/twin-commander): A twin-panel file manager specifically for the Redox OS 
* [Space Editor](https://codeberg.org/flovanco/space-editor): 2D Tile Map Editor compatible with OrbGame

## Contribution

If you want to help improve OrbTk you submit your feedback in the issue tracker, or make a pull request to fix an issue https://github.com/redox-os/orbtk/issues. You can also discuss OrbTk with us on the Redox chat https://redox-os.org/community/ (join the OrbTk channel).

#### Contribution check list

* Documentation for all `pub` structs, traits and funs
* Add tests if needed
* Use static &str for widget ids and new style definitions 
* For widget development check ProgressBar or Slider as example
* Add changes to changelog
* Expand examples or create a new one if needed
* `cargo fmt` at the end
* Create PR

## License

Licensed under MIT license ([LICENSE](LICENSE)).
