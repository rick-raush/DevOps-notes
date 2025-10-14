**What is Packer?**

**Packer** is an **open-source tool by HashiCorp** that is used to **automate the creation of machine images**.

* * *

### **Key Points**

1.  **Automated Image Building**
    
    - Instead of manually launching an OS, installing software, and configuring it, Packer does it automatically.
        
    - It creates ready-to-use images for cloud providers like AWS, Azure, GCP, VMware, etc.
        
2.  **Consistent Images**
    
    - Packer ensures every image is built the **same way every time**, reducing configuration drift.
3.  **Multi-Platform Support**
    
    - One Packer template can produce images for multiple platforms simultaneously.
        
    - Example: You can build an **AWS AMI** and a **VMware VMDK** from the same configuration.
        
4.  **Integration with Provisioners**
    
    - After launching the base OS, Packer can run **provisioners** to customize the image.
        
    - Examples: Shell scripts, Ansible, Chef, Puppet.
        
5.  **Post-Processing**
    
    - After building an image, Packer can do **extra tasks**, like generating a manifest, compressing images, or uploading them somewhere.

* * *

### **Packer Workflow (Simple Version)**

1.  **Variables** – Define reusable values (like region, instance type).
    
2.  **Builder** – Launches a VM or container and creates an image.
    
3.  **Provisioner** – Configures the image (install software, copy files, run scripts).
    
4.  **Post-Processor** – Optional tasks after image creation (e.g., generate manifest).
    

* * *

### **Example Analogy**

Think of Packer as a **recipe for baking a cake**:

- **Variables** = Ingredients (flour, sugar, eggs)
    
- **Builder** = Oven (where the cake is baked)
    
- **Provisioner** = Steps like mixing, frosting, decorations
    
- **Post-Processor** = Packing the cake in a box, labeling it
    

At the end, you always get **the same perfect cake** every time.

* * *

&nbsp;

&nbsp;

**high-level Packer template structure**, showing only the main sections without all the details:

* * *

### **Packer Template Structure**

```json
{
  "variables": {
    // User-defined variables like VPC IDs, instance type, architecture, etc.
  },

  "builders": [
    {
      // Defines the VM/image builder
      // e.g., amazon-ebs for creating AMIs
    }
  ],

  "provisioners": [
    {
      // Scripts or configuration management tools to customize the image
      // e.g., shell scripts
    },
    {
      // e.g., ansible-local provisioner for applying Ansible playbooks
    }
  ],

  "post-processors": [
    {
      // Optional steps to run after the image is built
      // e.g., manifest, uploading, etc.
    }
  ]
}
```

* * *

✅ **Explanation of Sections**

1.  **variables** – Store reusable values like VPC ID, instance type, architecture. These are referenced in builders or provisioners using `{{user 'variable_name'}}`.
    
2.  **builders** – Define **how and where the base image is created**. Example: `amazon-ebs` for creating an AWS AMI.
    
3.  **provisioners** – **Customize the image** after it's launched. Examples:
    
    - `shell` for running scripts.
        
    - `ansible-local` for running Ansible playbooks inside the instance.
        
4.  **post-processors** – Optional **actions after the image is built**, like:
    
    - Generating a manifest (`manifest.json`).
        
    - Uploading AMIs, compressing, etc.
        

* * *

&nbsp;

&nbsp;

In **HCL (HashiCorp Configuration Language) format**, Packer’s structure is a bit different from the JSON format you saw earlier. Let me break it down carefully.

* * *

### **1️⃣ JSON vs HCL Structure**

| Aspect | JSON Format | HCL Format |
| --- | --- | --- |
| Builders | `"builders": [...]` | `source` + `build` blocks |
| Provisioners | `"provisioners": [...]` | Inside `build` block |
| Variables | `"variables": { ... }` | `variable` blocks |
| Post-Processors | `"post-processors": [...]` | `post-processor` blocks inside `build` |

* * *

### **2️⃣ How HCL organizes a template**

1.  **Variables** – same idea as JSON, but using HCL syntax:

```hcl
variable "vpc_region" {
  type    = string
  default = "ap-south-1"
}
```

