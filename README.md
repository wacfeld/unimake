# unimake

universal make and live preview based on file extension

- we want to be able to write music in [abc notation](https://abcnotation.com/) while seeing a live preview of the sheet music.
we also want to be able to listen to the music on command.
- we want to be able to write markdown while looking at a live preview of it.
- we want to create diagrams in [dot](https://graphviz.org/docs/layouts/dot/) and see a live preview of it.


there exist tools which can do some of these things, but not others.
moreover, some of these tools are websites, or applications, meaning that we can not then use them with the text editor of our choice.

using makefiles, we can generalize this process.
however, having to separate every diagram/markdown file into its own directory with a makefile, and then having to separately maintain each makefile, is a pain.

unimake is a tool which eliminates the need for directories and specialized makefiles, using "universal" makefiles based on file extension (.abc, .md, .gv, etc.)

# examples

for example, say we want to view/listen music using abc notation.
then we create the following file named `abc`, and put it in some directory `/path/to/somewhere`, where all of our universal makefiles go.

```
all:
	make cont
	make once
cont:
	abcm2ps __SRCFILE__ -O out.ps
	ps2pdf out.ps out.pdf
viewcont:
	zathura out.pdf
once:
	abc2midi __SRCFILE__ -o out.mid
	timidity out.mid -Ow -o - | ffmpeg -i - -acodec libmp3lame -ab 64k out.mp3
viewonce:
	xterm -e "exec mpv out.mp3"
```

(notice how `__SRCFILE__` is used in place of an actual file name)

then we can run something like ```$ unimake file.abc /path/to/somewhere /tmp cont```, where `file.abc` contains our abc notation.
this will then create a temporary directary inside `/tmp`, where it will copy our makefile for this file type (`abc`) and `file.abc`, and continuously call `make cont` along with `zathura` (or the pdf viewer of your choice) to view your sheet music as you edit it.

similarly, `$ unimake file.abc /path/to/somewhere /tmp once` will do the same, but instead call `make once` and then use `mpv` to listen to your music as an mp3.

the two modes "once" and "cont" are for one-time and continuous preview, which can have different applications, for example sound and visuals.

if you use vim, you can then add mappings to do this on command, for example:
```vim
nnoremap <leader>o <cmd>silent! exec 'Dispatch! ' . '/path/to/unimake' . ' ' . expand('%:p') . ' ' . '/unimake/dir' . ' ' . '/tmp/dir' . ' once'<cr>
nnoremap <leader>c <cmd>silent! exec 'Dispatch! ' . '/path/to/unimake' . ' ' . expand('%:p') . ' ' . '/unimake/dir' . ' ' . '/tmp/dir' . ' cont'<cr>
```

then `<leader>o` will call unimake with mode once on the current buffer, etc.

this example uses the [vim-dispatch](https://github.com/tpope/vim-dispatch) plugin, but you can use anything you want to asynchronously call the commands.


a similar thing can be done for markdown, dot, tex, or anything else.
the only thing you need to do is create the corresponding makefile and put it in the designated folder.

# requirements
- python3

# installation
simply download `unimake` and then `$ chmod +x unimake`. put it in your `$PATH` if you want.

# work in progress

some things i am planning to add/change in the future:
- [ ] possibly use symlinks to avoid having to copy files constantly
- [ ] check the source file for changes instead of constantly updating
- [ ] option to set update interval (currently 1 second)
- [ ] (vim-specific) extract the source code from the vim buffer instead of the file, so that saving is not required, the way some related plugins work
- [ ] custom modes
