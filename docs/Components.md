# mauview Component Guide

This guide provides detailed information about each component in mauview, including usage patterns,
properties, and examples.

## Component Categories

1. [Layout Components](#layout-components) - Organize other components
2. [Input Components](#input-components) - Accept user input
3. [Display Components](#display-components) - Show information
4. [Container Components](#container-components) - Wrap and enhance other components

## Layout Components

### Flex

The Flex component arranges child components in rows or columns with flexible sizing.

#### Properties

- **Direction**: `FlexRow` (horizontal) or `FlexColumn` (vertical)
- **Children**: Components with fixed or proportional sizes

#### Methods

```go
SetDirection(direction Direction) *Flex
AddFixedComponent(component Component, size int) *Flex
AddProportionalComponent(component Component, proportion int) *Flex
RemoveComponent(component Component) *Flex
Clear() *Flex
GetComponents() []Component
```

#### Example: Two-Panel Layout

```go
// Create a horizontal split with sidebar and main content
layout := mauview.NewFlex().
    SetDirection(mauview.FlexRow).
    AddFixedComponent(sidebar, 30).      // 30 columns wide
    AddProportionalComponent(content, 1)  // Remaining space

// Create a vertical split with header, body, and footer
page := mauview.NewFlex().
    SetDirection(mauview.FlexColumn).
    AddFixedComponent(header, 3).         // 3 rows high
    AddProportionalComponent(body, 1).    // Flexible height
    AddFixedComponent(footer, 1)          // 1 row high
```

### Grid

The Grid component arranges children in a 2D grid with configurable rows and columns.

#### Properties

- **Rows/Columns**: Grid dimensions
- **Items**: Components with position and span

#### Methods

```go
SetRows(rows int) *Grid
SetColumns(columns int) *Grid
AddComponent(component Component, row, column, rowSpan, colSpan int) *Grid
RemoveComponent(component Component) *Grid
Clear() *Grid
```

#### Example: Dashboard Layout

```go
// Create a 3x3 grid dashboard
dashboard := mauview.NewGrid().
    SetRows(3).
    SetColumns(3).
    AddComponent(stats, 0, 0, 1, 2).     // Top-left, spans 2 columns
    AddComponent(alerts, 0, 2, 2, 1).    // Top-right, spans 2 rows
    AddComponent(chart, 1, 0, 2, 2).     // Bottom-left, spans 2x2
    AddComponent(controls, 2, 2, 1, 1)   // Bottom-right corner
```

### Center

Centers a component within available space using fixed dimensions.

#### Properties

- **Width/Height**: Fixed dimensions for centered content
- **Component**: The component to center

#### Example: Centered Dialog

```go
dialog := mauview.NewCenter(60, 20).  // 60x20 centered box
    SetComponent(
        mauview.NewBox(dialogContent).
            SetBorder(true).
            SetTitle("Confirmation"),
    )
```

### FractionalCenter

Centers a component using percentage-based dimensions.

#### Properties

- **Width/Height Fraction**: Percentage of available space (0.0-1.0)
- **Component**: The component to center

#### Example: Responsive Modal

```go
modal := mauview.NewFractionalCenter(0.8, 0.6).  // 80% width, 60% height
    SetComponent(
        mauview.NewBox(modalContent).
            SetBorder(true),
    )
```

## Input Components

### InputField

Single-line text input with advanced editing capabilities.

#### Features

- Text editing with cursor movement
- Selection support (Shift+arrows)
- Password masking
- Tab completion
- Vim bindings (optional)
- Placeholder text
- Change callbacks

#### Key Bindings

- **Arrow keys**: Move cursor
- **Ctrl+A/E**: Beginning/end of line
- **Ctrl+W**: Delete word backward
- **Ctrl+K**: Delete to end of line
- **Ctrl+U**: Delete to beginning of line
- **Tab**: Trigger completion
- **Enter**: Submit

#### Example: Search Box

```go
searchBox := mauview.NewInputField().
    SetPlaceholder("Search...").
    SetChangedFunc(func(text string) {
        // Live search as user types
        performSearch(text)
    }).
    SetSubmitFunc(func(text string) {
        // Execute search on Enter
        executeSearch(text)
    })
```

#### Example: Command Input with Completion

```go
cmdInput := mauview.NewInputField().
    SetPlaceholder("Enter command...").
    SetTabCompleteFunc(func(text string, cursor int) string {
        // Return completed command
        return completeCommand(text, cursor)
    }).
    SetVimBindings(true)  // Enable vim-style navigation
```

### InputArea

Multi-line text editor with full editing capabilities.

#### Features

- Multi-line editing
- Text selection (mouse and keyboard)
- Undo/redo support
- Clipboard operations
- Word wrapping
- Syntax highlighting (via custom drawing)
- Tab completion

#### Key Bindings

- **Ctrl+C/X/V**: Copy/Cut/Paste
- **Ctrl+A**: Select all
- **Ctrl+Z/Y**: Undo/Redo
- **Tab**: Indent or complete
- **Shift+Tab**: Unindent
- **Alt+Arrow**: Move by word

#### Example: Message Composer

```go
composer := mauview.NewInputArea().
    SetPlaceholder("Type a message...").
    SetWordWrap(true).
    SetTabCompleteFunc(func(text string, x, y int) string {
        // Complete @mentions or :emojis:
        return completeAtCursor(text, x, y)
    })

// Get composed message
message := composer.GetText()
```

#### Example: Code Editor

```go
editor := mauview.NewInputArea().
    SetWordWrap(false).
    SetText(initialCode)

// Add syntax highlighting via custom draw
type SyntaxEditor struct {
    *mauview.InputArea
}

func (s *SyntaxEditor) Draw(screen mauview.Screen) {
    s.InputArea.Draw(screen)
    // Apply syntax highlighting over the drawn text
    highlightSyntax(screen, s.GetLines())
}
```

### Form

Container for organizing form inputs with automatic navigation.

#### Features

- Tab navigation between fields
- Enter to submit
- Automatic focus management
- Label and description support

#### Example: Login Form

```go
type LoginFormItem struct {
    mauview.SimpleEventHandler
    label       string
    description string
    input       *mauview.InputField
}

func (f *LoginFormItem) GetLabel() string { return f.label }
func (f *LoginFormItem) GetDescription() string { return f.description }
func (f *LoginFormItem) Draw(screen mauview.Screen) { f.input.Draw(screen) }

// Create form
loginForm := mauview.NewForm()

// Add username field
usernameItem := &LoginFormItem{
    label: "Username",
    description: "Enter your username",
    input: mauview.NewInputField().SetPlaceholder("username"),
}

// Add password field
passwordItem := &LoginFormItem{
    label: "Password",
    description: "Enter your password",
    input: mauview.NewInputField().
        SetPlaceholder("password").
        SetMaskCharacter('*'),
}

loginForm.
    AddFormItem(usernameItem).
    AddFormItem(passwordItem).
    SetSubmitFunc(func() {
        username := usernameItem.input.GetText()
        password := passwordItem.input.GetText()
        performLogin(username, password)
    })
```

## Display Components

### TextView

Scrollable text display with rich formatting support.

#### Features

- Scrollable content
- Color tag support `[color]text[white]`
- Region highlighting
- Word wrapping
- Text search
- Dynamic content updates

#### Color Tags

```plain
[red]Error text[white]
[green]Success text[white]
[yellow]Warning text[white]
[blue]Info text[white]
[#00ff00]Custom hex color[white]
```

#### Example: Log Viewer

```go
logView := mauview.NewTextView().
    SetScrollable(true).
    SetDynamicColors(true).
    SetWordWrap(true)

// Append log entries with colors
logView.SetText(logView.GetText() +
    "[gray]2024-01-15 10:30:45[white] " +
    "[green][INFO][white] Application started\n")

// Auto-scroll to bottom
logView.ScrollToEnd()
```

#### Example: Help Text with Regions

```go
helpText := mauview.NewTextView().
    SetRegions(true).
    SetDynamicColors(true).
    SetText(
        `[yellow]Commands:[white]
        ["help"]help[""] - Show this help
        ["quit"]quit[""] - Exit application
        ["search"]search <term>[""] - Search for items`)

// Highlight a command when selected
helpText.Highlight("search")
```

### TextField

Simple text display component for labels and status text.

#### Features

- Thread-safe text updates
- Custom styling
- Minimal overhead

#### Example: Status Bar Items

```go
// Connection status
connStatus := mauview.NewTextField().
    SetText("Connected").
    SetStyle(tcell.StyleDefault.Foreground(tcell.ColorGreen))

// Update from goroutine
go func() {
    if connectionLost {
        connStatus.SetText("Disconnected")
        connStatus.SetStyle(tcell.StyleDefault.Foreground(tcell.ColorRed))
    }
}()
```

### Button

Interactive button with keyboard and mouse support.

#### Features

- Click handling
- Keyboard activation (Enter/Space when focused)
- Custom styling for normal/focused states

#### Example: Dialog Buttons

```go
okButton := mauview.NewButton("OK").
    SetSelectedFunc(func() {
        // Handle OK action
        closeDialog(true)
    })

cancelButton := mauview.NewButton("Cancel").
    SetSelectedFunc(func() {
        // Handle Cancel action
        closeDialog(false)
    })

// Arrange in a flex
buttonBar := mauview.NewFlex().
    SetDirection(mauview.FlexRow).
    AddProportionalComponent(mauview.NewBox(nil), 1).  // Spacer
    AddFixedComponent(okButton, 10).
    AddFixedComponent(cancelButton, 10)
```

### ProgressBar

Visual progress indicator with determinate and indeterminate modes.

#### Features

- Smooth animations using block characters
- Percentage display
- Indeterminate mode for unknown progress
- Thread-safe updates

#### Example: File Upload Progress

```go
progressBar := mauview.NewProgressBar()

// Update progress from upload goroutine
go func() {
    for progress := 0.0; progress <= 1.0; progress += 0.01 {
        progressBar.SetProgress(progress)
        time.Sleep(100 * time.Millisecond)
    }
}()
```

#### Example: Loading Spinner

```go
spinner := mauview.NewProgressBar().
    SetIndeterminate(true)  // Animated loading bar

// Stop after loading
go func() {
    loadData()
    spinner.SetIndeterminate(false)
    spinner.SetProgress(1.0)  // Show complete
}()
```

## Container Components

### Box

Versatile container with borders, titles, and focus handling.

#### Features

- Optional borders with focus indication
- Title support
- Event capture control
- Content padding

#### Border Styles

- Normal: Single-line border
- Focused: Different color/style when focused
- Double: Double-line borders
- Rounded: Rounded corners

#### Example: Panel with Title

```go
panel := mauview.NewBox(content).
    SetBorder(true).
    SetTitle("Settings").
    SetTitleAlign(mauview.AlignCenter)
```

#### Example: Modal Dialog with Event Capture

```go
modal := mauview.NewBox(dialogContent).
    SetBorder(true).
    SetTitle("Confirm Action").
    SetBlurCaptureFunc(func(event tcell.Event) bool {
        // Capture all events when not focused (modal behavior)
        if key, ok := event.(*tcell.EventKey); ok {
            if key.Key() == tcell.KeyEscape {
                closeModal()
                return true
            }
        }
        return true  // Block all other events
    })
```

## Advanced Patterns

### Custom Composite Components

```go
type SearchableList struct {
    *mauview.Flex
    searchInput *mauview.InputField
    listView    *mauview.TextView
    items       []string
}

func NewSearchableList(items []string) *SearchableList {
    s := &SearchableList{
        Flex:        mauview.NewFlex().SetDirection(mauview.FlexColumn),
        searchInput: mauview.NewInputField(),
        listView:    mauview.NewTextView(),
        items:       items,
    }

    s.searchInput.SetPlaceholder("Filter...").
        SetChangedFunc(func(text string) {
            s.updateFilter(text)
        })

    s.AddFixedComponent(s.searchInput, 3).
        AddProportionalComponent(s.listView, 1)

    s.updateFilter("")
    return s
}

func (s *SearchableList) updateFilter(filter string) {
    var filtered []string
    for _, item := range s.items {
        if strings.Contains(strings.ToLower(item), strings.ToLower(filter)) {
            filtered = append(filtered, item)
        }
    }

    text := strings.Join(filtered, "\n")
    s.listView.SetText(text)
}
```

### Focus Chain Management

```go
type FocusChain struct {
    components []mauview.FocusableComponent
    current    int
}

func (f *FocusChain) Next() {
    if f.current >= 0 && f.current < len(f.components) {
        f.components[f.current].Blur()
    }
    f.current = (f.current + 1) % len(f.components)
    f.components[f.current].Focus()
}

func (f *FocusChain) Previous() {
    if f.current >= 0 && f.current < len(f.components) {
        f.components[f.current].Blur()
    }
    f.current--
    if f.current < 0 {
        f.current = len(f.components) - 1
    }
    f.components[f.current].Focus()
}
```

### Dynamic Component Updates

```go
type LiveDataView struct {
    *mauview.TextView
    updateChan chan string
}

func NewLiveDataView() *LiveDataView {
    l := &LiveDataView{
        TextView:   mauview.NewTextView(),
        updateChan: make(chan string, 100),
    }

    go l.processUpdates()
    return l
}

func (l *LiveDataView) processUpdates() {
    ticker := time.NewTicker(100 * time.Millisecond)
    defer ticker.Stop()

    var buffer []string
    for {
        select {
        case update := <-l.updateChan:
            buffer = append(buffer, update)
        case <-ticker.C:
            if len(buffer) > 0 {
                current := l.GetText()
                new := current + strings.Join(buffer, "\n") + "\n"
                l.SetText(new)
                l.ScrollToEnd()
                buffer = buffer[:0]
            }
        }
    }
}

func (l *LiveDataView) AppendLine(line string) {
    l.updateChan <- line
}
```

## Component Guidelines

### When to Use Each Component

**Flex**

- Simple linear layouts
- Sidebars and panels
- Headers and footers
- Any row/column arrangement

**Grid**

- Complex 2D layouts
- Dashboards
- Form layouts with labels
- Table-like structures

**InputField**

- Single-line user input
- Search boxes
- Command input
- Form fields

**InputArea**

- Multi-line text input
- Message composition
- Code editing
- Notes and comments

**TextView**

- Displaying formatted text
- Logs and output
- Help text
- Read-only content

**Box**

- Adding borders to components
- Creating panels
- Modal dialogs
- Grouping related components

### Performance Tips

1. **Minimize Redraws**: Use `RedrawSoon()` instead of `Redraw()` for batched updates
2. **Efficient Text Updates**: For TextViews with frequent updates, batch changes
3. **Lazy Loading**: For large lists, implement virtual scrolling
4. **Event Handling**: Return `true` early from event handlers to stop propagation
5. **Color Tags**: Pre-process color tags for static text to avoid runtime parsing

### Accessibility Considerations

1. **Keyboard Navigation**: Ensure all interactive elements are keyboard accessible
2. **Focus Indicators**: Use clear visual focus indicators
3. **Tab Order**: Maintain logical tab order in forms and layouts
4. **Screen Reader**: Consider terminal screen reader compatibility
5. **Color Contrast**: Ensure sufficient contrast for readability
