# Automating Infrastructure on Google Cloud with Terraform

## Challenge scenario
You are a cloud engineer intern for a new startup. For your first project, your new boss has tasked you with creating infrastructure in a quick and efficient manner and generating a mechanism to keep track of it for future reference and changes. You have been directed to use Terraform to complete the project.

For this project, you will use Terraform to create, deploy, and keep track of infrastructure on the startup's preferred provider, Google Cloud. You will also need to import some mismanaged instances into your configuration and fix them.

In this lab, you will use Terraform to import and create multiple VM instances, a VPC network with two subnetworks, and a firewall rule for the VPC to allow connections between the two instances. You will also create a Cloud Storage bucket to host your remote backend.

### Task 1. Create the configuration files
 * In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:

        main.tf
        variables.tf
        modules/
        └── instances
            ├── instances.tf
            ├── outputs.tf
            └── variables.tf
        └── storage
            ├── storage.tf
            ├── outputs.tf
            └── variables.tf

 * Fill out the **variables.tf** files in the root directory and within the modules. Add three variables to each file: **region**, **zone**, and **project_id**. For their default values, use , *filled in at lab start*, and your Google Cloud Project ID.
 * Add the Terraform block and the Google Provider to the main.tf file. Verify the **zone** argument is added along with the **project** and **region** arguments in the Google Provider block.

 * Initialize Terraform.

<details>
<summary>Task 1 Answer</summary>
<br>

* Run the below commands in cloud Shell:

```
touch main.tf
touch variables.tf
mkdir -p modules/instances
touch modules/instances/instances.tf
touch modules/instances/outputs.tf
touch modules/instances/variables.tf
mkdir -p modules/storage
touch modules/storage/storage.tf
touch modules/storage/outputs.tf
touch modules/storage/variables.tf
```

* In Cloudshell, click the Open Editor under the new window.
Add the following to the each variables.tf files which you see under Storage, under Instances, under Modules, and fill in the GCP Project ID, Region, Zone as specified under the lab page:

```
variable "region" {
default = "<Region mentioned under Task 1>"
}
variable "zone" {
default = "<Zone mentioned under Task 1>"
}
variable "project_id" {
default = "<FILL IN PROJECT ID>"
}
```

* Add the following to the main.tf file:
 
```
terraform {
required_providers {
google = {
source = "hashicorp/google"
version = "4.47.0"
}
}
}
provider "google" {
project = var.project_id
region = var.region
zone = var.zone
}
module "instances" {
source = "./modules/instances"
}
 
```

* Run the below commands in cloud Shell in the root directory to initialize terraform.
```
terraform init
```
</details>

### Task 2. Import infrastructure
 * In the Google Cloud Console, on the Navigation menu, click **Compute Engine > VM Instances**. Two instances named **tf-instance-1** and **tf-instance-2** have already been created for you.

Note: by clicking on one of the instances, you can find its Instance ID, boot disk image, and machine type. These are all necessary for writing the configurations correctly and importing them into Terraform.

 * Import the existing instances into the instances module. To do this, you will need to follow these steps:

First, add the module reference into the main.tf file then re-initialize Terraform.
Next, write the resource configurations in the instances.tf file to match the pre-existing instances.

Name your instances tf-instance-1 and tf-instance-2.

For the purposes of this lab, the resource configuration should be as minimal as possible. To accomplish this, you will only need to include the following additional arguments in your configuration: machine_type, boot_disk, network_interface, metadata_startup_script, and allow_stopping_for_update. For the last two arguments, use the following configuration as this will ensure you won't need to recreate it:

    metadata_startup_script = <<-EOT
            #!/bin/bash
        EOT
    allow_stopping_for_update = true
    
Once you have written the resource configurations within the module, use the terraform import command to import them into your instances module.
 * Apply your changes. Note that since you did not fill out all of the arguments in the entire configuration, the apply will **update the instances in-place**. This is fine for lab purposes, but in a production environment, you should make sure to fill out all of the arguments correctly before importing.

