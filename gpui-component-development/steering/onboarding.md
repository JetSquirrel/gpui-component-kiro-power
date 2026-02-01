# GPUI Component Onboarding Guide

Complete setup guide for getting started with GPUI Component development.

## Prerequisites

### System Requirements

**Rust Installation:**
```bash
# Install Rust via rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Verify installation
rustc --version
cargo --version
```

**Platform-Specific Requirements:**

**macOS:**

macOS 15 or later
Xcode command line tools

```bash
# Install Xcode command line tools
xcode-select --install

# Optional: Install Homebrew for additional tools
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Linux (Ubuntu/Debian):**
```bash
#!/usr/bin/env bash

sudo apt update
# Test on Ubuntu 24.04
sudo apt install -y \
  gcc g++ clang libfontconfig-dev libwayland-dev \
  libwebkit2gtk-4.1-dev libxkbcommon-x11-dev libx11-xcb-dev \
  libssl-dev libzstd-dev \
  vulkan-validationlayers libvulkan1
```

**Windows:**
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

winget install Microsoft.VisualStudio.2022.Community --silent --override "--wait --quiet --add ProductLang En-us --add Microsoft.VisualStudio.Workload.NativeDesktop --includeRecommended"
scoop bucket add extras
scoop install cmake
```


### Development Tools (Recommended)

```bash
# Install useful cargo extensions
cargo install cargo-watch    # Auto-rebuild on file changes
cargo install cargo-machete  # Find unused dependencies
cargo install typos          # Spell checker
cargo install samply         # Performance profiler
```

## Project Setup

### Option 1: Start with GPUI Component Template

```bash
# Clone the GPUI Component repository
git clone https://github.com/your-org/gpui-component.git
cd gpui-component

# Run the Story Gallery to verify setup
cargo run

# Run an example
cargo run --example hello_world
```

### Option 2: Create New Project from Scratch

**1. Initialize Cargo Project:**
```bash
cargo new my-gpui-app
cd my-gpui-app
```

**2. Add GPUI Component Dependency:**

Edit `Cargo.toml`:
```toml
[package]
name = "my-gpui-app"
version = "0.1.0"
edition = "2021"

[dependencies]
gpui = "0.2.2"
gpui-component = "0.5.0"
# Optional: for default bundled icons
gpui-component-assets = "0.5.0"
anyhow = "1.0"
```

