# mauview Examples

This document provides practical examples of using mauview to build terminal user interfaces.

## Table of Contents

1. [Basic Examples](#basic-examples)
2. [Layout Examples](#layout-examples)
3. [Input Handling](#input-handling)
4. [Advanced UIs](#advanced-uis)
5. [Real-World Applications](#real-world-applications)

## Basic Examples

### Hello World

The simplest mauview application:

```go
package main

import (
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    text := mauview.NewTextView().
        SetText("Hello, World!").
        SetTextAlign(mauview.AlignCenter)

    app.SetRoot(text)
    app.Start()
}
```

### Basic Window with Border

```go
package main

import (
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    content := mauview.NewTextView().
        SetText("Welcome to mauview!\n\nPress Ctrl+C to exit.")

    window := mauview.NewBox(content).
        SetBorder(true).
        SetTitle("My First App")

    app.SetRoot(window)
    app.Start()
}
```

### Handling Key Events

```go
package main

import (
    "fmt"
    "github.com/gdamore/tcell/v2"
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    counter := 0
    text := mauview.NewTextField()
    text.SetText(fmt.Sprintf("Counter: %d", counter))

    // Create a component that handles key events
    handler := &KeyHandler{
        TextField: text,
        counter:   &counter,
        app:       app,
    }

    app.SetRoot(handler)
    app.Start()
}

type KeyHandler struct {
    *mauview.TextField
    counter *int
    app     *mauview.Application
}

func (k *KeyHandler) OnKeyEvent(event mauview.KeyEvent) bool {
    switch event.Key() {
    case tcell.KeyUp:
        *k.counter++
    case tcell.KeyDown:
        *k.counter--
    case tcell.KeyEscape:
        k.app.Stop()
        return true
    default:
        return false
    }

    k.SetText(fmt.Sprintf("Counter: %d (↑/↓ to change, ESC to exit)", *k.counter))
    return true
}
```

## Layout Examples

### Split Pane Layout

```go
package main

import (
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    // Left pane - file list
    fileList := mauview.NewTextView().
        SetText("file1.txt\nfile2.txt\nfile3.txt").
        SetDynamicColors(true)

    leftPane := mauview.NewBox(fileList).
        SetBorder(true).
        SetTitle("Files")

    // Right pane - file content
    fileContent := mauview.NewTextView().
        SetText("Select a file to view its contents...").
        SetWordWrap(true)

    rightPane := mauview.NewBox(fileContent).
        SetBorder(true).
        SetTitle("Content")

    // Create horizontal split
    split := mauview.NewFlex().
        SetDirection(mauview.FlexRow).
        AddFixedComponent(leftPane, 30).
        AddProportionalComponent(rightPane, 1)

    app.SetRoot(split)
    app.Start()
}
```

### Three-Panel Application

```go
package main

import (
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    // Header
    header := mauview.NewTextView().
        SetText("[yellow]My Application v1.0[white]").
        SetDynamicColors(true).
        SetTextAlign(mauview.AlignCenter)

    headerBox := mauview.NewBox(header).
        SetBorder(true)

    // Sidebar
    sidebar := mauview.NewTextView().
        SetText("Navigation\n\n• Dashboard\n• Settings\n• Help")

    sidebarBox := mauview.NewBox(sidebar).
        SetBorder(true).
        SetTitle("Menu")

    // Main content
    content := mauview.NewTextView().
        SetText("Main content area").
        SetWordWrap(true)

    contentBox := mauview.NewBox(content).
        SetBorder(true).
        SetTitle("Content")

    // Status bar
    status := mauview.NewTextField().
        SetText("Ready")

    statusBox := mauview.NewBox(status)

    // Arrange main area (sidebar + content)
    mainArea := mauview.NewFlex().
        SetDirection(mauview.FlexRow).
        AddFixedComponent(sidebarBox, 20).
        AddProportionalComponent(contentBox, 1)

    // Arrange vertically
    layout := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddFixedComponent(headerBox, 3).
        AddProportionalComponent(mainArea, 1).
        AddFixedComponent(statusBox, 1)

    app.SetRoot(layout)
    app.Start()
}
```

### Grid-based Dashboard

```go
package main

import (
    "fmt"
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    // Create widgets
    createWidget := func(title, content string) mauview.Component {
        text := mauview.NewTextView().
            SetText(content).
            SetDynamicColors(true)

        return mauview.NewBox(text).
            SetBorder(true).
            SetTitle(title)
    }

    // Create grid
    grid := mauview.NewGrid().
        SetRows(3).
        SetColumns(3).
        AddComponent(createWidget("CPU Usage", "[green]45%[white]"), 0, 0, 1, 1).
        AddComponent(createWidget("Memory", "[yellow]2.5GB / 8GB[white]"), 0, 1, 1, 1).
        AddComponent(createWidget("Disk", "[red]85% full[white]"), 0, 2, 1, 1).
        AddComponent(createWidget("Network", "↓ 1.2 MB/s\n↑ 0.3 MB/s"), 1, 0, 1, 1).
        AddComponent(createWidget("Processes", "152 running"), 1, 1, 1, 1).
        AddComponent(createWidget("Uptime", "5 days, 3:21:45"), 1, 2, 1, 1).
        AddComponent(createWidget("Logs", "System running normally..."), 2, 0, 1, 3)

    app.SetRoot(grid)
    app.Start()
}
```

## Input Handling

### Interactive Form

```go
package main

import (
    "fmt"
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    // Create form fields
    nameInput := mauview.NewInputField().
        SetPlaceholder("Enter your name...")

    emailInput := mauview.NewInputField().
        SetPlaceholder("Enter your email...")

    passwordInput := mauview.NewInputField().
        SetPlaceholder("Enter password...").
        SetMaskCharacter('*')

    // Create labels
    nameLabel := mauview.NewTextField().SetText("Name:")
    emailLabel := mauview.NewTextField().SetText("Email:")
    passwordLabel := mauview.NewTextField().SetText("Password:")

    // Result display
    result := mauview.NewTextView()

    // Submit button
    submitButton := mauview.NewButton("Submit").
        SetSelectedFunc(func() {
            name := nameInput.GetText()
            email := emailInput.GetText()
            password := passwordInput.GetText()

            result.SetText(fmt.Sprintf(
                "[green]Form submitted![white]\n\nName: %s\nEmail: %s\nPassword: %s",
                name, email, strings.Repeat("*", len(password)),
            ))
        })

    // Layout form using grid
    form := mauview.NewGrid().
        SetRows(7).
        SetColumns(2).
        AddComponent(nameLabel, 0, 0, 1, 1).
        AddComponent(nameInput, 0, 1, 1, 1).
        AddComponent(emailLabel, 1, 0, 1, 1).
        AddComponent(emailInput, 1, 1, 1, 1).
        AddComponent(passwordLabel, 2, 0, 1, 1).
        AddComponent(passwordInput, 2, 1, 1, 1).
        AddComponent(submitButton, 3, 1, 1, 1).
        AddComponent(result, 4, 0, 3, 2)

    // Wrap in a box
    formBox := mauview.NewBox(form).
        SetBorder(true).
        SetTitle("Registration Form")

    // Center the form
    centered := mauview.NewCenter(60, 20).
        SetComponent(formBox)

    app.SetRoot(centered)
    app.Start()
}
```

### Search Interface

```go
package main

import (
    "strings"
    "github.com/tulir/mauview"
)

func main() {
    app := mauview.NewApplication()

    // Sample data
    items := []string{
        "Apple", "Banana", "Cherry", "Date", "Elderberry",
        "Fig", "Grape", "Honeydew", "Ice cream", "Jackfruit",
    }

    // Search input
    searchInput := mauview.NewInputField().
        SetPlaceholder("Type to search...")

    // Results display
    results := mauview.NewTextView().
        SetDynamicColors(true)

    // Update results on search change
    updateResults := func(query string) {
        if query == "" {
            results.SetText("[gray]Enter a search term...[white]")
            return
        }

        var matches []string
        for _, item := range items {
            if strings.Contains(strings.ToLower(item), strings.ToLower(query)) {
                highlighted := strings.ReplaceAll(
                    item,
                    query,
                    fmt.Sprintf("[yellow]%s[white]", query),
                )
                matches = append(matches, highlighted)
            }
        }

        if len(matches) == 0 {
            results.SetText("[red]No matches found[white]")
        } else {
            results.SetText(strings.Join(matches, "\n"))
        }
    }

    searchInput.SetChangedFunc(updateResults)

    // Layout
    layout := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddFixedComponent(
            mauview.NewBox(searchInput).SetTitle("Search"),
            3,
        ).
        AddProportionalComponent(
            mauview.NewBox(results).
                SetBorder(true).
                SetTitle("Results"),
            1,
        )

    app.SetRoot(layout)
    app.Start()
}
```

### Command Palette

```go
package main

import (
    "strings"
    "github.com/gdamore/tcell/v2"
    "github.com/tulir/mauview"
)

type Command struct {
    Name        string
    Description string
    Action      func()
}

func main() {
    app := mauview.NewApplication()

    // Available commands
    commands := []Command{
        {"quit", "Exit the application", func() { app.Stop() }},
        {"help", "Show help information", func() { /* show help */ }},
        {"save", "Save current document", func() { /* save */ }},
        {"open", "Open a file", func() { /* open */ }},
        {"new", "Create new document", func() { /* new */ }},
    }

    // Command input with completion
    commandInput := mauview.NewInputField().
        SetPlaceholder("Type command...")

    // Results
    suggestions := mauview.NewTextView().
        SetDynamicColors(true)

    // Tab completion
    commandInput.SetTabCompleteFunc(func(text string, cursor int) string {
        for _, cmd := range commands {
            if strings.HasPrefix(cmd.Name, text) {
                return cmd.Name
            }
        }
        return text
    })

    // Update suggestions
    updateSuggestions := func(text string) {
        var matches []string
        for _, cmd := range commands {
            if strings.Contains(cmd.Name, text) {
                matches = append(matches,
                    fmt.Sprintf("[yellow]%s[white] - %s", cmd.Name, cmd.Description))
            }
        }
        suggestions.SetText(strings.Join(matches, "\n"))
    }

    commandInput.SetChangedFunc(updateSuggestions)

    // Execute command on Enter
    commandInput.SetSubmitFunc(func(text string) {
        for _, cmd := range commands {
            if cmd.Name == text {
                cmd.Action()
                return
            }
        }
    })

    // Layout
    palette := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddFixedComponent(commandInput, 3).
        AddProportionalComponent(suggestions, 1)

    paletteBox := mauview.NewBox(palette).
        SetBorder(true).
        SetTitle("Command Palette (Tab to complete)")

    centered := mauview.NewFractionalCenter(0.6, 0.4).
        SetComponent(paletteBox)

    app.SetRoot(centered)
    app.Start()
}
```

## Advanced UIs

### Tab Interface

```go
package main

import (
    "github.com/gdamore/tcell/v2"
    "github.com/tulir/mauview"
)

type TabView struct {
    *mauview.Flex
    tabs       []string
    contents   []mauview.Component
    activeTab  int
    tabBar     *mauview.TextView
    contentBox *mauview.Box
}

func NewTabView() *TabView {
    t := &TabView{
        Flex:       mauview.NewFlex().SetDirection(mauview.FlexColumn),
        tabBar:     mauview.NewTextView().SetDynamicColors(true),
        contentBox: mauview.NewBox(nil).SetBorder(true),
    }

    t.AddFixedComponent(t.tabBar, 1).
        AddProportionalComponent(t.contentBox, 1)

    return t
}

func (t *TabView) AddTab(name string, content mauview.Component) {
    t.tabs = append(t.tabs, name)
    t.contents = append(t.contents, content)

    if len(t.tabs) == 1 {
        t.SetActiveTab(0)
    } else {
        t.updateTabBar()
    }
}

func (t *TabView) SetActiveTab(index int) {
    if index >= 0 && index < len(t.tabs) {
        t.activeTab = index
        t.contentBox.SetContent(t.contents[index])
        t.contentBox.SetTitle(t.tabs[index])
        t.updateTabBar()
    }
}

func (t *TabView) updateTabBar() {
    var tabText string
    for i, tab := range t.tabs {
        if i > 0 {
            tabText += " │ "
        }
        if i == t.activeTab {
            tabText += fmt.Sprintf("[black:yellow] %s [-:-]", tab)
        } else {
            tabText += fmt.Sprintf(" %s ", tab)
        }
    }
    t.tabBar.SetText(tabText)
}

func (t *TabView) OnKeyEvent(event mauview.KeyEvent) bool {
    switch event.Key() {
    case tcell.KeyLeft:
        if t.activeTab > 0 {
            t.SetActiveTab(t.activeTab - 1)
            return true
        }
    case tcell.KeyRight:
        if t.activeTab < len(t.tabs)-1 {
            t.SetActiveTab(t.activeTab + 1)
            return true
        }
    }
    return t.Flex.OnKeyEvent(event)
}

func main() {
    app := mauview.NewApplication()

    tabs := NewTabView()

    // Add tabs
    tabs.AddTab("General",
        mauview.NewTextView().SetText("General settings..."))

    tabs.AddTab("Advanced",
        mauview.NewTextView().SetText("Advanced configuration..."))

    tabs.AddTab("About",
        mauview.NewTextView().SetText("About this application..."))

    app.SetRoot(tabs)
    app.Start()
}
```

### File Browser

```go
package main

import (
    "io/ioutil"
    "path/filepath"
    "github.com/tulir/mauview"
)

type FileBrowser struct {
    *mauview.Flex
    fileList    *mauview.TextView
    preview     *mauview.TextView
    currentPath string
}

func NewFileBrowser(path string) *FileBrowser {
    fb := &FileBrowser{
        Flex:        mauview.NewFlex().SetDirection(mauview.FlexRow),
        fileList:    mauview.NewTextView().SetDynamicColors(true),
        preview:     mauview.NewTextView().SetWordWrap(true),
        currentPath: path,
    }

    fileBox := mauview.NewBox(fb.fileList).
        SetBorder(true).
        SetTitle("Files")

    previewBox := mauview.NewBox(fb.preview).
        SetBorder(true).
        SetTitle("Preview")

    fb.AddFixedComponent(fileBox, 30).
        AddProportionalComponent(previewBox, 1)

    fb.refresh()
    return fb
}

func (fb *FileBrowser) refresh() {
    files, err := ioutil.ReadDir(fb.currentPath)
    if err != nil {
        fb.fileList.SetText("[red]Error reading directory[white]")
        return
    }

    var fileText string
    for _, file := range files {
        if file.IsDir() {
            fileText += fmt.Sprintf("[blue]📁 %s/[white]\n", file.Name())
        } else {
            fileText += fmt.Sprintf("📄 %s\n", file.Name())
        }
    }

    fb.fileList.SetText(fileText)
}

func main() {
    app := mauview.NewApplication()

    browser := NewFileBrowser(".")

    app.SetRoot(browser)
    app.Start()
}
```

### Progress Monitor

```go
package main

import (
    "time"
    "github.com/tulir/mauview"
)

type ProgressMonitor struct {
    *mauview.Flex
    tasks []*TaskProgress
}

type TaskProgress struct {
    name     string
    progress *mauview.ProgressBar
    status   *mauview.TextField
}

func NewProgressMonitor() *ProgressMonitor {
    return &ProgressMonitor{
        Flex: mauview.NewFlex().SetDirection(mauview.FlexColumn),
    }
}

func (pm *ProgressMonitor) AddTask(name string) *TaskProgress {
    task := &TaskProgress{
        name:     name,
        progress: mauview.NewProgressBar(),
        status:   mauview.NewTextField().SetText("Pending..."),
    }

    taskView := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddFixedComponent(
            mauview.NewTextField().SetText(name),
            1,
        ).
        AddFixedComponent(task.progress, 1).
        AddFixedComponent(task.status, 1)

    taskBox := mauview.NewBox(taskView).
        SetBorder(true)

    pm.AddFixedComponent(taskBox, 5)
    pm.tasks = append(pm.tasks, task)

    return task
}

func main() {
    app := mauview.NewApplication()

    monitor := NewProgressMonitor()

    // Add some tasks
    download := monitor.AddTask("Downloading files")
    process := monitor.AddTask("Processing data")
    upload := monitor.AddTask("Uploading results")

    // Simulate progress
    go func() {
        // Download
        for i := 0; i <= 100; i++ {
            download.progress.SetProgress(float64(i) / 100)
            download.status.SetText(fmt.Sprintf("Downloading... %d%%", i))
            time.Sleep(50 * time.Millisecond)
        }
        download.status.SetText("Complete!")

        // Process
        process.progress.SetIndeterminate(true)
        process.status.SetText("Processing...")
        time.Sleep(2 * time.Second)
        process.progress.SetIndeterminate(false)
        process.progress.SetProgress(1.0)
        process.status.SetText("Complete!")

        // Upload
        for i := 0; i <= 100; i++ {
            upload.progress.SetProgress(float64(i) / 100)
            upload.status.SetText(fmt.Sprintf("Uploading... %d%%", i))
            time.Sleep(30 * time.Millisecond)
        }
        upload.status.SetText("Complete!")
    }()

    centered := mauview.NewCenter(60, 20).
        SetComponent(
            mauview.NewBox(monitor).
                SetBorder(true).
                SetTitle("Task Progress"),
        )

    app.SetRoot(centered)
    app.Start()
}
```

## Real-World Applications

### Chat Interface

```go
package main

import (
    "fmt"
    "time"
    "github.com/tulir/mauview"
)

type ChatApp struct {
    *mauview.Flex
    messages   *mauview.TextView
    input      *mauview.InputArea
    userList   *mauview.TextView
    username   string
}

func NewChatApp(username string) *ChatApp {
    chat := &ChatApp{
        Flex:     mauview.NewFlex().SetDirection(mauview.FlexRow),
        messages: mauview.NewTextView().SetDynamicColors(true).SetScrollable(true),
        input:    mauview.NewInputArea().SetPlaceholder("Type a message..."),
        userList: mauview.NewTextView().SetDynamicColors(true),
        username: username,
    }

    // User list
    chat.userList.SetText("[green]● Alice[white]\n[green]● Bob[white]\n[yellow]● Charlie[white]")
    userBox := mauview.NewBox(chat.userList).
        SetBorder(true).
        SetTitle("Users")

    // Chat area
    chatArea := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddProportionalComponent(
            mauview.NewBox(chat.messages).
                SetBorder(true).
                SetTitle("Chat"),
            1,
        ).
        AddFixedComponent(
            mauview.NewBox(chat.input).
                SetBorder(true).
                SetTitle("Message"),
            5,
        )

    // Layout
    chat.AddFixedComponent(userBox, 20).
        AddProportionalComponent(chatArea, 1)

    // Handle message sending
    chat.input.SetSubmitFunc(func(text string) {
        if text != "" {
            timestamp := time.Now().Format("15:04")
            message := fmt.Sprintf("[gray]%s[white] [cyan]%s:[white] %s\n",
                timestamp, chat.username, text)

            current := chat.messages.GetText()
            chat.messages.SetText(current + message)
            chat.messages.ScrollToEnd()
            chat.input.SetText("")
        }
    })

    return chat
}

func main() {
    app := mauview.NewApplication()

    chat := NewChatApp("You")

    // Add welcome message
    chat.messages.SetText("[yellow]Welcome to the chat![white]\n\n")

    app.SetRoot(chat)
    app.Start()
}
```

### Log Viewer

```go
package main

import (
    "fmt"
    "strings"
    "time"
    "github.com/tulir/mauview"
)

type LogViewer struct {
    *mauview.Flex
    logView    *mauview.TextView
    filter     *mauview.InputField
    stats      *mauview.TextView
    allLogs    []string
}

func NewLogViewer() *LogViewer {
    lv := &LogViewer{
        Flex:    mauview.NewFlex().SetDirection(mauview.FlexColumn),
        logView: mauview.NewTextView().SetDynamicColors(true).SetScrollable(true),
        filter:  mauview.NewInputField().SetPlaceholder("Filter logs..."),
        stats:   mauview.NewTextField(),
    }

    // Top bar with filter and stats
    topBar := mauview.NewFlex().
        SetDirection(mauview.FlexRow).
        AddProportionalComponent(
            mauview.NewBox(lv.filter).SetTitle("Filter"),
            1,
        ).
        AddFixedComponent(
            mauview.NewBox(lv.stats).SetTitle("Stats"),
            30,
        )

    // Layout
    lv.AddFixedComponent(topBar, 3).
        AddProportionalComponent(
            mauview.NewBox(lv.logView).
                SetBorder(true).
                SetTitle("Logs"),
            1,
        )

    // Filter handler
    lv.filter.SetChangedFunc(func(text string) {
        lv.applyFilter(text)
    })

    return lv
}

func (lv *LogViewer) AddLog(level, message string) {
    timestamp := time.Now().Format("2006-01-02 15:04:05")

    var coloredLevel string
    switch level {
    case "ERROR":
        coloredLevel = "[red]ERROR[white]"
    case "WARN":
        coloredLevel = "[yellow]WARN [white]"
    case "INFO":
        coloredLevel = "[green]INFO [white]"
    case "DEBUG":
        coloredLevel = "[blue]DEBUG[white]"
    }

    logLine := fmt.Sprintf("[gray]%s[white] %s %s", timestamp, coloredLevel, message)
    lv.allLogs = append(lv.allLogs, logLine)

    lv.applyFilter(lv.filter.GetText())
    lv.updateStats()
}

func (lv *LogViewer) applyFilter(filter string) {
    if filter == "" {
        lv.logView.SetText(strings.Join(lv.allLogs, "\n"))
        return
    }

    var filtered []string
    for _, log := range lv.allLogs {
        if strings.Contains(strings.ToLower(log), strings.ToLower(filter)) {
            filtered = append(filtered, log)
        }
    }

    lv.logView.SetText(strings.Join(filtered, "\n"))
    lv.logView.ScrollToEnd()
}

func (lv *LogViewer) updateStats() {
    errors := 0
    warnings := 0
    for _, log := range lv.allLogs {
        if strings.Contains(log, "[red]ERROR") {
            errors++
        } else if strings.Contains(log, "[yellow]WARN") {
            warnings++
        }
    }

    lv.stats.SetText(fmt.Sprintf("Total: %d | Errors: %d | Warnings: %d",
        len(lv.allLogs), errors, warnings))
}

func main() {
    app := mauview.NewApplication()

    viewer := NewLogViewer()

    // Simulate incoming logs
    go func() {
        messages := []struct {
            level   string
            message string
        }{
            {"INFO", "Application started"},
            {"DEBUG", "Loading configuration..."},
            {"INFO", "Connected to database"},
            {"WARN", "Cache miss for key: user_123"},
            {"ERROR", "Failed to parse JSON response"},
            {"INFO", "Request processed successfully"},
            {"DEBUG", "Memory usage: 45MB"},
            {"WARN", "Slow query detected (>1s)"},
            {"INFO", "Background job completed"},
            {"ERROR", "Connection timeout to external API"},
        }

        for i, msg := range messages {
            viewer.AddLog(msg.level, msg.message)
            time.Sleep(time.Duration(500+i*100) * time.Millisecond)
        }
    }()

    app.SetRoot(viewer)
    app.Start()
}
```

### Configuration Editor

```go
package main

import (
    "encoding/json"
    "github.com/tulir/mauview"
)

type ConfigEditor struct {
    *mauview.Flex
    tree     *mauview.TextView
    editor   *mauview.InputArea
    preview  *mauview.TextView
    config   map[string]interface{}
}

func NewConfigEditor() *ConfigEditor {
    ce := &ConfigEditor{
        Flex:    mauview.NewFlex().SetDirection(mauview.FlexRow),
        tree:    mauview.NewTextView().SetDynamicColors(true),
        editor:  mauview.NewInputArea(),
        preview: mauview.NewTextView().SetDynamicColors(true),
        config: map[string]interface{}{
            "server": map[string]interface{}{
                "host": "localhost",
                "port": 8080,
                "ssl":  true,
            },
            "database": map[string]interface{}{
                "type": "postgres",
                "host": "db.example.com",
                "name": "myapp",
            },
            "logging": map[string]interface{}{
                "level": "info",
                "file":  "/var/log/app.log",
            },
        },
    }

    // Tree view
    treeBox := mauview.NewBox(ce.tree).
        SetBorder(true).
        SetTitle("Configuration")

    // Editor and preview
    editArea := mauview.NewFlex().
        SetDirection(mauview.FlexColumn).
        AddProportionalComponent(
            mauview.NewBox(ce.editor).
                SetBorder(true).
                SetTitle("Edit (JSON)"),
            1,
        ).
        AddProportionalComponent(
            mauview.NewBox(ce.preview).
                SetBorder(true).
                SetTitle("Preview"),
            1,
        )

    // Layout
    ce.AddFixedComponent(treeBox, 30).
        AddProportionalComponent(editArea, 1)

    // Initialize
    ce.updateTree()
    ce.updateEditor()

    // Handle editor changes
    ce.editor.SetChangedFunc(func(text string) {
        ce.updatePreview(text)
    })

    return ce
}

func (ce *ConfigEditor) updateTree() {
    tree := ce.renderTree(ce.config, "")
    ce.tree.SetText(tree)
}

func (ce *ConfigEditor) renderTree(data interface{}, indent string) string {
    var result string

    switch v := data.(type) {
    case map[string]interface{}:
        for key, value := range v {
            result += indent + "[yellow]" + key + ":[white]\n"
            result += ce.renderTree(value, indent+"  ")
        }
    default:
        result += fmt.Sprintf("%s%v\n", indent, v)
    }

    return result
}

func (ce *ConfigEditor) updateEditor() {
    jsonBytes, _ := json.MarshalIndent(ce.config, "", "  ")
    ce.editor.SetText(string(jsonBytes))
}

func (ce *ConfigEditor) updatePreview(text string) {
    var newConfig map[string]interface{}
    err := json.Unmarshal([]byte(text), &newConfig)

    if err != nil {
        ce.preview.SetText("[red]Invalid JSON:[white]\n" + err.Error())
    } else {
        ce.config = newConfig
        ce.updateTree()
        ce.preview.SetText("[green]✓ Valid JSON[white]\n\nConfiguration updated successfully.")
    }
}

func main() {
    app := mauview.NewApplication()

    editor := NewConfigEditor()

    app.SetRoot(editor)
    app.Start()
}
```

## Tips and Best Practices

### Performance Optimization

1. **Batch Updates**: When updating multiple components, do all updates then call `RedrawSoon()` once
2. **Lazy Loading**: For large lists, only render visible items
3. **Event Debouncing**: For expensive operations on text change, use a timer to debounce

### Error Handling

Always handle potential errors gracefully:

```go
func safeUpdate(textView *mauview.TextView, text string) {
    defer func() {
        if r := recover(); r != nil {
            textView.SetText("[red]Error occurred[white]")
        }
    }()

    textView.SetText(text)
}
```

### Thread Safety

Use channels for communication between goroutines:

```go
type SafeLogger struct {
    *mauview.TextView
    logChan chan string
}

func NewSafeLogger() *SafeLogger {
    sl := &SafeLogger{
        TextView: mauview.NewTextView(),
        logChan:  make(chan string, 100),
    }

    go sl.processLogs()
    return sl
}

func (sl *SafeLogger) processLogs() {
    for log := range sl.logChan {
        current := sl.GetText()
        sl.SetText(current + log + "\n")
        sl.ScrollToEnd()
    }
}

func (sl *SafeLogger) Log(message string) {
    sl.logChan <- message
}
```

### Custom Styling

Create consistent styles across your application:

```go
var AppStyles = struct {
    Title       tcell.Style
    Error       tcell.Style
    Success     tcell.Style
    Highlight   tcell.Style
}{
    Title:     tcell.StyleDefault.Bold(true).Foreground(tcell.ColorYellow),
    Error:     tcell.StyleDefault.Foreground(tcell.ColorRed),
    Success:   tcell.StyleDefault.Foreground(tcell.ColorGreen),
    Highlight: tcell.StyleDefault.Background(tcell.ColorDarkBlue),
}
```

## Conclusion

These examples demonstrate the flexibility and power of mauview for building terminal user interfaces.
From simple dialogs to complex applications, mauview provides the components and patterns needed for
effective TUI development.

For more information, refer to:

- [API Documentation](./API.md)
- [Component Guide](./Components.md)
- [mauview GitHub Repository](https://github.com/tulir/mauview)
