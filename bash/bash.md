### Bash functions

## Bash Cheatsheet

#### Creating functions

Generate folder and folder and files with the article name

```bash
article() {
  mkdir -p "./$1"
  touch "./$1/$1.html"
  touch "./$1/$1.md"
 }
```

```
article bash
```

---

#### Alias

Creating a shortcut to a foldes

```bash
 alias project="cd/Users/lucasvilela/CODE"
```
