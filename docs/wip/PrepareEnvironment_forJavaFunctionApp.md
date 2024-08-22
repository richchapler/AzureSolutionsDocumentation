# Prepare Environment: Java-based Function App Development

These are instructions for setting up an environment in which to develop an Azure Function App based on Java

1. **Install Java Development Kit (JDK)**: Download and install from https://www.oracle.com/java/technologies  

2. **Set "JAVA_HOME" Environment Variable (System)**: "C:\Program Files\Java\jdk-11" (as appropriate for the desired version)

   ![image](https://github.com/user-attachments/assets/d8feaa4e-89e5-4bc3-bdf2-4e923b5fb0f8)
  
     Note: Confirm by running command line:
     ```shell
     "C:\Program Files\Java\jdk-11\bin\java" -version
     ```    

   ![image](https://github.com/user-attachments/assets/eeb64307-69c9-477c-ad36-3d37cf4caf84)
  
3. **Add to "Path" Environment Variable (System)**: "%JAVA_HOME%\bin

   ![image](https://github.com/user-attachments/assets/6d0018d5-89f2-4894-86e4-adf2e93987f8)

     Note: Confirm by running command line:
     ```shell
     where java
     ```

     ![image](https://github.com/user-attachments/assets/cf71e1c6-c02a-4cb6-9dbe-80d05a0af8ae)
   
4. **Install Maven**: Download and install from https://maven.apache.org/download.cgi

5. **Install IntelliJ**: Download and install from https://www.jetbrains.com/idea/download 
   
6. **IntelliJ IDEA, Add JDK**:    

      - Navigate to IntelliJ IDEA > File > Project Structure > Platform Settings > SDKs 
      - Click "+" and on the resulting pop-up, select "Add JDK from disk..."
      - On the "Select Home Directory for JDK" popup, navigate to the JDK installation directory and then click "OK" and "OK" again
      
         ![image](https://github.com/user-attachments/assets/fd6afc39-0a2f-4c3c-86e3-58ad2c411518)

7. **Install IntelliJ IDEA, Azure Toolkit plug-in**  
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
   
11. **Write your function code**: Now, you'll write the code for your Azure Function.  
   - In the `Project` window, navigate to `src/main/java/<YourPackageName>`.  
   - Right-click on the package and select `New -> Azure Function`.  
   - In the new window, enter the function name and choose the trigger type (for example, HttpTrigger). The trigger determines how your function is invoked. An HttpTrigger means your function will run whenever it receives an HTTP request.  
   - Click `OK` to create the function. IntelliJ IDEA will generate a function class with a method that gets called when your function is triggered.  
   
11. **Test your function locally**: Before deploying your function to Azure, you should test it locally to make sure it works correctly.  
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
      azurite --silent --location C:\azurite --debug C:\azurite\debug.log --blobHost  --blobPort 10000 --queuePort 10001 --tablePort 10002  
      ```  
      In this command:    
      - `--silent` runs Azurite in silent mode.    
      - `--location C:\azurite` specifies the directory where Azurite should store its data.    
      - `--debug C:\azurite\debug.log` specifies the file where Azurite should write debug logs.    
      - `--blobHost 127.0.0.1` specifies the IP address that the Blob service should use.    
      - `--blobPort 10000` specifies the port that the Blob service should use.    
      - `--queuePort 10001` specifies the port that the Queue service should use.    
      - `--tablePort 10002` specifies the port that the Table service should use.    
     
   After running this command, each service should start on its own port.

![image](https://github.com/user-attachments/assets/fa1e95d8-f630-48fb-abc1-ccab0ff32e84)
     
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
