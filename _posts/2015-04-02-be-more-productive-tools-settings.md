---
title: Be More Productive
categories:
- DevEnv
- Shell
tags:
- iterm
- zsh
- editors
description: use tools to be more productive
date: 2015-04-02
classes: wide
---

I came to realize that I should've put more efforts to improve my workflow, to be more efficient and productive, and as for now, the best way to achieve this goal is to understand more about the tools I use in day-to-day development. Frankly speaking, I used to tell myself that I could customize my editor or terminal next time, but procrastination kicked in and you know how it ends.

# Oh-My-Zsh & iTerms 2

I remember myself tried zsh a year ago, but didn't stick with it mostly because it failed to sync with Apple Time Capsule, so I opted to bash as my local shell, and I've been happy with it. Since I am striving to be more productive, which means utilizing scripts for repetitive procedures and using more keyboard instead of mouse, I suppose a better shell is necessary in this case. That's why I decided to give [Oh-My-Zsh](http://ohmyz.sh/) another try.

## Oh-My-Zsh

### Steps

- Check out if `zsh` already installed by `zsh --version`(it is normally preinstalled on Mac OS);
- Go to System preference -> Users and Groups -> Choose your User profile;
- Open advanced settings (by right clicking on your user profile);
- Select `zsh` for Login Shell;
- Install `oh-my-zsh` by:

```raw
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh`

or

wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh
```

You can check the download by `cd ~/.oh-my-zsh`.

- After successful installation, you should have the default themed on-my-zsh installed. However, I personally would like to use the `agnoster` theme, and it requires a few extra steps to make it work, including:

- install this color theme in iTerm 2

      + download the source file, make sure it contains a file similar to `solarized.itermcolors`;
      + open iTerm color settings by `CMD+i`, click on the colors tab, select solarized file in `load presets`;
      + Don't forget to open the iTerm preference panel(by `CMD+,`), select the profile on the left, and make sure it is using the solarized color theme on the right, else the color theme won't be applied when you open iTerm the next time.

- install fonts on your laptop
  Install fonts is easy in latest Mac OS, open the font file, and click `install the font`.

- Modify the zsh file

    - Update the theme: `ZSH_THEME="agnoster"`
    - And in terms of this particular theme, you'd better add an user information:

```raw
# Default User
DEFAULT_USER="(whatever-your-system-name-is)"
```

Besides, if you used rvm for existed projects, such as jekyll, you need to add this line to the zshrc file as well:

```raw
# For RVM
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"
```


- Install Plugins

There are dozens of plugins bundled with oh-my-zsh, all you need to do is update in the plugin section:
`plugins=(rails git textmate ruby lighthouse)`


There is a detailed [tutorial](http://szarapka.com/zshohmyzsh/) here for further reference.


## iTerm2

In spite of using iTerm for a while, I was just scratching the very surface of this tool.

### Settings

After read the first section of the official document [here](https://iterm2.com/documentation-highlights.html), I just realized that for every type of product, be it an open source or a desktop app, it is always essential to *read the document*.

A few settings I'd like to use from now on including: 'Hotkey Window', 'Smart Cursor Color', 'Window Arrangements', 'Reuse previous session’s directory', 'Focus follow mouse', and the storage for 'Instant Replay'.

### Shortcut Keys

Those shortcut keys are awesome, which are well documented in the official document,  [here](http://jots.mypopescu.com/post/11351058130/iterm2-tips-tricks), [here](http://teohm.com/blog/2012/03/22/working-effectively-with-iterm2/), and also [here](http://adrianchen.co.uk/blog/2014/04/21/iterm-setup-tips/), some interesting ones are:

- `CMD+f` to find text, and use Tab and Shift+Tab to select more text to copy;
- `CMD+D`,`CMD+Shift+D` to split pane with current profile, whereas`CMD+Shift+Option+H`,`CMD+Shift+Option+V` to split pane with differnt profile;
- `CMD+;`, autocomplete;
- `CMD+Shift+H`, paste history;
- `CMD+Opt+B`, instant replay;
- `CMD+Enter`, full screen;
- `CMD+Shift+Enter`, the current panel will go to full screen;
- `CMD+Opt+E`, all your tabs will be shown at once;
- `ctrl+a`, beginning of line;
- `ctrl+e`, end of line;
- `ctrl+l`, clear screen;
- `CMD+Shift+]`, `CMD+Shift+[` for tab navigation;
- `CMD+/`, rainbow cursor;
- `CMD+Alt+i`, input for all panes;


Although I'd like to document a scenario mentioned in this [blog](http://chris-schmitz.com/develop-faster-with-iterm-profiles-and-window-arrangements/), which addresses how to start the dev environment with one click.

Basically you need to create new profile, select command instead of the default login shell, and fill it with `/bin/zsh`, and write the command you'd like to run initially in the 'send text at start' section.

Now back to the iTerm interface, split panes with different profiles (the shortcut key for this is different from split panes with current profile).

After you have all the tab and pane opened, save this window arrangement by `Command + Shift + S`, and you can use this set in 'window -> restore window arrangement' or `Command + Shift + R` later on.

Last but not least, you can add customized alias settings such as `alias -g pyhost="python -m SimpleHTTPServer"` in the zshrc file.


# WebStorm, Sublime Text, Atom

Those editor/IDE are great tools, and there are also Brackets, Textmate available. I use different apps for different scenarios, albeit I'd like to spend more time to be familiar with WebStorm because the features are fascinating.

# Other Tools

Beyond the terminal and editors, there are a bunch of apps on Mac OS help you be more productive(as a developer). I've been adopting apps such as the api documentation app - [Dash](https://itunes.apple.com/us/app/dash-docs-snippets/id458034879?mt=12), it is a freemium app for all kinds of api documentations, this [CheatSheet](http://www.cheatsheetapp.com/CheatSheet/) app allows you to look up shortcuts lists quickly, and this free window control app [Spectable](http://spectacleapp.com/) allows you control windows with hotkeys, there is another app called Moom does the same thing you can try that out as well.

Last but not least, there is a great app - [Alfred](http://www.alfredapp.com/) to improve your workflow, I used that before Mac OS Yosemite, but then opted to the awesome [Flashlight](https://github.com/nate-parrott/Flashlight), which is well supported for MAC OS 10.10+, and it's an open sourced project (with python), thus there are tons of plugins at your disposal, personally I'd say this is a compelling alternative to Alfred.


Lastly, I created a [repo](https://github.com/fengshuo/my-cheat-sheets) for cheatsheets.
