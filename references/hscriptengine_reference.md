# `hscriptengine` CLI Reference

`hscriptengine` is the headless script runner for HDevelopEVO `.hscript` files. It executes scripts from the command line without the HDevelopEVO IDE. It is part of the HDevelopEvo preview component.

Note: `hscriptengine` does not support a `--version` flag.

## Help output

```
Usage: hscriptengine [action] {options} [input file]

actions:
  run (default)           execute procedure from input *.hscript file
  check                   list all errors in input file or imported files [*]

options:
  --help                  show this screen
  --exec <proc_name>      the procedure to execute (default = 'main')
  --input <expr>|<stmt>   pass value (use once per input parameter)
      <expr>: HScript expression that evaluates to a value e.g "sin(1) + 4"
      <stmt>: Initialization code using the placeholder '#ARG'

examples:
  hscriptengine program.hscript
  hscriptengine --exec proc --input 42 --input 2.5 --input "'true'" lib.hscript
  hscriptengine --exec proc --input "[1, 2, 3]" lib.hscript
  hscriptengine --exec proc --input "read_image(#ARG, 'test.png')" lib.hscript

[*] Requires SDK license
```

## Typical usage patterns

- **Run a script:** `hscriptengine program.hscript`
- **Run a named procedure:** `hscriptengine --exec my_proc lib.hscript`
- **Pass scalar inputs:** `hscriptengine --exec proc --input 42 --input 2.5 lib.hscript`
- **Pass a string:** `hscriptengine --exec proc --input "'true'" lib.hscript`
- **Pass a tuple:** `hscriptengine --exec proc --input "[1, 2, 3]" lib.hscript`
- **Pass an image via operator:** `hscriptengine --exec proc --input "read_image(#ARG, 'test.png')" lib.hscript`
- **Static error check:** `hscriptengine check program.hscript`

## Authoring guidelines for `hscriptengine`-compatible scripts

- Avoid `stop()` — it blocks headless execution.
- Avoid unsupported `dev_*` operators: `dev_update_window`, `dev_update_pc`, `dev_update_var`, `dev_error_var`, `dev_open_dialog`. Use `dev_update_off()`/`dev_update_on()` procedures instead where available.
- Use `--input` to pass positional parameters to the executed procedure. Design procedures with typed parameters (`tuple`, `object`) for clean CLI invocation.
- Use `--exec` to call a specific `public proc` by name — useful for running individual procedures from a library file.
- Use the `check` action for static validation before runtime.

## Comparison with `hrun`

| Feature | `hrun` (`.hdev`) | `hscriptengine` (`.hscript`) |
|---------|-------------------|-------------------------------|
| Input file | `.hdev` | `.hscript` |
| Pass parameters | `-D Var=value` (global control variable) | `--input <expr>` (positional, per parameter) |
| Run specific procedure | `-t <procedure>` (threaded mode) | `--exec <proc_name>` (direct call) |
| Buffered windows | `-b` | Not needed (no visible windows by default) |
| Dump variables | `-d` / `-d=Var` | Not available |
| Check result | `-c <variable>` | Not available |
| JIT compilation | `-j` | Not available |
| Static error check | Not available | `check` action |
| Debug server | `--debug <port>` | Not available |
| Verbose mode | `-v` | Not available |
| Version flag | `-V` / `--version` | Not available |
| XL variant | `hrunxl` | Not available |
| C++ embedding API | Not available | `HScriptEngineCpp` library |
