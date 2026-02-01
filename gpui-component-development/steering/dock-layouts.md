# Dock Layouts Guide

Build complex panel layouts like VS Code or JetBrains IDE using the Dock system.

## Overview

The Dock system provides a complete IDE-style layout with:
- **DockArea**: Main container managing center area and left/bottom/right docks
- **DockItem**: Tree-based layout structure (Split, Tabs, Panel, Tiles)
- **Panel**: Content panels defined via `PanelView` trait
- **Layout persistence**: Save and restore layouts via JSON serialization

## Core Concepts

### DockArea

The main container that manages all docks:

```rust
use gpui::{Entity, Context, Window, SharedString};
use gpui_component::dock::{DockArea, DockItem, DockPlacement};

// Create a new DockArea
let dock_area: Entity<DockArea> = cx.new(|cx| {
    DockArea::new("main-dock", Some(1), window, cx)
});
```

Parameters:
- `id`: Unique identifier for the dock area
- `version`: Optional version for layout compatibility checks
- `window`: `&mut Window`
- `cx`: `&mut Context<Self>`

### DockItem Variants

DockItem is an enum representing different layout types:

```rust
pub enum DockItem {
    Split { axis, size, items, sizes, view },  // Split panels
    Tabs { size, items, active_ix, view },      // Tabbed panels
    Panel { size, view },                        // Single panel
    Tiles { size, items, view },                 // Tile layout
}
```

## Creating Layouts

### Tab Layout

Group panels into tabs:

```rust
use std::sync::Arc;
use gpui_component::dock::DockItem;

let weak_dock = dock_area.downgrade();

// Single tab
let single_tab = DockItem::tab(
    my_panel_entity,
    &weak_dock,
    window,
    cx,
);

// Multiple tabs
let tab_group = DockItem::tabs(
    vec![
        Arc::new(panel1_entity),
        Arc::new(panel2_entity),
        Arc::new(panel3_entity),
    ],
    &weak_dock,
    window,
    cx,
);
```

### Split Layout

Create horizontal or vertical splits:

```rust
use gpui::Axis;
use gpui_component::dock::DockItem;

let weak_dock = dock_area.downgrade();

// Vertical split
let v_split = DockItem::v_split(
    vec![top_item, bottom_item],
    &weak_dock,
    window,
    cx,
);

// Horizontal split
let h_split = DockItem::h_split(
    vec![left_item, right_item],
    &weak_dock,
    window,
    cx,
);

// Generic split with axis
let split = DockItem::split(
    Axis::Horizontal,
    vec![item1, item2, item3],
    &weak_dock,
    window,
    cx,
);

// Split with custom sizes
let sized_split = DockItem::split_with_sizes(
    Axis::Horizontal,
    vec![item1, item2],
    vec![Some(px(200.0)), None],  // First panel 200px, second auto
    &weak_dock,
    window,
    cx,
);
```

### Setting Item Size

Chain `.size()` to set the size of a DockItem:

```rust
let tabs = DockItem::tabs(panels, &weak_dock, window, cx)
    .size(px(360.0));
```

## Setting Up Docks

### Side Docks

Add left, right, or bottom docks:

```rust
dock_area.update(cx, |dock, cx| {
    // Left dock
    dock.set_left_dock(
        left_panels,      // DockItem
        Some(px(250.0)),  // Initial size (width)
        true,             // Open by default
        window,
        cx,
    );

    // Bottom dock
    dock.set_bottom_dock(
        bottom_panels,    // DockItem
        Some(px(200.0)),  // Initial size (height)
        true,             // Open by default
        window,
        cx,
    );

    // Right dock
    dock.set_right_dock(
        right_panels,     // DockItem
        Some(px(300.0)),  // Initial size (width)
        false,            // Closed by default
        window,
        cx,
    );
});
```

### Center Content

Set the main center area:

```rust
dock_area.update(cx, |dock, cx| {
    dock.set_center(center_dock_item, window, cx);
});
```

