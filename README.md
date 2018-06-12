# Go-Chat
Console chat utility that demonstrates [PubNub](https://www.pubnub.com/) integration with Golang.

## Features 
- Leverages the PubNub Network for chat with [Publish and Subscribe](https://www.pubnub.com/docs/go/data-streams-publish-and-subscribe).
- Uses [GoCUI](https://github.com/jroimartin/gocui) for a simple console user interface.
- Username and channel are configurable.

### Quick Start
Download the appropriate pre-compiled executable from here: https://github.com/chandler767/Go-Chat/releases

### Video
*Coming soon*

### Project Tutorial
Build your own chat app with PubNub and Golang.

1. Install the latest version of [Go](https://golang.org/) and setup your $GOPATH.
2. Use `go get` in your terminal to download GoCUI and PubNub messaging packages:
	```
	go get github.com/jroimartin/gocui
	go get github.com/pubnub/go/messaging
	```
3. Create a new directory for your project and create a file named `main.go`.
4. The `main.go` will contain the code for the layout of our chat application using [GoCUI](https://github.com/jroimartin/gocui). GoCUI is a minimalist Go package aimed at creating Console User Interfaces. In this example we create two views (input and ouput) for our chat app, call a manager function that updates our views when the window size changes, and bind the enter key to the text input to submit messages. 

```
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
	"strings"

	"github.com/jroimartin/gocui"
)

func drawchat(channel string, username string) {
	// Create a new GUI.
	g, err := gocui.NewGui(gocui.OutputNormal)
	if err != nil {
		panic(err)
		return
	}
	defer g.Close()
	g.Cursor = true

	// Update the views when terminal changes size.
	g.SetManagerFunc(func(g *gocui.Gui) error {
		termwidth, termheight := g.Size()
		_, err := g.SetView("output", 0, 0, termwidth-1, termheight-4)
		if err != nil {
			return err
		}
		_, err = g.SetView("input", 0, termheight-3, termwidth-1, termheight-1)
		if err != nil {
			return err
		}
		return nil
	})

	// Terminal width and height.
	termwidth, termheight := g.Size()

	// Output.
	ov, err := g.SetView("output", 0, 0, termwidth-1, termheight-4)
	if err != nil && err != gocui.ErrUnknownView {
		log.Println("Failed to create output view:", err)
		return
	}
	ov.Title = " Messages  -  <" + channel + "> "
	ov.FgColor = gocui.ColorRed
	ov.Autoscroll = true
	ov.Wrap = true

	// Send a welcome message.
	_, err = fmt.Fprintln(ov, "<App>: Welcome to Go-Chat powered by PubNub!")
	if err != nil {
		log.Println("Failed to print into output view:", err)
	}
	_, err = fmt.Fprintln(ov, "<App>: Press Ctrl-C to quit.")
	if err != nil {
		log.Println("Failed to print into output view:", err)
	}

	// Input.
	iv, err := g.SetView("input", 0, termheight-3, termwidth-1, termheight-1)
	if err != nil && err != gocui.ErrUnknownView {
		log.Println("Failed to create input view:", err)
		return
	}
	iv.Title = " New Message  -  <" + username + "> "
	iv.FgColor = gocui.ColorWhite
	iv.Editable = true
	err = iv.SetCursor(0, 0)
	if err != nil {
		log.Println("Failed to set cursor:", err)
		return
	}

	// Bind Ctrl-C so the user can quit.
	err = g.SetKeybinding("", gocui.KeyCtrlC, gocui.ModNone, func(g *gocui.Gui, v *gocui.View) error {
		return gocui.ErrQuit
	})
	if err != nil {
		log.Println("Could not set key binding:", err)
		return
	}

	// Bind enter key to input to send new messages.
	err = g.SetKeybinding("input", gocui.KeyEnter, gocui.ModNone, func(g *gocui.Gui, iv *gocui.View) error {
		// Read buffer from the beginning.
		iv.Rewind()

		// Get output view and print.
		ov, err := g.View("output")
		if err != nil {
			log.Println("Cannot get output view:", err)
			return err
		}
		_, err = fmt.Fprintf(ov, "<%s>: %s", username, iv.Buffer())
		if err != nil {
			log.Println("Cannot print to output view:", err)
		}

		// Reset input.
		iv.Clear()

		// Reset cursor.
		err = iv.SetCursor(0, 0)
		if err != nil {
			log.Println("Failed to set cursor:", err)
		}
		return err
	})
	if err != nil {
		log.Println("Cannot bind the enter key:", err)
	}

	// Set the focus to input.
	_, err = g.SetCurrentView("input")
	if err != nil {
		log.Println("Cannot set focus to input view:", err)
	}

	// Start the main loop.
	err = g.MainLoop()
	log.Println("Main loop has finished:", err)
}

func main() {
	reader := bufio.NewReader(os.Stdin)
	fmt.Print("Enter Channel Name: ")
	channel, err := reader.ReadString('\n')
	if err != nil {
		log.Println("Could not set channel:", err)
	}
	fmt.Print("Enter Desired Username: ")
	username, err := reader.ReadString('\n')
	if err != nil {
		log.Println("Could not set username:", err)
	}
	drawchat(strings.TrimSuffix(channel, "\n"), strings.TrimSuffix(username, "\n"))
}

```
5. Run the application: `go run main.go`. You should be able to enter a channel name, username, and send messages to yourself. Now we need to integrate PubNub to send messages to other users and receive messages.
6. To be continued.
 
