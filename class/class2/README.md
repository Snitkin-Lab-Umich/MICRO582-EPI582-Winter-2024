Class 2 – Introduction to Great Lakes and HPC
=============================================

Goal
----

- In this module, we will review the course's lab directory organization
- We will set up the compute environment for upcoming lab modules so that we have all the tools/pipelines installed and ready to use
- We will learn how to load Great lakes modules which are the pre-installed softwares provided by ARC-TS team.
- We will learn how to submit a cluster job.

Reviewing directory navigation
------------------------------

Setting up your compute environment
-----------------------------------

**Setting up environment variables in .bashrc file so your environment is all set for genomic analysis!**

Environment variables are the variables/values that describe the environment in which programs run in. All the programs and scripts on your unix system use these variables for extracting information such as: 

- What is my current working directory?, 
- Where are temporary files stored?, 
- Where are perl/python libraries?, 
- Where is Blast installed? etc. 

In addition to environment variables that are set up by system administators, each user can set their own environment variables to customize their experience. This may sound like something super advanced that isn't relevant to beginners, but that's not true! 

Some examples of ways that we will use environment variables in the class are: 

1) create shortcuts for directories that you frequently go to,

2) Setup a conda environment to install all the required tools and have them availble in your environment

3) setup a shortcut for getting on a cluster node, so that you don't have to write out the full command each time.

One way to set your environment variables would be to manually set up these variables everytime you log in, but this would be extremely tedious and inefficient. So, Unix has setup a way around this, which is to put your environment variable assignments in special files called .bashrc or .bash_profile. Every user has one or both of these files in their home directory, and what's special about them is that the commands in them are executed every time you login. So, if you simply set your environmental variable assignments in one of these files, your environment will be setup just the way you want it each time you login!

All the softwares/tools that we need in this workshop are installed in a directory "/scratch/micro612w21_class_root/micro612w21_class/shared/bin/" and we want the shell to look for these installed tools in this directory. For this, We will save the full path to these tools in an environment variable PATH.

> ***i. Make a backup copy of bashrc file in case something goes wrong.***
	
```

cp ~/.bashrc ~/bashrc_backup

#Note: "~/" represents your home directory. On great lakes, these means /home/username

```
	
> ***ii. Open ~/.bashrc file using any text editor and add the following lines at the end of your .bashrc file.***

***Note: Replace "username" under alias shortcuts with your own umich "uniqname".***

```
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/sw/arcts/centos7/python3.7-anaconda/2019.07/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/sw/arcts/centos7/python3.7-anaconda/2019.07/etc/profile.d/conda.sh" ]; then
        . "/sw/arcts/centos7/python3.7-anaconda/2019.07/etc/profile.d/conda.sh"
    else
        export PATH="/sw/arcts/centos7/python3.7-anaconda/2019.07/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<

##Micro612 Workshop ENV

#Aliases
alias islurm='srun --account=micro612w21_class --nodes=1 --ntasks-per-node=1 --mem-per-cpu=5GB --cpus-per-task=1 --time=12:00:00 --pty /bin/bash'
alias wd='cd /scratch/micro612w21_class_root/micro612w21_class/username/'
alias d1m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day1am'
alias d1a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day1pm'
alias d2m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day2am'
alias d2a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm'
alias d3m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day3am'
alias d3a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day3pm'


#Great Lakes Modules. They should remain commented for the day1 session.
#module load Bioinformatics
#module load perl-modules



#Bioinformatics Tools
export PATH=$PATH:/scratch/micro612w21_class_root/micro612w21_class/shared/bin/quast/

```



Note: Replace "username" under alias shortcuts with your own umich "uniqname". In the text editor, nano, you can do this by 

- typing Ctrl + \ and You will then be prompted to type in your search string (here, username). 
- Press return. Then you will be prompted to enter what you want to replace "username" with (here, your uniqname). 
- Press return. Then press a to replace all incidences or y to accept each incidence one by one. 

You can also customize the alias name such as wd, d1am etc. catering to your own need and convenience.

The above environment settings will set various shortcuts such as "islurm" for entering interactive great lakes session, "wd" to navigate to your workshop directory, call necessary great lakes modules and perl libraries required by certain tools and finally sets the path for bioinformatics programs that we will run during the workshop.

> ***iii. Save the file and Source .bashrc file to make these changes permanent.***

```

source ~/.bashrc

```

> ***iv. Check if the $PATH environment variable is updated***

```

echo $PATH

#You will see a long list of paths that has been added to your $PATH variable

wd

```

You should be in your workshop working directory that is /scratch/micro612w21_class_root/micro612w21_class/username 

> ***v. Set up a conda environment using a YML file***

The YML file - `micro612.yml` required for generating the conda environment is located here:

```
/scratch/micro612w21_class_root/micro612w21_class/shared/conda_envs/
```

Load great lakes anaconda package and set up a conda environment in the following way - 

```
# Load anaconda package from great lakes 
module load python3.7-anaconda/2019.07

# Set channel_priority to false so that it can install packages as per the YML file and not from loaded channels.
conda config --set channel_priority false

# Create a new conda environment - micro612 from a YML file
conda env create -f /scratch/micro612w21_class_root/micro612w21_class/shared/conda_envs/micro612.yml -n micro612

# Load your environment and use the tools
conda activate micro612

# Check if the tools were properly installed by conda and are callable from your environment 
bash /scratch/micro612w21_class_root/micro612w21_class/shared/conda_envs/check_micro612_installation.sh 

# Problem installing PyVCF and biopython with Conda channels
pip install PyVCF --user
pip install biopython --user

# Update one of the databases that you would need in one of the Kraken exercises 
ktUpdateTaxonomy.sh
```

Loading modules
---------------


Submit a job to cluster
-----------------------




