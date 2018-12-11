---
# title: "Deep Learning on the Cluster"
# author: [Coleman Broaddus]
# date: "2018-11-23"
# links-as-notes: true
# stylesheet: gh-md.css
---

<head> <link rel="stylesheet" type="text/css" href="gh-md.css"> </head>

# Deep Learning on the Cluster

## Get a project space on `lustre/` filesystem

Step 1: Ask Oscar or Juraj (pronounced "You-Rei") for a project space.
It will be something like `/lustre/projects/project-<yourname>/`.
You can access it from your local machine with Finder via `Cmd-K` and connecting to `smb://mack.mpi-cbg.de/projects/project-<yourname>/`.
Or just by logging on to falcon and `cd`-ing there.

## Scientific Computing Wiki

Go [here](https://wiki.mpi-cbg.de/scicomp/HPC). Read the section "Getting Started with HPC".

The only important commands are:

- `queue_summary`
- `squeue -u <yourname>`
- `scancel <jobid>`
- and `srun <lots of args>`

## The `srun` command

Here are some examples...

A batch job (non-interactive) runs the command `time python3 fly_predict.py`:

`srun -J predict1 -n 1 -c 1 --mem=128000 -p gpu --gres=gpu:1 --time=12:00:00 -e std.err -o std.out time python3 fly_predict.py &`

NOTE: The ampersand (`&`) prevents you from getting trapped inside the srun command!

If you want to open an ipython session on a GPU node then you need to run and interactive job.
This is done with the same command as above, but adding the `--pty` flag and running `bash` as your command ala...

`srun --pty -n 1 -c 1 --mem 128000 -p gpu --gres gpu:1 --time 12:00:00 bash`

Go to the scientific computing wiki to figure out what all the command line args do...

- `-n` number of nodes
- `-c` number of CPU cores per node
- `-p` which partition (GPU)
- `--gres gpu:1` Just because you've requested the GPU partition doesn't mean you get a GPU! Add this to actually request a "gpu resource"
- `--time ` maximum job time (after this time your job is killed)
- `--pty` makes your job interactive

## Submitting multiple jobs

Just call the `srun` command multiple times.
The submission system will figure out how to allocate the jobs onto the available nodes.
Write a for loop in python that will print the command with the various args.
Copy and paste the commands into your terminal.

## Accessing your data on the fileserver

If you're on falcon you can just do `/fileserver/myersspimdata/`.
But that's not gonna work if you're running a job on one of the GPU nodes!
Then you need to use these secret paths...

```python
myersspimdata = Path("/net/fileserver-nfs/stornext/snfs2/projects/myersspimdata/")
greencarpet   = Path("/net/fileserver-nfs/stornext/snfs4/projects/green-carpet/")
csbdeep       = Path("/lustre/projects/CSBDeep/ISBI/")
```

just put these names in your project's `utils.py` or something...

## Installing python stuff

First git clone the repository you want to install.
Then use `pip3 install --user -e <dir>` to install the repo in editable mode.
I don't know how to use python2. Don't use python2.

## Setting up your `.bashrc`

You want to add the following lines:

```bash
module load gcc/6.2.0
module load cuda/9.0.176
```

This version of cuda currently ^[Fri Nov 23 14:21:11 2018] works with the current/default Tensorflow1.9.0 on the cluster.

And while you're at it you may as well add these lines too:

```bash
alias ipython='ipython3 --matplotlib=Agg'
alias rsub='rsub --port 52699'
```

This way you can use matplotlib from the cluster! (Does this work with -X forwarding? I don't know.)
Why do people use matplotlib anyways when working with images? I don't know.
Just move the data to your local machine and do plotting there...

## Organizing python scipts 

Datagen | Train | Predict  

There is only one objective way of optimally separating these concerns, and that is to make three separate scripts for these three separate tasks.
Each script should be runnable by itself with no args from the command line, so that in 6 months when you forget what the hell it is supposed to do you can at least figure out if it's broken or not.
This also makes it really easy to run in batch mode via `srun`.

Naming examples:

`fly_datagen.py`, `fly_train.py`, and `fly_predict.py`.

Should be both importable with a main method and callable from the command line.

## Moving data to your local machine

You can access your project directory in finder via smb (described above), but if you want to work from home this is unbearably slow, so use `rsync` instead!
If you're using Uwe's `~/.ssh/config` then all this requires is a simple call like `rsuync`

## Use sublime (the best editor)

Go [here](https://stackoverflow.com/questions/37458814/how-to-open-remote-files-in-sublime-text-3) and follow instructions for using `rsub`.
Then you can simply login to falcon like `ssh falcon` (assuming you're at work, not at home, and using ethernet!)

## Steal Uwe's `~/.ssh/config`

Steal it. It's [here](config). But change the names, duh.

It will let you login to the cluster from home with the simple command `ssh efal` and it will automatically connect `rsub` to sublime with proper port forwarding!