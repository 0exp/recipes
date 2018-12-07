## Language Server Protocol for Ruby

- `gem install solargraph`
- `Package Control: Install Package` => `LSP`
- `Preferences => Package Settings => LSP => Settings`
- configure:

```javascript
{
  "clients": {
    "ruby (solorgraph)": {
      "command": ["/Users/DaiveR/.rbenv/shims/solargraph", "stdio"], // push your own path to solargraph
      "enabled": true,
      "env": {},
      "languageId": "ruby",
      "scopes": ["source.ruby"],
      "settings": {},
      "syntaxes": ["Packages/Ruby/Ruby.sublime-syntax"]
    }
  }
}
```