2.  **Source block** – defines the builder (like `amazon-ebs`) and all its configuration:

```hcl
source "amazon-ebs" "ubuntu" {
  region               = var.vpc_region
  source_ami_filter {
    filters = {
      name = "al2023-ami-*"
    }
    most_recent = true
    owners      = ["amazon"]
  }
  instance_type        = "t2.micro"
  ssh_username         = "ec2-user"
}
```

3.  **Build block** – references the source and includes provisioners and post-processors:

```hcl
build {
  sources = ["source.amazon-ebs.ubuntu"]

  provisioner "shell" {
    inline = ["sudo yum update -y"]
  }

  provisioner "ansible-local" {
    playbook_file = "setup.yml"
  }

  post-processor "manifest" {
    output = "manifest.json"
  }
}
```

* * *

### **3️⃣ Key Differences in HCL**

- **Source blocks** = JSON `builders`
    
    - You can define multiple sources and reference them in builds.
- **Build blocks** = JSON `builders + provisioners + post-processors`
    
    - You attach **provisioners** and **post-processors** to a build, which points to a source.
- Makes it easier to **reuse sources** for multiple builds.
    

* * *

✅ **Summary:**

- JSON: Everything is in one flat structure (`builders`, `provisioners`, `post-processors`).
    
- HCL: **Separation of concerns**
    
    - `source` → defines the VM/image
        
    - `build` → applies provisioners and post-processors
        

* * *

&nbsp;

### example - hcl2 format packer template

```HCL2
# -------------------------
# VARIABLES
# -------------------------
variable "vpc_region" {
  type    = string
  default = "ap-south-1"
}

variable "vpc_id" {
  type    = string
  default = "vpc-084c9afac4df10d15"
}

variable "vpc_public_sn_id" {
  type    = string
  default = "subnet-0f616169a9b681daa"
}

variable "vpc_public_sg_id" {
  type    = string
  default = "sg-0f74bef84e4f156ba"
}

variable "instance_type" {
  type    = string
  default = "c6g.large"
}

variable "ssh_username" {
  type    = string
  default = "ec2-user"
}

variable "architecture" {
  type    = string
  default = "arm64"
}

variable "Application" {
  type    = string
  default = "base-image"
}

variable "version" {
  type    = string
  default = "1.0"
}

variable "ssh_password" {
  type = string
  default = ""
}

# -------------------------
# SOURCE (Builder)
# -------------------------
source "amazon-ebs" "base_ami" {
  # --- Step 1: Base region & networking ---
  region                      = var.vpc_region
  vpc_id                       = var.vpc_id
  subnet_id                    = var.vpc_public_sn_id
  security_group_id            = var.vpc_public_sg_id
  associate_public_ip_address  = false
  iam_instance_profile         = "paytm-digital-cst-app-arm-nonprod-iam-role"

  # --- Step 2: Source AMI ---
  source_ami_filter {
    filters = {
      "virtualization-type" = "hvm"
      "name"                = "amazon-eks-node-al2023-${var.architecture}-standard-${var.version}*"
      "root-device-type"    = "ebs"
      "architecture"        = var.architecture
    }
    owners      = ["amazon"]
    most_recent = true
  }

  # --- Step 3: Instance details ---
  instance_type = var.instance_type
  ssh_username  = var.ssh_username

  # --- Step 4: Block devices ---
  launch_block_device_mappings {
    device_name           = "/dev/xvda"
    volume_size           = 50
    volume_type           = "gp3"
    throughput            = 125
    iops                  = 3000
    delete_on_termination = true
  }

  # --- Step 5: Temporary EC2 instance tags ---
  run_tags = {
    Name            = "CST-${var.Application}-${var.version}-${var.architecture}-${timestamp()}-Build"
    Application     = var.Application
    environment     = "Prod"
    businessunit    = "digital"
    techteam        = "digitalcst"
    OS_Architecture = var.architecture
    Packer          = "true"
  }

  # --- Step 6: Final AMI details ---
  ami_name        = "CST-${var.Application}-${var.version}-${var.architecture}-${timestamp()}-AMI"
  ami_description = "Amazon EKS base image for arm64 - customized"

  # --- Step 7: Final AMI tags ---
  tags = {
    Name            = "CST-${var.Application}-${var.version}-${var.architecture}-${timestamp()}-AMI"
    Application     = var.Application
    environment     = "Prod"
    businessunit    = "digital"
    techteam        = "digitalcst"
    OS_Architecture = var.architecture
    Source_AMI      = "{{ .SourceAMI }}"
    Packer          = "true"
  }
}

# -------------------------
# BUILD (Provisioners + Post-processors)
# -------------------------
build {
  sources = ["source.amazon-ebs.base_ami"]

  provisioner "shell" {
    execute_command = "echo '${var.ssh_password}' | {{.Vars}} sudo -E -S bash '{{.Path}}'"
    scripts = [
      "../scripts_common/common-packages-al2023.sh",
      "../scripts_common/add-ansible-key.sh",
      "../scripts_common/ansible_setup.sh",
      "../scripts_common/enable-ssm.sh",
      "../scripts_common/enable-cronie.sh",
      "../scripts_common/disable-swap.sh"
    ]
  }

  provisioner "ansible-local" {
    playbook_file = "../../ansible/cst_prod_eks_base_image.yml"
    command       = "sudo /bin/ansible-pull"
    role_paths = [
      "../../ansible/roles/common",
      "../../ansible/roles/common_install",
      "../../ansible/roles/qualyscloud_agent",
      "../../ansible/roles/users-cst",
      "../../ansible/roles/cortex",
      "../../ansible/roles/rsyslog-cst"
    ]
    extra_arguments = [
      "-d /var/lib/ansible/oauth-images",
      "-U git@bitbucket.org:paytmteam/oauth-images.git",
      "-i 127.0.0.1,",
      "--private-key=/root/.ssh/ansible_readonly",
      "--accept-host-key",
      "--checkout master",
      "-e target_host=all"
    ]
  }

  post-processor "manifest" {
    output     = "manifest.json"
    strip_path = true
  }
}

```

