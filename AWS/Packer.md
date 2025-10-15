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

The **repository-based installation** vs the **manual download method**. Let me break it down carefully:

* * *

## **1️⃣ Manual download method (what we did first)**

- You **downloaded a specific version** of Packer as a zip from HashiCorp’s website.
    
- Then you manually **unzipped** it and **moved the binary** to `/usr/local/bin`.
    
- Pros:
    
    - You control the exact version.
        
    - Doesn’t rely on system repositories.
        
- Cons:
    
    - No automatic updates — you have to manually download and update Packer.

* * *

## **2️⃣ Repository-based installation (yum method)**

Steps:

1.  **Install yum-config-manager**

```bash
sudo yum install -y yum-utils
```

- Installs `yum-config-manager`, a tool to manage yum repositories.

2.  **Add the HashiCorp repository**

```bash
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
```

- This adds the official HashiCorp repository to your system so yum can find Packer and other HashiCorp tools.

3.  **Install Packer via yum**

```bash
sudo yum -y install packer
```

- Now yum installs Packer **directly from the repository**.
    
- Pros:
    
    - Automatically resolves dependencies.
        
    - Easier to **update Packer** later using `yum update packer`.
        
- Cons:
    
    - You get the **latest version available in the repo**, not necessarily a specific version.

* * *

### **Key Difference**

| Aspect | Manual Download | Yum Repository |
| --- | --- | --- |
| Version control | Yes (you choose) | Limited to repo version |
| Updates | Manual | Automatic via `yum update` |
| Ease of installation | Slightly more steps | Fewer steps, easier to manage |
| Dependency resolution | Manual | Handled by yum |

* * *

💡 **Rule of thumb:**

- If you want **latest and easy updates**, use the **repo method**.
    
- If you need **a specific version**, use **manual download**.
    

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
# source "<builder-type>" "<source-name>" { ... }

