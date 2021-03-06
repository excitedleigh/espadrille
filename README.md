# Espadrille

Write short, simple scripts using the power of the entire Python ecosystem. Currently supports macOS.

Shell scripts are a pain to write, unless you've spent years mastering their arcane syntax and the myriad workarounds for the shell's everything-is-a-string philosophy. Writing a Python script and chucking it in your `$PATH` seems like a much more appealing option, but sometimes you want more than just Python's standard library. Installing the libraries you want in your global Python seems icky, but doing it properly and creating a virtualenv and packaging metadata files doesn't seem worth the effort.

Espadrille lets you write a single-file script that depends on external libraries, and takes care of installing them and keeping them isolated from the rest of your system for you.

Espadrille is named after a type of shoe, which is also [an unconventional packaging method for Pythons](https://www.abc.net.au/news/2019-02-25/snake-hitchhikes-in-suitcase-from-queensland-to-scotland/10846236).

⚠️ Espadrille is a very rough-around-the-edges proof of concept at the moment. It may eat your homework.

## Installation

To install a pre-built binary (currently macOS Mojave only):

```
pip install espadrille
```

To install from source, assuming you have a Rust toolchain installed:

```
cargo install espadrille
```

In the future, pip will be able to install binaries for more platforms, and possibly build from source as well.

## Usage

Write a Python script with a shebang line of `#!/usr/bin/env espadrille <dependencies> --`, like this:

```python
#!/usr/bin/env espadrille requests --

import requests

requests.get('https://example.com')
```

Then, `chmod +x` that script, and run it.

## FAQs

- **Which Python interpreter/version will my script run under?** Whichever one the `python3` command invokes. If this ever changes, you'll get whichever one the `python3` command invoked the first time Espadrille saw the particular combination of dependencies you're using (i.e. at the time it created the virtualenv). Future versions of Espadrille will be a bit smarter about this, and may even let you specify the version of Python to use.
- **Which versions of my dependencies will I get?** The latest version as of the first time Espadrille saw the particular combination of dependencies you're using (i.e. at the time it created the virtualenv). If you need a specific version, you can specify a version number in your shebang (e.g. `requests>=2` or `requests==2.19.1`, or any other format Pip accepts). Future versions of Espadrille will be a bit smarter about this, by upgrading things periodically.
- **How do I wrap my dependency list around multiple lines?** Don't. When you get to the point where you need to, write a setup.py file, or use flit or poetry, because you now have a proper project rather than a short, simple script.
- **Why did Espadrille reinstall all of my dependencies, even though I only changed one?** Espadrille works out which virtualenv to use by hashing the list of dependencies. When that list changes, you get a whole new virtualenv. Fortunately, because Pip prefers wheels and keeps local caches of things it installs, this is usually pretty fast unless you're depending on a large C dependency that doesn't have a wheel for your platform.
- **Why is my disk filling up with virtualenvs?** See above. Future versions of Espadrille will be a bit smarter about cleaning up virtualenvs that haven't been used in a while. For now, just delete Espadrille's cache folder; it'll recreate all the virtualenvs it's actually using the next time it needs them.
- **Wait, this is a Python tool, why is it written in Rust?** Because Espadrille uses virtualenvs, if Espadrille itself was written in Python, you'd have to wait for the Python interpreter to start up twice. In reality, this probably isn't a huge deal in most cases, so this leads us to the real reason: it seemed like a fun idea.
- **How good would it be if Espadrille just figured out which packages to install from `import` statements?** Heaps good. Unfortunately, in Python, the name of the _distribution_ (the thing you install from the PyPI through Pip) doesn't have to match the name of the _package_ (the thing that you import). This means that two different distributions on PyPI could expose the same package, whether by accident or malice. Since Espadrille's whole job is essentially "download code from the Internet and run it", it's important to make sure you're getting the code you expect.
- **Isn't downloading code from the Internet and running it super insecure?** If you were going to download each individual package from PyPI and manually audits its contents before installing it, then Espadrille isn't for you. But let's face it, you weren't going to do that. You were going to just blindly `pip install` it anyway, which is exactly what Espadrille does. That said, there are certainly things that Espadrille could do to keep you safe, and future versions might do some of them.
- **Why doesn't this work on Linux?** On Linux, shebangs can only have one argument, and the shebang for a Espadrille script is usually going to be `/usr/bin/env espadrille foo bar --`, which means `env` is going to try to find a binary whose filename is `espadrille foo bar --`. Future versions of Espadrille will read dependencies from a different part of the file, to prevent this.