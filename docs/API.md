# mauview API Documentation

This document explains the core architecture, interfaces, and component responsibilities in mauview.
Rather than exhaustive code examples, this focuses on understanding what each part does and how
components work together.

## Table of Contents

1. [Core Architecture](#core-architecture)
2. [Component Interface](#component-interface)
3. [Event System](#event-system)
4. [Layout System](#layout-system)
5. [Component Responsibilities](#component-responsibilities)
6. [Application Lifecycle](#application-lifecycle)
7. [Custom Component Development](#custom-component-development)
8. [Integration Patterns](#integration-patterns)

## Core Architecture

mauview's architecture centers around a tree of components, with each component responsible for:

1. **Rendering itself** when asked to draw
2. **Managing its own state** (text content, focus, etc.)
3. **Handling user events** it cares about
4. **Laying out child components** (for container components)

### The Component Tree

Every mauview application has a component tree starting with a root component. Events flow down from
the application to components, and drawing happens by walking the tree and asking each component to
render itself.

```plain
Application
└── Root Component (e.g., Flex layout)
    ├── Header (e.g., Box with title)
    │   └── Status Text (e.g., TextField)
    ├── Main Content (e.g., another Flex)
    │   ├── Sidebar (e.g., TextView)
    │   └── Content Area (e.g., InputArea)
    └── Footer (e.g., Box with buttons)
        ├── OK Button
        └── Cancel Button
```

## Component Interface

The `Component` interface defines the contract that all UI elements must implement:

```go
type Component interface {
    Draw(screen Screen)          // Render yourself to the screen
    GetRect() (x, y, w, h int)   // Where are you positioned?
    SetRect(x, y, w, h int)      // Move/resize yourself
    GetFocusable() bool          // Can you receive keyboard focus?
    OnPasteEvent(*PasteEvent) bool // Handle clipboard paste
}
```

**Key Insight**: Components don't know about their parents or children through this interface. The
tree structure is managed by container components that hold references to their children.

### Optional Event Interfaces

Components implement additional interfaces to handle specific types of events:

#### KeyHandler

Components that want to process keyboard input implement `OnKeyEvent(KeyEvent) bool`. Return `true`
to consume the event and stop it from propagating to other components.

**Example responsibilities**: InputField processes text input, Button responds to Enter/Space, Form
handles Tab navigation.

#### MouseHandler

Components that respond to mouse interactions implement `OnMouseEvent(MouseEvent) bool`. This
includes clicks, drags, and scroll wheel events.

**Example responsibilities**: Button detects clicks, TextView handles scroll wheel, InputArea
processes text selection.

#### FocusHandler

Components that can receive keyboard focus implement `Focus()`, `Blur()`, and `HasFocus() bool`.
The application manages which component has focus and routes keyboard events accordingly.

**Example responsibilities**: Input components highlight when focused, containers track which child
has focus.

## Event System

### Event Flow Architecture

Events in mauview follow a bubbling pattern similar to DOM events:

1. **Application captures** keyboard/mouse events from the terminal
2. **Application determines target** (usually the focused component for keyboard, position-based for
   mouse)
3. **Event bubbles up** through parent components until one handles it
4. **First handler returning `true`** consumes the event and stops propagation

### Event Types

**KeyEvent**: Represents keyboard input (keys, modifiers, special keys like arrows/function keys)

- Handled by components implementing `KeyHandler`
- Common use: Text input, navigation, shortcuts

**MouseEvent**: Represents mouse interactions (clicks, movement, scroll wheel)

- Handled by components implementing `MouseHandler`
- Common use: Button clicks, text selection, scrolling

**PasteEvent**: Represents clipboard paste operations

- Handled via `OnPasteEvent()` on the Component interface
- Common use: Pasting text into input fields

### Focus Management

The application maintains a **focus chain** - the sequence of focusable components that can receive
keyboard input.

**Focus Chain Rules**:

- Only one component has focus at a time
- Tab/Shift+Tab moves through focusable components
- Keyboard events go to the focused component first
- Non-focusable components can't receive keyboard events

## Layout System

Layout components are responsible for positioning and sizing their child components. They implement
the core Component interface but also manage a collection of children.

### Layout Component Responsibilities

1. **Store child components** and their layout parameters
2. **Calculate positions and sizes** when `SetRect()` is called
3. **Call `SetRect()` on children** to inform them of their new bounds
4. **Draw children in order** during the `Draw()` call

### Flex Layout

**Purpose**: Arranges children in a row or column with flexible sizing.

**Key Concepts**:

- **Direction**: FlexRow (horizontal) or FlexColumn (vertical)
- **Fixed components**: Have a specific size (e.g., sidebar width, header height)
- **Proportional components**: Share remaining space based on proportion values

**Layout Algorithm**:

1. Calculate total fixed space needed
2. Distribute remaining space proportionally
3. Position each child based on cumulative sizes

### Grid Layout

**Purpose**: Arranges children in a 2D grid with specific cell positions.

**Key Concepts**:

- **Grid dimensions**: Number of rows and columns
- **Cell spanning**: Components can span multiple rows/columns
- **Position-based**: Each child has explicit row/column coordinates

**Layout Algorithm**:

1. Calculate cell sizes based on grid dimensions
2. Position each child at its grid coordinates
3. Handle spanning by extending across multiple cells

### Center Layout

**Purpose**: Centers a single child component within available space.

**Key Concepts**:

- **Fixed centering**: Center component uses fixed dimensions
- **Fractional centering**: Center component uses percentage of available space

**Layout Algorithm**:

1. Calculate child size (fixed or percentage-based)
2. Calculate position to center within available space
3. Position child and fill remaining space with background

## Component Responsibilities

### Application

**Role**: Central coordinator and event loop manager

**Responsibilities**:

- Initialize and manage the terminal screen (via tcell)
- Capture keyboard, mouse, and paste events from the terminal
- Maintain the component tree and focus chain
- Route events to appropriate components
- Trigger and coordinate drawing cycles
- Handle terminal resize events
- Provide thread-safe component update mechanisms

**Key Methods**: `SetRoot()`, `Start()`, `Stop()`, `SetFocus()`, `Redraw()`

### Layout Components (Flex, Grid, Center)

**Role**: Organize and position child components

**Responsibilities**:

- Store references to child components
- Calculate optimal positions and sizes for children
- Handle layout recalculation when container is resized
- Draw children in proper order (usually front-to-back)
- Forward events to appropriate children based on position

**Key Insight**: Layout components rarely draw anything themselves - they focus on positioning their
children.

### Container Components (Box)

**Role**: Add visual styling and behavior to wrapped components

**Responsibilities**:

- Draw decorative elements (borders, titles, backgrounds)
- Handle event capture and filtering
- Provide focus indication
- Manage padding/margins around child content

**Key Pattern**: Box wraps exactly one child component and enhances it.

### Input Components (InputField, InputArea, Form)

**Role**: Accept and process user input

**Responsibilities**:

- Maintain internal text/state based on user input
- Implement text editing operations (insert, delete, selection)
- Provide keyboard shortcuts (Ctrl+A, Ctrl+V, etc.)
- Validate input and call callbacks on changes
- Handle focus indication and cursor display

**Key Insight**: Input components are stateful - they maintain the current input value and editing state.

### Display Components (TextView, TextField, Button, ProgressBar)

**Role**: Present information to the user

**Responsibilities**:

- Render text, graphics, or interactive elements
- Handle formatting (colors, alignment, word wrapping)
- Provide scrolling for content larger than available space
- Update display when underlying data changes

**Key Pattern**: Display components are often driven by external data and re-render when that data changes.

## Application Lifecycle

The `Application` struct is the central coordinator that manages the entire UI lifecycle:

### Initialization Phase

1. **Screen setup**: Initialize tcell screen and terminal capabilities
2. **Component tree**: Accept a root component via `SetRoot()`
3. **Focus chain**: Build initial list of focusable components

### Runtime Phase

1. **Event capture**: Listen for keyboard, mouse, and terminal events
2. **Event routing**: Determine which component should handle each event
3. **State updates**: Components update their internal state
4. **Redraw cycles**: Trigger screen updates when components change
5. **Focus management**: Handle Tab navigation and focus changes

### Shutdown Phase

1. **Event loop termination**: Stop processing events
2. **Screen cleanup**: Restore terminal to original state
3. **Resource cleanup**: Allow components to clean up resources

### Key Application Methods

**SetRoot(Component)**: Defines the top-level component of your UI tree. This component and its
children define your entire interface.

**Start()**: Begins the event loop and takes control of the terminal. This is a blocking call that
runs until the application shuts down.

**Stop()**: Shuts down gracefully and restores terminal state. Usually called from an event handler.

**SetFocus(Component)**: Changes which component receives keyboard input. The component must
implement FocusHandler.

**Redraw()/RedrawSoon()**: Triggers screen updates. `Redraw()` updates immediately, `RedrawSoon()`
batches updates for the next frame.

### Thread Safety

mauview provides thread-safe component updates through the application's event queue:

- **Safe**: Call `RedrawSoon()` from any goroutine to trigger updates
- **Safe**: Update component state from goroutines, then call `RedrawSoon()`
- **Unsafe**: Directly manipulating the component tree from multiple goroutines
- **Unsafe**: Drawing to the screen outside the main event loop

## Custom Component Development

### Basic Component Pattern

To create a custom component:

1. **Define a struct** that will hold your component's state
2. **Implement the Component interface** (`Draw`, `GetRect`, `SetRect`, etc.)
3. **Implement optional interfaces** (`KeyHandler`, `MouseHandler`, `FocusHandler`) as needed
4. **Manage internal state** in response to events and external updates

### Component State Management

**Internal State**: Components should manage their own display state (text content, selection,
scroll position, etc.)

**External State**: Accept data from parent components via method calls, not through the Component interface

**State Updates**: When internal state changes, call `RedrawSoon()` to trigger a visual update

### Event Handling Best Practices

**Event Consumption**: Return `true` from event handlers only when you actually handle the event.
This prevents it from bubbling to parent components.

**Event Routing**: For container components, determine which child should receive an event before
forwarding it.

**Focus Interaction**: If your component can receive focus, implement `FocusHandler` and provide
visual focus indication.

### Drawing Best Practices

**Clipping**: Only draw within your assigned rectangle (from `GetRect()`)

**Efficiency**: Avoid expensive operations in `Draw()` - pre-compute what you can

**Layering**: Draw background elements first, foreground elements last

**Text Handling**: Use mauview's text utilities for proper Unicode and color handling

## Integration Patterns

### Data Binding Pattern

For components that display external data:

1. **Accept data via method calls** (e.g., `SetText()`, `SetItems()`)
2. **Store data in component state**
3. **Call `RedrawSoon()`** when data changes
4. **Re-render in `Draw()`** method using current data

### Event Callback Pattern

For components that need to notify parent components:

1. **Accept callback functions** during component creation
2. **Call callbacks** when significant events occur (user actions, state changes)
3. **Let parent components** decide how to respond to events

### Composite Component Pattern

For building complex components from simpler ones:

1. **Use layout components** (Flex, Grid) as the foundation
2. **Compose child components** for different areas of functionality
3. **Coordinate state** between child components
4. **Provide a unified interface** to parent components

### Modal/Dialog Pattern

For overlay components:

1. **Use Center layout** to position dialog in screen center
2. **Implement event capture** to prevent interaction with background
3. **Provide escape mechanisms** (ESC key, outside clicks)
4. **Manage focus** appropriately (focus dialog, restore previous focus on close)

## Complete API Reference

### Application Methods

**NewApplication() *Application**

- Creates new application instance
- Initializes tcell screen
- Sets up event queue
- Returns ready-to-use application

**SetRoot(component Component) *Application**

- Sets the top-level component
- Component fills entire terminal
- Replaces any existing root
- Triggers initial layout calculation

**Start() error**

- Takes control of terminal
- Starts event loop (blocking)
- Processes keyboard/mouse events
- Returns when Stop() is called

**Stop()**

- Gracefully shuts down application
- Restores terminal state
- Stops event processing
- Safe to call from any goroutine

**SetFocus(component Component) *Application**

- Changes keyboard focus
- Component must implement FocusHandler
- Previous focus is lost
- Returns self for chaining

**GetFocus() Component**

- Returns currently focused component
- Returns nil if no component has focus
- Used for focus management
- Read-only operation

**Redraw() *Application**

- Forces immediate screen update
- Expensive operation
- Use RedrawSoon() instead when possible
- Blocks until drawing complete

**RedrawSoon() *Application**

- Schedules redraw for next frame
- Batches multiple calls efficiently
- Thread-safe
- Preferred for frequent updates

**GetScreen() tcell.Screen**

- Returns underlying tcell screen
- For advanced terminal operations
- Direct screen access bypasses component system
- Use with caution

### Core Component Interface

```go
type Component interface {
    Draw(screen Screen)
    GetRect() (x, y, width, height int)
    SetRect(x, y, width, height int)
    GetFocusable() bool
    OnPasteEvent(event *PasteEvent) bool
}
```

### Optional Component Interfaces

#### KeyHandler

```go
type KeyHandler interface {
    OnKeyEvent(event KeyEvent) bool
}
```

- Return true to consume the event
- Event stops propagating if consumed
- Use for keyboard shortcuts and input

#### MouseHandler

```go
type MouseHandler interface {
    OnMouseEvent(event MouseEvent) bool
}
```

- Handles clicks, drags, scroll wheel
- Position is relative to component bounds
- Return true to consume event

#### FocusHandler

```go
type FocusHandler interface {
    Focus()
    Blur()
    HasFocus() bool
}
```

- Focus() called when component gains focus
- Blur() called when component loses focus
- HasFocus() returns current focus state

### Layout Components

#### Flex

**NewFlex() *Flex**

- Creates flexible layout container
- Default direction is FlexColumn (vertical)
- No children initially
- Returns ready-to-use flex container

**SetDirection(direction FlexDirection) *Flex**

- FlexRow: horizontal layout
- FlexColumn: vertical layout
- Changes how children are arranged
- Triggers layout recalculation

**AddFixedComponent(component Component, size int) *Flex**

- Adds child with fixed size
- Size is width (row) or height (column)
- Component gets exactly this much space
- Added to end of children list

**AddProportionalComponent(component Component, proportion int) *Flex**

- Adds child with proportional size
- Gets proportion/total proportions of remaining space
- Higher numbers get more space
- Proportion of 0 means minimal space

**RemoveComponent(component Component) *Flex**

- Removes child from layout
- Component stops receiving events
- Layout recalculates automatically
- No-op if component not found

**Clear() *Flex**

- Removes all children
- Resets to empty container
- Useful for dynamic layouts
- Layout becomes minimal size

#### Grid

**NewGrid() *Grid**

- Creates 2D grid layout
- No initial size or children
- Must set dimensions before use
- Returns ready-to-configure grid

**SetSize(rows, columns int) *Grid**

- Defines grid dimensions
- Existing components may be repositioned
- Cannot shrink below largest occupied cell
- Triggers complete layout recalculation

**SetComponent(row, col int, component Component) *Grid**

- Places component at grid position
- Replaces any existing component at position
- Position must be within grid bounds
- Component spans single cell

**SetComponentSpan(row, col, rowSpan, colSpan int, component Component) *Grid**

- Places component spanning multiple cells
- rowSpan/colSpan must be positive
- Span cannot exceed grid boundaries
- Replaces any components in spanned area

**RemoveComponent(component Component) *Grid**

- Removes component from grid
- Leaves grid cell empty
- No layout shift occurs
- Safe to call during iteration

#### Center

**NewCenter(component Component) *Center**

- Centers component in available space
- Component uses its preferred size
- Remaining space is filled with background
- Returns configured center container

**NewFractionalCenter(widthFraction, heightFraction float64) *Center**

- Centers component using fraction of available space
- Fractions should be between 0.0 and 1.0
- 0.5 means half the available space
- Good for modal dialogs

**SetComponent(component Component) *Center**

- Changes the centered component
- Replaces existing component
- Layout recalculates automatically
- Component can be nil to clear

### Container Components

#### Box

**NewBox(component Component) *Box**

- Wraps component with optional border/title
- Component can be nil initially
- No border by default
- Returns configured box

**SetBorder(show bool) *Box**

- Shows or hides border around component
- Border reduces available space for content
- Uses box drawing characters
- Updates automatically

**SetTitle(title string) *Box**

- Sets title text in top border
- Only visible if border is enabled
- Title is truncated if too long
- Empty string removes title

**SetTitleAlign(align Align) *Box**

- AlignLeft: title on left side
- AlignCenter: title in center
- AlignRight: title on right side
- Only affects title position

**SetComponent(component Component) *Box**

- Changes wrapped component
- Component can be nil
- Layout adjusts to new component
- Previous component loses focus

### Input Components

#### InputField

**NewInputField() *InputField**

- Creates single-line text input
- Empty initially
- Cursor at beginning
- Returns ready-to-use input

**SetText(text string) *InputField**

- Sets current input text
- Replaces any existing text
- Cursor moves to end
- Triggers change callback if set

**GetText() string**

- Returns current input text
- Always returns complete string
- No formatting or processing
- Safe to call anytime

**SetPlaceholder(text string) *InputField**

- Sets placeholder text shown when empty
- Only visible when input is empty
- Uses dimmed styling
- Automatically hidden when typing

**SetMaskCharacter(char rune) *InputField**

- Masks input with specified character
- Useful for password fields
- Set to 0 to disable masking
- Display only, actual text unchanged

**SetChangedFunc(handler func(text string)) *InputField**

- Called when text changes
- Includes programmatic changes
- Handler receives current text
- Can be nil to disable

**SetDoneFunc(handler func(key tcell.Key)) *InputField**

- Called when user finishes input
- Triggered by Enter, Tab, or Escape
- Handler receives the key pressed
- Used for form navigation

#### InputArea

**NewInputArea() *InputArea**

- Creates multi-line text editor
- Supports text selection
- Built-in scrolling
- Returns ready-to-use editor

**SetText(text string) *InputArea**

- Sets editor content
- Replaces existing text
- Cursor moves to end
- Scroll position resets

**GetText() string**

- Returns complete editor content
- Includes all line breaks
- No processing or formatting
- Efficient for large text

**SetPlaceholder(text string) *InputArea**

- Shown when editor is empty
- Can be multi-line
- Uses dimmed styling
- Disappears when typing starts

**SetWordWrap(wrap bool) *InputArea**

- Enables soft line wrapping
- Long lines wrap at word boundaries
- Does not insert actual line breaks
- Improves readability

**SetChangedFunc(handler func()) *InputArea**

- Called when content changes
- No parameters (use GetText())
- Includes cursor movement
- High frequency callback

#### Form

**NewForm() *Form**

- Creates form with multiple inputs
- Handles tab navigation
- No items initially
- Returns configured form

**AddInputField(label, text, placeholder string, maskChar rune, changedFunc func(string)) *Form**

- Adds labeled input field
- Label appears to the left
- Returns form for chaining
- Field becomes focusable

**AddTextArea(label, text, placeholder string) *Form**

- Adds labeled text area
- Suitable for longer input
- Returns form for chaining
- Area becomes focusable

**AddButton(label string, selected func()) *Form**

- Adds clickable button
- Callback triggered on activation
- Returns form for chaining
- Button becomes focusable

**SetHorizontal(horizontal bool) *Form**

- true: items arranged in row
- false: items arranged in column
- Changes entire form layout
- Default is vertical

### Display Components

#### TextView

**NewTextView() *TextView**

- Creates scrollable text display
- No content initially
- Not editable
- Returns ready-to-use view

**SetText(text string) *TextView**

- Sets display content
- Supports color tags
- Replaces existing content
- Scroll position resets

**SetTextAlign(align Align) *TextView**

- AlignLeft: left-aligned text
- AlignCenter: center-aligned text
- AlignRight: right-aligned text
- Affects all lines

**SetDynamicColors(dynamic bool) *TextView**

- Enables [color] tag processing
- Format: [foreground:background]
- Example: [red:blue]colored text[white:-]
- Disable for literal brackets

**SetScrollable(scrollable bool) *TextView**

- Enables vertical scrolling
- Arrow keys scroll when focused
- Mouse wheel also works
- Required for large content

**SetWordWrap(wrap bool) *TextView**

- Enables line wrapping
- Long lines break at word boundaries
- Preserves formatting
- Improves readability

**ScrollToEnd() *TextView**

- Scrolls to show last line
- Useful for logs and chat
- Only works if scrollable
- Updates immediately

#### TextField

**NewTextField() *TextField**

- Creates simple text display
- Single line only
- Not scrollable
- Returns ready-to-use field

**SetText(text string) *TextField**

- Sets displayed text
- Truncated if too long
- No line breaks supported
- Updates immediately

**SetTextAlign(align Align) *TextField**

- Controls text alignment
- Same options as TextView
- Affects single line
- Default is left-aligned

#### Button

**NewButton(label string) *Button**

- Creates clickable button
- Label is button text
- No action initially
- Returns configured button

**SetSelectedFunc(handler func()) *Button**

- Sets click handler
- Called on Enter or Space
- Called on mouse click
- Can be nil to disable

**SetLabel(label string) *Button**

- Changes button text
- Updates display immediately
- Can be empty string
- Triggers redraw

#### ProgressBar

**NewProgressBar() *ProgressBar**

- Creates progress indicator
- 0% progress initially
- Uses block characters
- Returns ready-to-use bar

**SetProgress(progress float64) *ProgressBar**

- Sets completion percentage
- Range: 0.0 to 1.0
- Values outside range are clamped
- Updates display immediately

**SetMax(max int) *ProgressBar**

- Sets maximum value for integer progress
- Use with SetProgress(current/max)
- Useful for file transfers
- Must be positive

### Event Types

#### KeyEvent

```go
type KeyEvent interface {
    Key() tcell.Key      // Special keys (Enter, Tab, etc.)
    Rune() rune          // Character keys
    Modifiers() tcell.ModMask  // Ctrl, Alt, Shift
}
```

#### MouseEvent

```go
type MouseEvent interface {
    Position() (x, y int)    // Click position
    Button() tcell.ButtonMask // Which button
    Modifiers() tcell.ModMask // Key modifiers
}
```

#### PasteEvent

```go
type PasteEvent struct {
    Text string    // Pasted content
}
```

### Color and Styling

#### Color Tags

- `[red]text[white]`: Colored text
- `[#ff0000]text[-]`: Hex colors
- `[red:blue]text[-:-]`: Foreground and background
- `[-]` or `[white]`: Reset to default

#### Common Colors

- Basic: black, red, green, yellow, blue, magenta, cyan, white
- Bright: brightred, brightgreen, etc.
- Hex: #rrggbb format
- System: ColorNames from tcell

### Best Practices

#### Performance

- Use RedrawSoon() instead of Redraw() for frequent updates
- Avoid expensive operations in Draw() methods
- Batch state changes before triggering redraws
- Use proportional layouts to avoid fixed sizing

#### Event Handling

- Return true from event handlers only when consuming the event
- Handle errors gracefully in user callbacks
- Don't perform long operations in event handlers
- Use goroutines for background work, then RedrawSoon()

#### Focus Management

- Implement FocusHandler for interactive components
- Provide visual focus indicators
- Test tab navigation thoroughly
- Consider accessibility requirements

#### Layout Design

- Use Flex for most layouts
- Grid for complex 2D arrangements
- Center for modals and dialogs
- Box for visual grouping and borders

#### Component Development

- Keep Draw() methods efficient
- Store state in component structs
- Use callbacks for parent communication
- Handle edge cases gracefully
