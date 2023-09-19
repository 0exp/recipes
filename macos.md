### Install from damaged installer

```shell
sudo xattr -r -d com.apple.quarantine /path/to/your/damaged/isntaller

# then - execute your file ;)
```

### Show hidden files in Finder

```shell
command + shift + .
```

### brew: switch package version

```shell
# for example: ("readline") from any to installed 7.0.5

brew switch readline 7.0.5
```

### Kill Finder

1. `defaults write com.apple.finder QuitMenuItem -bool false`
2. `killall Finder`

New features:
  - Finder => Main Menu => Quit Finder
  - Command-Q works as expected :)
