# Jupyter (in Visual Studio Code)

This guide walks you through installing Jupyter via pip, adding and configuring the Jupyter extension in Visual Studio Code, and verifying that everything is set up correctly for working with Jupyter notebooks.

---

## 1. Install Jupyter

1. Open **Visual Studio Code** and then the integrated terminal by navigating to **View > Terminal**

2. Execute the following command to install Jupyter:

   ```powershell
   pip install jupyter
   ```

   <img src="https://github.com/user-attachments/assets/b86a2382-9dea-4dd4-81f9-41a191da00a3" width="800" title="Snipped: March 13, 2025" />

-------------------------

## 2. Install the Jupyter Extension

1. Navigate to the **Extensions** view in Visual Studio Code
2. In the search bar, type **Jupyter**
3. Select the **Jupyter** extension from the list and click Install

   <img src="https://github.com/user-attachments/assets/3ac8dc62-cc5a-4264-992f-a809c0524c55" width="800" title="Snipped: March 13, 2025" />
   
-------------------------

## 3. Create a New Jupyter Notebook

After installing Jupyter and its extension, verify that you can create and work with notebooks.

   <img src="https://github.com/user-attachments/assets/2e33974c-bb49-4d80-a267-b151dbcd086d" width="800" title="Snipped: March 13, 2025" />

1. In Visual Studio Code, click **File** > **New File**.
2. When prompted, search for and select **Jupyter Notebook**.

   <img src="https://github.com/user-attachments/assets/066afd03-d3ff-435e-a651-ae6a28a57bc3" width="800" title="Snipped: March 13, 2025" />

A new notebook file (typically with the extension `.ipynb`) will open, allowing you to start writing and executing cells.

---

## 4. Select the Python Interpreter

For Jupyter notebooks to work correctly, you must select the appropriate Python interpreter.

1. Click **View** > **Command Palette** in Visual Studio Code.
2. In the Command Palette, search for **Python: Select Interpreter** and select it.

   <img src="https://github.com/user-attachments/assets/bfd68ebc-350c-421e-9ec1-1f1f8fd3b35e" width="800" title="Open Command Palette" />

3. From the list of interpreters, select the **Recommended** interpreter (the one that points to your installed Python version).

   <img src="https://github.com/user-attachments/assets/4abe9278-ecf8-49bd-aa00-330e06fed88b" width="800" title="Select Recommended Interpreter" />

Selecting the correct interpreter ensures that your Jupyter notebook uses the proper Python environment.
