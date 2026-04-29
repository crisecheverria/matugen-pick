# matugen-pick

A 25-line bash wrapper that turns picking a wallpaper into one command on
macOS: browse a folder of images with [fzf](https://github.com/junegunn/fzf)
(kitty image previews), set the pick as your desktop wallpaper via
[desktoppr](https://github.com/scriptingosx/desktoppr), then hand off to
[matugen](https://github.com/InioX/matugen) which prompts for a source color
and regenerates Material You themes for any apps you've templated.

## Demo

https://github.com/user-attachments/assets/d72d92d5-5a6e-439e-871d-23cfaafeaf83

## What does matugen do? (explain like I'm 5)

Imagine you painted a picture and an artist friend said "okay, I'll pick out a
whole set of crayons that match your picture — a dark one for the background,
a bright one for cool stuff, a softer one for less important stuff." That's
matugen.

The flow:

1. You hand it a wallpaper image.
2. It looks at the colors and grabs one as the "main" color (in the demo above,
   it's the warm brown it pulled out of the wallpaper).
3. It uses Google's Material You math — the same thing Android phones do to
   recolor your apps based on your wallpaper — to generate a coordinated set
   of about 30 colors: backgrounds, foregrounds, accents, borders, etc.
4. It writes those colors into config files for whatever apps you've templated,
   so kitty, nvim, etc. suddenly look like they belong with your wallpaper.

`matugen-pick` is just the "browse a folder and pick the wallpaper" front end.
The cool effect comes from also templating your apps to read matugen's output
— see [Setting up matugen targets](#setting-up-matugen-targets-nvim-kitty)
below.

## Requirements

- macOS (uses `desktoppr` for wallpaper, and the kitty image protocol for previews)
- [kitty](https://sw.kovidgoyal.net/kitty/) — for the `kitten icat` image preview
- [fzf](https://github.com/junegunn/fzf)
- [matugen](https://github.com/InioX/matugen)
- [desktoppr](https://github.com/scriptingosx/desktoppr)

For Neovim integration:

- [matugen.nvim](https://github.com/daedlock/matugen.nvim) — reads the
  `colors.json` matugen produces and rebuilds the colorscheme live whenever
  the file changes.

```sh
brew install fzf desktoppr
cargo install matugen   # or: brew install matugen
```

## Install

```sh
install -m 755 matugen-pick ~/.local/bin/
```

Make sure `~/.local/bin` is on your `PATH`.

## Usage

```sh
matugen-pick                     # browses ~/Documents/backgrounds
matugen-pick ~/Pictures/walls    # browse a different folder
```

`fzf` opens with a thumbnail preview pane. Hit `Enter` on a file:

1. `desktoppr` sets it as your macOS desktop wallpaper.
2. `matugen image` prompts you to pick the source color extracted from the
   image, then runs every template in your matugen config.

## Setting up matugen targets (nvim, kitty, …)

The wrapper itself doesn't know what colors you want recoloured — that's
matugen's job, driven by `~/.config/matugen/config.toml` and the templates
it points at. On macOS the matugen config lives at
`~/Library/Application Support/com.InioX.matugen/`; symlinking that to
`~/.config/matugen/` keeps everything tidy:

```sh
mkdir -p ~/Library/Application\ Support
ln -s ~/.config/matugen "$HOME/Library/Application Support/com.InioX.matugen"
```

A minimal `~/.config/matugen/config.toml` that drives both nvim
(via [matugen.nvim](https://github.com/daedlock/matugen.nvim)) and kitty:

```toml
[config]

[templates.nvim]
input_path = "~/.config/matugen/templates/colors.json"
output_path = "~/.config/matugen/colors.json"

[templates.kitty]
input_path = "~/.config/matugen/templates/kitty.conf"
output_path = "~/.config/kitty/matugen-theme.conf"
post_hook = "for s in /tmp/kitty-*; do kitty @ --to \"unix:$s\" load-config; done"
```

The `post_hook` glob handles kitty's auto-suffixed listen sockets — set
`allow_remote_control socket-only` and `listen_on unix:/tmp/kitty` in
`kitty.conf`, and add `include matugen-theme.conf` after your base theme
include so matugen overrides the parts it knows about.

`templates/colors.json` (read by matugen.nvim) and `templates/kitty.conf`
are plain Tera templates — see the
[matugen template docs](https://github.com/InioX/matugen) for the full
list of available color tokens. The
[matugen.nvim README](https://github.com/daedlock/matugen.nvim) lists the
flat MD3 token names it expects in the JSON.

## License

MIT
