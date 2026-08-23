# VIP-Team-AIDA3.github.io

Book-style course site built with Jupyter Book and deployed with GitHub Pages.

## Local preview

```bash
git switch dev
source .venv/bin/activate

jupyter-book clean . --html
jupyter-book build .

python -m http.server 8000 --bind 127.0.0.1 --directory _build/html
```