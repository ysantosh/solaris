# solaris
solaris commands and helps
## Basic SMF commands
### SMF consists of three command line utilities:
* svcs: allows you to examine the state of your services and determine what went wrong.
* svcadm: enable, disable, and restart a service.
* svccfg: load manifest files (XML) that maintain configurations for each service.
### svcs - check status of services
The svcs command displays information about the state of your services. Services status are in three different states:

* online: the service is enabled and functioning normally
* offline: the service is stopped and disabled
* maintenance: the service has encountered a problem and is on hold until the problem is addressed by an administrator.

Running the ```svcs``` command with the ```-a``` argument displays a list of all online and offline services. By default ```svcs``` prints out:
```
online         15:18:16 svc:/site/splunk:default
disabled       Oct_10   svc:/network/physical:nwam
```

* status: the current state of the service.
* stime: when the service entered the current state.
* FMRI: the name of the service.
Each service is identified by a Fault Management Resource Identifier (FMRI). For example, the FMRI svc:/site/splunk:default

The above FMRI breaks down in the following way:

String	Description
* svc:	The service type
* /site/splunk	The service name
* :default	The service instance

### svcadm - manage service Starting, stopping and Restarting
The svcadm command is used to enable, disable, restart, or refresh services. 

For example, To start the splunk service you can use

```svcadm enable splunk```

You can see the status and start output at logs  ``` /var/svc/log/site-splunk:default.log ``` 

You use disable to stop services.

```svcadm disable splunk```

You use restart to refresh a service. 

```svcadm restart splunk```

If the service doesn't respond to the ```restart``` command ,you will need to ```disable``` and ```re-enable``` the service.

If a service enters maintenance mode or becomes disabled, you will need to perform some maintenance before SMF can restart the service. This include 
* Resolving any conflicts that prevent the service from running
* Running ```svcadm clear``` to clear the service state and so that it can resume

For example if splunk process runs on port 9998 and it is aquired by some process then
* Resolve the IP conflict by modifying the splunk configuration to use another IP or terminating the conflicting service.
* Clear the state using 

``` svcadm clear svc:/site/splunk:default ``` 

Once the state is cleared, the service can resume. 

Each service has an assigned service restarter agent that is responsible for carrying out actions against it. The default service restarter is ```svc.startd```
* If a service instance is in maintenance mode, this command informs the restarter agent that the service was repaired.
* If a service instance is disabled, this command requests that the restarter agent transition the service to the online state.

### Troubleshooting a service is in maintenance mode
SMF will place a service in maintenance mode when the service encounters something that causes it to crash. This usually indicates an error with the service but can also occur if your SmartMachine is running out of resources (RAM or disk space).
To verify if a service is in maintenance mode run this command:

```svcs -a```

This will show all running and disabled services. If a service is in maintenance mode, you will see something similar to this:

```maintenence OCT_10 svc:/site/splunk:default```

Review the log to root-cause why the service was in maintenance mode.

```more $(svcs -L service_name)```

Take the service out of maintenance mode:

```svcadm clear service_name```

### Examining service contracts using svcs -p
SMF maintains a contract with every running service it manages. The contract keeps track of what processes are running for any given service. Using the -p option, you can determine all the processes that belong to a service.
```
[root@splunkmaster1 /etc]# svcs -p  splunk
STATE          STIME    FMRI
online         15:18:16 svc:/site/splunk:default
               15:13:15      303 splunkd
               15:13:15      304 splunkd
               15:13:15      307 mongod
               15:13:16      313 python2.7
               15:13:18      330 splunkd
 ```

The following example demonstrates how SMF restarts a service when it stops unexpectedly:

``` 
kill -9 330
[root@splunkmaster1 /etc]# svcs -p  splunk
STATE          STIME    FMRI
online*        15:18:16 svc:/site/splunk:default
               15:13:15      303 splunkd
               15:13:15      304 splunkd
               15:13:15      307 mongod
               15:13:16      313 python2.7
               15:13:18      341 splunkd
 ```
Even though the splunk daemon was unexpectedly terminated, it was automatically restarted by SMF. Notice that the STIME shows that the splunk service is back online. The inclusion of an asterisk with the "online" state indicates that the service is currently in transition. However, the splunk service is already back online by the time the next command is run.

### Configuring services using svccfg
The svccfg command allows you to import, export, and modify service configurations. You specify entities to manipulate by using the -s option with an FMRI. The following example will set an environment variable for the specified FMRI with the value you specify.

``` svccfg -s FMRI setenv ENV_VARIABLE value```

You can invoke ```svccfg``` directly with individual subcommands or by specifying a script file. If you make any changes to a service using this command, you need to restart the service for the changes to take effect.

#### Enabling SMF Access

If you want to enable users without root access to manage SMF, you can modify a user profile as follows:

Open /etc/user_attr for edit.

Add this line replacing "myuser" with the login you want to enable:
```myuser::::profiles=Service Management```

After this change, the specified user is able to manage SMF (import, stop, start) without access to root. This minimizes the need for unnecessarily sharing root access among users.




