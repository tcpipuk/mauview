# mauview Documentation

mauview is a Terminal User Interface (TUI) library for Go that provides a component-based
architecture for building interactive terminal applications. It's designed to be the "React" of
terminal UIs - composable, event-driven, and focused on component reusability.

## What is mauview?

mauview sits between your application logic and the terminal, handling:

- **Screen management**: Drawing components to terminal cells
- **Event routing**: Keyboard, mouse, and paste events to appropriate components
- **Layout calculation**: Automatic sizing and positioning of UI elements
- **Focus management**: Tab navigation and keyboard accessibility
- **Component lifecycle**: Creation, updates, and cleanup

## Documentation Structure

- **[API.md](./API.md)** - Complete API reference documentation
- **[Components.md](./Components.md)** - Detailed component guide
- **[Examples.md](./Examples.md)** - Code examples and tutorials

## Quick Links

### Getting Started

- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Architecture Overview](#architecture-overview)

### Core Concepts

- [Component System](./API.md#core-interfaces)
- [Event Handling](./API.md#event-system)
- [Layout Management](./API.md#layout-components)
- [Styling](./API.md#styling-system)

### Component Reference

- [Input Components](./API.md#input-components)
- [Display Components](./API.md#display-components)
- [Layout Components](./API.md#layout-components)

## Installation

```bash
go get github.com/tulir/mauview
```

## Basic Usage

```go
package main

import "github.com/tulir/mauview"

func main() {
    app := mauview.NewApplication()

    root := mauview.NewBox(
        mauview.NewTextView().SetText("Hello, mauview!"),
    ).SetBorder(true).SetTitle("My App")

    app.SetRoot(root)
    app.Start()
}
```

## Architecture Philosophy

mauview follows a **declarative component model** where you describe what your UI should look like,
and the library handles how to make it happen. Components are:

- **Composable**: Larger components built from smaller ones
- **Stateful**: Each component manages its own internal state
- **Event-driven**: Components respond to user input through event handlers
- **Drawable**: Each component knows how to render itself to screen

### Core Responsibilities

**Application**: The root coordinator that manages the terminal screen, input events, component tree,
focus chain, and main event loop.

**Components**: Everything is a `Component` interface that defines how to render (`Draw`) and
respond to layout changes (`SetRect`/`GetRect`).

**Event System**: Components implement optional handler interfaces (`KeyHandler`, `MouseHandler`,
etc.) to process specific event types.

**Layout Managers**: Specialized components that automatically calculate child positions and sizes.

## Key Features

- **Component-Based**: Build complex UIs from simple, reusable components
- **Event-Driven**: Full keyboard, mouse, and paste event support
- **Flexible Layouts**: Multiple layout managers (Flex, Grid, Center)
- **Rich Text**: Color tags, text wrapping, and Unicode support
- **Input Handling**: Advanced text input with selection, clipboard, and undo/redo
- **Thread-Safe Updates**: Safe component updates from goroutines
- **Extensible**: Easy to create custom components

## Component Categories

### Layout Components

- **Flex**: Linear layouts (rows/columns) with fixed or proportional sizing
- **Grid**: 2D layouts with cell-based positioning
- **Center**: Centers content within available space

*These handle the "where" - calculating positions and sizes for child components.*

### Container Components

- **Box**: Adds borders, titles, and event capture to wrapped components

*These handle "presentation" - visual styling and event isolation.*

### Input Components

- **InputField**: Single-line text input with editing capabilities
- **InputArea**: Multi-line text editor with selection and clipboard
- **Form**: Manages multiple inputs with tab navigation

*These handle "user interaction" - accepting and processing user input.*

### Display Components

- **TextView**: Scrollable text with color tags and formatting
- **TextField**: Simple text display for labels and status
- **Button**: Interactive clickable element
- **ProgressBar**: Visual progress indication

*These handle "information display" - showing data to the user.*

## Event Flow

1. User input (keyboard/mouse) → tcell → Application
2. Application dispatches event to root component
3. Event propagates through component tree
4. Components handle or pass through events
5. First handler to return `true` consumes the event

## When to Use mauview

**Perfect for:**

- Chat clients, file managers, system monitors
- Development tools (debuggers, log viewers, REPLs)
- Applications requiring rich text input/display
- Terminal apps with complex layouts

**Consider alternatives for:**

- Simple command-line tools (use flags/prompts instead)
- Applications requiring graphical elements (charts, images)
- Web-based interfaces (use actual web frameworks)

## Key Architectural Decisions

**Component-First Design**: Everything is a component, making the system highly composable and testable.

**Event Bubbling**: Events flow up the component tree until handled, similar to DOM events.

**Automatic Layout**: Layout components handle the math so you focus on structure, not coordinates.

**Focus Chain Management**: Built-in keyboard navigation between focusable components.

**Thread-Safe Updates**: Components can be safely updated from goroutines using the application's
event queue.

## Common Patterns

### Modal Dialogs

```go
dialog := mauview.NewFractionalCenter(0.5, 0.3).
    SetComponent(
        mauview.NewBox(content).
            SetBorder(true).
            SetTitle("Dialog"),
    )
```

### Status Bar

```go
statusBar := mauview.NewFlex().
    AddProportionalComponent(leftStatus, 1).
    AddFixedComponent(centerStatus, 20).
    AddProportionalComponent(rightStatus, 1)
```

### Scrollable List

```go
textView := mauview.NewTextView().
    SetScrollable(true).
    SetDynamicColors(true)
```

## Understanding the Component Lifecycle

1. **Creation**: Component struct is initialized with default state
2. **Layout**: Parent calls `SetRect()` to assign position and size
3. **Drawing**: `Draw()` method renders component to screen buffer
4. **Events**: Handler methods process user input
5. **Updates**: State changes trigger redraws via `RedrawSoon()`
6. **Cleanup**: Components can implement cleanup when removed

## Data Flow Patterns

**Top-Down Data**: Parent components pass data to children via method calls
**Bottom-Up Events**: Child components notify parents via callback functions
**Side Effects**: Use goroutines to handle async operations, then update UI on main thread

For complete API documentation, see [API.md](./API.md).
