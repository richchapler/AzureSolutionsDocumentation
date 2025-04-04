# Jupyter

## Install pip (if not already installed)

1. **Check if pip is installed:**  
   Open PowerShell and run:
   ```powershell
   pip --version
   ```
   If you see an error like:
   ```
   pip: The term 'pip' is not recognized as a name of a cmdlet, function, script file, or executable program.
   ```
   then pip is not installed or not added to your PATH.

2. **Download get-pip.py:**  
   Use the following command to download the pip installer:
   ```powershell
   curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
   ```

3. **Run the installer:**  
   Execute the script with Python:
   ```powershell
   python get-pip.py
   ```
   *Note: If the `python` command is not recognized, ensure Python is installed and added to your system's PATH.*

4. **Verify the installation:**  
   After installation, check the pip version again:
   ```powershell
   pip --version
   ```
   You should now see output similar to:
   ```
   pip 23.x.y from C:\PythonXX\lib\site-packages\pip (python 3.X)
   ```

## Installed?

Open PowerShell as an administrator and execute the following command to check if Jupyter is installed:

```powershell
jupyter --version
```

Expected output (version may vary):

```
6.5.4
```

## Not installed...

Execute the following PowerShell command to install Jupyter:

```powershell
pip install jupyter
```

## Install Jupyter Extension in Visual Studio Code

Open Visual Studio Code and navigate to Extensions (`Ctrl+Shift+X`).

<img src="https://github.com/user-attachments/assets/3ac8dc62-cc5a-4264-992f-a809c0524c55" width="800" title="Snipped February 5, 2025" />

Search for and select "Jupyter" and click "Install".

<img src="https://github.com/user-attachments/assets/d618a771-041c-40a2-a351-6c63cc51d0bc" width="800" title="Snipped February 5, 2025" />

## Verify Jupyter Notebook Functionality

1. Open Visual Studio Code
2. Click **File** > **New File**, then search for and select **Jupyter Notebook**

<img src="https://github.com/user-attachments/assets/066afd03-d3ff-435e-a651-ae6a28a57bc3" width="800" title="Snipped February 5, 2025" />

3. Click **View** > **Command Palette** (`Ctrl+Shift+P`)
4. Search for and select **Python: Select Interpreter**

<img src="https://github.com/user-attachments/assets/bfd68ebc-350c-421e-9ec1-1f1f8fd3b35e" width="800" title="Snipped February 5, 2025" />

5. Select the **Recommended** interpreter.

<img src="https://github.com/user-attachments/assets/4abe9278-ecf8-49bd-aa00-330e06fed88b" width="800" title="Snipped February 5, 2025" />

