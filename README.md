# Code Scanning Terraform Demo (infrastructure as code)

Welcome to the code scanning for Terraform demo! For this demo we'll make use of "TerraGoat", a "Vulnerable by Design" Terraform repository created by [Bridgecrew](https://github.com/bridgecrewio). TerraGoat is a learning and training project that demonstrates how common configuration errors can find their way into production cloud environments, and how you can use GitHub Advanced Security with tfsec to find these vulnerabilities. 

## Introduction

Code scanning is a feature of GitHub Advnaced Security that you use to analyze the code in a GitHub repository to find security vulnerabilities in your software. Any problems identified by the analysis are shown in GitHub.

Code scanning allows you to fully integrate your favorite (open source) scanning tools so all relevant information is shown to the developer the moment a vulnerablity is identified.  

This demo shows you how you can integrate tfsec and/or bridgecrew to scan your terraform definitions for vulnerablities.

## Instructions

<details>
<summary>Fork this repo</summary>
<p> 
  
Begin by [forking this repo](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/fork-a-repo).
</p>
</details>

<details>
<summary>Enable Code Scanning</summary>
<p> 


#### Security tab

Click on the `Security` tab.

<img width="880" alt="Screenshot 2021-05-13 at 15 50 27" src="https://user-images.githubusercontent.com/24505883/118135201-06633a80-b403-11eb-94ca-829ae2e4f200.png">

#### Set up code scanning

Click `Set up code scanning`.

<img src="https://user-images.githubusercontent.com/6920330/96745792-8311c700-1394-11eb-83fd-e47d09bf148e.png" width="70%"/>

#### Setup Workflow

Click the `Setup this workflow` button by tfsec.

<img width="532" alt="Screenshot 2021-05-13 at 15 49 25" src="https://user-images.githubusercontent.com/24505883/118135249-15e28380-b403-11eb-9183-6d094098e9a1.png">


GitHub Code Scanning analyses which programming languages are used in a repository, and provides suggestions for scanning tools based on this information. Other scanning tools that are available in the overview include `Bridgecrew`, `Kubesec`, and others.
See tfsec's documentation for more information about [configuring](https://tfsec.dev/) this tool.
</p>
</details>

<details>
<summary>Actions Workflow file</summary>
<p>

#### Actions Workflow

The Actions Workflow file contains a number of different sections including:
1. Checking out the repository
2. Run tfsec
3. Upload results

<img width="746" alt="Screenshot 2021-05-13 at 15 59 23" src="https://user-images.githubusercontent.com/24505883/118136324-4a0a7400-b404-11eb-93eb-d749a0ad4e36.png">

Click `Start Commit` -> `Commit this file` to commit the changes to _main_ branch.
</p>
</details>

<details>
  
<summary>Workflow triggers</summary>
<p>

#### Workflow triggers

There are a [number of events](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows) that can trigger a GitHub Actions workflow. In this example, the workflow will be triggered on

<img width="640" alt="Screenshot 2021-05-13 at 16 02 17" src="https://user-images.githubusercontent.com/24505883/118136649-a077b280-b404-11eb-9085-a850144418d0.png">

- push to _main_ branch
- pull request to merge to _main_ branch
- on schedule, at 05:21 UTC on Thursdays.

Setting up the new actions workflow and committing it to _main_ branch in the step above will trigger the scan.

</p>
</details>


<details>
<summary>GitHub Actions Progress</summary>

<p>
 
#### GitHub Actions Progress

Click `Actions` tab -> `tfsec`

Click the specific workflow run. You can view the progress of the Workflow run until the analysis completes.

<img width="1274" alt="Screenshot 2021-05-13 at 16 03 44" src="https://user-images.githubusercontent.com/24505883/118136881-e0d73080-b404-11eb-8691-e45661f5a596.png">

</p>
</details>

<details>
<summary>Security Issues</summary>
<p>
  
Once the Workflow has completed, click the `Security` tab -> ` Code Scanning Alerts`. An security alert "EKS cluster should not have open CIDR range for public access" should be visible.

