# Autocompletion

## Introduction

The plugin responsable for autocompletion is [nvim-cmp](https://github.com/hrsh7th/nvim-cmp). This plugin is designed to be unopinionated and modular. What this means for us (the users) is that we have to assemble various pieces to get a good experience.

When using a preset lsp-zero will configure nvim-cmp for you. This config will include a "completion source" to get data from your LSP servers. It will create keybindings to control de completion menu. Setup a snippet engine ([luasnip](https://github.com/L3MON4D3/LuaSnip)) to expand the snippet that come from your LSP server. Finally, change the "formatting" of the completion items, it will add a label that tells the name of the source for that item.

Here is the code lsp-zero will setup for you.

```lua
local cmp = require('cmp')
local cmp_select_opts = {behavior = cmp.SelectBehavior.Select}

cmp.setup({
  sources = {
    {name = 'nvim_lsp'},
  },
  mapping = {
    ['<C-y>'] = cmp.mapping.confirm({select = true}),
    ['<C-e>'] = cmp.mapping.abort(),
    ['<C-u>'] = cmp.mapping.scroll_docs(-4),
    ['<C-d>'] = cmp.mapping.scroll_docs(4),
    ['<Up>'] = cmp.mapping.select_prev_item(cmp_select_opts),
    ['<Down>'] = cmp.mapping.select_next_item(cmp_select_opts),
    ['<C-p>'] = cmp.mapping(function()
      if cmp.visible() then
        cmp.select_prev_item(cmp_select_opts)
      else
        cmp.complete()
      end
    end),
    ['<C-n>'] = cmp.mapping(function()
      if cmp.visible() then
        cmp.select_next_item(cmp_select_opts)
      else
        cmp.complete()
      end
    end),
  },
  snippet = {
    expand = function(args)
      require('luasnip').lsp_expand(args.body)
    end,
  },
  window = {
    documentation = {
      max_height = 15,
      max_width = 60,
    }
  },
  formatting = {
    fields = {'abbr', 'menu', 'kind'},
    format = function(entry, item)
      local short_name = {
        nvim_lsp = 'LSP',
        nvim_lua = 'nvim'
      }

      local menu_name = short_name[entry.source.name] or entry.source.name

      item.menu = string.format('[%s]', menu_name)
      return item
    end,
  },
})
```

## Preset settings

You can tweak the behavior of nvim-cmp using a preset. You can control what lsp-zero is going to do with nvim-cmp. The [minimal](https://github.com/VonHeikemen/lsp-zero.nvim/blob/v2.x/doc/md/api-reference.md#minimal) preset has the following settings:

```lua
manage_nvim_cmp = {
  set_sources = 'lsp',
  set_basic_mappings = true,
  set_extra_mappings = false,
  use_luasnip = true,
  set_format = true,
  documentation_window = true,
}
```

If you want to know the details of each property go to [the api reference](https://github.com/VonHeikemen/lsp-zero.nvim/blob/v2.x/doc/md/api-reference.md#manage_nvim_cmp). But what this means is you can do stuff like this.

```lua
local lsp = require('lsp-zero').preset({
  manage_nvim_cmp = {
    set_sources = 'recommended'
  }
})
```

In this particular example I'm saying that I want to setup all the "recommended" sources for nvim-cmp.

## Recommended sources

In nvim-cmp a "source" is a plugin (a neovim plugin) that provides the actual data displayed in the completion menu. If you set `manage_nvim_cmp.set_sources` to the string `'recommended'`, lsp-zero will try to setup the following sources (if they are installed):

* [cmp-buffer](https://github.com/hrsh7th/cmp-buffer): provides suggestions based on the current file.

* [cmp-path](https://github.com/hrsh7th/cmp-path): gives completions based on the filesystem.

* [cmp_luasnip](https://github.com/saadparwaiz1/cmp_luasnip): it shows snippets in the suggestions.

* [cmp-nvim-lsp](https://github.com/hrsh7th/cmp-nvim-lsp): show data send by the language server.

## Keybindings

### Basic mappings

These are the keybindings you get when you enable `manage_nvim_cmp.set_basic_mappings`. They are meant to follow Neovim's default whenever possible.

* `<Ctrl-y>`: Confirms selection.

* `<Ctrl-e>`: Cancel the completion.

* `<Down>`: Navigate to the next item on the list.

* `<Up>`: Navigate to previous item on the list.

* `<Ctrl-n>`: If the completion menu is visible, go to the next item. Else, trigger completion menu.

* `<Ctrl-p>`: If the completion menu is visible, go to the previous item. Else, trigger completion menu.

* `<Ctrl-d>`: Scroll down in the item's documentation.

* `<Ctrl-u>`: Scroll up in the item's documentation.

### Extra mappings

These are the keybindings you get when you enable `manage_nvim_cmp.set_extra_mappings`. These enable tab completion and navigation between snippet placeholders.

* `<Ctrl-f>`: Go to the next placeholder in the snippet.

* `<Ctrl-b>`: Go to the previous placeholder in the snippet.

* `<Tab>`: Enables completion when the cursor is inside a word. If the completion menu is visible it will navigate to the next item in the list.

* `<Shift-Tab>`: When the completion menu is visible navigate to the previous item in the list.

## Customizing nvim-cmp

What I actually recommend is using `cmp` directly. Let lsp-zero do the "minimal" config, then use the module `cmp` any extra feature you want.

Make sure you setup `cmp` after lsp-zero, so you can override every option properly.

```lua
local lsp = require('lsp-zero').preset({})

lsp.on_attach(function(client, bufnr)
  lsp.default_keymaps({buffer = bufnr})
end)

lsp.setup()

local cmp = require('cmp')

cmp.setup({
  ---
  -- Add you own config here...
  ---
})
```

### Use Enter to confirm completion

You'll want to add an entry to the `mapping` option of nvim-cmp. You can assign `<CR>` to the function `cmp.mapping.confirm`.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  mapping = {
    ['<CR>'] = cmp.mapping.confirm({select = false}),
  }
})
```

In that example `Enter` will only confirm the selected item. You need to select the item before pressing enter.

If you want to confirm without selecting the item, use this.

```lua
['<CR>'] = cmp.mapping.confirm({select = true}),
```

### Adding a source

Yes, I know I have a thing that configures this for you but at this point it is only there for backwards compatibility with the `v1.x` branch. There is no "intuitive" way to override that config, so advise you do this yourself to gain better control.

This is what the "recommended" configuration for `sources` looks like.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  sources = {
    {name = 'path'},
    {name = 'nvim_lsp'},
    {name = 'buffer', keyword_length = 3},
    {name = 'luasnip', keyword_length = 2},
  }
})
```

Once you have this you can change the priority by changing the order in the list. You can delete sources or you can add more.

### Add an external collection of snippets

By default luasnip is only configured to expand snippets. Also, the only snippets you get will come from your LSP server. If you want to load **custom snippets** into the completion menu you need add [cmp_luasnip](https://github.com/saadparwaiz1/cmp_luasnip) as a source in nvim-cmp.

Now, we don't need to write our own snippets, we can download a collection like [friendly-snippets](https://github.com/rafamadriz/friendly-snippets) and then parse them using a luasnip loader.

Here is the code you would need to load `friendly-snippets` into nvim-cmp.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')
local cmp_action = require('lsp-zero').cmp_action()

require('luasnip.loaders.from_vscode').lazy_load()

cmp.setup({
  sources = {
    {name = 'nvim_lsp'},
    {name = 'luasnip'},
  },
  mapping = {
    ['<C-f>'] = cmp_action.luasnip_jump_forward(),
    ['<C-b>'] = cmp_action.luasnip_jump_backward(),
  }
})
```

If you want to use [honza/vim-snippets](https://github.com/honza/vim-snippets), you'll have to call a different loader.

```lua
require('luasnip.loaders.from_snipmate').lazy_load()
```

### Preselect first item

Make the first item in completion menu always be selected.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  preselect = 'item',
  completion = {
    completeopt = 'menu,menuone,noinsert'
  },
})
```

### Basic completions for Neovim's lua api

Add the [cmp-nvim-lua](https://github.com/hrsh7th/cmp-nvim-lua) source.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  sources = {
    {name = 'nvim_lsp'},
    {name = 'nvim_lua'},
  },
  mapping = {
    ['<C-f>'] = cmp_action.luasnip_jump_forward(),
    ['<C-b>'] = cmp_action.luasnip_jump_backward(),
  }
})
```

### Enable "Super Tab"

If you are the kind of person who likes to do everything with the `Tab` key, try this:

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')
local cmp_action = require('lsp-zero').cmp_action()

cmp.setup({
  mapping = {
    ['<Tab>'] = cmp_action.luasnip_supertab(),
    ['<S-Tab>'] = cmp_action.luasnip_shift_supertab(),
  }
})
```

If the completion menu is visible it will navigate to the next item in the list. If the cursor on the trigger of a snippet it'll expand it. If the cursor can jump to a snippet placeholder, it moves to it. If the cursor is in the middle of a word that doesn't trigger a snippet it displays the completion menu. Else, it acts like a regular `Tab` key.

### Regular tab complete

Trigger the completion menu when the cursor is inside a word. If the completion menu is visible it will navigate to the next item in the list. If the line is empty it uses the fallback.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')
local cmp_action = require('lsp-zero').cmp_action()

cmp.setup({
  mapping = {
    ['<Tab>'] = cmp_action.tab_complete(),
    ['<S-Tab>'] = cmp_action.select_prev_or_fallback(),
  }
})
```

### Invoke completion menu manually

In case you want to trigger the completion menu manually using `Control + Space`.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  completion = {
    autocomplete = false
  },
  mapping = {
    ['<C-Space>'] = cmp.mapping.complete(),
  }
})
```

### Adding borders to completion menu

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  window = {
    completion = cmp.config.window.bordered(),
    documentation = cmp.config.window.bordered(),
  }
})
```

### Change formatting of completion item

Here is a basic example that adds icons based on the name of the source.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  formatting = {
    -- changing the order of fields so the icon is the first
    fields = {'menu', 'abbr', 'kind'},

    -- here is where the change happens
    format = function(entry, item)
      local menu_icon = {
        nvim_lsp = 'λ',
        luasnip = '⋗',
        buffer = 'Ω',
        path = '🖫',
        nvim_lua = 'Π',
      }

      item.menu = menu_icon[entry.source.name]
      return item
    end,
  },
})
```

### lsp-kind

[lspkind.nvim](https://github.com/onsails/lspkind.nvim) should work too.

```lua
-- Make sure you setup `cmp` after lsp-zero

local cmp = require('cmp')

cmp.setup({
  formatting = {
    fields = {'abbr', 'kind', 'menu'},
    format = require('lspkind').cmp_format({
      mode = 'symbol', -- show only symbol annotations
      maxwidth = 50, -- prevent the popup from showing more than provided characters
      ellipsis_char = '...', -- when popup menu exceed maxwidth, the truncated part would show ellipsis_char instead
    })
  }
})
```

