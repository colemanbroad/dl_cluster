---
title: "Deep Learning on the Cluster"
author: [Coleman Broaddus]
date: "2018-11-23"
fontsize: 10pt
links-as-notes: true
---

<!-- build command: -->
<!-- pandoc dl_cluster.md --template eisvogel -o example.pdf -->

# Get a project space on `lustre/` filesystem

Step 1: Ask Oscar or Juraj (pronounced "You-Rei") for a project space.
It will be something like `/lustre/projects/project-<yourname>/`.
You can access it from your local machine with Finder via `Cmd-K` and connecting to `smb://mack.mpi-cbg.de/projects/project-<yourname>/`
Or by logging on to falcon and `cd`-ing there.

# Scientific Computing Wiki

Go here: https://wiki.mpi-cbg.de/scicomp/HPC
Read the section "Getting Started with HPC"

The only important commands are:

- `queue_summary`
- `squeue -u <yourname>`
- `scancel <jobid>`
- and `srun <lots of args>`

# The `srun` command

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

# Submitting multiple jobs

Just call the `srun` command multiple times.
The submission system will figure out how to allocate the jobs onto the available nodes.
Write a for loop in python that will print the command with the various args.
Copy and paste the commands into your terminal.

# Accessing your data on the fileserver

If you're on falcon you can just do `/fileserver/myersspimdata/`.
But that's not gonna work if you're running a job on one of the GPU nodes!
Then you need to use these secret paths...

```python
myersspimdata = Path("/net/fileserver-nfs/stornext/snfs2/projects/myersspimdata/")
greencarpet   = Path("/net/fileserver-nfs/stornext/snfs4/projects/green-carpet/")
csbdeep       = Path("/lustre/projects/CSBDeep/ISBI/")
```

just put these names in your project's `utils.py` or something...

# Installing python stuff

First git clone the repository you want to install.
Then use `pip3 install --user -e <dir>` to install the repo in editable mode.
I don't know how to use python2. Don't use python2.

# Setting up your `.bashrc`

You want to add the following lines:

```
module load gcc/6.2.0
module load cuda/9.0.176
```

This version of cuda currently ^[Fri Nov 23 14:21:11 2018] works with the current/default Tensorflow`1.9.0` on the cluster.

# Use sublime (the best editor)

Go here and follow instructions for using `rsub` https://stackoverflow.com/questions/37458814/how-to-open-remote-files-in-sublime-text-3
Then you can simply login to falcon like `ssh falcon` (assuming you're at work, not at home, and using ethernet!)

# Steal Uwe's `~/.ssh/config`

It will let you login to the cluster from home with the simple command `ssh efal` and it will automatically connect `rsub` to sublime with proper port forwarding!