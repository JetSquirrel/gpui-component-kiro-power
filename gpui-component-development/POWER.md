---
name: "gpui-component-development"
displayName: "GPUI Component Development"
description: "Complete guide for developing cross-platform desktop UI components with GPUI. Covers architecture, best practices, testing, and common workflows for the 60+ component library."
keywords: ["gpui", "component", "desktop", "ui", "rust", "cross-platform"]
author: "GPUI Component Library"
---

# GPUI Component Development

## Overview

GPUI Component is a comprehensive UI component library for building desktop applications using [GPUI](https://gpui.rs). This power provides complete guidance for developing with the library's 60+ cross-platform desktop UI components, inspired by macOS/Windows controls and combined with shadcn/ui design principles.

This knowledge base covers everything from project setup and component development patterns to testing strategies and troubleshooting common issues. Whether you're new to GPUI or an experienced developer, this guide will help you build robust desktop applications efficiently.

**Key Areas Covered:**
- Project initialization and setup requirements
- Component development patterns and architecture
- Theme system and styling approaches
- Complex layout systems (Dock system)
- Stateful and stateless component patterns
- Desktop UI conventions and best practices

## Available Steering Files

This power includes specialized steering files for different aspects of GPUI Component development:

- **onboarding** - Complete setup guide, project initialization, and first component
- **component-patterns** - Component development patterns, architecture, and best practices
- **theme-styling** - Theme system, styling approaches, and visual design
- **dock-layouts** - Complex panel layouts and dock system usage

Call action "readSteering" to access specific topics as needed.

## Quick Start

### Critical First Step: Component Initialization
Add dependencies to your Cargo.toml:
```toml
[dependencies]
gpui = "0.2.2"
gpui-component = "0.5.0"
# Optional, for default bundled assets (Lucide icons)
gpui-component-assets = "0.5.0"
```

> **Note**: Check [crates.io](https://crates.io/crates/gpui-component) for the latest version.

**⚠️ IMPORTANT:** You MUST call `gpui_component::init(cx)` at your application's entry point before using any GPUI Component features.

```rust
fn main() {
    let app = Application::new().with_assets(gpui_component_assets::Assets);
    app.run(move |cx| {
        // This must be called first
        gpui_component::init(cx);

        cx.spawn(async move |cx| {
            cx.open_window(WindowOptions::default(), |window, cx| {
                let view = cx.new(|_| MyView);
                // The first level view in a window must be a Root
                cx.new(|cx| Root::new(view, window, cx))
            })?;
            Ok::<_, anyhow::Error>(())
        }).detach();
    });
}
```

### Root View Requirement

Every window MUST start with a `Root` view that manages:
- Sheet (side panels)
- Dialog (dialogs) 
- Notification (notifications)
- Keyboard navigation (Tab/Shift-Tab)

### Basic Project Structure

```
my-gpui-app/
├── Cargo.toml
├── src/
│   ├── main.rs          # Application entry point
│   ├── views/           # Application views
│   └── components/      # Custom components
├── themes/              # Custom themes (optional)
└── examples/            # Usage examples
```

## Core Architecture Principles

### 1. Stateless vs Stateful Components

**Stateless Elements (Preferred):**
Most components use stateless `RenderOnce` elements, making them simple and predictable.

**Stateful Components:**
Some components like `Dropdown`, `List`, `Table`, and `InputState` manage their own internal state. These use the `Render` trait and require creating an `Entity` to hold in your view struct.

```rust
// Stateful component example
struct MyView {
    input: Entity<InputState>,
}

impl MyView {
    fn new(window: &Window, cx: &mut Context<Self>) -> Self {
        let input = cx.new(|cx| InputState::new(window, cx).default_value("Hello"));
        Self { input }
    }
}
```

Here's a simple stateless example to get you started:

```rust
use gpui::*;
use gpui_component::{button::*, *};

pub struct HelloWorld;

impl Render for HelloWorld {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div()
            .v_flex()
            .gap_2()
            .size_full()
            .items_center()
            .justify_center()
            .child("Hello, World!")
            .child(
                Button::new("ok")
                    .primary()
                    .label("Let's Go!")
                    .on_click(|_, _, _| println!("Clicked!")),
            )
    }
}

fn main() {
    let app = Application::new().with_assets(gpui_component_assets::Assets);

    app.run(move |cx| {
        // This must be called before using any GPUI Component features.
        gpui_component::init(cx);

        cx.spawn(async move |cx| {
            cx.open_window(WindowOptions::default(), |window, cx| {
                let view = cx.new(|_| HelloWorld);
                // This first level on the window, should be a Root.
                cx.new(|cx| Root::new(view, window, cx))
            })?;

            Ok::<_, anyhow::Error>(())
        })
        .detach();
    });
}
```

### 2. Size System

Components support consistent sizing via the `Sizable` trait:
- `xs` - Extra small
- `sm` - Small  
- `md` - Medium (default)
- `lg` - Large

```rust
Button::new("save")
    .large()  // Sets size to lg
    .label("Save Changes")
```

### 3. Component Variants

Most components support visual variants:

```rust
Button::new("btn").primary()   // Primary action
Button::new("btn").danger()    // Destructive action
Button::new("btn").warning()   // Warning action
Button::new("btn").success()   // Success action
Button::new("btn").ghost()     // Ghost/transparent
Button::new("btn").outline()   // Outlined style
```

### 4. Desktop UI Conventions

- **Mouse cursor**: Buttons use `default` cursor, not `pointer` (desktop app convention)
- **Exception**: Link buttons use `pointer` cursor
- **Keyboard navigation**: Full Tab/Shift-Tab support through Root view
- **Platform consistency**: Follows macOS/Windows control patterns

## Common Development Commands

### Development and Testing

```bash
# Run Story Gallery (component showcase)
cargo run

# Run specific examples
cargo run --example hello_world
cargo run --example table

# Build the project
cargo build

# Lint check
cargo clippy -- --deny warnings

# Format code
cargo fmt --check

# Spell check
typos

# Check for unused dependencies
cargo machete
```

### Performance Profiling

```bash
# View FPS on macOS (Metal HUD)
MTL_HUD_ENABLED=1 cargo run

# Profile with samply
samply record cargo run
```

## gpui-component Repository Structure

If contributing to or exploring the gpui-component library itself:

- **`crates/ui`** - Core UI component library (published as `gpui-component`)
- **`crates/story`** - Gallery application for showcasing and testing components
- **`crates/macros`** - Procedural macros (`IntoPlot` derive)
- **`crates/assets`** - Static assets (published as `gpui-component-assets`)
- **`crates/webview`** - WebView component support
- **`examples/`** - Various example applications

For your own projects, see the Basic Project Structure in Quick Start above.

## Key Dependencies

- **GPUI**: Git version from Zed repository (core UI framework)
- **Tree-sitter**: Syntax highlighting support
- **Ropey**: Rope data structure for text with `RopeExt` trait extensions
- **rust-i18n**: Internationalization support
- **markdown**: Markdown rendering capabilities
- **html5ever**: Basic HTML rendering support
- **lsp-types**: LSP integration for text editing

## Platform Support

Fully tested and supported on:
- **macOS** (aarch64, x86_64)
- **Linux** (x86_64)  
- **Windows** (x86_64)

CI runs complete test suite on all platforms.

## Icon System

The `Icon` element requires SVG files to be provided:
- Use [Lucide](https://lucide.dev) or other icon libraries
- Name SVG files according to the `IconName` enum in `crates/ui/src/icon.rs`
- Icons are not included by default - you must provide them

## Internationalization

Uses `rust-i18n` crate with localization files in `crates/ui/locales/`:
- **Default languages**: `en`, `zh-CN`, `zh-HK`
- Add additional languages as needed
- Follow existing patterns for new translations

## Best Practices

### Component Development
- Follow existing naming patterns from macOS/Windows controls
- Reference existing components in `crates/ui/src` for patterns
- Use stateless design with `RenderOnce` when possible
- Implement `Sizable` trait for consistent sizing
- Follow CSS-like styling with `Styled` trait

### Code Style
- Follow naming and organization patterns from existing code
- AI-generated code must be refactored to match project style
- Mark AI-generated portions when submitting PRs
- Use desktop cursor conventions (default, not pointer)

### Testing Strategy
- Focus on complex logic and core functionality
- Every component should have a builder pattern test
- Avoid excessive simple tests
- Use property-based testing for comprehensive coverage

## Getting Help

### Documentation Resources
- **Official Docs**: https://longbridge.github.io/gpui-component/
- **API Reference**: https://docs.rs/gpui-component/latest/gpui_component/
- **GPUI Framework**: https://gpui.rs and https://docs.rs/gpui/latest/gpui/
- **Examples**: https://github.com/longbridge/gpui-component/tree/main/examples
- **Story Gallery**: Clone the repo and run `cargo run` to explore components interactively

## Next Steps

1. **For new projects**: Read the "onboarding" steering file for complete setup
2. **For component development**: Read "component-patterns" for architecture guidance  
3. **For styling**: Read "theme-styling" for theme system usage
4. **For complex layouts**: Read "dock-layouts" for panel management

This power provides everything you need to build robust, cross-platform desktop applications with GPUI Component. Use the steering files to dive deep into specific areas as needed.