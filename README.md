Data Version Control (DVC) with Amazon S3
Overview

Data Version Control (DVC) is an open-source tool designed for versioning and managing large datasets, machine-learning models, experiment data, and ML pipelines.

In a typical Machine Learning or MLOps project:

Git is used to version source code, configuration, and documentation.
DVC is used to version and manage large datasets and ML artifacts.
Amazon S3 is used as remote storage for the actual datasets and model files.

The key idea is:

Git → Tracks dataset version and metadata
          ↓
DVC → Manages large data and ML artifacts
          ↓
S3 → Stores the actual dataset/model


This allows us to keep large files out of the Git repository while still maintaining complete version history and reproducibility.

Architecture

The architecture used in this project is:

                         GitHub
                            │
             ┌──────────────┼──────────────┐
             │              │              │
        Python Code     Configuration   Documentation
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

How the components work together
GitHub
  │
  ├── Python code
  ├── Configuration files
  ├── Documentation
  ├── .gitignore
  └── .dvc metadata
          │
          ↓
         DVC
          │
          ↓
    Amazon S3 Remote
          │
          ↓
 Actual dataset/model files


Git does not store the large dataset itself.

Instead, Git stores the small DVC metadata file:

customer_churn.csv.dvc


DVC uses this metadata to identify and retrieve the corresponding dataset from the configured remote storage.

Git vs DVC
Git	DVC
Source code	Large datasets
Small files	Large files
Configuration	ML models
Documentation	Experiment data
Application versioning	Dataset versioning
Code history	Data history

Git and DVC are therefore complementary rather than competing tools.

A Machine Learning project can use both:

Git
 ├── Source code
 ├── Configuration
 ├── Documentation
 └── DVC metadata

DVC
 └── Manages large data/model artifacts

S3
 └── Stores actual large files

Why Use DVC?

Machine Learning projects depend heavily on data.

If the dataset changes but the code remains the same, the model results can also change. Therefore, tracking only source code with Git is not sufficient for reproducible ML experiments.

DVC solves this problem by allowing both code and data versions to be associated with each other.

For example:

Git Commit A
      │
      ├── Code Version 1
      └── Dataset Version 1
              │
              ↓
         Model Result A


Git Commit B
      │
      ├── Code Version 2
      └── Dataset Version 2
              │
              ↓
         Model Result B


This makes it possible to understand exactly which version of the code and data produced a particular model.

Main Uses of DVC
1. Version Datasets

DVC allows different versions of a dataset to be tracked.

For example:

Dataset Version 1
Dataset Version 2
Dataset Version 3


You can switch between versions when reproducing previous experiments or investigating changes in model performance.

2. Store Large Files Outside Git

Large datasets and models should generally not be stored directly inside Git repositories.

Instead, DVC creates a small metadata file.

For example:

customer_churn.csv


is tracked by:

customer_churn.csv.dvc


The actual dataset can be stored in Amazon S3.

Therefore:

GitHub
   │
   └── customer_churn.csv.dvc

Amazon S3
   │
   └── Actual customer_churn.csv data

3. Use Cloud Storage

DVC supports multiple types of remote storage.

Examples include:

Amazon S3
Google Cloud Storage
Azure Blob Storage
SSH storage
MinIO
Other supported DVC remotes

In this project, Amazon S3 is used as the DVC remote.

4. Reproduce ML Experiments

DVC makes it possible to associate:

Code version
      +
Dataset version
      +
Model result


This allows previous experiments to be reproduced more reliably.

5. Share Datasets With a Team

Instead of sending large datasets between team members, developers can use a common DVC remote.

For example:

dvc pull


downloads the required data from the configured remote storage.

6. Track Machine Learning Models

DVC can also be used to manage large ML model files.

For example:

model.pkl
model.joblib
model.pt
model.h5


The model can be tracked using DVC while the actual large file is stored in remote storage.

Project Example

The example in this project uses a customer churn dataset:

customer_churn.csv


The overall architecture is:

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


The Git repository contains the DVC metadata, while the actual dataset is stored in S3.

Prerequisites

Before starting, make sure the following are available:

