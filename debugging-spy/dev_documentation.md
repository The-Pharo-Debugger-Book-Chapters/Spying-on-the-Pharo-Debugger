# Documentation for developers

## Overview

Debugging Spy is a tool with many objects interacting together. Let's classify Debugging Spy's components.

### Instrumentation

- `DSSpyInstrumenter` is responsible of the instrumentation of the system. You must use it to instrument your Pharo environment with class methods `instrumentSystem` and `stopInstrumentation`. 

- `DSSpy` is mostly defining rules about instrumentation: does the code written has to be recorded? and the clipboard content? class' names? ...

- `DSMetaLink` are specific MetaLinks for instrumentations done by Debugging Spy. It allows us to differentiate them from the classic MetaLinks.

### Recording

Many recording classes exist in Debugging Spy. Indeed, wanted data are different from an action to another.
So each recording class implement what data do we want to record and how they should be recorded.

Super class + main information 

Example with a window 

### Commands

For Debugger commands instrumentation, Debugging Spy use its own command system. 
It is very similar to recording classes, the difference is in the way of instrumenting (here by an extension in the Debugger).

### Storing and visualization

- `DSRecordRegistry` stores records.

- `DSRecordHistory` builds an history object more readable. It shorts records and infers information from them.

TODO: Give the available API for the `DSRecordHistory` => allowing to sort records according to some parameters 


