---
title: "Debugging the installation process for Q# + Python on Linux"
author: glassnotes
date: 2020-06-02
categories:
  - blog
tags:
  - contributed-post
  - installation
  - python
  - documentation
---

I'm Olivia, I'm a quantum computing researcher at <a href="https://www.triumf.ca/" target="_blank">TRIUMF</a> in Vancouver, Canada. I work with nuclear and high-energy physicists on finding problems in their domains that might benefit from quantum computing. I do a lot of programming for my own research projects in quantum circuit synthesis/optimization and tomography. I also love writing, working with students, and teaching - especially teaching scientists from other fields about how quantum computing works.

I recently got started with the QDK to contribute to development of a new library for Q# about  <a href="https://github.com/qsharp-community/qram" target="_blank">quantum RAM</a>. However, I ran into a few issues during the installation process. Turns out they were easily fixable, but I had to slightly tweak the commands from the installation guide and the sequence of tweaks is not written in full anywhere. One of our goals for developing the qRAM library is to fully document the process, to help others develop libraries of their own. So here I've compiled everything together for the next person who might run into similar issues.

Thanks to Chris Granade and Sarah Kaiser for helping me debug!

------------------

My machine is running Ubuntu 19.10.  I have no prior experience with anything .NET, so I started installation from scratch. I followed these <a href="https://docs.microsoft.com/en-us/quantum/install-guide/pyinstall" target="_blank">installation instructions</a> for getting set up to develop with Q# in conjunction with Python and Jupyter notebooks.

###  Step 1 

The first step was pretty straightforward:

![Q# Python installation step 1](/assets/images/qsharp-linux-python-step1.png "Q# Python installation step 1")

I already had Python installed on my machine with <a href="https://www.anaconda.com/products/individual" target="_blank">Anaconda</a>. I was able to install the .NET Core SDK 3.1 following its <a href="https://docs.microsoft.com/en-ca/dotnet/core/install/linux-package-manager-ubuntu-1910" target="_blank">Linux installation instructions page</a>. At the time of writing, the instructions are to first add the product key:

```
$ wget https://packages.microsoft.com/config/ubuntu/19.10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
$ sudo dpkg -i packages-microsoft-prod.deb
```

Then, actually install the package by running:

```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https
$ sudo apt-get update
$ sudo apt-get install dotnet-sdk-3.1
```

### Step 2

Enter Python and Q#. 

![Q# Python installation step 2](/assets/images/qsharp-linux-python-step2.png "Q# Python installation step 2")

The first thing I did was make a new Anaconda environment. Then, I switched over to it before installing the qsharp package.

```
$ conda create --name qdk python=3.8
$ conda activate qdk
$ python -m pip install qsharp
```

Loosely, the <a href= "https://docs.python.org/3/using/cmdline.html#cmdoption-m" target="_blank">`-m` flag </a> is telling python to use the `pip` module from the path stored in Python's `sys.path`. This ensures that `qsharp` gets installed on the version of Python within our active environment rather than the "global" Python. You can see what this is set to by running `which python`, or checking within Python itself. If you're in the conda environment, you'll see that it points to the Python of your environment and not just something like`/usr/bin/python`.

![Checking your Python path](/assets/images/qsharp-linux-python-pythonpath.png "Checking your Python path")

### Step 3

This is where I started running into trouble...

![Q# Python installation step 3](/assets/images/qsharp-linux-python-step3.png "Q# Python installation step 3")

The first command ran with no issues, but my terminal yelled at me after the second: 

![Oh no!!](/assets/images/qsharp-linux-python-iqsharpinstall-error.png "Oh no!!")

We know that it can't be one of the first two reasons - the spelling is fine, and `dotnet-iqsharp` certainly exists having run the first command just fine. The issue then must be with the path. It turns out that when installing the .NET Core on Linux, it is a <a href="https://natemcmaster.com/blog/2018/05/12/dotnet-global-tools/#common-errors" target="_blank">common problem</a> that the tools don't get added automatically to your path. The way to solve this, as per the link above, is by adding them to the path explicitly:

```
$ echo "export PATH=\"\$PATH:\$HOME/.dotnet/tools\"" >> ~/.zshrc
```

I use `zsh` so I am sending the output to `~/.zshrc`. If you use a different shell such as `bash` you'll need to replace `~/.zshrc` with whatever the name of your configuration file is (probably `~/.bash_profile` or `~/.bashrc`).

Now that the `dotnet` command knows where to find our tools, we should be golden. However, this is what happened next...

![Trying again now that dotnet is properly](/assets/images/qsharp-linux-python-first-solution.png "Trying again now that dotnet is properly on our path.")

Even when I ran with`sudo`, I ended up with the same error as before! Let's go back to:

```
➜ dotnet iqsharp install
[Errno 13] Permission denied: '/usr/local/share/jupyter'
Perhaps you want to install with `sudo` or `--user`?
```

I noticed that the path here is wrong - it is trying to use the `jupyter` command from `/usr/local/share`, but it should be using the one from the `qdk` conda environment:

![Jupyter path](/assets/images/qsharp-linux-python-jupyter-path.png "Checking the path of jupyter to make sure we have the one for the environment we want.") 

The solution then is to use the `--user` flag:

![We did it!](/assets/images/qsharp-linux-python-final-solution.png "We did it!") 

With that, you should be able to run `import qsharp` in Python, and you're ready to go! 

