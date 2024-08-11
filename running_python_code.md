# Project Setup Guide

This guide will walk you through the steps to set up and run a Python 3 application on a VPS using a virtual environment. The application/program will stay running 
in the background, even while logged out of VPS.

## Prerequisites

- A VPS with a Unix-based operating system (e.g., Ubuntu).
- Root or sudo access to install packages.

## Steps

### 1. Update and Upgrade System Packages

First, update and upgrade the system packages to ensure you have the latest versions:

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Essential Packages

Install essential packages and development tools, including Python 3 and `git`:

```bash
sudo apt install -y python3 python3-venv python3-pip git build-essential
```

- `python3`: The Python 3 interpreter.
- `python3-venv`: The package for creating virtual environments.
- `python3-pip`: The package manager for Python 3.
- `git`: For version control.
- `build-essential`: Development tools required for compiling some packages.

### 3. Clone the Git Repository

Clone the Git repository containing your Python project:

```bash
git clone https://github.com/yourusername/your-repo.git
```

Replace `https://github.com/yourusername/your-repo.git` with the URL of your Git repository.

### 4. Navigate to the Project Directory

Change to the directory of the cloned repository:

```bash
cd your-repo
```

Replace `your-repo` with the name of your repository folder.

### 5. Remove Existing Virtual Environment (if any)

If there is an existing virtual environment that you want to remove, you can do so with:

```bash
rm -rf venv
```

### 6. Initialize a New Virtual Environment

Create a new virtual environment in the `venv` directory:

```bash
python3 -m venv venv
```

### 7. Activate the Virtual Environment

Activate the virtual environment using the following command:

```bash
source venv/bin/activate
```

### 8. Install Required Dependencies

Install the required Python packages listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 9. Run the Python Script in the Background

To run your Python script in the background, use `nohup`:

```bash
nohup python3 main.py &
```

Replace `main.py` with the name of your main Python file. The `nohup` command allows the process to continue running even if you disconnect from the SSH session.

## Additional Notes

- To stop the background process, you can use `ps aux | grep python` to find the process ID and `kill <PID>` to terminate it.
- Make sure to periodically check logs and manage your application as needed.
