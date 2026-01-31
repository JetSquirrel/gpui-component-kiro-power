# Component Development Patterns

Comprehensive guide to developing components with GPUI Component library patterns and best practices.

## Component Architecture

### Stateless vs Stateful Components

**Stateless Components (Preferred):**
Use `RenderOnce` trait for components without internal state:

```rust
use gpui_component::prelude::*;

#[derive(IntoElement)]
pub struct Badge {
    label: SharedString,
    variant: BadgeVariant,
    size: Size,
}

impl Badge {
    pub fn new(label: impl Into<SharedString>) -> Self {
        Self {
            label: label.into(),
            variant: BadgeVariant::Default,
            size: Size::Medium,
        }
    }

    pub fn primary(mut self) -> Self {
        self.variant = BadgeVariant::Primary;
        self
    }

    pub fn large(mut self) -> Self {
        self.size = Size::Large;
        self
    }
}

impl RenderOnce for Badge {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        div()
            .px_2()
            .py_1()
            .rounded_md()
            .text_sm()
            .when(self.variant == BadgeVariant::Primary, |el| {
                el.bg(cx.theme().colors().primary)
                  .text_color(cx.theme().colors().primary_foreground)
            })
            .when(self.size == Size::Large, |el| {
                el.px_3().py_2().text_base()
            })
            .child(self.label)
    }
}
```

**Stateful Components:**
Use `Render` trait for components with internal state:

```rust
pub struct Select<T> {
    id: ElementId,
    options: Vec<SelectOption<T>>,
    selected: Option<usize>,
    placeholder: Option<SharedString>,
    on_change: Option<Box<dyn Fn(Option<&T>, &mut WindowContext)>>,
}

impl<T> Select<T> {
    pub fn new(id: impl Into<ElementId>) -> Self {
        Self {
            id: id.into(),
            options: Vec::new(),
            selected: None,
            placeholder: None,
            on_change: None,
        }
    }

    pub fn option(mut self, value: T, label: impl Into<SharedString>) -> Self {
        self.options.push(SelectOption {
            value,
            label: label.into(),
        });
        self
    }

    pub fn selected(mut self, index: usize) -> Self {
        self.selected = Some(index);
        self
    }

    pub fn on_change<F>(mut self, handler: F) -> Self
    where
        F: Fn(Option<&T>, &mut WindowContext) + 'static,
    {
        self.on_change = Some(Box::new(handler));
        self
    }
}

impl<T> Render for Select<T> {
    fn render(&mut self, cx: &mut ViewContext<Self>) -> impl IntoElement {
        // Implementation with state management
        div()
            .id(self.id.clone())
            .child("Select implementation")
    }
}
```

### Builder Pattern Implementation

All components should follow the builder pattern for consistent API:

```rust
#[derive(IntoElement)]
pub struct Button {
    id: ElementId,
    label: Option<SharedString>,
    variant: ButtonVariant,
    size: Size,
    disabled: bool,
    loading: bool,
    on_click: Option<Box<dyn Fn(&ClickEvent, &mut WindowContext)>>,
}

impl Button {
    pub fn new(id: impl Into<ElementId>) -> Self {
        Self {
            id: id.into(),
            label: None,
            variant: ButtonVariant::Default,
            size: Size::Medium,
            disabled: false,
            loading: false,
            on_click: None,
        }
    }

    // Fluent API methods
    pub fn label(mut self, label: impl Into<SharedString>) -> Self {
        self.label = Some(label.into());
        self
    }

    pub fn primary(mut self) -> Self {
        self.variant = ButtonVariant::Primary;
        self
    }

    pub fn disabled(mut self, disabled: bool) -> Self {
        self.disabled = disabled;
        self
    }

    pub fn on_click<F>(mut self, handler: F) -> Self
    where
        F: Fn(&ClickEvent, &mut WindowContext) + 'static,
    {
        self.on_click = Some(Box::new(handler));
        self
    }

    // Helper methods
    pub fn clickable(&self) -> bool {
        !self.disabled && !self.loading && self.on_click.is_some()
    }
}
```

## Size System Implementation

### Sizable Trait

Implement the `Sizable` trait for consistent sizing:

```rust
use gpui_component::Sizable;

impl Sizable for Button {
    fn with_size(mut self, size: Size) -> Self {
        self.size = size;
        self
    }
}

// This automatically provides:
// .xsmall(), .small(), .medium(), .large()
```

