---
layout: post
title:  "Windows Terminal setup"
date:   2021-05-25 08:44:52 -0400
categories: windows
---

### Windows Terminal
Windows Terminal is a modern feature-rich open source terminal from Microsoft that runs many shells, including PowerShell. The easiest way to install it is from the [Microsoft Store](https://aka.ms/terminal) or via Chocolately with the command `choco install microsoft-windows-terminal`.

Go ahead and open Windows Terminal, which should default to PowerShell. Have a look around, it's already pretty awesome. There's support for multiple tabs, color schemes, custom fonts, custom shells, and more. We are going to customize it even further by setting up git support, customizing the prompt, installing custom fonts, setting the color scheme, and optionally adding a couple of extra tools.

### posh-git and oh-my-posh
posh-git allows git information to be displayed within PowerShell and provides tab completion of git commands and branches.

oh-my-posh is a prompt theme engine for PowerShell.

To install these tools, run the following commands in PowerShell

```ps
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

After installing posh-git and oh-my-posh, your PowerShell profile needs to be configured to load them on startup. You can get the location of your profile by executing `$profile` within PowerShell. Open this file with your favorite text editor, `code $profile`, and add the following lines:

```
Import-Module posh-git
Import-Module oh-my-posh
```

Save the file and keep it open, we'll come back to it later when configuring these tools.

### PowerShell execution policy
At this point if you run a new instance of PowerShell, you may get an error:
```
. : File C:\Users\~\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 cannot be loaded. The
file C:\Users\~\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1 is not digitally signed. You
cannot run this script on the current system. For more information about running scripts and setting execution policy,
see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:3
+ . 'C:\Users\~\Documents\WindowsPowerShell\Microsoft.Powe ...
+   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

PowerShell is setup to not allow executing the profile script that you created in the last step. The easiest way to get around this is to set the execution policy to unrestricted. This comes with some security risks. If you do not want to take on these risks, search for running signed scripts and signing your profile script.

To set the execution policy to unrestricted, run Windows Terminal or PowerShell as Administrator and execute the following

```
Set-ExecutionPolicy Unrestricted
```

### Nerd Font
Nerd Fonts are extended fonts that provide additional glyphs. Many prompt themes in oh-my-posh use these fonts to display information. Themes that indicate that they are minimal typically do not require Nerd Fonts. 

At a minimum, I'd recommend installing [Cascadia Mono PL](https://github.com/microsoft/cascadia-code/releases). While this is not a Nerd Font, it does provide additional glyphs for most powerline style themes.

For fuller experience choose your favorite font from https://www.nerdfonts.com/font-downloads. The default Windows Terminal font is Cascadia Mono and the Nerd Font version is Caskaydia Cove (at the time of writing this, there was only the normal weight available and it didn't seem to have all glyphs yet). You might also find familiarity in FiraCode and Inconsolata. I recommend FiraCode.

To install a font, unzip the zip file, right click on the font file and click install. Install the versions that are marked Windows Compatible. Font file type doesn't matter much, but ttf font files are common for Windows.

To configure Windows Terminal to use the font installed, navigate within Windows Terminal to Settings > Profiles > Windows PowerShell > Appearance > Font face, select the font you just installed and click save. If the font you installed isn't shown, close and reopen Windows Terminal and try again.

### oh-my-posh prompt themes
Right now there is no theme set for oh-my-posh and PowerShell is showing a modified prompt from posh-git and looks something like 
`C:\Source\maui [main ≡]>`, which is already a big step up from the default `PS C:\Source\maui>`. However, oh-my-posh include a lot of prebuilt themes that can provide additional information and just make the prompt look fun.

To get and preview all oh-my-posh themes run `Get-PoshThemes`.

To set a theme go back to your PowerShell profile (or open it again with `code $profile`) and add the following line, replacing powerline with your theme of choice.

```
Set-PoshPrompt -Theme powerline
```

If some of the fonts look a little off, you might want to try a different Nerd Font. Some fonts simply look better than others.

Also, if you don't find any additional value out of the oh-my-posh customization, I'd recommend uninstalling oh-my-posh with `Remove-Module oh-my-posh` and removing `Import-Module oh-my-posh` from your profile.

### Windows Terminal color schemes
Windows Terminal ships with some pretty nice looking color schemes. To select one, navigate to Settings > Profiles > Windows PowerShell > Appearance > Color scheme.

### Adding the Git Bash shell
Windows Terminal is not a shell, but rather a host. Out of the box it includes PowerShell and Command Prompt. If you'd like to setup a different shell, it's pretty straight forward. One way is to navigate to Settings and click the plus icon. Here you can setup your shell. Assuming Git for Windows is installed, you can add the Git Bash shell with the following

>Name: Git Bash
>Command line: "%PROGRAMFILES%\git\bin\bash.exe"
>Icon: %PROGRAMFILES%\git\mingw64\share\git\git-for-windows.ico

Alternatively, you can click the settings icon after navigating to Settings. This will open a `settings.json` file. In the `profiles` `list` array add the following object

```json
{
  "commandline": "\"%PROGRAMFILES%\\git\\bin\\bash.exe\"",
  "icon": "%PROGRAMFILES%\\git\\mingw64\\share\\git\\git-for-windows.ico",
  "name": "Git Bash",
},
```

While it is out of the scope of this article, it is possible to customize the Bash shell with oh-my-posh in a very portable manner, check out https://ohmyposh.dev/ for more info.

### Quickly open .sln files
If you find yourself opening .sln files from within PowerShell often, this [Open-Solution PowerShell module](https://gist.github.com/refactorsaurusrex/2a0d8efd88ceb30eb69488c1b11c7682) is very handy. Once installed, the module will search through the current folder and open the .sln file just by exectuing `sln`. If there are multiple .sln files, it'll list them all. No more need for the finger gymnastics of typing `**\*.sln` then tabbing though the results

### Enjoy!
That's everything. Let me know if you found this helpful, if you ran into any issues or if you have any suggestions.

