## ssh
Basic ssh commands and tunnels.

#### Forward Auth
```bash
ssh -A foo.bar
```

## vim
Some tips and tricks in [vim8](https://github.com/vim/vim).

#### abolish
Search replace
```vim
:S/old/new/g
```

## utils
Collection of misc shell commands.

#### watch
Watch files in local dir every half second.
```bash
watch -d -n.5 ls -lat
```

## web

#### httpie
Great for debugging REST APIs.
```bash
apt-get install httpie
```

```bash
http --json "https://ubergarm.com/blog/archive/"
```
