# Click Manual
Welcome to the Click manual! This document will provide a starting point for everyone who wants to try out Click and learn more about its features and design. It will also document some internal workings. However, please note that this manual is not a technical documentation and does not cover all widgets or features; refer to the examples and the source code including the doc comments instead for a more direct view. It is also not a proper tutorial/introducing to UI development, although it explains the most important concepts first.

## Table of Contents
<table>
<tr><td width=50% valign=top>

* [What's the point of Click? What even is a GUI framework?](#whats-the-point-of-click-what-even-is-a-gui-framework)
* [Core Principles](#core-principles)
    * [Widgets](#widgets)
    * [Optimized Immediate-Mode](#optimized-immediate-mode)
    * [Controlled State Management](#controlled-state-management)
    * [FlexBox Pattern](#flexbox-pattern)
    * [Unified Pointer Interface](#unified-pointer-interface)
* [Installing Click](#installing-click)

</td><td width=50% valign=top>

* [Building your first App](#building-your-first-app)
* [FlexBoxes, Rows, and Columns](#flexboxes-rows-and-columns)
* [Sizings](#sizings)
* [Images, Families and Virtual Coordinates](#images-families-and-virtual-coordinates)
* [State Management](#state-management)
* [Unified Pointer Interface](#unified-pointer-interface)
* [Troubleshooting](#troubleshooting)

</td></tr>
</table>

## What's the point of Click? What even is a GUI framework?
One thing about software development - and (computer) science in general - is that people who aren't particularly familiar with the field often don't even realize the need for lots of software. This includes GUI frameworks. So what exactly is a GUI framework? Do you need one?

Generally speaking, you'll want to use a GUI ("graphical user interface") framework whenever you're building software that uses a graphical interface to display information and interact with users - such as a smartphone app, an interactive website[^1], or a video game's settings menu.

Now, you might wonder what you actually require the framework for. Imagine the following:
You are building an app, let's say for a railway company.

The app should contain text fields for typing in the departure and arrival stations, buttons for navigating and submitting requests, switches for toggling options, and more. These components are known as widgets (at least in Click); one major job of GUI frameworks is to implement a base logic for all widgets and provide some common widgets (buttons, switches, etc.) so that you do not have to rewrite them from scratch. After all, your computer only knows about pixels and raw mouse/touch events and does not understand the concept of a text box or a radio button by itself.[^2]

You also want your app to work on different aspect ratios and be easily extendable, so you try to avoid specifying as many absolute positions and sizes as possible while still maintaining a pixel-perfect and accessible layout. This is also what a GUI framework helps you with: it provides widgets like `Row`s, `Column`s, or `Center`s so you can simply declare all visible widgets and their arrangement on the screen, and the framework will automatically position and size most of them properly.

Additionally, high-level frameworks like Click provide powerful APIs to handle different types of user input (mouse, touch, stylus) in a single codebase and help you with other parts of your application logic like state management or establishing consistent color schemes.

Now, this was probably a lot of (new) information,  some of which was quite abstract (and significantly simplified), but learning UI and app development really is not that complicated. Just read some more chapters of this manual and look at the showcase projects under the `examples` folder, and you will quickly develop an intuition of what building GUIs in Click is all about.

[^1]: Note that HTML already kind of defines its own GUI framework, so you don't necessarily have to use another one for web development, although basically all bigger web projects use some web GUI framework like React or Angular.

[^2]: There are obviously different levels to what a computer "knows". While on the lowest level the transistors only know about Boolean logic, most operating systems do indeed have a concept of widgets like text boxes or buttons themselves - they basically include a default GUI framework for that operating system in their own code. There are different types of GUI frameworks, including _native_ ones that make use of these OS-provided widgets and _non-native_ frameworks like Click that implement all widgets again from scratch. Both approaches have advantages and downsides.

## Core Principles
> [!NOTE]  
> This chapter is fairly technical and might require some advanced coding/UI development knowledge to truly understand. It certainly won't hurt to read it though, if you're interested.

There are many different approaches to designing and building GUI frameworks; this section will outline the approach taken by Click and why this approach is suitable for crafting modern, portable user-interfaces in ASPL.

### Widgets
User interfaces in Click are fundamentally trees of [widgets](https://en.wikipedia.org/wiki/Graphical_widget). These include interactive ones like buttons but also layouting widgets like rows or visual-only widgets like images.

Widgets are implemented as classes. Complex interfaces are built using composition and inheritance.

### Optimized Immediate-Mode
Click is an immediate-mode GUI framework. This means that - disregarding caching and state management for a second - an app describes, constructs, and draws all currently visible widgets from scratch every single frame. Compared to retain-mode GUIs, for which the developer creates persistent widgets that store state internally and update on input etc., immediate-mode reduces the need for state synchronization and eliminates the complexity of deciding when to rebuild a widget. This is a huge deal and prevents countless bugs as well as the need for a lot of boilerplate code. However, as you probably think right now, naive immediate-mode frameworks are computationally much more expensive than retain-mode libraries. This is why Click uses a special state management mechanism to cache widgets between frames and only actually rebuild them whenever their state changes. This approach combines the best from both worlds (immediate-mode and retain-mode), leading to apps that look and feel like immediate-mode to the developer, including the elimination of state synchronization bugs, and still perform great under most conditions.

### Controlled State Management
To allow for efficient immediate-mode code and to prevent state synchronization issues, Click provides a standard interface for managing application state. Stateful values are wrapped in `State` instances, and widgets depending on them have to explicitly subscribe. Click can then periodically check which states have changed since the last frame and only rebuild the widgets that need to.

### FlexBox Pattern
Layouting in Click is fundamentally based on the [flexbox](https://en.wikipedia.org/wiki/Flexbox) design pattern:

All widgets can choose between three different so-called "sizings" for both their width and their height: fixed, tight, or loose. For the fixed sizing, a widget must declare its dimensions (along that axis) as soon as it is constructed. A tight widget will determine its size based on the space required by its children, while a loose widget may expand to fill the remaining available space along that axis. Some widgets might hardcode their sizing, while others might accept a sizing in their constructor.

This being said, the `FlexBox` widget provides a feature-rich implementation of an actual flexbox. It makes use of this sizing system together with the parameters `mainAxisAlignment` and `crossAxisAlignment` to automatically position and size both its children and itself. The `Row`, `Column`, and `SingleChildBox` widgets are simply thin wrappers around the `FlexBox`. Together, these widgets make up most of the layouting widgets used in Click. As a result, developers rarely need to specify explicit positions in their code and typically only set sizes for widgets with fixed sizing.

### Unified Pointer Interface
TODO

## Installing Click
You can install Click as a globally available ASPL module using the following command:
```
aspl install https://github.com/aspl-lang/click
```
Alternatively, you can manually clone this repository and make it available as a module using:
```
aspl install path/to/click/checkout
```
The latter allows you to place Click in any folder you wish, which is useful if you want to make changes to it or experiment heavily with the examples.

## Building your first App
Once you've installed Click, you can create your first app! Simply create a new directory somewhere, and inside that directory create two files, let's call them `main.aspl` and `Root.aspl`, with the following contents:
```aspl
import click

var window = new AppWindow("Hello Click!", 500, 500, new Root())
window.show()
```
```aspl
import graphics

import click.widgets
import click.layouting

class Root extends CompositeWidget {

    method build(BuildContext context) returns Widget{
        return new SingleChildBox(
            child = new Text(
                text = "Hello!", 
                font = Font:default.withSize(FontSize:px(50)),
                color = Color:fromRGB(235, 235, 235)
            ),
            color = Color:fromRGB(20, 20, 20),
            mainAxisAlignment = MainAxisAlignment.Center,
            crossAxisAlignment = CrossAxisAlignment.Center,
            sizing = Sizing:ll()
        )
    }

}
```
Now, compile your app like this:
```
aspl compile path/to/directory
```
And that's it! You should now have an executable that opens up an app with some "Hello!" text in the center.

> [!TIP]  
> In practice, you will want to build most apps using the ASPL C backend (`-backend c`) (as it is much faster than the AIL backend) and using the D3D11 graphics API on Windows (`-d3d11`).

> [!TIP]  
> Make sure to also check out the [showcase examples](examples) for several real-world UI cases. They naturally incorporate all concepts explained in this manual into high-quality, minimalistic apps.

## FlexBoxes, Rows, and Columns
Admittedly, the UI introduced above is not very useful. In practice, you will usually want to display multiple widgets on the screen at the same time. To achieve this, you can use the `Row` or `Column` widgets instead of the `SingleChildBox`:
```aspl
new Column(
    children = list<Widget>[
        new Text(
            text = "Hello!", 
            font = Font:default.withSize(FontSize:px(50)),
            color = Color:fromRGB(235, 235, 235)
        ),
        new Switch(
            active = true,
            width = 100
        )
    ],
    color = Color:fromRGB(20, 20, 20),
    mainAxisAlignment = MainAxisAlignment.Center,
    crossAxisAlignment = CrossAxisAlignment.Center,
    childGap = 50,
    sizing = Sizing:ll()
)
```
As you can see, the column arranges its (two) children vertically, placing one below the other. The `childGap` property ensures that the column leaves 50 pixels of space between them.

`Row`s, `Column`s and `SingleChildBox`es are all just specialized versions of the `FlexBox` widget. Click's layouting system is fundamentally based on the flexbox design pattern. As a result, you as the developer rarely need to specify explicit positions in your code and typically only define sizes for widgets with fixed sizing.

TODO: Document more features of the `FlexBox`.

## Sizings
TODO

## Images, Families and Virtual Coordinates
TODO

## State Management
TODO

## Unified Pointer Interface
TODO

## Troubleshooting
Even though Click aims at making UI and app development as easy and fun as possible, you will sooner or later run into issues and bugs. Most of these will likely originate from your own code, but since Click is still relatively young, there is a small chance of encountering a bug in the framework itself. If this happens, please open an issue so it can be fixed promptly.

Click supports both the C and the AIL backend of the ASPL programming language. It should also work on every platform supported by ASPL, including Windows, Linux, Android, and macOS, although the latter one is not actively tested. iOS support is planned but currently not a priority. If you encounter issues with your app, you can choose your backend and operating system freely for debugging.

TODO: Add more troubleshooting documentation.