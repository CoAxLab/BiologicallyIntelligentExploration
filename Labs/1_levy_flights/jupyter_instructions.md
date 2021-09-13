## Python Resources

Python is an object-oriented language with great strengths in scientific computing, including incredible packages for neuroimaging, psychophysical experimentation, and machine learning. 

Python code can be written and executed in a variety of ways:
  - Arbitrary text editor (e.g. VIM, Atom) + standard unix terminal
  - Integrated Development Environment (IDE), e.g. Spyder or PyCharm
  - Jupyter Notebooks locally through regular Jupyter Notebooks or JupyterLab (which has many more features and many of the benefits of an IDE)
  - Jupyter Notebooks remotely, through regular Jupyter Notebooks, JupyterLab, JupyterHub, Google Colab Notebooks, Binder, etc.

We highly encourage you to use the Anaconda distribution of Python, which ships with many scientific packages, the Spyder IDE, and jupyter notebooks installed, along with its most important feature - the `conda` package manager. 

### Python Tutorial

https://github.com/clappm/uPNC_2021/blob/master/tutorial.ipynb
  
### Install Anaconda
Follow [these instructions](https://docs.anaconda.com/anaconda/install/)
Miniconda is the better choice for people who do not have a lot of available disk space. You can always download the rest of the packages later with `conda install` or `pip install`.

### Verify your installation
Make sure that your installation is working. Open the Anaconda Application and try to launch the Spyder IDE or a local Jupyter Notebooks server. 

Now open up a terminal (Mac or Linux) or Anaconda command prompt (Windows).
```bash
# check that your system is now using Anaconda's python
which python

# and that you installed Python 3.8
python -V
```
Don't worry too much about the version - it's good to have your base install have the newest version of Python, but you can always create virtual environments later with earlier versions of Python to, e.g., build upon someone else's code which is only compatible with earlier versions.



# Running Jupyter Notebooks Locally

### Install JupyterLab
Within your teminal/command prompt, you now have access to the conda package manager. You should read up about it [here](https://docs.conda.io/projects/conda/en/latest/index.html)
Use conda to install Jupyterlab:
```bash
# jupyterlab is located on the conda-forge channel, along with many other packages
conda install -c conda-forge jupyterlab
```

### Launch a local JupyterLab server
Launching a jupyterlab server is now as easy as: 
```bash
jupyter-lab
# now open a browser and go to the address given by the jupyter-lab terminal output
```
Explore the interface. Much better than regular Jupyter Notebooks. Although still lacking in some areas. For that, we can install (or even develop!) extensions. I recommend installing the following 3 which are sure to improve your experience:
```bash
# we will need nodejs for the extensions
conda install nodejs

# a file tree
jupyter labextension install jupyterlab_filetree

# table of contents for notebooks (super awesome!!!)
jupyter labextension install @jupyterlab/toc

# variable explorer (similar to those offered in various IDEs)
jupyter labextension install @lckr/jupyterlab_variableinspector

```
After you install the extensions, you might need to restart your JupyterLab server for them to take effect. If they are working, you will see extra icons on the left side of JupyterLab. 

Many more useful extensions can be found [here](https://awesomeopensource.com/projects/jupyterlab-extension)

### (extra) Set up a remote JupyterLab server
For this step, you need to have a remote machine to SSH in to. We will pretend that you have an environment variable pointing to its address called `SERVER_ID`. We will assume you are logging into the CNBC Mind cluster. Other clusters using SLURM will require a nearly identical approach. Launching a remote JupyterLab server is incredibly useful, and allows you to run interactive jobs, and plotting, all in one environment and taking advantage of the resources of your computing cluster. For example, you can work on code that requires a GPU or more RAM than your local machine has available. Cool. It is the same basic idea as launching a local server, with a couple added ingredients. 
  - an interactive job launched on the remote server in which you can run jupyterlab, ideally within a `screen` or `tmux` session to allow you to leave the jupyterlab server running after disconnecting your SSH connection. 
  - SSH piping. We will assume you have an environment variable `PORT=8888` for example, but we will refer to it by the variable name so you can use a port that is available.  

Follow these instructions to get it going (credit to the approach taken here goes to PNC/ML Ph.D. student Jayanth Koushik).
```bash
# log into the remote cluster and pipe between localhost port $PORT locally and remotely 
ssh -L $PORT:localhost:$PORT $SERVER_ID
# we are now in the server
# we want to be able to end our SSH session and resume where we left off.
# let's use a screen session (tmux would also work). we will name the session jupyterlab for easy access later
screen -S jupyterlab
# we are now in the screen session
# let's run an interactive CPU job
srun -p cpu --cpus-per-task=6 --gres=gpu:0 --mem=20GB --time=14-00:00:00 --pty bash
# we are now in the interactive job
# we need the ssh port to reach inside the computing node we are on
ssh -N -f -R $PORT:localhost:$PORT $SLURM_SUBMIT_HOST
# now when we launch the jupyterlab server, we will be able to access it locally!
jupyter-lab --no-browser --port=$PORT
```
Now, open a browser and go to `localhost:$PORT/lab` (but type in the port number) 

You can kill your SSH session, turn off your computer, etc. and the jupyterlab server will still be there (until it is killed remotely or your interactive job expires). Just remember to pipe through the correct port when you SSH in.

### Create a conda virtual environment 
For simple Python use, one base installation is usually fine. But once you start working on multiple projects, or interacting with other people's code, it gets to be very dangerous to do all of this in one environment. Further, it makes reproducing your results much more difficult. Virtual environments were developed to solve this problem. Python has virtual environments natively, but we recommend using conda environments which are even more isolated than virtual environments (at the expense of greater disk use) and can control the environment beyond just the python packages. For more insight into the differences between conda and standard virtual environments, do some googling. Additionally, we have provided a solution to using conda environments within JupyterLab, whereas we do not know how to do so for python virtual envs (there is probably a solution somewhere...). 

Open a terminal/command prompt and create a conda virtual environment:
```bash
# using the base python version, i.e. 3.7
conda create --name some_name
# using the base python version, and additionally installing the full anaconda distrubiton
conda create --name some_name anaconda
# if for some reason you need python 2
conda create --name some_name_2 --python=2.7
# for help on the env creation function
conda env --help
```

I recommend installing the conda kernels extension, which will make it easy to switch between different virtual environments (e.g. for different projects with different requirements) within the same JupyterLab server. 
```bash
# assuming you are launching jupyterlab from the base environment
conda install nb_conda_kernels
# then just install ipykernel in other environments to allow them to show up in jupyterlab from the base env
conda activate some_name
conda install ipykernel
```

