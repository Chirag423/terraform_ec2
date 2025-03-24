# automating aws using ec2
We are going to try & learn the automation of provisioning of EC2 instances from AWS using Terraform in Ubuntu.

Terraform is an Infrastructure as Code (IaC) tool developed by HashiCorp. It helps us to define, provision, and manage cloud infrastructure using a declarative configuration language called HashiCorp Configuration Language (HCL).
It follows a declaractive approach, meaning we define the desired state of our infrastructure, and Terraform ensures that the actual infrastructure matches that state.
Terraform's workflow follows three major steps : 
- Code: we define infrastructure resources using a .tf file, this step is generally called writing configuration.
- Plan: Terrform compares the inital state and the final state and shows what changes it's going to make 
- Apply: Terraform provisions or updates resources based on the configuration to reach the final state
---

## Installing Terraform
### step 1:
- First we need to make sure that we have gnupg, software-properties-common, and curl packages installed using the below command
  - gnupg: Since gnupg is used for verifying software packages and managing keys, it's needed when adding third-party repositories (like Docker, Node.js, etc.), as they require GPG key verification.
  - software-properties-common: needed for using the add-apt-repository(allows users to easily add Personal Package Archives (PPAs) and other repositories) command to manage software sources.
 ```
 sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
 ```

### Step 2:
- Now we install the Hashicorp GPG key
  - A GPG (GNU Privacy Guard) key is a cryptographic key used to verify the authenticity of a software source. It          ensures that packages from a repository have not been tampered with and come from a trusted source.
  ```
  wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
  ```
  - using `wget` we download the gpg key from hashicorp's website, using `-0-` we ensure it doesn't get saved in a file in current working directory, as we need to store process it into binary before storing it in correct directory for later use
  - we convert the output recieved from wget by passing it to `gpg --dearmor` as input using pipe(|) commnad into binary format required by apt
  - using `tee` we write the input to the specified file location while `> /dev/null` supresses the output to keep the terminal clean

### Step 3:
- Verify the gpg key's fingerprint to ensure it's integrity and authenticity
  - A GPG fingerprint is a unique identifier for a GPG key. It is a 40-character hexadecimal string that helps verify the authenticity of a key.
  ```
  gpg --no-default-keyring \
  --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
  --fingerprint
  ```
  - 1st we we prevent gpg from using default keyring(it is a file that stores public and private GPG keys allowing the system to manage, store, and verify cryptographic keys securely) and then specify the exact keyring file to use and `--fingerprint` displays the fingerprint of the key in the specified keyring

### Step 4: 
- Now we add the official HashiCorp APT repository to our system so that we can install packages like Terraform, Vault, Consul, Packer, and Nomad using apt.
  ```
  echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
  ```
  - `deb` Specifies that this is a Debian-based APT repository.
  - `[signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg]`Ensures that only packages signed with the HashiCorp GPG key are trusted. Uses the key previously added to /usr/share/keyrings/hashicorp-archive-keyring.gpg.
  - `$(lsb_release -cs)`Runs the command lsb_release -cs, which returns the codename of our Ubuntu/Debian release. This ensures that APT downloads the correct package versions for our distribution.

### Step 5: 
- Download the package information from HashiCorp.
  - 1st update APT
  ```
  sudo apt update
  ```
  - then install terraform from the new repository
  ```
  sudo apt-get install terraform
  ```
  - Verify the installation by running `terraform -help`

- Now we enable tab completion for terraform (optional), run `touch ~/.bashrc` to ensure the shell config file exist and the run `terraform -install-autocomplete` to install terrform autocomplete, Once done restart the terminal or run `source ~/.bashrc` 

## Building infrastructure
### Step 1 : Install AWS CLI and Configure Credentials
- download aws cli
  ```
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  ```
- unzip the installer
  ```
  unzip awscliv2.zip
  ```
- install aws cli
  ```
  sudo ./aws/install
  ```
- Configure aws cli authentication
  ```
  aws configure
  ```
  - You will be prompted to enter(you can find these details by clicking on you profile in aws console and visiting the security credential section and clicking on create access key:
     - AWS Access Key ID [None]: YOUR_ACCESS_KEY
     - AWS Secret Access Key [None]: YOUR_SECRET_KEY
     - Default region name [None]: eu-west-2 ( you can look for it at yout AWS Management Console URL)
     - Default output format [None]: json
    ![Screenshot from 2025-03-23 23-26-09](https://github.com/user-attachments/assets/b0d5cb5e-722a-4b7d-99eb-73b078c410ae)


### Step 2 : Create a Terraform Project Directory & Write Terraform Configuration Files
- create and enter terraform project directory
  ```
  mkdir ~/terraform-aws && cd ~/terraform-aws
  ```
- Create a file named main.tf to define your infrastructure using `nano main.tf` enter the following configuration
  ```
  terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
  }
  provider "aws" {
    profile = "default"
    region  = "us-east-1"
  }
  resource "aws_instance" "terraform_test" {
    ami           = "ami-0cbf43fd299e3a464" # Update to latest AMI for your region
    instance_type = "t2.micro"

    tags = {
        Name = "testingterraform"
    }
  }
  ```
- To downloads necessary Terraform provider plugins run `terraform init`
  ![Screenshot from 2025-03-23 23-41-00](https://github.com/user-attachments/assets/ba257721-f389-4ec8-a88b-18434ffa53a7)


- Now run `terraform plan`, which will give an output something like shown below
  ![Screenshot from 2025-03-23 23-55-40](https://github.com/user-attachments/assets/2b310606-04f8-40b2-b0c3-5b71dcc80359)


- To create the infrastructure run `terraform apply`
  ![Screenshot from 2025-03-24 00-27-16](https://github.com/user-attachments/assets/88aa7b2d-c831-4005-b454-5b697f54e95f)

  ![Screenshot from 2025-03-24 00-28-01](https://github.com/user-attachments/assets/13251cdc-eedc-415b-956b-059a341e4d38)

  ![Screenshot from 2025-03-24 00-34-05](https://github.com/user-attachments/assets/f8e1cebb-aa16-4d0d-8e3c-bf1e33ee1859)


- run `terraform destroy` to cleanup everything

  ![Screenshot from 2025-03-24 00-36-44](https://github.com/user-attachments/assets/c39d02a2-f7ae-408f-8cc9-5336657a1cda)

  
