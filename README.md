# Remote Notebook Kernels for Python and R

This works on a mac, and should probably work on a linux machine too, but I didn't test it on one.
It should also work on servers other than whovian with trivial changes.

## Ingredients

- password-less ssh access to whovian
- anaconda installed on whovian and local machine (with R-packages on whovian)
- remote_ikernel on local machine


## Password-less ssh access to whovian

We need to set up password-less ssh from your local machine to whovian.
If you can already enter `ssh whovian` and connect without being prompted for a password, you can skip this section.


Basiaclly what we need to do is pste the contents of `~/.ssh/id_rsa.pub` from your local machine into the  `~/.ssh/authorized_keys` file on whovian.
If you don't have those files, here's how to generate them.

### On your local machine

If `.ssh` doesn’t exist on your local machine or on whovian, create it with `mkdir ~/.ssh`.
If `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` don’t exist on your local machine, create a key pair with `ssh-keygen -t rsa -b 4096`, and just hit enter when asked for a password.

Make sure that the permissions of the `.ssh` directory are set correctly, or ssh will complain and not function properly.
You probably want the `.ssh` directory permissions to be `700`,
the public key `id_rsa.pub` to be `644`, and your private key `id_rsa` to be `600`.
You can set all of these with `chmod`, e.g. `chmod 644 id_rsa.pub`.

Lastly, append to or create the file `~/.ssh/config` so that it contains

```
Host whovian
    HostName whovian.systemsbiology.net
    User <your_whovian_username>
```

where you should change `<your_whovian_username>` to your actual username on whovian.


### On whovian

If the `~/.ssh/authorized_keys` file doesn't exist, create it.
Set its permissions to `600` with

```
chmod 600 ~/.ssh/authorized_keys
```

Now paste the contents of `~/.ssh/id_rsa.pub` on your local macine into `~/.ssh/authorized_keys` on whovian.


### Test it out

In a new terminal on your local machine, you should now be able to enter `ssh whovian` and connect without being prompted for a password.


## Install Anaconda

We need to install [Anaconda](https://www.continuum.io/downloads) Python on both the local machine and your home directory on whovian, for slightly different reasons.
On the local machine, we need it for the Jupyter Notebook infrastructure, and on whovian, we need it for the IRkernel package.
It shouldn't matter if you use the Python 2.7 or Python 3.5 versions of Anaconda, though I've only tested with 2.7.

### On your local machine

From [this link](https://www.continuum.io/downloads#_macosx), grab the command line installer and run it with

```
bash Anaconda2-4.1.0-MacOSX-x86_64.sh
```
where you might need to change the verison numbers if they've been updated.

Let Anaconda add prepend itself to you path, so the that the Anaconda versions of Python is the default one.  You can also manually do this by adding

```
export PATH="${HOME}/anaconda2/bin:$PATH"
```

to your `~/.bashrc` file.
You might have to change `anaconda2` to `anaconda3` or just `anaconda` depending on how things got installed.

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

in the terminal should open a web page where you can start new notebook.
In the upper right hand corner should be a drop-down menu that says "New", and the only option for a notebook should be Python.


### On whovian

On the local machine, you just need anaconda for the `jupyter-noteboook` infrastructure, but on whovian, you need to also install R via Anaconda.

From [this link](https://www.continuum.io/downloads#_unix) get the command line installer for linux and run it with

```
bash Anaconda2-4.1.0-Linux-x86_64.sh
```
where you might need to change the verison numbers if they've been updated.

Let Anaconda add prepend itself to your path in `~/.bashrc` (_not_ `~/.bash_profile`, or the remote kernel won't work), so the that the Anaconda versions of Python or R are the default ones.  You can also manually do as in the previous section.


#### Install R

We also need to install R using Anaconda's package manager:

```
conda install -c r r-essentials
```


#### Test it out

In a new terminal, connect to whovian and

```
which R
```

should be something like `~/anaconda/bin/R`.

Note that this is a different R than RStudio or the system R, so you’ll probably have to install all the packages you like to use in this R.


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

## Test out remote notebooks

From your local machine you should be able to open a new remote jupyter notebook of whatever type you like form the drop-down menu.

### Python

In a remote Python notebook, entering

```
ls
```

should produce a list of files in your home directory on whovian.
Entering

```python
%matplotlib inline

import numpy as np
import matplotlib.pyplot as plt

mu, sigma = 0, 0.1
s = np.random.normal(mu, sigma, 1000)

count, bins, ignored = plt.hist(s, 30, normed=True)
plt.plot(bins, 1/(sigma * np.sqrt(2 * np.pi)) * np.exp( - (bins - mu)**2 / (2 * sigma**2) ), linewidth=2, color='r')
plt.show()
```

should produce a plot of a random sample from a normal distribution.


### R

In a remote R notebook, entering 

```r
list.files()
```

should produce a list of files in your home directory on whovian.
Entering

```r
library(ggplot2)
p <- ggplot(data = iris, aes(x = Sepal.Length, y = Petal.Length))
p + geom_point(aes(colour = Species))
```

should produce an inline plot of the iris data.