<details>
<summary>Task 2 Answer</summary>
<br>

* Navigate to Compute Engine > VM Instances. Click on “tf-instance-1”. Copy the Instance ID  somewhere to be used later.
* Navigate to Compute Engine > VM Instances. Click on “tf-instance-2”. Copy the Instance ID somewhere to be used later.
* Next, navigate to Modules/Instances/instances.tf in Open Editor. Copy the following configuration into the file:
```
resource "google_compute_instance" "tf-instance-1" {
name = "tf-instance-1"
machine_type = "n1-standard-1"
zone = var.zone
metadata_startup_script = <<-EOT
#!/bin/bash
EOT
allow_stopping_for_update = true
boot_disk {
initialize_params {
image = "debian-cloud/debian-10"
}
}
network_interface {
network = "default"
}
}
resource "google_compute_instance" "tf-instance-2" {
name = "tf-instance-2"
machine_type = "n1-standard-1"
zone = var.zone
metadata_startup_script = <<-EOT
#!/bin/bash
EOT
allow_stopping_for_update = true
boot_disk {
initialize_params {
image = "debian-cloud/debian-10"
}
}
network_interface {
network = "default"
}
}
```

* To import the first instance, use the following command in Cloudshell, using the Instance ID for tf-instance-1 you copied down earlier.
```
terraform import module.instances.google_compute_instance.tf-instance-1 <INSTANCE-1-ID>
```

* To import the second instance, use the following command, using the Instance ID for tf-instance-2 you copied down earlier.
```
terraform import module.instances.google_compute_instance.tf-instance-2 <INSTANCE-2-ID>
```

* Now, run the below commands in Cloudshell. Type ‘yes’ when prompted for value
```
terraform plan
terraform apply
```

</details>

### Task 3. Configure a remote backend
 * Create a Cloud Storage bucket resource inside the storage module. For the bucket name, use **Bucket Name**. For the rest of the arguments, you can simply use:

        location = "US"
        force_destroy = true
        uniform_bucket_level_access = true

Note: You can optionally add output values inside of the outputs.tf file.
 * Add the module reference to the main.tf file. Initialize the module and apply the changes to create the bucket using Terraform.

 * Configure this storage bucket as the remote backend inside the main.tf file. Be sure to use the **prefix** terraform/state so it can be graded successfully.

 * If you've written the configuration correctly, upon init, Terraform will ask whether you want to copy the existing state data to the new backend. Type yes at the prompt.

<details>
<summary>Task 3 Answer</summary>
<br>

