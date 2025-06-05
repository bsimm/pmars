# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pMARS (portable Memory Array Redcode Simulator) is a cross-platform Core War simulator that implements the ICWS'94 standard with backwards compatibility to ICWS'88. Core War is a programming game where virus-like programs written in Redcode assembly language fight each other in a simulated memory space.

## Build System

The project uses traditional Unix Makefiles. Main build commands:

```bash
# Build the default version (X11 graphics with EXT94 extensions)
cd src && make

# Clean build artifacts
cd src && make clean

# Strip the final binary (done automatically)
strip pmars
```

**macOS/XQuartz Notes**: The Makefile has been updated to work with modern XQuartz installations (X11 libraries at `/opt/X11/` instead of `/usr/X11R6/`).

## Build Configuration

Build options are controlled via preprocessor defines in `src/config.h` or via CFLAGS in the Makefile:

- `-DGRAPHX`: Enables platform-specific graphical core display
- `-DSERVER`: Tournament version without debugger (mutually exclusive with GRAPHX)
- `-DEXT94`: Enables ICWS'94 extensions (SEQ, SNE, NOP opcodes and A-field addressing modes)
- `-DXWINGRAPHX`: X11 display support
- `-DSMALLMEM`: 16-bit addresses for reduced memory usage
- `-DPERMUTATE`: Enables -P switch for permutation
- `-DRWLIMIT`: Enables read/write limits

Current default configuration includes: EXT94, XWINGRAPHX, PERMUTATE, RWLIMIT.

## Architecture

### Core Components

- **pmars.c**: Main entry point, initialization, and cleanup
- **sim.c**: Core simulation engine that executes Redcode instructions
- **asm.c**: Redcode assembler that parses .red files into executable format
- **eval.c**: Expression evaluator for assembly-time calculations
- **disasm.c**: Disassembler for debugging output
- **cdb.c**: Interactive debugger implementation
- **pos.c**: Warrior positioning logic
- **clparse.c**: Command-line argument parsing
- **token.c**: Tokenizer for Redcode source parsing

### Display Modules

The simulator supports multiple display backends:
- **curdisp.c**: Curses-based text display
- **uidisp.c**: User interface display functions
- **xwindisp.c**: X11 graphical display
- **lnxdisp.c**: Linux SVGA display
- **grxdisp.c**: Graphics display routines

### Key Headers

- **global.h**: Global definitions, memory allocation macros, platform compatibility
- **config.h**: Compile-time configuration switches
- **sim.h**: Simulation engine structures and definitions
- **asm.h**: Assembler tokens, symbols, and function declarations

### Data Flow

1. **Parse**: Command-line options processed by clparse.c
2. **Assemble**: Redcode files (.red) parsed by asm.c using token.c
3. **Load**: Warriors positioned in core memory by pos.c
4. **Simulate**: Core execution handled by sim.c with optional display updates
5. **Debug**: Interactive debugging via cdb.c when enabled

## Redcode Language

The assembler supports ICWS'94 Redcode with extensions:
- Standard opcodes: MOV, ADD, SUB, MUL, DIV, MOD, JMP, JMZ, JMN, DJN, CMP, SLT, SPL, DAT
- Extended opcodes (EXT94): SEQ, SNE, NOP, LDP, STP, PIN
- Addressing modes: immediate (#), direct ($), indirect (@), predecrement (<), postincrement (>)
- Extended A-field modes (EXT94): *, {, }
- Preprocessor: EQU, FOR/ROF loops, ORG, END

## Testing

Test warriors are located in `warriors/` directory:
- `aeka.red`: Complex multi-strategy warrior example
- `validate.red`: Validation tests
- `test_eval.red`: Expression evaluation tests
- `pspace.red`: P-space functionality tests

## Platform Support

The codebase is designed for maximum portability across:
- Unix/Linux (curses and X11 displays)
- DOS (16-bit and 32-bit versions)
- Mac (with separate GUI code)
- VMS (separate build scripts)
- Windows/OS2 (limited display support)

Platform-specific code is isolated using preprocessor conditionals based on compiler-defined macros.

## Adding SDL2 Graphics Support

To add modern SDL2 graphics support for cross-platform compatibility:

### 1. **New Display Module**
Create `sdl2disp.c` following the existing display interface pattern:
- **Core functions**: `display_init()`, `display_close()`, `display_clear()`, `display_update()`
- **Pixel manipulation**: `display_read()`, `display_write()`, `display_exec()`, etc.
- **Input handling**: `display_gets()`, `display_getkey()` for debugger interaction
- **Color management**: Map the existing color constants to SDL2 colors

### 2. **Build System Changes**
```bash
# New preprocessor flag
-DSDL2GRAPHX

# Link against SDL2
LIB = -lSDL2

# Conditional compilation in Makefile
```

### 3. **Integration Points**
The display system is cleanly abstracted in `sim.c`:
```c
#if defined(SDL2GRAPHX)
#include "sdl2disp.c"
#elif defined(XWINGRAPHX) 
#include "xwindisp.c"
// ... other displays
#endif
```

### 4. **Core Display Interface**
Based on existing patterns in `xwindisp.c`, implement:
- **Window management**: Create SDL2 window/renderer
- **Core visualization**: 8000-cell memory array as colored pixels/rectangles  
- **Warrior tracking**: Different colors for each warrior's processes
- **Real-time updates**: Smooth animation of instruction execution
- **Keyboard/mouse input**: Navigation and debugger controls

### 5. **Key Advantages of SDL2**
- **Cross-platform**: Works on macOS, Linux, Windows without X11 dependency
- **Modern graphics**: Hardware acceleration, better performance than X11
- **Simplified input**: Unified keyboard/mouse/gamepad handling
- **No external dependencies**: Self-contained graphics library

### 6. **Implementation Strategy**
1. Copy `xwindisp.c` → `sdl2disp.c` as starting template
2. Replace X11 calls with SDL2 equivalents:
   - `XCreateWindow()` → `SDL_CreateWindow()`
   - `XDrawPoint()` → `SDL_RenderDrawPoint()`
   - `XNextEvent()` → `SDL_PollEvent()`
3. Add `config.h` option for SDL2GRAPHX
4. Update Makefile with SDL2 detection/linking

The existing architecture is well-designed for this - the display layer is completely modular, so SDL2 support would be a clean addition without touching the core simulation engine.