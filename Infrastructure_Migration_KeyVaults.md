# Infrastructure Migration: Key Vault

![image](https://user-images.githubusercontent.com/44923999/211107896-768da90e-2e3b-4124-809e-9773c5cd18bd.png)

Key Vault migration is not a simple topic, nor is there a point-and-click, "Explorer" type tool.

This documentation will detail step-by-step instructions for one of the options.

| Option           | Pros       | Cons     |
| ---------------- | ---------- | -------- |
| **Azure Portal** | - Familiar<br>- Key not exposed | - Manual<br>- One at a Time |

**This list of Pros and Cons is based on my research, experience and perception ONLY.**

Your answer to "why use Option X?" may be as simple as the fact that you favor that option.

## Prepare Infrastructure

In addition to the items listed at the beginning of this documentation, this solution requires the following resources:

* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault/)
  * Instance #1 - "...Main" (on Subscription 1)
  * Instance #2 - "...Branch" (on Subscription 1)
  * Instance #3 - "...Branch" (on Subscription 2)
* [**Secret**](https://learn.microsoft.com/en-us/azure/key-vault/secrets) in "Main" Key Vault

## Exercise 1: Migrate Secret(s) using Azure Portal

### Use Case

* Our "**Main**" Key Vault (on Subscription 1) includes all secrets for our group
* We want to "copy" a Secret from the "**Main**" Key Vault to a "**Branch**" Key Vault (on Subscription 1)
* We also want to "copy" the Secret to Subscription 2

### Proposed Solution

In this exercise, we will "copy" a secret from our source Key Vault to our destination Key Vault in three steps:

* Step 1: Download Backup of Secret123 from "**Main**" Key Vault (on Subscription 1)
* Step 2: Restore Backup of Secret123 to "**Branch**" Key Vault (on Subscription 1)
* Step 3: Move "**Branch**" Key Vault (on Subscription 1) to "**Branch**" Key Vault (on Subscription 2)
* Step 4: Confirm Success

### Step 1: Download Backup

* Open Azure Portal and navigate to the "**...main**" Key Vault
* Navigate to **Secrets** in the **Objects** group of the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/211365071-481196e9-a51c-476f-b724-1677098258a2.png" width="800" title="Snipped: January 9, 2023" />

* Click to select your Secret

  <img src="https://user-images.githubusercontent.com/44923999/211365372-54e7ce60-7142-4272-b0d4-071fb4259cfe.png" width="800" title="Snipped: January 9, 2023" />

* Click "**Download Backup**" and then **Download** on the "**Creating a backup**" pop-up
* Confirm download of the file to the **Downloads** folder on your local device

### Step 2: Restore Backup

* Navigate to the "**...branch**" Key Vault, then **Secrets** in the **Objects** group of the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/211366627-1e5dab97-575a-420b-b459-d6576703e9d6.png" width="800" title="Snipped: January 9, 2023" />

* Click "**Restore Backup**", select the previously downloaded file on the resulting pop-up **Open** dialog box and then click **Open**

### Step 3: Move Key Vault

* Navigate to the "**...branch**" Key Vault

  <img src="https://user-images.githubusercontent.com/44923999/211375255-64611bc4-2d8b-434a-a78d-036895573341.png" width="800" title="Snipped: January 9, 2023" />

* Click **Move** and then "**Move to another subscription**" in the resulting drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/211375540-3b22e566-9817-4fdc-bcb4-8b0b726477c6.png" width="800" title="Snipped: January 9, 2023" />

* Complete the "**Move resources**" >> "**Source + target**" form, and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/211561970-b6f5e7a1-09c6-4b87-8eb3-15cc91fd3124.png" width="800" title="Snipped: January 10, 2023" />

* On the "**Move resources**" >> "**Resources to move**" form, wait for "Checking whether these resources can be moved..." validation and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/211562181-65da535f-2226-4777-b585-b8058d112fcc.png" width="800" title="Snipped: January 10, 2023" />

* Review selections on the "**Move resources**" >> "**Review**" form, check the "I understand..." checkbox and then click **Move**

### Step 4: Confirm Success

* Monitor progress and confirm "Moving resources complete" success message in the **Notifications** pane

  <img src="https://user-images.githubusercontent.com/44923999/211564077-3972ad5e-665b-492b-a922-4d640cb9fc31.png" width="800" title="Snipped: January 10, 2023" />

* Switch to the second subscription and navigate to the new Resource Group to confirm presence of the moved Key Vault

  <img src="https://user-images.githubusercontent.com/44923999/211564463-a10d5afa-1464-4409-929b-e1f568f7a33c.png" width="800" title="Snipped: January 10, 2023" />

* Navigate to **Secrets** in the **Objects** group in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/211565345-1fc462df-133d-4519-96af-ba4faa7af058.png" width="800" title="Snipped: January 10, 2023" />

* Confirm inclusion of appropriate secrets
