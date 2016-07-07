# Remote Jupyter Notebook Kernels for R, Python, and Julia

This works on a mac, and should probably work on a linux machine too, but I didn't test it on one. 

## Ingredients
- password-less ssh access to whovian
- anaconda installed on whovian and local machine (with R-packages on whovian)
- remote_ikernel on local machine


## Password-less ssh access to whovian

We need to set up password-less ssh from your local machine to whovian.
If you can already enter `ssh whovian` and connect without being prompted for a password, you can skip this section.

In the `~/.ssh` directory on your local machine, find `id_rsa.pub` and paste its contents into your `~/.ssh/authorized_keys` file on whovian.

If `.ssh` doesn’t exist on your local machine or on whovian, create it with `mkdir ~/.ssh`.
If `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` don’t exist on your local machine, create a key pair with `ssh-keygen -t rsa -b 4096`, and just hit enter when asked for a password.

Make sure that the permissions of the `.ssh` dir on both whovian and the local machine are set correctly.
You probably want the `.ssh` directory permissions to be `700`,
the public key `id_rsa.pub` to be `644`, and your private key `id_rsa` to be `600`.
You can set all of these with `chmod`, e.g. `chmod id_rsa.pub 644`.

Lastly, on the local machine append to or create `~/.ssh/config` so that it contains

```
Host whovian
    HostName whovian.systemsbiology.net
    User <your_username>
```

In a new terminal, you should now be able to enter `ssh whovian` and connect without being prompted for a password.

## Install anaconda

We need to install [anaconda](https://www.continuum.io/downloads) python on both the local machine and your home directory on whovian, for slgightly differnet readsons.
On the local machine, we need it for the Jupyter Notebook infrastructure, and on whovian, we need it for the IRkernel package.
It shouldn't matter if you use the Python 2.7 or Python 3.5 suite, though I've only tested with 2.7.

### On your local machine

From [this link](https://www.continuum.io/downloads#_macosx), grab the command line installer and run it with

```
bash Anaconda_someversion_OSX-x86_64.sh
```
for whatever the downloaded script is named.

Let anaconda add prepend itself to you path, so the that the
anaconda versions of Python or R are the default ones.

Open a new terminal, and

```
which python
```
should be something like `/Users/<your_username>/anaconda/bin/python`.

#### Test notebook

On your local machine, entering

```
jupyter-notebook
```

in ther terminal should open a web page where you can start new notebook.
In the upper right hand corner should be a drop-down menu that says "New", and the only option for a notebook should be Python.

### On whovian

On the local machine, you just need anaconda for the `jupyter-noteboook` infrastructure, but on whovian, you need to also install R and related packages via anaconda.

From [this link](https://www.continuum.io/downloads#_unix) get the command line installer for linux and run it with

```
bash Anaconda_someversion_Linux-x86_64.sh
```
for whatever the downloaded script is named.

Let anaconda add prepend itself to you path, so the that the
anaconda versions of Python or R are the default ones.

We also need to isntall R using anaconda's package manager:

```
conda install -c r r-essentials
```

In a new terminal, connect to whovian and

```
which R
```
should be something like `~/anaconda/bin/R`.

Note that this is a different R than RStudio or the system R, so you’ll have to install all the packages you like to use in this R that you'll be connecting to on whovian

## Configure the remote kernel

This only needs to happen on your local machine.
Install the remote kernel manager with:

```
pip install remote_ikernel
```

Entering `remote_ikernel` in the terminal should display a help message.

### Python

To add a remote Python kernel, in the terminal on your local machine, enter:

```
remote_ikernel manage --add \
    --name="Python" \
    --kernel_cmd="ipython kernel -f {connection_file}" \
    --interface=ssh \
    --host=whovian
```

If you open a new `jupyter-notebook` session, you should now see "SSH whovian Python" in the drop-down menu for types of notebooks.

### R

To add a remote R kernel, in the terminal on your local machine, enter:

```
remote_ikernel manage --add \
    --name="R" \
    --kernel_cmd="R --slave -e IRkernel::main\(\) --args {connection_file}" \
    --verbose \
    --interface=ssh \
    --host=whovian
```

If you open a new `jupyter-notebook` session, you should now see "SSH whovian R" in the drop-down menu for types of notebooks.

## Julia

This takes a bit more effort, since I don't know how to install a Julia binary on whovian, so we need to compile from source.

### On whovian

First we need to install Julia on whovian.  Unfortunately the Julia installer depends on cmake, which isn't installed on whovian.

#### Install cmake

The easiest way to install CMake is from source.
Head over to the [CMake downloads page](http://www.cmake.org/download/) and get the latest “Unix/Linux Source” *.tar.gz file.
You can download it form the terminal on whovian with

```
curl -O "https://cmake.org/files/v3.6/cmake-3.6.0.tar.gz"
```
where you might have to change the verison numbers.

Now compile and install with the following commands.
Make sure to set the `--prefix` flag correctly, otherwise you won’t have permissions to install files at the default location.

```
tar -xf cmake*.tar.gz

cd cmake*

./configure --prefix=$HOME

make

make install
```

You should now have the most up-to-date installation of cmake in `~/bin`.
Check the version by typing:

```
cmake --version
```

If this fails because `~/bin` isn't in your path, you can add it by appending `export PATH="$PATH:${HOME}/bin"` to your `~/.bashrc` file and logging out and logging back in.

#### Install Julia

From you home directory on whovian:

```
git clone https://github.com/JuliaLang/julia.git
```
then `cd julia`, switch to the stable branch with `git checkout release-0.4`, and enter `make`.
It will take a while to compile everything from source.

To add the julia binary to your path, add a symbolic link to the julia binary to your `~/bin` directory with 

```
ln -s ~/julia/usr/bin/julia ~/bin/julia
```

you should now be able to start up the julia REPL by entering `julia` in the terminal.

##### Add IJulia kernel

Start julia, and enter

```
Pkg.add("IJulia")
```

It might take a little while to downlaod the package.

### On your local machine

To add a remote Julia kernel, in the terminal on your local machine, enter:


```
remote_ikernel manage --add \
   --name="Julia" \
   --kernel_cmd="/users/rdonovan/bin/julia -i /users/rdonovan/.julia/v0.4/IJulia/src/kernel.jl {connection_file}" \
   --interface=ssh \
   --host=whovian \
   --language=julia
```
   
If you open a new `jupyter-notebook` session, you should now see "SSH whovian Julia" in the drop-down menu for types of notebooks.
