# Write mkdocs guide

all syntax guide for write mkdown file

---

## start project

```bash
mkdocs new my-project
cd my-project
```
### Project layout
![The initial MkDocs layout](imgs/initial-layout.png)

## link file
home page is [index file](./index.md). 
## image
![Screenshot](imgs/zcu102BootMode.png)

## Tables

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell


| Left Align | Center Align | Right Align |
| :--------- | :----------: | ----------: |
| Text       | Text         | Text        |
| Left       | Middle       | Right       |

## PDF file
[Open docker_cheatsheet.pdf](./Tool/chatsheet/docker_cheatsheet.pdf){target="_blank"}
<!-- [📄 Open docker_cheatsheet.pdf](./Tool/chatsheet/docker_cheatsheet.pdf){target="_blank"} -->

## Fenced code blocks

```
Fenced code blocks are like Standard
Markdown’s regular code blocks, except that
they’re not indented and instead rely on
start and end fence lines to delimit the
code block.
```

```python
def fn():
    pass
```