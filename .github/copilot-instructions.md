# Copilot instructions

## Build, test, and formatting

- There is no build system or configured test suite in this repository.
- Format Lua files with `stylua lua/`; check formatting without changing files with `stylua --check lua/`. Formatting is configured by `.stylua.toml` (2-space indentation, Unix line endings).
- No single-test command is configured because there are no test files or test runner.

## Architecture

- This is a Lua Neovim plugin. `lua/supermaven-nvim/init.lua` is the plugin entry point: it merges configuration, installs completion keymaps and user commands, registers the optional nvim-cmp source, and starts the service.
- `api.lua` and `commands.lua` expose lifecycle and account actions both as Lua functions and `:Supermaven*` commands. `config.lua` owns defaults and merges user options into them.
- `document_listener.lua` watches buffer, cursor, and insert-mode events. It forwards document changes to `binary/binary_handler.lua`, which manages the bundled `sm-agent` process, sends JSON-line state updates, tracks response states, and asks `textual.lua` to interpret streamed response items.
- Completion display and acceptance are handled by `completion_preview.lua` using Neovim extmarks. `cmp.lua` adapts the active preview into an nvim-cmp completion item; nvim-cmp is optional.
- `binary/binary_fetcher.lua` locates or downloads the platform-specific agent. `util.lua` centralizes buffer/text helpers and Neovim-version compatibility; `logger.lua` handles plugin logging.

## Repository conventions

- Keep modules under `lua/supermaven-nvim/` and require them with the `supermaven-nvim.*` module name. Modules generally export an `M` table; stateful service and preview objects use colon-methods.
- The completion agent communicates asynchronously over stdio using newline-delimited JSON. The agent prefixes response lines with `SM-MESSAGE `; preserve the protocol and state-ID flow when changing process or completion handling.
- Configuration defaults live in `config.lua` and are merged with `vim.tbl_deep_extend("force", ...)`. Add option defaults there and consume them through the config module. The README is the user-facing reference for setup options, commands, and the Lua API.
- Filetype exclusions accept either a map of filetype-to-`true` values or a list of filetypes. Respect the existing checks when adjusting completion eligibility.
- Follow `.stylua.toml`: two-space indentation, Unix line endings, and the configured automatic quote preference.