### example - json format packer template

```json
{
  "variables": {
    "vpc_region": "ap-south-1",
    "vpc_id": "vpc-084c9afac4df10d15",
    "vpc_public_sn_id": "subnet-0f616169a9b681daa",
    "vpc_public_sg_id": "sg-0f74bef84e4f156ba",
    "instance_type": "c6g.large",
    "ssh_username": "ec2-user",
    "architecture": "arm64",
    "Application": "base-image",
    "version": "1.0",
    "ssh_password": ""
  },

  "builders": [
    {
      "type": "amazon-ebs",

      // --- Step 1: Define base region and network ---
      "region": "{{user `vpc_region`}}",
      "vpc_id": "{{user `vpc_id`}}",
      "subnet_id": "{{user `vpc_public_sn_id`}}",
      "security_group_id": "{{user `vpc_public_sg_id`}}",
      "associate_public_ip_address": false,
      "iam_instance_profile": "paytm-digital-cst-app-arm-nonprod-iam-role",

      // --- Step 2: Choose source AMI ---
      "source_ami_filter": {
        "filters": {
          "virtualization-type": "hvm",
          "name": "amazon-eks-node-al2023-{{user `architecture`}}-standard-{{user `version`}}*",
          "root-device-type": "ebs",
          "architecture": "{{user `architecture`}}"
        },
        "owners": ["amazon"],
        "most_recent": true
      },

      // --- Step 3: Launch instance details ---
      "instance_type": "{{user `instance_type`}}",
      "ssh_username": "{{user `ssh_username`}}",

      // --- Step 4: Configure block devices (root volume) ---
      "launch_block_device_mappings": [
        {
          "device_name": "/dev/xvda",
          "volume_size": 50,
          "volume_type": "gp3",
          "throughput": 125,
          "iops": 3000,
          "delete_on_termination": true
        }
      ],

      // --- Step 5: Tag the temporary EC2 instance ---
      "run_tags": {
        "Name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-Build",
        "Application": "{{user `Application`}}",
        "environment": "Prod",
        "businessunit": "digital",
        "techteam": "digitalcst",
        "OS_Architecture": "{{user `architecture`}}",
        "Packer": "true"
      },

      // --- Step 6: Define the final AMI details ---
      "ami_name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-AMI",
      "ami_description": "Amazon EKS base image for arm64 - customized",

      // --- Step 7: Tag the final AMI ---
      "tags": {
        "Name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-AMI",
        "Application": "{{user `Application`}}",
        "environment": "Prod",
        "businessunit": "digital",
        "techteam": "digitalcst",
        "OS_Architecture": "{{user `architecture`}}",
        "Source AMI": "{{ .SourceAMI }}",
        "Packer": "true"
      }
    }
  ],

  "provisioners": [
    {
      "type": "shell",
      "execute_command": "echo '{{ user `ssh_password` }}' | {{.Vars}} sudo -E -S bash '{{.Path}}'",
      "scripts": [
        "../scripts_common/common-packages-al2023.sh",
        "../scripts_common/add-ansible-key.sh",
        "../scripts_common/ansible_setup.sh",
        "../scripts_common/enable-ssm.sh",
        "../scripts_common/enable-cronie.sh",
        "../scripts_common/disable-swap.sh"
      ]
    },
    {
      "type": "ansible-local",
      "playbook_file": "../../ansible/cst_prod_eks_base_image.yml",
      "command": "sudo /bin/ansible-pull",
      "role_paths": [
        "../../ansible/roles/common",
        "../../ansible/roles/common_install",
        "../../ansible/roles/qualyscloud_agent",
        "../../ansible/roles/users-cst",
        "../../ansible/roles/cortex",
        "../../ansible/roles/rsyslog-cst"
      ],
      "extra_arguments": [
        "-d /var/lib/ansible/oauth-images",
        "-U git@bitbucket.org:paytmteam/oauth-images.git",
        "-i 127.0.0.1,",
        "--private-key=/root/.ssh/ansible_readonly",
        "--accept-host-key",
        "--checkout master",
        "-e target_host=all"
      ]
    }
  ],

  "post-processors": [
    {
      "type": "manifest",
      "output": "manifest.json",
      "strip_path": true
    }
  ]
}

```

