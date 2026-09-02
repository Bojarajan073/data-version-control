** DVC (Data Version Control) with Amazon S3 **

DVC (Data Version Control) is an open-source tool used in Machine Learning and MLOps to version and manage:

Large datasets
Machine-learning models
ML pipelines
Experiment data

DVC works together with Git to provide version control for both code and data.

Git is for code and metadata, DVC is for data versioning, and Amazon S3 stores the actual large files.

Table of Contents
Overview
Architecture
Git vs DVC
Main Uses of DVC
Project Architecture
Prerequisites
Installing DVC
Initialize Git
Initialize DVC
Add Dataset to DVC
Configure Amazon S3 Remote
Verify Remote
Configure AWS Credentials
Commit DVC Metadata to Git
Push Dataset to S3
Download Dataset
Check DVC Status
Restore Dataset
Complete Workflow
Important DVC Commands
Project Structure
Experiment Reproducibility
Team Collaboration
Security Considerations
Troubleshooting
Quick Reference
Conclusion
Overview

Machine Learning projects depend on both source code and data.

Git works very well for source code, configuration files, and documentation. However, storing large datasets and ML models directly in Git is not ideal.

DVC solves this problem by allowing large datasets and ML artifacts to be versioned while storing the actual files in remote storage such as Amazon S3.

The overall concept is:

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
     Amazon S3
          │
          └── Actual dataset

Main Uses of DVC
1. Version Datasets

DVC allows different versions of a dataset to be tracked.

Dataset Version 1
Dataset Version 2
Dataset Version 3


You can return to an older dataset version when required for experiment reproduction or debugging.

2. Store Large Files Outside Git

Instead of committing a large dataset directly to Git:

customer_churn.csv


DVC creates a small metadata file:

customer_churn.csv.dvc


The relationship is:

GitHub
 │
 └── customer_churn.csv.dvc
          │
          ↓
         DVC
          │
          ↓
     Amazon S3
          │
          └── Actual customer_churn.csv

3. Use Cloud Storage

DVC supports several remote storage systems:

Amazon S3
Google Cloud Storage
Azure Blob Storage
SSH storage
MinIO
Other supported DVC remotes

This project uses Amazon S3.

4. Reproduce ML Experiments

DVC allows a particular dataset version to be associated with a particular code version and model result.

Code Version
     +
Dataset Version
     +
Model Result


This makes ML experiments more reproducible.

5. Share Datasets With a Team

Team members can download the required dataset from the common DVC remote:

dvc pull

6. Track ML Models

DVC can also manage large ML model files such as:

model.pkl
model.joblib
model.pt
model.h5

Project Architecture

This project uses the following dataset:

customer_churn.csv


The project architecture is:

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

Before starting, make sure the following are available:

Git
DVC
Homebrew for macOS
AWS CLI
AWS account
Amazon S3 bucket

You also need AWS credentials with appropriate permissions to access the S3 bucket.

The S3 bucket used in this project is:

dvc-mlops-poc


Important: Never commit AWS credentials, access keys, or secret keys to GitHub.

Installing DVC

On macOS, DVC can be installed using Homebrew:

brew install dvc


Verify the installation:

dvc --version

1. Initialize Git

If Git has not already been initialized:

git init


Check the repository status:

git status

2. Initialize DVC

Initialize DVC inside the Git repository:

dvc init


This creates the .dvc directory.

The project will now contain both Git and DVC metadata:

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


DVC creates:

data/customer_churn.csv.dvc


The directory now looks like:

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


Conceptually:

customer_churn.csv
        │
        └── Actual large dataset

customer_churn.csv.dvc
        │
        └── DVC metadata


The .dvc file can be committed to Git because it is small.

4. Configure Amazon S3 Remote

Add the S3 bucket as a DVC remote:

dvc remote add -d dvc-poc s3://dvc-mlops-poc

Command Breakdown
dvc      remote      add      -d      dvc-poc       s3://dvc-mlops-poc
 │         │          │       │           │                 │
 │         │          │       │           │                 └── S3 location
 │         │          │       │           └── Remote name
 │         │          │       └── Default remote
 │         │          └── Add remote
 │         └── Remote management
 └── DVC command

