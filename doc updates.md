Simplest steps for doc editing,

1) If no local copy already

```
git clone https://github.com/darrylcauldwell/piCluster.git
```

2) Open folder in VS code
3) Open terminal in VS code and cd to mkdocs sub-folder and run

```
mkdocs serve
```

4) Browser to  http://127.0.0.1:8000/
5) Update files in mkdocs/docs browser realtime render
6) When finished in terminal in VS code create static site files and direct output from default to the place GitHub pages is configured to publish from.

```
mkdocs build -d ../docs
```