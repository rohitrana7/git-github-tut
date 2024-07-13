## GitHub Actions 
- workflows are automated scripts that run on GitHub's servers to perform various tasks such as building, testing, and deploying code.
  Your workflow is defined in a YAML file (usually located in .github/workflows/)

- workflow.yaml
	```
	on: [push]
	
	jobs:
	  build:
	    runs-on: ubuntu-latest
	    steps:
	      - uses: actions/checkout@v3
	      - uses: actions/setup-node@v3
	        with:
	          node-version: '18'
	```
- "on" - The workflow is triggered by events specified in the on key. In this example, it runs when there is a push event.
	  
- "runs-on": ubuntu-latest: 
	- This specifies the environment where your workflow will run. 
	- GitHub provides hosted runners, which are virtual machines with pre-configured environments. 
	- ubuntu-latest means your job will run on a virtual machine with the latest Ubuntu version available.

- FYI: The runner(ubuntu-latest) pulls the action like "actions/checkout", "actions/setup-node", etc. from the GitHub Marketplace
- Using an action
	- Actions are reusable units of code that can be built and distributed by anyone on GitHub. You can find a variety of actions in GitHub Marketplace, and also in the official Actions repository.
	- "actions/checkout": This step clones your repository into the runner’s(ubuntu-latest) file system.
	- "actions/setup-node": This action installs the specified version of Node.js (version 18 in this case) on the runner.

- Hosted Runners(ubuntu-latest): GitHub provides a hosted runner which is a virtual machine managed by GitHub. 
	- These runners are ephemeral, meaning they are created for the duration of the job and destroyed afterwards. 
	- They are pre-configured with various software and tools, making it easy to set up and run workflows without needing to manage the infrastructure.

- GitHub Actions leverages AWS services like EC2 to provide scalable, isolated, and efficient environments for running workflows.
	- Yes, GitHub Actions does utilize AWS EC2 for its runners, among other cloud providers. 
	- GitHub hosts its runners on several major cloud platforms, including AWS, Azure, and Google Cloud. 
	- The runners, which are virtual machines executing the workflows, are often provisioned on EC2 instances.

- Checkout v4 marketplace- This action checks-out your repository, so your workflow can access it.
	- Only a single commit is fetched by default, for the ref/SHA that triggered the workflow.
	- The auth token is persisted in the local git config. This enables your scripts to run authenticated git commands. The token is removed during post-job cleanup. 

- Fetch only the root files
  ```
	- uses: actions/checkout@v4
	  with:
	    sparse-checkout: .
  ```
	
- Fetch only the root files and .github and src folder
  ```
	- uses: actions/checkout@v4
	  with:
	    sparse-checkout: |
	      .github
	      src
  ```
	  
- Checkout a different branch
  ```
	- uses: actions/checkout@v4
	  with:
	    ref: my-branch
  ```
	
- Setup Node.js environment v4 marketplace- optionally downloading and caching distribution of the requested Node.js version, and adding it to the PATH
	- Optionally caching npm/yarn/pnpm dependencies. E.g.:
   	```
	steps:
	- uses: actions/checkout@v4
	- uses: actions/setup-node@v4
	  with:
	    node-version: 18
	- run: npm ci
	- run: npm test
  	```

- The node-version input is optional. If not supplied, the node version from PATH will be used. However, it is recommended to always specify Node.js version and don't rely on the system one.

- Customizing when workflow runs are triggered
	-Set your workflow to run on push events to the main and release/* branches
	```
	on:
	  push:
	    branches:
	    - main
	    - release/*
	```
- Set your workflow to run on pull_request events that target the main branch
	```
	on:
	  pull_request:
	    branches:
	    - main
	```
- Set your workflow to run every day of the week from Monday to Friday at 2:00 UTC
	```
	on:
	  schedule:
 		# Runs "at 04:00 UTC (9.30 AM IST) every week days" (see https://crontab.guru). Although you can check cron format at below line
 		# -cron: "min hour day(month) month day(week)"
	  	- cron: "30 3 * * 1-5"
	```

- Manually running a workflow
	- To manually run a workflow, you can configure your workflow to use the workflow_dispatch event. This enables a "Run workflow" button on the Actions tab.
	```
	on:
	  workflow_dispatch:
	```

- Running your jobs on different operating systems
	- GitHub Actions provides hosted runners for Linux, Windows, and macOS.
	- To set the operating system for your job, specify the operating system using runs-on:
	```
	jobs:
	  my_job:
	    name: deploy to staging
	    runs-on: ubuntu-22.04
	```
- The available virtual machine types are:
	```
	ubuntu-latest, ubuntu-22.04, or ubuntu-20.04
	windows-latest, windows-2022, or windows-2019
	macos-latest, macos-13, or macos-12
	```

- Running steps or jobs conditionally
	- GitHub Actions supports conditions on steps and jobs using data present in your workflow context.
	- For example, to run a step only as part of a push and not in a pull_request, you can specify a condition in the if: property based on the event name:
	```
	steps:
	- run: npm publish
	  if: github.event_name == 'push'
	```
 
 - Running a job across a matrix of operating systems and runtime versions
	-You can automatically run a job across a set of different values, such as different versions of code libraries or operating systems.
	- For example, this job uses a matrix strategy to run across 3 versions of Node and 3 operating systems:
	```
	jobs:
	  test:
	    name: Test on node ${{ matrix.node_version }} and ${{ matrix.os }}
	    runs-on: ${{ matrix.os }}
	    strategy:
	      matrix:
	        node_version: ['18.x', '20.x']
	        os: [ubuntu-latest, windows-latest, macOS-latest]
	
	    steps:
	    - uses: actions/checkout@v4
	    - name: Use Node.js ${{ matrix.node_version }}
	      uses: actions/setup-node@v4
	      with:
	        node-version: ${{ matrix.node_version }}
	
	    - name: npm install, build and test
	      run: |
	        npm install
	        npm run build --if-present
	        npm test
	```