### Size-Based Styling

Apply different styles based on size:

```rust
impl RenderOnce for Button {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        div()
            .flex()
            .items_center()
            .justify_center()
            .rounded_md()
            .font_medium()
            .when(self.size == Size::XSmall, |el| {
                el.px_2().py_1().text_xs()
            })
            .when(self.size == Size::Small, |el| {
                el.px_3().py_1().text_sm()
            })
            .when(self.size == Size::Medium, |el| {
                el.px_4().py_2().text_sm()
            })
            .when(self.size == Size::Large, |el| {
                el.px_6().py_3().text_base()
            })
            .child(self.label.unwrap_or_default())
    }
}
```

## Variant System

### Defining Variants

Create enums for component variants:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ButtonVariant {
    Default,
    Primary,
    Secondary,
    Destructive,
    Outline,
    Ghost,
    Link,
}

impl ButtonVariant {
    pub fn is_link(&self) -> bool {
        matches!(self, Self::Link)
    }

    pub fn is_text(&self) -> bool {
        matches!(self, Self::Ghost | Self::Link)
    }

    pub fn no_padding(&self) -> bool {
        matches!(self, Self::Link | Self::Ghost)
    }
}
```

### Variant-Based Styling

Apply styles based on variant:

```rust
impl RenderOnce for Button {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .when(self.variant == ButtonVariant::Primary, |el| {
                el.bg(theme.colors().primary)
                  .text_color(theme.colors().primary_foreground)
                  .hover(|el| el.bg(theme.colors().primary.opacity(0.9)))
            })
            .when(self.variant == ButtonVariant::Outline, |el| {
                el.border_1()
                  .border_color(theme.colors().border)
                  .hover(|el| el.bg(theme.colors().accent))
            })
            .when(self.variant == ButtonVariant::Ghost, |el| {
                el.hover(|el| el.bg(theme.colors().accent))
            })
            .when(self.variant.is_link(), |el| {
                el.cursor_pointer() // Links use pointer cursor
            })
            .when(!self.variant.is_link(), |el| {
                el.cursor_default() // Buttons use default cursor
            })
            .child(self.label.unwrap_or_default())
    }
}
```

## Event Handling Patterns

### Click Events

Standard click event handling:

```rust
pub struct Button {
    on_click: Option<Box<dyn Fn(&ClickEvent, &mut WindowContext)>>,
}

impl Button {
    pub fn on_click<F>(mut self, handler: F) -> Self
    where
        F: Fn(&ClickEvent, &mut WindowContext) + 'static,
    {
        self.on_click = Some(Box::new(handler));
        self
    }
}

impl RenderOnce for Button {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        div()
            .when_some(self.on_click, |el, handler| {
                el.on_click(move |event, cx| handler(event, cx))
            })
            .child("Button")
    }
}
```

### Change Events

For input components:

```rust
pub struct Input {
    on_change: Option<Box<dyn Fn(&str, &mut WindowContext)>>,
}

impl Input {
    pub fn on_change<F>(mut self, handler: F) -> Self
    where
        F: Fn(&str, &mut WindowContext) + 'static,
    {
        self.on_change = Some(Box::new(handler));
        self
    }
}
```

### Custom Events

Define custom events for complex interactions:

```rust
#[derive(Clone, Debug)]
pub struct SelectionChanged {
    pub selected_index: Option<usize>,
    pub selected_value: String,
}

pub struct Select {
    on_selection_changed: Option<Box<dyn Fn(&SelectionChanged, &mut WindowContext)>>,
}

impl Select {
    pub fn on_selection_changed<F>(mut self, handler: F) -> Self
    where
        F: Fn(&SelectionChanged, &mut WindowContext) + 'static,
    {
        self.on_selection_changed = Some(Box::new(handler));
        self
    }

    fn handle_selection(&self, index: usize, value: String, cx: &mut WindowContext) {
        if let Some(ref handler) = self.on_selection_changed {
            handler(&SelectionChanged {
                selected_index: Some(index),
                selected_value: value,
            }, cx);
        }
    }
}
```

## Styling Patterns

### Theme Integration

Always use theme colors and values:

```rust
impl RenderOnce for Card {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        let theme = cx.theme();
        
