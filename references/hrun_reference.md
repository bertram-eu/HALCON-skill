# `hrun` CLI Reference

`hrun` (v25.11) is the headless script runner for HDevelop `.hdev` files. It executes scripts from the command line without the HDevelop IDE. This is especially useful when HDevelop is not installed on the target system, for example on embedded systems.

A variant `hrunxl` exists with identical options for HALCON XL (supports images larger than 32k x 32k).

## Help output

```
Usage: hrun [options] <script>

Options:
  -b, --buffered            dev_open_window opens buffered instead of visible
                            windows.
  -c, --check <variable>    Check the result of variable after running the
                            script. If it is undefined or false (i.e. 0 for
                            integer, 0.0 for float, not 'true' for string
                            variables), exit with an error. In threaded mode,
                            check global control variables.
  -d, --dump=[variable]     Dump the values of variables used by the script. If
                            a variable is specified, only dump it instead of
                            all variables. In threaded mode, dump the values of
                            global control variables. This option can be used
                            multiple times.
  -D <variable>=<value>     Set the global control variable <variable> to the
                            string <value>. This option can be used multiple
                            times.
  -j, --jit                 Run procedures JIT-compiled.
  -l, --linger              Don't exit after the end of the script until the
                            user presses a key.
  -t <procedure>            Run procedure in a separate thread. This option
                            will switch hrun into threaded mode, which requires
                            the script to have procedures named 'init' and
                            'cleanup', which are run in the main thread before
                            and after the threaded procedures, respectively.
  --no-spinlocks            Disable HALCON spinlocks.
  -p, --procedure path      Add 'path' to the search path for external
                            procedures. If this option is used multiple times,
                            the paths are searched in the order they were added
                            on the command line.
  -P, --pedantic            Don't ignore invalid or unresolved lines in the
                            script.
  -W, --windowthread        Set the 'use_window_thread' system option to
                            'false' (default is 'true').
  --debug <port>            Start a debug server on port <port> and wait for a
                            debugger to connect to it.
  -h, --help                Print this message and exit.
  -v, --verbose             Be verbose.
  -V, --version             Print the version number and exit.
```

## Typical usage patterns

- **Run a script:** `hrun my_script.hdev`
- **Run headless (no windows):** `hrun -b my_script.hdev`
- **Pass parameters:** `hrun -D Threshold=128 -D ImageDir=./images my_script.hdev`
- **Check result variable:** `hrun -c Result my_script.hdev` (exits non-zero if `Result` is false/undefined)
- **Dump variables for debugging:** `hrun -d my_script.hdev` or `hrun -d=MyVar my_script.hdev`
- **Add external procedure paths:** `hrun -p ./my_procedures my_script.hdev`
- **JIT-compiled execution:** `hrun -j my_script.hdev`
- **Threaded execution:** `hrun -t my_procedure my_script.hdev` (script must have `init` and `cleanup` procedures)
- **Remote debugging:** `hrun --debug 57786 my_script.hdev`
- **Strict mode:** `hrun -P my_script.hdev` (fails on invalid/unresolved lines instead of ignoring them)

## Authoring guidelines for `hrun`-compatible scripts

- Avoid interactive operators (`dev_open_dialog`, `stop`) — they block headless execution.
- Use `-b` for buffered (invisible) windows when no display is available.
- Use `-D` to inject global control variables as runtime parameters. Design `main()` with overridable globals at the top so they can be set via `-D`.
- Use `-c` to assert a result variable for CI/testing workflows (exit code reflects pass/fail).
- For threaded mode (`-t`), the script must define `init` and `cleanup` procedures that run in the main thread before and after the threaded procedures. In threaded mode, `-c` and `-d` operate on global control variables rather than local variables.
