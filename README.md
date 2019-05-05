# Description
This python script allows you to install dotfiles by following a given installation procedure.
It also supports user input application dependencies and conditional instructions.
This script currently only supports packages from the Arch User Repository (AUR) and uses the packet manager `yay` to install its dependencies. 

# How To
After installation you need to set your dotfiles workspace up.
To do this you need to create a directory with subdirectories for all of your programms that should installed.
These subdirectories all contain the files they require and an `install.json` file which defines the installation procedure.

For example:
```
my_dotfiles
│   
└───application1
│   │   file1
│   │   install.json
│
└───application2
│   │   filerc
│   │   install.json
|
...
```
An `install.json` file defines its dependencies which will be installed during the setup process and the process to restore dotfiles.
To define the setup process, the following instructions are supported:
- `run <any bash command>` - runs a bash command
- `copy <path/to/file/from> <path/to/file/to>` - copies a file or directory to another location
- `append <path/to/file/from> <path/to/file/to>` - appends the content of a file to the end of another file
- `replace <path/to/file> <stringToBeReplaced> <replacement>` - replaces all matches of a string in a file (use %20 instead of a space)

For user input one can use an json object with the following attributes as a unstruction:
- `question` - the text the user will see on execution
- `variable` - a variable name that is used to store the answer (recommended structure: {variablename})
- `answer` - another object that holds possible answers as keys and arrays of instructions as values (use '*' as key for any answer)

Such a file could look like this.
```json
{
    "dependencies": [
        "git"
    ],
    "install": [
        "copy config_base ~/.config/app/config",
        "append shortcuts_applications ~/.config/app/config",
        {
            "question": "Install extra shortcuts? (yes/no)",
            "variable": "{extra_shortcuts}",
            "answer": {
                "yes": [
                    "append shortcuts_extra ~/.config/app/config"
                ],
                "no": []
            }
        },
        {
            "question": "Please enter the path to your wallpaper.",
            "variable": "{wallpaper}",
            "answer": {
                "*": [
                    "replace ~/.config/app/config [WALLPAPER] {wallpaper}"
                ]
            }
        }
    ]
}
```

Additional notices regarding user input and variables:

1. Variables do not use any scoping. So if a variable is set, it can be used in any of the future instructions - even in scripts for other applications. If two user input blocks use the same variable and one block has already been executed, the second answer will autofill to the answer of the first question on runtime.

2. If there is no answer for one user input defined, the script will search for the instructions for the key `*`. If even this key is not given, the script will re-run the user input block until a valid answer has been given.

# Installation
To install this application you can use the provided `install.sh` script which copies the python script to `/usr/local/bin`.
```bash
$ sudo ./install.sh
```
Afterwards you can run the python script by typing `sudo dotfiles_make </home/directory>` after changing directory to your dotfiles.

# Examples
An example can be found [here](https://github.com/mklpiening/dotfiles).