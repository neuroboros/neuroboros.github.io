# Setup environment for Colab

Certain packages need to be downloaded and setup before using them.
Copy this block to the beginning of the Notebook and run it before running other code.
You might need to restart the runtime to make it work (in the menu: Runtime - Restart runtime).

```bash
%%bash
# https://handbook.datalad.org/en/latest/intro/installation.html
pip install datalad-installer
apt install -y netbase
datalad-installer --sudo ok git-annex -m datalad/git-annex:release
git config --global filter.annex.process "git-annex filter-process"
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
pip install datalad

pip install -U neuroboros
pip install -U hyperalignment
```