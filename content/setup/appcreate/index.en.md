---
title : "Create a web app"
weight : 20
---

Now that we have our Cloud9 development environment up and running, let's create our simple Java web application. 

### Install Maven & Java

[Apache Maven](https://maven.apache.org/) is a build automation tool used for Java projects. In this workshop we will use Maven to help initialize our sample application and package it into a Web Application Archive (WAR) file.


1. Install Apache Maven using the commands below (enter them in the terminal prompt of Cloud9):
:::code{language=bash showLineNumbers=false showCopyAction=true}
sudo wget https://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo
sudo yum install -y apache-maven
:::

2. Maven comes with Java 7. For the build image that we're going to use later on we will need to use at least Java 8. Therefore we are going to install Java 8, or more specifically [Amazon Correto 8](https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/what-is-corretto-8.html), which is a free, production-ready distribution of the Open Java Development Kit (OpenJDK) provided by Amazon:
:::code{language=bash showLineNumbers=false showCopyAction=true}
sudo amazon-linux-extras enable corretto8
sudo yum install -y java-1.8.0-amazon-corretto-devel
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64
export PATH=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/bin/:$PATH
:::

3. Verify that Java 8 and Maven are installed correctly:
:::code{language=bash showLineNumbers=false showCopyAction=true}
java -version
mvn -v
:::
![Cloud9 Java](/static/cloud9-java.png)

:::alert{header="Important" type="warning"}
If the command above doesn't return **openjdk version 1.8** (=> Java 8), then run the following command that allows you to choose the correct Java version:
```
sudo alternatives --config java
```
:::


### Create the Application

1. Use mvn to generate a sample Java web app
:::code{language=bash showLineNumbers=false showCopyAction=true}
mvn archetype:generate \
    -DgroupId=com.wildrydes.app \
    -DartifactId=unicorn-web-project \
    -DarchetypeArtifactId=maven-archetype-webapp \
    -DinteractiveMode=false
:::

![mvn Create Success](/static/mvn-create.png)

2. Verify the folder structure has been created for the application. You should have **index.jsp** file and a **pom.xml**.
:::code{language=bash showLineNumbers=false showCopyAction=true}
.
├── README.md
└── unicorn-web-project
    ├── pom.xml
    └── src
        └── main
            ├── resources
            └── webapp
                ├── index.jsp
                └── WEB-INF
                    └── web.xml

6 directories, 4 files
:::

3. Modify the **index.jsp** file to customize the HTML code (just to make it your own!). You can modify the file by double-clicking on it in the Cloud9 IDE. We will be modifying this further to include the full Unicorn branding later.
:::code{language=html showLineNumbers=false showCopyAction=true}
<html>
<body>
<h2>Hello Unicorn World!</h2>
<p>This is my first version of the Wild Rydes application!</p>
</body>
</html>
:::

Great work! Now that we have some Java source code locally, let's start with the first lab, where we're going to store the source code in a Git repository. Introducing AWS CodeCommit!