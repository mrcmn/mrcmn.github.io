# Overview

In this lab, you'll create a node group and add your lab node to this group. This will enable you to target this specific group with desired state configurations as well as enabling easy targeting for ad-hoc automation.

You'll perform the following steps:

* Create node group
* Add your node to your node group

# Steps

### Step 1: Create node group

> If you’re working with only one platform, either _Linux_ or _Windows_, then create only the node groups that are relevant to your targets platform. 

1. From the navigation bar on the left, click on **Node groups**.
2. Click **Add group...**
3. Ensure your Node group parameters match the following:

    #### Linux

    | Parent name       | Group Name      |  Environment       | Environment group       | Description |
    | -----------       | -----------     |-----------         |-----------              | -----------  |
    | All Nodes         | Lab Group Linux | production         | [unchecked]             | My Linux lab node group  |

    #### Windows

    | Parent name       | Group Name        |  Environment       | Environment group       | Description |
    | -----------       | -----------       |-----------         |-----------              | -----------  |
    | All Nodes         | Lab Group Windows | production         | [unchecked]             | My Windows lab node group  |




4. Once complete, click **Add**


### Step 2: Add your node to your node group

#### Linux


1. Click on **Lab Group Linux** group.
2. In the **Rules** tab, under the **Certname** heading, enter the first few characters of your Linux servers hostname that you installed the agent on earlier, then select that node from the list that appears.
3. Click **Pin node**.
4. To save the changes, click **Commit 1 change** in the lower right hand corner of the screen.
5. Verify that your node has been successfully added to the group by reviewing the **Matching nodes** tab.

 		

#### Windows


1. Click on **Lab Group Windows** node group.
2. In the **Rules** tab, under the **Certname** heading, enter the first few characters of your Windows servers hostname that you installed the agent on earlier, then select that node from the list that appears.
3. Click **Pin node**.
4. To save the changes, click **Commit 1 change** in the lower right hand corner of the screen.
5. Verify that your node has been successfully added to the group by reviewing the **Matching nodes** tab.