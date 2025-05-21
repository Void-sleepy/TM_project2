# TM_project2
# Turing Machine Simulator Documentation

## Overview

This Python implementation provides a complete Turing Machine simulator with support for custom machine definitions, built-in examples, and interactive execution modes. The simulator includes validation, step-by-step execution, configuration management, and comprehensive error handling.

## Table of Contents

1. [Class: TuringMachine](#class-turingmachine)
2. [Built-in Machine Generators](#built-in-machine-generators)
3. [Configuration Management](#configuration-management)
4. [Main Interface](#main-interface)
5. [Usage Examples](#usage-examples)
6. [File Format](#file-format)

---

## Class: TuringMachine

The core class that represents and simulates a Turing Machine.

### Constructor

```python
TuringMachine(blank_symbol='_', tapePadding=5, defaultMovementDirection='R')
```

**Parameters:**
- `blank_symbol` (str): Symbol representing empty tape cells (default: '_')
- `tapePadding` (int): Number of extra cells to display around the active tape region (default: 5)
- `defaultMovementDirection` (str): Default head movement direction when undefined ('L' or 'R', default: 'R')

**Raises:**
- `ValueError`: If tapePadding is negative or defaultMovementDirection is invalid

### Core Attributes

#### Machine Definition
- `states` (Set[str]): Set of all machine states
- `input_alphabet` (Set[str]): Valid input symbols
- `tape_alphabet` (Set[str]): All symbols that can appear on tape
- `transitions` (Dict): State transition function as `{(state, symbol): (new_state, write_symbol, direction)}`
- `start_state` (str): Initial state
- `accept_states` (Set[str]): States that indicate string acceptance
- `reject_states` (Set[str]): States that indicate string rejection

#### Runtime State
- `tape` (Dict[int, str]): Infinite tape represented as sparse dictionary
- `head` (int): Current tape head position
- `current_state` (str): Current machine state
- `steps_taken` (int): Number of execution steps performed

#### Metadata
- `creation_date` (str): UTC timestamp of machine creation
- `creator` (str): System username of machine creator
- `last_run_time` (datetime): Duration of last execution

### Methods

#### `validate() -> None`
Validates the complete machine definition for consistency and completeness.

**Validates:**
- Non-empty state set
- Start state exists in states
- Accept/reject states are subsets of states  
- Tape alphabet contains input alphabet and blank symbol
- All transition states and symbols are valid
- All transition directions are 'L' or 'R'

**Raises:**
- `ValueError`: If any validation check fails

#### `run(input_str: str, max_steps: int = 10000, step_mode: bool = False) -> bool`
Executes the Turing Machine on the given input string.

**Parameters:**
- `input_str` (str): Input string to process (max 10,000 characters)
- `max_steps` (int): Maximum execution steps before timeout (default: 10,000)
- `step_mode` (bool): If True, pauses after each step for user input

**Returns:**
- `bool`: True if string is accepted, False if rejected or timed out

**Process:**
1. Validates input string against input alphabet
2. Initializes tape with input string
3. Sets current state to start state
4. Executes transitions until halt state or max steps reached
5. Returns acceptance status

**Output:** Prints detailed execution trace including tape state and head position

#### `print_state() -> None`
Displays current machine state with visual tape representation.

**Output Format:**
```
Tape: _[0]11_
Head: 0, State: q1
```
- Brackets `[]` indicate head position
- Shows tape padding around active region

#### `reset() -> None`
Resets machine to initial state, clearing tape and resetting counters.

#### `get_stats() -> dict`
Returns dictionary with execution statistics and metadata.

**Returns:**
```python
{
    "tape_size": int,           # Number of non-blank tape cells
    "num_states": int,          # Total states in machine
    "num_transitions": int,     # Total transition rules
    "steps_taken": int,         # Steps in last execution
    "last_run_time": datetime,  # Duration of last run
    "creator": str,             # Username of creator
    "creation_date": str        # UTC creation timestamp
}
```

#### `save_config(filename: str) -> None`
Saves machine configuration to file in standard format.

**File Format:** See [File Format](#file-format) section below.

---

## Built-in Machine Generators

### `create_0n1n_tm() -> TuringMachine`
Creates a Turing Machine that recognizes the language {0ⁿ1ⁿ | n ≥ 0}.

**Language:** Strings with equal numbers of 0s followed by 1s
- Accepts: "", "01", "0011", "000111"
- Rejects: "0", "1", "001", "0111"

**Algorithm:**
1. Mark leftmost 0 with X, move right
2. Find corresponding 1, mark with Y, move left
3. Return to leftmost unmarked 0
4. Repeat until all 0s matched with 1s
5. Accept if only X and Y remain

### `create_palindrome_tm() -> TuringMachine`
Creates a Turing Machine that recognizes palindromes over alphabet {a, b}.

**Language:** Strings that read the same forwards and backwards
- Accepts: "", "a", "b", "aba", "abba", "baaab"
- Rejects: "ab", "abc", "abab"

**Algorithm:**
1. Erase leftmost symbol, remember it
2. Move to rightmost symbol
3. Check if it matches, erase if so
4. Return to leftmost position
5. Repeat until string exhausted
6. Accept if all symbols successfully matched

---

## Configuration Management

### `parse_config(lines: List[str]) -> TuringMachine`
Parses machine configuration from list of text lines.

**Parameters:**
- `lines` (List[str]): Configuration text split into lines

**Returns:**
- `TuringMachine`: Configured and validated machine

**Raises:**
- `ValueError`: If configuration is invalid or malformed

### `load_tm(filename: str, blank_symbol: str = '_') -> TuringMachine`
Loads machine configuration from file.

**Parameters:**
- `filename` (str): Path to configuration file
- `blank_symbol` (str): Blank symbol to use (default: '_')

### `load_tm_from_console(blank_symbol: str = '_') -> TuringMachine`
Interactive console input for machine configuration.

**Process:**
1. Displays format instructions
2. Reads configuration lines until double-enter
3. Parses and validates configuration
4. Returns configured machine

---

## Main Interface

### `main() -> None`
Interactive command-line interface for the Turing Machine simulator.

**Menu Options:**
1. **Use default TM for {0ⁿ1ⁿ | n ≥ 0}** - Loads built-in equal 0s and 1s recognizer
2. **Use TM for palindromes over {a,b}** - Loads built-in palindrome recognizer  
3. **Load TM from file** - Loads machine from configuration file
4. **Enter TM manually** - Interactive machine definition
5. **Save current TM** - Saves current machine to file
6. **Exit** - Terminates program

**Execution Flow:**
1. Select machine source (built-in, file, or manual)
2. Enter input strings for testing
3. Choose normal or step-by-step execution mode
4. View results and execution statistics
5. Return to main menu or continue with same machine

---

## Usage Examples

### Basic Usage
```python
# Create and run built-in machine
tm = create_0n1n_tm()
result = tm.run("0011")  # Returns True (accepted)

# Check execution statistics
stats = tm.get_stats()
print(f"Steps taken: {stats['steps_taken']}")
```

### Custom Machine Definition
```python
# Create custom machine
tm = TuringMachine()
tm.states = {'q0', 'q1', 'accept'}
tm.input_alphabet = {'a', 'b'}
tm.tape_alphabet = {'a', 'b', '_'}
tm.start_state = 'q0'
tm.accept_states = {'accept'}
tm.transitions = {
    ('q0', 'a'): ('q1', 'a', 'R'),
    ('q1', 'b'): ('accept', 'b', 'R')
}
tm.validate()
```

### File Operations
```python
# Save machine configuration
tm.save_config("my_machine.tm")

# Load machine from file
tm = load_tm("my_machine.tm")
```

---

## File Format

Configuration files use a structured text format with sections:

```
# Comments start with #
# Created by: username
# Date: YYYY-MM-DD HH:MM:SS

States:
q1,q2,q3,accept,reject

Input Alphabet:
0,1

Tape Alphabet:
0,1,X,Y,_

Start:
q1

Accept:
accept

Reject:
reject

Transitions:
q1,0 -> q2,X,R
q2,1 -> q3,Y,L
q3,X -> q1,X,R
```

**Format Rules:**
- Each section header ends with colon
- Lists are comma-separated
- Transitions: `current_state,symbol -> new_state,write_symbol,direction`
- Comments and empty lines ignored
- Sections can appear in any order

**Required Sections:**
- States, Input Alphabet, Tape Alphabet, Start, Accept, Reject, Transitions

**Optional Elements:**
- Comments (lines starting with #)
- Empty lines
- Reject states (can be empty)

---

## Error Handling

The simulator includes comprehensive error handling:

- **Input Validation**: Checks input strings against input alphabet
- **Configuration Validation**: Validates machine definitions for completeness and consistency
- **Runtime Protection**: Prevents infinite loops with step limits
- **File Handling**: Graceful handling of missing or malformed files
- **User Input**: Validates all user inputs with helpful error messages

**Common Error Messages:**
- "Input character 'x' not in input alphabet"
- "Start state 'q' not in states"
- "Max steps reached, halting"
- "No transition defined" (leads to rejection)

---

## Dependencies

- **typing**: Type hints for better code documentation
- **datetime**: Execution timing and metadata timestamps
- **getpass**: System username detection
- **os**: Environment variable access for username fallback

No external libraries required - uses only Python standard library.


```
Fares hany   221000846

Hussin yasser 22100800

Mohammed Mohammed 221000663

omar Mohamed  221000495
```