        div()
            .bg(theme.colors().card)
            .border_1()
            .border_color(theme.colors().border)
            .rounded_lg()
            .shadow_sm()
            .p_6()
            .child(self.content)
    }
}
```

### Conditional Styling

Use `when` for conditional styles:

```rust
div()
    .when(self.disabled, |el| {
        el.opacity(0.5).cursor_not_allowed()
    })
    .when(self.selected, |el| {
        el.bg(cx.theme().colors().accent)
    })
    .when_some(self.error, |el, error| {
        el.border_color(cx.theme().colors().destructive)
          .child(div().text_color(cx.theme().colors().destructive).child(error))
    })
```

### Responsive Design

Handle different screen sizes:

```rust
div()
    .flex()
    .when(cx.viewport_size().width < px(768.0), |el| {
        el.flex_col() // Stack vertically on small screens
    })
    .when(cx.viewport_size().width >= px(768.0), |el| {
        el.flex_row().gap_4() // Horizontal layout on larger screens
    })
```

## State Management Patterns

### Local Component State

For components that need internal state:

```rust
pub struct Toggle {
    checked: bool,
    on_change: Option<Box<dyn Fn(bool, &mut WindowContext)>>,
}

impl Toggle {
    pub fn checked(mut self, checked: bool) -> Self {
        self.checked = checked;
        self
    }

    fn toggle(&mut self, cx: &mut ViewContext<Self>) {
        self.checked = !self.checked;
        if let Some(ref handler) = self.on_change {
            handler(self.checked, cx);
        }
        cx.notify(); // Trigger re-render
    }
}

impl Render for Toggle {
    fn render(&mut self, cx: &mut ViewContext<Self>) -> impl IntoElement {
        div()
            .on_click(cx.listener(|this, _event, cx| {
                this.toggle(cx);
            }))
            .when(self.checked, |el| {
                el.bg(cx.theme().colors().primary)
            })
            .child("Toggle")
    }
}
```

### Shared State

Use global state for data shared across components:

```rust
#[derive(Clone)]
pub struct AppSettings {
    pub theme_mode: ThemeMode,
    pub language: String,
}

impl Global for AppSettings {}

// Set global state
cx.set_global(AppSettings {
    theme_mode: ThemeMode::Dark,
    language: "en".to_string(),
});

// Access in components
impl Render for SettingsView {
    fn render(&mut self, cx: &mut ViewContext<Self>) -> impl IntoElement {
        let settings = cx.global::<AppSettings>();
        
        div()
            .child(format!("Current theme: {:?}", settings.theme_mode))
            .child(format!("Language: {}", settings.language))
    }
}
```

## Accessibility Patterns

### Keyboard Navigation

Support keyboard navigation:

```rust
impl RenderOnce for Button {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        div()
            .id(self.id.clone())
            .focusable()
            .on_key_down(|event, cx| {
                if event.keystroke.key == "Enter" || event.keystroke.key == " " {
                    // Trigger click
                    if let Some(ref handler) = self.on_click {
                        handler(&ClickEvent::default(), cx);
                    }
                }
            })
            .child(self.label.unwrap_or_default())
    }
}
```

### ARIA Labels

Provide accessible labels:

```rust
div()
    .role("button")
    .aria_label("Close dialog")
    .aria_pressed(self.pressed)
    .when(self.disabled, |el| {
        el.aria_disabled(true)
    })
```

## Error Handling Patterns

### Validation States

Handle validation and error states:

```rust
#[derive(IntoElement)]
pub struct Input {
    value: String,
    error: Option<SharedString>,
    required: bool,
}

impl Input {
    pub fn error(mut self, error: impl Into<SharedString>) -> Self {
        self.error = Some(error.into());
        self
    }

    pub fn required(mut self) -> Self {
        self.required = true;
        self
    }

    fn validate(&self) -> Option<String> {
        if self.required && self.value.trim().is_empty() {
            Some("This field is required".to_string())
        } else {
            None
        }
    }
}

