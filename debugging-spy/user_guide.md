# User Guide

This chapter provides an user guide for anyone who want to use Debugging Spy.

## Installation

You can import Debugging Spy in a Pharo image by running this code in a Playground:

```Smalltalk
Metacello new
    baseline: 'DebuggingSpy';
    repository: 'github://Pharo-XP-Tools/DebuggingSpy:P14';
    load.
```

Caution: the example above loads the P14 version and you should adapt the code according to the version desired. 

## User interface

### Open the browser

Debugging Spy comes with a dedicated UI: the Debugging Record Browser. This UI can be opened by using the button on the world menu (at the top right of the IDE).

![Debugging Spy user interface](./graphics/browser_interface.png)

The following actions can be done by using the interface: 
- add a recording file (or a folder) in the browser
- start a new recording session
- stop the current recording session
- filter displayed records

### Starting and stopping the instrumentation

The instrumentation can be started and stopped by clicking on the associated buttons in the browser's toolbar.

When the instrumentation is started, a timer window is instantiated in the bottom-right corner of the screen. This timer window displays the last event in the experiment, the elapsed time since the start of the experiment, the current time and provides a button to stop the instrumentation.

![Timer window](./graphics/timer_window.png)

### Adding files into the browser

Events recorded are stored in a *.ston* file locally - on the user's computer - and can be found in the ds-spy directory of the image.

To add new recording files in the browser, you have to click on the **Add** button in the toolbar. A dialog will be opened, allowing to select some record files. 

The selected files will be added in the list and displayed on the right part of the screen.

![Displayed records in user interface](./graphics/displaying_records.png)

As seeing on the previous screenshot, the elements displayed in the user interface's table are colored. Color indicates the window where the element happened. Any element in the top right table could be selected to show in the next table the events it contains.
In the *Activity* tab, each element is a Window Jump (a list of events that happened in the same window before switching to another).
In the *Windows* tab, each element is a Window, the events displayed in the records table would be the list of all the events that happened in the selected window.


### Visualizing a file's records and history

After selecting a file in the list, you can visualize the corresponding data by doing : 
- `CMD + R` for the raw records.
- `CMD + H` for the associated history.

More information on the history object in the 'Advanced feature' section. 

### Filtering displayed records

The records are displayed using their class name, which could also be used to filter the type of records we would like to see (or not). The filter window could be opened by clicking on the **Filter** toolbar's button. Then, any class that is selected to be filtered (has been moved to the filter's right side) would **not** be displayed in the browser.

![Filter window](./graphics/filtering_records.png)

### Record anonymization

The Debugging Spy API permits to anonymize the records since their data might be sensible.
To make a record anonymous, use the **anonymize** method:
```Smalltalk
aRecord anonymize
``` 

This method will return a copy of the specified record that will be filtered based on a filter that is defined in the record's class in the **anonymousFilter** method. This filter defines the slots that are going to be kept in the copy, every slot that is not defined in that filter will be set at nil.

## Advanced features

### History 

Upon inspection, the history looks like this:

![History inspection](./graphics/history_inspection.png)

The history object exposes data organized in different perspectives :

- `records`

List of raw records ordered by datetime.  

- `windows` and `filteredWindows`

List of windows opened. Each window contains: 
 - its own list of events
 - a list of active periods (periods of activity in the window with associated events) 
 - a toolInfo object (describing the tool associated to the window)

The filtered version do not keep unknown windows.

- `windowJumps` and `filteredWindowJumps`

The sequential list of activity per window. This allows us to track activity within each window until a switch occurs, showing which window the user jumps to, what they do there, and when they return. Each window jump includes a start event (*startEvent*), an end event (*stopEvent*), a collection of events (*events*) recorded from entry to exit of the window, and the window linked to the activity (*window*, see the previous point). Each window jump corresponds to an activity period from the previous point.  

The filtered version do not keep activities with : a duration of less than 0.5 seconds, less than 3 records or an unknown window.

Some windows may have unusual names, such as:  

- **external window** → a window that was already open before measurements began (typically, in our data, this is the window displaying instructions).  

- **Weird titles like "Color: a color window"** → this is an application window, typically the program being debugged, rather than a tool window.

