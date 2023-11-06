# DVD STORE 3.5 GUIDE

* Created by Dave Jaffe <davejaffe7@gmail.com> and Todd Muirhead  <tmuirhead@vmware.com>
* DVD Store 2.1 features added by Girish Khadke
* Last updated: 3/30/21
* Formatted for markdown and edited by Julie Brodeur: 11/6/23

**Contents**

[toc]

# Introduction

DVD Store Version 3.5 (DS35) is a complete, open source, online e-commerce test application with a backend database component, a web application layer, and driver programs. The goal of designing the database component and the midtier application was to use many advanced database features (transactions, stored procedures, triggers, referential integrity) while keeping the database easy to install and understand. The DS35 workload may be used to test databases or as a stress tool for any purpose.

The distribution includes code for SQL Server, Oracle, MySQL, and Postgres. (MySQL and Postgres versions will be added after the initial release of Oracle and SQL Server.) 

Included in the release are data generation programs, shell scripts to build data for any size DVD Store database with three standard sizes, database build scripts and stored procedures, PHP web pages, and a C# driver program. Load data for the small DVD Store database is included in the kit.

The DS35 files are separated into database-independent data load files under `./ds35/data_files`, driver programs under `./ds35/drivers`, and database-specific build scripts, loader programs, and driver
programs in the following directories:

```
./ds35/mysqlds35
./ds35/oracleds35
./ds35/sqlserverds35
./ds35/pgsqlds35
```

The DVD Store kit is available at https://github.com/dvdstore.

In the DVD Store 2.x versions, the kit contained multiple `.tar.gz` files. To simplify things, you can just download one zip file from GitHub or use the GitHub clone feature to get a local copy.

