---
title: Interactive shell
tags:
  - gcloud
  - interactive
reference url: https://cloud.google.com/sdk/docs/interactive-gcloud
---
reference: https://cloud.google.com/sdk/docs/interactive-gcloud

1. ## Installation and use
    
    The `gcloud` interactive shell is included in the `gcloud beta` components.
    
    1. To check if you have the `gcloud beta` components installed, run the following command:
        
        ```
        gcloud components list
        ```
        
    2. If you don't see the `gcloud beta` components listed, to install the beta components, run the following command:
        
        ```
        gcloud components install beta
        ```
        
    3. To enter the `gcloud` interactive mode, run the following command:
        
        ```
        gcloud beta interactive
        ```
        
        Your usual shell prompt is replaced with the `gcloud` interactive shell prompt `$`.
        
    4. To get auto-suggestions and inline help, start typing a command.
        
    5. To save time when you're working with a command for a while, type the part of the command you'll reuse and then press `F7`. For example, to work with `gcloud compute`, type `gcloud compute` and then press `F7`. You can then type subcommands such as `list` without needing to first type `gcloud compute`. When you're no longer using the command, press `Ctrl-C` and `F7` to clear the context.
        
    6. To exit the interactive shell press `Ctrl-D` or `F9`.
        
    
    ## Auto-completion and help
    
    `gcloud interactive` has auto prompting for commands and flags, and displays inline help snippets in the lower section as you type a command.
    
    Static information, like command and sub-command names, and flag names and enumerated flag values, are auto-completed using dropdown menus.
    
    ![gcloud interactive shell example session](https://cloud.google.com/static/sdk/docs/images/gcloudinteractiveshell.gif)
    
    ## Shortcuts
    
    To accomplish common tasks, you can use the following shortcuts:
    
    |Action|Shortcut|
    |---|---|
    |Complete a file path or resource argument|`Tab`|
    |Refine the dropdown completion menu|Continue typing the command|
    |Scroll through the menu|`Tab`, `Shift+Tab`, or arrow keys|
    |Select a highlighted item or directory|`Space` or `/`|
    |Toggle the active help section, ON when enabled, OFF when disabled|`F2`|
    |Set the context for command input to avoid retyping command prefixes|`F7`|
    |Clear the context for command input|`Ctrl-C` and `F7`|
    |Open a web browser tab or window to display the complete man page for the current command|`F8`|
    |Exit|`F9` or `Ctrl+D`|
    
    ## Bash compatibility
    
    `bash` completion configs, aliases, exports, functions, `set -o` settings, and variables initialized in your `.bashrc` are all available at the interactive command prompt. The interactive command line edit mode is derived from the `set -o emacs` or `set -o vi` setting.