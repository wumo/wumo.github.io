---
title: "Custom Configuration for Alacritty "
tags: ["terminal","windows 10","Alacritty"]
---
Custom Configuration for Alacritty (Windows 10) 
Install [Alacritty](https://github.com/jwilm/alacritty).

Launch PowerShell and edit `alacritty.yml`:
```powershell
cd $env:APPDATA\alacritty\
code alacritty.yml # use your favourite editor (here is vscode)
```
<!--more-->

# Color
Modify `alacritty.yml` as following to customize color (ANSI X3.64 color):
```yaml
schemes:
  Monokai: &dark # custom theme name
    # Default colors
    primary:
      background: '0x272822'
      foreground: '0xCACACA'

    # Normal colors
    normal:
      black:   '0x272822'
      red:     '0xA70334'
      green:   '0x74AA04'
      yellow:  '0xB6B649'
      blue:    '0x01549E'
      magenta: '0x89569C'
      cyan:    '0x1A83A6'
      white:   '0xCACACA'

    # Bright colors
    bright:
      black:   '0x7C7C7C'
      red:     '0xF3044B'
      green:   '0x8DD006'
      yellow:  '0xCCCC81'
      blue:    '0x0383F5'
      magenta: '0xa87db8'
      cyan:    '0x58C2E5'
      white:   '0xFFFFFF'

# Colors (Monokai Dark)
colors: *dark # use predefined theme

background_opacity: 0.95

mouse_bindings:
  - { mouse: Right, action: PasteSelection }

selection:
  # When set to `true`, selected text will be copied to the primary clipboard.
  save_to_clipboard: true
```

# Context Menu

Write a `install.cmd` file and run as administrator to register context menu:
```powershell
set menu_name=Alacritty Here
set exe_path=???\alacritty.exe

reg add "HKCU\Software\Classes\Directory\shell\%menu_name%" /v "Icon" /d "\"%exe_path%\"" /f
reg add "HKCU\Software\Classes\Directory\shell\%menu_name%\command" /d "\"%exe_path%\"" /f

reg add "HKCU\Software\Classes\Directory\Background\shell\%menu_name%" /v "Icon" /d "\"%exe_path%\"" /f
reg add "HKCU\Software\Classes\Directory\Background\shell\%menu_name%\command" /d "\"%exe_path%\"" /f

reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\%menu_name%" /v "Icon" /d "\"%exe_path%\"" /f
reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\%menu_name%\command" /d "\"%exe_path%\"" /f
```
Write a `uninstall.cmd` file and run as administrator to unregister context menu:
```powershell
set menu_name=Alacritty Here

reg delete "HKCU\Software\Classes\Directory\shell\%menu_name%" /f
reg delete "HKCU\Software\Classes\Directory\Background\shell\%menu_name%" /f
reg delete "HKCU\Software\Classes\LibraryFolder\Background\shell\%menu_name%" /f
```