&nbsp;

&nbsp;

**Packer template (JSON format)** for building a **custom Amazon Machine Image (AMI)** for **EKS (Elastic Kubernetes Service) worker nodes** on **Amazon Linux 2023**, specifically for **ARM64 architecture**.

Let’s break it down **section by section** 👇

* * *

## 🧩 1️⃣ **variables section**

```json
"variables": {
  "vpc_region": "ap-south-1",
  "vpc_id": "vpc-084c9afac4df10d15",
  "vpc_public_sn_id": "subnet-0f616169a9b681daa",
  "vpc_public_sg_id": "sg-0f74bef84e4f156ba",
  "instance_type": "c6g.medium",
  "ssh_username": "ec2-user",
  "architecture": "arm64",
  "Application": "eks",
  "version": "1.31"
}
```

These are **user variables** that make the template reusable.  
You can override them at build time if needed.  
They define:

- **VPC & Subnet details** where the build instance will be launched.
    
- **Security group** for the builder.
    
- **Instance type** → `c6g.medium` (Graviton / ARM).
    
- **AMI architecture** → ARM64.
    
- **Application/version** for naming tags.
    

* * *

## ⚙️ 2️⃣ **builders section**

This defines **how the AMI is created** — i.e., the base instance Packer spins up and snapshots.

```json
"type": "amazon-ebs"
```

→ Uses the **Amazon EBS builder** (creates an EC2 instance, runs setup, then creates an AMI from it).

### Key properties:

| Field | Purpose |
| --- | --- |
| `launch_block_device_mappings` | Defines root volume — 50GB gp3 EBS, 3000 IOPS, throughput 125 MB/s. |
| `region`, `vpc_id`, `subnet_id` | Network placement for the temporary instance. |
| `iam_instance_profile` | IAM role that allows Packer to run tasks, access S3, etc. |
| `associate_public_ip_address` | False → uses private subnet / no public IP. |
| `security_group_id` | Security group for builder instance. |
| `source_ami_filter` | Chooses the latest Amazon EKS node AMI for AL2023, ARM64, version 1.31. |
| `instance_type`, `ssh_username` | Launch details. |
| `ami_name`, `ami_description` | Name and description for the new AMI. |