## Complete Example

Here's a full workspace setup:

```rust
use std::sync::Arc;
use gpui::{px, Axis, Context, Entity, Window};
use gpui_component::dock::{DockArea, DockItem, DockPlacement};

fn setup_workspace(window: &mut Window, cx: &mut Context<MyWorkspace>) -> Entity<DockArea> {
    cx.new(|cx| {
        let dock_area = DockArea::new("workspace", Some(1), window, cx);
        let weak_dock = cx.entity().downgrade();

        // Create left sidebar panels
        let left_panels = DockItem::v_split(
            vec![
                DockItem::tab(file_explorer_panel, &weak_dock, window, cx),
                DockItem::tabs(
                    vec![
                        Arc::new(search_panel),
                        Arc::new(git_panel),
                    ],
                    &weak_dock,
                    window,
                    cx,
                ).size(px(300.0)),
            ],
            &weak_dock,
            window,
            cx,
        );

        // Create bottom panel (terminal, output, etc.)
        let bottom_panels = DockItem::tabs(
            vec![
                Arc::new(terminal_panel),
                Arc::new(output_panel),
                Arc::new(problems_panel),
            ],
            &weak_dock,
            window,
            cx,
        );

        // Create center editor area
        let center_content = DockItem::tabs(
            vec![
                Arc::new(editor_panel_1),
                Arc::new(editor_panel_2),
            ],
            &weak_dock,
            window,
            cx,
        );

        // Assemble the layout
        dock_area.set_left_dock(left_panels, Some(px(250.0)), true, window, cx);
        dock_area.set_bottom_dock(bottom_panels, Some(px(200.0)), true, window, cx);
        dock_area.set_center(center_content, window, cx);

        dock_area
    })
}
```

## Creating Custom Panels

Implement the `Panel` trait for your custom panels:

```rust
use gpui::{Entity, SharedString, Window, Context, Render, IntoElement, div, Styled};
use gpui_component::dock::{Panel, PanelEvent, PanelState};

pub struct FileExplorer {
    // Your panel state
}

impl Panel for FileExplorer {
    fn panel_id(&self) -> SharedString {
        "file-explorer".into()
    }

    fn title(&self, _window: &Window, _cx: &App) -> SharedString {
        "Files".into()
    }

    fn popup_menu(&self, menu: PopupMenu, _window: &Window, _cx: &App) -> PopupMenu {
        menu  // Add custom menu items if needed
    }

    fn dump(&self, _cx: &App) -> PanelState {
        PanelState::new(self)
    }
}

impl Render for FileExplorer {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .size_full()
            .p_4()
            .child("File Explorer Content")
    }
}

impl gpui::EventEmitter<PanelEvent> for FileExplorer {}
```

## Dock Control Features

### Toggle Docks

```rust
// Toggle a dock open/closed
dock_area.update(cx, |dock, cx| {
    dock.toggle_dock(DockPlacement::Left, window, cx);
});

// Check if dock is open
let is_open = dock_area.read(cx).is_dock_open(DockPlacement::Left, cx);
```

### Lock Layout

Prevent panel dragging/splitting (resize still allowed):

```rust
dock_area.update(cx, |dock, cx| {
    dock.set_locked(true, window, cx);
});

let is_locked = dock_area.read(cx).is_locked();
```

### Collapsible Docks

Configure which docks can be collapsed:

```rust
use gpui::Edges;

dock_area.update(cx, |dock, cx| {
    dock.set_dock_collapsible(
        Edges {
            left: true,
            bottom: true,
            right: true,
            ..Default::default()
        },
        window,
        cx,
    );
});
```

### Add Panels Dynamically

```rust
dock_area.update(cx, |dock, cx| {
    dock.add_panel(
        Arc::new(new_panel),
        DockPlacement::Center,
        None,  // Optional bounds
        window,
        cx,
    );
});
```

## Layout Persistence

### Save Layout

