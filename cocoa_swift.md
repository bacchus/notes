<a id="home"></a>
# Cocoa-Swift
[Course](https://www.lynda.com/Swift-tutorials/Cocoa-Swift-3-Essential-Training/512722-2.html "lynda.com")

* [Intro](#Intro)
* [XCode](#XCode)
* [UI composer](#UI_composer)
* [Project struct](#Project_struct)
* [Swift syntax](#Swift_syntax)
* [Delegation & Protocols](#Delegation_Protocols)
* [Data ui](#Data_ui)


***

<a id="Intro"></a>
## Intro [home](#home)
- [Developing for Mac](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/OSX_Technology_Overview/About/About.html "Guide")
- developer.apple.com -> design -> human-interface-guidelines -> macos

#### Distibution
- developer.apple.com/programs/enroll
- menu -> product -> archive
- app permissions:  app -> capabilities -> app sandbox
- prefs -> accounts


<a id="XCode"></a>
## XCode [home](#home)
new proj -> macos -> cocoa app

#### toolbars
+ toolbar up-up right
    - 2 - asistant editor (side-by-side view) or alt-click
+ right up pane
+ two central tabs
    - 3 - class props
    - 4 - widget props (looks like slider)
    - 5 - layout props (looks like ruler)
    - 6 - actions (looks like arrow ->)
    - 7 - binding (looks like circle labyrinth)
+ right down pane
    - 1 - file templates lib
    - 2 - code templates lib
    - 3 - widgets
    - 4 - resources


***

<a id="UI_composer"></a>
## UI composer [home](#home)
info.plist -> main nib file name -> app main .xib file

on widget r-click-drag to AppDelegate:
- outlet as member
- action as callback 

if problems:
- check 6. actions pane 
- 'gray circles' near line num in delegate
- delegate cube on left pane

#### menu
drag to first responder upside cubes larger view  
also do same as with buttons

#### radiobutton grouping
- must be in same container
- connect to same callback (l-drag from grey circle)
- in 3. class props set id

#### label
multi text field

    label.stringValue = "Clicked!"

#### number formatters: date, number, money
dragg from widgets -> formatter to label

    label.objectValue = NSDate()

#### slider
in slider props -> control -> continuous  
to label direct value listener:  
label action pane -> received actions -> takeIntegerValue -> drag to slider

##### image button, image view
drag resources (ex: images) to Assets.xcassets  
imgBtn -> props -> image -> file name  
Appicon - add icon to app `icon_256x256@2x.png`


##### progrmaticaly create widget
    let label = NSTextField(labelWithString: "hello")
    label.frame = NSRect(x:100, y:100, width:frame.width, height:frame.height) // 0;0 - botom:left
    window.contentView?.adSubview(label)

    let button = NSButton(title: "title", target: self, aciton: #selector(exmpleAction))
    // --//--
    func exmpleAction() {
        print("pressed")
    }

#### layouts
- tab view
- table view
- outline view

grouping: select widgets -> menu editor -> embed in -> box or other  
autolayout: r-drag on parent or neiba -> see options  
see botom-r butons in ui editor  

#### toolbar
drag to window

#### storyobards
creates Main.storyobard  
code in NSViewControler  
add widgets to view-controler-scene


***

<a id="Project_struct"></a>
## Project struct [home](#home)
- NSApplication - event loop
- AppDelegate - app-level events
- NSWidnow - root view
- View - ui widgets

for new class from 3. widgets pane drag 'object' cube  
to cubes area under delegate cube  
int this obj 3. class props pane choose your class name  
after this steps you can r-click-drag outlets/actions to new class


***

<a id="Swift_syntax"></a>
## Swift syntax [home](#home)
- ; is not needed
- var - variable
- let - const

example:

    var window: NSWindow!

    NSException(name: NSExceptionName.illegalSelectorException
        , reason: "boo! not cool!", userInfo: nil).raise()

    let alert = NSAlert()
    alert.messageText = "hi!"
    //alert.runModal() or
    alert.addButtonWithTitle("btn1")
    alert.beginSheetModalForWindow(window) { (code: NSModalResponse) -> Void in
        if (code == NSAlertFirstButtonReturn) {
            print("btn1")
        }
    }

    func comboBoxSelectionDidChange(notification: NSNotification) {
        let combo: NSComboBox = notification.object as! NSComboBox
        print(combo.objectValueOfSelectedItem!)
        label.stringValue = "Clicked!"
    }

    let data:[String] = ["a", "b", "c"]

    // key-value coding
    class Book: NSObject {
        var tiile: String = "Examle"    
        var author: String = "Person"
        var pages: Int = 0;
    }

    let book: Book = Book()
    book.setValue("Jimmmy", forKey: "author")
    print(book.value(forKey: "author"))

    // add label -> in 7. bindings bind to delegate -> model key path -> self.book.titile


***

<a id="Delegation_Protocols"></a>
## Delegation & Protocols(interface) [home](#home)
NSApplicationDelegate   
- applicationDidFinishLaunching
- applicationWillTerminate
- applicationDidBecomeActive

NSComboBoxDelegate
- to handle comboBoxSelectionDidChange

#### custom delegate
    protocol CustomDelegate {
        func customDelegateExample()
    }

    class OtherObject: NSObject, CustomDelegate {
        func customDelegateExample() {
            print("func")
        }
    }

    // inside AppDelegate declare
    var other: OtherObject!
    var myDelegate: CustomDelegate!

    // and instantiate
    other = OtherObject()
    myDelegate = other

    // and use
    myDelegate.customDelegateExample()


***

<a id="Data_ui"></a>
## Data ui [home](#home)
table view, from actions grag to boxes delegate  
NSTableViewDataSource
- numberOfRows
- tableView(objectValueFor tableColumn:)
- for columns set each id

code:

    if (tableColumn?.identifier == "letter")
        return leters[row]
