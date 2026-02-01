# Theme System and Styling Guide

Complete guide to the GPUI Component theme system, styling approaches, and visual design patterns.

## Theme System Overview

GPUI Component uses a comprehensive theme system that supports:
- **Light/Dark mode switching**
- **Consistent color palettes**
- **Typography configuration**
- **Syntax highlighting themes**
- **UI parameters (borders, shadows, spacing)**

### Theme Access

Access the current theme using the `ActiveTheme` trait:

```rust
use gpui::*;
use gpui_component::ActiveTheme;

impl Render for MyView {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .bg(theme.background)
            .text_color(theme.foreground)
            .border_1()
            .border_color(theme.border)
            .child("Themed content")
    }
}
```

## Color System

### Core Color Palette

The theme provides a comprehensive color system:

```rust
let theme = cx.theme();

// Background colors
theme.background          // Main background
theme.foreground          // Main text color
theme.card                // Card backgrounds
theme.popover             // Popover backgrounds
theme.muted               // Muted backgrounds
theme.muted_foreground    // Muted text

// Border and accent colors
theme.border              // Default borders
theme.input               // Input borders
theme.ring                // Focus rings
theme.accent              // Accent backgrounds
theme.accent_foreground   // Accent text

// Semantic colors
theme.primary             // Primary actions
theme.primary_foreground  // Primary text
theme.secondary           // Secondary actions
theme.secondary_foreground // Secondary text
theme.destructive         // Destructive actions
theme.destructive_foreground // Destructive text

// Status colors (if available)
// theme.success           // Success states
// theme.warning           // Warning states
// theme.error             // Error states
```

### Using Colors in Components

```rust
impl RenderOnce for StatusBadge {
    fn render(self, _window: &mut Window, cx: &mut App) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .px_2()
            .py_1()
            .rounded_md()
            .text_sm()
            .when(self.status == Status::Success, |el| {
                el.bg(theme.primary)  // Use primary for success
                  .text_color(theme.primary_foreground)
            })
            .when(self.status == Status::Warning, |el| {
                el.bg(theme.accent)
                  .text_color(theme.accent_foreground)
            })
            .when(self.status == Status::Error, |el| {
                el.bg(theme.destructive)
                  .text_color(theme.destructive_foreground)
            })
            .child(self.message)
    }
}
```

### Color Opacity and Variations

Create color variations using opacity:

```rust
// Semi-transparent backgrounds
div()
    .bg(theme.primary.opacity(0.1))  // 10% opacity
    .border_color(theme.primary.opacity(0.3))

// Hover states with opacity
div()
    .bg(theme.accent)
    .hover(|el| {
        el.bg(theme.accent.opacity(0.8))
    })
```

## Typography System

### Font Configuration

The theme includes font configuration:

```rust
let theme = cx.theme();

// System font (UI text)
let ui_font = theme.font_family();

// Monospace font (code)
let mono_font = theme.font_mono();

// Using fonts in elements
div()
    .font_family(ui_font)
    .text_base()
    .child("UI Text")

div()
    .font_family(mono_font)
    .text_sm()
    .child("Code text")
```

### Text Sizing

Use consistent text sizes:

```rust
div()
    .text_xs()      // 12px
    .text_sm()      // 14px
    .text_base()    // 16px (default)
    .text_lg()      // 18px
    .text_xl()      // 20px
    .text_2xl()     // 24px
    .text_3xl()     // 30px
```

### Text Styling

Apply text styling consistently:

```rust
div()
    .font_normal()     // Normal weight
    .font_medium()     // Medium weight
    .font_semibold()   // Semi-bold
    .font_bold()       // Bold
    .italic()          // Italic
    .underline()       // Underlined
    .line_through()    // Strikethrough
```

## Layout and Spacing

### Spacing System

Use consistent spacing values:

```rust
div()
    .p_1()    // 4px padding
    .p_2()    // 8px padding
    .p_3()    // 12px padding
    .p_4()    // 16px padding
    .p_6()    // 24px padding
    .p_8()    // 32px padding
    
    .m_1()    // 4px margin
    .m_2()    // 8px margin
    .m_4()    // 16px margin
    
    .gap_1()  // 4px gap
    .gap_2()  // 8px gap
    .gap_4()  // 16px gap
```

### Flexbox Layouts

Common flexbox patterns:

```rust
// Horizontal layout with center alignment
div()
    .flex()
    .items_center()
    .justify_center()
    .gap_2()

// Vertical stack
div()
    .flex()
    .flex_col()
    .gap_4()

// Space between items
div()
    .flex()
    .justify_between()
    .items_center()

// Responsive flex direction
div()
    .flex()
    .when(cx.viewport_size().width < px(768.0), |el| {
        el.flex_col()
    })
    .when(cx.viewport_size().width >= px(768.0), |el| {
        el.flex_row()
    })
```

### Grid Layouts

Use CSS Grid for complex layouts:

```rust
div()
    .grid()
    .grid_cols_3()      // 3 columns
    .gap_4()
    .children(items.into_iter().map(|item| {
        div()
            .bg(theme.colors().card)
            .p_4()
            .rounded_lg()
            .child(item)
    }))
```