* * *

### 🏷️ Tags and Run Tags

Both `run_tags` and `tags` are metadata applied:

- **run_tags** → attached to the temporary EC2 instance during build.
    
- **tags** → attached to the resulting AMI.
    

They store info like:

- OS, architecture
    
- Application name/version
    
- Environment (`Prod`)
    
- Team info (`digitalcst`)
    
- Source AMI used
    

💡 `{{isotime "2006-01-02-15-04"}}` dynamically appends a timestamp.

* * *

## 🧰 3️⃣ **provisioners section**

Provisioners define **what setup/configuration steps** run on the instance before creating the AMI.

There are **two provisioners** here:

* * *

### (a) **Shell Provisioner**

```json
{
  "type": "shell",
  "execute_command": "echo '{{ user `ssh_password` }}' | {{.Vars}} sudo -E -S bash '{{.Path}}'",
  "scripts": [
    "../scripts_common/common-packages-al2023.sh",
    "../scripts_common/add-ansible-key.sh",
    "../scripts_common/ansible_setup.sh",
    "../scripts_common/enable-ssm.sh",
    "../scripts_common/enable-cronie.sh",
    "../scripts_common/disable-swap.sh"
  ]
}
```

Runs shell scripts on the instance in order:

| Script | Purpose |
| --- | --- |
| `common-packages-al2023.sh` | Installs base utilities (curl, jq, git, etc.). |
| `add-ansible-key.sh` | Adds SSH key for Ansible access. |
| `ansible_setup.sh` | Installs and configures Ansible. |
| `enable-ssm.sh` | Enables AWS SSM agent for management. |
| `enable-cronie.sh` | Enables cron service. |
| `disable-swap.sh` | Disables swap (common for Kubernetes nodes). |

* * *

### (b) **ansible-local Provisioner**

```json
{
  "type": "ansible-local",
  "playbook_file": "../../ansible/cst_prod_eks_131_base_image.yml",
  "command": "sudo /bin/ansible-pull",
  "role_paths": [
    "../../ansible/roles/common",
    "../../ansible/roles/common_install",
    "../../ansible/roles/users-cst",
    "../../ansible/roles/rsyslog-cst"
  ],
  "extra_arguments": [
    "-d /var/lib/ansible/oauth-images",
    "-U git@bitbucket.org:paytmteam/oauth-images.git",
    "-i 127.0.0.1,",
    "--private-key=/root/.ssh/ansible_readonly",
    "--accept-host-key",
    "--checkout master",
    "-e target_host=all"
  ]
}
```

This runs Ansible **inside** the temporary instance (via `ansible-pull`):

- Pulls playbook from **Bitbucket repo** (`oauth-images.git`)
    
- Executes the specified playbook (`cst_prod_eks_131_base_image.yml`)
    
- Uses provided roles for common setup, logging, user creation, etc.
    

So basically:

> Shell provisioners prepare the system → then Ansible does detailed app/system configuration.

* * *

## 📦 4️⃣ **post-processors section**

```json
"post-processors": [
  {
    "type": "manifest",
    "output": "manifest.json",
    "strip_path": true
  }
]
```

After the AMI is built, it:

- Generates a `manifest.json` file summarizing AMI details (AMI ID, region, name, etc.)
    
- Useful for CI/CD pipelines (to pass AMI ID to next steps).
    

* * *

## ✅ Overall Flow

1.  **Packer reads variables.**
    
2.  **Launches EC2** in your subnet using **EKS base AMI**.
    
3.  **Runs shell scripts** to prep the environment.
    
4.  **Runs Ansible** for configuration.
    
5.  **Creates new AMI** from that configured instance.
    
6.  **Outputs manifest.json** with AMI info.
    
7.  **Tags everything** with environment/team metadata.
    

* * *

## 💡 Outcome

You get a **custom Amazon Linux 2023-based EKS node AMI** (ARM64, v1.31), preconfigured with:

- Common packages
    
- SSM & Cron
    
- Ansible setup
    
