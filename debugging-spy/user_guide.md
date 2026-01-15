# User Guide

## Installation

You can import Debugging Spy in your pharo image by running this code in a Playground:

```Smalltalk
Metacello new
    baseline: 'DebuggingSpy';
    repository: 'github://StevenCostiou/DebuggingSpy:P12';
    load.
```

## Instrumentation of the system

### Log data in your computer

In order to start the instrumentation of your Pharo IDE, you must checkout the branch corresponding to your Pharo's version and then load the baseline.

When it's done, run the following line to instrument the system:
```Smalltalk
DSSpyInstrumenter instrumentSystem
```

After that, the system starts logging.
Logs are serialized in the image working directory, in the *ds-spy* folder. You can access it with 'System' -> 'File Browser' then in the 'Bookmarks' section click on 'Working directory'.
Data files direct.


### Log data to a remote server

The first step are the same but in addition you need to configure the information about the server on which you want to log your data. 

TODO (when the code will be ready)

### Stop instrumentation 

When you want to stop the instrumentation, you can run the following code:

```Smalltalk
DSSpyInstrumenter stopInstrumentation
```

The system will stops logging and all instrumentation will be removed. 
If you want to re-instrument your system, you can go back to the point before.

## Read logged data

### Materialize raw logs

First, you need to get the reference to your log file: it should be in *ds-spy* from your working directory. Then execute the following code:

```Smalltalk
raw := DSSpy materialize: 'ds-spy/file_with_records' asFileReference
```
where `file_with_records` is your log file.

Upon inspection, you obtain a raw list of event, chronologically sorted:
<img width="695" alt="Capture d’écran 2025-03-09 à 22 50 33" src="https://github.com/user-attachments/assets/5c52dfeb-4f9c-4c61-bbb9-1112d4323157" />

### Build event history

DebuggingSpy provides a history object with an API to explore what happened during logging.
The history is obtained by executing:

```Smalltalk
history := DSRecordHistory on: raw
```
Upon inspection, the history looks like this:

![Capture d’écran 2025-03-10 à 14 08 43](https://github.com/user-attachments/assets/7b4464a5-7f5e-4b67-a6e5-3079cad98fcf)

The history object exposes data organized in different perspectives:

- **records** → the sequential list of logged events.  

- **windows** → the complete list of open windows. Each window contains its own list of events, a list of events grouped by active periods (*activePeriods*, i.e., each period marks an interruption in the window's activity), and the source event (*sourceEvent*) that triggered the window's opening. *(Note: this information is difficult to retrieve automatically and requires manual interpretation to be useful.)*  

- **windowJumps** → the sequential list of activity per window. This allows us to track activity within each window until a switch occurs, showing which window the user jumps to, what they do there, and when they return. Each window jump includes a start event (*startEvent*), an end event (*stopEvent*), a collection of events (*events*) recorded from entry to exit of the window, and the window linked to the activity (*window*, see the previous point). Each window jump corresponds to an activity period from the previous point.  

Some windows may have unusual names, such as:  

- **external window** → a window that was already open before measurements began (typically, in our data, this is the window displaying instructions).  

- **Weird titles like "Color: a color window"** → this is an application window, typically the program being debugged, rather than a tool window.

- Windows that correspond to the opening of a debugger, and only those, have a *source event* indicating which event triggered the window's opening. This applies only at the *window* level, not at the *jump* level. A jump is triggered by a mouse movement from one window to another. To determine the event that triggered the opening of the window being jumped to, one must use *"window sourceEvent"* from the jump.

- The activity records, also referred to as *jumps* or *basic blocks* depending on the context, now respond to *windowId*. This information indicates that the activity was performed in a window of the same id. This is a lazy accessor.

The history object exposes an API to explore the logged execution: (TODO: the API should be documented)

![Capture d’écran 2025-03-10 à 14 37 33](https://github.com/user-attachments/assets/95592964-d3c8-4bae-92d0-5ee2aa82f6b1)


## Quotation

To cite the use of this tool, please use: https://hal.science/hal-04858378v1

```bib
@softwareversion{costiou:hal-04858378v1,
  TITLE = {{Debugging Spy}},
  AUTHOR = {Costiou, Steven and Van{\`e}gue, Adrien},
  URL = {https://inria.hal.science/hal-04858378},
  NOTE = {},
  INSTITUTION = {{Centre Inria de l'Universit{\'e} de Lille}},
  YEAR = {2024},
  MONTH = Dec,
  SWHID = {swh:1:dir:0f63d67301c0ad3174d17c89a13b52e595837877;origin=https://github.com/Pharo-XP-Tools/DebuggingSpy;visit=swh:1:snp:ae7703ee9ee6f77eb10697b7d9fdf90678a768a0;anchor=swh:1:rev:fbec9a14cbdaa478f498196c0f993062441ef3ae},
  VERSION = {1.0},
  REPOSITORY = {https://github.com/Pharo-XP-Tools/DebuggingSpy},
  LICENSE = {MIT License},
  KEYWORDS = {Debug ; Software instrumentation},
  HAL_ID = {hal-04858378},
  HAL_VERSION = {v1},
}
```

## List of recorded events

- Browsing actions: 'Browse', 'Senders' and 'Implementors'.

- Clipboard actions: 'Copy' and 'Paste'.

- Code interactions: 'Debug it', 'Do it', 'Do it and go', 'Inspect it' and 'Print it'.

- Debugger commands (all): 'Step into', 'Step over', 'Through', ...

- Exceptions raised and not caught.

- Halts (all): 'Halt', 'Halt once', 'Halt if', ...

- Mouse events: 'Mouse enter' and 'Mouse leave'. When the mouse is entering or leaving a window.


| **Type of traces**         | **User activity/block event or action** | **Debugging action** | **Navigation/inspection action** | **Debugging event** | **Code edition action** |
|-----------------------------|------------------------------------------|-----------------------|-----------------------------------|---------------------|--------------------------|
| Breakpoint                 |                                          | x                     |                                   |                     |                          |
| Variable breakpoint        |                                          | x                     |                                   |                     |                          |
| Halt change                |                                          | x                     |                                   |                     |                          |
| Halt hit                   |                                          |                       |                                   | x                   |                          |
| Clipboard copy             |                                          |                       |                                   |                     | x                        |
| Clipboard paste            |                                          |                       |                                   |                     | x                        |
| Debug it                   | x                                        |                       |                                   |                     |                          |
| Do it                      | ?                                        |                       |                                   |                     |                          |
| Do it and go               | ?                                        |                       |                                   |                     |                          |
| Print it                   |                                          |                       | x                                 |                     |                          |
| Browse context             |                                          |                       | x                                 |                     |                          |
| Debugger action            |                                          | x                     |                                   |                     |                          |
| Debugger opening           | x                                        |                       |                                   |                     |                          |
| Full browse                |                                          |                       | x                                 |                     |                          |
| Inspect attribute          | x                                        |                       |                                   |                     |                          |
| Expand attribute           |                                          |                       | x                                 |                     |                          |
| Navigate attribute         |                                          |                       | x                                 |                     |                          |
| Inspect method             | x                                        |                       |                                   |                     |                          |
| Inspect method source      | x                                        |                       |                                   |                     |                          |
| Inspect object             | x                                        |                       |                                   |                     |                          |
| Select inspector page      |                                          |                       | x                                 |                     |                          |
| Logging error              |                                          |                       |                                   | x                   |                          |
| Method added               |                                          |                       |                                   |                     | x                        |
| Method modified            |                                          |                       |                                   |                     | x                        |
| Method removed             |                                          |                       |                                   |                     | x                        |
| Source code change         |                                          |                       |                                   |                     | x                        |
| Mouse down table           |                                          |                       | x                                 |                     |                          |
| Mouse enter table          |                                          |                       | x                                 |                     |                          |
| Mouse enter text editor    |                                          |                       | x                                 |                     |                          |
| Mouse enter window         | x                                        |                       |                                   |                     |                          |
| Mouse leave window         | x                                        |                       |                                   |                     |                          |
| Playground opened          | x                                        |                       |                                   |                     |                          |
| Playground read            |                                          |                       | x                                 |                     |                          |
| Playground write           |                                          |                       |                                   |                     | x                        |
| Query browse               |                                          |                       | x                                 |                     |                          |
| Start scrolling            |                                          |                       | x                                 |                     |                          |
| Stop scrolling             |                                          |                       | x                                 |                     |                          |
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







### Build visualizations
TODO