# The Jump Box Web Server Set up
## What is a Jump Box Web Server?
The Jump Box Web Server is the Web Server that is connected between the Private Database and the NAT Instance. It allows the Private Database to be able to be accessed from public via NAT Instance using Public Subnet and Internet Gateway (IGW). This secure the public access to the private database because the private database stays in the private subnet.
# Steps by Steps
## Setting up the NAT Instance
1. Create VPC with two subnets
   - Create VPC using the console
     - **Name Tag**: Optionally provide a name of your VPC
     - **IPv4 CIDR Block**: Specify an IPv4 CIDR block for your VPC (ex., 10.0.0.0/16)
        ![VPC1](https://github.com/Shutima/desktop-tutorial/blob/version1/VPC1.jpg)
        ![VPC2](https://github.com/Shutima/desktop-tutorial/blob/version1/VPC2.jpg)
   - Create two Subnets: one Public Subnet and one Private Subnet
     - Open Amazon VPC Console > **Subnets** > **Create subnet**
     - **Name Tag**: Optionally provide a name of your subnet
     - **VPC**: Choose the VPC that was created
     - **Availability Zone**: Optionally choose an Availability Zone that your subnet will reside
     - **IPv4 CIDR Block**: Specify an IPv4 CIDR block for your subnet, for example, 10.0.1.0/24 for Public Subnet and 10.0.2.0/24 for Private Subnet
        ![subnet1](https://github.com/Shutima/desktop-tutorial/blob/master/subnet1.jpg)
        ![subnet2](https://github.com/Shutima/desktop-tutorial/blob/master/subnet2.jpg)
   - Attach an Internet Gateway to the VPC
     - Open Amazon VPC Console > **Internet Gateways** > **Create internet gateway**
     - Optionally name your Internet Gateway, then choose **Create**
        ![IGW1](https://github.com/Shutima/desktop-tutorial/blob/master/IGW1.png)
     - Choose the Internet Gateway you just created, then choose **Actions** > **Attach to VPC**
     - Select your VPC from the list, then choose **Attach**
        ![IGW2](https://github.com/Shutima/desktop-tutorial/blob/master/IGW2.png)
   - Create a custome Route Table that sends traffic destined outside the VPC to the Internet Gateway, and then associate it with one subnet, making it a public subnet
     - Open Amazon VPC Console > **Route Tables** > **Create Route Table**
     - In the **Create Route Table** dialog box, optionally name your route table, then select your VPC, and then choose **Yes, Create**
     - Select the custom route table that you just created.
     - On the **Routes** tab, choose **Edit, Add another route**, and add the following routes as necessary. Choose **Save** when you're done.
       - For IPv4 traffic, specify 0.0.0.0/0 in the **Destination** box, and select the internet gateway ID in the **Target** list.
       - For IPv6 traffic, specify ::/0 in the **Destination** box, and select the internet gateway ID in the **Target** list.
     - On the **Subnet Associations** tab, choose **Edit**, select **Associate** check box for the subnet, then **Save**
        ![RouteTable1](https://github.com/Shutima/desktop-tutorial/blob/master/RouteTable1.jpg)
        ![RouteTable2](https://github.com/Shutima/desktop-tutorial/blob/master/RouteTable2.png)
        ![RouteTable3](https://github.com/Shutima/desktop-tutorial/blob/master/RouteTable3.jpg)
2. Create NATSG Security Group. This will need to be specified when launching the NAT Instance.
   - Define the NATSG Security Group in order to enable your NAT instance to receive Internet-bound traffic from instances in a private subnet, as well as SSH traffic from your network. The NAT Instance can also send traffic to the internet, which enables the instances in the private subnet to get software updates.

**NATSG: Recommended Rules**
**Inbound**
| Source                                       | Protocol | Port Range | Comments                                                                                       |
|----------------------------------------------|----------|------------|------------------------------------------------------------------------------------------------|
| 10.0.1.0/24                                  | TCP      | 80         | Allow inbound HTTP traffic from servers in the private subnet                                  |
| 10.0.1.0/24                                  | TCP      | 443        | Allow inbound HTTPS traffic from servers in the private subnet                                 |
| Public IP address range of your home network | TCP      | 22         | Allow inbound SSH access to the NAT instance from your home network (over the Internet gateway)|

**Outbound**
| Destination                                  | Protocol | Port Range | Comments                                                                                       |
|----------------------------------------------|----------|------------|------------------------------------------------------------------------------------------------|
| 0.0.0.0/0                                    | TCP      | 80         | Allow outbound HTTP access to the Internet                                                     |
| 0.0.0.0/0                                    | TCP      | 443        | Allow outbound HTTPS access to the Internet                                                    |

   - Open Amazon VPC Console > **Security Groups** > **Create Security Group**
   - In the **Create Security Group** dialog box, specify the NATSG as the name of the security group, and provide description. Select the ID of your VPC from the **VPC** list, then choose **Create**
   - Select the NATSG group that just created. The details pane displays the details for the security group, and tabs for inbound and outbound rules.
   - Add rules for inbound traffic via **Inbound Rules** tab as followed:
     a. Choose **Edit**.
     b. Choose **Add another rule**, select **HTTP** from the **Type** list. In the **Source** field, specify IP address range of your private subnet.
     c. Choose **Add another rule**, select **HTTPS** from the **Type** list. In the **Source** field, specify IP address range of your private subnet.
     d. Choose **Add another rule**, select **SSH** from the **Type** list. In the **Source** field, specify the public IP address range of your network.
     e. Choose **Save**.
   - Add rules for outbound traffic via **Outbound Rules** tab as followed:
     a. Choose **Edit**.
     b. Choose **Add another rule**, and select **HTTP** from the **Type** list. In the **Destination** field, specify 0.0.0.0/0 
     c. Choose **Add another rule**, and select **HTTPS** from the **Type** list. In the **Destination** field, specify 0.0.0.0/0 
     d. Choose **Save**.
        ![SG1](https://github.com/Shutima/desktop-tutorial/blob/master/SG1.jpg)
        ![SG2](https://github.com/Shutima/desktop-tutorial/blob/master/SG2.jpg)
        ![SG3](https://github.com/Shutima/desktop-tutorial/blob/master/SG3.jpg)

3. Launch an instance into your public subnet from an AMI that is configured to run a NAT instance.
   a. Open Amazon E2 Console
   b. **Launch Instance** and do as followed:
      - On **Choose an Amazon Machine Image (AMI)** page, select **Community AMIs** category and seach for amzn-ami-vpc-nat. Choose the latest edition > **Select**.
      - On **Configure Instance Details** page, select the VPC created from **Network** list, and select your Public Subnet from **Subnet** list
      - On **Configure Security Group** page, select the **Select an existing security group** option, and select the NATSG Security Group that you just created > **Review and Launch**
      - Review the configure and **Launch**
        ![NATinstance](https://github.com/Shutima/desktop-tutorial/blob/master/NATinstance.jpg)
        ![NATinstance2](https://github.com/Shutima/desktop-tutorial/blob/master/NATinstance2.jpg)
        ![NATinstance3](https://github.com/Shutima/desktop-tutorial/blob/master/NATinstance3.jpg)
4. Disable the SrcDestCheck attribute for the NAT instance 
   - Open Amazon E2 Console > **Instances**
   - Select NAT Instance, choose **Actions, Networking, Change Source/Dest. Check**.
   - For the NAT Instance, verify that this attribute is disabled. Otherwise, choose **Yes, Disable**.
        ![NATins_Change1](https://github.com/Shutima/desktop-tutorial/blob/master/NATins_Change1.jpg)
        ![NATins_Change2](https://github.com/Shutima/desktop-tutorial/blob/master/NATins_Change2.jpg)
5. If you forgot to assign the public IP address to your NAT Instance during launch (step 3), you need to associate an Elastic IP address with it
   - Open Amazon VPC Console > **Elastic IPs** > **Allocate new address**
   - Choose **Allocate**
   - Select Elastic IP Address from the list > **Actions, Associate address**
   - Select network interface resource, then select the network interface from the NAT Instance. Select the address to assiciate to the Elastic IP with from the **Private IP** list > choose **Associate**
6. Update the main route table to send traffic to the NAT Instance
   - Open Amazon VPC Console > **Route Tables**
   - Select the main route table, the one that lable **Main** is **Yes**. The details pane displays tabs for routes, associations and route propagation.
   - On the **Routes** tab > **Edit**, specify 0.0.0.0/0 in the **Destination** box, select the instance ID of the NAT instance from **Target** list > **Save**
   - On the **Subnet Association** tab > **Edit** > **Associate** check box for the subnet > **Save**
        ![MainRT1](https://github.com/Shutima/desktop-tutorial/blob/master/mainRT1.jpg)
        ![MainRT2](https://github.com/Shutima/desktop-tutorial/blob/master/mainRT2.jpg)
        ![MainRT3](https://github.com/Shutima/desktop-tutorial/blob/master/mainRT3.jpg)
        ![MainRT4](https://github.com/Shutima/desktop-tutorial/blob/master/mainRT4.jpg)
7. Create a Jump Box Web Server instance
   - Open Amazon EC2 Console > **Launch Instance**
   - On the **Configure Instance Details** page, choose the created VPC, **Public Subnet** on the same VPC, and **Enable** for **Auto-assign Public IP**
   - Add Tags, for example "Jump Box Web Server"
   - On the **Configure Security Group**, choose existing security group that was created (NATSG_Shutima) > **Review and Launch**
        ![JumpBoxIn1](https://github.com/Shutima/desktop-tutorial/blob/master/JumpBoxIn1.jpg)
        ![JumpBoxIn2](https://github.com/Shutima/desktop-tutorial/blob/master/JumpBoxIn2.jpg)
        ![JumpBoxIn3](https://github.com/Shutima/desktop-tutorial/blob/master/JumpBoxIn3.jpg)
8. Create a third instance for Private Database. Follow the steps from 7) but this time, need to specify as followed (same VPC, Public subnet, disable **Auto-assign Public IP**)
        ![PrivateDB](https://github.com/Shutima/desktop-tutorial/blob/master/PrivateDB.jpg)
        ![PrivateDB2](https://github.com/Shutima/desktop-tutorial/blob/master/PrivateDB2.jpg)
9. As soon as all the three instances are configured, you can connect to each instance as per following commands, make sure you go into the directory where the permission key *.pem is located:
   - Connet to NAT Instance
   >ssh -i "Shutima_A19_KeyPair.pem" ec2-user@18.207.153.244
   - Connect to Jump Box Web Server
   >ssh -i "Shutima_A19_KeyPair.pem" ec2-user@34.207.220.24
   - Connect to Private Database
   >ssh -i "Shutima_A19_KeyPair.pem" ec2-user@10.0.2.4
10. Test all of the instances