- Logging, users, etc.
    
- Ready for production use under your team’s tagging standards.
    

* * *

&nbsp;

&nbsp;

Let’s now go **deeper into the `builders` section** of your Packer JSON file, since that’s the **core part** that actually defines *how the AMI is created*.

* * *

## 🧱 **What the Builder Does**

In Packer, a **builder** is the component that knows *how* to create a machine image.  
Here you’re using the **`amazon-ebs` builder**, which:

1.  Launches a temporary EC2 instance (based on a source AMI).
    
2.  Runs your provisioners (shell + Ansible).
    
3.  Stops the instance, snapshots the EBS volume.
    
4.  Registers the snapshot as a new AMI.
    
5.  Terminates the temporary instance.
    

&nbsp;

| #   | Stage | What Happens | Related Fields / Sections |
| --- | --- | --- | --- |
| **1** | **Load variables** | Packer reads all `variables` and substitutes `{{user \`var\`}}\`. | `variables` |
| **2** | **Find base AMI** | Uses `source_ami_filter` to locate latest matching Amazon AMI. | `source_ami_filter`, `owners`, `most_recent` |
| **3** | **Launch EC2 instance** | In your VPC/subnet with the IAM role, SG, instance type, etc. | `region`, `vpc_id`, `subnet_id`, `security_group_id`, `iam_instance_profile`, `instance_type`, `ssh_username` |
| **4** | **Attach EBS block device** | Creates and attaches `/dev/xvda` 50 GB gp3 volume. | `launch_block_device_mappings` |
| **5** | **Tag the running instance** | Adds metadata like `run_tags` for identification. | `run_tags` |
| **6** | **Run provisioners** ⚙️ | 🔹 Packer SSHs into the instance (using `ssh_username`).🔹 Executes all scripts in `"provisioners"` sequentially (shell → ansible-local). | `provisioners` section |
| **7** | **Stop instance & create AMI** | After provisioners succeed, Packer stops the instance and snapshots the volume. | `ami_name`, `ami_description` |
| **8** | **Tag the new AMI** | Applies `tags` metadata to the AMI. | `tags` |
| **9** | **Cleanup** | Terminates temporary instance and deletes temporary resources. | internal |
| **10** | **Output manifest** | Writes AMI details (ID, region, name) to `manifest.json`. | `post-processors` |

So let’s go through your builder block **line by line** 👇

* * *

## 🗂️ **Builder Definition**

1.  Define base region and network
    
2.  Choose source AMI
    
3.  Launch instance details
    
4.  Configure block devices (root volume)
    
5.  Tag the temporary EC2 instance
    
6.  Define the final AMI details (once build is done)
    
7.  Tag the final AMI
    

```json
{
  "type": "amazon-ebs",

  // --- Step 1: Define base region and network ---
  "region": "{{user `vpc_region`}}",
  "vpc_id": "{{user `vpc_id`}}",
  "subnet_id": "{{user `vpc_public_sn_id`}}",
  "security_group_id": "{{user `vpc_public_sg_id`}}",
  "associate_public_ip_address": false,
  "iam_instance_profile": "paytm-digital-cst-app-arm-nonprod-iam-role",

  // --- Step 2: Choose source AMI ---
  "source_ami_filter": {
    "filters": {
      "virtualization-type": "hvm",
      "name": "amazon-eks-node-al2023-{{user `architecture`}}-standard-{{user `version`}}*",
      "root-device-type": "ebs",
      "architecture": "{{user `architecture`}}"
    },
    "owners": ["amazon"],
    "most_recent": true
  },

  // --- Step 3: Launch instance details ---
  "instance_type": "{{user `instance_type`}}",
  "ssh_username": "{{user `ssh_username`}}",

  // --- Step 4: Configure block devices (root volume) ---
  "launch_block_device_mappings": [
    {
      "device_name": "/dev/xvda",
      "volume_size": 50,
      "volume_type": "gp3",
      "throughput": 125,
      "iops": 3000,
      "delete_on_termination": true
    }
  ],

  // --- Step 5: Tag the temporary EC2 instance ---
  "run_tags": {
    "Name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-Build",
    "Application": "{{user `Application`}}",
    "environment": "Prod",
    "businessunit": "digital",
    "techteam": "digitalcst",
    "OS_Architecture": "{{user `architecture`}}",
    "Packer": "true"
  },

  // --- Step 6: Define the final AMI details (once build is done) ---
  "ami_name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-AMI",
  "ami_description": "Amazon EKS base image for arm64 - customized",
  
  // --- Step 7: Tag the final AMI ---
  "tags": {
    "Name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-AMI",
    "Application": "{{user `Application`}}",
    "environment": "Prod",
    "businessunit": "digital",
    "techteam": "digitalcst",
    "OS_Architecture": "{{user `architecture`}}",
    "Source AMI": "{{ .SourceAMI }}",
    "Packer": "true"
  }
}