#### Security Alert View

Clicking on the security alert will provide details about the security alert including:
- A description of the issue
- The line of code that triggered the security alert
- A tag to the type of alert (Error, Warning, Note)
- The ability to dismiss the alert depending on certain conditions (false positive? won't fix? used in tests?)
- A link to tfsec's documentation for more information about the rule and mitigations options

<img width="1470" alt="Screenshot 2021-05-13 at 16 05 17" src="https://user-images.githubusercontent.com/24505883/118137368-6c50c180-b405-11eb-8f8f-7d4b5cb56fd6.png">

</details>

<details>
<p>  
  
<summary>Fix the Security Alert</summary>

In order to fix this specific alert, we will need to ensure that the destination file paths is the only location where files can be written to.

Click on the `Code` tab and [Edit](https://docs.github.com/en/free-pro-team@latest/github/managing-files-in-a-repository/editing-files-in-your-repository) the `terraform/aws/eks.tf` file. Navigate to Line 68 of the `eks.tf` file and modify the line:

```tf
  vpc_config {
    endpoint_private_access = true
    subnet_ids              = ["${aws_subnet.eks_subnet1.id}", "${aws_subnet.eks_subnet2.id}"]
  }
```

to

```tf
  vpc_config {
    endpoint_public_access = false
    public_access_cidrs = ["10.2.0.0/8"]
    subnet_ids              = ["${aws_subnet.eks_subnet1.id}", "${aws_subnet.eks_subnet2.id}"]
  }
```

Click `Create a new branch for this commit and start a pull request`, name the branch `eks-endpoint-fix`, and create the Pull Request.

#### Pull Request Status Check

In the Pull Request, you will notice that the tfsec Analysis has started as a status check. Wait until it completes.

<img width="959" alt="Screenshot 2021-05-13 at 17 41 17" src="https://user-images.githubusercontent.com/24505883/118150116-7af1a580-b412-11eb-8f9c-29e939c43777.png">


#### Security Alert Details

After the workflow completes, click on `Details` by the `Code scanning results / tfsec` status check. 

<img width="1007" alt="Screenshot 2021-05-13 at 17 42 12" src="https://user-images.githubusercontent.com/24505883/118150203-952b8380-b412-11eb-8ed6-1ed1c03980e6.png">


#### Fixed Alert

Notice that Code Scanning has detected that this Pull Request will fix two vulnerabilies that were detected before.

<img width="1155" alt="Screenshot 2021-05-13 at 17 43 02" src="https://user-images.githubusercontent.com/24505883/118150334-bb512380-b412-11eb-8ca9-53d27c9b714a.png">

Merge the Pull Request. After the Pull Request has been merged, another Workflow will kick off to scan the repository for any vulnerabilties. 

#### Closed Security Alerts

After the final Workflow has completed, navigate back to the `Security` tab and click `Closed`. Notice that the two alerts now shows up as closed issues.

<img width="1322" alt="Screenshot 2021-05-13 at 17 44 59" src="https://user-images.githubusercontent.com/24505883/118150555-fa7f7480-b412-11eb-8891-ca524994f2e2.png">

#### Traceability

Click on the security alert and notice that it details when the fix was made, by whom, and the specific commit. This provides full traceability to detail when and how a security alert was fixed and exactly what was changed to remediate the issue.

<img width="1015" alt="Screenshot 2021-05-13 at 17 45 56" src="https://user-images.githubusercontent.com/24505883/118150666-16831600-b413-11eb-89ca-69ca248bf1f1.png">


</p>
</details>

## Next Steps

Ready to talk about GitHub Advanced Security and Code Scanning? [Contact Sales](https://enterprise.github.com/contact) for more information!

Check out [GitHub Advanced Security](https://github.com/features/security) for more information about GitHub Advanced Security.

Check out our [documentation](https://docs.github.com/en/code-security/secure-coding/about-code-scanning) for more information and configuration options.



