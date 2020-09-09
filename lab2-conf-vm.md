# Lab 2: Setup your Oracle Cloud environment

## Overview


In this 10-minutes lab, you will prepare your Oracle Cloud environment to run the rest of the lab.

 
## Create a Virtual Cloud Network

In this step, we will create a Virtual Cloud Network (VCN), i.e. a software-defined private network in the Oracle Cloud Infrastructure. See [here](https://docs.cloud.oracle.com/en-us/iaas/Content/Network/Tasks/managingVCNs.htm) for more details on VCN.


1. Log on the OCI console, select **Networking** in **Core Infrastructure** (top left hamburger button) then **Virtual Cloud Networks**.

![](./images/lab2-1.png " ")

2. You will now create a Virtual Cloud Networks using the VCN Wizard, click **Start VCN Wizard**.

![](./images/lab2-2.png " ")

3. Select **VCN with Internet Connectivity** ➡ **Start VCN Wizzard**.

4. Give it a meaningful name, ex. "HOL_VCN", and keep other default values.

![](./images/lab2-3.png " ")

5. Clicking **Next** will display a summary of the VCN configuration. Review it and click **Create** to actually create the VCN.

After a couple of seconds, your VCN will be created (including a public and a private subnet, routing tables, an internet gateway, etc.).

![](./images/lab2-4.png " ")

You still need to do one thing, i.e. configure a security rule to allow requests coming from the Internet to reach your Java application running on OCI. For this, you will define an Ingress Rule on the VCN public subnet (not the private one!) to open port 8080.

1. **Core Infrastructure** ➡ **Networking** ➡ **Virtual Cloud Networks**, click on your newly created VCN to see its details.

![](./images/lab2-5.png " ")

2. Click on the public subnet (not the private one!)

![](./images/lab2-6.png " ")

2. Click on the default security list and **Add Ingress Rules**.

![](./images/lab2-7.png " ")

3. Fill in the **Source CIDR** and the **Destination port** as follow, and click **Add Ingress Rules**.

![](./images/lab2-8.png " ")

You now have a VNC properly confugired, you can move on to the next step.

## Provision a Compute Instance

In this step, you will configure and provision a new Compute Instance that we will use to test new Java features.

Compute Instances can be physical (bare metal) or virtual, they come in different shapes (memory, CPUs, storage, network, GPUs…). You can also choose between different OS…

For this lab, you will configure a VM based instance using the _default shape_ and the _Oracle Linux 7.8_ (OEL) image.

1. From the top left hamburger menu, select **Core Infrastructure** ➡ **Compute** ➡ **Instances**, click **Create Instance**.

💡 If you don't see **Create Instance** button, make sure that your **root** compartment is selected (check left sidebar **List Scope** - 
**COMPARTMENT**).

2. Keep the default OS image, i.e. _Oracle Linux 7.8_ and the _default shape_.

![](./images/lab2-9.png " ")

⚠️ OCI will generate the SSH keys pair required to authenticate in this new instance.
You must save the generated private key on your machine. Without it, your instance will be useless as you won't be able to log in! The generated public key will be automatically 'injected' in the instance during the provisioning.


3. Hit **Save Private Key**. Depending on your browser, the private key will either be downloaded and saved on your HD (ex. in the `~/Downloads` folder) or you might get a prompt asking where it should be saved.

![](./images/lab2-10.png " ") 

4. You can now click **Create**.

After 60~90 seconds, the big left square will switch from the **PROVISIONING** state (orange) to the **RUNNING** state (green). That means that your instance is up and running.

![](./images/lab2-11.png " ") 

⚠️ Make sure to write down the Public IP Address of your instance as you will need it!

Your instance is now up and running, you can connect to it!

In a shell on your computer, use `ssh` to connect to the instance on OCI: `ssh {username}@{public_ip}`.

💡 If you are on Windows, check [here](https://docs.cloud.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm#linux) how to use OpenSSH or PuTTY.

The default OEL username is **opc**. You also need to specify the path of the private key using the `-i` flag.
The final command should look like :

`ssh -i ~/Downloads/ssh-key-2020-xxx.key opc@158.xxx.xxx.xxx`.

💡 Some OS (ex. OSX) might complain about your private key's permissions being too loose (ex. "WARNING: UNPROTECTED PRIVATE KEY FILE!"). If that's the case, make sure to adjust your private key permissions so that it can't be read by others, `chmod 700 ssh-key-2020-xxx.key`.

If you get  should get a message saying "The authenticity of host '158.xxx.xxx.xxx' can't be established…", you can ignore it by typing **yes**. You are now connected to your OCI instance!

💡 You can also ingore the "LC_CTYPE: cannot change locale" warning, this will be corrected in the next step.



## Configure the instance for Java development


You know have a compute instance running Linux on OCI. Next, you will install the latest version of OpenJDK and other tools required for the Lab (Maven, Git, Helidon).

In your instance, run the following command:

```
source <(curl -L https://gist.githubusercontent.com/delabassee/a11e09dcf5a85dae87a5fd6a96ce77ea/raw/0504dc93b3af111b59c0bfc5abd7e0c5bb184362/vm-setup.sh)
```

The script should take around ~90 seconds. In the meantime, you can check what it is doing by typing its URL (https://gist.githubusercontent.com/delabassee/...) in a browser. In a nutshell, it: 
* fixes the "LC_CTYPE: cannot change locale" warning,
* installs various tools (git, tree, bat)
* installs the latest OpenJDK version,
* installs Apache Maven,
* installs the Helidon CLI,
* configures the VM firewall to open the 8080 port,
* and some miscellaneous details (ex. setting the path). 

Once the script has run, you can test your instance by issuing, for example, `java -version`.

Now that everything is correctly set-up, you can move to the next lab.





