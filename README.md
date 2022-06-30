<div align="center">

  <h1>refactoring.nvim</h1>
  <h5>The Refactoring library based off the Refactoring book by Martin Fowler</h5>
  <h6>'If I use an environment that has good automated refactorings, I can trust those refactorings' - Martin Fowler</h6>

[![Lua](https://img.shields.io/badge/Lua-blue.svg?style=for-the-badge&logo=lua)](http://www.lua.org)
[![Neovim Nightly](https://img.shields.io/badge/Neovim%20Nightly-green.svg?style=for-the-badge&logo=neovim)](https://neovim.io)
![Work In Progress](https://img.shields.io/badge/Work%20In%20Progress-orange?style=for-the-badge)

</div>

## Table of Contents

- [Installation](#installation)
  - [Requirements](#requirements)
  - [Setup Using Packer](#packer)
- [Features](#features)
  - [Supported Languages](#supported-languages)
  - [Refactoring Features](#refactoring-features)
  - [Debug Features](#debug-features)
- [Configuration](#configuration)
  - [Configuration for Refactoring Operations](#config-refactoring)
    - [Using Direct Remaps](#config-refactoring-direct)
    - [Using Built-In Neovim Selection](#config-refactoring-builtin)
    - [Using Telescope](#config-refactoring-telescope)
  - [Configuration for Debug Operations](#config-debug)
    - [Customizing Printf and Print Var Statements](#config-debug-stringification)
      - [Customizing Printf Statements](#config-debug-stringification-printf)
      - [Customizing Print Var Statements](#config-debug-stringification-print-var)
  - [Configuration for Type Prompt Operations](#config-prompt)

## Installation<a name="installation"></a>

### Requirements<a name="requirements"></a>

- **Neovim Nightly**
- Treesitter
- Plenary

### Setup Using Packer<a name="packer"></a>

```lua
use {
    "ThePrimeagen/refactoring.nvim",
    requires = {
        {"nvim-lua/plenary.nvim"},
        {"nvim-treesitter/nvim-treesitter"}
    }
}
```

## Features<a name="features"></a>

### Supported Languages<a name="supported-languages"></a>

Given that this is a work in progress, the languages supported for the
operations listed below is **constantly changing**. As of now, these languages
are supported (with individual support for each function may vary):

- TypeScript
- JavaScript
- Lua
- C/C++
- Golang
- Python
- Java
- PHP
- Ruby

### Refactoring Features<a name="refactoring-features"></a>

- Support for various common refactoring operations
  - **106: Extract Function**
    - In visual mode, extracts the selected code to a separate function
    - Optionally prompts for function param types and return types (see
      [configuration for type prompt operations](#config-prompt))
    - Also possible to Extract Block.
    - Both Extract Function and Extract Block have the capability to extract to
      a separate file.
  - **119: Extract Variable**
    - In visual mode, extracts occurences of a selected expression to its own
      variable, replacing occurences of that expression with the variable
  - **123: Inline Variable**
    - Inverse of extract variable
    - Replaces all occurences of a variable with its value
    - Can be used in normal mode or visual mode
      - Using this function in normal mode will automatically find the variable
        under the cursor and inline it
      - Using this function in visual mode will find the variable(s) in the
        visual selection.
        - If there is more than one variable in the selection, the plugin will
          prompt for which variable to inline,
        - If there is only one variable in the visual selection, it will
          automatically inline that variable

### Debug Features<a name="debug-features"></a>

- Also comes with various useful features for debugging
  - **Printf:** Automated insertion of print statement to mark the calling of a
    function
  - **Print var:** Automated insertion of print statement to print a variable
    at a given point in the code. This map can be made with either visual or
    normal mode:
    - Using this function in visual mode will print out whatever is in the
      visual selection.
    - Passing `{ normal = true }` to the function will automatically find the variable
      under the cursor and print it from normal mode without needing visual mode at all
  - **Cleanup:** Automated cleanup of all print statements generated by the
    plugin

## Configuration<a name="configuration"></a>

There are many ways to configure this plugin. Below are some example configurations.

**Setup Function**

No matter which configuration option you use, you must first call the
setup function.

```lua
require('refactoring').setup({})
```

Here are all the available options for the setup function and their defaults:

```lua
require('refactoring').setup({
    prompt_func_return_type = {
        go = false,
        java = false,

        cpp = false,
        c = false,
        h = false,
        hpp = false,
        cxx = false,
    },
    prompt_func_param_type = {
        go = false,
        java = false,

        cpp = false,
        c = false,
        h = false,
        hpp = false,
        cxx = false,
    },
    printf_statements = {},
    print_var_statements = {},
})
```

See each of the sections below for details on each configuration option.

### Configuration for Refactoring Operations<a name="config-refactoring"></a>

#### Using Direct Remaps<a name="config-refactoring-direct"></a>

If you want to make remaps for a specific refactoring operation, you can do so
by configuring the plugin like this:

```lua
-- Remaps for the refactoring operations currently offered by the plugin
vim.api.nvim_set_keymap("v", "<leader>re", [[ <Esc><Cmd>lua require('refactoring').refactor('Extract Function')<CR>]], {noremap = true, silent = true, expr = false})
vim.api.nvim_set_keymap("v", "<leader>rf", [[ <Esc><Cmd>lua require('refactoring').refactor('Extract Function To File')<CR>]], {noremap = true, silent = true, expr = false})
vim.api.nvim_set_keymap("v", "<leader>rv", [[ <Esc><Cmd>lua require('refactoring').refactor('Extract Variable')<CR>]], {noremap = true, silent = true, expr = false})
vim.api.nvim_set_keymap("v", "<leader>ri", [[ <Esc><Cmd>lua require('refactoring').refactor('Inline Variable')<CR>]], {noremap = true, silent = true, expr = false})

-- Extract block doesn't need visual mode
vim.api.nvim_set_keymap("n", "<leader>rb", [[ <Cmd>lua require('refactoring').refactor('Extract Block')<CR>]], {noremap = true, silent = true, expr = false})
vim.api.nvim_set_keymap("n", "<leader>rbf", [[ <Cmd>lua require('refactoring').refactor('Extract Block To File')<CR>]], {noremap = true, silent = true, expr = false})

-- Inline variable can also pick up the identifier currently under the cursor without visual mode
vim.api.nvim_set_keymap("n", "<leader>ri", [[ <Cmd>lua require('refactoring').refactor('Inline Variable')<CR>]], {noremap = true, silent = true, expr = false})
```

Notice that these maps (except the last two) are **visual mode** remaps, and
that ESC is pressed before executing the command. As of now, these are both
necessary for the plugin to work.

#### Using Built-In Neovim Selection<a name="config-refactoring-builtin"></a>

You can also set up the plugin to prompt for a refactoring operation to apply
using Neovim's built in selection API. Here is an example remap to demonstrate
this functionality:

```lua
-- prompt for a refactor to apply when the remap is triggered
vim.api.nvim_set_keymap(
    "v",
    "<leader>rr",
    ":lua require('refactoring').select_refactor()<CR>",
    { noremap = true, silent = true, expr = false }
)
```

This remap should also be made in **visual mode**, or functionality for some
refactors will not work properly.

#### Using Telescope<a name="config-refactoring-telescope"></a>

If you would prefer to use Telescope to choose a refactor when you're in visual
mode, you can do so use using the **Telescope extension.** Here is an example
config for this setup:

```lua
-- load refactoring Telescope extension
require("telescope").load_extension("refactoring")

-- remap to open the Telescope refactoring menu in visual mode
vim.api.nvim_set_keymap(
	"v",
	"<leader>rr",
	"<Esc><cmd>lua require('telescope').extensions.refactoring.refactors()<CR>",
	{ noremap = true }
)
```

### Configuration for Debug Operations<a name="config-debug"></a>

Finally, you can configure remaps for the debug operations of this plugin like
this:

```lua
-- You can also use below = true here to to change the position of the printf
-- statement (or set two remaps for either one). This remap must be made in normal mode.
vim.api.nvim_set_keymap(
	"n",
	"<leader>rp",
	":lua require('refactoring').debug.printf({below = false})<CR>",
	{ noremap = true }
)

-- Print var

-- Remap in normal mode and passing { normal = true } will automatically find the variable under the cursor and print it
vim.api.nvim_set_keymap("n", "<leader>rv", ":lua require('refactoring').debug.print_var({ normal = true })<CR>", { noremap = true })
-- Remap in visual mode will print whatever is in the visual selection
vim.api.nvim_set_keymap("v", "<leader>rv", ":lua require('refactoring').debug.print_var({})<CR>", { noremap = true })

-- Cleanup function: this remap should be made in normal mode
vim.api.nvim_set_keymap("n", "<leader>rc", ":lua require('refactoring').debug.cleanup({})<CR>", { noremap = true })
```

#### Customizing Printf and Print Var Statements<a name="config-debug-stringification"></a>

It is possible to override the statements used in the printf and print var
functionalities.

##### Customizing Printf Statements<a name="config-debug-stringification-printf"></a>

You can add to the printf statements for any language by adding something like
the below to your configuration:

```lua
require('refactoring').setup({
  -- overriding printf statement for cpp
  printf_statements = {
      -- add a custom printf statement for cpp
      cpp = {
          'std::cout << "%s" << std::endl;'
      }
  }
})
```

In any custom printf statement, it is possible to optionally add a max of
**one %s** pattern, which is where the debug path will go. For an example custom
printf statement, go to [this folder](lua/refactoring/tests/debug/printf),
select your language, and click on `multiple-statements/printf.config`.

##### Customizing Print Var Statements<a name="config-debug-stringification-print-var"></a>

The print var functionality can also be extended for any given language,
as shown below:

```lua
require('refactoring').setup({
  -- overriding printf statement for cpp
  print_var_statements = {
      -- add a custom print var statement for cpp
      cpp = {
          'printf("a custom statement %%s %s", %s)'
      }
  }
})
```

In any custom print var statement, it is possible to optionally add a max of
**two %s** patterns, which is where the debug path and the actual variable
reference will go, respectively. To add a literal "%s" to the string, escape the
sequence like this: `%%s`. For an example custom print var statement, go to
[this folder](lua/refactoring/tests/debug/print_var), select your language, and
view `multiple-statements/print_var.config`.

**Note:** for either of these functions, if you have multiple custom
statements, the plugin will prompt for which one should be inserted. If you
just have one custom statement in your config, it will override the default
automatically.

### Configuration for Type Prompt Operations<a name="config-prompt"></a>

For certain languages like Golang, types are required for functions that return
an object(s) and parameters of functions. Unfortunately, for some parameters
and functions there is no way to automatically find their type. In those
instances, we want to provide a way to input a type instead of inserting a
placeholder value.

By default all prompts are turned off. The configuration below shows how to
enable prompts for all the languages currently supported.

```lua
require('refactoring').setup({
    -- prompt for return type
    prompt_func_return_type = {
        go = true,
        cpp = true,
        c = true,
        java = true,
    },
    -- prompt for function parameters
    prompt_func_param_type = {
        go = true,
        cpp = true,
        c = true,
        java = true,
    },
})
```
