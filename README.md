# Remote Jupyter Notebook Kernels for Python and R

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