```rust
use gpui_component::dock::DockAreaState;

fn save_layout(dock_area: &Entity<DockArea>, cx: &App) -> DockAreaState {
    dock_area.read(cx).dump(cx)
}

// Serialize to JSON
let state = save_layout(&dock_area, cx);
let json = serde_json::to_string_pretty(&state).unwrap();
std::fs::write("layout.json", json).unwrap();
```

### Load Layout

```rust
use gpui_component::dock::DockAreaState;

fn load_layout(
    dock_area: Entity<DockArea>,
    window: &mut Window,
    cx: &mut Context<MyApp>,
) -> anyhow::Result<()> {
    let json = std::fs::read_to_string("layout.json")?;
    let state: DockAreaState = serde_json::from_str(&json)?;

    dock_area.update(cx, |dock, cx| {
        dock.load(state, window, cx)
    })
}
```

### DockAreaState Structure

```rust
pub struct DockAreaState {
    pub version: Option<usize>,
    pub center: PanelState,
    pub left_dock: Option<DockState>,
    pub right_dock: Option<DockState>,
    pub bottom_dock: Option<DockState>,
}

pub struct DockState {
    panel: PanelState,
    placement: DockPlacement,
    size: Pixels,
    open: bool,
}
```

## Panel Registry

Register panels for deserialization:

```rust
use gpui_component::dock::PanelRegistry;

// In your app initialization
fn register_panels(cx: &mut App) {
    PanelRegistry::register::<FileExplorer>(cx);
    PanelRegistry::register::<TerminalPanel>(cx);
    PanelRegistry::register::<EditorPanel>(cx);
    // Register all your panel types
}
```

## Listening to Events

Subscribe to dock events:

```rust
use gpui_component::dock::DockEvent;

cx.subscribe(&dock_area, |this, dock_area, event, window, cx| {
    match event {
        DockEvent::LayoutChanged => {
            // Layout changed, maybe save it
            this.save_layout(&dock_area, window, cx);
        }
        DockEvent::DragDrop(drag) => {
            // Handle drag-drop events
        }
    }
}).detach();
```

## Best Practices

1. **Version your layouts**: Use the `version` parameter to handle layout schema changes
2. **Register all panels**: Call `PanelRegistry::register` for each Panel type before loading layouts
3. **Debounce saves**: DockEvent::LayoutChanged fires frequently; debounce save operations
4. **Use weak references**: Get `WeakEntity<DockArea>` via `.downgrade()` when creating nested items
5. **Handle load failures gracefully**: Fall back to a default layout if loading fails

## Common Patterns

### IDE-Style Layout

```rust
// Left: File tree + Search
// Center: Editors in tabs
// Bottom: Terminal + Output
// Right: Properties panel (optional)

let center = DockItem::h_split(
    vec![
        DockItem::tabs(editor_panels, &weak_dock, window, cx),
        DockItem::tabs(preview_panels, &weak_dock, window, cx).size(px(400.0)),
    ],
    &weak_dock,
    window,
    cx,
);
```

### Dashboard Layout

```rust
// Use Tiles for dashboard-style grid layouts
let tiles = DockItem::tiles(
    vec![widget1, widget2, widget3, widget4],
    vec![
        TileMeta { bounds: Bounds { origin: point(px(0.), px(0.)), size: size(px(200.), px(200.)) }, z_index: 0 },
        // ... more tile positions
    ],
    &weak_dock,
    window,
    cx,
);
```

## Troubleshooting

### Panels Not Loading
- Ensure all panel types are registered with `PanelRegistry::register`
- Check layout version matches your code
- Verify `panel_id()` returns consistent values

### Resize Not Working
- Minimum panel size is enforced (PANEL_MIN_SIZE = 32px)
- Check if layout is locked with `is_locked()`

### Events Not Firing
- Ensure you call `.detach()` on subscriptions
- Check that panels emit `PanelEvent::LayoutChanged` when modified
