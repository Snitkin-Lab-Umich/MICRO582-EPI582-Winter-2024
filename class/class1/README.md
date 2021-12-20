Class 1 – Setting up Great Lakes account and Introduction to UNIX
=================================================================

Goal
-----

- In this module, we will log into Great Lakes account and get access to the course working directory.  
- Get acquanted with the basics of UNIX file systems
- Learn how to navigate the directory structure using command line interface.
- Run basic UNIX commands to explore the course working directory.

Sign up for Great Lakes account
-------------------------------
**Log in to great lakes**

Log into your great lakes account using the ssh command below. SSH stands for secure shell and can be used to connect or transfer data to/from remote hosts such as Great Lakes.

```
ssh username@greatlakes.arc-ts.umich.edu
```

Replace username with your umich uniqname and run the above command to log in to Great Lakes.



Intro to Unix
-------------
 
 UNIX systems allows users to perform complex and powerful tasks, often with just a few keystrokes or lines of code. It helps users automate repetitive tasks and easily combine smaller tasks into larger, more powerful workflows. Use of the shell is fundamental to a wide range of advanced computing tasks, including high-performance computing.

Once you log in, By default Great lakes will take you to your home directory which should be something like - `/home/uniqname` where uniqname will be your username that we used for logging in. You can check your present location in UNIX file system with `pwd` command. 

`pwd` stands for present working directory and is a very handy command to quickly find out where you are in the file system. Lets run this command and find where we actually are.

```
pwd
```

My present working directory is `/home/esnitkin`

The course working direcory will be `/scratch/epid582w22_class_root/epid582w22_class/`. So lets use UNIX's `cd` command to go to the course directory. `cd` stands for change directory and it lets you move around the file system using the absolute/relative paths.

```
cd /scratch/epid582w22_class_root/epid582w22_class/
```

Navigating directory structure
------------------------------

Lets explore what folders we have in this course directory using the command `ls`

```
ls
```

You shoukd see a list of folders named after the uniqnames of the course attendees. You should also see a folder names `shared_data` that will contain the data for the lab module.

Lets go to shared_data directory and explore what data folder exists in this directory.

```
cd shared_data
```

You should see a folder names class that will contain all the data that we will be working for the lab module.


Running UNIX commands
---------------------

```
ls -la

mkdir test

touch REAME.md
```