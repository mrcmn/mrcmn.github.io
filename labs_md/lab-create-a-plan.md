# Overview

In this lab you will be creating a Puppet Plan which will consist of two Tasks, which will run sequentially. You’ll be reusing the Task from the previous lab along with an additional Task to achieve this end to end workflow.

You'll perform the following steps:

* Create secondary Task
* Create a simple Plan
* Review Task and Plan

> This lab assumes you have completed the following <span style="text-decoration:underline;">prerequisite</span> labs - <a href="https://puppet-enterprise-guide.com/labs/lab-create-a-task.html" target="_blank">Create a Task</a> and <a href="https://puppet-enterprise-guide.com/labs/lab-run-your-task.html" target="_blank">Running your Task</a>

# Steps

### Step 1: Create secondary Task

1. Navigate to the **tasks** folder - **site-modules/adhoc/tasks**
2. Click the **+** icon in the navigation bar at the top and select **New file**
3. Name the script as shown below and add the relevant content to your script:

    #### Linux: `lin_network.sh`

    ```bash
    #!/bin/bash
    netinfo=$(ip addr)
    echo -e "Network Info:\n \n$netinfo"
    ```

    #### Windows: `win_network.ps1`

    ```powershell
    $netinfo = ipconfig
    write-output "Networking Info:" $netinfo
    ```

4. Within **site-modules/adhoc/tasks** create your relevant Task Metadata files with the appropriate names and content, as shown below:

    #### Linux: `lin_network.json`

    ```json
    {
       "description": "Displays network information for a given Linux node via the ip command",
       "input_method": "environment"
    }
    ```

    #### Windows: `win_network.json`

    ```json
    {
       "description": "Displays system information for a given Windows node via the ipconfig command",
       "input_method": "powershell"
    }
    ```

    Your tasks directory should look similar to this:

    #### Linux

    <div class="noninteractive">

    ```
    ├─ site-modules/    
        └─ adhoc/      
     	  └─ tasks/
           	└─ lin_info.sh
           	└─ lin_network.sh
    ```
    </div>

    #### Windows

    <div class="noninteractive">

    ```
    ├─ site-modules/    
        └─ adhoc/      
     	  └─ tasks/
            └─ win_info.ps1 
            └─ win_network.ps1 
    ```
    </div>
    
### Step 2: Create a simple Plan


1. Within **site-modules/adhoc/plans** create a Plan file which will use both the Task from the previous lab and the one you’ve just created. Name the Plan and add the relevant content to your Plan as shown below: 

    #### Linux: `all_info_linux.pp`

    ```puppet
    # @summary This is a lab plan for Linux that displays system and network info.
    # @param targets The targets to run on.
    # @param myparam1 Input a custom value of your choice.
    plan adhoc::all_info_linux(
      TargetSpec $targets,
      String $myparam1
    ) {

      run_task('adhoc::lin_info', $targets, 'myparam1' => $myparam1)

      run_task('adhoc::lin_network', $targets)

    }
    ```

    #### Windows: `all_info_windows.pp`

    ```puppet
    # @summary This is a lab plan for Windows that displays system and network info.
    # @param targets The targets to run on.
    # @param myparam1 Input a custom value of your choice.
    plan adhoc::all_info_windows(
      TargetSpec $targets,
      String $myparam1
    ) {

      run_task('adhoc::win_info', $targets, 'myparam1' => $myparam1)

      run_task('adhoc::win_network', $targets)

    }
    ```

### Step 3: Review Task and Plan

1. Review the contents of the new Task and Plan content:

    #### Linux Task

    The `adhoc::lin_network` task echos the output of the `ip addr` command to the console.

    #### Windows Task

    The `adhoc::win_network` task writes an output of the `ipconfig` command to the console.

    #### Windows/Linux Plan 

    The Plans Metadata lives at the top of the plan manifest. This includes a description for the Plan itself as well as any descriptions of parameters the Plan accepts. The layout of the Plan manifest is very similar to a regular Puppet manifest which you’ll see later in the guide. The name of the Plan is specified below the Plan metadata. After that, Plan parameters are specified. 

    Following parameters, Plan steps are then specified which form the basis of what you want to achieve as part of this Plan. 

    This Plan is using two `run_task` steps in this simple workflow. In each of the `run_task `steps, you specify the name of the Task you want to run, followed by the targets it should run against and any parameters that you want to pass to the Task. Notice that in this example, you’re assigning the `'myparam1'` Task parameter directly to the Plan parameter `$myparam1`. This allows the user of the Plan to pass Task parameters to any Tasks that require it.

    You can find many more Plan steps (also known as Plan functions) that can be used within Plans <a href="https://puppet.com/docs/bolt/latest/plan_functions.html" target="_blank">here</a>.
