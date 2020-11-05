# Step to delete lab resources

When you're finishing the lab, perform the following steps to Delete resources from the account to avoid additional charges

1. Delete ECSs
2. Delete Security Groups
3. Delete NAT Gateway
4. Release EIPs
5. Delete Subnets and Route Tables
6. Delete VPC

Detail steps are listed below

### Task 1 - Delete ECS 
1. Navigate to HUAWEI CLOUD Console and select **Elastic Cloud Server** 
2. Select the ECS that you have created for this workshop e.g. ecs-red-private or ecs-red-bastion (You can select multiple ECSs)
3. Click **More**, choose **Delete** 
4. On pop-up windows, check the box on these following options:
    - Release the EIPs bound to the ECSs
    - Delete all data disks attached to the ECSs
5. Click **Yes** to confirm the deletion of the ECSs
6. Wait few seconds and the ECSs should be deleted from your account

### Task 2 - Delete Security Groups
1. Navigate to **HUAWEI CLOUD Console** and select **Virtual Private Cloud**
2. Click **Access Control** and select **Security Groups**
3. Look for the security groups that you have created for this lab. For example sg-ssh-internet or sg-ssh-internal then click **More** and select **Delete**
4. On pop-up windows, confirm the security group that you're going to delete then click **Yes**
5. Do the same for another security group

### Task 3 - Delete NAT Gateway
1. Navigate to **HUAWEI CLOUD Console** and select **NAT Gateway**
2. Select NAT Gateway that you have created for this workshop e.g. nat-red
3. On the NAT Gateway detail screen, delete SNAT Rule by clicking **Delete** at the SNAT Rule ID
4. On the pop-up windows, confirm the SNAT Rule that you're going to delete and click **Yes**
5. Once the SNAT Rule has deleted, navigate back to **NAT Gateway Console**
6. Click **Delete** at the NAT Gateway that you have created. 
7. On the pop-up windows, confirm the NAT Gateway that you're going to delete and click **Yes**

### Task 4 - Release EIP
Because when we delete our ECSs and choose to release the EIP during the deletion, only 1 EIP that was bound to NAT Gateway will be left. 

1. Navigate to **Elastic IP Console** 
2. Select the EIP that show status as **Unbound** 
3. Click **Release** to release the EIP back to the pool
4. On pop-up windows, confirm the EIP that you're releasing and click **Yes**


### Task 5 - Delete subnets and Route Table

1. Navigate to **Virtual Private Cloud Console** and select **Subnet**
2. Look for the subnet that has custom route table associated. e.g. subnet-pub-red
3. Click **Change Route Table** and select the default route table to make change, then click **OK**
4. Click **Delete** to delete the subnet
5. Wait few seconds and delete another subnet
6. Navigate to **Virtual Private Cloud Console** and select **Route Table**
7. Look at the custom type route table, click **Delete**
8. Leave the default route table as is

### Task 6 - Delete VPC
1. Navigate to **Virtual Private Cloud Conole** 
2. select the VPC that you have created for this work shop e.g. vpc-red
3. Click **Delete** to delete the VPC
4. On pop-up windows, confirm the VPC that you're deleting and click **Yes**

# Now you have deleted all the resources that you have created during this workshop!!
