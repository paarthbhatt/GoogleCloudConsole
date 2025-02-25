# Reinforcement Learning Lab

## Task 1: Reinforcement Learning 101

### Overview
Reinforcement learning (RL) is a type of machine learning where an agent takes actions in an environment to maximize a given objective (a reward) over a sequence of steps. Unlike traditional supervised learning, every data point is not labeled, and the agent only has access to "sparse" rewards.

### Key Algorithms
Two powerful and easy-to-implement deep RL algorithms that have gained attention recently are Deep Q-Network (DQN) and Deep Deterministic Policy Gradient (DDPG).

#### Deep Q-Network (DQN)
- Introduced by Google DeepMind in a 2015 Nature paper.
- Incorporates deep neural networks into Q-Learning.
- Tested in the Atari Game Engine Simulator.
- Uses a deep neural network as a function approximator to predict Q-values.
- Employs a second deep neural network to estimate target values.

### Conceptual Process Diagram
A conceptual process diagram of the Reinforcement Learning problem is provided to illustrate the workflow.

### Practical Implementation
The following model highlights the source files, shell command, and endpoint to get an RL job running on Google Cloud.

## Task 2: Create a Vertex AI Workbench Instance

### Step 1: Enable APIs
In the Google Cloud console, from the Navigation menu, select Vertex AI and click "Enable All Recommended APIs."

### Step 2: Create a New Instance
1. On the left-hand side, click "Workbench."
2. At the top of the Workbench page, ensure you are in the Instances view.
3. Click the "Create New" button.
4. Configure the instance:
   - **Name**: `lab-workbench`
   - **Region**: Set to your desired region.
   - **Zone**: Set to your desired zone.
   - **Advanced Options**: Customize as needed (e.g., machine type, disk size).
5. Click "Create."

### Step 3: Launch JupyterLab
1. Wait for the instance creation to complete (a green checkmark will appear next to its name).
2. Click "Open JupyterLab" next to the instance name to launch the JupyterLab interface.
3. In JupyterLab, click the Terminal icon to open a terminal window.

## Task 3: Copy the Sample Code

### Step 1: Copy Notebook File
Run the following command in the terminal to copy the notebook file:
```sh
gcloud storage cp -r gs://project_id-labconfig-bucket/* .
```

### Step 2: Open Notebook
1. From the left-hand menu, select `early_rl` > `notebook name`. This will open the notebook in a new tab.
2. Click "Check my progress" to verify the objective.

## Task 4: Run Through the Notebook

### Instructions
1. Ensure you have selected the Python 3 kernel in the notebook.
2. Read through the notebook and run all code blocks using `Shift + Enter`.
3. Return to the lab instructions after completing the notebook.

### Conclusion
Congratulations! In this lab, you learned the basic principles of reinforcement learning (RL). After creating a JupyterLab instance, you cloned a sample repository and ran through a notebook where you received hands-on practice with the fundamentals of RL. You are now ready to take more labs in this series.