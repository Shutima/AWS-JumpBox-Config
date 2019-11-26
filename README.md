# Welcome to GitHub Desktop!

This is your README. READMEs are where you can communicate what your project is and how to use it.

Write your name on line 6, save it, and then head back to GitHub Desktop.
# The Jump Box Web Server Set up
## What is a Jump Box Web Server?
The Jump Box Web Server is the Web Server that is connected between the Private Database and the NAT Instance. It allows the Private Database to be able to be accessed from public via NAT Instance using Public Subnet and Internet Gateway (IGW). This secure the public access to the private database because the private database stays in the private subnet.
# Steps by Steps
## Create VPC with two subnets
1. Create VPC using the console
   - **Name Tag**: Optionally provide a name of your VPC
   - **IPv4 CIDR Block**: Specify an IPv4 CIDR block for your VPC (ex., 10.0.0.0/16)
   ![VPC1](images/VPC1.jpg)