Git
DVC
Homebrew (for macOS installation)
AWS account
Amazon S3 bucket
AWS credentials with appropriate S3 permissions

For this project, the S3 bucket is:

dvc-mlops-poc


Important: The AWS credentials must have permission to access the S3 bucket before running dvc push or dvc pull.

Installing DVC

On macOS, DVC can be installed using Homebrew.

brew install dvc


Verify the installation:

dvc --version


You should see the installed DVC version.

Step 1: Initialize Git

If the project is not already a Git repository, initialize Git:

git init


Check the repository status:

git status


Git will be responsible for tracking source code, configuration, documentation, and DVC metadata.

Step 2: Initialize DVC

Initialize DVC inside the Git repository:

dvc init


This creates the DVC configuration directory:

.dvc/


The project will now contain both Git and DVC metadata.

A simplified project structure may look like:

project/
│
├── .git/
│
├── .dvc/
│   └── config
│
├── data/
│   └── customer_churn.csv
│
├── README.md
└── ...

Step 3: Add the Dataset to DVC

Suppose the dataset is located at:

data/customer_churn.csv


Run:

dvc add data/customer_churn.csv


DVC will create:

data/customer_churn.csv.dvc


The resulting structure will look like:

data/
├── customer_churn.csv
└── customer_churn.csv.dvc


The .dvc file is a small metadata file that allows DVC to identify and retrieve the corresponding data.

Step 4: Understanding the .dvc File

After running:

dvc add data/customer_churn.csv


DVC generates:

customer_churn.csv.dvc


Conceptually, this file contains information such as:

Dataset path
Dataset hash
File size
DVC metadata


The exact contents are generated and managed by DVC.

The important distinction is:

customer_churn.csv
        │
        └── Actual large dataset

customer_churn.csv.dvc
        │
        └── Small DVC metadata file


The metadata file can safely be committed to Git.

Step 5: Configure Amazon S3 as the DVC Remote

Add the S3 bucket as a DVC remote:

dvc remote add -d dvc-poc s3://dvc-mlops-poc


Let's break down the command:

dvc      remote      add      -d      dvc-poc       s3://dvc-mlops-poc
 │         │          │       │           │                 │
 │         │          │       │           │                 │
 │         │          │       │           │                 └── Remote destination
 │         │          │       │           └── Remote name
 │         │          │       └── Set as default remote
 │         │          └── Add remote
 │         └── Remote management
 └── DVC command


Here:

dvc → DVC command-line tool
remote → Manage DVC remote storage
add → Add a new remote
-d → Set this remote as the default remote
dvc-poc → Name of the remote
s3://dvc-mlops-poc → Amazon S3 bucket
Step 6: Verify the DVC Remote

List configured DVC remotes:

dvc remote list


Expected output:

dvc-poc    s3://dvc-mlops-poc


This confirms that the S3 bucket has been configured as a DVC remote.

Step 7: Inspect DVC Configuration

You can inspect the DVC configuration using:

cat .dvc/config


Depending on the project configuration, you may see something similar to:

['remote "dvc-poc"']
    url = s3://dvc-mlops-poc

[core]
    remote = dvc-poc


This configuration tells DVC:

A remote named dvc-poc exists.
The remote points to the S3 bucket.
dvc-poc is the default remote.
Step 8: Configure AWS Credentials

Before pushing data to S3, AWS credentials must be configured.

The credentials need sufficient permissions to access the configured S3 bucket.

A typical AWS CLI configuration can be created using:

aws configure


You will be prompted for credentials such as:

AWS Access Key ID
AWS Secret Access Key
Default region
Output format


Security: Never commit AWS access keys, secret keys, passwords, or other credentials to GitHub.

For production environments, prefer secure credential mechanisms such as IAM roles, environment-based credentials, or other AWS-supported authentication mechanisms.

Step 9: Commit DVC Metadata to Git

After running dvc add, DVC modifies .gitignore and creates the .dvc metadata file.

Check the changes:

git status


You may see files such as:

.dvc/
.gitignore
data/customer_churn.csv.dvc


Stage the DVC metadata:

git add .dvc .gitignore data/customer_churn.csv.dvc


Commit the changes:

git commit -m "Configure DVC S3 remote"


The Git repository now contains the information required to identify the dataset version.

Step 10: Push the Dataset to Amazon S3

Now upload the actual dataset to the configured DVC remote:

dvc push


DVC will upload the required data to:

Amazon S3
     │
     └── dvc-mlops-poc


The dataset itself does not need to be committed to Git.

Git vs S3 After dvc push

After pushing the data, the architecture becomes:

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
└── dvc-mlops-poc
    │
    └── DVC-managed dataset objects


Conceptually:

Git
 ├── .dvc/config
 ├── .gitignore
 └── customer_churn.csv.dvc
          │
          ↓
         DVC
          │
          ↓
         S3
          │
          └── Actual dataset

Complete DVC Workflow

The complete workflow for this project is:

Initialize Git
      ↓
Initialize DVC
      ↓
Add customer_churn.csv
      ↓
Generate customer_churn.csv.dvc
      ↓
Configure Amazon S3 remote
      ↓
Verify DVC remote
      ↓
Commit DVC metadata to Git
      ↓
Configure AWS credentials
      ↓
Run dvc push
      ↓
Dataset stored in Amazon S3

Important DVC Commands
Command	Purpose
dvc init	Initialize DVC in a Git repository
dvc add file.csv	Start tracking a dataset
dvc remote add	Configure remote storage
dvc remote list	List configured remotes
dvc push	Upload DVC-tracked data to remote storage
dvc pull	Download data from remote storage
dvc status	Check whether data is synchronized
dvc checkout	Restore data according to the current DVC version
dvc push

The command:

dvc push


uploads the DVC-managed data to the configured remote.

In this project:

Local Dataset
     │
     │ dvc push
     ↓
Amazon S3


The Git repository continues to contain only the metadata.

dvc pull

When another developer clones the Git repository, the actual dataset may not be present locally.

After obtaining the repository, they can run:

dvc pull


DVC reads the metadata tracked by Git and retrieves the required dataset from the configured S3 remote.

The workflow is:

GitHub
   │
   ├── Code
   └── .dvc metadata
          │
          ↓
         DVC
          │
          ↓
      Amazon S3
          │
          ↓
   Local dataset

dvc status

To check whether local data is synchronized with the DVC-tracked version:

dvc status


This can help identify whether data has changed or whether the local workspace is different from the tracked state.

dvc checkout

The command:

dvc checkout


restores DVC-tracked files according to the DVC metadata associated with the current project state.

This is particularly useful when switching between Git commits containing different dataset versions.

For example:

Git Commit A
     ↓
Dataset Version A

Git Commit B
     ↓
Dataset Version B


Switching the Git commit and running:

dvc checkout


allows the corresponding data version to be restored.

Reproducing an Old Experiment

One of the major benefits of combining Git and DVC is experiment reproducibility.

Suppose an old experiment used:

Code Version: A
Dataset Version: A
Model Result: A


Later, the project changes:

Code Version: B
Dataset Version: B
Model Result: B


If you need to reproduce the original experiment:

Checkout old Git commit
        ↓
Run dvc checkout
        ↓
Restore old dataset
        ↓
Run ML pipeline
        ↓
Reproduce previous result


This creates a reproducible connection between:

Code
 +
Data
 +
Model

Team Collaboration

DVC is particularly useful when multiple developers or ML engineers work on the same project.

A team can use:

GitHub
   │
   ├── Source code
   ├── DVC metadata
   └── Configuration
          │
          ↓
       DVC Remote
          │
          ↓
     Amazon S3


A new team member can:

git clone <repository>


and then retrieve the required datasets:

dvc pull


This avoids manually sharing large files between team members.

Recommended Project Structure

A typical project can be organized as:

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


The actual organization can vary depending on the project.

End-to-End Command Reference

The following commands summarize the complete activity.

Install DVC
brew install dvc

Initialize Git
git init

Initialize DVC
dvc init

Add the dataset
dvc add data/customer_churn.csv

Configure S3 remote
dvc remote add -d dvc-poc s3://dvc-mlops-poc

