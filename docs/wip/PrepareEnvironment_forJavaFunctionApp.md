1. **Install IntelliJ IDEA**: You can download it from the official JetBrains website (Download IntelliJ IDEA – The Leading Java and Kotlin IDE (jetbrains.com)). Choose the version that suits your needs (Community or Ultimate). After downloading, install it following the instructions.    
  
2. **Install Java on your system**:  
   1. Download Java JDK: Go to the Oracle website (for Java 11) or AdoptOpenJDK (for other versions). Download the appropriate JDK for your operating system.  
   2. Install Java JDK: Run the installer and follow the instructions to install the JDK on your system.  
   
3. **Set JAVA_HOME on Windows**:  
   1. Right-click on My Computer or This PC and select Properties.  
   2. Click on Advanced system settings.  
   3. In the System Properties window that opens, go to the Advanced tab and click on Environment Variables.  
   4. In the Environment Variables window, under System variables, click New.  
   5. In the New System Variable window, enter JAVA_HOME as the variable name and “C:\Program Files\Java\jdk-22” as the variable value (NO TRAILING WHACK and DOUBLE QUOTES ARE IMPORTANT)  
   6. Click OK in all windows to apply the changes.  
   
4. **Test by entering this directly in a command line**: "C:\Program Files\Java\jdk-22\bin\java" -version  
   
5. **To add the JDK to your Path, follow these steps**:  
   1. Press Win + X and choose System.  
   2. Click on Advanced system settings.  
   3. In the System Properties window that appears, click on Environment Variables.  
   4. In the Environment Variables window, under System variables, find and select the Path variable, then click on Edit.  
   5. In the Edit Environment Variable window, click on New.  
   6. Type %JAVA_HOME%\bin and press Enter.  
   7. Click OK in all windows to apply the changes.  
   
6. **Install Maven and add it to your PATH**:  
   1. Download Maven: You can download Maven from the official Apache Maven website. Choose the binary zip archive (apache-maven-3.x.x-bin.zip).  
   2. Extract the zip file: Once the download is complete, extract the zip file to a directory on your system. For example, you might extract it to C:\Program Files\Apache\Maven.  
   3. Add Maven to your PATH:  
      1. On Windows:  
         1. Search for "Environment Variables" in your start menu and select "Edit the system environment variables".  
         2. In the System Properties window that appears, click on "Environment Variables".  
         3. In the Environment Variables window, under "System variables", find the Path variable, select it, and click on "Edit".  
         4. In the Edit Environment Variable window, click on "New", and then add the path to the bin directory of your Maven installation. For example, if you extracted Maven to C:\Program Files\Apache\maven, you would add C:\Program Files\Apache\maven\bin.  
         5. Click "OK" in each window to close them.  
   
7. **Install Azure Toolkit for IntelliJ**: This toolkit allows you to develop, run, and debug Azure applications directly from IntelliJ IDEA.  
   - Open IntelliJ IDEA.  
   - Go to `File -> Settings -> Plugins`.  
   - In the search bar, type `Azure Toolkit for IntelliJ` and click on the search result.  
   - Click on `Install` and wait for the installation to complete.  
   - Restart IntelliJ IDEA to complete the installation.  
   
8. **Create a new Azure Functions project**: Azure Functions is a serverless solution that allows you to write less code, maintain less infrastructure, and save on costs.  
   - In IntelliJ IDEA, click on `File -> New -> Project`.  
   - In the new window, select `Azure Functions` under `Azure` on the left panel.  
   - Choose your project SDK (Software Development Kit). If you have installed the JDK as per the previous steps, it should appear in the dropdown menu.  
   - Click `Next`.  
   - Enter your project name and choose a location on your system where the project files will be stored.  
   - Click `Finish` to create the project.  
   
9. **Write your function code**: Now, you'll write the code for your Azure Function.  
   - In the `Project` window, navigate to `src/main/java/<YourPackageName>`.  
   - Right-click on the package and select `New -> Azure Function`.  
   - In the new window, enter the function name and choose the trigger type (for example, HttpTrigger). The trigger determines how your function is invoked. An HttpTrigger means your function will run whenever it receives an HTTP request.  
   - Click `OK` to create the function. IntelliJ IDEA will generate a function class with a method that gets called when your function is triggered.  
   
