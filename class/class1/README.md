Class 1 – Setting up Great Lakes account and Introduction to UNIX
=================================================================

Goal
-----

- In this module, we will log into Great Lakes account and get access to the course working directory.  
- Get acquanted with the basics of UNIX file system.
- Learn how to navigate course directory structure using command line interface.
- Run basic UNIX commands to explore the course working directory.

Sign up for Great Lakes account
-------------------------------
**Log in to great lakes**

Log into your great lakes account using the ssh command below. SSH stands for secure shell and can be used to connect or transfer data to/from remote hosts such as Great Lakes.

Replace username with your umich uniqname and run the ssh command to log in to Great Lakes.

```
ssh username@greatlakes.arc-ts.umich.edu
```

***The directory on Great Lakes that you will be working in is - `/scratch/epid582w22_class_root/epid582w22_class/`***

You can find a folder named after your umich uniqname; for example if my uniqname is `apirani`, I will run all my analysis in `/scratch/epid582w22_class_root/epid582w22_class/esnitkin/`.

Intro to Unix
-------------
 
 UNIX systems allows users to perform complex and powerful tasks, often with just a few keystrokes/commands or lines of code. It helps users automate repetitive tasks and easily combine smaller tasks into larger, more powerful workflows. Bioinformatics data analyis involves generation of lots of files with different types of extensions. Exploring these files using a GUI becomes cumbersome when you have different combination of files/folder for different samples. Therfore, use of the shell is fundamental to a wide range of advanced data analysis and high-performance computing tasks.

Once you log in, By default Great lakes will take you to your home directory which should be something like - `/home/uniqname` where uniqname will be your username that we used for logging in. At any point in time, You can check your current location in UNIX file system with `pwd` command. 

`pwd` stands for present working directory and is a very handy command to quickly find out where you are in the file system. This lets you then move around and go to your desired directory. Lets run this command and find where we actually are.

```
pwd
```

My present working directory is `/home/esnitkin`

The course working direcory will be `/scratch/epid582w22_class_root/epid582w22_class/`. So lets use UNIX's `cd` command to go to the course directory. 

`cd` stands for change directory and it lets you move around the file system using the absolute/relative paths.

```
cd /scratch/epid582w22_class_root/epid582w22_class/
```

Navigating directory structure
------------------------------

Lets explore what folders we have in this course directory using the command `ls`

The `ls` command lists the names of files in a particular Unix directory. If you type the ls command with no parameters or qualifiers, the command displays the files listed in your current working directory.

```
ls
```

You should see a list of folders named after the uniqnames of the course attendees. You should also see a folder named `shared_data` that contains data for the lab modules.


Lets list the folders available in shared_data directory.

```
ls shared_data
```

You should see four folders namely - bin, conda_envs, data, database.

- data folder contains all the data that we will be working on.
- conda_envs contains the conda YML file that we will use to set up our environment. This will let us install all the tools required for the lab.
- bin folder contains tools that were seperately installed due to dependency conflicts in our conda envireonment.
- database contains public databases that we will use for annotation analysis.

Data Carpentry: Introducing the Shell & Navigating Files and Directories
--------------------------------------------------------------------------

We will be working with data carpentry lessons [Introducing the Shell](https://datacarpentry.org/shell-genomics/01-introduction/index.html) and [Navigating Files and Directories](https://datacarpentry.org/shell-genomics/02-the-filesystem/index.html) to go over some basic unix commands and learn how to navigate directories.

Data for these lessons are located here - `/scratch/epid582w22_class_root/epid582w22_class/shared_data/data/class1`