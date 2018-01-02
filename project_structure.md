# Source Code

All source codes used in this book are available on
[github](https://github.com/zkoss/zkessentials). As our example
application has 3 different configurations, our source code is divided
into 3 branches:
![ center](images/ze-ch2-3branches.png)
* [**zk8**](https://github.com/zkoss/zkessentials/tree/zk8): contains examples from chapter 1 to chapter 7.
* [**zk8-spring**](https://github.com/zkoss/zkessentials/tree/zk8-spring): has examples integrated with Spring
* [**zk8-jpa**](https://github.com/zkoss/zkessentials/tree/zk8-jpa): contains examples which integrate with Spring and
persiste data into a database with JPA.



![ center](images/ze-ch2-download-zip.png)

You can click the "ZIP" icon to download the current selected branch as
a zip file.


# Run Example Application

After you download the source code, you will find it is a [Apache Maven](http://maven.apache.org/) project with jetty plugin configured. Therefore, you can start the example application on Jetty without deploying.

## No Maven Installed

## Maven Installed
Navigate to the root folder of the example project, e.g. it's "zkessentials" and type the command:

`mvn jetty:run`

Then visit the URL http://localhost:8080/essentials/, and you should
see the page below.

![](images/ze-ch2-index.png)


# Project Structure

The example project is a Maven project, and it's folder structure follows Maven's default convention. We name the source code packages according to each chapter, and each
package contains the classes used in the example of that chapter. Some
common classes are separated to an independent package as they are used
in multiple chapters, e.g. the classes under `org.zkoss.essentials.entity.*`
are entity class. We also define some service layer interfaces under
`org.zkoss.essentials.service.*` because different chapters have different
implementations.

For ZUL pages, we put them in an independent folder for each chapter
under `src/main/webapp/`. Under "WEB-INF" folder, **web.xml** contains
minimal configuration to run ZK and for its detail please refer to [ ZK
Installation Guide \\ Create and Run Your First ZK Application
Manually](http://books.zkoss.org/wiki/ZK%20Installation%20Guide/Quick%20Start/Create%20and%20Run%20Your%20First%20ZK%20Application%20Manually).
The "zk.xml" is optional configuration descriptor of ZK. Provide this
file if you need to configure ZK differently from the default behavior.
Refer to [ZK Configuration
Reference/zk.xml](http://books.zkoss.org/wiki/ZK%20Configuration%20Reference/zk.xml) for more
detail.