* In Editor, add the following code to the Modules/Storage/storage.tf file and save:
``` 
resource "google_storage_bucket" "storage-bucket" {
name          = "<YOUR-BUCKET name as in task 3>"
location      = "US"
force_destroy = true
uniform_bucket_level_access = true
}
``` 
* Next, in Editor add the following to the main.tf file and save:
``` 
module "storage" {
source     = "./modules/storage"
}
``` 
* Run the below commands in Cloud shell. Enter ‘yes’ when prompted for a value:
```
terraform init
terraform apply
``` 
* In Editor, in file main.tf, delete the part of the script which is seen in the screenshot below:
![image](https://github.com/user-attachments/assets/b122d8ac-3b5e-40d4-82c1-ef86571b1f01)

* Now add the below script at the starting in main.tf file and save:
```
terraform {
backend "gcs" {
  bucket  = "<YOUR BUCKET name as in task 3>"
prefix  = "terraform/state"
}
required_providers {
  google = {
    source = "hashicorp/google"
    version = "4.53.0"
  }
}
}
```
 
Your main.tf file should resemble as in the screenshot below:
![image](https://github.com/user-attachments/assets/4e604810-613b-48c1-8c62-f80417dba2f5)

* Run the below commands in Cloud Shell. Enter ‘yes’ when prompted.
```
terraform init -upgrade
terraform init
```

</details>

### Task 4. Modify and update infrastructure
 * Navigate to the **instances** module and modify the **tf-instance-1** resource to use an e2-standard-2 machine type.

 * Modify the **tf-instance-2** resource to use an e2-standard-2 machine type.

 * Add a third instance resource and name it **Instance Name**. For this third resource, use an e2-standard-2 machine type. Make sure to change the machine type to e2-standard-2 **to all the three instances**.

 * Initialize Terraform and apply your changes.

<details>
<summary>Task 4 Answer</summary>
<br>  
        
* In Editor, navigate to modules/instances/instances.tf. Modify tf-instance-1 and tf-instance-2 to use machine-type as mentioned in Task 4 instructions.
Refer the screenshot below. Check the changes which have been made in ‘machine_type’ for both the instances:
![image](https://github.com/user-attachments/assets/aff574df-10c9-426a-888f-c2c0e37b024d)

* Add below script for “tf-instance-3” and save:
``` 
resource "google_compute_instance" "<Instance Name as in task 4>" {
name         = "<Instance Name as in Task 4>"
machine_type = "n1-standard-2"
zone         = var.zone
 
 boot_disk {
  initialize_params {
    image = "debian-cloud/debian-10"
  }
}
 
 network_interface {
network = "default"
}
metadata_startup_script = <<-EOT
      #!/bin/bash
  EOT
allow_stopping_for_update = true
}
```
 
* Run the below commands in Cloud Shell. Enter ‘yes’ when prompted for value.
```
terraform init
terraform apply
```
</details>

### Task 5. Destroy resources
Destroy the third instance **Instance Name** by removing the resource from the configuration file. After removing it, initialize terraform and apply the changes.

<details>
<summary>Task 5 Answer</summary>
<br>
        
* Run the below commands in Cloud Shell:
```
terraform taint module.instances.google_compute_instance.<Instance3_name>
```
 
* Next, run the below commands in Cloud Shell and type ‘yes’ when prompted.
```
terraform plan
terraform apply
```
 
* In Editor, navigate to modules/instances/instances.tf and remove script for ‘tf-instance-3’.
* Run the below command in Cloud Shell. Type ‘yes’ when prompted for value.
```
terraform apply
```

</details>

### Task 6. Use a module from the Registry
 * In the Terraform Registry, browse to the Network Module.

 * Add this module to your main.tf file. Use the following configurations:

Use version 6.0.0 (different versions might cause compatibility errors).
Name the VPC **VPC Name**, and use a **global** routing mode.
Specify 2 subnets in the region, and name them subnet-01 and subnet-02. For the subnets arguments, you just need the **Name**, **IP**, and **Region**.
Use the IP 10.10.10.0/24 for subnet-01, and 10.10.20.0/24 for subnet-02.
You do not need any secondary ranges or routes associated with this VPC, so you can omit them from the configuration.
 
 * Once you've written the module configuration, initialize Terraform and run an apply to create the networks.

 * Next, navigate to the instances.tf file and update the configuration resources to connect **tf-instance-1** to subnet-01 and **tf-instance-2** to subnet-02.

Note: Within the instance configuration, you will need to update the network argument to **VPC Name**, and then add the subnetwork argument with the correct subnet for each instance.

<details>
<summary>Task 6 Answer</summary>
<br>

* In the Terraform Registry, add the following into the main.tf and save:
```
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 6.0.0"
 
   project_id   = "<PROJECT_ID mentioned in left pane>"
  network_name = "<VPC_NAME mentioned in left pane>"
  routing_mode = "GLOBAL"
 
   subnets = [
      {
          subnet_name           = "subnet-01"
          subnet_ip             = "10.10.10.0/24"
          subnet_region         = "<enter region from task 6>"
      },
      {
          subnet_name           = "subnet-02"
          subnet_ip             = "10.10.20.0/24"
          subnet_region         = "<enter region from task 6>"
          subnet_private_access = "true"
          subnet_flow_logs      = "true"
          description           = "This subnet has a description"
      },
  ]
}
```

* Run the below commands in Cloud Shell. Type ‘yes’ when prompted.
```
terraform init
terraform apply
```

* In Editor, navigate to the instances.tf file and remove the previous script in it. Add the configuration resources to connect “tf-instance-1” to “subnet-01” and “tf-instance-2” to “subnet-02” and save:
```
resource "google_compute_instance" "tf-instance-1"{
name         = "tf-instance-1"
machine_type = "n1-standard-2"
zone         = “<copy from variables.tf or task1>”
 
 boot_disk {
  initialize_params {
    image = "debian-cloud/debian-10"
  }
}
 
 network_interface {
  network = “<VPC_NAME>”
   subnetwork = "subnet-01"
}
metadata_startup_script = <<-EOT
      #!/bin/bash
  EOT
allow_stopping_for_update = true
}
 
resource "google_compute_instance" "tf-instance-2"{
name         = "tf-instance-2"
machine_type = "n1-standard-2"
zone         = “<copy from variables.tf or task1>”
 
 boot_disk {
  initialize_params {
    image = "debian-cloud/debian-10"
  }
}
 
 network_interface {
  network = "<VPC_NAME>"
   subnetwork = "subnet-02"
}
 
 metadata_startup_script = <<-EOT
      #!/bin/bash
  EOT
allow_stopping_for_update = true
}
 
 
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 6.0.0"
 
   project_id   = "<PROJECT_ID>"
  network_name = "<VPC_NAME>"
  routing_mode = "GLOBAL"
 
   subnets = [
      {
          subnet_name           = "subnet-01"
          subnet_ip             = "10.10.10.0/24"
          subnet_region         = "<as given in task 6>"
      },
      {
          subnet_name           = "subnet-02"
          subnet_ip             = "10.10.20.0/24"
          subnet_region         = "<as given in task 6>"
          subnet_private_access = "true"
          subnet_flow_logs      = "true"
          description           = "This subnet has a description"
      },
  ]
}
```

* Run the below commands in Cloud Shell. Type ‘yes’ when prompted.
```
terraform init
terraform apply
```
Ignore if you see any error as in the screenshot below:
![image](https://github.com/user-attachments/assets/eedaecfc-03b6-44c7-80f5-b9af5a6fde11)

</details>

### Task 7. Configure a firewall
Create a firewall rule resource in the **main.tf** file, and name it **tf-firewall**.
 * This firewall rule should permit the **VPC Name** network to allow ingress connections on all IP ranges (0.0.0.0/0) on **TCP port 80**.
 * Make sure you add the source_ranges argument with the correct IP range (0.0.0.0/0).
 * Initialize Terraform and apply your changes.

Note: To retrieve the required network argument, you can inspect the state and find the ID or self_link of the google_compute_network resource you created. It will be in the form projects/PROJECT_ID/global/networks/**VPC Name**.

<details>
<summary>Task 7 Answer</summary>
<br>

* Add the following resource to the main.tf file and fill in the GCP Project ID and save:

```
resource "google_compute_firewall" "tf-firewall"{
name    = "tf-firewall"
network = "projects/<PROJECT_ID>/global/networks/<VPC_Name>"
 
 allow {
  protocol = "tcp"
  ports    = ["80"]
}
 
 source_tags = ["web"]
source_ranges = ["0.0.0.0/0"]
}
```
 
* Run the below commands in Cloud Shell. Type ‘yes’ when prompted.

```
terraform init

terraform apply
```
 
Ignore if you see any error as in the screenshot below:
![image](https://github.com/user-attachments/assets/c77c1f95-fdcd-4df9-bd27-75ba8378eebf)

</details>

## Sample Answer
Checkout the collapsed section for each task!! :grinning:
