# Doria TUI

`Doria\Tui` is a terminal user interface library for the Doria Programming
Language. An application keeps its state in one value, changes it in response
to key and resize events, and describes each screen as a tree of elements. The
library lays the tree out in terminal cells and redraws only the cells that
changed. The same source runs in a Linux terminal, a macOS terminal and a
native Windows console.

The library is built in public layers: the terminal session and cell surface,
layout and elements, input and focus, and application models. Widgets and
application models written outside the library use the same interfaces as the
built-in ones.

The library is written in Doria and draws through `Doria\Std\Term`. Application
code never writes an escape sequence.

## Using it

Add the package to an application with Baton, naming the release tag to use:

```text
baton add dorialang/tui --source git --url https://github.com/dorialang/doria-tui.git --tag <release-tag>
```

## License

Apache License 2.0. See [LICENSE](LICENSE).
