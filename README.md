# Doria TUI

`Doria\Tui` is a terminal user interface library for the Doria Programming
Language. An application keeps its state in one value, changes it in response
to key and resize events, and describes each screen as a tree of elements. The
library lays the tree out in terminal cells and redraws only the cells that
changed. The same source runs in a Linux terminal, a macOS terminal and a
native Windows console.

The library is written in Doria and draws through `Doria\Std\Term`. Application
code never writes an escape sequence.

## Using it

Add the package to an application's `Baton.toml`:

```toml
[dependencies]
"dorialang/tui" = { source = "git", url = "https://github.com/dorialang/doria-tui.git", tag = "v2026.3.1", version = "^2026.3" }
```

## License

Apache License 2.0. See [LICENSE](LICENSE).