To install DS35, unzip the file in your preferred directory.  This document will assume that everything is in a `\ds35\` directory.

DS3 has 3 _standard_ sizes (based on the original standard sizes from DS2):

| Database | Size | Customers | Orders     | Products |
|----------|------|-----------|--------|----------|
| Small  | 10 MB| 20,000 | 1,000/month | 10,000 |
| Medium | 1 GB | 2,000,000 | 100,000/month | 100,000 |
| Large  | 100 GB | 200,000,000 | 10,000,000/month | 1,000,000 |

You can easily create other sizes using the utilities described below.

The DS35 project includes data files for the _Small_ size.

Most of the directories contain readme's with further instructions.

# What's new?

## What's new in DVD Store 3.5?

There are some great new changes and enhancements in the DS35.  Further details about these changes are in this document. <!-- Did you also add support for Postgres or other databases? Or was that already included in previous versions? -->

* To be able to scale the DVD Store workload to support larger-size databases, handle more users, and saturate bigger systems with many cores, DS35 adds the ability to have multiple stores per database. The user workload flow is still the same as before, but now it can be scaled across as many stores as you specify. Each store has its own set of DVD Store tables: customers, products, order history, reviews, etc. 
* The names for tables, indexes, stored procedures, and everything in DS35 now end in a number to denote which store they are part of.
* If you want to run DS35 to be exactly the same as DS30, simply specify that you want 1 store.  
* In addition to making the workload more scalable for bigger systems, we changed how the databases are created, loaded, indexed, and analyzed. These changes allow you to create much bigger databases, much faster because we implemented parallelization across tables and stores.
* With previous versions of DVD Store, everything was created in a single-threaded serial script. The tables were created, then loaded one at a time, then indexed one at a time, and, finally, analyzed one at a time.  
* With DS35, loading, indexing, and analysis are now scripted so that during each of these phases, all tables are loaded, indexed, and analyzed in parallel. For example, for the load phase, the customers, orders, purchases, order history, order detail, review, review details, membership, products, and inventory tables are all loaded at the same time. This dramatically reduces the time to load.
* All stores are also done in parallel. For example, if you specified 5 stores, then all 5 will be loaded at the same time, then all indexes across all stores will be created at the same time. This also dramatically reduces the time to load.
* Database creation has been almost completely automated to accomplish the above-mentioned parallelization and make it easier to set up and use. It is designed to be done from a remote client or driver system that is not the database host. It is still possible to run it all locally on the database system if desired, but we recommend using a remote driver.

## What's new in DVD Store 3.0?

* Product reviews are now part of DVD Store 3.0 (DS30). The <!-- simulated? --> customers of DS30 can now write new reviews and view existing reviews for products. Additionally, these customers have the ability to rate the helpfulness of reviews on a scale from 1 to 10.  There are seven new stored procedures to enable this functionality.
* Premium membership has been added to DS30, which enables customers to be able to sign up as bronze, silver, or gold-level premium members.  
* We updated the driver program to simulate customers taking advantage of these new features, which results in a new order process:
  * Log in or register a new customer
  * Sign up for a premium membership
  * Browse products by title, actor, or category
  * Browse product reviews
  * Create a new product review
  * Rate a review as helpful
  * Purchase

* We wrote new data generation programs to generate load files for the new reviews, helpfulness of reviews, and membership tables.  

* The `InstallDVDStore.pl` Perl script was updated so that it makes calls to the new data generation programs.

* All database build scripts were updated to create the new tables, new stored procedures, and new indexes.


# Quickstart steps

We give simple instructions for building the small DS35 database using the data included in the kit without use of the Perl scripts in each database directory; for example:
```
.\ds35\sqlserverds35\ds35_sqlserver_readme.txt
```

This section gives an overview of the steps needed to get a DS35 of any size up and running quickly using the new Perl scripts.

Here is a short list of steps that can get you started quickly by using the new Perl scripts. Additional details are given later in this document.

1. Install a supported operating system (Windows or Linux) on a VM with Oracle, SQL Server, MySQL, or PostgreSQL.
2. Download DS35 to a database system.
3. Ensure that Perl is set up on the system.
4. Run `InstallDVDStore.pl`.
5. Use the scripts  `InstallDVDStore.pl` creates to create and load the database.
6. On the system to be used as the driver, run `CreateConfigFile.pl`.
7. Run the driver program with the parameter file that `CreateConfigFile.pl` creates.

# Install and run DVD Store 3.x

## Step 1. Implement these prerequisites

### Install a supported database

To run DS35, one supported database (SQL Server, Oracle,  MySQL, or PostgreSQL) must already be installed and configured.

### If you are using a Windows machine, run these Perl scripts on it

To be able to run the DS30 or DS35 Perl scripts on a Windows machine that generates data and builds scripts for your database, you must have Perl installed on that machine. The scripts have been tested with Perl from ActiveState and Cygwin.

**ActiveState:**

1. Download the free Community Edition at http://www.activestate.com/activeperl/downloads. 
2. Select **ActivePerl** for the version of Windows you have and run the downloaded file to install Perl.

**Cygwin:**

1. Download `setup.exe` from http://www.cygwin.com/.

2. Run the installer `setup.exe` to install Cygwin.

3. The installer for Cygwin will ask for mirrors from which to choose to copy setup files. (If this is unsuccessful, you can retry by selecting another mirror since some mirrors may not be active.)

4. After selecting a mirror, the installer will ask which packages to install. (These packages are for emulating some Linux commands, including Perl.)

5. Select packages for gcc and Perl and their dependent packages. Or you can just select the default package and select the Perl checkbox. 

   **Notes:** 

   * Perl and gcc are not included in the default package and must be selected in addition to the defaults. 
   * gcc is required to generate the best quality data on Windows.

6. After the setup is successful, add the cygwin bin folder to the Path environment variable. For example, if cygwin is installed in folder `c:\cygwin\` then set the path variable as `c:\cygwin\bin\`.

7. Make sure that the above steps are successfully completed.

8. Now you can open the cygwin shell, navigate to the `ds3` or `ds35` directory, and run the Perl scripts. Alternatively, you can run Perl scripts from the DOS shell successfully if you have installed Cygwin and Perl.

The Oracle driver program `ds3oracledriver.exe` is now compiled with the 64-bit Oracle Databse 19c and Oracle Data Provider for .NET.

## Step 2. Use Perl scripts for data generation and script file generation

Perl scripts that automate the task of generating CSV data files are included with DS30 and DS35. They create database build and cleanup scripts and configuration files that will be used to run the driver program.

There are two Perl scripts:

* `InstallDVDStore.pl`
* `CreateConfigFile.pl`

### InstallDVDStore.pl

This Perl script generates CSV text files to be used to load as data in the DVD Store database and is structured based on the DVD Store schema. These files are stored in the `cust`, `orders`, `reviews`, `membership`, and `prod` directories under the `data_files` directory.  This Perl script also generates customized build script files for the database type selected under the database-specific directory.

`InstallDVDStore.pl` will ask for the following:

1. **Database Size as an Integer:** Enter any integer value you want. For example: `20`, `40`, `50`, `100`, `150`, etc.) <!--How large of a database size can you use?-->

2. **Database Size in MB/GB:** Indicate if the initial size entered is in megabytes or gigabytes. Possible values are `mb`, `gb`, `MB`, and `GB`.

3. **Database Type:** Specify one of the following values: 

   * `mssql` - for SQL Server
   * `mysql` - for MySQL
   * `pgsql` - for PostgreSQL
   * `oracle` - for Oracle Database 

5. **System OS Type:** Specify one of the following values: 
   * `win` - for Windows
   * `linux` - for Linux

6. **Database File Paths:** For Oracle and SQL Server databases, specify the paths where the following database files will be stored: primary dbfile, tables, indexes, and log. For multiple paths, you can provide a semi-colon-separated list. For a MySQL database, there is no need to enter this parameter. The following example provides 4 different paths:

​       `c:\sql\;d:\sql\;e:\sql\;f:\sql\ ` 

#### Database build scripts

The database build scripts are created based on the following set of template files that are included in DVD Store. <!-- Nothing for Postgres? Also, should the below locations and script names be updated to mysqlds35, etc and /ds35/...? -->

##### MySQL
| Database build script                   | Location              |
| --------------------------------------- | --------------------- |
| `mysqlds3_cleanup_generic_template.sql` | `/ds3/mysqlds3/build` |

##### Oracle
| Database build script                   | Location              |
| --------------------------------------- | --------------------- |
| `oracleds3_create_all_generic_template.sh` | `/ds3/oracleds3` |
| `oracleds3_cleanup_generic_fk_disabled_template.sh` |`/ds3/oracleds3/build`|
| `oracleds3_cleanup_generic_fk_disabled_template.sql` |`/ds3/oracleds3/build`|
| `oracleds3_cleanup_generic_template.sh` |`/ds3/oracleds3/build`|
| `oracleds3_cleanup_generic_template.sql` |`/ds3/oracleds3/build`|
| `oracleds3_create_db_generic_template.sql` |`ds3/oracleds3/build`|
| `oracleds3_create_tablespaces_generic_template.sql` |`/ds3/oracleds3/build`|

##### SQL Server
| Database build script                   | Location              |
| --------------------------------------- | --------------------- |
| `sqlserverds3_create_all_generic_template.sql` | `/ds3/sqlserverds3`
| `sqlserverds3_cleanup_generic_template.sql` | `/ds3/sqlserverds3/build`
| `sqlserverds3_create_db_generic_template.sql` | `/ds3/sqlserverds3/build`|
| `sqlserverds3_create_ind_generic_template.sql` | `/ds3/sqlserverds3/build`|

Build script files are generated from template files with ```_generic_template``` in the file name. The build scripts generated from these template files will have ```_databasesize``` in the file name.

For example: For a 20 megabyte SQL Server database instance, a file generated from: 
```
template file sqlserverds3_create_all_generic_template.sql 
```
will be named:
```
sqlserverds3_create_all_20mb.sql
```
New scripts are generated in the same folder in which their respective template files reside.
		
**Notes:**

1. The primary reason that the build scripts are created, instead of directly running the commands, is so that you can edit them later. It is very easy to modify these files to account for changing circumstances like changing paths where database files (primary, index, or table) or full-text index files will be stored. 
2. You can also change the default size of the database file and the default size of increments of growth in case of an overflow in size. <!-- Is this right? The original text said, "User can also change the default size of DBFile size and default size of increments of growth in case of overflow in size." -->
3. For the SQL Server database, you can specify seven different database file paths for the primary dbfile, misc dbfile, customer table, order table dbfile, index dbfile, log file, and full-text index file, respectively. The template script `sqlserverds3_create_all_generic_template.sql` assumes two dbfiles per table. If you want to change the number of dbfiles and file size or paths, edit the newly created script. If you specify only one path, the Perl script will assume the same paths for all dbfiles mentioned above and will create a database build script with the same path name for all dbfiles.
4. For Oracle, you can specify four paths for the following tablespaces respectively: customer, index, misc, and order. If you specify only one path, the Perl script will assume the same paths for all dbfiles mentioned above and will create a database build script with the same path name for all dbfiles.
5. All paths should be trailed by the following character: `\` for Windows and `/` for Linux. For example: 
    * For Windows, the path can be `c:\sql\dbfiles\;d:\sql\dbfiles\`
    * For Linux, the path can be ` /sql/dbfiles/`
7. Multiple paths will always be separated by the semi-colon character: `;`

After the execution of this Perl script, run the `CreateConfigFile.pl` Perl script to generate the configuration file used to drive the ds driver program.
	
### CreateConfigFile.pl

This Perl script will create a configuration file named `DriverConfig.txt` under the `\ds3` folder. After this file is created, you can edit this file for parameters like `n_threads`, `run_time`, `think_time`, etc.

The script asks for the following:
1. **Target host or hosts.** You can enter single or multiple targets on which to drive your workload. For multiple targets, the semi-colon (`;`)character will seperate each target. When creating a config file for a web server target, the web server port number can be included in the `target` parameter. The format to use for this is `<hostname/ip>:<portnumber>` where  `portnumber` is optional.
2. **Database size.** You must specify the database instance size created on the specified target host. For example: 5gb, 20gb, etc. All the targets must be the same size. If different sizes are needed, then you must run mulitple driver programs.
3. **Target hostname for perfmon display for windows targets. Optional parameter. Target Linux machine username, password, and IP address or machine name.** This 
parameter uses the format `<username>:<password>:<IP Address>`. For multiple Linux targets, you can give a semi colon (`;`) seperated list.
4. <!--This looks like it used to be #5, did I skip a step?--> **User has to specify value for detailed_view parameter which is used to show or not to show detailed view of statistics for each target.**  You must specify Y (for yes) and N (for no) as an answer.

## Step 3. Run the driver program using the configuration file

After all above steps and database creation, run the driver program from the command prompt as follows:
```
	ds3sqlserverdriver.exe --config_file=c:\ds3\DriverConfig.txt
