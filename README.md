# GitHub Action Configuration

This repository includes a GitHub Action that allows you to automate tasks in your workflow. Follow these steps to set up and use this action in your own repository.

## Setup Steps

### 1. Clone this Repository

First, clone this repository to your GitHub account.

```shell
git clone https://github.com/your-username/your-repository.git
```

### 2. Create a Workflow File
In your repository, create a directory named .github/workflows if it doesn't already exist. Then, put the YAML file within this directory to define your workflow. You can name this file whatever you like.

### 3. Understanding GitHub Action Steps

```
      - name: Instalar jq
        run: |
          sudo apt-get update
          sudo apt-get install jq
```

This GitHub Action step is named "Install jq" and it consists of a run block, which contains the following shell commands:

sudo apt-get update: This command updates the package list on the system to ensure it has the latest information about available packages.
sudo apt-get install jq: This command installs the jq tool on the system.
The purpose of this GitHub Action step is to install the jq tool, which is a lightweight and flexible command-line JSON processor, on the system where the action is being executed. This step ensures that jq is available and can be used in subsequent actions or scripts within the workflow.

```
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # important!
```

This GitHub Action step uses the actions/checkout action with version 4. It checks out the source code repository for your workflow. The with block specifies the configuration options for the action. In this case, it sets fetch-depth to 0, which means the action will fetch the entire commit history. This can be important if your subsequent steps depend on the full commit history.

```
      - uses: shivammathur/setup-php@v2
        with:
          php-version: 8.1
```

This GitHub Action step uses the shivammathur/setup-php action with version 2. It sets up a PHP environment for your workflow. The with block specifies the configuration option php-version, which is set to 8.1. This action will ensure that PHP version 8.1 is installed and available for use in your workflow.

These steps are typically used in a GitHub Actions workflow to prepare the environment and fetch the necessary code and dependencies before executing other tasks or jobs.

```
      - name: Getting Added or Modified filenames path
        id: curl
        run: |
          response=$(curl -s -X GET "https://api.github.com/repos/${GITHUB_REPOSITORY}/pulls/${{ github.event.pull_request.number }}/files" -H "Authorization: Bearer $GITHUB_TOKEN")
          files=$(echo "$response" | jq -r '.[] | select(.status == "added" or .status == "modified") | select(.filename | endswith(".php")) | .filename')
          files_list=($files)
          files=$(IFS=' '; echo "${files_list[*]}")
          echo "::set-output name=filtered_files::$files"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- This GitHub Action step is named "Getting Added or Modified filenames path." It performs the following tasks:
  - It sends an HTTP GET request to the GitHub REST API to retrieve information about the files modified or added in the pull request associated with the current workflow run. 
  - It uses the jq tool to process the JSON response and extract the filenames of files that have the "added" or "modified" status and end with the ".php" file extension. 
  - It stores these filtered filenames in the filtered_files output, which can be accessed in subsequent steps or jobs in your workflow. 
  - It sets the GITHUB_TOKEN environment variable using the GitHub Actions secret ${{ secrets.GITHUB_TOKEN }}. This token is required to authenticate with the GitHub API.

This step is commonly used to obtain a list of PHP files that were modified or added in a pull request, which can be useful for various purposes within your workflow.

```
      - name: Install PHP Insights
        run: |
          curl -OL https://getcomposer.org/download/latest-2.x/composer.phar
          php composer.phar global config --no-plugins allow-plugins.dealerdirect/phpcodesniffer-composer-installer true
          php composer.phar global require nunomaduro/phpinsights
          php composer.phar clearcache -q
          php ~/.composer/vendor/bin/phpinsights --version
```
- This GitHub Action step is named "Install PHP Insights." It performs the following tasks:
  - It downloads the latest version of Composer (composer.phar) from https://getcomposer.org/download/latest-2.x/composer.phar. 
  - It configures Composer to allow the dealerdirect/phpcodesniffer-composer-installer plugin using php composer.phar global config. 
  - It installs PHP Insights globally using Composer with php composer.phar global require. 
  - It clears the Composer cache with php composer.phar clearcache -q to ensure that the installation is clean. 
  - It checks the installed PHP Insights version using php ~/.composer/vendor/bin/phpinsights --version. 
  
This step is commonly used to install and configure PHP Insights, a tool for analyzing and improving PHP code quality, in a GitHub Actions workflow. After this step, you can use PHP Insights to analyze your PHP code within the same workflow.

```
      - name: Run PHP Insights
        if: github.event_name == 'pull_request'
        run: |
          files="${{ steps.curl.outputs.filtered_files }}"
          php ~/.composer/vendor/bin/phpinsights analyse $files --ansi -v --no-interaction --format=github-action --min-quality=90 --min-complexity=90 --min-architecture=90 --min-style=95
```

- This GitHub Action step is named "Run PHP Insights." It performs the following tasks:
  - It checks if the GitHub event that triggered the workflow is a "pull_request." This is specified using the if condition to ensure that the step only runs in pull requests. 
  - It retrieves the list of filtered files from a previous step named "curl" using ${{ steps.curl.outputs.filtered_files }}. This list of files contains those that are "added" or "modified" and have a ".php" file extension. 
  - It runs PHP Insights analysis on the filtered files using the phpinsights command. The --ansi, -v, --no-interaction, and --format=github-action flags specify various options for the analysis. 
  - It also sets specific quality, complexity, architecture, and style thresholds using --min-quality, --min-complexity, --min-architecture, and --min-style. These thresholds can be customized based on your project's requirements.
  
This step is commonly used to analyze the PHP code in pull requests, ensuring that it meets specific quality and style standards. The results of the analysis can be viewed within GitHub Actions or as part of the pull request checks.