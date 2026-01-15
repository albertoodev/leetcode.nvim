# AGENTS.md

Guidelines for AI agents working on leetcode.nvim.

## Build/Lint/Test

No CI/CD. Manual validation only.

**Formatting:**
```bash
stylua lua/ --check  # Check formatting
stylua lua/          # Auto-format
```
Config in `.stylua.toml`: 100 cols, 4 spaces, double quotes, always parentheses.

**Linting:**
```bash
nvim -l /dev/stdin <<'EOF'
vim.api.nvim_set_hl(0, "Error", {undercurl=true})
vim.lsp.start({
  name = "lua-language-server",
  cmd = {"lua-language-server"},
  settings = {Lua = {diagnostics = {globals = {"vim", "require"}}}}
})
vim.lsp.buf.document_symbol()
EOF
```

**Manual testing:**
```bash
nvim --headless -u NONE -c "set rtp+=." -c "lua require('leetcode').setup()" -c "qa"
```

## Code Style

**Imports:** `require()` at top, external deps first, local references.
```lua
local curl = require("plenary.curl")
local log = require("leetcode.logger")
local config = require("leetcode.config")
```

**Module pattern:**
```lua
---@class lc.ModuleName
local M = {}

function M.public_fn() end

---@private
local function private_fn() end

return M
```

**Type annotations (EmmyLua):** Required for all functions.
```lua
---@param title_slug string
---@param lang lc.lang
---@return integer|nil
function M.detect_duplicate_question(title_slug, lang) end

---@alias lc.lang
---| "cpp"
---| "python3"
---| "javascript"
```

**Naming:** `snake_case` for vars/funcs, `PascalCase` for types/classes.

**Indentation:** 4 spaces, double quotes, 100 char limit.

**Error handling:** Use `pcall()`, return `(result, err)`, `assert()` for preconditions.
```lua
local ok, res = pcall(fn)
if not ok then log.error(res); return nil, {msg=res} end
assert(vim.fn.has("nvim-0.9.0") == 1, "Neovim >= 0.9.0 required")
```

**Logging:**
```lua
log.debug("msg")  -- Debug
log.info("msg")   -- Info
log.warn("msg")   -- Warning
log.error("msg")  -- Error
```

**Table operations:** Prefer `vim.tbl_*` functions.
```lua
vim.tbl_filter(function(item) return condition end, tbl)
vim.tbl_map(function(item) return transform(item) end, tbl)
vim.tbl_deep_extend("force", base, override)
```

**API patterns:** Support sync and callback modes.
```lua
utils.query(query, {}, {
    callback = function(res, err) end,
    endpoint = urls.auth,
})

-- Sync
local res, err = utils.query(query, {}, { endpoint = urls.auth })
```

**Auth guard:**
```lua
if not config.auth.is_signed_in then error("not signed in") end
```

**Current question lookup:**
```lua
local tabp = vim.api.nvim_get_current_tabpage()
local tabs = utils.question_tabs()
local tab = vim.tbl_filter(function(t) return t.tabpage == tabp end, tabs)[1]
```

**Global state:** `_Lc_state` (menu, questions, etc.)

## Project Structure

- `lua/leetcode.lua` - Entry point
- `lua/leetcode/` - Core modules (api/, config/, cache/, command/, parser/, picker/, runner/, theme/)
- `lua/leetcode-ui/` - UI components
- `lua/leetcode-plugins/` - Optional plugins (cn, non_standalone)

## Key Conventions

1. No comments unless explaining complex logic
2. Use `vim.NIL` for API comparison
3. Access hooks via `config.user.hooks[event_name]`
4. Validate config before use
5. Wrap async callbacks with `vim.schedule_wrap()`
6. Use `log.debug()` for tracing API calls
7. Auth guard: `if not config.auth.is_signed_in then error(...) end`
8. Use `config.domain` for CN/global switching

## Configuration Access

```lua
local config = require("leetcode.config")
config.user      -- User config (merged defaults + user opts)
config.lang      -- Current language
config.auth      -- User auth status
config.storage   -- Storage paths
```