> **Note**: Check [crates.io](https://crates.io/crates/gpui-component) for the latest versions.

**3. Create Basic Application Structure:**

Create `src/main.rs`:
```rust
use gpui::*;
use gpui_component::{button::*, *};

struct MyApp {
    message: SharedString,
}

impl MyApp {
    fn new() -> Self {
        Self {
            message: "Hello, GPUI Component!".into(),
        }
    }
}

impl Render for MyApp {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .items_center()
            .justify_center()
            .size_full()
            .gap_4()
            .child(
                div()
                    .text_xl()
                    .child(self.message.clone())
            )
            .child(
                Button::new("hello-button")
                    .label("Click Me!")
                    .primary()
                    .on_click(cx.listener(|this, _event, _window, _cx| {
                        this.message = "Button clicked!".into();
                    }))
            )
    }
}

fn main() {
    let app = Application::new().with_assets(gpui_component_assets::Assets);

    app.run(move |cx| {
        // CRITICAL: Initialize GPUI Component first
        gpui_component::init(cx);

        cx.spawn(async move |cx| {
            cx.open_window(WindowOptions::default(), |window, cx| {
                let app_view = cx.new(|_| MyApp::new());
                // Root view is required for all windows
                cx.new(|cx| Root::new(app_view, window, cx))
            })?;
            Ok::<_, anyhow::Error>(())
        })
        .detach();
    });
}
```

**4. Run Your Application:**
```bash
cargo run
```

> **✅ Checkpoint:** If `cargo run` fails, check:
> 1. All dependencies are correctly specified in Cargo.toml
> 2. `gpui_component::init(cx)` is called first in app.run()
> 3. Root view wraps your main view
> 4. The window callback has both `window` and `cx` parameters

## Project Structure Best Practices

### Recommended Directory Layout

```
my-gpui-app/
├── Cargo.toml
├── src/
│   ├── main.rs              # Application entry point
│   ├── app.rs               # Main application logic
│   ├── views/               # Application views
│   │   ├── mod.rs
│   │   ├── main_view.rs
│   │   └── settings_view.rs
│   ├── components/          # Custom components
│   │   ├── mod.rs
│   │   └── custom_button.rs
│   ├── models/              # Data models
│   │   ├── mod.rs
│   │   └── app_state.rs
│   └── utils/               # Utility functions
│       ├── mod.rs
│       └── helpers.rs
├── assets/                  # Static assets
│   ├── icons/
│   └── images/
├── themes/                  # Custom themes (optional)
│   └── custom_theme.json
├── examples/                # Usage examples
│   └── basic_usage.rs
└── tests/                   # Integration tests
    └── app_tests.rs
```

### Module Organization

**src/main.rs:**
```rust
mod app;
mod views;
mod components;
mod models;
mod utils;

use app::App;

fn main() {
    App::run();
}
```

**src/app.rs:**
```rust
use gpui::*;
use gpui_component::{button::*, *};
use crate::views::MainView;

pub struct App;

impl App {
    pub fn run() {
        let app = Application::new().with_assets(gpui_component_assets::Assets);

        app.run(move |cx| {
            // Initialize GPUI Component first
            gpui_component::init(cx);

            cx.spawn(async move |cx| {
                cx.open_window(WindowOptions::default(), |window, cx| {
                    let main_view = cx.new(|cx| MainView::new(window, cx));
                    cx.new(|cx| Root::new(main_view, window, cx))
                })?;
                Ok::<_, anyhow::Error>(())
            })
            .detach();
        });
    }
}
```

## Essential Concepts

### 1. Root View Requirement

Every window MUST start with a `Root` view:

```rust
// ✅ Correct - Root as top-level view
cx.new(|cx| Root::new(my_view, window, cx))

// ❌ Incorrect - Direct view without Root
cx.new(|_| MyView::new())
```

The Root view provides:
- Sheet management (side panels)
- Dialog management
- Notification system
- Keyboard navigation (Tab/Shift-Tab)

### 2. Component Initialization

Always call `gpui_component::init(cx)` before using components:

```rust
fn main() {
    let app = Application::new().with_assets(gpui_component_assets::Assets);

    app.run(move |cx| {
        // This MUST be first
        gpui_component::init(cx);
        
        // Now you can use GPUI Component features
        // ...
    });
}
```

### 3. Theme Access

Access the current theme using the `ActiveTheme` trait:

```rust
use gpui_component::ActiveTheme;

impl Render for MyView {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .bg(theme.background)
            .text_color(theme.foreground)
            .child("Themed content")
    }
}
```

### 4. Component Builder Pattern

Most components use a builder pattern:

```rust
Button::new("save-button")
    .label("Save Changes")
    .primary()
    .large()
    .tooltip("Save your work")
    .on_click(|_event, cx| {
        // Handle click
    })
```

## First Component Example

Create a custom component following GPUI Component patterns:

**src/components/counter.rs:**
```rust
use gpui::*;
use gpui_component::prelude::*;

#[derive(IntoElement)]
pub struct Counter {
    id: ElementId,
    count: i32,
    on_change: Option<Box<dyn Fn(i32, &mut WindowContext) + 'static>>,
}

impl Counter {
    pub fn new(id: impl Into<ElementId>) -> Self {
        Self {
            id: id.into(),
            count: 0,
            on_change: None,
        }
    }

    pub fn count(mut self, count: i32) -> Self {
        self.count = count;
        self
    }

    pub fn on_change<F>(mut self, handler: F) -> Self
    where
        F: Fn(i32, &mut WindowContext) + 'static,
    {
        self.on_change = Some(Box::new(handler));
        self
    }
}

impl RenderOnce for Counter {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        let count = self.count;
        let on_change = self.on_change;

        div()
            .flex()
            .items_center()
            .gap_2()
            .child(
                Button::new(format!("{}-decrement", self.id))
                    .label("-")
                    .on_click(move |_event, cx| {
                        if let Some(ref handler) = on_change {
                            handler(count - 1, cx);
                        }
                    })
            )
            .child(
                div()
                    .px_4()
                    .py_2()
                    .border_1()
                    .border_color(cx.theme().colors().border)
                    .rounded_md()
                    .child(format!("{}", count))
            )
            .child(
                Button::new(format!("{}-increment", self.id))
                    .label("+")
                    .on_click(move |_event, cx| {
                        if let Some(ref handler) = on_change {
                            handler(count + 1, cx);
                        }
                    })
            )
    }
}
```

**Usage:**
```rust
Counter::new("my-counter")
    .count(5)
    .on_change(|new_count, _cx| {
        println!("Count changed to: {}", new_count);
    })
```

## Development Workflow

### 1. Development Server

Use cargo-watch for auto-rebuild:

```bash
# Install cargo-watch
cargo install cargo-watch

# Run with auto-rebuild
cargo watch -x run
```

### 2. Component Development Cycle

1. **Create component** in `src/components/`
2. **Add to Story Gallery** for visual testing
3. **Write tests** following project patterns
4. **Document usage** with examples
5. **Integrate** into main application

### 3. Story Gallery Usage

The Story Gallery (`cargo run`) is essential for component development:

- **Visual testing** - See components in isolation
- **Interactive testing** - Test different states and props
- **Documentation** - Examples serve as documentation
- **Regression testing** - Catch visual regressions

Add your components to the gallery:

**crates/story/src/stories/my_component_story.rs:**
```rust
use gpui::*;
use gpui_component::prelude::*;
use crate::components::Counter;

pub fn counter_story(cx: &mut WindowContext) -> impl IntoElement {
    StoryContainer::new("Counter", cx)
        .child(
            section!("Basic Usage")
                .child(Counter::new("basic").count(0))
        )
        .child(
            section!("With Initial Value")
                .child(Counter::new("initial").count(10))
        )
        .child(
            section!("Interactive Example")
                .child(Counter::new("interactive").on_change(|count, _| {
                    println!("Interactive counter: {}", count);
                }))
        )
}
```

## Testing Setup

### Unit Tests

Create tests following project patterns:

**src/components/counter.rs:**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use gpui::TestAppContext;

    #[gpui::test]
    fn test_counter_builder(cx: &mut TestAppContext) {
        let counter = Counter::new("test-counter")
            .count(5);

        assert_eq!(counter.count, 5);
        assert_eq!(counter.id, ElementId::from("test-counter"));
    }

    #[test]
    fn test_counter_increment_logic() {
        // Test pure logic without GPUI context
        let initial_count = 5;
        let incremented = initial_count + 1;
        assert_eq!(incremented, 6);
    }
}
```

### Integration Tests

**tests/app_tests.rs:**
```rust
use gpui::TestAppContext;
use my_gpui_app::App;