Parameter	Description
dvc	DVC command-line tool
remote	Manage remote storage
add	Add a remote
-d	Set as default remote
dvc-poc	Remote name
s3://dvc-mlops-poc	S3 bucket location
5. Verify the DVC Remote

List configured DVC remotes:

dvc remote list


Expected output:

dvc-poc    s3://dvc-mlops-poc

Check DVC Configuration

Display the DVC configuration:

cat .dvc/config


Example:

['remote "dvc-poc"']
    url = s3://dvc-mlops-poc

[core]
    remote = dvc-poc


This configuration indicates:

dvc-poc is the remote name.
The remote points to the S3 bucket.
dvc-poc is the default remote.
6. Configure AWS Credentials

Before running:

dvc push


AWS credentials must be configured.

Using the AWS CLI:

aws configure


You will be prompted for:

AWS Access Key ID
AWS Secret Access Key
Default region
Default output format


The configured AWS identity must have permission to access the S3 bucket.

Security: Never commit AWS access keys or secret keys to GitHub.

7. Commit DVC Metadata to Git

After running:

dvc init


and:

dvc add data/customer_churn.csv


check the Git status:

git status


You may see:

.dvc/
.gitignore
data/customer_churn.csv.dvc


Stage the DVC metadata:

git add .dvc .gitignore data/customer_churn.csv.dvc


Commit the changes:

git commit -m "Configure DVC S3 remote"


At this point, Git tracks the DVC metadata while DVC manages the actual dataset.

8. Push the Dataset to S3

Upload the actual dataset to the configured DVC remote:

dvc push


The data is uploaded to the S3-backed DVC remote.

The resulting architecture is:

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


The actual dataset is stored in S3 rather than directly in Git.

9. Download the Dataset

When another developer clones the Git repository, the actual dataset may not exist locally.

Run:

dvc pull


DVC reads the metadata tracked in Git and downloads the required data from Amazon S3.

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

Check whether the local dataset is synchronized:

dvc status


This helps identify differences between the local workspace and the DVC-tracked state.

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


After switching Git commits, dvc checkout can restore the corresponding dataset version.

Complete Workflow

The complete workflow for this project is:

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

Important DVC Commands
Command	Purpose
dvc init	Initialize DVC
dvc add <file>	Start tracking a dataset/file
dvc remote add	Configure remote storage
dvc remote list	List configured remotes
dvc push	Upload data to remote storage
dvc pull	Download data from remote storage
dvc status	Check data synchronization
dvc checkout	Restore data according to DVC metadata
Project Structure

A typical ML project using DVC can look like:

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

Experiment Reproducibility

One of the biggest advantages of using Git and DVC together is experiment reproducibility.

Suppose an old experiment used:

Code Version A
Dataset Version A
Model Result A


A later experiment may use:

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


This provides a reproducible relationship between:

Code
  +
Data
  +
Model

Team Collaboration

DVC makes it easier for multiple ML engineers and developers to work with the same datasets.

The shared architecture can look like:

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


This avoids manually sharing large datasets between developers.

Security Considerations

AWS credentials should never be committed to GitHub.

Do not commit files containing:

.env
.aws/credentials
AWS access keys
AWS secret keys
Passwords
API keys


Use secure AWS authentication mechanisms and follow the principle of least privilege when granting S3 permissions.

For production environments, use appropriate IAM roles or other secure AWS credential mechanisms.

Troubleshooting
DVC Command Not Found

If DVC is not installed:

brew install dvc


Verify:

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
Install DVC
brew install dvc

Initialize Git
git init

Initialize DVC
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

Check Status
dvc status

Restore Dataset
dvc checkout

End-to-End Command Reference
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

Git → Tracks code and data metadata
DVC → Versions and manages data
S3  → Stores the actual large data

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


This setup provides a practical foundation for:

Dataset versioning
ML model versioning
Experiment reproducibility
Team collaboration
Large file management
Cloud-based data storage
MLOps workflows
