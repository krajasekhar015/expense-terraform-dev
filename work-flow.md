## VPC (Virtual Private Cloud) Module 
Here, we are using our own VPC module.
* First, we need to create `VPC` with CIDR (Here, we are using `10.0.0.0/16` where first three octets are blocked)
     * Expose the `vpc_id` in the `outputs.tf` file
* Then, we need to create `Internet Gateway` and we have to attach to VPC
* Now, we need to  create `subnets` (frontend, backend & database) in two availability zones
     * Here we have to use `count` with `length` function since we are creating multiple subnets in multiple availability zones
     * User has to pass two CIDR blocks mandatorily. So, here we have to use `validation` function with `length` function to validate variable size
     * Using data-source, we will get availability zones dynamically and store in a local variable
     * Expose all the `subnet_ids` in the `outputs.tf` file
* Create `Database Subnet Group` which we will use in RDS
     * We need to provide database subnet ids which are exposed in outputs.tf file
* Create `Elastic IP` Address
* Create `NAT Gateway` in public subnet 
* Create `Route Tables` for public, private and database 
* Now, we need to create `routes`
     * create `route` for `public` with `internet gateway`
     * create `route` for `private` with `NAT gateway`
     * create `route` for `database` with `NAT gateway`
* At last, `associate route tables with subnets`
     * Associate public route table with public subnet
     * Associate private route table with private subnet
     * Associate database route table with database subnet
* Now, we will create vpc_pering which is optional (user may or may not use vpc_peering)
     * If user wish to use vpc_peering, then the condition is true and it will process
     * we need to provide requestor vpc_id and acceptor vpc_id
     * Here, we will create routes from requestor (public, private & database) to acceptor (here, we need to provide acceptor CIDR)
     * Then, we will create route from acceptor to requestor (here, we need to provide requestor CIDR) <br>

       > [!NOTE] <br>
       > Here, for every individual resource we need to use condition (whether vpc_peering is required or not by user) otherwise, terraform will create a 
         resource even if user don't want vpc_peering.

---
## Security Group Module
Here, we are using our own security group module
* We will create `security group` where user has to provide `vpc_id` and `ingress rules`
     * We need to expose the `security group id` in the `outputs.tf` file
---
## 1. Creating VPC 
This is the first step, where we will create vpc using our own vpc module as source.
* We will create vpc with subnets (public, private & database) 
     * We will push `vpc_id` and `subnet_ids` to the ssm parameter store
---
## 2. Creating Security Group
After creating VPC, we will create security groups using our own security group module.
* Here, we will create `security groups` for mysql, backend, frontend, bastion, ansible
     * Here, we will connect to bastion server and from there we will access our servers (frontend, backend and database)
* Now, we need to `create security group rules`
     * Create a security group rule for mysql which is allowing connection on `port-no.3306` from the instances attached to Backend Security group
     * Create a security group rule for backend which is allowing connection on `port-no.8080` from the instances attached to frontend Security group  
     * Create a security group rule for frontend which is allowing connection on `port-no.80` from public 
     * Create a security group rule for mysql, backend, frontend which are allowing connection on `port-no.22` from bastion
     * Create a security group rule for mysql, backend, frontend which are allowing connection on `port-no.22` from ansible
     * Create a security group rule for ansible and bastion which are allowing connection on `port-no.22` from public (internet `0.0.0.0/0`)
* Then, we need to store the security group ids (mysql, backend, frontend, bastion, ansible) in the ssm parameter store
---
## 3. Creating Bastion Server
* Due to security issues, we won't connect directly to our servers (frontend, backend & database) from any network. So, we will create bastion server and using this we will access our servers.
* First, we need to create ec2 instance. 
     * Using data-source, we will get the ami_id (here, we are using our own ami)
* We need to provide `bastion security group id` (we have stored all the SG ID's in the SSM Parameter Store) and We need to create this instance in `public subnet`
---  
## 4. Creating Application Servers
* First, we create `mysql` server
     * We need to provide mysql security group id
     * We need to create in database subnet 
* Create `backend` server
     * We need to provide backend security group id
     * We need to create in private subnet 
* Create `frontend` server
     * We need to provide frontend security group id
     * We need to create in public subnet 
* Create `ansible` server
     * We need to provide ansible security group id
     * We need to create in public subnet
          * Here, we will integrate terraform with ansible using file function
          * Create a file (any name) with an extension `.sh` (Here, we will create expense.sh)
          * In this file, write a shell script
               * Install Ansible
               * Clone the expense-ansbile repository
               * Run ansible inventory of mysql.yml, backend.yml and frontend.yml
* Create `DNS records`
---