#[gpui::test]
fn test_app_initialization(cx: &mut TestAppContext) {
    // Test app initialization
    gpui_component::init(cx);
    
    // Add your integration tests here
}
```

## Common Patterns

### State Management

**Local State (Simple):**
```rust
struct MyView {
    counter: i32,
}

impl MyView {
    fn increment(&mut self, cx: &mut ViewContext<Self>) {
        self.counter += 1;
        cx.notify(); // Trigger re-render
    }
}
```

**Global State:**
```rust
use gpui_component::prelude::*;

#[derive(Clone)]
struct AppState {
    user_name: String,
    theme_mode: ThemeMode,
}

impl Global for AppState {}

// Set global state
cx.set_global(AppState {
    user_name: "John".to_string(),
    theme_mode: ThemeMode::Dark,
});

// Access global state
let state = cx.global::<AppState>();
```

### Event Handling

**Button Clicks:**
```rust
Button::new("action-button")
    .label("Perform Action")
    .on_click(cx.listener(|this, _event, cx| {
        this.perform_action(cx);
    }))
```

**Keyboard Shortcuts:**
```rust
use gpui_component::action;

action!(PerformAction);

impl MyView {
    fn register_actions(cx: &mut ViewContext<Self>) {
        cx.on_action(|this, _: &PerformAction, cx| {
            this.perform_action(cx);
        });
    }
}
```

## Next Steps

1. **Explore Components**: Run `cargo run` to see the Story Gallery
2. **Read Component Patterns**: Check the "component-patterns" steering file
3. **Learn Theme System**: Read "theme-styling" for visual customization
4. **Study Examples**: Look at `examples/` directory for real usage
5. **Join Community**: Contribute to the project and ask questions

You're now ready to start building with GPUI Component! The Story Gallery is your best friend for development and testing.