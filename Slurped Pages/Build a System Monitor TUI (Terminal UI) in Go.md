---
link: https://penchev.com/posts/create-tui-with-go/
byline: Ivan Penchev
site: Ivan Penchev
date: 2024-11-02T12:22
excerpt: "The goal of this article is to: Learn how to access and read system information. Learn how to build a text-based user interface in the Terminal. We will be using two libraries to achieve this goal shirou/gopsutil and charmbracelet/bubbletea If you want to skip my ramblings and just see the final solution, the source code is available here."
twitter: https://twitter.com/@ivan-penchev
slurped: 2025-10-20T17:47
title: Build a System Monitor TUI (Terminal UI) in Go
tags:
  - go
  - bubble
  - tui
---

> The goal of this article is to:
> 
> 1. Learn how to access and read system information.
> 2. Learn how to build a text-based user interface in the Terminal.
> 
> We will be using two libraries to achieve this goal [shirou/gopsutil](https://github.com/shirou/gopsutil) and [charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea)
> 
> If you want to skip my ramblings and just see the final solution, the source code is available [here](https://github.com/ivan-penchev/system-monitor-tui).
## Why a TUI?[](https://penchev.com/posts/create-tui-with-go/#why-a-tui)

As an engineer, I often have to interact with systems. And I have to do it in a manner where it is consistent, meaning if I am creating a vendor account in 2022 or today, I must always use the same procedure. The problem? Human memory is far from flawless, we may have done a procedure over, and over, and over again… and forget to do a small step. You don’t have to look further than the case of the [2018 Hawaii false missile alert](https://en.wikipedia.org/wiki/2018_Hawaii_false_missile_alert) to see the potential consequences of such lapses.

So how do we as engineers combat this? We try to automate the interactions so we do not have to rely on our memory. Scripting repetitive tasks is a common approach. I can’t remember how often I have seen, or written myself, a bash script similar to:

#!/bin/bash<br><br>echo "Select an action:"<br>select action in "Create Vendor Account" "Update Vendor Information" "Deactivate Vendor Account" "Exit"<br>do<br>    case $action in<br>        "Create Vendor Account")<br>            # Command to create a vendor account<br>            echo "Creating vendor account..."<br>            ;;<br>        "Update Vendor Information")<br>            # Command to update vendor information<br>            echo "Updating vendor information..."<br>            ;;<br>        "Deactivate Vendor Account")<br>            # Command to deactivate a vendor account<br>            echo "Deactivating vendor account..."<br>            ;;<br>        "Exit")<br>            break<br>            ;;<br>        *)<br>            echo "Invalid option. Please try again."<br>            ;;<br>    esac<br>done|

In layman’s terms, this is a ‘menu’ that allows the user to choose from a list of actions. Once an action is selected, the corresponding command is executed. While this is a step in the right direction, the interface is purely text-based and can be cumbersome to navigate, especially for more complex interactions.

While this script is functional, it lacks the user-friendly interface that a TUI can provide. A TUI provides a graphical interface within a text-only terminal environment, improving user interaction by offering elements like menus, buttons, and forms. It bridges the gap between the ease of use found in Graphical User Interfaces (GUIs) and the power and accessibility of command-line interfaces. TUIs are particularly useful when GUIs are not practical.

Moreover, TUIs can offer a better user experience than simple scripts. They can provide immediate feedback, error handling, and a more intuitive navigation system, which can be essential for complex or critical tasks. By automating procedures through a TUI, engineers can ensure that processes are followed accurately and efficiently, minimizing the risk of human error and increasing productivity.

## Starting up[](https://penchev.com/posts/create-tui-with-go/#starting-up)

Let us create a basic skeleton for our project. Usually I would advice you follow the [Standard Go Project Layout](https://github.com/golang-standards/project-layout), but due to the expected size of this project, I will just plug everything into one package.

mkdir system-monitor-tui<br>cd system-monitor-tui<br>go mod init github.com/ivan-penchev/system-monitor-tui<br>touch main.go

package main<br><br>import (<br>	"fmt"<br>)<br><br>func main() {<br>	printSystemInfo()<br>}<br><br>func printSystemInfo() {<br>	// TO-DO: Retrieve system information<br>	fmt.Println("CPU Percentage    :", "TO-DO")<br>	fmt.Println("Memory Percentage :", "TO-DO")<br>	fmt.Println("Running Processes :", "TO-DO")<br>}

Congrats we did the the equivalent of a “hello world” :)

With our project structure in place, we can now focus on retrieving system resources using the `gopsutil` library.

## Getting System Resources using gopsutil[](https://penchev.com/posts/create-tui-with-go/#getting-system-resources-using-gopsutil)

`gopsutil` is a Go package that provides a set of functions to retrieve system and process metrics from various platforms. It serves as a convenient and cross-platform way to gather information about system resources such as CPU usage, memory usage, disk usage, and more.

In other words, with this package, we can run our code on Windows, Linux, and Mac, and it will still provide us with the system resources.

touch systeminfo.go<br>go get github.com/shirou/gopsutil/v4

We would create 3 methods:

1. `GetCPUStats` this function retrieves CPU usage statistics and converts them into percentage values. The `cpu.Times(false)` function returns the CPU times, and the function calculates the percentage of time spent in various states (user, system, idle, etc.).
2. `GetMEMStats` this function retrieves memory usage statistics using `mem.VirtualMemory()`. It returns the total, used, free, and available memory, along with the percentage of used memory.
3. `GetProcesses` this function retrieves a list of running processes. It gathers information such as PID, name, username, memory usage, CPU usage percentage, and running time for each process. The processes are then sorted by CPU usage percentage in descending order, and the top n processes are returned.

**View the full systeminfo.go file**

package main<br><br>import (<br>	"fmt"<br>	"sort"<br>	"time"<br><br>	"github.com/shirou/gopsutil/v4/cpu"<br>	"github.com/shirou/gopsutil/v4/mem"<br>	"github.com/shirou/gopsutil/v4/process"<br>)<br><br>func GetCPUStats() (cpu.TimesStat, error) {<br>	stats, err := cpu.Times(false)<br>	if err != nil {<br>		return cpu.TimesStat{}, err<br>	}<br>	if len(stats) == 0 {<br>		return cpu.TimesStat{}, nil<br>	}<br><br>	currStats := stats[0]<br><br>	total := currStats.User + currStats.System + currStats.Idle + currStats.Nice +<br>		currStats.Iowait + currStats.Irq + currStats.Softirq + currStats.Steal +<br>		currStats.Guest<br><br>	if total == 0 {<br>		return cpu.TimesStat{}, nil<br>	}<br><br>	// Overwrite TimesStat fields with percentage values<br>	currStats.User = (currStats.User / total) * 100<br>	currStats.System = (currStats.System / total) * 100<br>	currStats.Idle = (currStats.Idle / total) * 100<br>	currStats.Nice = (currStats.Nice / total) * 100<br>	currStats.Iowait = (currStats.Iowait / total) * 100<br>	currStats.Irq = (currStats.Irq / total) * 100<br>	currStats.Softirq = (currStats.Softirq / total) * 100<br>	currStats.Steal = (currStats.Steal / total) * 100<br>	currStats.Guest = (currStats.Guest / total) * 100<br><br>	return currStats, nil<br>}<br><br>func GetMEMStats() (mem.VirtualMemoryStat, error) {<br>	v, err := mem.VirtualMemory()<br>	if err != nil {<br>		return mem.VirtualMemoryStat{}, err<br>	}<br><br>	return mem.VirtualMemoryStat{<br>		Total:       v.Total,<br>		Used:        v.Used,<br>		Free:        v.Free,<br>		UsedPercent: v.UsedPercent,<br>		Available:   v.Available,<br>	}, nil<br>}<br><br>type ProcessInfo struct {<br>	PID         int32<br>	Name        string<br>	Username    string<br>	Memory      uint64<br>	CPUPercent  float64 // CPU usage percentage<br>	RunningTime string<br>}<br><br>func GetProcesses(n int) ([]ProcessInfo, error) {<br>	procs, err := process.Processes()<br>	if err != nil {<br>		return nil, err<br>	}<br><br>	var processInfos []ProcessInfo<br>	for _, p := range procs {<br>		pid := p.Pid<br>		name, err := p.Name()<br>		if err != nil {<br>			name = "Unknown"<br>		}<br><br>		createTime, err := p.CreateTime()<br>		if err != nil {<br>			createTime = 0<br>		}<br><br>		startTime := time.Unix(createTime/1000, 0)<br>		runningTime := time.Since(startTime).Truncate(time.Second)<br><br>		username, err := p.Username()<br>		if err != nil {<br>			name = "Unknown"<br>		}<br><br>		memoryInfo, err := p.MemoryInfo()<br>		if err != nil {<br>			processInfos = append(processInfos, ProcessInfo{<br>				PID:         pid,<br>				Name:        name,<br>				RunningTime: runningTime.String(),<br>				Username:    username,<br>				Memory:      0,<br>				CPUPercent:  0,<br>			})<br>			continue<br>		}<br><br>		memory := memoryInfo.RSS<br><br>		cpuPercent, err := p.CPUPercent()<br>		if err != nil {<br>			cpuPercent = 0<br>		}<br><br>		processInfos = append(processInfos, ProcessInfo{<br>			PID:         pid,<br>			Name:        name,<br>			RunningTime: runningTime.String(),<br>			Username:    username,<br>			Memory:      memory,<br>			CPUPercent:  cpuPercent,<br>		})<br>	}<br><br>	sort.Slice(processInfos, func(i, j int) bool {<br>		return processInfos[i].CPUPercent > processInfos[j].CPUPercent<br>	})<br><br>	if len(processInfos) > n {<br>		processInfos = processInfos[:n]<br>	}<br><br>	return processInfos, nil<br>}<br><br>func convertBytes(bytes uint64) (string, string) {<br>	const (<br>		KB = 1024<br>		MB = KB * 1024<br>		GB = MB * 1024<br>	)<br><br>	switch {<br>	case bytes >= GB:<br>		return fmt.Sprintf("%.2f", float64(bytes)/float64(GB)), "GB"<br>	case bytes >= MB:<br>		return fmt.Sprintf("%.2f", float64(bytes)/float64(MB)), "MB"<br>	case bytes >= KB:<br>		return fmt.Sprintf("%.2f", float64(bytes)/float64(KB)), "KB"<br>	default:<br>		return fmt.Sprintf("%d", bytes), "B"<br>	}<br>}

Now that we have our system information functions ready, let’s integrate them into our main program.

package main<br><br>import (<br>	"fmt"<br>)<br><br>func main() {<br>	printSystemInfo()<br>}<br><br>func printSystemInfo() {<br>	cpuUsage, _ := GetCPUStats()<br>	memoryUsage, _ := GetMEMStats()<br>	runningProcesses, _ := GetProcesses(10) // Get top 10 CPU intensive processes<br><br>	fmt.Println("CPU Percentage    :", cpuUsage)<br>	fmt.Println("Memory Percentage :", memoryUsage)<br>	fmt.Println("Running Processes :", runningProcesses)<br>}

### Looking and understanding the output[](https://penchev.com/posts/create-tui-with-go/#looking-and-understanding-the-output)

Now if everything worked as expected you would be able to see the output. Lets examine it, to understand it better. this would help us when we have to design our TUI.

1. CPU Percentage output

{<br>  "cpu": "cpu-total",      // The CPU identifier.<br>  "user": 0.2,             // Percentage of CPU time spent in user mode.<br>  "system": 0.5,           // Percentage of CPU time spent in system mode.<br>  "idle": 99.2,            // Percentage of CPU time spent idle.<br>  "nice": 0.0,             // Percentage of CPU time spent on low-priority processes.<br>  "iowait": 0.0,           // Percentage of CPU time spent waiting for I/O operations.<br>  "irq": 0.0,              // Percentage of CPU time spent servicing interrupts.<br>  "softirq": 0.0,          // Percentage of CPU time spent servicing software interrupts.<br>  "steal": 0.0,            // Percentage of CPU time stolen by other operating systems running in a virtualized environment.<br>  "guest": 0.0,            // Percentage of CPU time spent running guest operating systems.<br>  "guestNice": 0.0         // Percentage of CPU time spent running guest operating systems with a low priority.<br>}

1. Memory Percentage output:

{<br>  "total": 33411727360,    // Total physical memory available.<br>  "available": 17795051520, // Memory available for use.<br>  "used": 15616675840,     // Memory currently used.<br>  "usedPercent": 46,       // Percentage of memory used.<br>  "free": 17795051520,     // Free memory.<br>}

1. Running Processes output:

 {<br>    "PID": 7896,                          // Process ID.<br>    "Name": "system-monitor-tui.exe",     // Name of the process.<br>    "Username": "NETA\\ivan",             // Username of the process owner.<br>    "Memory": 7536640,                    // Memory used by the process (in bytes).<br>    "CPUPercent": 85.02660992784301,      // CPU usage percentage of the process.<br>    "RunningTime": "0s"                   // How long the process has been running.<br>  }<br>  //... omitted<br>

### Using go routines and channels to refresh information?[](https://penchev.com/posts/create-tui-with-go/#using-go-routines-and-channels-to-refresh-information)

> This section is optional if you are already familiar with how goroutines and channels work.
> 
> The reason I am highlighting this, is because when using [bubbletea](https://github.com/charmbracelet/bubbletea) the framework abstract those concepts away. Yet it is still a good idea to be familiar with the “magic” under the hood of the framework.

Goroutines are distinct from traditional threads in that they are managed by the Go runtime, which is responsible for their scheduling and execution. They are more lightweight than operating system threads, and many goroutines can run concurrently on a small number of operating system threads. This enables efficient parallelism and concurrency without the overhead associated with traditional threading models.

In Go, you can create a new goroutine by using the go keyword followed by a function call. For example:

func main() {<br>	go worker() // Start a worker goroutine<br><br>	// Keep the main goroutine running<br>	select {}<br>}<br><br>func worker() {<br>	for {<br>		fmt.Println("Working...")<br>		time.Sleep(time.Second)<br>	}<br>}

In this example, the worker function runs in a separate goroutine, printing “Working…” every second. The select{} statement ensures that the main Goroutine continues running indefinitely, preventing the program from exiting immediately, thus allowing the worker goroutine to continue its work concurrently.

func main() {<br>	// Start the worker goroutine using an anonymous function<br>	go func() {<br>	 for {<br>		 worker()<br>		 time.Sleep(time.Second)<br>	 }<br>	}()<br><br>	// Keep the main goroutine running<br>	select {}<br>}<br><br>func worker() {<br>	for {<br>		fmt.Println("Working...")<br>		// You may include additional logic here if needed<br>	}

Now that you’ve learned how goroutines work , update your program to continuously print CPU usage, memory usage, and running processes.

package main<br><br>import (<br>	"fmt"<br>	"os"<br>	"os/signal"<br>	"sync"<br>	"syscall"<br>	"time"<br>)<br><br>func main() {<br>	// Create a channel to listen for OS signals<br>	stopChan := make(chan os.Signal, 1)<br>	signal.Notify(stopChan, syscall.SIGINT, syscall.SIGTERM)<br><br>	// Create a channel to signal when to print system info<br>	printChan := make(chan struct{})<br><br>	// Use a wait group to wait for all goroutines to finish<br>	var wg sync.WaitGroup<br><br>	// Start a goroutine to handle the printing<br>	wg.Add(1)<br>	go func() {<br>		defer wg.Done()<br>		printSystemInfo(printChan)<br>	}()<br><br>	// Create a ticker to signal every 10 seconds<br>	ticker := time.NewTicker(10 * time.Second)<br>	defer ticker.Stop()<br><br>	// Main loop to handle signals and ticker<br>    // as well as keep the main goroutine running<br>	for {<br>		select {<br>		case <-ticker.C:<br>			printChan <- struct{}{}<br>		case <-stopChan:<br>			fmt.Println("Received stop signal, stopping...")<br>			close(printChan)<br>			wg.Wait()<br>			return<br>		}<br>	}<br>}<br>func printSystemInfo(printChan chan struct{}) {<br>	for range printChan {<br>		cpuUsage, _ := GetCPUStats()<br><br>		memoryUsage, _ := GetMEMStats()<br><br>		runningProcesses, _ := GetProcesses(10)<br><br>		fmt.Println("CPU Percentage    :", cpuUsage)<br>		fmt.Println("Memory Percentage :", memoryUsage)<br>		fmt.Println("Running Processes :", runningProcesses)<br>	}<br>}

This is a very standard pattern for fetching and refreshing data. Our framework abstracts and handles all this underneath, so we wouldn’t actually be writing this directly.

## Writing TUI using bubbletea[](https://penchev.com/posts/create-tui-with-go/#writing-tui-using-bubbletea)

Creating a Text User Interface (TUI) can be a complex task, especially if you aim to build a robust, interactive, and visually appealing interface. Using a framework like Bubble Tea can significantly simplify this process. Bubble Tea is a modern TUI framework for Go, inspired by [The Elm Architecture](https://guide.elm-lang.org/architecture/). Here are some specific benefits of using Bubble Tea:

Firstly, it employs a Model-Update-View (MUV) architecture, which ensures a clear separation of concerns. The model is responsible for holding the application state, the update function handles state changes, and the view function renders the state. This structured approach simplifies the management of complex state transitions.

Secondly, Bubble Tea provides a declarative UI, which significantly enhances ease of use. It allows developers to straightforwardly declare what the UI should look like, making it easier to understand and reason about the UI and its changes over time.

Thirdly, Bubble Tea includes a rich library of built-in components, such as text inputs, lists, and tables, which can be used out of the box. These components are also highly customizable, allowing developers to extend and modify them to meet specific needs.

Lastly, Bubble Tea supports concurrency, enabling asynchronous operations. This feature allows background tasks, such as data fetching, to be handled without blocking the UI, ensuring a smooth and responsive user experience.

For our project we would be using the following packages:

1. [charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea) - is the core framework for building TUIs in Go.
2. [charmbracelet/bubbles](https://github.com/charmbracelet/bubbles) - a collection of reusable components for building TUIs. It includes various elements like text inputs, lists, tables, and more, which can be easily integrated into your application.
3. [charmbracelet/lipgloss](https://github.com/charmbracelet/lipgloss) - a package for styling terminal applications. It provides tools to add colors, styles, and layouts to your TUI, allowing you to create visually appealing interfaces.

`|   |   | |---|---| |1<br>2<br>3<br>4|touch tui.go<br>go get github.com/charmbracelet/lipgloss<br>go get github.com/charmbracelet/bubbletea<br>go get github.com/charmbracelet/bubbles|`

With these packages installed, we can start defining our TUI model.

### Define our Model (struct)[](https://penchev.com/posts/create-tui-with-go/#define-our-model-struct)

In order to create our TUI we must define a model. For something to be considered a `Model` it must have the following 3 methods:

type Model interface {<br>    // Init is the first function that will be called. It returns an optional<br>    // initial command. To not perform an initial command return nil.<br>    Init() Cmd<br><br>    // Update is called when a message is received. Use it to inspect messages<br>    // and, in response, update the model and/or send a command.<br>    Update(Msg) (Model, Cmd)<br><br>    // View renders the program's UI, which is just a string. The view is<br>    // rendered after every Update.<br>    View() string<br>}

We need to store some data for our Model. We will use the following model to store the data for our state:

type model struct {<br>	width  int<br>	height int<br><br>	processTable table.Model<br>	tableStyle   table.Styles<br>	baseStyle    lipgloss.Style<br>	viewStyle    lipgloss.Style<br><br>	CpuUsage cpu.TimesStat<br>	MemUsage mem.VirtualMemoryStat<br>}<br><br>type TickMsg time.Time

Next, let’s define the view for our model.

### Define our Model (View)[](https://penchev.com/posts/create-tui-with-go/#define-our-model-view)

The view defines how the user interface (UI) is rendered and what elements are displayed.

func (m model) View() string {<br>	// Sets the width of the column to the width of the terminal (m.width) and adds padding of 1 unit on the top.<br>	// Render is a method from the lipgloss package that applies the defined style and returns a function that can render styled content.<br>	column := m.baseStyle.Width(m.width).Padding(1, 0, 0, 0).Render<br>	// Set the content to match the terminal dimensions (m.width and m.height).<br>	content := m.baseStyle.<br>		Width(m.width).<br>		Height(m.height).<br>		Render(<br>			// Vertically join multiple elements aligned to the left.<br>			lipgloss.JoinVertical(lipgloss.Left,<br>				column(m.ViewHeader()),<br>				column(m.ViewProcess()),<br>			),<br>		)<br><br>	return content<br>}

**View ViewHeader() function**

// Uses lipgloss.JoinVertical and lipgloss.JoinHorizontal to arrange the header content.<br>// It displays the last update time and various system statistics (CPU and memory usage) in a structured format.<br>func (m model) ViewHeader() string {<br>	// defines the style for list items, including borders, border color, height, and padding.<br>	list := m.baseStyle.<br>		Border(lipgloss.NormalBorder(), false, true, false, false).<br>		BorderForeground(Color.Border).<br>		Height(4).<br>		Padding(0, 1)<br><br>	// applies bold styling to the text.<br>	listHeader := m.baseStyle.Bold(true).Render<br><br>	// helper function that formats a key-value pair with an optional suffix. It aligns the value to the right and renders it with the specified style.<br>	listItem := func(key string, value string, suffix ...string) string {<br>		finalSuffix := ""<br>		if len(suffix) > 0 {<br>			finalSuffix = suffix[0]<br>		}<br><br>		listItemValue := m.baseStyle.Align(lipgloss.Right).Render(fmt.Sprintf("%s%s", value, finalSuffix))<br><br>		listItemKey := func(key string) string {<br>			return m.baseStyle.Render(key + ":")<br>		}<br><br>		return fmt.Sprintf("%s %s", listItemKey(key), listItemValue)<br>	}<br><br>	return m.viewStyle.Render(<br>		lipgloss.JoinVertical(lipgloss.Top,<br>			fmt.Sprintf("Last update: %d milliseconds ago\n", time.Now().Sub(m.lastUpdate).Milliseconds()),<br>			lipgloss.JoinHorizontal(lipgloss.Top,<br>				// Progress Bars<br>				list.Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader("% Usage"),<br>						listItem("CPU", fmt.Sprintf("%s %.1f", ProgressBar(100-m.CpuUsage.Idle, m.baseStyle), 100-m.CpuUsage.Idle), "%"),<br>						listItem("MEM", fmt.Sprintf("%s %.1f", ProgressBar(m.MemUsage.UsedPercent, m.baseStyle), m.MemUsage.UsedPercent), "%"),<br>					),<br>				),<br><br>				// CPU<br>				list.Border(lipgloss.NormalBorder(), false).Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader("CPU"),<br>						listItem("user", fmt.Sprintf("%.1f", m.CpuUsage.User), "%"),<br>						listItem("sys", fmt.Sprintf("%.1f", m.CpuUsage.System), "%"),<br>						listItem("idle", fmt.Sprintf("%.1f", m.CpuUsage.Idle), "%"),<br>					),<br>				),<br>				list.Border(lipgloss.NormalBorder(), false).Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader(""),<br>						listItem("nice", fmt.Sprintf("%.1f", m.CpuUsage.Nice), "%"),<br>						listItem("iowait", fmt.Sprintf("%.1f", m.CpuUsage.Iowait), "%"),<br>						listItem("irq", fmt.Sprintf("%.1f", m.CpuUsage.Irq), "%"),<br>					),<br>				),<br>				list.Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader(""),<br>						listItem("softirq", fmt.Sprintf("%.1f", m.CpuUsage.Softirq), "%"),<br>						listItem("steal", fmt.Sprintf("%.1f", m.CpuUsage.Steal), "%"),<br>						listItem("guest", fmt.Sprintf("%.1f", m.CpuUsage.Guest), "%"),<br>					),<br>				),<br><br>				// MEM<br>				list.Border(lipgloss.NormalBorder(), false).Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader("MEM"),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Total)<br>							return listItem("total", value, unit)<br>						}(),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Used)<br>							return listItem("used", value, unit)<br>						}(),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Available)<br>							return listItem("free", value, unit)<br>						}(),<br>					),<br>				),<br>				list.Render(<br>					lipgloss.JoinVertical(lipgloss.Left,<br>						listHeader(""),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Active)<br>							return listItem("active", value, unit)<br>						}(),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Buffers)<br>							return listItem("buffers", value, unit)<br>						}(),<br>						func() string {<br>							value, unit := convertBytes(m.MemUsage.Cached)<br>							return listItem("cached", value, unit)<br>						}(),<br>					),<br>				),<br>			),<br>		),<br>	)<br>}

**View ViewProcess() and ProgressBar() functions**

// Gets the View from the table component and renders is on our View<br>func (m model) ViewProcess() string {<br>	return m.viewStyle.Render(m.processTable.View())<br>}<br><br>// creates a visual representation of a percentage as a progress bar.<br>func ProgressBar(percentage float64, baseStyle lipgloss.Style) string {<br>	totalBars := 20<br>	fillBars := int(percentage / 100 * float64(totalBars))<br>	// renders the filled part of the progress bar with a green color.<br>	filled := baseStyle.<br>		Foreground(Color.Green).<br>		Render(strings.Repeat("\|", fillBars))<br>	// renders the empty part of the progress bar with a secondary color.<br>	empty := baseStyle.<br>		Foreground(Color.Secondary).<br>		Render(strings.Repeat("\|", totalBars-fillBars))<br><br>	return baseStyle.Render(fmt.Sprintf("%s%s%s%s", "[", filled, empty, "]"))<br>}

With our view in place, let’s move on to initializing our model.

### Define our Model (Init)[](https://penchev.com/posts/create-tui-with-go/#define-our-model-init)

The `Init` function is part of the `tea.Model` interface and is called once when the Bubble Tea program starts. It is used to initialize the model and set up any initial commands. If we do not wish to perform any initial commands, we simply return nil.

// Calls the tickEvery function to set up a command that sends a TickMsg every second.<br>// This command will be executed immediately when the program starts, initiating the periodic updates.<br>func (m model) Init() tea.Cmd {<br>	return tickEvery()<br>}<br><br>func tickEvery() tea.Cmd {<br>	// tea.Every function is a helper function from the Bubble Tea framework<br>	// that schedules a command to run at regular intervals.<br>	return tea.Every(time.Second,<br>		// Callback function that takes the current time (t time.Time) as a parameter and returns a message (tea.Msg).<br>		// This callback is invoked every second.<br>		func(t time.Time) tea.Msg {<br>			return TickMsg(t)<br>		})<br>}

### Define our Model (Update)[](https://penchev.com/posts/create-tui-with-go/#define-our-model-update)

Once we have initialized our model, and configured how it must look (view), the last thing we must do is handle updates. This is where the Update function of the model comes into play.

// Takes a tea.Msg as input and uses a type switch to handle different types of messages.<br>// Each case in the switch statement corresponds to a specific message type.<br>func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {<br>	switch msg := msg.(type) {<br><br>	// message is sent when the window size changes<br>	// save to reflect the new dimensions of the terminal window.<br>	case tea.WindowSizeMsg:<br>		m.height = msg.Height<br>		m.width = msg.Width<br><br>	// message is sent when a key is pressed.<br>	case tea.KeyMsg:<br>		switch msg.String() {<br>		// Toggles the focus state of the process table<br>		case "esc":<br>			if m.processTable.Focused() {<br>				m.tableStyle.Selected = m.baseStyle<br>				m.processTable.SetStyles(m.tableStyle)<br>				m.processTable.Blur()<br>			} else {<br>				m.tableStyle.Selected = m.tableStyle.Selected.Background(Color.Highlight)<br>				m.processTable.SetStyles(m.tableStyle)<br>				m.processTable.Focus()<br>			}<br>		// Moves the focus up in the process table if the table is focused.<br>		case "up", "k":<br>			if m.processTable.Focused() {<br>				m.processTable.MoveUp(1)<br>			}<br>		// Moves the focus down in the process table if the table is focused.<br>		case "down", "j":<br>			if m.processTable.Focused() {<br>				m.processTable.MoveDown(1)<br>			}<br>		// Quits the program by returning the tea.Quit command.<br>		case "q", "ctrl+c":<br>			return m, tea.Quit<br>		}<br>	// This custom message is sent periodically by the tickEvery function.<br>	// The model's lastUpdate field is updated to the current time.<br>	// Fetching CPU Stats, Memory Stats & Processes<br>	// Returning Command: The tickEvery command is returned to ensure that the TickMsg continues to be sent periodically.<br>	case TickMsg:<br>		m.lastUpdate = time.Time(msg)<br>		cpuStats, err := GetCPUStats()<br>		if err != nil {<br>			slog.Error("Could not get CPU info", "error", err)<br>		} else {<br>			m.CpuUsage = cpuStats<br>		}<br><br>		memStats, err := GetMEMStats()<br>		if err != nil {<br>			slog.Error("Could not get memory info", "error", err)<br>		} else {<br>			m.MemUsage = memStats<br>		}<br><br>		procs, err := GetProcesses(5)<br>		if err != nil {<br>			slog.Error("Could not get processes", "error", err)<br>		} else {<br>			rows := []table.Row{}<br>			for _, p := range procs {<br>				memString, memUnit := convertBytes(p.Memory)<br>				rows = append(rows, table.Row{<br>					fmt.Sprintf("%d", p.PID),<br>					p.Name,<br>					fmt.Sprintf("%.2f%%", p.CPUPercent),<br>					fmt.Sprintf("%s %s", memString, memUnit),<br>					p.Username,<br>					p.RunningTime,<br>				})<br>			}<br>			m.processTable.SetRows(rows)<br>		}<br><br>		return m, tickEvery()<br>	}<br>	// If the message type does not match any of the handled cases, the model is returned unchanged, and no new command is issued.<br>	return m, nil<br>}<br><br>func (m model) View() string {<br>	// Sets the width of the column to the width of the terminal (m.width) and adds padding of 1 unit on the top.<br>	// Render is a method from the lipgloss package that applies the defined style and returns a function that can render styled content.<br>	column := m.baseStyle.Width(m.width).Padding(1, 0, 0, 0).Render<br>	// Set the content to match the terminal dimensions (m.width and m.height).<br>	content := m.baseStyle.<br>		Width(m.width).<br>		Height(m.height).<br>		Render(<br>			// Vertically join multiple elements aligned to the left.<br>			lipgloss.JoinVertical(lipgloss.Left,<br>				column(m.ViewHeader()),<br>				column(m.ViewProcess()),<br>			),<br>		)<br><br>	return content<br>}

Finally, let’s put everything together and run our TUI application.

### Putting it all together[](https://penchev.com/posts/create-tui-with-go/#putting-it-all-together)

As a final step, after defining our Model, we need to run the application. To do this, we will update our main.go file.

func main() {<br>	tableStyle := table.DefaultStyles()<br>	tableStyle.Selected = lipgloss.NewStyle().Background(Color.Highlight)<br><br>	// Creates a new table with specified columns and initial empty rows.<br>	processTable := table.New(<br>		// We use this to define our table "header"<br>		table.WithColumns([]table.Column{<br>			{Title: "PID", Width: 10},<br>			{Title: "Name", Width: 25},<br>			{Title: "CPU", Width: 12},<br>			{Title: "MEM", Width: 12},<br>			{Title: "Username", Width: 12},<br>			{Title: "Time", Width: 12},<br>		}),<br>		table.WithRows([]table.Row{}),<br>		table.WithFocused(true),<br>		table.WithHeight(20),<br>		table.WithStyles(tableStyle),<br>	)<br><br>	m := model{<br>		processTable: processTable,<br>		tableStyle:   tableStyle,<br>		baseStyle:    lipgloss.NewStyle(),<br>		viewStyle:    lipgloss.NewStyle(),<br>	}<br><br>	// Create a new Bubble Tea program with the model and enable alternate screen<br>	p := tea.NewProgram(m, tea.WithAltScreen())<br><br>	// Run the program and handle any errors<br>	if _, err := p.Run(); err != nil {<br>		log.Fatalf("Error running program: %v", err)<br>	}<br>}

With our main.go file updated, we can now run our application and see the TUI in action.

If you wired everything correctly, you can run the code with `go run .` The result should look like: [![create-tui-with-go](https://penchev.com/assets/img/create-tui-with-go.png)](https://penchev.com/assets/img/create-tui-with-go.png)

## Conclusion[](https://penchev.com/posts/create-tui-with-go/#conclusion)

While the software we created is practically ‘cloning’ [htop](https://htop.dev/), it is nonetheless impressive what you can achieve with the available libraries for Go.

We have successfully built a cross-platform TUI that provides real-time system monitoring. And it only took an hour to create.

Keep building, keep coding!

> The following post has been hugely inspired by [World Cup 2022 CLI Dashboard](https://github.com/cedricblondeau/world-cup-2022-cli-dashboard/tree/main)