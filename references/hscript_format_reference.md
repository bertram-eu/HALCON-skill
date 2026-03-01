# HScript (`.hscript`) Format Reference

HDevelopEVO uses plain-text `.hscript` files instead of HDevelop's XML-based `.hdev` files.
The HALCON operator calls, control flow, and data model are identical — only the file format and a few language-level constructs differ.

## File format comparison

| Aspect | HDevelop (`.hdev`) | HDevelopEVO (`.hscript`) |
|--------|-------------------|--------------------------|
| File format | XML (`<hdevelop>` root, `<procedure>`, `<body>`, `<l>` for code lines, `<c>` for comments) | Plain text (code written directly, no XML wrapping) |
| File extension — scripts | `.hdev` | `.hscript` |
| File extension — procedure (single) | `.hdvp` | `.hscript` (same extension, procedure file contains `proc`/`endproc` blocks) |
| File extension — procedure library | `.hdpl` | `.hscript` (same extension, file contains multiple `proc`/`endproc` blocks) |
| IDE | HDevelop | HDevelopEVO |
| CLI runner | `hrun` | `hscriptengine` |
| Comments | `* comment` (inside `<c>` tags in XML) | `* comment` or `// comment` |
| Line comments | Only `*` at line start | `*` at line start **and** `//` anywhere |
| Procedure definition | Embedded in XML `<procedure name="...">` with `<interface>` blocks | `proc name (params)` ... `endproc` |
| Procedure visibility | N/A (all procedures are accessible) | `public proc` (exported) vs `proc` (file-local) |
| Import mechanism | `import` keyword or HDevelop procedure search path | `from './path/to/file' import *` |
| Interface declaration | XML tags: `<io>`, `<oo>`, `<ic>`, `<oc>` | Inline in proc signature: `object`, `out object`, `tuple`, `out tuple` |
| Main entry point | `<procedure name="main">` | Top-level code (no `main` wrapper needed) |

## Comment syntax

```hscript
* This is a line comment (works in both .hdev and .hscript)
// This is also a valid comment (HScript only)
// HScript supports C-style line comments
```

## Procedure syntax

### HDevelop (`.hdvp` / `.hdev` XML)
```xml
<procedure name="detect_fin">
<interface>
<io>
  <par name="Image" base_type="iconic" dimension="0"/>
</io>
<oo>
  <par name="FinRegion" base_type="iconic" dimension="0"/>
</oo>
<oc>
  <par name="FinArea" base_type="ctrl" dimension="0"/>
</oc>
</interface>
<body>
<l>binary_threshold (Image, Dark, 'max_separability', 'dark', UsedThreshold)</l>
<l>area_center (FinRegion, FinArea, Row, Column)</l>
</body>
</procedure>
```

### HScript (`.hscript`)
```hscript
public proc detect_fin(object Image, out object FinRegion, out tuple FinArea)
  binary_threshold (Image, Dark, 'max_separability', 'dark', UsedThreshold)
  area_center (FinRegion, FinArea, Row, Column)
endproc
```

### Procedure parameter types

| XML interface tag | HScript parameter prefix | Meaning |
|-------------------|-------------------------|---------|
| `<io>` (`base_type="iconic"`) | `object` | Input iconic (image/region/XLD) |
| `<oo>` (`base_type="iconic"`) | `out object` | Output iconic |
| `<ic>` (`base_type="ctrl"`) | `tuple` | Input control (number/string/handle) |
| `<oc>` (`base_type="ctrl"`) | `out tuple` | Output control |

### Procedure visibility

```hscript
* Exported — callable from other files that import this file
public proc my_public_proc(tuple Input, out tuple Output)
  Output := Input * 2
  return ()
endproc

* File-local — only callable within the same file
proc my_helper(tuple X, out tuple Y)
  Y := X + 1
  return ()
endproc
```

## Import syntax

HScript uses Python-style `from ... import` to load external procedures:

```hscript
* Import all public procedures from a file (relative path, no extension)
from '../procedures/general/disp_message' import *
from '../procedures/general/dev_open_window_fit_image' import *

* The imported procedures can then be called directly
disp_message (WindowHandle, 'Hello', 'window', 12, 12, 'black', 'true')
```

In HDevelop (`.hdev`), external procedures are found via the procedure search path or `import` statement — no `from` syntax.

## Main program structure

### HDevelop (`.hdev`)
All top-level code must be inside a `<procedure name="main">` block. Local procedures are separate `<procedure>` elements in the same XML file.

### HScript (`.hscript`)
Top-level code runs directly as the main program — no `main` wrapper is needed. Local procedures are defined with `proc`/`endproc` anywhere in the file (typically at the top or bottom).

```hscript
from '../procedures/general/disp_message' import *

* --- Local procedure ---
proc initialize_window(tuple Width, tuple Height, out tuple WindowHandle)
    dev_open_window (0, 0, Width, Height, 'black', WindowHandle)
    return ()
endproc

* --- Main program (top-level code) ---
read_image (Image, 'rings_and_nuts')
get_image_size (Image, Width, Height)
initialize_window (Width, Height, WindowHandle)
dev_display (Image)
```

## Operators and control flow

All HALCON operators and control-flow constructs are identical in both formats:

```hscript
* Assignment
Threshold := 128

* Operator call
read_image (Image, 'fabrik')
threshold (Image, Region, Threshold, 255)

* Control flow
for i := 1 to 10 by 1
    select_obj (Objects, Obj, i)
endfor

while (Condition)
    * ...
endwhile

if (Area > 1000)
    Result := 'ok'
elseif (Area > 500)
    Result := 'marginal'
else
    Result := 'nok'
endif

switch (Mode)
case 1:
    * ...
    break
case 2:
    * ...
    break
default:
    * ...
    break
endswitch

try
    * ...
catch (Exception)
    * ...
endtry
```

## Unsupported HDevelop features in HDevelopEVO

Some HDevelop `dev_*` operators are not available in HDevelopEVO. These appear as `FIXME` comments in converted example scripts:

```hscript
* FIXME: no equivalent dev_update_window ('off')
* FIXME: no equivalent dev_update_pc ('off')
* FIXME: no equivalent dev_update_var ('off')
```

The affected operators are primarily IDE-update controls:
- `dev_update_window` — use `dev_update_off()` / `dev_update_on()` procedures instead where available
- `dev_update_pc` — no direct equivalent
- `dev_update_var` — no direct equivalent
- `dev_error_var` — no direct equivalent
- `dev_open_dialog` — no direct equivalent (HDevelopEVO uses a different dialog system)

For the `hscriptengine` CLI reference (help output, usage patterns, authoring guidelines, and comparison with `hrun`), see [`hscriptengine_reference.md`](hscriptengine_reference.md).