source "amazon-ebs" "base_ami" {
  # --- Step 1: Base region & networking ---
  region                      = var.vpc_region
  vpc_id                       = var.vpc_id
  subnet_id                    = var.vpc_public_sn_id
  security_group_id            = var.vpc_public_sg_id
  associate_public_ip_address  = false ##no public ip is given
  iam_instance_profile         = "paytm-digital-cst-app-arm-nonprod-iam-role"

  # --- Step 2: Source AMI ---
  source_ami_filter {
    filters = {
      "virtualization-type" = "hvm"
      "name"                = "amazon-eks-node-al2023-${var.architecture}-standard-${var.version}*"
      "root-device-type"    = "ebs"
      "architecture"        = var.architecture ##packer build -var "architecture=arm64" .
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

Let’s compare **HCL2 vs JSON** for specifying the **Amazon builder/plugin requirement** in Packer.

* * *

## **1️⃣ HCL2 Format**

```hcl
packer {
  required_plugins {
    amazon = {
      version = ">= 1.2.8"
      source  = "github.com/hashicorp/amazon"
    }
  }
}
```

**Explanation:**

- `packer {}` → Top-level block for Packer configuration.
    
- `required_plugins {}` → Declares the plugins your template needs.
    
- `amazon = { ... }` → Specifies the Amazon EBS builder plugin with version and source.
    

✅ Key points:

- Explicitly tells Packer **which plugin to download**.
    
- Required for HCL2 templates using external builders.
    

* * *

## **2️⃣ JSON Format**

In JSON, you **don’t explicitly declare plugins** like HCL2.

- Packer **infers the plugin from the builder type**.

Example JSON builder:

```json
{
  "builders": [
    {
      "type": "amazon-ebs",
      "region": "ap-south-1",
      "instance_type": "t2.micro",
      "source_ami": "ami-0c55b159cbfafe1f0",
      "ssh_username": "ec2-user",
      "ami_name": "example-ami-{{timestamp}}"
    }
  ]
}
```

**Explanation:**

- `"type": "amazon-ebs"` → Tells Packer to use the **Amazon EBS builder plugin**.
    
- Packer automatically downloads the required plugin from its plugin registry or local cache.
    
- There’s **no explicit `required_plugins` block** in JSON.
    

* * *

### **✅ Key Difference**

| Feature | HCL2 | JSON |
| --- | --- | --- |
| Plugin declaration | Explicit `required_plugins` | Implicit via `"type"` |
| Version control of plugin | Yes (`version = ">= 1.2.8"`) | No  |
| Source of plugin | Explicit (`source = "..."`) | Default registry inferred |
| Mandatory? | For HCL2 external plugins | Not required |

* * *

In short:

- **HCL2** → Explicit, gives control over plugin version and source.
    
- **JSON** → Implicit, simpler, Packer handles plugin automatically.
    

* * *

&nbsp;

&nbsp;

&nbsp;

This `provisioner "shell"` block is a **core part of Packer** where we tell it *what setup commands or scripts to run* inside the temporary EC2 instance **after** it launches but **before** the AMI is created.

Let’s break it down clearly 👇

* * *

## 🧩 **Context**

When Packer builds an AMI, it:

1.  Launches a temporary EC2 instance.
    
2.  SSHs into it.
    
3.  Runs whatever scripts you define under **provisioners**.
    
4.  Creates the final AMI from that configured instance.
    

* * *

## ⚙️ **Your snippet**

```hcl
provisioner "shell" {
  execute_command = "echo '${var.ssh_password}' | {{.Vars}} sudo -E -S bash '{{.Path}}'"
  scripts = [
    "../scripts_common/common-packages-al2023.sh",
    "../scripts_common/add-ansible-key.sh",
    "../scripts_common/ansible_setup.sh"
  ]
}
```

* * *

## 🔍 **Step-by-step explanation**

### 1\. `provisioner "shell"`

This tells Packer:

> “Run shell commands (or scripts) inside the instance.”

Packer supports different provisioner types (like `ansible`, `chef`, `file`, etc.), and this is the simplest — just a shell script runner.

* * *

### 2\. `execute_command`

```bash
echo '${var.ssh_password}' | {{.Vars}} sudo -E -S bash '{{.Path}}'
```

This defines **how Packer executes** the scripts inside the instance.

Let’s decode it:

| Part | Meaning |
| --- | --- |
| `echo '${var.ssh_password}'` | Sends the SSH password (if defined) into the command input. Used only if password authentication is needed — often optional when SSH key is used. |
| `{{.Vars}}` | A Packer template variable — injects environment variables like `PACKER_BUILD_NAME`, `PACKER_BUILDER_TYPE`, etc., into the command environment. |
| `sudo -E -S` | Runs the command as `root` using `sudo`. • `-E` keeps the environment variables. • `-S` reads the password from stdin (that’s why we echo it). |
| `bash '{{.Path}}'` | Runs the shell script file Packer uploaded. `{{.Path}}` is replaced by the actual script path inside the instance. |

✅ So effectively, it means:

> “Run the uploaded script as root using sudo, preserving environment variables.”

* * *

### 3\. `scripts = [...]`

This is a **list of shell scripts** that Packer will run in order.

Each path:

```bash
../scripts_common/common-packages-al2023.sh
../scripts_common/add-ansible-key.sh
../scripts_common/ansible_setup.sh
```

is a local file on your machine that Packer copies to the instance, then runs via the command defined in `execute_command`.

So:

- First script installs common packages
    
- Second adds an Ansible SSH key
    
- Third installs/configures Ansible
    

* * *

## 💡 **If you don’t define `execute_command`**

Packer will use a **default command** (typically `sudo -E bash '{{.Path}}'`), which works fine if:

- You’re using SSH keys (not passwords)
    
- Your default SSH user (like `ec2-user`) has passwordless `sudo`
    

So `execute_command` is optional unless you need custom sudo handling or environment setup.

* * *

### ✅ **Summary Table**

| Field | Purpose |
| --- | --- |
| `provisioner "shell"` | Runs shell scripts on the instance |
| `execute_command` | Custom command for how to execute those scripts (with sudo, env, password) |
| `scripts` | Ordered list of local shell scripts to run during provisioning |

* * *

&nbsp;

&nbsp;

&nbsp;

Packer to use **Ansible** to configure the instance.

Let’s unpack this line by line 👇

* * *

## 🧩 **What `provisioner "ansible-local"` means**

Packer supports several types of **provisioners** — one of them is `ansible-local`.

This type tells Packer:

> “Install Ansible **inside** the temporary instance, then run the given playbook **locally** there.”

So instead of running Ansible from your own workstation (via SSH like `ansible-playbook`), Packer:

1.  Uploads your playbooks and roles to the instance.
    
2.  Installs Ansible on that instance.
    
3.  Executes the playbook *on itself*.
    

That’s why it’s called **ansible-local**.

* * *

## ⚙️ **Snippet**

```hcl
provisioner "ansible-local" {
  playbook_file = "../../ansible/cst_prod_eks_base_image.yml"
  command       = "sudo /bin/ansible-pull"
}
```

Let’s go line by line.

* * *

### 🧠 1. `provisioner "ansible-local"`

- Tells Packer to use **Ansible** as the configuration engine.
    
- It will install Ansible (if not already present) and then run your playbook.
    

* * *

### 📄 2. `playbook_file = "../../ansible/cst_prod_eks_base_image.yml"`

- Specifies **which Ansible playbook** to run inside the instance.
    
- The path is **relative to your Packer template** (where you run `packer build`).
    
- Packer copies that playbook into the instance’s temporary working directory, usually `/tmp/packer-provisioner-ansible-local/`.
    

This playbook usually:

- Installs required packages,
    
- Configures services,
    
- Sets permissions, users, etc.
    

* * *

### 🧰 3. `command = "sudo /bin/ansible-pull"`

By default, Packer’s `ansible-local` provisioner runs:

```bash
ansible-playbook <playbook_file>
```

But here, you’re **overriding** that command and telling it to use:

```bash
sudo /bin/ansible-pull
```

Let’s break that down:

| Part | Meaning |
| --- | --- |
| `sudo` | Run as root (since system-level setup usually needs root privileges). |
| `/bin/ansible-pull` | Runs Ansible in **pull mode** instead of push mode. |

* * *

### 💡 What is **Ansible Pull Mode**?

Normally, Ansible uses a **push** model — the control node connects to remote hosts and pushes configuration.

In **pull mode**, the remote host (your EC2 instance) itself *pulls* playbooks from a Git repository and applies them locally.

In your case, the playbook might include a line like:

```yaml
- hosts: localhost
```

So Ansible runs everything *inside the instance*.

You often combine this with parameters like:

```hcl
extra_arguments = [
  "-U git@bitbucket.org:paytmteam/oauth-images.git",
  "--checkout master",
  "--accept-host-key",
  "-i 127.0.0.1,"
]
```

to make `ansible-pull` download the latest version of your repo before running.

* * *

## 🧠 **Putting it all together**

So when Packer reaches this provisioner step:

1.  It copies your Ansible playbook (and optionally roles) into the instance.
    
2.  Installs Ansible if needed.
    
3.  Runs:
    
    ```bash
    sudo /bin/ansible-pull -U git@bitbucket.org:paytmteam/oauth-images.git \
    -d /var/lib/ansible/oauth-images \
    -i 127.0.0.1, \
    -e target_host=all \
    --checkout master
    ```
    
    (depending on your extra args)
    
4.  Ansible configures the instance as per your playbook.
    
5.  Once done, Packer stops the instance and creates the AMI.
    

* * *

### ✅ **Summary**

| Field | Meaning |
| --- | --- |
| `provisioner "ansible-local"` | Run Ansible **inside** the instance |
| `playbook_file` | Path to the main Ansible playbook |
| `command` | Overrides default Ansible command; here using `ansible-pull` for self-provisioning |
| Typical usage | Install packages, configure services, or harden the image before AMI creation |

* * *

- **ansible** → Runs playbook remotely from your machine; requires Ansible installed locally and network access to the instance.
    
- **ansible-local** → Runs playbook inside the instance; Packer uploads the playbook; no local Ansible needed.
    
- **ansible-pull** → Runs playbook inside the instance; instance pulls it from a Git repo; good for keeping AMIs automatically updated.
    

| Feature | ansible | ansible-local | ansible-pull |
| --- | --- | --- | --- |
| Where playbook runs | Remote | Local | Local |
| Playbook source | Local copy | Uploaded copy | Git repo |
| Requires Ansible on host | Yes | No  | No (installed inside instance) |
| Network dependency | Yes (control → instance) | No  | Yes (instance → Git repo) |
| Best use case | Quick ad-hoc, CI builds | Standard Packer builds | Auto-update AMIs from Git |

&nbsp;

&nbsp;

&nbsp;

The **execution stage** of your Packer workflow.  
Let’s go step by step so you understand **what each command does** and **how to confirm the AMI is created** in AWS.

* * *

## 🧩 1️⃣ Prerequisites checklist before running

Make sure the following are ready:

✅ **Packer installed** → `packer version`  
✅ **AWS credentials configured** → `aws sts get-caller-identity`

- (should return your AWS account info — that’s how Packer authenticates)  
    ✅ **Your `.pkr.hcl` file** (e.g. `ubuntu_base.pkr.hcl`) is correct  
    ✅ **SSH key** specified in your builder actually exists in AWS  
    ✅ **Ansible roles/playbook path** is valid (if using provisioners)

&nbsp;

The **best practice** for EC2 instances running Packer, because you don’t need to manage static credentials at all. Let’s go step by step.

* * *

## **Step 1: Create an IAM Role for EC2**

1.  Go to the **AWS Management Console → IAM → Roles → Create Role**.
    
2.  Select **EC2** as the trusted entity (since your EC2 instance will assume this role).
    
3.  Attach a policy with **permissions Packer needs**. There are a few ways:
    

### **Option A: Use managed policies**

- `AmazonEC2FullAccess` → Allows creating instances, AMIs, snapshots, etc.
    
- `AmazonSSMManagedInstanceCore` → Optional, if you use SSM for provisioning.
    

### **Option B: Use a custom policy** (more secure)

You can restrict permissions to only what you need for your Packer build. Example JSON:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:DescribeInstances",
                "ec2:CreateImage",
                "ec2:TerminateInstances",
                "ec2:DescribeImages",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcs",
                "ec2:DescribeSecurityGroups",
                "ec2:CreateTags",
                "ec2:DeleteTags",
                "ec2:DescribeKeyPairs",
                "ec2:DescribeSnapshots"
            ],
            "Resource": "*"
        }
    ]
}
```

> You can further restrict `Resource` to only your VPC, subnets, and instances if you want maximum security.

* * *

## **Step 2: Attach the Role to Your EC2 Instance**

1.  Go to **EC2 → Instances → Select your instance → Actions → Security → Modify IAM Role**.
    
2.  Choose the role you just created.
    
3.  Save changes.
    

> After this, your EC2 instance automatically gets temporary credentials via the instance metadata service. No keys are stored on disk.

* * *

## **Step 3: Verify IAM Role is working**

On the EC2 instance:

```bash
aws sts get-caller-identity
```

- You should see an ARN like:  
    `arn:aws:sts::123456789012:assumed-role/MyPackerRole/i-0abcd1234efgh5678`
    
- If you see this, Packer and AWS CLI will now work without `aws configure`.
    

* * *

## **Step 4: Run Packer**

Now you can proceed normally:

```bash
packer init .
packer validate .
packer build my-pkr-file.pkr.hcl
```

- Packer will use the **IAM role’s temporary credentials** automatically.
    
- You don’t need any access keys.
    

* * *

💡 **Tip:**  
If your Packer template includes creating AMIs in a different region, make sure the role has permissions across those regions, or you’ll get `UnauthorizedOperation` errors.

* * *

&nbsp;

Ah, this is a **common point of confusion**, so let’s clarify carefully.

* * *

## **1️⃣ IAM Role**

- **Definition:** A set of **permissions** (policy) that defines what actions are allowed (e.g., `ec2:RunInstances`, `s3:GetObject`).
    
- **Purpose:** It’s an **identity** in AWS that can be assumed by users, services, or EC2 instances.
    
- **Example:** `packer-temp-ec2-role` with permission to create AMIs, launch EC2 instances, etc.
    

* * *

## **2️⃣ Instance Profile**

- **Definition:** A **container that allows EC2 to assume a role**.
    
- You **cannot attach a role directly** to an EC2 instance — you attach an **instance profile**, which references a role.
    
- Think of it as a “wrapper” around the IAM role so EC2 can use it.
    

> Every instance profile contains exactly **one IAM role**.

* * *

### **How they relate**

| Concept | Description |
| --- | --- |
| IAM Role | The permissions themselves (what can be done). |
| Instance Profile | A “wrapper” that EC2 uses to assume the role. EC2 instances need this. |
| Use case in EC2 | EC2 cannot directly use an IAM role; it uses an **instance profile** that has a role. |

* * *

### **Example Flow**

1.  Create IAM Role → `packer-temp-ec2-role` with EC2 permissions.
    
2.  Create Instance Profile → `packer-temp-ec2-profile` and attach `packer-temp-ec2-role` to it.
    
3.  Launch EC2 → assign `packer-temp-ec2-profile` as the instance profile.
    
4.  EC2 assumes the role inside the instance → temporary credentials available via metadata.
    

* * *

### **In Packer Template**

```hcl
source "amazon-ebs" "example" {
    iam_instance_profile = "packer-temp-ec2-profile"
}
```

- Here **`iam_instance_profile`** = the instance profile attached to EC2.
    
- Inside AWS, that profile references a role which has the actual permissions.
    

* * *

💡 **Key Point:**  
If you only have a free-tier test EC2 and don’t want to create roles yet, you can **skip the `iam_instance_profile` line**. Packer will still work if your EC2 control plane has credentials via AWS keys or default IAM role.

* * *

&nbsp;

&nbsp;

* * *

## ⚙️ 2️⃣ Initialize the Packer template

Run this **inside the folder** where your `.pkr.hcl` file is located:

```bash
packer init .
```

🧠 Meaning:  
This downloads and sets up the **required plugin** (like the `amazon` plugin you defined in your `packer` block).  
It’s similar to `terraform init`.

* * *

## 🧩 3️⃣ Validate the configuration

Before running the build, always check syntax:

```bash
packer validate .
```

✅ If successful, you’ll see:  
`The configuration is valid.`  
❌ If not, Packer will show exactly where the error is (syntax or variable missing, etc.)

* * *

## 🚀 4️⃣ Build the AMI

Now the actual build:

```bash
packer build <yourfile>.pkr.hcl
```

Example:

```bash
packer build ubuntu_base.pkr.hcl
```

🧠 What happens internally:

1.  Packer spins up a **temporary EC2 instance** using your source AMI.
    
2.  It connects via SSH and runs your **provisioners** (e.g. shell or ansible).
    
3.  When done, it **creates a new AMI** (based on that configured instance).
    
4.  Then it **terminates** the temporary EC2 instance.
    

* * *

## 🧾 5️⃣ Output you’ll see

If everything goes well, near the end you’ll get a line like:

```
amazon-ebs.base_ami: AMI: ami-0a12b34cd56ef7890 created
```

💡 **Copy this AMI ID** — that’s your new custom AMI.

* * *

## 🧭 6️⃣ Verify the AMI in AWS

### Option 1: AWS CLI

```bash
aws ec2 describe-images --image-ids ami-0a12b34cd56ef7890 --query 'Images[*].{Name:Name,ID:ImageId,State:State}' --output table
```

You should see:

```
---------------------------------
|       DescribeImages          |
+----------------+--------------+
| ID             | ami-0a12b34  |
| Name           | my-new-ami   |
| State          | available    |
+----------------+--------------+
```

### Option 2: AWS Console

- Go to **EC2 → AMIs** (on left sidebar)
    
- Choose **Owned by me**
    
- You’ll see your AMI listed there with the name from your `ami_name` field in the packer file.
    

* * *

## 🧹 7️⃣ Clean up (optional)

If you want to delete the AMI later:

```bash
aws ec2 deregister-image --image-id ami-0a12b34cd56ef7890
```

And also delete associated snapshots (if any).

* * *

&nbsp;

Excellent — this is a **common confusion when moving from Terraform syntax** to Packer HCL2 👍

Let’s go through this carefully.

* * *

## 🧩 Root Cause

You wrote:

```hcl
${timestamp() | clean_resource_name}
```

That syntax (`|`) is **Terraform’s template filter** style, but **Packer HCL2 does not support the `|` operator** in interpolation.  
So Packer is reading `|` as a **bitwise OR** — hence the error.

* * *

## ✅ Correct Fix (for Packer HCL2)

You should use **the `clean_resource_name()` function**, not the `|` pipe filter.

So instead of:

```hcl
${timestamp() | clean_resource_name}
```

Use:

```hcl
${clean_resource_name(timestamp())}
```

* * *

## 🛠 Fixed Example for You

Here’s how to fix **all three** places in your file:

```hcl
run_tags = {
  Name        = "CST-${var.Application}-${var.version}-${var.architecture}-${clean_resource_name(timestamp())}-Build"
  Application = var.Application
  environment = "Prod"
  Packer      = "true"
}

ami_name        = "CST-${var.Application}-${var.version}-${var.architecture}-${clean_resource_name(timestamp())}-AMI"
ami_description = "Amazon EKS base image for amd - customized"

tags = {
  Name        = "CST-${var.Application}-${var.version}-${var.architecture}-${clean_resource_name(timestamp())}-AMI"
  Application = var.Application
  environment = "Prod"
  Source_AMI  = "{{ .SourceAMI }}"
  Packer      = "true"
}
```

* * *

## ✅ Validate Again

Now re-run:

```bash
packer validate .
```

It should **pass cleanly**.  
If it still errors, try the `formatdate()` version (it’s simpler and 100% safe):

```hcl
${formatdate("YYYYMMDDHHmmss", timestamp())}
```

Example:

```hcl
ami_name = "CST-${var.Application}-${var.version}-${var.architecture}-${formatdate("YYYYMMDDHHmmss", timestamp())}-AMI"
```

* * *

&nbsp;

&nbsp;