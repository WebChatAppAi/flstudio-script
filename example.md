# FL Studio Piano Roll Scripting Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Script Structure](#script-structure)
3. [Core Classes](#core-classes)
   - [Note](#note)
   - [Score](#score)
   - [Marker](#marker)
   - [ScriptDialog](#scriptdialog)
   - [Utils](#utils)
4. [Working with Notes](#working-with-notes)
5. [Working with Markers](#working-with-markers)
6. [Creating User Interfaces](#creating-user-interfaces)
7. [Musical Functions](#musical-functions)
8. [Randomization and Humanization](#randomization-and-humanization)
9. [Pattern Generation](#pattern-generation)
10. [Best Practices](#best-practices)
11. [Example Scripts](#example-scripts)

## Introduction

FL Studio Piano Roll scripts are written in Python and allow you to create powerful tools for manipulating MIDI data. The API is contained in the `flpianoroll` package, which provides access to notes, markers, and user interface elements.

```python
import flpianoroll as flp
```

## Script Structure

Every FL Studio Piano Roll script follows a basic structure with two main functions:

```python
import flpianoroll as flp

def createDialog():
    # Create and configure a dialog
    form = flp.ScriptDialog('My Script', 'Description of what my script does')
    form.AddInputKnob('Parameter', 0.5, 0, 1)
    return form

def apply(form):
    # Get values from the dialog
    parameter = form.GetInputValue('Parameter')
    
    # Manipulate notes
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        # Do something with note
        
    # Add new notes if needed
    new_note = flp.Note()
    new_note.number = 60  # Middle C
    new_note.time = 0
    new_note.length = flp.score.PPQ  # Quarter note
    flp.score.addNote(new_note)
```

## Core Classes

### Note

The `Note` class represents a single MIDI note in the Piano Roll.

#### Properties

| Property  | Type          | Description                                       | Range               |
|-----------|---------------|---------------------------------------------------|---------------------|
| number    | int           | MIDI note number (pitch)                          | 0-131               |
| time      | int           | Start time in ticks                               | ≥ 0                 |
| length    | int           | Duration in ticks                                 | ≥ 0                 |
| velocity  | float         | Note velocity                                     | 0.0-1.0 (default 0.8) |
| pan       | float         | Note panning                                      | 0.0-1.0 (default 0.5) |
| release   | float         | Release time                                      | 0.0-1.0             |
| color     | int           | Color group / MIDI channel                        | 0-15                |
| fcut      | float         | Filter cutoff / Mod X                             | 0.0-1.0 (default 0.5) |
| fres      | float         | Filter resonance / Mod Y                          | 0.0-1.0 (default 0.5) |
| pitchofs  | int           | Pitch offset in cents                             | -120 to 120         |
| slide     | bool          | Slide note flag                                   | True/False          |
| porta     | bool          | Portamento flag                                   | True/False          |
| muted     | bool          | Mute flag                                         | True/False          |
| selected  | bool          | Selection state                                   | True/False          |
| repeats   | int           | Note repeats                                      | 0-14                |
| group     | int           | Group index for note grouping                     | ≥ 0                 |

#### Methods

| Method    | Description                                                |
|-----------|------------------------------------------------------------|
| clone()   | Creates a copy of the note (not yet added to the score)    |

#### Examples

Creating a new note:

```python
# Create a new quarter note at middle C
note = flp.Note()
note.number = 60  # Middle C
note.time = 0
note.length = flp.score.PPQ  # Quarter note
note.velocity = 0.8
flp.score.addNote(note)
```

Cloning a note:

```python
# Clone a note and place it an octave higher
original_note = flp.score.getNote(0)
new_note = original_note.clone()
new_note.number += 12  # Move up an octave
new_note.time += flp.score.PPQ  # Place one beat later
flp.score.addNote(new_note)
```

### Score

The `Score` class represents the entire Piano Roll, containing notes and markers.

#### Properties

| Property   | Type   | Description                                            |
|------------|--------|--------------------------------------------------------|
| PPQ        | int    | Pulses Per Quarter note (ticks in one quarter note)    |
| tsnum      | int    | Time signature numerator (e.g., 4 in 4/4)              |
| tsden      | int    | Time signature denominator (e.g., 4 in 4/4)            |
| noteCount  | int    | Number of notes in the score                           |
| markerCount| int    | Number of markers in the score                         |

#### Methods

| Method                      | Description                                            |
|-----------------------------|--------------------------------------------------------|
| clear([all])                | Remove notes and markers. Pass True to clear all instead of just selected |
| clearNotes([all])           | Remove notes. Pass True to clear all instead of just selected |
| clearMarkers([all])         | Remove markers. Pass True to clear all instead of just selected |
| getTimelineSelection()      | Returns a tuple with start and end time of the selection |
| getDefaultNoteProperties()  | Returns a Note with the default properties             |
| getNextFreeGroupIndex()     | Returns the next available group index                 |
| addNote(note)               | Add a new note to the score                           |
| getNote(index)              | Get a note by index                                   |
| deleteNote(index)           | Delete a note by index                                |
| addMarker(marker)           | Add a new marker to the score                         |
| getMarker(index)            | Get a marker by index                                 |
| deleteMarker(index)         | Delete a marker by index                              |

#### Examples

Accessing score information:

```python
# Get time signature and PPQ
ppq = flp.score.PPQ
ts_num = flp.score.tsnum
ts_den = flp.score.tsden
print(f"Time signature: {ts_num}/{ts_den}, PPQ: {ppq}")

# Get timeline selection
selection = flp.score.getTimelineSelection()
if selection[1] != -1:  # Check if there is a selection
    start_time = selection[0]
    end_time = selection[1]
    print(f"Selection from tick {start_time} to {end_time}")
```

Working with notes:

```python
# Loop through all notes
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    
    # Increase velocity of selected notes
    if note.selected:
        note.velocity = min(1.0, note.velocity * 1.2)

# Clear selected notes
flp.score.clearNotes(False)  # False means only clear selected notes

# Clear all notes and markers
flp.score.clear(True)  # True means clear all, not just selected

# Create note groups
group_idx = flp.score.getNextFreeGroupIndex()
for i in range(flp.score.noteCount):
    if flp.score.getNote(i).selected:
        flp.score.getNote(i).group = group_idx
```

### Marker

The `Marker` class represents markers in the Piano Roll, such as time signatures or labels.

#### Properties

| Property  | Type    | Description                                     |
|-----------|---------|-------------------------------------------------|
| time      | int     | Marker position in ticks                       |
| name      | string  | Marker name/label                              |
| mode      | int     | Marker mode (0 for regular, 12 for key marker) |
| tsnum     | int     | Time signature numerator (for time sig markers)|
| tsden     | int     | Time signature denominator (for time sig markers)|
| scale_root| int     | Root note for key markers                      |
| scale_helper| string| Scale information for key markers              |

#### Examples

Creating a marker:

```python
# Create a regular marker
marker = flp.Marker()
marker.time = flp.score.PPQ * 4  # Position at bar 2
marker.name = "Verse"
marker.mode = 0  # Regular marker
flp.score.addMarker(marker)

# Create a time signature marker
ts_marker = flp.Marker()
ts_marker.time = 0
ts_marker.mode = 12  # Time signature mode
ts_marker.tsnum = 3
ts_marker.tsden = 4
ts_marker.name = "3/4 Time"
flp.score.addMarker(ts_marker)

# Create a key marker
key_marker = flp.Marker()
key_marker.time = 0
key_marker.mode = 12  # Key marker mode
key_marker.name = "C Minor"
key_marker.scale_root = 0  # C
# Scale helper format: comma-separated list of 1 (not in scale) or 0 (in scale)
# For C minor: C(0), C#(1), D(0), D#(1), E(0), F(0), F#(1), G(0), G#(1), A(0), A#(1), B(0)
key_marker.scale_helper = "0,1,0,1,0,0,1,0,1,0,1,0"
flp.score.addMarker(key_marker)
```

### ScriptDialog

The `ScriptDialog` class allows you to create user interfaces for your scripts.

#### Properties

| Property           | Type    | Description                                      |
|--------------------|---------|--------------------------------------------------|
| restoreFormValues  | bool    | Whether to save/restore values between runs      |

#### Methods

| Method                                      | Description                                            |
|---------------------------------------------|--------------------------------------------------------|
| ScriptDialog(title, description)            | Create a new dialog with title and description         |
| setText(description)                        | Change the dialog description                          |
| addInput(name, value, hint='')              | Add a generic input control                           |
| addInputKnob(name, value, min, max, hint='') | Add a knob with floating-point value                  |
| addInputKnobInt(name, value, min, max, hint='') | Add a knob with integer value                      |
| addInputCombo(name, valueList, value, hint='') | Add a dropdown with multiple options               |
| addInputCheckbox(name, value, hint='')       | Add a checkbox with boolean value                    |
| addInputText(name, value, hint='')           | Add a text input field                               |
| addInputSurface(presetName)                 | Add a Control Surface plugin preset                   |
| getInputValue(aName)                        | Get the current value of an input control            |
| execute()                                   | Show the dialog                                      |

#### Examples

Creating a dialog:

```python
def createDialog():
    form = flp.ScriptDialog("Note Generator", "Generate notes with custom parameters")
    
    # Add a dropdown for note selection
    form.AddInputCombo("Root Note", "C,C#,D,D#,E,F,F#,G,G#,A,A#,B", 0, "Select root note")
    
    # Add an integer knob for octave
    form.AddInputKnobInt("Octave", 4, 1, 8, "Select octave")
    
    # Add float knobs for parameters
    form.AddInputKnob("Velocity", 0.8, 0.1, 1.0, "Note velocity")
    form.AddInputKnob("Duration", 0.5, 0.1, 2.0, "Note duration as ratio of quarter note")
    
    # Add checkboxes for options
    form.AddInputCheckbox("Create Chord", True, "Create a chord instead of single note")
    form.AddInputCheckbox("Use Arpeggio", False, "Create arpeggio pattern")
    
    # Add a text input
    form.AddInputText("Pattern Name", "My Pattern", "Enter a name for this pattern")
    
    # Add a Control Surface preset (must be in same folder as script)
    form.AddInputSurface("MyControlSurface")
    
    return form

def apply(form):
    # Retrieve values from the form
    root_note = form.GetInputValue("Root Note")  # Returns index of selected item (0-11)
    octave = form.GetInputValue("Octave")  # Returns integer (1-8)
    velocity = form.GetInputValue("Velocity")  # Returns float (0.1-1.0)
    duration = form.GetInputValue("Duration")  # Returns float (0.1-2.0)
    create_chord = form.GetInputValue("Create Chord")  # Returns boolean
    use_arpeggio = form.GetInputValue("Use Arpeggio")  # Returns boolean
    pattern_name = form.GetInputValue("Pattern Name")  # Returns string
    
    # Use values to create notes
    note_number = (octave * 12) + root_note
    note_length = int(flp.score.PPQ * duration)
    
    # Create a note
    note = flp.Note()
    note.number = note_number
    note.time = 0
    note.length = note_length
    note.velocity = velocity
    flp.score.addNote(note)
```

### Utils

The `Utils` class provides utility functions for showing messages and logging.

#### Methods

| Method                         | Description                                           |
|--------------------------------|-------------------------------------------------------|
| ProgressMsg(msg, pos, total)   | Shows a progress message with position and total      |
| ShowMessage(msg)               | Shows a message in a dialog box                       |
| log(msg)                       | Writes a string to the FL Studio debug log tab        |

#### Examples

Using utilities:

```python
# Show a dialog message
flp.Utils.ShowMessage("Script completed successfully!")

# Log debug information
flp.Utils.log("Processing note with pitch: 60")

# Show progress message
total_notes = flp.score.noteCount
for i in range(total_notes):
    # Process notes
    flp.Utils.ProgressMsg(f"Processing note {i+1}", i, total_notes)
```

## Working with Notes

### Creating and Adding Notes

```python
# Create a new note
note = flp.Note()
note.number = 60  # Middle C
note.time = 0  # Start at beginning
note.length = flp.score.PPQ  # Quarter note duration
note.velocity = 0.8
note.color = 0  # Default color
flp.score.addNote(note)

# Create a series of notes (C major scale)
for i, pitch in enumerate([60, 62, 64, 65, 67, 69, 71, 72]):
    note = flp.Note()
    note.number = pitch
    note.time = i * flp.score.PPQ  # Each note one quarter later
    note.length = flp.score.PPQ // 2  # Eighth note duration
    note.velocity = 0.7 + (i * 0.03)  # Gradually increasing velocity
    flp.score.addNote(note)
```

### Modifying Existing Notes

```python
# Loop through all notes
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    
    # Transpose up by a perfect fifth
    note.number += 7
    
    # Shorten note duration by half
    note.length = note.length // 2
    
    # Set color based on pitch
    note.color = note.number % 16
    
    # Add slide to notes in a specific range
    if 60 <= note.number <= 72:
        note.slide = True
```

### Selecting and Filtering Notes

```python
# Select notes in a specific pitch range
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    note.selected = (60 <= note.number <= 72)

# Select notes with specific duration
min_length = flp.score.PPQ  # Quarter note
for i in range(flp.score.noteCount):
    if flp.score.getNote(i).length >= min_length:
        flp.score.getNote(i).selected = True

# Get only selected notes
selected_notes = []
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    if note.selected:
        selected_notes.append(note)
```

### Deleting Notes

```python
# Delete all notes
flp.score.clearNotes(True)

# Delete selected notes
flp.score.clearNotes(False)

# Delete notes by index (in reverse order to avoid index shifting)
for i in reversed(range(flp.score.noteCount)):
    note = flp.score.getNote(i)
    if note.number < 60:  # Delete all notes below middle C
        flp.score.deleteNote(i)
```

### Grouping Notes

```python
# Create a new group of selected notes
group_idx = flp.score.getNextFreeGroupIndex()
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    if note.selected:
        note.group = group_idx

# Identify notes in the same group
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    if note.group == group_idx:
        # Process notes in this group
        pass
```

## Working with Markers

### Creating and Adding Markers

```python
# Create a simple marker
marker = flp.Marker()
marker.time = flp.score.PPQ * 4  # Bar 2
marker.name = "Verse"
marker.mode = 0  # Regular marker
flp.score.addMarker(marker)

# Create a time signature marker
ts_marker = flp.Marker()
ts_marker.time = 0
ts_marker.name = "3/4"
ts_marker.mode = 12  # Time signature mode
ts_marker.tsnum = 3
ts_marker.tsden = 4
flp.score.addMarker(ts_marker)

# Create a key marker (C minor)
key_marker = flp.Marker()
key_marker.time = 0
key_marker.name = "C Minor"
key_marker.mode = 12  # Key marker mode
key_marker.scale_root = 0  # C
key_marker.scale_helper = "0,1,0,1,0,0,1,0,1,0,1,0"  # C minor scale
flp.score.addMarker(key_marker)
```

### Reading Markers

```python
# Loop through all markers
for i in range(flp.score.markerCount):
    marker = flp.score.getMarker(i)
    
    # Check if it's a time signature marker
    if marker.mode == 12 and hasattr(marker, 'tsnum') and hasattr(marker, 'tsden'):
        print(f"Time signature marker at tick {marker.time}: {marker.tsnum}/{marker.tsden}")
    
    # Check if it's a key marker
    elif marker.mode == 12 and hasattr(marker, 'scale_root'):
        note_names = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"]
        root_name = note_names[marker.scale_root]
        print(f"Key marker at tick {marker.time}: {root_name}")
    
    # Regular marker
    else:
        print(f"Marker at tick {marker.time}: {marker.name}")
```

### Detecting Scale from Key Markers

```python
# Scale interval definitions
scales_intervals = {
    "Major": [0, 2, 4, 5, 7, 9, 11],
    "Minor": [0, 2, 3, 5, 7, 8, 10],
    "Minor Harmonic": [0, 2, 3, 5, 7, 8, 11],
    "Minor Melodic": [0, 2, 3, 5, 7, 9, 11]
}

# Check for key markers in the score
detected_scale = None
detected_root = None

for m in range(flp.score.markerCount):
    marker = flp.score.getMarker(m)
    
    if marker.mode == 12:  # Key marker
        # Get scale information from scale_helper
        scale_helper = marker.scale_helper.split(",")
        scale_intervals = [i for i in range(len(scale_helper)) if scale_helper[i] == '0']
        
        # Normalize intervals relative to root
        root = marker.scale_root
        normalized_intervals = []
        for interval in scale_intervals:
            normalized = (interval - root) % 12
            normalized_intervals.append(normalized)
        normalized_intervals.sort()
        
        # Check if matches a known scale
        for scale_name, intervals in scales_intervals.items():
            if normalized_intervals == intervals:
                detected_scale = scale_name
                detected_root = root
                break
        
        # Break after first key marker
        if detected_scale:
            break

if detected_scale:
    note_names = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"]
    print(f"Detected scale: {note_names[detected_root]} {detected_scale}")
```

## Creating User Interfaces

### Basic Dialog Setup

```python
def createDialog():
    form = flp.ScriptDialog("My Script", "Description goes here")
    
    # Add controls
    form.AddInputKnob("Parameter 1", 0.5, 0, 1)
    form.AddInputKnobInt("Parameter 2", 5, 0, 10)
    
    return form

def apply(form):
    # Get values
    param1 = form.GetInputValue("Parameter 1")
    param2 = form.GetInputValue("Parameter 2")
    
    # Use the values in your script
    # ...
```

### Controls Reference

```python
def createDialog():
    form = flp.ScriptDialog("Controls Example", "Shows all types of controls")
    
    # Knob with floating-point value
    form.AddInputKnob("Float Knob", 0.5, 0, 1, "Hint text for this control")
    
    # Knob with integer value
    form.AddInputKnobInt("Int Knob", 5, 0, 10, "Interger values only")
    
    # Dropdown menu
    form.AddInputCombo("Dropdown", "Option 1,Option 2,Option 3", 0, "Select an option")
    
    # Checkbox
    form.AddInputCheckbox("Checkbox", True, "Enable this feature")
    
    # Text input
    form.AddInputText("Text Input", "Default text", "Enter custom text")
    
    # Advanced control surface (requires .fst file in same directory)
    form.AddInputSurface("MyControlSurface")
    
    return form
```

### Handling Complex UI Logic

```python
def createDialog():
    form = flp.ScriptDialog("Advanced UI", "Uses conditional UI logic")
    
    # Store the form for later reference
    global my_form
    my_form = form
    
    # Add a mode selector
    form.AddInputCombo("Mode", "Simple,Advanced,Expert", 0)
    
    # Add simple mode controls
    form.AddInputKnob("Volume", 0.8, 0, 1)
    
    # Add advanced mode controls (will toggle visibility based on mode)
    form.AddInputKnob("Attack", 0.2, 0, 1)
    form.AddInputKnob("Decay", 0.3, 0, 1)
    form.AddInputKnob("Sustain", 0.7, 0, 1)
    form.AddInputKnob("Release", 0.5, 0, 1)
    
    # Add expert mode controls
    form.AddInputKnob("Modulation", 0.0, 0, 1)
    form.AddInputKnob("Resonance", 0.5, 0, 1)
    
    # Set initial visibility
    updateControlVisibility(0)  # Start in simple mode
    
    return form

def updateControlVisibility(mode):
    # Simple mode - show only volume
    if mode == 0:
        # Use UI library to show/hide controls (note: this is conceptual, not actual API)
        my_form.setControlVisible("Attack", False)
        my_form.setControlVisible("Decay", False)
        my_form.setControlVisible("Sustain", False)
        my_form.setControlVisible("Release", False)
        my_form.setControlVisible("Modulation", False)
        my_form.setControlVisible("Resonance", False)
    
    # Advanced mode - show ADSR
    elif mode == 1:
        my_form.setControlVisible("Attack", True)
        my_form.setControlVisible("Decay", True)
        my_form.setControlVisible("Sustain", True)
        my_form.setControlVisible("Release", True)
        my_form.setControlVisible("Modulation", False)
        my_form.setControlVisible("Resonance", False)
    
    # Expert mode - show everything
    elif mode == 2:
        my_form.setControlVisible("Attack", True)
        my_form.setControlVisible("Decay", True)
        my_form.setControlVisible("Sustain", True)
        my_form.setControlVisible("Release", True)
        my_form.setControlVisible("Modulation", True)
        my_form.setControlVisible("Resonance", True)

def apply(form):
    # Get the selected mode
    mode = form.GetInputValue("Mode")
    
    # Update control visibility based on mode
    updateControlVisibility(mode)
    
    # Get control values based on mode
    volume = form.GetInputValue("Volume")
    
    if mode >= 1:  # Advanced or Expert mode
        attack = form.GetInputValue("Attack")
        decay = form.GetInputValue("Decay")
        sustain = form.GetInputValue("Sustain")
        release = form.GetInputValue("Release")
    
    if mode == 2:  # Expert mode
        modulation = form.GetInputValue("Modulation")
        resonance = form.GetInputValue("Resonance")
    
    # Apply settings based on mode
    # ...
```

## Musical Functions

### Scale and Chord Definitions

```python
# Define scales
SCALES = {
    "Major": [0, 2, 4, 5, 7, 9, 11],
    "Minor": [0, 2, 3, 5, 7, 8, 10],
    "Minor Harmonic": [0, 2, 3, 5, 7, 8, 11],
    "Minor Melodic": [0, 2, 3, 5, 7, 9, 11],
    "Dorian": [0, 2, 3, 5, 7, 9, 10],
    "Phrygian": [0, 1, 3, 5, 7, 8, 10],
    "Lydian": [0, 2, 4, 6, 7, 9, 11],
    "Mixolydian": [0, 2, 4, 5, 7, 9, 10],
    "Locrian": [0, 1, 3, 5, 6, 8, 10],
    "Blues": [0, 3, 5, 6, 7, 10],
    "Pentatonic Major": [0, 2, 4, 7, 9],
    "Pentatonic Minor": [0, 3, 5, 7, 10]
}

# Define chord types
CHORDS = {
    "Major": [0, 4, 7],
    "Minor": [0, 3, 7],
    "Diminished": [0, 3, 6],
    "Augmented": [0, 4, 8],
    "Sus2": [0, 2, 7],
    "Sus4": [0, 5, 7],
    "Major 7": [0, 4, 7, 11],
    "Dominant 7": [0, 4, 7, 10],
    "Minor 7": [0, 3, 7, 10],
    "Half-Diminished 7": [0, 3, 6, 10],
    "Diminished 7": [0, 3, 6, 9],
    "Minor-Major 7": [0, 3, 7, 11],
    "Major 9": [0, 4, 7, 11, 14],
    "Dominant 9": [0, 4, 7, 10, 14],
    "Minor 9": [0, 3, 7, 10, 14],
    "Major 11": [0, 4, 7, 11, 14, 17],
    "Dominant 11": [0, 4, 7, 10, 14, 17],
    "Minor 11": [0, 3, 7, 10, 14, 17]
}

# Note names
NOTE_NAMES = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"]
```

### Creating Scale Notes

```python
def create_scale_notes(root, scale_name, octave, note_count=8):
    """Create notes in a specific scale"""
    if scale_name not in SCALES:
        return []
    
    # Get scale intervals
    intervals = SCALES[scale_name]
    
    # Calculate base note
    base_note = (octave * 12) + root
    
    # Create notes
    notes = []
    for i in range(note_count):
        # Get scale degree (wrapping if needed)
        scale_idx = i % len(intervals)
        octave_offset = i // len(intervals)
        
        # Calculate note number
        note_num = base_note + intervals[scale_idx] + (octave_offset * 12)
        
        # Create note
        note = flp.Note()
        note.number = note_num
        note.time = i * flp.score.PPQ  # Quarter note spacing
        note.length = flp.score.PPQ // 2  # Eighth note duration
        note.velocity = 0.8
        
        notes.append(note)
    
    return notes
```

### Creating Chord Notes

```python
def create_chord(root, chord_type, octave, inversion=0):
    """Create notes for a specific chord"""
    if chord_type not in CHORDS:
        return []
    
    # Get chord intervals
    intervals = CHORDS[chord_type]
    
    # Calculate base note
    base_note = (octave * 12) + root
    
    # Create chord notes
    notes = []
    for i, interval in enumerate(intervals):
        # Apply inversion
        if i < inversion:
            note_num = base_note + interval + 12
        else:
            note_num = base_note + interval
        
        # Create note
        note = flp.Note()
        note.number = note_num
        note.time = 0  # All notes start at the same time
        note.length = flp.score.PPQ  # Quarter note duration
        note.velocity = 0.8
        
        notes.append(note)
    
    return notes
```

### Creating Chord Progressions

```python
def create_chord_progression(root, scale_name, octave, progression):
    """Create a chord progression based on scale degrees
    
    Args:
        root: Root note (0-11 for C through B)
        scale_name: Name of the scale
        octave: Base octave
        progression: List of scale degrees (1-7) to use for chords
    """
    if scale_name not in SCALES:
        return []
    
    # Get scale intervals
    scale_intervals = SCALES[scale_name]
    
    # Define chord qualities for each scale degree
    if scale_name == "Major":
        # Major scale chord qualities (I-ii-iii-IV-V-vi-vii°)
        chord_qualities = ["Major", "Minor", "Minor", "Major", "Major", "Minor", "Diminished"]
    elif scale_name == "Minor":
        # Natural minor chord qualities (i-ii°-III-iv-v-VI-VII)
        chord_qualities = ["Minor", "Diminished", "Major", "Minor", "Minor", "Major", "Major"]
    else:
        # Default to all major chords
        chord_qualities = ["Major"] * 7
    
    # Create notes for the progression
    all_notes = []
    for i, degree in enumerate(progression):
        # Convert 1-based degree to 0-based index
        degree_idx = degree - 1
        
        # Get root note for this chord
        chord_root = (root + scale_intervals[degree_idx]) % 12
        
        # Get chord quality for this degree
        chord_type = chord_qualities[degree_idx]
        
        # Create chord notes
        chord_notes = create_chord(chord_root, chord_type, octave)
        
        # Set time for each note based on position in progression
        for note in chord_notes:
            note.time = i * flp.score.PPQ * 4  # One chord per bar
            all_notes.append(note)
    
    return all_notes
```

### Creating Arpeggios

```python
def create_arpeggio(root, chord_type, octave, pattern, duration=None):
    """Create an arpeggio based on a chord
    
    Args:
        root: Root note (0-11 for C through B)
        chord_type: Type of chord to arpeggiate
        octave: Base octave
        pattern: Arpeggio pattern (e.g., "up", "down", "updown")
        duration: Duration for each note (defaults to 1/16 note)
    """
    if chord_type not in CHORDS:
        return []
    
    # Get chord intervals
    intervals = CHORDS[chord_type]
    
    # Calculate base note
    base_note = (octave * 12) + root
    
    # Create chord notes
    chord_notes = [base_note + interval for interval in intervals]
    
    # Set default duration to 1/16 note if not specified
    if duration is None:
        duration = flp.score.PPQ // 4
    
    # Create arpeggio notes
    arp_notes = []
    
    if pattern == "up":
        # Ascending pattern
        pattern_indices = list(range(len(chord_notes)))
    elif pattern == "down":
        # Descending pattern
        pattern_indices = list(reversed(range(len(chord_notes))))
    elif pattern == "updown":
        # Up and down pattern
        pattern_indices = list(range(len(chord_notes))) + list(reversed(range(1, len(chord_notes) - 1)))
    elif pattern == "random":
        # Random pattern
        pattern_indices = [random.randint(0, len(chord_notes) - 1) for _ in range(8)]
    else:
        # Default to up pattern
        pattern_indices = list(range(len(chord_notes)))
    
    # Create notes based on pattern
    for i, idx in enumerate(pattern_indices):
        note = flp.Note()
        note.number = chord_notes[idx]
        note.time = i * duration
        note.length = duration
        note.velocity = 0.8
        arp_notes.append(note)
    
    return arp_notes
```

## Randomization and Humanization

### Basic Randomization

```python
import random

def randomize_velocity(min_vel=0.6, max_vel=0.9):
    """Randomize velocity of all notes"""
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        note.velocity = random.uniform(min_vel, max_vel)

def randomize_timing(max_offset=10):
    """Randomize timing of notes slightly"""
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        offset = random.randint(-max_offset, max_offset)
        note.time = max(0, note.time + offset)

def randomize_length(factor=0.2):
    """Randomize note length by a percentage"""
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        variation = 1.0 + random.uniform(-factor, factor)
        note.length = max(1, int(note.length * variation))
```

### Advanced Humanization

```python
import time
import math

def rand_int(seed=None):
    """Generate a random integer with seed"""
    state = time.time_ns() if seed is None else seed
    while True:
        state = (1103515245 * state + 12345) & 0x7fffffff
        yield state

def rand_uniform(seed=None):
    """Generate a random float between 0 and 1"""
    gen = rand_int(seed)
    while True:
        yield next(gen) / 0x80000000

def rand_sin_recursive(n=3, seed=None):
    """Generate a quasi-normal distribution"""
    gen = rand_uniform(seed)
    while True:
        x = next(gen)
        for _ in range(n):
            x = math.acos(-2 * x + 1) / math.pi
        yield x

def humanize_notes(seed=None, time_range=0.1, vel_range=0.2, duration_range=0.1):
    """Apply realistic humanization to notes
    
    Args:
        seed: Random seed for reproducible results
        time_range: Max timing variation as ratio of quarter note
        vel_range: Max velocity variation as ratio
        duration_range: Max duration variation as ratio
    """
    # Initialize random generators
    rng_time = rand_sin_recursive(3, seed)
    rng_vel = rand_sin_recursive(3, seed + 1 if seed else None)
    rng_dur = rand_sin_recursive(3, seed + 2 if seed else None)
    
    # Calculate max offset in ticks
    max_time_offset = int(flp.score.PPQ * time_range)
    
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        
        # Vary timing
        time_offset = int(max_time_offset * (2 * next(rng_time) - 1))
        note.time = max(0, note.time + time_offset)
        
        # Vary velocity (weighted toward original)
        vel_offset = vel_range * (2 * next(rng_vel) - 1)
        note.velocity = max(0.1, min(1.0, note.velocity * (1 + vel_offset)))
        
        # Vary duration (weighted toward original)
        dur_offset = duration_range * (2 * next(rng_dur) - 1)
        note.length = max(1, int(note.length * (1 + dur_offset)))
```

### Velocity Curves

```python
def apply_velocity_curve(curve_type, intensity=1.0):
    """Apply a velocity curve to all notes
    
    Args:
        curve_type: "linear", "exponential", "logarithmic", "sine", "random"
        intensity: Strength of the effect (0.0-1.0)
    """
    if flp.score.noteCount == 0:
        return
    
    # Sort notes by time
    notes = [(i, flp.score.getNote(i)) for i in range(flp.score.noteCount)]
    notes.sort(key=lambda x: x[1].time)
    
    # Find time range
    start_time = notes[0][1].time
    end_time = max(n[1].time for n in notes)
    time_range = max(1, end_time - start_time)  # Prevent division by zero
    
    for idx, note in notes:
        # Get normalized position in time (0.0-1.0)
        pos = (note.time - start_time) / time_range
        
        # Apply curve based on type
        if curve_type == "linear":
            # Linear curve: y = x
            factor = pos
        
        elif curve_type == "exponential":
            # Exponential curve: y = x^2
            factor = pos ** 2
        
        elif curve_type == "logarithmic":
            # Logarithmic curve: y = log(1 + 9x)/log(10)
            factor = math.log(1 + 9 * pos) / math.log(10)
        
        elif curve_type == "sine":
            # Sine curve: y = (sin((x-0.5)*pi) + 1)/2
            factor = (math.sin((pos - 0.5) * math.pi) + 1) / 2
        
        elif curve_type == "random":
            # Random curve
            factor = random.random()
        
        else:
            factor = pos  # Default to linear
        
        # Apply velocity change with intensity control
        base_velocity = 0.5  # Middle velocity
        velocity_range = 0.5  # Max deviation from base
        
        # Calculate new velocity
        new_velocity = base_velocity + (factor * 2 - 1) * velocity_range * intensity
        
        # Ensure velocity is within valid range
        new_velocity = max(0.1, min(1.0, new_velocity))
        
        # Apply to note
        flp.score.getNote(idx).velocity = new_velocity
```

## Pattern Generation

### Euclidean Rhythm Generator

```python
def euclidean_rhythm(steps, pulses, offset=0):
    """Generate an Euclidean rhythm
    
    Args:
        steps: Total number of steps in the pattern
        pulses: Number of onsets (notes) to distribute
        offset: Rotation offset
    
    Returns:
        List of boolean values where True means a note
    """
    if steps < 1 or pulses < 1:
        return []
    
    # Ensure pulses doesn't exceed steps
    pulses = min(pulses, steps)
    
    # Generate pattern using Bjorklund's algorithm
    pattern = []
    counts = []
    remainders = []
    divisor = steps - pulses
    
    remainders.append(pulses)
    level = 0
    
    while True:
        counts.append(divisor // remainders[level])
        remainders.append(divisor % remainders[level])
        divisor = remainders[level]
        level += 1
        if remainders[level] <= 1:
            break
    
    counts.append(divisor)
    
    # Build the pattern
    def build(level):
        if level == -1:
            return [True]
        elif level == -2:
            return [False]
        else:
            return build(level-1) * counts[level] + build(level-2) * remainders[level+1]
    
    # Get the pattern and rotate it if needed
    pattern = build(level)
    pattern = pattern[offset:] + pattern[:offset]
    
    return pattern

def create_euclidean_pattern(note_number, steps, pulses, offset=0):
    """Create notes based on Euclidean rhythm
    
    Args:
        note_number: MIDI note number
        steps: Total number of steps in the pattern
        pulses: Number of onsets (notes) to distribute
        offset: Rotation offset
    
    Returns:
        List of Note objects
    """
    # Generate pattern
    pattern = euclidean_rhythm(steps, pulses, offset)
    
    # Create notes
    notes = []
    step_duration = flp.score.PPQ // 4  # 16th note duration
    
    for i, has_note in enumerate(pattern):
        if has_note:
            note = flp.Note()
            note.number = note_number
            note.time = i * step_duration
            note.length = step_duration
            note.velocity = 0.8
            notes.append(note)
    
    return notes
```

### Drum Pattern Generator

```python
def create_drum_pattern(pattern_type, variation=0):
    """Create a drum pattern
    
    Args:
        pattern_type: "basic", "rock", "funk", "hiphop", "house", "breakbeat"
        variation: Pattern variation (0-3)
    
    Returns:
        List of Note objects
    """
    # Define standard drum note numbers
    KICK = 36
    SNARE = 38
    CLOSED_HH = 42
    OPEN_HH = 46
    CRASH = 49
    
    # Pattern definitions (1/16th note grid, 1 bar)
    patterns = {
        "basic": {
            "kick":  [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0],
            "snare": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0],
            "hihat": [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
        },
        "rock": {
            "kick":  [1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0],
            "snare": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0],
            "hihat": [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
        },
        "funk": {
            "kick":  [1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0],
            "snare": [0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0],
            "hihat": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
        },
        "hiphop": {
            "kick":  [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0],
            "snare": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0],
            "hihat": [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
        },
        "house": {
            "kick":  [1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, i, 1, 0, 0, 0],
            "snare": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0],
            "hihat": [0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0]
        },
        "breakbeat": {
            "kick":  [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            "snare": [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1],
            "hihat": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
        }
    }
    
    # Apply variations
    if variation > 0:
        pattern = patterns.get(pattern_type, patterns["basic"])
        
        if variation == 1:
            # Variation 1: Add ghost notes
            if "snare" in pattern:
                for i in range(len(pattern["snare"])):
                    if pattern["snare"][i] == 0 and random.random() < 0.2:
                        pattern["snare"][i] = 0.5  # Ghost note (lower velocity)
        
        elif variation == 2:
            # Variation 2: Change hihat pattern
            if "hihat" in pattern:
                for i in range(len(pattern["hihat"])):
                    if i % 2 == 1 and pattern["hihat"][i] > 0:
                        pattern["hihat"][i] = 0  # Remove some hihats
        
        elif variation == 3:
            # Variation 3: Add fills
            if "kick" in pattern and "snare" in pattern:
                fill_pos = random.randint(12, 15)
                pattern["kick"][fill_pos] = 1
                pattern["snare"][fill_pos] = 1
    
    # Create notes from patterns
    notes = []
    step_duration = flp.score.PPQ // 4  # 16th note duration
    
    if pattern_type in patterns:
        pattern = patterns[pattern_type]
        
        # Create kick notes
        if "kick" in pattern:
            for i, vel in enumerate(pattern["kick"]):
                if vel > 0:
                    note = flp.Note()
                    note.number = KICK
                    note.time = i * step_duration
                    note.length = step_duration
                    note.velocity = 0.8 if vel == 1 else 0.5  # Lower velocity for ghost notes
                    notes.append(note)
        
        # Create snare notes
        if "snare" in pattern:
            for i, vel in enumerate(pattern["snare"]):
                if vel > 0:
                    note = flp.Note()
                    note.number = SNARE
                    note.time = i * step_duration
                    note.length = step_duration
                    note.velocity = 0.8 if vel == 1 else 0.5  # Lower velocity for ghost notes
                    notes.append(note)
        
        # Create hihat notes
        if "hihat" in pattern:
            for i, vel in enumerate(pattern["hihat"]):
                if vel > 0:
                    # Alternate between closed and open hihat
                    hihat_type = CLOSED_HH if i % 4 != 2 else OPEN_HH
                    note = flp.Note()
                    note.number = hihat_type
                    note.time = i * step_duration
                    note.length = step_duration if hihat_type == CLOSED_HH else step_duration * 2
                    note.velocity = 0.8 if vel == 1 else 0.5  # Lower velocity for ghost notes
                    notes.append(note)
    
    return notes
```

## Best Practices

### Efficient Note Handling

```python
# Working with many notes efficiently

# 1. Collect and sort notes first
notes = []
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    notes.append((i, note))  # Store index with note

# Sort by time
notes.sort(key=lambda x: x[1].time)

# Process notes
for idx, note in notes:
    # Process notes sequentially
    pass

# 2. Delete notes in reverse order to avoid index shifting
for i in reversed(range(flp.score.noteCount)):
    if should_delete(flp.score.getNote(i)):
        flp.score.deleteNote(i)

# 3. Clear and rebuild for large changes
temp_notes = []
for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    if keep_note(note):
        # Modify note
        note.velocity *= 1.2
        temp_notes.append(note)

# Clear score and add modified notes
flp.score.clearNotes(True)
for note in temp_notes:
    flp.score.addNote(note)
```

### Error Handling

```python
# Add error handling to scripts
try:
    # Potentially risky code
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        note.number += 12  # Transpose up an octave
except Exception as e:
    # Log error and show message
    flp.Utils.log(f"Error: {str(e)}")
    flp.Utils.ShowMessage(f"An error occurred: {str(e)}")
    
    # Try to recover if possible
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        if note.number > 127:  # Fix invalid note numbers
            note.number = 127
```

### Performance Optimization

```python
# 1. Avoid unnecessary loops
note_count = flp.score.noteCount
for i in range(note_count):  # Cache the count instead of calling it each time
    note = flp.score.getNote(i)
    # Process note

# 2. Pre-calculate values outside loops
ppq = flp.score.PPQ
quarter_note = ppq
eighth_note = ppq // 2
sixteenth_note = ppq // 4

for i in range(flp.score.noteCount):
    note = flp.score.getNote(i)
    # Use pre-calculated values
    if note.length < sixteenth_note:
        note.length = eighth_note

# 3. Batch operations instead of individual ones
notes_to_add = []
for i in range(10):
    note = flp.Note()
    note.number = 60 + i
    note.time = i * ppq
    note.length = ppq
    notes_to_add.append(note)

# Add all notes at once
for note in notes_to_add:
    flp.score.addNote(note)
```

### Script Organization

```python
# Organize scripts with clear function structure

# Constants at the top
PPQ = flp.score.PPQ
NOTE_C4 = 60
DEFAULT_VELOCITY = 0.8

# Helper functions
def create_note(pitch, time, length, velocity=DEFAULT_VELOCITY):
    """Create a note with the given parameters"""
    note = flp.Note()
    note.number = pitch
    note.time = time
    note.length = length
    note.velocity = velocity
    return note

def transpose_notes(notes, semitones):
    """Transpose a list of notes by semitones"""
    for note in notes:
        note.number += semitones
    return notes

# Main functions
def create_chord(root, chord_type, time):
    """Create a chord at the specified time"""
    # Implementation here
    pass

def create_pattern(pattern_type, length):
    """Create a rhythmic pattern"""
    # Implementation here
    pass

# Dialog and application
def createDialog():
    form = flp.ScriptDialog("Organized Script", "A well-organized script example")
    # Add controls
    return form

def apply(form):
    # Get values from dialog
    
    # Use helper functions to implement logic
    
    # Apply changes to score
```

## Example Scripts

### Simple Note Generator

```python
import flpianoroll as flp

def createDialog():
    form = flp.ScriptDialog("Simple Note Generator", "Generate a series of notes with custom settings")
    
    form.AddInputCombo("Root Note", "C,C#,D,D#,E,F,F#,G,G#,A,A#,B", 0)
    form.AddInputKnobInt("Octave", 4, 1, 7)
    form.AddInputKnobInt("Number of Notes", 8, 1, 32)
    form.AddInputCombo("Pattern", "Scale,Arpeggio,Chromatic", 0)
    form.AddInputKnob("Velocity", 0.8, 0.1, 1.0)
    form.AddInputCheckbox("Clear Existing Notes", True)
    
    return form

def apply(form):
    # Get values from dialog
    root = form.GetInputValue("Root Note")
    octave = form.GetInputValue("Octave")
    num_notes = form.GetInputValue("Number of Notes")
    pattern_type = form.GetInputValue("Pattern")
    velocity = form.GetInputValue("Velocity")
    clear_notes = form.GetInputValue("Clear Existing Notes")
    
    # Clear existing notes if requested
    if clear_notes:
        flp.score.clearNotes(True)
    
    # Define patterns
    if pattern_type == 0:  # Scale
        intervals = [0, 2, 4, 5, 7, 9, 11, 12]  # Major scale
    elif pattern_type == 1:  # Arpeggio
        intervals = [0, 4, 7, 12, 7, 4, 0]  # Major arpeggio
    else:  # Chromatic
        intervals = [i for i in range(13)]  # Chromatic scale
    
    # Calculate base note
    base_note = (octave * 12) + root
    
    # Create notes
    for i in range(min(num_notes, len(intervals))):
        note = flp.Note()
        note.number = base_note + intervals[i]
        note.time = i * flp.score.PPQ  # Quarter note spacing
        note.length = flp.score.PPQ // 2  # Eighth note duration
        note.velocity = velocity
        flp.score.addNote(note)
    
    # Show confirmation
    flp.Utils.ShowMessage(f"Generated {num_notes} notes successfully!")
```

### Humanize Script

```python
import flpianoroll as flp
import random
import math

def createDialog():
    form = flp.ScriptDialog("Humanize", "Add human-like variations to notes")
    
    form.AddInputKnob("Time Variation", 0.1, 0, 0.5, "Timing randomization (0-50%)")
    form.AddInputKnob("Velocity Variation", 0.2, 0, 0.5, "Velocity randomization (0-50%)")
    form.AddInputKnob("Length Variation", 0.1, 0, 0.5, "Length randomization (0-50%)")
    form.AddInputKnobInt("Seed", random.randint(1, 1000), 1, 1000, "Random seed for reproducible results")
    form.AddInputCombo("Distribution", "Uniform,Triangle,Gaussian", 2, "Type of random distribution")
    form.AddInputCheckbox("Apply to Selected Only", False, "Affect only selected notes")
    
    return form

def apply(form):
    # Get values from dialog
    time_var = form.GetInputValue("Time Variation") 
    vel_var = form.GetInputValue("Velocity Variation")
    len_var = form.GetInputValue("Length Variation")
    seed = form.GetInputValue("Seed")
    dist_type = form.GetInputValue("Distribution")
    selected_only = form.GetInputValue("Apply to Selected Only")
    
    # Set random seed
    random.seed(seed)
    
    # Distribution functions
    def get_random_value():
        if dist_type == 0:  # Uniform
            return random.random() * 2 - 1  # Range: -1 to 1
        elif dist_type == 1:  # Triangle
            return (random.random() + random.random()) - 1  # Simple triangle approximation
        else:  # Gaussian
            # Box-Muller transform for quasi-normal distribution
            u1 = random.random()
            u2 = random.random()
            z = math.sqrt(-2 * math.log(u1)) * math.cos(2 * math.pi * u2)
            return max(-1, min(1, z * 0.5))  # Clamp to -1 to 1
    
    # Process notes
    for i in range(flp.score.noteCount):
        note = flp.score.getNote(i)
        
        # Skip if not selected and we're only processing selected notes
        if selected_only and not note.selected:
            continue
        
        # Humanize timing
        if time_var > 0:
            max_time_shift = int(flp.score.PPQ * time_var / 4)  # Max shift in ticks
            time_shift = int(get_random_value() * max_time_shift)
            note.time = max(0, note.time + time_shift)
        
        # Humanize velocity
        if vel_var > 0:
            # Get random value and scale by variation amount
            vel_shift = get_random_value() * vel_var
            note.velocity = max(0.1, min(1.0, note.velocity * (1 + vel_shift)))
        
        # Humanize length
        if len_var > 0:
            # Get random value and scale by variation amount
            len_shift = get_random_value() * len_var
            note.length = max(1, int(note.length * (1 + len_shift)))
    
    # Show confirmation
    flp.Utils.ShowMessage("Humanization applied successfully!")
```

### Chord Progression Generator

```python
import flpianoroll as flp
import random

def createDialog():
    form = flp.ScriptDialog("Chord Progression Generator", "Create chord progressions in various styles")
    
    # Basic settings
    form.AddInputCombo("Key", "C,C#,D,D#,E,F,F#,G,G#,A,A#,B", 0)
    form.AddInputCombo("Scale", "Major,Minor,Dorian,Mixolydian", 0)
    form.AddInputKnobInt("Octave", 4, 2, 6)
    
    # Progression settings
    form.AddInputCombo("Style", "Pop,Jazz,Classical,Random", 0)
    form.AddInputKnobInt("Number of Chords", 4, 1, 8)
    form.AddInputKnobInt("Bars per Chord", 1, 1, 4)
    
    # Chord settings
    form.AddInputCombo("Chord Type", "Triads,7th Chords,9th Chords,Mixed", 0)
    form.AddInputKnobInt("Inversion", 0, 0, 3)
    form.AddInputCheckbox("Add Extensions", False)
    
    # Playback settings
    form.AddInputKnob("Velocity", 0.8, 0.5, 1.0)
    form.AddInputKnob("Strum Amount", 0, 0, 0.5)
    form.AddInputCheckbox("Clear Existing Notes", True)
    
    return form

def apply(form):
    # Get values from form
    key = form.GetInputValue("Key")
    scale_type = form.GetInputValue("Scale")
    octave = form.GetInputValue("Octave")
    style = form.GetInputValue("Style")
    num_chords = form.GetInputValue("Number of Chords")
    bars_per_chord = form.GetInputValue("Bars per Chord")
    chord_type = form.GetInputValue("Chord Type")
    inversion = form.GetInputValue("Inversion")
    add_extensions = form.GetInputValue("Add Extensions")
    velocity = form.GetInputValue("Velocity")
    strum_amount = form.GetInputValue("Strum Amount")
    clear_notes = form.GetInputValue("Clear Existing Notes")
    
    # Clear existing notes if requested
    if clear_notes:
        flp.score.clearNotes(True)
    
    # Define scales
    scales = {
        0: [0, 2, 4, 5, 7, 9, 11],  # Major
        1: [0, 2, 3, 5, 7, 8, 10],  # Minor
        2: [0, 2, 3, 5, 7, 9, 10],  # Dorian
        3: [0, 2, 4, 5, 7, 9, 10]   # Mixolydian
    }
    
    # Define chord progressions by style
    progressions = {
        0: [  # Pop
            [1, 5, 6, 4],  # I-V-vi-IV
            [1, 4, 5, 1],  # I-IV-V-I
            [1, 6, 4, 5],  # I-vi-IV-V
            [6, 4, 1, 5]   # vi-IV-I-V
        ],
        1: [  # Jazz
            [2, 5, 1, 6],  # ii-V-I-vi
            [1, 6, 2, 5],  # I-vi-ii-V
            [3, 6, 2, 5],  # iii-vi-ii-V
            [1, 7, 3, 6]   # I-vii-iii-vi
        ],
        2: [  # Classical
            [1, 4, 5, 1],  # I-IV-V-I
            [1, 5, 6, 3],  # I-V-vi-iii
            [6, 2, 5, 1],  # vi-ii-V-I
            [1, 6, 4, 5]   # I-vi-IV-V
        ]
    }
    
    # Define chord types
    chord_types = {
        0: {  # Triads
            "major": [0, 4, 7],
            "minor": [0, 3, 7],
            "diminished": [0, 3, 6],
            "augmented": [0, 4, 8]
        },
        1: {  # 7th Chords
            "major7": [0, 4, 7, 11],
            "dominant7": [0, 4, 7, 10],
            "minor7": [0, 3, 7, 10],
            "half-diminished7": [0, 3, 6, 10]
        },
        2: {  # 9th Chords
            "major9": [0, 4, 7, 11, 14],
            "dominant9": [0, 4, 7, 10, 14],
            "minor9": [0, 3, 7, 10, 14]
        }
    }
    
    # Define chord qualities based on scale degree
    chord_qualities = {
        0: {  # Major scale
            1: "major",
            2: "minor",
            3: "minor",
            4: "major",
            5: "major",
            6: "minor",
            7: "diminished"
        },
        1: {  # Minor scale
            1: "minor",
            2: "diminished",
            3: "major",
            4: "minor",
            5: "minor",
            6: "major",
            7: "major"
        },
        2: {  # Dorian scale
            1: "minor",
            2: "minor",
            3: "major",
            4: "major",
            5: "minor",
            6: "diminished",
            7: "major"
        },
        3: {  # Mixolydian scale
            1: "major",
            2: "minor",
            3: "diminished",
            4: "major",
            5: "minor",
            6: "minor",
            7: "major"
        }
    }
    
    # Select progression based on style
    if style == 3:  # Random
        prog = random.choice(sum(progressions.values(), []))
    else:
        prog = random.choice(progressions[style])
    
    # Limit progression to requested number of chords
    prog = prog[:num_chords]
    
    # Get selected scale
    scale = scales[scale_type]
    
    # Calculate base note for the key
    base_note = (octave * 12) + key
    
    # Calculate ticks per chord
    ticks_per_chord = bars_per_chord * 4 * flp.score.PPQ
    
    # Create chords
    for chord_idx, degree in enumerate(prog):
        # Adjust degree to 0-based index
        degree_idx = degree - 1
        
        # Get chord root note from scale
        chord_root = base_note + scale[degree_idx % 7]
        
        # Determine chord quality
        quality = chord_qualities[scale_type][degree]
        
        # Select chord intervals based on type and quality
        if chord_type == 3:  # Mixed
            # Randomly select between triad, 7th, and 9th
            type_idx = random.randint(0, 2)
        else:
            type_idx = chord_type
        
        # Get intervals for this chord
        intervals = chord_types[type_idx][quality]
        
        # Add extensions if requested
        if add_extensions and random.random() < 0.3:
            # 30% chance to add extensions if enabled
            if len(intervals) == 3:  # Triad
                # Add 6th or maj7 for major, 7th for minor
                if quality == "major":
                    if random.random() < 0.5:
                        intervals.append(9)  # Add 6th
                    else:
                        intervals.append(11)  # Add maj7
                elif quality == "minor":
                    intervals.append(10)  # Add m7
        
        # Apply inversion
        for i in range(inversion):
            if i < len(intervals):
                intervals[i] += 12
        
        # Create notes for this chord
        for i, interval in enumerate(intervals):
            note = flp.Note()
            note.number = chord_root + interval
            
            # Set start time based on chord position
            note.time = chord_idx * ticks_per_chord
            
            # Apply strum if enabled
            if strum_amount > 0:
                strum_offset = int(i * strum_amount * flp.score.PPQ / 4)
                note.time += strum_offset
            
            # Set note length to chord duration minus strum offset
            note.length = ticks_per_chord - (strum_offset if strum_amount > 0 else 0)
            
            # Set velocity
            note.velocity = velocity * (1 - i * 0.05)  # Slightly lower velocity for upper notes
            
            # Add note to score
            flp.score.addNote(note)
    
    # Show confirmation
    flp.Utils.ShowMessage(f"Created {len(prog)} chord progression in {['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'][key]} {['Major', 'Minor', 'Dorian', 'Mixolydian'][scale_type]}")
```

This comprehensive documentation covers all aspects of the FL Studio Piano Roll Scripting
