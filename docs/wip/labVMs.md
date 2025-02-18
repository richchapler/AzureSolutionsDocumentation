## **Azure DevTest Labs Instructor VM Setup – End-to-End Guide**

### **1️⃣ Create and Configure Resource Groups & the Base VM in Azure (Not DevTest Labs)**

- Create two resource groups:
  - **`InstructorBase-RG`**: For the **Shared Image Gallery (SIG)** and **base template VMs**.
  - **`InstructorVMs-RG`**: For **instructor-created VMs** deployed in DevTest Labs.
- **Create a Windows 11 Enterprise VM (`aivision`)** in **`InstructorBase-RG`** directly in **Azure (not DevTest Labs)**.
- **Ensure correct configurations**: Networking, storage, security settings.
- **Connect via RDP**, configure system settings, install necessary software, and apply updates.
- **Ensure everything is properly set up before capturing the image.**

📌 **Note:** Before capturing the image, ensure that any lab-specific configurations (e.g., pre-installed software like Visual Studio Code, environment settings, or security policies) are applied. This ensures all future VMs provisioned from the image are ready for use without additional configuration.

------

### **2️⃣ Capture the VM as an Image in a Shared Image Gallery (SIG)**

- **Use “Capture Image”** to create an image version in a **Shared Image Gallery (SIG)** within `InstructorBase-RG`.
- **Select “Specialized”** (since Sysprep was skipped, and configurations should remain intact).
- **Save the image in SIG (`vbdcomputegallery`)**, making it reusable for future deployments.

------

### **3️⃣ Create an Image Definition in SIG**

- **Define an Image Name**: `aivision`.

- Set Metadata Fields

  :

  - **Publisher:** `rchapler`
  - **Offer:** `ai`
  - **SKU:** `win11-24h2-ent`
  - **VM Generation:** `Gen 2`
  - **OS Type:** `Windows`
  - **Security Type:** `Standard`

- **Publish the first version (`1.0.0`)** to make it available for deployment.

------

### **4️⃣ Configure Replication for Faster Deployment**

- **Enable replication in `West US` and `East US`** to optimize VM creation time.
- **Use `1` replica per region** for availability and cost balance.
- **Select “Standard HDD LRS”** (upgrade to SSD for improved performance if needed).

------

### **5️⃣ Create a DevTest Lab in Azure**

1. **Go to the Azure Portal** → Search for **“Azure DevTest Labs”**.
2. Click **"+ Create"**.
3. Enter the following details:
   - **Lab Name:** `InstructorLab`
   - **Subscription & Resource Group:** Use `InstructorVMs-RG`.
   - **Region:** Match your **Shared Image Gallery (SIG) region**.
4. Click **Review + Create** → **Create**.

✅ **This sets up a DevTest Lab for managing instructor VMs.**

------

### **6️⃣ Attach Shared Image Gallery (SIG) to DevTest Labs**

1. **Go to your DevTest Lab (`InstructorLab`).**
2. Click **“Configuration and Policies”** in the left menu.
3. Expand **“Virtual Machine Bases”** → Click **“Shared Image Galleries”**.
4. Click **“Attach”** and select **`vbdcomputegallery`**.
5. Click **OK**.

✅ **Now, DevTest Labs can deploy VMs using the `aivision` image.**

------

### **7️⃣ Create an Instructor VM in DevTest Labs**

1. **Go to “My Virtual Machines”** in DevTest Labs.
2. Click **"+ Create"**.
3. **Choose “Shared Image Gallery”** as the base.
4. **Select `aivision 1.0.0`** as the image source.
5. Configure VM settings:
   - **VM Name**: `Instructor-VM-01`
   - **Size**: `Standard D2s_v3`
   - **Artifacts**: (Optional)
6. **Click “Create”** and wait for the VM to be provisioned.

✅ **This verifies that instructors can now create VMs from the shared image.**

------

### **8️⃣ Make VMs Claimable for Instructors (Optional)**

1. **After creating the VM, go to “My Virtual Machines” in DevTest Labs.**
2. Click on the newly created VM (`Instructor-VM-01`).
3. Click **“Make Claimable”**.
4. Confirm the action.

✅ **Now, instructors only need to claim a VM instead of creating one from scratch.**

------

### **9️⃣ (Optional) Enable Auto-Provisioning**

1. **Go to “Configuration and Policies” in DevTest Labs.**
2. Find **“Auto-Provisioning”** settings.
3. Set a rule like:
   - **Keep at least 3 claimable VMs at all times.**
   - If the number of claimable VMs drops below 3, **Azure automatically creates a new one from the SIG image**.

✅ **This ensures instructors always have a VM ready to claim.**


_NOTE: MAKE SURE THAT NSG >> NETWORK >> ALLOW FOR RDP 3389_



------

## **✅ Final Outcome**

- **A standardized, pre-configured Windows 11 VM (`aivision`) is now available via DevTest Labs**.
- **Instructors can create VMs from this image OR claim pre-created VMs**.
- **Future updates can be handled by publishing new versions (`1.1.0`, `2.0.0`, etc.) in SIG**.
- **Resource groups are now properly segmented for image management (`InstructorBase-RG`) and instructor VM deployments (`InstructorVMs-RG`).**

