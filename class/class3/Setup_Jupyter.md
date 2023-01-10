Setting up Jupyter Notebook with Great Lakes Open OnDemand
----------------------------------------------------------

Advanced Research Computing's Open OnDemand offers several interactive applications for a user to use: RStudio, Jupyter Lab and Notebook, MATLAB, Spark and the ability to set up various Linux remote desktops on Great Lakes or Lighthouse. For all our subsequent class labs, we will be running genomics analysis in a Jupyter Notebook environment provided by Advanced Research Computing's Open OnDemand. Going forward, We will call opening OnDemand interactive apps as an interactive notebook session.

More details about different Advanced Research Computing's Open OnDemand offerings can be found [here](https://arc.umich.edu/open-ondemand/)

Lets set the interactive notebook session.

- Step 1

    Go to Great Lakes interactive session [link](https://greatlakes.arc-ts.umich.edu/pun/sys/dashboard/) and login with your UMICH uniqname and Level-1 password. 
    
    This will land you to the Great Lakes Homepage. 
    
    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/GL-Homepage.png)

    At the top bar of the page, you would find dropdown buttons that lets us perform various interactive tasks using a web server instead of an terminal. Lets go through each of these buttons so that we are comfortable with the interactive session.

   ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/GL-Homepage_Highlights_2.png)

    ***Files:*** If you click "Files", you would get an option to go to the Home Directory. This would be your Great Lakes Home directory path (/home/username) that you usually land up when you login to your great lakes accounts from command line.
    
    ***Jobs:*** "Jobs" will give two options - "Active Jobs" and "Job Composer". Active Jobs shows you the jobs that are running on cluster under your account. "Job Composer" lets you compose and submit a job from a specified location. Composing a job and submitting it from here is similar to creating a sbatch script and submitting it with sbatch on command line.
    
    ***Clusters***: Clusters button lets you open a great lakes login terminal from a webpage.

    ***Interactive Apps:*** "Interactive Apps" button lets you start an interactive app such as Jupyter Notebook, Rstudio etc. 

- Step 2

    Jupyter Notebook allows us to create and share documents with other researchers. We can integrate live code, equations, computational output, visualizations, and other multimedia resources, along with explanatory text in a single document. This feature makes Jupyter Notebooks one of the most popular tool for reproducing projects. 

    Lets go ahead and start the Jupyter Notebook session by clicking "Interactive Apps" -> "Jupyter Notebook"

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Jupyter-Notebook-Interactive-App-highlight.png)

    This will land you to the Job composer page. Make sure you use "epid582w23_class" as the slurm account name under which you will start this job. If all the parameters looks okay, hit the launch button.

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Jupyter-Notebook-Job-Start-highlights-job.png)

    You would see a page that would look something like the below page. This indicates that your job is in queue and is about to start.

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Session_starting.png)

- Step 3

    When the jobs starts, it will give you an option to "Connect to Jupyter". 

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Connect-to-jupyter.png)

    Hit that button and you will land to your home directory and will look something like the image below. Note: As a default, the Jupyter Notebook navigation will always start from your home directory.

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Jupyter-Home.png)

    Now, click on the folder icon as shown below "Select items to perform actions on them." line (See below screenshot for reference). 

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/link-to-root_final.png)

    Click on scratch -> epid582w23_class -> "your username" -> class3 -> Setup_Jupyter.ipynb

    This will open up jupyter notebook that we will run to set up the notebook environment.

- Step 4

    Jupyter Notebook requires you to have your Bash Kernel setup in order to run Bash commands. (Kernels are programming language specific processes that run independently and interact with the Jupyter Applications and their user interfaces.)

    We installed Bash kernel while setting up our conda environment in our previous steps. Now, we will make sure that we are able to run the commands from this notebook. 

    Let check if the kernel is installed. Click on "Kernel" at the top dropdown and select "Change Kernel". Select the Bash option. If you dont see a Bash option, this means that the Bash kernel was not installed/setup properly.

    ![alt tag](https://raw.githubusercontent.com/Snitkin-Lab-Umich/MICRO582-EPI582-Winter-2023/main/class/class3/img/Bashkernel.png)

    Lets run a few bash commands (from notebook - go to the end of notebook) to check if we can run Bash commands.