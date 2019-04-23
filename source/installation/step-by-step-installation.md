# Step by step installation

As for many other open source products, there are several ways to install OpenFisca. If you have no idea whether you want to use brew or pip or sudo and you never heard those words before, just follow this guide!

## Setting your working environment

**1 - Install the free [Docker platform](https://www.docker.com/get-started).**

*On Windows operating system, you need to have Windows 10 or higher to use Docker.*

**2 - When Docker is installed, run the app and create a free Docker ID.**

![Docker screen capture](/img/docker_hub.png)

**3 - Create a folder where you will install OpenFisca. In this example, we will call it “my-openfisca”.**

**4 - Point your folder in the terminal. On a mac, you can right-click in the folder and click on “Open folder in terminal”.**

![Window screen capture](/img/open-terminal.png)

![Terminal screen capture](/img/in-terminal.png)


**5 - Now that your folder is opened in the terminal, we can ask the terminal to install Python 3 in this folder.**

Python 3 is the programmation language used by OpenFisca. 
Just copy-past this command line and paste it into your Terminal.

    docker run --rm -it -v $PWD:/my-openfisca -w /my-openfisca python:3.7 bash
    

**6 - Check that it's all good.**
use this command line :

    `pip list` 

You should get this list of packages:

    ```sh
     Package    Version
     ---------- -------
     pip        *.*.*   
     setuptools *.*.* 
     wheel      *.*.* 
    ```


## Add a tax & benefit system. 

Let’s start with an imaginary country called *wonderland*. 
Each existing country-package is somewhere on github. 
You can reach them by exploring our website. 

We will start with *wonderland* country-package. The corresponing github repository is: https://github.com/openfisca/country-wonderland

Let’s now ask the terminal to reach this repository and clone it onto your computer, adding this command line in your terminal: 


    git clone https://github.com/openfisca/country-wonderland.git


## All good! You can now get ready to do wonderful things with the law!

Oh no! I don’t know what to do with my country system. Can I get a [tutorial]()?

## Your feedback is precious!

How did the installation go?

[All good!]()

[Something went wrong]()