10. **Test your function locally**: Before deploying your function to Azure, you should test it locally to make sure it works correctly.  
   - Right-click on your project and select `Run -> Functions: host start`.  
   - IntelliJ IDEA will start the Azure Functions runtime and your function will be ready to test locally.  
   
11. **Deploy your function to Azure**: Once you're satisfied with your function, you can deploy it to Azure to make it publicly accessible.  
   - Right-click on your project and select `Deploy -> Deploy to Azure`.  
   - In the new window, sign in to your Azure account. If you don't have one, you'll need to create one.  
   - Select your subscription, and choose or create a new Function App. The Function App is a way to organize and collectively manage your functions.  
   - Click `Run` to deploy your function to Azure.  

12. **Install Azurite**: Azurite is a lightweight Azure Storage emulator that is ideal for testing your Azure applications locally. It emulates Azure Blob, Queue, and Table services.    
   1. Install Azurite using npm (Node Package Manager): Open a command prompt and run the following command: `npm install -g azurite`. If you don't have npm installed, you can download it from the official Node.js website.    
   2. Create a directory for Azurite: For example, you can create a directory at `C:\azurite`.    
     
13. **Start Azurite**:    
   1. Open a new command prompt window.    
   2. Navigate to the directory where you installed Azurite.    
   3. Run the following command:

      ```shell
      azurite --silent --location C:\azurite --debug C:\azurite\debug.log --blobHost 127.0.0.1 --blobPort 10002 --queuePort 10003 --tablePort 10004  
      ```  
      In this command:    
      - `--silent` runs Azurite in silent mode.    
      - `--location C:\azurite` specifies the directory where Azurite should store its data.    
      - `--debug C:\azurite\debug.log` specifies the file where Azurite should write debug logs.    
      - `--blobHost 127.0.0.1` specifies the IP address that the Blob service should use.    
      - `--blobPort 10002` specifies the port that the Blob service should use.    
      - `--queuePort 10003` specifies the port that the Queue service should use.    
      - `--tablePort 10004` specifies the port that the Table service should use.    
     
   After running this command, each service should start on its own port.

   ![image](https://github.com/user-attachments/assets/32ad764a-d058-4d59-9dd7-17dacbe1a1b6)
     
   **Note**: The Azurite command prompt window needs to be left active during testing. If you close the window, Azurite will stop running, and your Azure application won't be able to access the emulated storage services.    
     
14. **Test your function with Azurite**: Now, you can test your Azure Function with Azurite.    
   - In IntelliJ IDEA, right-click on your project and select `Run -> Functions: host start`.    
   - IntelliJ IDEA will start the Azure Functions runtime, and your function will be ready to test locally with Azurite.    
     
15. **Deploy your function to Azure**: Once you're satisfied with your function and it works correctly with Azurite, you can deploy it to Azure.    
   - Right-click on your project and select `Deploy -> Deploy to Azure`.    
   - In the new window, sign in to your Azure account. If you don't have one, you'll need to create one.    
   - Select your subscription, and choose or create a new Function App. The Function App is a way to organize and collectively manage your functions.    
   - Click `Run` to deploy your function to Azure.

   16. Build your project: Before running your function, you need to build your project. In the root directory of your project, run the command mvn clean package. This will clean any previous build and create a new package.

17. Run your function locally: After building your project, navigate to the target\azure-functions\<function-app-name> directory in your project root and run the func start command. Replace <function-app-name> with the name of your function app.

18. Debugging: If you encounter any issues while running your function, you can use the --verbose flag with the func start command to get more detailed output. This might help identify the issue.

19. Rebuilding: If you make any changes to your function code, you need to rebuild your project using mvn clean package before running your function again.

20. Azure Functions Core Tools: Ensure that you have the correct version of Azure Functions Core Tools installed. You can check this by running func --version in your command line. For Java functions, you should have version 3.x or later.
