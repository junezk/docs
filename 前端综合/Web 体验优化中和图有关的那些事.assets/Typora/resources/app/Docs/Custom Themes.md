## Change Themes

Typora has 5~6 built-in themes. Changing writing theme can be done by selecting theme under the `theme` menu bar. You could also download, install, modify or write your own custom theme to stylize Typora.

Typora use CSS to stylish all contents, each theme under `theme` menu is one `.css` files under "typora's theme folder". So, briefly speaking, you could add/modify themes by adding/modifying correspond css files under "typora's theme folder".

## Get Typora Themes

We have an office website [Typora Theme Gallery](http://theme.typora.io) for designers/developers to share their custom themes with others. You could download theme from there.

## Install Custom Themes

1. Open Theme Folder. (see instructions below)
2. Copy or move `.css` file and related resources, like fonts or images, into the newly opened folder.
3. Restart typora, then select it from `Themes` menu.

## Open Theme Folder

### macOS

Open preference panel by <code>cmd+`</code>, then click "Open Theme Folder"

<img src="img/Snip20160921_1.png" style="zoom:50%" />

### Windows/Linux

Open preference panel from `File` → `Preference` from menubar, then click "Open Theme Folder":

<img src="img/Snip20160921_2.png" style="zoom:50%" />

## Modify Current Styles

Sometimes, you may just want to change font family for all themes, for change font-color for headings for a specific themes. In this case, you do not need to copy/modify a whole exiting css file, just [Add Custom CSS](http://support.typora.io/Add-Custom-CSS/) is enough. In brief:

- Create & write a `base.user.css` under theme folder, the css rules inside it will be applied to all themes.
- Create & write a `{theme-css-name}.user.css` under theme folder, the css rules inside it will only be applied to theme file `{theme-css-name}.css`.

Please note that the built-in CSS theme files will get overwritten completely on typora version up, so, write your custom css into `*.user.css`, instead of the existing files, will prevent your modifications from being lost after update.

## Write My Own Theme

Please refer to [Write Custom Theme for Typora](http://theme.typora.io/doc/Write-Custom-Theme/).