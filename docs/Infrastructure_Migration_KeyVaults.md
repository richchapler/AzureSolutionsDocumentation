# Infrastructure Migration: Key Vault

![image](https://user-images.githubusercontent.com/44923999/211107896-768da90e-2e3b-4124-809e-9773c5cd18bd.png)

Key Vault migration is not a simple topic, nor is there a point-and-click, "Explorer" type tool.

This documentation details step-by-step instructions for some of the options.

| Option           | Pros       | Cons     |
| ---------------- | ---------- | -------- |
| **Azure Portal** | - Familiar | - Manual |

**This list of Pros and Cons is based on my research, experience and perception ONLY.**

Your answer to "why use Option X?" may be as simple as the fact that you favor that option.

## Prepare Infrastructure

In addition to the items listed at the beginning of this documentation, this solution requires the following resources:

* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault/) (a source and a target instance in the same Subscription)
* [**Secret**](https://learn.microsoft.com/en-us/azure/key-vault/secrets)

## Exercise 1: Copy Secret using Azure Portal

### Use Case

* Our **Main** "key ring" (Subscription 1, Key Vault 1) includes all secrets for our group
* We want to "copy" Secret X from the **Main** "key ring" to a **Branch** "key ring" (Subscription 2, Key Vault 3)

*Questions for Nathan:*

1) *Why not just give the new user / group access to the secret in the **Main** key ring?*
2) *Why do we want duplicate copies of a given secret?* 

### Proposed Solution

In this exercise, we will "copy" a secret from our source Key Vault to our destination Key Vault in three steps:

* Step 1: Download Secret X from **Main** (Subscription 1, Key Vault 1) and Import to **Branch** (Subscription 1, Key Vault 2)
* Step 2: Move Secret X from **Branch** (Subscription 1, Key Vault 2) to **Copy** (Subscription 2, Key Vault 2)

### Step 1: LOREM IPSUM

* Open Azure Portal
