Yes. Below is the complete README.md content as a single Markdown file. Copy everything inside the code block into a file named README.md.

# DVC (Data Version Control) with Amazon S3

DVC (Data Version Control) is an open-source tool used in Machine Learning and MLOps to version and manage:

- Large datasets
- Machine-learning models
- ML pipelines
- Experiment data

DVC works together with Git to provide version control for both **code and data**.

> **Git is for code and metadata, DVC is for data versioning, and Amazon S3 stores the actual large files.**

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Git vs DVC](#git-vs-dvc)
- [Main Uses of DVC](#main-uses-of-dvc)
- [Project Architecture](#project-architecture)
- [Prerequisites](#prerequisites)
- [Installing DVC](#installing-dvc)
- [Initialize Git](#1-initialize-git)
- [Initialize DVC](#2-initialize-dvc)
- [Add Dataset to DVC](#3-add-dataset-to-dvc)
- [Configure Amazon S3 Remote](#4-configure-amazon-s3-remote)
- [Verify Remote](#5-verify-the-dvc-remote)
- [Configure AWS Credentials](#6-configure-aws-credentials)
- [Commit DVC Metadata to Git](#7-commit-dvc-metadata-to-git)
- [Push Dataset to S3](#8-push-the-dataset-to-s3)
- [Download Dataset](#9-download-the-dataset)
- [Check DVC Status](#10-check-dvc-status)
- [Restore Dataset](#11-restore-dataset)
- [Complete Workflow](#complete-workflow)
- [Important DVC Commands](#important-dvc-commands)
- [Project Structure](#project-structure)
- [Experiment Reproducibility](#experiment-reproducibility)
- [Team Collaboration](#team-collaboration)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Quick Reference](#quick-reference)
- [Conclusion](#conclusion)

---

# Overview

Machine Learning projects depend on both **source code and data**.

Git works very well for source code, configuration files, and documentation. However, storing large datasets and ML models directly in Git is not ideal.

DVC solves this problem by allowing large datasets and ML artifacts to be versioned while storing the actual files in remote storage such as Amazon S3.

The overall concept is:

```text
Git
 │
 │ Tracks code and DVC metadata
 ↓
DVC
 │
 │ Manages dataset versions
 ↓
Amazon S3
 │
 │ Stores actual large files
 ↓
Dataset / Model

Architecture

The architecture used in this project is:

                         GitHub
                            │
             ┌──────────────┼──────────────┐
             │              │              │
        Python Code   Configuration   Documentation
                            │
                       .dvc metadata
                            │
                            ↓
                           DVC
                            │
                            ↓
                    Amazon S3 Bucket
                            │
                            ↓
                  customer_churn.csv


The responsibilities of each component are:

GitHub
 │
 ├── Python source code
 ├── Configuration
 ├── Documentation
 └── DVC metadata
          │
          ↓
         DVC
          │
          ↓
     Amazon S3
          │
          └── Actual dataset

Key Concept
Git → Tracks dataset version and metadata

DVC → Manages the dataset and ML artifacts

S3 → Stores the actual large dataset/model


Git does not store the actual large dataset.

Instead, Git stores a small .dvc metadata file that points DVC to the corresponding data.

Git vs DVC
Git	DVC
Source code	Large datasets
Small files	Large files
Configuration files	ML models
Documentation	Experiment data
Code versions	Dataset versions
Application history	Data history

Git and DVC are complementary tools.

A typical ML project uses both:

Git
 │
 ├── Source code
 ├── Configuration
 ├── Documentation
 └── DVC metadata
          │
          ↓
         DVC
          │
          ↓
     Remote Storage
          │
          ↓
     Actual Dataset

Main Uses of DVC
1. Version Datasets

DVC allows different versions of a dataset to be tracked.

For example:

Dataset Version 1
Dataset Version 2
Dataset Version 3


You can return to an older dataset version when reproducing previous experiments.

2. Store Large Files Outside Git

Large datasets should generally not be stored directly inside Git.

For example:

customer_churn.csv


is tracked by DVC using:

customer_churn.csv.dvc


The .dvc file is small and can be committed to Git.

The actual dataset is stored in the DVC remote.

GitHub
 │
 └── customer_churn.csv.dvc

Amazon S3
 │
 └── Actual customer_churn.csv

3. Use Cloud Storage

DVC supports multiple remote storage systems, including:

Amazon S3
Google Cloud Storage
Azure Blob Storage
SSH storage
MinIO
Other supported DVC remotes

This project uses Amazon S3 as the DVC remote.

4. Reproduce ML Experiments

DVC allows you to associate a specific:

Code Version
     +
Dataset Version
     +
Model Result


This makes ML experiments reproducible.

For example:

Git Commit A
    │
    ├── Code Version A
    └── Dataset Version A
             │
             ↓
        Model Result A


Later:

Git Commit B
    │
    ├── Code Version B
    └── Dataset Version B
             │
             ↓
        Model Result B

5. Share Datasets With a Team

DVC allows a team to use a common remote storage location.

Team members can download the required data using:

dvc pull


This avoids manually transferring large datasets between developers.

6. Track ML Models

DVC can also manage large ML model files.

Examples include:

model.pkl
model.joblib
model.pt
model.h5


The model metadata can be tracked through Git while the actual model can be stored in remote storage.

Project Architecture

This project uses a customer churn dataset:

customer_churn.csv


The architecture is:

                  GitHub
                     │
          ┌──────────┴──────────┐
          │                     │
     Python Code          .dvc Metadata
                                │
                                ↓
                               DVC
                                │
                                ↓
                       Amazon S3 Bucket
                                │
                                ↓
                       customer_churn.csv


The important distinction is:

GitHub
 │
 ├── Python code
 ├── Configuration
 ├── Documentation
 └── customer_churn.csv.dvc
          │
          ↓
         DVC
          │
          ↓
     Amazon S3
          │
          └── Actual dataset

Prerequisites

Before starting, make sure the following are installed or available:

Git
DVC
Homebrew (macOS)
AWS CLI
AWS account
Amazon S3 bucket

You also need AWS credentials with appropriate permissions to access the S3 bucket.

The S3 bucket used in this project is:

dvc-mlops-poc

Installing DVC
macOS

Install DVC using Homebrew:

brew install dvc


Verify the installation:

dvc --version


If DVC is installed correctly, the command will display the installed DVC version.

1. Initialize Git

If the project is not already a Git repository, initialize Git:

git init


Check the repository status:

git status


Git will be responsible for tracking:

Source code
Configuration
Documentation
DVC metadata
2. Initialize DVC

Initialize DVC inside the Git repository:

dvc init


This creates the .dvc directory.

The project will now contain both Git and DVC metadata.

Example:

project/
│
├── .git/
├── .dvc/
├── data/
└── README.md

3. Add Dataset to DVC

Assume the dataset is located at:

data/customer_churn.csv


Add the dataset to DVC:

dvc add data/customer_churn.csv


DVC will create:

data/customer_churn.csv.dvc


The directory will look like:

data/
├── customer_churn.csv
└── customer_churn.csv.dvc


The .dvc file contains metadata that DVC uses to identify and retrieve the dataset.

The actual dataset is not stored in Git.

Understanding the .dvc File

After running:

dvc add data/customer_churn.csv


DVC creates:

data/customer_churn.csv.dvc


Conceptually, the .dvc file contains information such as:

Dataset path
Dataset hash
File information
DVC metadata


The important distinction is:

customer_churn.csv
        │
        └── Actual dataset

customer_churn.csv.dvc
        │
        └── DVC metadata


The .dvc file is small and can be committed to Git.

4. Configure Amazon S3 Remote

Add the S3 bucket as a DVC remote:

dvc remote add -d dvc-poc s3://dvc-mlops-poc

Command Breakdown
dvc      remote      add      -d      dvc-poc       s3://dvc-mlops-poc
 │         │          │       │           │                 │
 │         │          │       │           │                 └── S3 location
 │         │          │       │           └── Remote name
 │         │          │       └── Set as default remote
 │         │          └── Add remote
 │         └── Remote management
 └── DVC command

Parameters
Parameter	Meaning
dvc	DVC command-line tool
remote	Manage remote storage
add	Add a remote
-d	Set the remote as default
dvc-poc	Name of the DVC remote
s3://dvc-mlops-poc	S3 bucket location
5. Verify the DVC Remote

List the configured DVC remotes:

dvc remote list


Expected output:

dvc-poc    s3://dvc-mlops-poc


This confirms that the S3 bucket has been configured as a DVC remote.

Check DVC Configuration

Display the DVC configuration:

cat .dvc/config


You may see:

['remote "dvc-poc"']
    url = s3://dvc-mlops-poc

[core]
    remote = dvc-poc


This configuration means:

dvc-poc is the remote name.
The remote points to the S3 bucket.
dvc-poc is the default remote.
6. Configure AWS Credentials

Before running:

dvc push


AWS credentials must be configured.

If AWS CLI is installed, run:

aws configure


You will be prompted for:

AWS Access Key ID
AWS Secret Access Key
Default region
Default output format


The AWS credentials must have sufficient permissions to access the S3 bucket.

Important: Never commit AWS access keys, secret keys, passwords, or other credentials to GitHub.

7. Commit DVC Metadata to Git

After running:

dvc init


and:

dvc add data/customer_churn.csv


check the Git status:

git status


You may see files such as:

.dvc/
.gitignore
data/customer_churn.csv.dvc


Stage the DVC metadata:

git add .dvc .gitignore data/customer_churn.csv.dvc


Commit the changes:

git commit -m "Configure DVC S3 remote"


Git now tracks the DVC metadata required to identify the dataset version.

8. Push the Dataset to S3

Upload the actual dataset to the configured DVC remote:

dvc push


DVC will upload the required data to the S3-backed DVC remote.

The important distinction is:

GitHub
│
├── .dvc/
│   └── config
│
├── .gitignore
│
└── data/
    └── customer_churn.csv.dvc


Amazon S3
│
└── DVC-managed dataset objects


The actual dataset is stored in S3, not directly in Git.

9. Download the Dataset

When another developer clones the Git repository, the actual dataset may not exist locally.

They can retrieve the dataset using:

dvc pull


The process is:

GitHub
   │
   ├── Source Code
   └── DVC Metadata
          │
          ↓
         DVC
          │
          ↓
      Amazon S3
          │
          ↓
   Local Dataset

10. Check DVC Status

Check whether the local dataset is synchronized with the DVC-tracked version:

dvc status


This can help identify changes between the local data and the tracked DVC state.

11. Restore Dataset

Use:

dvc checkout


to restore DVC-tracked files according to the DVC metadata associated with the current Git state.

For example:

Git Commit A
     │
     ↓
Dataset Version A


Git Commit B
     │
     ↓
Dataset Version B


If you switch Git commits, DVC can restore the corresponding dataset version.

Experiment Reproducibility

One of the main advantages of using Git and DVC together is experiment reproducibility.

Suppose an old experiment used:

Code Version A
Dataset Version A
Model Result A


Later, the project changes:

Code Version B
Dataset Version B
Model Result B


To reproduce the old experiment:

Checkout old Git commit
          │
          ↓
    dvc checkout
          │
          ↓
 Restore old dataset
          │
          ↓
   Run ML pipeline
          │
          ↓
 Reproduce old result


This provides a connection between:

Code
  +
Data
  +
Model

Team Collaboration

DVC is useful when multiple ML engineers or developers work on the same project.

A shared architecture can look like:

                    GitHub
                       │
            ┌──────────┴──────────┐
            │                     │
       Source Code          DVC Metadata
                                  │
                                  ↓
                                 DVC
                                  │
                                  ↓
                            Amazon S3
                                  │
                                  ↓
                           Shared Dataset


A new team member can clone the repository:

git clone <repository-url>


Then download the dataset:

dvc pull


This avoids manually sharing large datasets between team members.

Important DVC Commands
Command	Purpose
dvc init	Initialize DVC
dvc add <file>	Start tracking a dataset or file
dvc remote add	Configure remote storage
dvc remote list	List configured remotes
dvc push	Upload data to remote storage
dvc pull	Download data from remote storage
dvc status	Check data synchronization
dvc checkout	Restore data according to DVC metadata
Complete Workflow

The complete workflow used in this project is:

Initialize Git
      │
      ↓
Initialize DVC
      │
      ↓
Add customer_churn.csv
      │
      ↓
Generate customer_churn.csv.dvc
      │
      ↓
Configure S3 as DVC remote
      │
      ↓
Verify DVC remote
      │
      ↓
Configure AWS credentials
      │
      ↓
Commit DVC metadata to Git
      │
      ↓
Run dvc push
      │
      ↓
Dataset stored in Amazon S3

Data Flow

The complete data flow can be represented as:

                    LOCAL MACHINE
                         │
                         │
                 customer_churn.csv
                         │
                         │ dvc add
                         ↓
                    DVC Metadata
                         │
                         ↓
              customer_churn.csv.dvc
                         │
                         │ git commit
                         ↓
                      GitHub
                         │
                         ↓
                        DVC
                         │
                         │ dvc push
                         ↓
                  Amazon S3 Bucket
                         │
                         ↓
              Actual Dataset Storage

Project Structure

A typical ML project using DVC can have the following structure:

ml-project/
│
├── .dvc/
│   └── config
│
├── data/
│   ├── customer_churn.csv
│   └── customer_churn.csv.dvc
│
├── src/
│   └── ...
│
├── notebooks/
│   └── ...
│
├── models/
│   └── ...
│
├── README.md
├── .gitignore
└── requirements.txt


The exact project structure may vary depending on the application.

Git, DVC, and S3 Responsibilities
Git

Git tracks:

Source code
Configuration
Documentation
DVC metadata
Dataset version references
DVC

DVC manages:

Dataset versions
Large files
ML models
Data dependencies
Data transfer between local and remote storage
Amazon S3

Amazon S3 stores:

Actual datasets
Large ML artifacts
DVC-managed objects
Model files

The relationship is:

Git
 │
 └── Code + Metadata
          │
          ↓
         DVC
          │
          └── Data Version Management
                    │
                    ↓
                   S3
                    │
                    └── Actual Data

Security Considerations

AWS credentials must never be committed to GitHub.

Do not commit files containing:

.env
.aws/credentials
AWS access keys
AWS secret keys
Passwords
API keys


Use secure AWS authentication mechanisms and follow the principle of least privilege when granting permissions.

For production environments, use appropriate IAM roles or other secure AWS credential mechanisms instead of storing long-lived credentials in the project.

Troubleshooting
DVC Command Not Found

If you receive an error indicating that DVC is not installed:

brew install dvc


Then verify:

dvc --version

S3 Permission Error

If:

dvc push


fails with an AWS permission error, check:

AWS credentials
IAM permissions
S3 bucket name
AWS region
DVC remote configuration

Verify the remote:

dvc remote list


Check the configuration:

cat .dvc/config

Dataset Not Available Locally

Run:

dvc pull


Then check:

dvc status


Verify the configured remote:

dvc remote list

Dataset Changes Are Not Reflected

Check the current DVC state:

dvc status


If the dataset was intentionally modified and should represent a new version, update the DVC tracking information and commit the resulting .dvc metadata change to Git.

Quick Reference
Installation
brew install dvc

Git Initialization
git init

DVC Initialization
dvc init

Track Dataset
dvc add data/customer_churn.csv

Configure S3 Remote
dvc remote add -d dvc-poc s3://dvc-mlops-poc

List Remotes
dvc remote list

View Configuration
cat .dvc/config

Check Git Changes
git status

Stage DVC Metadata
git add .dvc .gitignore data/customer_churn.csv.dvc

Commit Metadata
git commit -m "Configure DVC S3 remote"

Push Dataset
dvc push

Download Dataset
dvc pull

Check DVC Status
dvc status

Restore Dataset
dvc checkout

End-to-End Example

The following commands represent the complete setup:

# Install DVC
brew install dvc

# Initialize Git
git init

# Initialize DVC
dvc init

# Add dataset to DVC
dvc add data/customer_churn.csv

# Configure Amazon S3 as DVC remote
dvc remote add -d dvc-poc s3://dvc-mlops-poc

# Verify remote
dvc remote list

# View DVC configuration
cat .dvc/config

# Check Git changes
git status

# Stage DVC metadata
git add .dvc .gitignore data/customer_churn.csv.dvc

# Commit DVC metadata
git commit -m "Configure DVC S3 remote"

# Push dataset to S3
dvc push

Final Architecture
                         ┌─────────────────────┐
                         │       GitHub        │
                         │                     │
                         │ Python Source Code  │
                         │ Configuration       │
                         │ Documentation       │
                         │ DVC Metadata        │
                         └──────────┬──────────┘
                                    │
                                    ↓
                         ┌─────────────────────┐
                         │        DVC          │
                         │                     │
                         │ Dataset Versioning  │
                         │ Data Management     │
                         │ Model Tracking      │
                         └──────────┬──────────┘
                                    │
                                    │ push / pull
                                    ↓
                         ┌─────────────────────┐
                         │     Amazon S3       │
                         │                     │
                         │ DVC Remote Storage  │
                         │                     │
                         │ Dataset             │
                         │ ML Models           │
                         │ Large Artifacts     │
                         └─────────────────────┘

Key Takeaway

The most important concept in this project is:

GitHub
  │
  ├── Python Code
  ├── Configuration
  ├── Documentation
  └── DVC Metadata
          │
          ↓
         DVC
          │
          ↓
     Amazon S3
          │
          └── Actual Dataset / ML Artifacts


In simple terms:

Git   → Tracks code and data metadata
DVC   → Versions and manages data
S3    → Stores the actual large data


This combination provides a practical foundation for:

Dataset versioning
ML model versioning
Experiment reproducibility
Team collaboration
Large file management
Cloud-based data storage
MLOps workflows
Conclusion

DVC extends Git-based version control to datasets and machine-learning artifacts that are too large or unsuitable for direct Git storage.

For this project:

GitHub → Code + Configuration + Documentation + DVC Metadata

DVC → Dataset and ML Artifact Version Management

Amazon S3 → Actual Large File Storage


The overall workflow is:

Git
 ↓
DVC Init
 ↓
DVC Add
 ↓
Configure S3 Remote
 ↓
Git Commit
 ↓
DVC Push
 ↓
Amazon S3


This setup enables reproducible, collaborative, and scalable Machine Learning and MLOps workflows.