Verify the remote
dvc remote list

Inspect configuration
cat .dvc/config

Check Git changes
git status

Stage DVC metadata
git add .dvc .gitignore data/customer_churn.csv.dvc

Commit changes
git commit -m "Configure DVC S3 remote"

Push data to S3
dvc push

Download data
dvc pull

Check DVC status
dvc status

Restore tracked data
dvc checkout

Data Flow

The complete data flow can be summarized as:

                    LOCAL MACHINE
                         │
                         │
                 customer_churn.csv
                         │
                         │ dvc add
                         ↓
                    DVC Metadata
                         │
                         │
                         ↓
                  customer_churn.csv.dvc
                         │
                         │ git commit
                         ↓
                      GitHub
                         │
                         │
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

Key Concept

The most important concept to understand is that Git does not directly store the large dataset.

Instead:

                 GitHub
                    │
                    │
             DVC metadata
                    │
                    ↓
                   DVC
                    │
                    │
                    ↓
              Amazon S3
                    │
                    │
                    ↓
             Actual dataset


Therefore:

Git

Tracks:

Code
Configuration
Documentation
DVC metadata
Dataset version references
DVC

Manages:

Dataset versions
Large files
ML models
Data dependencies
Data transfer between local and remote storage
Amazon S3

Stores:

Actual datasets
Large ML artifacts
DVC-managed objects
Benefits

Using Git + DVC + Amazon S3 provides several benefits:

Version Control

Both code and data can be versioned.

Reproducibility

Previous ML experiments can be recreated using the corresponding code and data versions.

Collaboration

Teams can share datasets through a common remote storage location.

Scalability

Large datasets do not need to be stored directly in Git.

Cloud Storage

Amazon S3 provides centralized remote storage for DVC-managed files.

Separation of Concerns
Git → Code and metadata
DVC → Data version management
S3 → Data storage

Important Security Considerations

AWS credentials should never be committed to Git.

Do not add files containing secrets such as:

.aws/credentials
.env
access keys
secret keys
passwords


to the repository.

Use secure authentication mechanisms and ensure the AWS identity has only the permissions required to access the relevant S3 bucket.

For production environments, follow your organization's AWS security and IAM policies.

Troubleshooting
dvc push fails with an AWS permission error

Check that:

AWS credentials are configured.
The credentials are valid.
The AWS identity has permission to access the S3 bucket.
The bucket name is correct.
The configured AWS region is correct where applicable.

Verify the configured DVC remote:

dvc remote list

Dataset is not downloaded

Run:

dvc pull


If the data still cannot be retrieved, verify the DVC remote:

dvc remote list


and inspect the configuration:

cat .dvc/config

Dataset changes are not reflected

Check DVC status:

dvc status


If the dataset has been intentionally modified and should represent a new version, update its DVC tracking information appropriately and commit the resulting .dvc metadata change to Git.

Final Architecture

The final MLOps-oriented architecture is:

                         ┌─────────────────────┐
                         │       GitHub        │
                         │                     │
                         │  Python source code │
                         │  Configuration      │
                         │  Documentation      │
                         │  DVC metadata       │
                         └──────────┬──────────┘
                                    │
                                    │
                                    ↓
                         ┌─────────────────────┐
                         │        DVC          │
                         │                     │
                         │ Data versioning     │
                         │ Data management     │
                         │ ML artifact tracking│
                         └──────────┬──────────┘
                                    │
                                    │ dvc push / pull
                                    ↓
                         ┌─────────────────────┐
                         │     Amazon S3       │
                         │                     │
                         │  DVC Remote Storage │
                         │                     │
                         │ customer_churn.csv │
                         │ ML models           │
                         │ Other large files   │
                         └─────────────────────┘

Conclusion

DVC extends Git-based version control to the data and ML artifacts that are too large or unsuitable to store directly in Git.

For this project, the responsibilities are clearly separated:

GitHub
  │
  └── Code + Configuration + Documentation + DVC Metadata

DVC
  │
  └── Dataset and ML artifact version management

Amazon S3
  │
  └── Actual large dataset/model storage


The complete workflow is:

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
