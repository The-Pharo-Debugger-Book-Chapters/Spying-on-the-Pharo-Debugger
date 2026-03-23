# Documentation for developers

This part of the book is for developers and details the different components of Debugging Spy.

## Overview

Debugging Spy is a tool with many objects interacting together. Let's classify Debugging Spy's components.

### Instrumentation

#### `DSSpyInstrumenter` 

This class is responsible of the instrumentation of the system. You must use it to instrument your Pharo environment with class methods `instrumentSystem` and `stopInstrumentation`. 

Methods are organized in different categories: 

- `instrumentSomething` methods instruments a specific action, such as "Debug it", by using Metalinks.

```smalltalk 
instrumentDebugIt
	"Instruments different debugIt methods"

	RubSmalltalkEditor link: DSDebugItRecord link toAST: (RubSmalltalkEditor >> #debugIt) ast.
	SpCodeDebugItCommand link: DSDebugItRecord link toAST: (SpCodeDebugItCommand >> #execute) ast.
```

- `logSomething` groups some `instrumentSomething` methods by categories, for example clipboard actions which include "Paste" and "Copy" actions.

```smalltalk 
logClipboardActions
	"Instruments commands linked to clipboard"

	self instrumentCopySelection.
	self instrumentPaste
```

- `listenToSomething` listens to announcements linked to a group of events, for example `listenToWindowEvents` is listening for "window opened", "window closed" and "window activated" events.

```smalltalk
listenToWindowEvents
	"Listen to the announcements link to windows and record the event"

	self currentWorld announcer when: WindowOpened send: #logWindowOpened: to: DSSpy.
	self currentWorld announcer when: WindowClosed send: #logWindowClosed: to: DSSpy.
	self currentWorld announcer when: WindowActivated send: #logWindowActivated: to: DSSpy
```

Note that the `instrumentSystem` method calls every instrumentation methods for now, but one future goal is to allow users to choose what should be recorded or not. 

```smalltalk

instrumentSystem
	"Instrument every thing in the system, feel free to comment things you don't want to record"

	"Start debugging session"
	DSSpy recordingSession: true. 
	
	"Listen to announcements"
	self listenToWindowEvents.
	self listenToMethodChanges.
	self listenToDebugPointChanges. 
	
	"Instruments events/user's actions"
	self logClipboardActions.
	self logMouseEvents.
	self logCodeInteractions.
	self logBrowsingActions.
	self logDebuggerActions.
	
	"Intruments exceptions"
	self instrumentExceptionSignalling

```

#### `DSSpy` 

It is mostly defining rules about instrumentation: does the code written has to be recorded? and the clipboard content? class' names? ... This class is also providing helper's methods for other classes in the package.

The provided API of `DSSpy class` is:

- `#recordClipboardContent` indicates if the clipboard's content should be recorded or not.
- `#recordSourceCode` indicates if the source code selected, when doing actions like "Do It", should be recorded.
- `#recordingSession` indicates if you are currently in a recording session.

#### `DSMetaLink` 

These are specific MetaLinks for instrumentations done by Debugging Spy. It allows us to differentiate them from the classic MetaLinks.

`DSMetaLink` and `DSMetaLinkInstaller` just inherits from `MetaLink` and `MetaLinkInstaller` respectively. The only difference is that `DSMetaLinkInstaller` reinstalls links when an instrumented method is modified (so the instrumentation is not deleted).

#### Commands

For Debugger commands instrumentation, Debugging Spy use its own command system. 
It is very similar to recording classes, the difference is in the way of instrumenting (here by an extension in the Debugger).

Likewise, we replace Sindarin's commands by commands from Debugging Spy which allow us to record every user's actions with Sindarin.

TODO: put pictures of code? Not good for clarity (too many methods called)

#### Extensions

Some instrumentations are done by adding extensions to existing code.
For example, `Halt` class is instrumented with:

```smalltalk
signal

	DSSpy recordingSession ifTrue: [
			[
				| sender |
				sender := thisContext sender sender sender sender sender.
				DSHaltHitRecord for: {
						sender sourceNodeExecuted.
						#halt } ]
				on: Error
				do: [ :err | DSSpy log: #ERROR key: #HALT_HIT ] ].
	super signal
```

As you can see, we used `DSSpy class>>#recordingSession` in order to unsure that we are in a recording session before recording.

This way of recording actions avoids issues due to MetaLink or Announcements and seems to be the best way of recording when it is possible to use it.

### Recording

Many recording classes exist in Debugging Spy. Indeed, wanted data are different from an action to another.
So each recording class implement what data do we want to record and how they should be recorded.

All records inherit from `DSAbstractEventRecord` and are then classify by abstract classes. 
For example, we have `DSClipboardCopyRecord` (recording the action of copying) which inherits from `DSClipboardActionRecord` (the recording category) and then this super class inherits from `DSAbstractEventRecord`.

Here is an example of the hierarchy view in the browser : TODO add screenshots or schema

The main methods in recording class are: 

- `DSAbstractEventRecord class >> for:` which is the entry point for recording (used for instrumentation)
- `DSAbstractEventRecord >> record: anObject` which is re-defined by its subclass and specified what is recorded

### Storing and visualization

#### `DSRecordRegistry` 

This object stores records during a recording session.

#### `DSSTONFileLogger` 

It logs records as STON file. 

#### `DSRecordHistory`

It builds an history object more readable from a STON file. It shorts records and infers information from them.

`DSRecordHistory` provides an API to sort your records: (maybe to move in the user guide)

- `#absoluteTimeTaken`, returns the absolute time taken to perform the recording of user events, including unmonitoring activities (interruptions or activities outside the IDE).
- `#countDebugActions`, returns how much debug actions have been done by the user (add or remove debugPoint, executing code, steps in debugger and methods created, modified or removed).
- `#timeTaken`, returns the time taken to perform the recording of user events. It is calculated as:
	- last log minus the first log timestamp minus time gaps (or discrepancies)
	- time gaps are calculated as the sum of time differences between two following events with a time delta > 5 min.
	We consider that, if the user did not do anything (basically typing or moving the mouse) for more than 5 min, the she was away from the task.

## Record data filtering

Debugging Spy API defines a class `DSRecordDataFilter` which permits to instanciate filters that will be used to remove and transform some information in the records. A filter will define operations that will be applied on a record's slots.

The filters could keep the value of a slot when applied, which needs to be specified using the **with** method. 

The filters also permit to transform their data in order to keep a certain slot's value but to modify it. In that case, it uses the **transform:with:** method that takes in argument a bloc that will be used to modify the data.

The API also provides a **hash:** method that is a transformation that will set a slot's value as its identity hash. That method is used in particular in the Debugging Spy Anonymizer that permits to make the data fully anonymous using this filters.

In fact, every slot that is not used for any operation will be set at nil after applying the filter.

After defining a filter, it might be used on a record with the **applyOn:** method that will return a **deepCopy** of the record that has been filtered.

Here is some example of record data filtering: 
```Smalltalk
| filter filteredRecord |

filter := DSRecordDataFilter new
	transform: #uuid with: [ :uuid | uuid asString ];
	hash: #dateTime;
	with: #windowId;
	yourself.

filteredRecord := includeFilter applyOn: aRecord. 
```

## Logging 

Our tool uses a STON logger that is defined in the `DSSTONFileLogger` class. This class also defines the method `#defaultLoggingDirectoryName` that returns the name of the directory where the record files will be logged when the system is instrumented (which is *ds-spy* by default).

## Testing

Most of the code is tested. To ensure that, we defined a testing strategy divided in two main categories: test scenarios and classical tests. 

### Common tests

As in all projects, you will find unit tests and functional tests. 

### Testing scenarios

Testing scenarios were defined to ensure that we correctly record scenarios which often happen during experiments. 

These scenarios are describing a succession of actions and the expected results. Our goal is then to match the result. 