## Border and Shadow System

### Borders

Apply consistent borders:

```rust
div()
    .border_1()                              // 1px border
    .border_2()                              // 2px border
    .border_color(theme.colors().border)     // Border color
    .border_t_1()                            // Top border only
    .border_b_1()                            // Bottom border only
    .border_l_1()                            // Left border only
    .border_r_1()                            // Right border only
```

### Border Radius

Use consistent border radius:

```rust
div()
    .rounded_none()    // No radius
    .rounded_sm()      // Small radius
    .rounded_md()      // Medium radius (default)
    .rounded_lg()      // Large radius
    .rounded_xl()      // Extra large radius
    .rounded_full()    // Fully rounded (pills/circles)
```

### Shadows

Apply shadows for depth:

```rust
div()
    .shadow_sm()       // Small shadow
    .shadow_md()       // Medium shadow
    .shadow_lg()       // Large shadow
    .shadow_xl()       // Extra large shadow
```

## Component Styling Patterns

### Card Component

Standard card styling:

```rust
#[derive(IntoElement)]
pub struct Card {
    children: Vec<AnyElement>,
}

impl RenderOnce for Card {
    fn render(self, _window: &mut Window, cx: &mut App) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .bg(theme.card)
            .border_1()
            .border_color(theme.border)
            .rounded_lg()
            .shadow_sm()
            .p_6()
            .children(self.children)
    }
}
```

### Input Component

Styled input with focus states:

```rust
impl RenderOnce for Input {
    fn render(self, _window: &mut Window, cx: &mut App) -> impl IntoElement {
        let theme = cx.theme();
        
        // Note: For actual input, use gpui_component::input::InputState
        div()
            .bg(theme.background)
            .border_1()
            .border_color(theme.input)
            .rounded_md()
            .px_3()
            .py_2()
            .text_sm()
            .child(self.value.unwrap_or_default())
            .when(self.error.is_some(), |el| {
                el.border_color(theme.destructive)
            })
    }
}
```

### Button Variants

Different button styles:

```rust
impl RenderOnce for Button {
    fn render(self, _window: &mut Window, cx: &mut App) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .flex()
            .items_center()
            .justify_center()
            .px_4()
            .py_2()
            .rounded_md()
            .font_medium()
            .text_sm()
            .cursor_default()  // Desktop convention
            .when(self.variant == ButtonVariant::Primary, |el| {
                el.bg(theme.primary)
                  .text_color(theme.primary_foreground)
                  .hover(|el| el.bg(theme.primary.opacity(0.9)))
            })
            .when(self.variant == ButtonVariant::Secondary, |el| {
                el.bg(theme.secondary)
                  .text_color(theme.secondary_foreground)
                  .hover(|el| el.bg(theme.secondary.opacity(0.8)))
            })
            .when(self.variant == ButtonVariant::Outline, |el| {
                el.border_1()
                  .border_color(theme.input)
                  .bg(theme.background)
                  .hover(|el| el.bg(theme.accent))
            })
            .when(self.variant == ButtonVariant::Ghost, |el| {
                el.hover(|el| el.bg(theme.accent))
            })
            .when(self.variant == ButtonVariant::Link, |el| {
                el.text_color(theme.primary)
                  .underline()
                  .cursor_pointer()  // Links use pointer cursor
            })
            .when(self.disabled, |el| {
                el.opacity(0.5)
                  .cursor_not_allowed()
            })
            .child(self.label.unwrap_or_default())
    }
}
```

## Interactive States

### Hover States

Add hover effects:

```rust
// Inside RenderOnce::render(self, _window: &mut Window, cx: &mut App)
let theme = cx.theme();

div()
    .bg(theme.card)
    .hover(|el| {
        el.bg(theme.accent)
          .shadow_md()
    })
    .transition_all(Duration::from_millis(150))
```

### Focus States

Implement focus indicators:

```rust
// Inside RenderOnce::render
let theme = cx.theme();

div()
    .focusable()
    .focus(|el| {
        el.outline_none()
          .ring_2()
          .ring_color(theme.ring)
          .ring_offset_2()
    })
```

### Active States

Handle active/pressed states:

```rust
// Inside RenderOnce::render
let theme = cx.theme();

div()
    .active(|el| {
        el.scale(0.95)
          .bg(theme.accent.opacity(0.8))
    })
```

### Disabled States

Style disabled elements:

```rust
div()
    .when(self.disabled, |el| {
        el.opacity(0.5)
          .cursor_not_allowed()
          .pointer_events_none()
    })
```

## Dark Mode Support

### Automatic Theme Switching

The theme system automatically handles light/dark mode:

```rust
// Colors automatically adjust based on current theme mode
// Inside RenderOnce::render(self, _window: &mut Window, cx: &mut App)
let theme = cx.theme();

div()
    .bg(theme.background)     // White in light, dark in dark mode
    .text_color(theme.foreground) // Black in light, white in dark mode
```

### Theme Mode Detection

Check current theme mode:

```rust
use gpui_component::theme::ThemeMode;
use gpui_component::ActiveTheme;

// Inside render method
let theme = cx.theme();
let is_dark = theme.mode == ThemeMode::Dark;

div()
    .when(is_dark, |el| {
        el.child("Dark mode specific content")
    })
    .when(!is_dark, |el| {
        el.child("Light mode specific content")
    })
```

## Custom Theme Creation

### Theme Configuration

Create custom themes by extending the base theme:

```rust
use gpui_component::theme::{Theme, ThemeColors};

let custom_colors = ThemeColors {
    primary: rgb(0x3b82f6),           // Custom blue
    primary_foreground: rgb(0xffffff),
    secondary: rgb(0x64748b),         // Custom gray
    // ... other colors
};

let custom_theme = Theme::new()
    .with_colors(custom_colors)
    .with_font_family("Inter".into())
    .with_font_mono("JetBrains Mono".into());
```

### Theme JSON Files

Define themes in JSON files:

```json
{
  "name": "Custom Theme",
  "mode": "light",
  "colors": {
    "background": "#ffffff",
    "foreground": "#0f172a",
    "primary": "#3b82f6",
    "primary_foreground": "#ffffff",
    "secondary": "#64748b",
    "secondary_foreground": "#ffffff",
    "accent": "#f1f5f9",
    "accent_foreground": "#0f172a",
    "destructive": "#ef4444",
    "destructive_foreground": "#ffffff",
    "border": "#e2e8f0",
    "input": "#e2e8f0",
    "ring": "#3b82f6"
  },
  "fonts": {
    "ui": "Inter",
    "mono": "JetBrains Mono"
  }
}
```

## Responsive Design

### Viewport-Based Styling

Adapt to different screen sizes:

```rust
let viewport = cx.viewport_size();
let is_mobile = viewport.width < px(768.0);
let is_tablet = viewport.width >= px(768.0) && viewport.width < px(1024.0);
let is_desktop = viewport.width >= px(1024.0);

div()
    .when(is_mobile, |el| {
        el.flex_col().p_4()
    })
    .when(is_tablet, |el| {
        el.flex_row().p_6()
    })
    .when(is_desktop, |el| {
        el.grid().grid_cols_3().p_8()
    })
```

### Container Queries

Use container-based responsive design:

```rust
div()
    .w_full()
    .max_w_4xl()  // Max width constraint
    .mx_auto()    // Center horizontally
    .px_4()       // Horizontal padding
    .when(viewport.width >= px(640.0), |el| {
        el.px_6()
    })
    .when(viewport.width >= px(1024.0), |el| {
        el.px_8()
    })
```

## Animation and Transitions

### CSS Transitions

Add smooth transitions:

```rust
div()
    .bg(theme.colors().card)
    .transition_all(Duration::from_millis(200))
    .hover(|el| {
        el.bg(theme.colors().accent)
          .shadow_lg()
    })
```

### Transform Animations

Apply transforms:

```rust
div()
    .scale(1.0)
    .transition_transform(Duration::from_millis(150))
    .hover(|el| {
        el.scale(1.05)
    })
    .active(|el| {
        el.scale(0.95)
    })
```

## Best Practices

### 1. Always Use Theme Colors

```rust
// ✅ Good - Uses theme colors
div().bg(theme.background)

// ❌ Bad - Hard-coded colors
div().bg(rgb(0xffffff))
```

### 2. Consistent Spacing

```rust
// ✅ Good - Uses spacing scale
div().p_4().m_2().gap_3()

// ❌ Bad - Custom spacing
div().p(px(17.0)).m(px(9.0))
```

### 3. Semantic Color Usage

```rust
// ✅ Good - Semantic colors
div().bg(theme.destructive)  // For errors

// ❌ Bad - Generic colors
div().bg(theme.primary)      // For errors
```

### 4. Responsive Design

```rust
// ✅ Good - Responsive layout
div()
    .flex()
    .when(is_mobile, |el| el.flex_col())
    .when(is_desktop, |el| el.flex_row())

// ❌ Bad - Fixed layout
div().flex().flex_row()
```

### 5. Accessibility

```rust
// ✅ Good - Sufficient contrast
div()
    .bg(theme.primary)
    .text_color(theme.primary_foreground)

// ✅ Good - Focus indicators
div()
    .focusable()
    .focus(|el| el.ring_2().ring_color(theme.ring))
```

## Troubleshooting

### Common Styling Issues

**Colors not updating with theme changes:**
```rust
// ✅ Always access theme in render method
impl Render for MyView {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let theme = cx.theme(); // Fresh theme access
        div().bg(theme.background)
    }
}
```

**Layout not responsive:**
```rust
// ✅ Check viewport size in render
let viewport = cx.viewport_size();
div().when(viewport.width < px(768.0), |el| el.flex_col())
```

**Focus states not working:**
```rust
// ✅ Make element focusable first
div()
    .focusable()  // Required for focus states
    .focus(|el| el.ring_2())
```

The theme system provides a solid foundation for creating beautiful, consistent, and accessible user interfaces with GPUI Component.