impl RenderOnce for Input {
    fn render(self, cx: &mut WindowContext) -> impl IntoElement {
        let has_error = self.error.is_some();
        
        div()
            .flex()
            .flex_col()
            .gap_1()
            .child(
                input()
                    .value(self.value)
                    .border_1()
                    .when(has_error, |el| {
                        el.border_color(cx.theme().colors().destructive)
                    })
                    .when(!has_error, |el| {
                        el.border_color(cx.theme().colors().border)
                    })
            )
            .when_some(self.error, |el, error| {
                el.child(
                    div()
                        .text_sm()
                        .text_color(cx.theme().colors().destructive)
                        .child(error)
                )
            })
    }
}
```

## Performance Patterns

### Memoization

Use memoization for expensive computations:

```rust
use std::sync::Arc;

#[derive(IntoElement)]
pub struct ExpensiveComponent {
    data: Arc<Vec<String>>,
    cached_result: Option<String>,
}

impl ExpensiveComponent {
    fn compute_expensive_result(&mut self) -> &str {
        if self.cached_result.is_none() {
            // Expensive computation
            let result = self.data.iter()
                .map(|s| s.to_uppercase())
                .collect::<Vec<_>>()
                .join(", ");
            self.cached_result = Some(result);
        }
        self.cached_result.as_ref().unwrap()
    }
}
```

### Lazy Loading

Implement lazy loading for large datasets:

```rust
pub struct LazyList<T> {
    items: Vec<T>,
    visible_range: Range<usize>,
    item_height: Pixels,
}

impl<T> LazyList<T> {
    fn render_visible_items(&self, cx: &mut WindowContext) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .children(
                self.items[self.visible_range.clone()]
                    .iter()
                    .enumerate()
                    .map(|(index, item)| {
                        self.render_item(index, item, cx)
                    })
            )
    }
}
```

## Testing Patterns

### Component Testing

Test components following project patterns:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use gpui::TestAppContext;

    #[gpui::test]
    fn test_button_builder(cx: &mut TestAppContext) {
        let button = Button::new("test-button")
            .label("Click Me")
            .primary()
            .large()
            .disabled(false);

        assert_eq!(button.label, Some("Click Me".into()));
        assert_eq!(button.variant, ButtonVariant::Primary);
        assert_eq!(button.size, Size::Large);
        assert!(!button.disabled);
    }

    #[gpui::test]
    fn test_button_clickable_logic(cx: &mut TestAppContext) {
        let clickable = Button::new("test").on_click(|_, _| {});
        assert!(clickable.clickable());

        let disabled = Button::new("test").disabled(true).on_click(|_, _| {});
        assert!(!disabled.clickable());

        let loading = Button::new("test").loading(true).on_click(|_, _| {});
        assert!(!loading.clickable());
    }

    #[test]
    fn test_button_variant_methods() {
        assert!(ButtonVariant::Link.is_link());
        assert!(ButtonVariant::Ghost.is_text());
        assert!(ButtonVariant::Link.no_padding());
    }
}
```

## Documentation Patterns

### Component Documentation

Document components with examples:

```rust
/// A flexible button component with multiple variants and sizes.
/// 
/// # Examples
/// 
/// ```rust
/// // Basic button
/// Button::new("save")
///     .label("Save")
///     .primary()
/// 
/// // Button with click handler
/// Button::new("delete")
///     .label("Delete")
///     .destructive()
///     .on_click(|event, cx| {
///         // Handle deletion
///     })
/// 
/// // Disabled button
/// Button::new("submit")
///     .label("Submit")
///     .disabled(true)
/// ```
pub struct Button {
    // ...
}
```

### Story Examples

Create comprehensive stories for the gallery:

```rust
pub fn button_story(cx: &mut WindowContext) -> impl IntoElement {
    StoryContainer::new("Button", cx)
        .child(
            section!("Variants")
                .child(Button::new("default").label("Default"))
                .child(Button::new("primary").label("Primary").primary())
                .child(Button::new("secondary").label("Secondary").secondary())
                .child(Button::new("destructive").label("Destructive").destructive())
        )
        .child(
            section!("Sizes")
                .child(Button::new("xs").label("Extra Small").xsmall())
                .child(Button::new("sm").label("Small").small())
                .child(Button::new("md").label("Medium"))
                .child(Button::new("lg").label("Large").large())
        )
        .child(
            section!("States")
                .child(Button::new("disabled").label("Disabled").disabled(true))
                .child(Button::new("loading").label("Loading").loading(true))
        )
}
```

These patterns provide a solid foundation for building consistent, maintainable, and accessible components with the GPUI Component library.