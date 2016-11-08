#CSYE 6225 - HW 8
###Colin Dickerson
###November 8th, 2016

This README file contains all of the steps necessary to complete HW #8. Source files for this assignment are contained in the **src** directory. The instructions assume you have created a google cloud platform account, but have no VM's allocated.

####Creating the VMs
To Begin we will create 4 NEW VM instances: Load Balancer, Server A, Server B, and Client. To create a VM instance, navigate to the Compute Engine section of the Cloud Platform screen, then Select "VM Instances". Repeat the following procedure 4 times, once for each VM.

1. Click the "Create Instance" button.
2. Name the VM accordingly
3. Select a "Zone" that is closest to your physical location. My personal selection was "us-east1-d"
4. For "Machine type"  keep the default selection: 1 vCPU, 3.75GB Memory
5. For "Boot disk" keep the default selection: 10GB, Debian GNU/Linux 8 (jessie)
6. For Identity and API Access keep the default selections: Comput Engine default service account & Allow default access.
7. **IMPORTANT**: For the Firewall, check the box next to "Allow HTTP traffic". This is requred for the system to operate correctly. Note: this is not strictly necessary for the Client VM, but the client VM is only for testing so it's ok to leave it enabled for now.

####Configuring the Server VMs
The next phase in setting up the server environment is to configure the two Server VMs (server-a & server-b) with Apache and uploading the html scripts into the apache instances. The following procedure needs to be repeated twice (once per server vm)

1. Access the VMs console window via SSH or using the browser console window
2. Update the apt-get cache using the following command: `sudo apt-get update`.
3. Install Apache using the following command: `sudo apt-get install -y apache2`.
4. Install PHP using the following command: `sudo apt-get install -y php5 libapache2-mod-php5 php5-mcrypt`
5. Open the Apache2 http root: `cd /var/www/html`
6. Download the server script from the CSYE 6225 GitHub Repo. Note the load.php file was provided by the professor as a way to increase the load on our servers.
    ```
    
    sudo wget https://raw.githubusercontent.com/CSYE6225/loader-engima/master/load.php
    
    ```
7. Create a random data file for use by the _load.php_ script: `sudo dd if=/dev/urandom of=1.log bs=5M count=2`

####Configuring the Load Balancer VM
The next phase in the server configuration is to configure the _Load Balancer_ server to balance incoming requests between the two _Server_ VMs. To do this we will again use Apache, but configured as a Load Balancer instead of a full web server.

1. Access the load balancer VMs console window via SSH or using the browser console window
2. Update the apt-get cache using the following command: `sudo apt-get update`.
3. Install Apache using the following command: `sudo apt-get install -y apache2`.
4. Install The Essential Build Tools using the following command: `sudo apt-get install -y build-essential`.
5. Install The Proxy Mod using the following command: `sudo apt-get install -y libapache2-mod-proxy-html`.
6. Install libxml2-dev using the following command: `sudo apt-get install -y libxml2-dev`. 
7. Install the XXXXXX library using the following command: `sudo apt-get install -y XXXXXXX`.
8. Type the command `sudo a2enmod`. You should receive a prompt stating "Which module(s) do you want to enable (wildcards ok)?". Enter the following module names and press enter: 
    ```bash
    
    proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html lbmethod_byrequests
    
    ```
9. Edit the defualt configuration file by typing the following command: `sudo nano /etc/apache2/sites-enabled/000-default.conf`. Append the following configuration element to the end of the file. *NOTE:* Replace the 0.0.0.0 element with the real external IP address of Server A, and Server B:
    ```xml
    
    <Proxy balancer://mycluster>
        BalancerMember http://0.0.0.0/
        BalancerMember http://0.0.0.0/
    </Proxy>
    
    ```
10. Restart the apache service using the following command: `sudo service apache2 restart`. The service should restart without any errors.

####Test With Client VM
This next step is to write a script file to test the load balancer using the client VM.

1. Access the client VMs console window via SSH or using the browser console window
2. Create a new script file using the command `sudo nano test.sh`
3. Modify the file contents to include the following code. Note replace 0.0.0.0 with the external IP of the load balancer server:
    ```bash
    #!/bin/sh

    while (sleep 5)
    do
    curl http://0.0.0.0/load.php
    done 
    ```
4. Mark the script file as executable using the following command: `sudo chmod +X test.sh`
5. Run the test script using the command `sudo ./test.sh`

####Capturing Server Load Data
This final procedure goes over how to capture the load data on Server A and Server B. Note the *grep* command was provided by the professor.

1. Access Server As VM console window via SSH or using the browser console window.
2. Run the following command to print out a list of server request per minute (seperated into minute buckets)
    ```
    
    grep load.php /var/log/apache2/access.log|awk '{print $10,$4}' |sed 's/\[//g' | awk -F\: '{print $1"_"$2"_"$3}' | awk '{arr[$2]+=$1}END{for(i in arr) print i,arr[i]/60}'
    
    ```