- Windows that correspond to the opening of a debugger, and only those, have a *source event* indicating which event triggered the window's opening. This applies only at the *window* level, not at the *jump* level. A jump is triggered by a mouse movement from one window to another. To determine the event that triggered the opening of the window being jumped to, one must use *"window sourceEvent"* from the jump.

- The activity records, also referred to as *jumps* or *basic blocks* depending on the context, now respond to *windowId*. This information indicates that the activity was performed in a window of the same id. This is a lazy accessor.

`DSRecordHistory` also provides an API to sort your records: 

- `#absoluteTimeTaken`, returns the absolute time taken to perform the recording of user events, including unmonitoring activities (interruptions or activities outside the IDE).
- `#countDebugActions`, returns how much debug actions have been done by the user (add or remove debugPoint, executing code, steps in debugger and methods created, modified or removed).
- `#timeTaken`, returns the time taken to perform the recording of user events. It is calculated as:
	- last log minus the first log timestamp minus time gaps (or discrepancies)
	- time gaps are calculated as the sum of time differences between two following events with a time delta > 5 min.
	We consider that, if the user did not do anything (basically typing or moving the mouse) for more than 5 min, the she was away from the task.

### List of recorded events

Here is the list of events that Debugging Spy is recording for now:

- Browsing actions: 'Browse', 'Senders' and 'Implementors'.

- Clipboard actions: 'Copy' and 'Paste'.

- Code interactions: 'Debug it', 'Do it', 'Do it and go', 'Inspect it' and 'Print it'.

- Debugger commands (all): 'Step into', 'Step over', 'Through', ...

- Exceptions raised and not caught.

- Halts (all): 'Halt', 'Halt once', 'Halt if', ...

- Mouse events: 'Mouse enter' and 'Mouse leave'. When the mouse is entering or leaving a window.

We can classify them in a table:

| **Type of traces**         | **User activity/block event or action** | **Debugging action** | **Navigation/inspection action** | **Debugging event** | **Code edition action** |
|-----------------------------|------------------------------------------|-----------------------|-----------------------------------|---------------------|--------------------------|
| DebugPoint                 |                                          | x                     |                                   |                     |                          |
| Watch DebugPoint           |                                          | x                     |                                   |                     |                          |
| Halt change                |                                          | x                     |                                   |                     |                          |
| Halt hit                   |                                          |                       |                                   | x                   |                          |
| Clipboard copy             |                                          |                       |                                   |                     | x                        |
| Clipboard paste            |                                          |                       |                                   |                     | x                        |
| Debug it                   | x                                        |                       |                                   |                     |                          |
| Do it                      | ?                                        |                       |                                   |                     |                          |
| Do it and go               | ?                                        |                       |                                   |                     |                          |
| Print it                   |                                          |                       | x                                 |                     |                          |
| Browse                     |                                          |                       | x                                 |                     |                          |
| Implementors               | x                                        |                       |                                   |                     |                          |
| Senders                    | x                                        |                       |                                   |                     |                          |
| Inspect                    | x                                        |                       |                                   |                     |                          |
| Logging error              |                                          |                       |                                   | x                   |                          |
| Method added               |                                          |                       |                                   |                     | x                        |
| Method modified            |                                          |                       |                                   |                     | x                        |
| Method removed             |                                          |                       |                                   |                     | x                        |
| Source code change         |                                          |                       |                                   |                     | x                        |
| Mouse enter window         | x                                        |                       |                                   |                     |                          |
| Mouse leave window         | x                                        |                       |                                   |                     |                          |
| Step                       |                                          | x                     |                                   |                     |                          |
| Window activated           | x                                        |                       |                                   |                     |                          |
| Window opened              | x                                        |                       |                                   |                     |                          |
| Window closed              | x                                        |                       |                                   |                     |                          |
| Proceed command            |                                          | x                     |                                   |                     |                          |
| Restart command            |                                          | x                     |                                   |                     |                          |
| Return value command       |                                          | x                     |                                   |                     |                          |
| Run to selection command   |                                          | x                     |                                   |                     |                          |
| Step into                  |                                          | x                     |                                   |                     |                          |
| Step over                  |                                          | x                     |                                   |                     |                          |
| Step through               |                                          | x                     |                                   |                     |                          |