```

* * *

### 🔹 **1\. `"type": "amazon-ebs"`**

This tells Packer to use the **Amazon Elastic Block Store (EBS)** builder —  
the standard method to create AMIs by snapshotting an EBS-backed instance.

* * *

### 🔹 **2\. `launch_block_device_mappings`**

Defines how the root volume (disk) of the instance is set up.

| Key | Meaning |
| --- | --- |
| `device_name` | The root device name (`/dev/xvda` is typical for AL2023). |
| `volume_size` | 50 GB EBS volume. |
| `volume_type` | `gp3` → General Purpose SSD (fast and cheaper than gp2). |
| `throughput` | 125 MB/s — gp3 allows tuning throughput separately. |
| `iops` | 3000 → I/O operations per second. |
| `delete_on_termination` | Ensures the disk is deleted when instance is terminated (so no orphaned volumes). |

So this defines **how large and how performant** your AMI’s root disk will be.

* * *

### 🔹 **3\. Region, VPC, and Networking**

| Field | Purpose |
| --- | --- |
| `region` | Uses `ap-south-1` (Mumbai). |
| `vpc_id`, `subnet_id` | Places the temporary build instance in your VPC and subnet. |
| `security_group_id` | Security group that controls SSH/HTTP/SSM access during the build. |
| `associate_public_ip_address: false` | No public IP — it will use private connectivity or a NAT/SSM session. |

This ensures the instance is launched in your **controlled network**, not in default VPC.

* * *

### 🔹 **4\. IAM Instance Profile**

```json
"iam_instance_profile": "paytm-digital-cst-app-arm-nonprod-iam-role"
```

- This IAM role gives permissions for:
    
    - Accessing S3 or Bitbucket (if needed via secrets)
        
    - Communicating with AWS Systems Manager
        
    - Creating AMIs (Packer needs EC2:CreateImage permissions)
        

* * *

### 🔹 **5\. Source AMI Filter**

```json
"source_ami_filter": {
  "filters": {
    "virtualization-type": "hvm",
    "name": "amazon-eks-node-al2023-{{user `architecture`}}-standard-{{user `version`}}*",
    "root-device-type": "ebs",
    "architecture": "{{user `architecture`}}"
  },
  "owners": ["amazon"],
  "most_recent": true
}
```

This part is **critical** — it chooses the **base AMI** you’ll customize.

- It searches for official **Amazon EKS node images** (`amazon-eks-node-al2023-...`) published by AWS.
    
- `{{user` architecture`}}` → `arm64`
    
- `{{user` version`}}` → `1.31` (EKS version)
    
- `owners`: `"amazon"` ensures only Amazon-official AMIs.
    
- `most_recent: true` ensures it picks the latest version.
    

✅ **So this finds the latest EKS 1.31 AL2023 ARM64 base image as your starting point.**

* * *

### 🔹 **6\. Instance Type and SSH**

| Key | Description |
| --- | --- |
| `instance_type` | `c6g.medium` (Graviton2 ARM-based instance) — must match architecture. |
| `ssh_username` | Default Amazon Linux 2023 user → `ec2-user`. |

* * *

### 🔹 **7\. AMI Name and Description**

```json
"ami_name": "CST-{{user `Application`}}-{{user `version`}}-{{user `architecture`}}-{{isotime \"2006-01-02-15-04\"}}-AMI",
"ami_description": "amazon base AMI for arm64"
```

Packer will name the created AMI something like:

```
CST-eks-1.31-arm64-2025-10-13-12-35-AMI
```

So you can easily identify:

- Team (`CST`)
    
- App (`eks`)
    
- Version (`1.31`)
    
- Arch (`arm64`)
    
- Build timestamp
    

* * *

### 🔹 **8\. Tags and Run Tags**

Two tag sets:

| Tag Type | Applied To | Used For |
| --- | --- | --- |
| `run_tags` | The temporary **EC2 instance** created during build | Helpful for debugging, cost tracking |
| `tags` | The final **AMI image** | For searching, filtering, governance |

Example tags include:

- OS version
    
- Architecture
    
- Application
    
- Environment (`Prod`)
    
- Team and business unit
    
- Source AMI ID (recorded dynamically)
    

* * *

# ⚙️ **Builder Flow (Step-by-Step)**

| #   | Stage | What Happens | Related Fields / Sections |
| --- | --- | --- | --- |
| **1** | **Load variables** | Packer reads all `variables` and substitutes `{{user \`var\`}}\`. | `variables` |
| **2** | **Find base AMI** | Uses `source_ami_filter` to locate latest matching Amazon AMI. | `source_ami_filter`, `owners`, `most_recent` |
| **3** | **Launch EC2 instance** | In your VPC/subnet with the IAM role, SG, instance type, etc. | `region`, `vpc_id`, `subnet_id`, `security_group_id`, `iam_instance_profile`, `instance_type`, `ssh_username` |
| **4** | **Attach EBS block device** | Creates and attaches `/dev/xvda` 50 GB gp3 volume. | `launch_block_device_mappings` |
| **5** | **Tag the running instance** | Adds metadata like `run_tags` for identification. | `run_tags` |
| **6** | **Run provisioners** ⚙️ | 🔹 Packer SSHs into the instance (using `ssh_username`).🔹 Executes all scripts in `"provisioners"` sequentially (shell → ansible-local). | `provisioners` section |
| **7** | **Stop instance & create AMI** | After provisioners succeed, Packer stops the instance and snapshots the volume. | `ami_name`, `ami_description` |
| **8** | **Tag the new AMI** | Applies `tags` metadata to the AMI. | `tags` |
| **9** | **Cleanup** | Terminates temporary instance and deletes temporary resources. | internal |
| **10** | **Output manifest** | Writes AMI details (ID, region, name) to `manifest.json`. | `post-processors` |

1.  **Find base AMI** via `source_ami_filter`.
    
2.  **Launch EC2 instance** (c6g.medium) in `ap-south-1`, using given VPC/subnet/SG/IAM role.
    
3.  **Attach a 50GB gp3 EBS volume** as root disk.
    
4.  **Tag** the instance with metadata.
    
5.  **Run provisioners** (shell scripts + Ansible).
    
6.  **Stop instance → snapshot volume → register new AMI**.
    
7.  **Tag the AMI** with the same metadata.
    
8.  **Terminate the temporary EC2 instance**.
    

* * *

## 🧭 **In Summary**

| Section | Description |
| --- | --- |
| **amazon-ebs** | Tells Packer to build using EC2 + EBS snapshot. |
| **Networking fields** | Control where the instance runs (subnet, SG, etc.). |
| **source_ami_filter** | Chooses your starting base image (EKS node). |
| **Block device mapping** | Defines disk size/type/performance. |
| **IAM role** | Grants AWS permissions for build operations. |
| **Tags** | Help you identify and manage AMIs. |

* * *

&nbsp;

&nbsp;

**Overall Advantages / Differences between formats -** 

| Feature | JSON | HCL2 |
| --- | --- | --- |
| Human readability | Less readable | Cleaner, easier to maintain |
| Modularity | Limited | Sources and builds separated, easier to reuse |
| Variables | Basic | Strong typing, default, validation |
| Provisioners / Post-processors | Fully supported | Fully supported |
| Flexibility | Static structure | Dynamic expressions, loops, conditionals |
| Official recommendation | Older format | Preferred for modern Packer templates |

&nbsp;

&nbsp;

&nbsp;