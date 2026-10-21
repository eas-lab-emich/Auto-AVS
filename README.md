# Read me - Running Auto-AVS on AWS

The Auto-Autophagic Vacuole Simulator (Auto-AVS) program generates simulated yeast vacuoles filled with autophagic bodies, takes random slices, and then compares the size and number of the simulated autophagic body cross-sections with the size and number of the real autophagic body cross-sections you have generated through Transmission Electron Microscopy (TEM) imaging. By running a series of simulations and comparing the simulated results of each with your real results, you can generate an estimate of the actual size and number of the bodies in your sample. You should start with varying the size of the bodies to generate an estimate of body size, and then keep that constant as you vary the number of bodies (this is best because observed size is independent of number, but observed number is partially dependent on size).

These instructions are for running Auto-AVS on an Amazon Web Services (AWS) Linux Server. This is the primary strategy that we use and recommend. It can also be run on a local linux machine with python installed; running on Windows might require some modifications to the code (e.g. file path conventions).

- [Initial Setup](#initial-setup)
- [Connecting to the EC2 instance with MobaXterm](#connecting-to-the-ec2-instance-with-mobaxterm)
- [Installing Miniconda](#installing-miniconda)
- [Installing tmux](#installing-tmux)
- [Downloading Auto-AVS from Github](#downloading-auto-avs-from-github)
- [Installing CC3D](#installing-cc3d)
- [Installing packages](#installing-packages)
- [Running the Pipeline](#running-the-pipeline)

## Initial Setup

1. Login to AWS
2. Navigate to EC2
3. Click on “Instances”
4. If need be, create a new Instance with “Launch an instance”
   - a. Choose Ubuntu
   - b. 64-bit (x86)
   - c. Recommended Instance type: r6a.large (2 cores, 16 GB RAM. CC3D uses only 1 core, but needs sufficient RAM for the simulation size.)
   - d. Set up a key pair for access and an appropriate security group
   - e. Recommend 50GB Root Volume (under “Configure Storage”)
5. Connect to the instance using MobaXterm ([see below](#connecting-to-the-ec2-instance-with-mobaxterm))
6. Install miniconda ([see below](#installing-miniconda))
7. Install tmux ([see below](#installing-tmux))
8. Download the Auto-AVS repository from Github ([see below](#downloading-auto-avs-from-github))
9. Install CC3D ([see below](#installing-cc3d)).
10. Install packages ([see below](#installing-packages))

Now you are ready to [run the pipeline](#running-the-pipeline)!

## Connecting to the EC2 instance with MobaXterm

Instructions on installing and using MobaXterm are here: https://www.freecodecamp.org/news/connect-to-your-ec2-instance-using-mobaxterm/

1. In MobaXterm, choose “Start new session”, then select “SSH” as the session type
2. In “remote host”, enter the Public IP Address of the EC2 instance
3. In “Advanced SSH Settings”, check the “Use private key” option, and browse for the correct .pem file.
4. As the username, the default is ubuntu

## Installing Miniconda

Instructions at https://dev.to/monikaprajapati_70/quick-guide-installing-conda-on-ec2-the-right-way-2aoc

```bash
# Paste the following commands one line at a time into the command line
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p $HOME/miniconda
~/miniconda/bin/conda init bash
# When prompted to close and reopen, type “exit” to close the shell, then “R” to reconnect
source ~/.bashrc
sudo apt install -y libgl1 libglx-mesa0 libxrender1 libsm6 libxext6
conda clean -a -y
```

## Installing tmux

Tmux allows you to detach from sessions so they can keep running on the server when you are no longer connected. Instructions on how to use it are here: https://hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/

```bash
#To install tmux, paste the following command in the command line
sudo apt-get install tmux
```

## Downloading Auto-AVS from Github

1. Clone Repository
   ```bash
   #Execute the clone command from the home directory:
   cd ~
   git clone https://github.com/eas-lab-emich/Auto-AVS.git
   ```
2. Access Repository
   ```bash
   #Navigate to the cloned repository:
   cd Auto-AVS
   ```
3. Update Repository
   ```bash
   #from the Auto-AVS folder, enter the command
   git pull
   ```
   If there are conflicts, you can instead overwrite the local server with the GitHub version using the command below.

   > **Warning:** this permanently erases any changes you have made to your local copy, including your edits to Model_Parameters.txt. Save a copy of anything you want to keep first.

   ```bash
   git fetch origin
   git reset --hard origin/main
   ```

## Installing CC3D

```bash
# Paste these commands one line at a time into the command line and follow any on-screen prompts
#(some of these commands may take a little while to execute)
cd src
conda create -n cc3d python=3.12
conda activate cc3d
conda install -c conda-forge -c compucell3d compucell3d=4.8.0
wget https://sourceforge.net/projects/cc3d/files/4.8.0/linux/cc3d-installer-linux-4.8.0-x86-64bit.sh
chmod +x cc3d-installer-linux-4.8.0-x86-64bit.sh
./cc3d-installer-linux-4.8.0-x86-64bit.sh
```

Using the MobaXterm file browser, create a new folder called “cc3dSimulation” within the CompuCell3D folder. Then, upload the simulation file and the associated Simulation folder into the cc3dSimulation folder (you can do this in the file browser by a simple drag and drop). An example cc3d simulation is provided for you to use.

To have Pipeline_Interface.py invoke cc3d we have to:

1. Update the runScript.sh file (in the CompuCell3D folder) to be the following:
   ```bash
   #!/bin/bash
   SCRIPT_DIR="$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
   source ${SCRIPT_DIR}/miniconda3/bin/activate base
   export exit_code=0
   python -m cc3d.run_script $* --current-dir=${SCRIPT_DIR}
   exit_code=$?
   cd ${SCRIPT_DIR}
   exit ${exit_code}
   ```
2. Run the following commands:
   ```bash
   sed -i 's/\r$//' /home/ubuntu/CompuCell3D/runScript.sh
   chmod +x /home/ubuntu/CompuCell3D/runScript.sh
   ```

## Installing packages

From the folder Auto-AVS (may need to access with `cd ..`), run

```bash
conda activate cc3d
pip install -r requirements.txt
```

## Running the Pipeline

1. Connect to the instance you set up using MobaXterm
2. Modify the run parameters by using the MobaXterm file browser to navigate to Auto-AVS/src/attributes and editing the values in the file “Model_Parameters.txt”. Save your changes
3. Run the pipeline (from the home directory, base environment) with the following commands:
   ```bash
   tmux
   cd Auto-AVS/src
   conda activate cc3d
   python Pipeline_Interface.py
   ```
4. Follow the on-screen instructions from there.
   - a. Usually you will use either option 1, to run the pipeline with CC3D to deform the bodies, or option 4, to run the pipeline with spherical bodies, without CC3D (which is much faster).
5. Once the run is started, use “C-b d” (ctrl+b, then d - see tmux help) to detach from the session so it can keep running while you close the MobaXterm window. To reattach, type
   ```bash
   tmux attach -t 0
   ```
6. When the run is done, you will find your data in Auto-AVS/src/runs, in a timestamped folder.
7. To compare the simulated data you generated with your real data in order to generate an estimate for the original size and number of the autophagic bodies in your sample, choose option 6 (AVSStats) from the Pipeline_Interface.py menu and follow the on-screen instructions.
   - a. Some example data (from actual TEM images of yeast autophagic bodies) is provided for you to play around with; to generate your own TEM data, see the instructions in Backues et al. (2014) Autophagy 10(1), 155–164. https://doi.org/10.4161/auto.26856
