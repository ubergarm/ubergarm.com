st - simple terminal
===

## Overview

I want pretty colors and fonts in [neovim](https://neovim.io/)

[suckless simple terminal](http://st.suckless.org/) `st` provides true color support out of the box and with [argbbg](http://st.suckless.org/patches/argbbg) patch to support transparent backgrounds through [compton](https://github.com/chjj/compton) and [scrollback](http://st.suckless.org/patches/scrollback) it makes a fine replacement for rxvt-unicode (while there is a patch, it seems 256 colors out of the box may remain standard).

## Clone and Patch

Clone

    mkdir suckless && cd suckless
    git clone http://git.suckless.org/st
    cd st

Apply argbbg patch

    git checkout -b argbbg
    curl http://st.suckless.org/patches/st-git-20151129-argbbg.diff | git apply
    git commit -am 'Applies argbbg patches'
    git checkout master
    git rebase argbbg master

Apply scrollback patches

    git checkout -b scrollback
    curl http://st.suckless.org/patches/st-git-20151122-scrollback.diff | git apply
    curl http://st.suckless.org/patches/st-git-20151106-scrollback-mouse.diff | git apply
    git commit -am 'Applies scrollback patches'
    git checkout master
    git rebase scrollback master

## Optional Change Default Font

Plenty of lovely patched fonts available at [powerline](https://github.com/powerline/fonts).

    fc-list # to see what fonts you have installed
    git checkout -b myfont

Edit `config.def.h` and change the font line to something like:

    static char font[] = "DejaVu Sans Mono for Powerline:pixelsize=14:antialias=true:autohint=true";

Bring it together now

    git commit -am 'Changes font to DejaVu Sans Mono for Powerline'
    git checkout master
    git rebase myfont master

## Build

    make
    sudo make install

## Dynamic Font Change

You can skip the above font change and simply invoke like this.

    st -f "DejaVu Sans Mono for Powerline:pixelsize=16:antialias=true:autohint=true" &

## Color Test

    awk 'BEGIN{
    s="/\\/\\/\\/\\/\\"; s=s s s s s s s s;
    for (colnum = 0; colnum<77; colnum++) {
        r = 255-(colnum*255/76);
        g = (colnum*510/76);
        b = (colnum*255/76);
        if (g>255) g = 510-g;
        printf "\033[48;2;%d;%d;%dm", r,g,b;
        printf "\033[38;2;%d;%d;%dm", 255-r,255-g,255-b;
        printf "%s\033[0m", substr(s,colnum+1,1);
        }
    printf "\n";
    }'

## Results

![xmonad-simple-terminal-neovim](/content/images/2015/12/st.jpg)

## TODO

You'll need to find a patched version of tmux to support true color in that handy terminal multiplexor.

## References

[XVilka gist](https://gist.github.com/XVilka/8346728)