```
You can run the driver program without specifying the config file, but you will need to provide all command-line parameters to the driver program. We recommend using the config file.

### To drive the workload on multiple target machines 

In DVD Store v2.0, driver programs were able to drive workload only against a single machine with a database server or web server installed. In v2.0, in order to drive workloads on multiple systems with a database or web server installed, you needed to open and run multiple instances of the driver programs at same time.
	
In DVD Store v2.1, we modified the driver program so that it distributes the threads equally against multiple target machines specified either on the command line or in a configuration file.

In addition to showing the total stats across all targets, the driver program collects the statistics for each individual machine. The stats are displayed on the console based on the `detailed_view` parameter, which is specified in the configuration file. If `detailed_view` is set to value `Y`, the driver program will print individual and aggregate statistics (like operations per minute, response times, CPU%, etc.) for each target machin every 10 seconds and after the run completes. 

If the value of `detailed_view` is `N`, the driver program will print aggregate or total statistics every 10 second and print total aggregate and individual statistics for each target machine after the run completes.

The prerequisites for driving a workload on multiple machines are as follows:
1. Each machine must have a database instance of the same size.
2. Threads will get spread equally on each machine. For example, if the `n_threads` parameter is set as `2` in the configuration file, and it has two target machine names, 4 threads will be spawned and will spread equally among two machines, which is 2 threads per machine.

Run the driver program as follows:
```	
ds3webdriver.exe --config_file=c:\ds3\DriverConfig.txt
```
The contents of `DriverConfig.txt` will look like this:
```
target=10.115.66.150;10.115.66.134
n_threads=4
ramp_rate=10
run_time=1
db_size=5gb
warmup_time=1
think_time=0.85
pct_newcustomers=20
n_searches=3
search_batch_size=5
n_line_items=5
virt_dir=ds3
page_type=aspx
windows_perf_host=10.115.66.150;10.115.66.134
detailed_view=Y
```
This example drives the workload against two target Windows machines with 5gb database instances and spawns 2 threads per machine. It prints individual and aggregate satistics to the console.
	
To get Linux CPU utilization data, specify the `linux_perf_host` parameter. A sample config file  that runs the driver against multiple Linux targets looks like this:
```
target=10.115.66.150;10.115.66.134
n_threads=4
ramp_rate=10
run_time=1
db_size=5gb
warmup_time=1
think_time=0.85
pct_newcustomers=20
n_searches=3
search_batch_size=5
n_line_items=5
virt_dir=ds3
page_type=php
linux_perf_host=root:qazwsx:10.115.66.150;root:xswzaq:10.115.66.134
detailed_view=Y
```

### Modifications to the driver program <!-- Shouldn't this section go in the release notes? Only include "To use a custom db size"? -->

#### Custom database size

In DVD Store v2.0, the driver program could drive a workload only on standard database sizes of S | M | L (10mb | 1gb | 100GB). However, in the current version of DVD Store, we have modified the driver program such that it can drive workloads on custom database sizes. 

#### To use a custom database size

The database size can be specified by the parameter `db_size` in the configuration file. Values of `db_size` can be S | M | L (to maintain backward compatibility with standard size databases) or any value of database size like 200mb, 20gb, 150gb, etc.
	
If you are driving database workloads on multiple target machines, each machine should have a database instance of the same size.

#### Streamlined query for SQL Server driver

For SQL Server, we discovered that a single query that was created during the purchase phase of the order made up a large percentage of the plan cache. Although the query was essentially the same each time, the SQL Server did not recognize it as the same and instead created a new plan and stored it in the plan cache each time. To remediate this problem, we converted the way the query was created so that it was a parameterized query against SQL Server that created a plan once and then reused it for every order.  
	
This change is significant because it changes the workload for SQL Server from the 2.0 version of the DVD Store. This change was found to increase the OPM by as much as 20% starting in version 2.1 of DVD Store.  

#### Additional error handling

We also modified the web driver program in a couple of places to more gracefully handle unexpected HTTP errors. We added a max timeout of 60 seconds for the initial connection of all threads. If all threads are not connected in 60 seconds, the driver program exits.

### Modifications to C programs for data generation

In DVD Store v2.0, data generation C programs generate data only for standard size instances (S | M | L). After generating CSV files, they needed to be converted into the proper CR/LF format with `unix2dos` to be used on a Windows system.

In version 2.1 of DVD Store, we modified C programs to generate CSV files for any database size. Also there is no need to convert	generated CSV files, which saves a lot of time for converting huge CSV files into the required format. If you use the `InstallDVDStore.pl` script, the data generation will be run automatically, and it will not be necessary to run these commands. The information is provided here as additional detail for those who are interested.

We modified the following programs, which are described in the following subsections:
```
ds3_create_cust.c
ds3_create_inv.c
ds3_create_orders.c
ds3_create_prod.c
```

#### ds3_create_cust.c

This is a new command-line argument is introduced for the C program `n_Sys_Type`. This denotes a system type for which CSV files needs to be generated in the correct format. `0` is for Linux, and `1` is for Windows. You can run this C program as follows:
```
./ds3_create_cust      1  10000 US   S 0 > us_cust.csv &
./ds3_create_cust  10001  20000 ROW  S 0 > row_cust.csv &
```
The above statements will generate customer data files for a 10mb instance on a Linux machine.

#### ds3_create_inv.c 

This is a new command-line argument for the C program `n_Sys_Type`. This denotes a system type for which CSV files needed to be generated in the correct format. `0` is for Linux, and `1` is for Windows.

You can run this C program as follows:
```
./ds3_create_inv 10000 0 > ../prod/inv.csv
```
The above statement will generate an inventory data file for a 10mb instance on a Linux machine

 #### ds3_create_orders.c

The following new command-line arguments are introduced for this C program:
```
n_Sys_Type (0 for linux and 1 for windows)
n_Max_Prod_Id (max product Id for custom database size)
n_Max_Cust_Id (max customer Id for custom database size)
```
This C program can be run as follows:
```
./ds3_create_orders     1  1000 jan S  1 0 10000 20000 &
./ds3_create_orders  1001  2000 feb S  2 0 10000 20000 &
./ds3_create_orders  2001  3000 mar S  3 0 10000 20000 &
./ds3_create_orders  3001  4000 apr S  4 0 10000 20000 &
./ds3_create_orders  4001  5000 may S  5 0 10000 20000 &
./ds3_create_orders  5001  6000 jun S  6 0 10000 20000 &
./ds3_create_orders  6001  7000 jul S  7 0 10000 20000 &
./ds3_create_orders  7001  8000 aug S  8 0 10000 20000 &
./ds3_create_orders  8001  9000 sep S  9 0 10000 20000 &
./ds3_create_orders  9001 10000 oct S 10 0 10000 20000 &
./ds3_create_orders 10001 11000 nov S 11 0 10000 20000 &
./ds3_create_orders 11001 12000 dec S 12 0 10000 20000 &
```
The above statements will generate order data files for a 10 MB instance on a Linux machine.

#### ds3_create_prod.c

The following command-line argument is introduced for this C program:
```
n_Sys_Type 
```
Use `0` for Linux and `1` for Windows.

Run this program as follows:
```
./ds3_create_prod 10000 0 > prod.csv
```
This statement will generate a products data file for a 10mb instance for a Linux machine.

> **Note:** It is also possible to copy the data files back and forth from Windows to Linux using the text or ascii filetype in programs such as WinSCP.

##  Step 4. For SQL Server 2008: Run the maintenance task

After the database build and bulk load finishes for SQL Server 2008, to get better performance, you must update the statistics of tables in the DS2 database. In previous versions of SQL Server (SQL Server 2005 and 2000), `sqlmaint.exec`, a command-line utility that came with the database server installation, was used.  This utility is obsolete and no longer included with SQL Server 2008, so you need to create a maintenance task needs to accomplish the same function in SQL Server 2008.

In the SQL Server 2008 Management Studio user interface, follow these steps:

1. Go to the Object Explorer and expand the database server tree.
2. Under the server tree, expand **Management** and right-click on **Maintenance Plans**.
3. Left-click on **Maintenance Plan Wizard**.
4. In the wizard, click **Next** and enter the name of the plan as `ds3`.
5. Click **Next** and check the **Update Statistics** checkbox. Click **Next**. 
6. Click **Next** again, and then choose the database **DS2** <!--Should this be DS3?-->. Click **OK**.
7. Ensure the **All Existing Statistics** and **Sample By** checkboxes are set along with the value **18 percent**.
8. After you have completed the above step, click **Next** twice to create a task under **Maintenance Plans** under the **Management** object, which is under the **SQL Server** tree.
9. Now right-click on the task `ds3` created from the above steps to see a menu. Click **Execute** to update the statistics on all tables in the DS2 database <!--Should this be DS3?--> using the task created in the previous steps.

For previous versions of SQL Server, you can directly invoke the `sqlmaint.exe` utility 
from the command prompt, as specified in older documentation.

## Step 5. For Oracle Database: Use the SpringSource web tier for database testing using the web driver

In DVD Store 2.1, the SpringSource MVC implementation of web pages for Oracle 
Database is provided with an old JSP implementation. If you want to use 
the SpringSource web tier instead of JSP, here are the steps to ensure the web tier 
is set up correctly on the web server:

1. To compile SpringSource code, you need JDK installed on the web server--preferably JDK 1.6 or later.

2. After installing, set the environment variable `JAVA_HOME` to the path where the JDK is installed. For example, ` JAVA_HOME `will have the value `C:\Program Files\Java\jdk1.6.0_20\`.
	
3. Also install Apache Ant to compile SpringSource code using Ant build commands.

4. After installing Ant, set the `ANT_HOME` environment variable to the path where Ant 
	is installed. For example, `ANT_HOME` will have value `C:\apache-ant-1.8.1\`.
	
5. Add similar entries for the PATH enviroment variable too. For example, for Java, the entry in the PATH environment variable will be `C:\Program Files\Java\jdk1.6.0_20\bin\` For Ant, the PATH entry will be `C:\apache-ant-1.8.1\bin\`.
	
6. Install Apache Tomcat web we server version 5.5 or above.

7. Set the `CATALINA_HOME` environment variable to the path where Tomcat is installed. For example, `CATALINA_HOME` will have value `C:\Program Files\Apache Software Foundation\Tomcat 5.5\`.
	
8. Download OpenSource SpringSource Framework 2.5.5 _with dependencies_ from the following link: http://sourceforge.net/projects/springframework-2/2.5.5/.
	
9. Unzip the zip file downloaded in the above step into a folder on system where the web server is installed. For example, unzip the file into the folder `c:\springsource-framework2.5.5\`.

10. Go to` /ds3/oracleds3/web/springsource/` and copy the `ds3web` folder into the `webapps` directory of the Tomcat 5.5 installation. For example, copy the `ds3web` folder in DVD Store 2.1 from  `C:\ds3\oracleds3\web\springsource\` to `C:\Program Files\Apache Software Foundation\Tomcat 5.5\webapps\` 

11. Copy following files from SpringSource Framework to ds3web folder in the `webapps` directory of Tomcat; that is, `/ds3web/WEB-INF/lib/`. 

| FileName   | Copy From Path                                 | Copy To Path                                                 |
| ---------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `spring.jar` | `C:\springsource-framework2.5.5\dist\spring.jar` | `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\` |
| `spring-webmvc.jar` | `C:\springsource-framework2.5.5\dist\modules\spring-webmvc.jar` | `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\` |
| `standard.jar` | C:\springsource-framework2.5.5\lib\jakarta-taglibs\standard.jar | C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\commons-logging.jar |
| `commons-logging.jar` | `C:\springsource-framework2.5.5\lib\jakarta-commons\commons-logging.jar` | `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\` |
| `servlet-api.jar` | `C:\springsource-framework2.5.5\lib\j2ee\servlet-api.jar` | `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\` |
| `jstl.jar` | `C:\springsource-framework2.5.5\lib\j2ee\jstl.jar` | `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\` |

Also obtain the Oracle J connector (`ojdbc14.jar`) from http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.html.
 	
1. Copy `ojdbc14.jar` to the `lib` folder under `ds3web` in Apache Tomcat webapps. That is, `C:\Program Files\Apache Software Foundation\Tomcat5.5\webapps\ds3web\WEB-INF\lib\`

1. Open a command prompt and go to `/webapps/ds3web/WEB-INF/` directory under the Tomcat installation. <!--Need where Tomcat installation is here?-->

1. Type `ant clean` and press **Enter**.  This step will clean up the older `*.class`  files under the `ds3web` folder.

1. Type `ant` and press **Enter**. This step will build all code under the `ds3web`  directory using the `build.xml` file in `/ds3web/WEB-INF/` and will put `*.class` files into their respective folders.

1. For the web driver to work for the Oracle web tier, you must create two triggers on the database server. The script to create triggers is located at `/ds3/oracleds3/web/springsource/oracleds3_create_trigger_springsource_only.sql`.  Run this script at the command prompt as `sqlplus ds3/ds3 @oracleds3_create_trigger_springsource_only.sql`

1. After the Ant build and all the above steps are complete, start the Apache Tomcat web server and check whether the DVD Store web tier is up and running using the following URL: `http://<hostname>:8080/dslogin.html`.

1. Now to run the web driver program and with SpringSource web pages being hosted by Tomcat. Some the parameter values for the driver program should be:
   ```
   virt_dir=ds3web
   page_type=html
   ```

You are done!
