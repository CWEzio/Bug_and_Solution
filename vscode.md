# Settings
## Relative line number
`vscode-vim` provided a quiet handy feature, `smart relative line`, that use absolute line number in insert mode and relative line number otherwise.

# keybinding
## Window control
- use ``Ctrl + ` `` to focus on the terminal
- use `Ctrl + 1` to focus on the editor  
    > when there are multiple editor windows, use, for example, `Ctrl + 2` to refer to other windows. In other words, `Ctrl + index` to focus on the editor
- Navigate between groups using `Ctrl + PageDown` and `Ctrl + PageUp` (also suitable for terminals)
- `Ctrl + Tab` to change between tabs in a group.
- `Ctrl + (PgUp / PgDn)` to cycle through tabs in a group
- use `alt + number` to navigate between different tabs
- Use the command in the command palette, like `close all other editors in group`

## Customize keybindings
1. Open keyboard shortcuts editor:
    - `Ctrl+Shift+P` to open the command palette
    - In the command palette, select command `Preferences: Open Keyboard Shortcuts`
2. Modify keybindings:
    - Search for a command, right-click and choose `Change Keybinding`

You can also modify the keybindings in the `keybindings.json` file. You can open this file by command `Preferences: Open Keyboard Shortcuts (JSON)`. You can see all the commands that you have customized in this json file.
> The `-` symbol in front of a command means that you've disabled this command. For example: 
> ```json
>    {
>    "key": "ctrl+p",
>    "command": "-extension.vim_ctrl+p",
>    }
> ```
> The command `extension.vim_ctrl+p` is disabled.

## Show hover information
Sometimes you want to show hover information, to remind you the function definition etc., as:
<img src="./asset/vscode/hover_information.png">
In vim mode, simple put the cursor on top of the function does not show the hover information, you need to:
1. `ctrl + shift + p` to open control palette
2. search for `show or focus hover`
> note that the hot key for `show or focus hover` is `ctrl + K ctrl + I`. However, I am using `vscode-vim` and `ctrl + K` does not work.

## Trigger parameter hint
<img src="./asset/vscode/parameter_hint.png">

- `ctrl + shift + space`

## Trigger autocomplete
- `ctrl + space`

## Reject autocomplete suggestion
- `Escape`

# python
## Select notebook kernel
To select a kernel,  click the `select kernel` button or type `select kernel` in the search palette. 

If you want to select an existing python enviornment, but cannot see it in the list:
1. `ctrl + shift + p`, type `select python interpreter`
2. choose that python environment as the interpreter
3. then you should be able to see that environment in the selection list.

After you select the environment, if the kernel is not already installed in that environment, then vscode will automatically install it. 

## Set PythonPath
Check [this answer for details](https://stackoverflow.com/questions/53653083/how-to-correctly-set-pythonpath-for-visual-studio-code)
1. Open the `settings.json` file and insert:
    ```
    "terminal.integrated.env.windows": {
    "PYTHONPATH": "${workspaceFolder}/src;${workspaceFolder}/tests"
    },
    "python.envFile": "${workspaceFolder}/.env",
    ```
    > If you only cares about whether the editor can correctly find the package path, only setting `python.envFile` is enough.
2. create `.env` file and put the pythonpath in this file. For example:
    ```
    PYTHONPATH=/home/chenwang/VCD/softgym:${PYTHONPATH}
    PYFLEXROOT="/home/chenwang/VCD/softgym/PyFlex"
    PYTHONPATH="${PYFLEXROOT}/bindings/build:${PYTHONPATH}"
    LD_LIBRARY_PATH=${PYFLEXROOT}/external/SDL2-2.0.4/lib/x64:$LD_LIBRARY_PATH
    ```
    > You can also put other path to this file.

## Debug. Add command line argument
```json
{
    "name": "Python: main_generate_data",
    "type": "python",
    "request": "launch",
    "program": "VCD/main.py",
    "console": "integratedTerminal",
    "justMyCode": true,
    "args": ["--gen_data", "1", "--dataf", "./data/vcd"]
}
```
`args` specifies arguments to pass to the Python program. Each element of the argument string that's separated by a space should be contained within quotes.

## Compare different files with vscode
You can compare/diff two files with vscode. 
1. Open the first file and focus on its window.
2. Open command palette, use command `File: Compare Active File With`
3. Select the second file to compare.
- You can change between inline view and side-to-side view by use command `Compare: Toggle Inline View`.
- The inline view shows the changes made in the second file from the first view.
- Note that in the side-to-side view, two sides are in the same window. Therefore, switch between two sides can not be done with keyboard shortcuts like `C-w h`. Instead, you can use the following commands:
    1. `workbench.action.compareEditor.focusPrimarySide`
    2. `workbench.action.compareEditor.focusSecondarySide`
    3. `workbench.action.compareEditor.focusOtherSide`

    I bind `ctrl+alt+n` with `workbench.action.compareEditor.focusOtherSide` to switch between different sides.
    
You can also compare two files with mouse:
1. Right-click the first file to be compared in the file explorer and select `Select for Compare`
2. Right-click the second file to be compared in the file explorer and select `Compare with Selected`

# Markdown
- [Official documentation](https://code.visualstudio.com/docs/languages/markdown) of supported markdown features.

## Markdown math
`vscode` use `KaTeX` to render math. Here is a list of [supported functions](https://katex.org/docs/supported.html).
- [KaTex support table](https://katex.org/docs/support_table)
- [Katex support function](https://katex.org/docs/supported)
### Katex
Math
- Different style
    - $\sum_{i=1}^n$
    - $\sum\limits_{i=1}^n$
    - $\displaystyle\sum_{i=1}^n$

# Copilot
[Official guide for vscode](https://docs.github.com/en/copilot/using-github-copilot/getting-started-with-github-copilot?tool=vscode#prerequisites-2)
## Keybindings
- Start inline chat `Ctrl + K N` (customized)
- Accept suggestion `Tab`
- See alternative suggestions `Alt + ]` for next suggestion, `Alt + [` for previous suggestion.
- `Ctrl + →` accept next word.
- `Ctrl + Enter` to see multiple suggestions in a new tab.

# SSH
## Drag and drop file
You can drag a file from your local machine to the explorer to copy the file to the remote machine. You can also do the reverse things. For more information, refer to [this release note](https://github.com/microsoft/vscode-docs/blob/main/remote-release-notes/v1_36.md).
> Make sure you have permission to these files.
