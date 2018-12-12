# monit-control

################################################################################
#############################  Monit control file  #############################
################################################################################


##
#
# Comments begin with a '#' and extend through the end of the line. Keywords
# are case insensitive. All path's MUST BE FULLY QUALIFIED, starting with '/'.
#
# Below you will find examples of some frequently used statements. For 
# information about the control file and a complete list of statements and 
# options, please have a look in the Monit manual.
#
# For your convenience I left some of Monit original advices and added some of my
# own along with my own customizations, enjoy.
#
# Termiux
# http://sysadminlog.cowhosting.net
#
##


###############################################################################
## Global section
###############################################################################

##
# Start Monit in the background (run as a daemon):
##

set daemon  30

##
# Set syslog logging with the 'daemon' facility. If the FACILITY option is
# omitted, Monit will use 'user' facility by default. If you want to log to 
# a standalone log file instead, specify the full path to the log file. Example: set logfile /var/log/monit.log
##

set logfile syslog facility log_daemon                       

##
# Set the location of the Monit id file which stores the unique id for the
# Monit instance. The id is generated and stored on first Monit start. By 
# default the file is placed in $HOME/.monit.id.
##

set idfile /var/.monit.id

##
# Set the location of the Monit state file which saves monitoring states
# on each cycle. By default the file is placed in $HOME/.monit.state. If
# the state file is stored on a persistent filesystem, Monit will recover
# the monitoring state across reboots. If it is on temporary filesystem, the
# state will be lost on reboot which may be convenient in some situations.
##

set statefile /var/.monit.state

##
# Set the list of mail servers for alert delivery. Multiple servers may be 
# specified using a comma separator. By default Monit uses port 25 - it is
# possible to override this with the PORT option.
##

set mailserver localhost
##
# backup.bar.baz port 10025,  # backup mailserver on port 10025
# localhost                   # fallback relay
##

##
# Set event queue location and size
##

set eventqueue basedir /var/monit slots 500
##
# By default Monit will drop alert events if no mail servers are available. 
# If you want to keep the alerts for later delivery retry, you can use the 
# EVENTQUEUE statement. The base directory where undelivered alerts will be 
# stored is specified by the BASEDIR option. You can limit the maximal queue
# size using the SLOTS option (if omitted, the queue is limited by space 
# available in the back end filesystem).
##


## >>> This section is for use with M/Monit which I'm not using and will not cover. <<<
##
## 		Send status and events to M/Monit (for more informations about M/Monit 
## 		see http://mmonit.com/). By default Monit registers credentials with 
## 		M/Monit so M/Monit can smoothly communicate back to Monit and you don't
## 		have to register Monit credentials manually in M/Monit. It is possible to
## 		disable credential registration using the commented out option below. 
## 		Though, if safety is a concern we recommend instead using https when
## 		communicating with M/Monit and send credentials encrypted.
##
## 		set mmonit http://admin:monit@10.225.83.125:8080/collector 
## 		and register without credentials     # Don't register credentials
#
#
#


##
# Monit by default uses the following alert mail format:
#
## --8<--
#
set mail-format {
From: root@$HOST                         
Subject: Monit alert --  $EVENT $SERVICE  

message: 

Event description:

$EVENT Service $SERVICE                   
                                          
 	Date:        $DATE                  
 	Action:      $ACTION                
 	Host:        $HOST                  
 	Description: $DESCRIPTION           
                                           
 Monit monitoring system                   
}
## --8<--
#
# You can override this message format or parts of it, such as subject
# or sender using the MAIL-FORMAT statement. Macros such as $DATE, etc.
# are expanded at runtime. For example, to override the sender, use:
#
# set mail-format { from: root@localhost
# subject: $SERVICE $EVENT at $DATE
# message: Monit $ACTION $SERVICE at $DATE on $HOST: $DESCRIPTION.
# }
#
#
# You can set alert recipients whom will receive alerts if/when a 
# service defined in this file has errors. Alerts may be restricted on 
# events by using a filter as in the second example below. 
##

##
# Custom Mail formats
##

## ================== SERVER ALERT MESSAGES ==================== ##

##
# Send alerts on all events except those on braquets. Resend alerts if condition persists every 15 cycles
##

## ======= Message Starts ======= ##
set alert root@localhost but not on { 
	checksum, permission, uid,gid 
	} with reminder on 15 cycles 
## ======= Message Ends ======= ##

##
# Send alerts on checksum, permission, uid, gid, with the shown mail format. This is the email format
# that monit uses to mail me when a checksum, permission, uid of gid is changed in the server
##

## ======= Message Starts ======= ##
set alert root@localhost on {
           checksum, permission, uid, gid,
        } with the mail-format {

From: root@$HOST
Subject: Monit Intrusion Detection Alarm! --  $EVENT $SERVICE

message:

Event description:

There seems to be an intrusion alert on host $HOST more information about the event can be found below.


Unless an authorized adminisitrator is making changes to the server this indicates an intruder on your system modifying your system configuration, files, and/or permissions. Please look in to the matter.

If an authorized administrator is doing the modifications you can safely ignore this message

$EVENT Service $SERVICE

        Date:        $DATE
        Action:      $ACTION
        Host:        $HOST
        Description: $DESCRIPTION

 Monit monitoring system

}
## ======= Message Ends ======= ##


##
#
# Set Monit Server configuration values
# port, use of ssl and cert location, also allowed hosts, and users
#
# To access monit web interface you will have to enter your server hostname or ip address along with the 
# port number specified in here. Example: http://my.server.com:2812/
#
##

set httpd port 2812 
	enable ssl
	pemfile /etc/pki/tls/certs/cert.pem
	allow my.dns.ip.addr		# DO NOT erease this line or name resolves wont work (DNS server)
	allow localhost         	# allow localhost to connect to the server and
	allow my.server.ip.addr		# This is localhost! for some reason it complains if missing
	allow myPC.mydomain.com
	allow admin:"superL33Tpass0*&!2012$" 				# This user as admin privilegies
	allow monitoring:"monitorME@912!0*" read-only 		# This user as read only privilegies
##
# I'm not using groups for the moment
##

## 
# Groups usage examples
#
#     allow @monit           # allow users of group 'monit' to connect (rw)
#     allow @users readonly  # allow users of group 'users' to connect readonly
##

##
# Monit has an embedded web server which can be used to view status of 
# services monitored and manage services from a web interface. See the
# Monit Wiki if you want to enable SSL for the web server. 
#
# 	  set httpd port 2812 and use address 10.225.83.125
#     use address localhost  # only accept connection from localhost
#	  set mmonit http://monit:monit@10.225.83.125:8080/collector
##

##########################################################################################################
########################################## 		Services		##########################################
##########################################################################################################

	##
	# Check server and services for load conditions. Also depending on service request a server answer and take actions upon
	# conditions like, not answering, process dead, process consuming lots of resources, etc.
	#
	# NOTE: Most services have sentence similar to this: "if failed host 127.0.0.1 port 80 protocol http for 2 cycles then alert"
	# 		I force Monit to recheck the service a second time (2 cycles) before complaining or taking any action. I noticed that 
	#		sometime the server takes longer to answer than monit expects, making monit think service is not working. This eliminates those
	#		false negatives
	##

# =====================		Server		===========================

	check system ju1x10c1.ju.us.bosch.com
    	if loadavg (5min) > 3 then alert
    	if memory usage > 75% then alert
    	if swap usage > 25% then alert
    	if cpu usage (user) > 70% then alert
    	if cpu usage (system) > 50% then alert
    	if cpu usage (wait) > 45% then alert

# =====================		Apache		===========================
   

	check process httpd with pidfile /var/run/httpd.pid
		start program = "/sbin/service httpd start" with timeout 60 seconds
        stop program  = "/sbin/service httpd stop"
		if cpu > 60% for 5 cycles then alert
		if cpu > 80% for 10 cycles then restart
		if memory usage > 75% then alert
		if memory usage > 90% then restart
		if children > 250 then restart
		if loadavg(5min) greater than 10 for 30 cycles then stop
		if failed host 127.0.0.1 port 80 protocol http for 2 cycles then alert
		if failed host 127.0.0.1 port 80 protocol http for 4 cycles then restart
		if failed port 443 type tcpssl protocol http
		with timeout 15 seconds	then restart
		if 5 restarts within 5 cycles then timeout


# =====================         Vsftp          ===========================
		
    check process vsftpd with pidfile /var/run/vsftpd/vsftpd.pid
        start program = "/sbin/service vsftpd start" with timeout 60 seconds
        stop program  = "/sbin/service vsftpd stop"
        if cpu > 60% for 5 cycles then alert
        if cpu > 80% for 10 cycles then restart
        if memory usage > 75% then alert
        if memory usage > 90% then restart
        if failed port 21 protocol ftp for 2 cycles then alert
        if failed port 21 protocol ftp for 4 cycles then restart
        if 5 restarts within 50 cycles then timeout


# =====================		Ssh		===========================

	check process sshd with pidfile /var/run/sshd.pid
		start program "/sbin/service sshd start"
		stop program "/sbin/service sshd stop"
		if cpu > 60% for 5 cycles then alert
		if cpu > 80% for 10 cycles then restart
		if memory usage > 75% then alert
		if memory usage > 90% then restart
		if failed host 127.0.0.1 port 1234 protocol ssh for 2 cycles then alert
		if failed host 127.0.0.1 port 1234 protocol ssh for 4 cycles then restart
		if 5 restarts within 50 cycles then timeout

# =====================		MySQL		===========================

	check process mysqld with pidfile /var/run/mysqld/mysqld.pid
		start program = "/sbin/service mysqld start"
		stop program = "/sbin/service mysqld stop"
		if cpu > 60% for 5 cycles then alert
		if cpu > 80% for 10 cycles then restart
		if memory usage > 75% then alert
		if memory usage > 90% then restart
		if failed host 127.0.0.1 port 3306 for 2 cycles then alert
		if failed host 127.0.0.1 port 3306 for 4 cycles then restart
		if 5 restarts within 50 cycles then timeout

# =====================		Syslogd		===========================

	check process syslogd with pidfile /var/run/syslogd.pid
		start program = "/sbin/service syslog start"
		stop program = "/sbin/service syslog stop"
        if cpu > 60% for 2 cycles then alert
		if cpu > 80% for 5 cycles then restart
		if memory usage > 75% then alert
		if memory usage > 90% then restart
		if 5 restarts within 50 cycles then timeout

# =====================		Crond		===========================

	check process cron with pidfile /var/run/crond.pid
		start program = "/sbin/service crond start"
		stop  program = "/sbin/service crond stop"
		if 5 restarts within 5 cycles then timeout


# =====================		Sendmail	===========================

 	check process sendmail with pidfile /var/run/sendmail.pid
		start program = "/sbin/service sendmail start"
		stop  program = "/sbin/service sendmail stop"
		if failed port 25 protocol smtp for 5 cycles then restart
		if 5 restarts within 10 cycles then timeout

		
##########################################################################################################
########################################## 		File System		##########################################
##########################################################################################################

##
# Some simple alerts when running out of space
##

# =====================		File System Space     	======================= #

	check filesystem Data-Storage with path /dev/data
		if space usage > 70% for 20 cycles then alert

	check filesystem System-Storage with path /dev/mapper/VolGroup-SystemStorage
		if space usage > 70% for 20 cycles then alert
	##	
	# If you read carefully you can see that before getting alerts the space must be over 70% for at least
	# 20 cyles. This may seem like a lot, however I set it this way cause I have backups and other process 
	# running. This can increase the space usage in my disks for short periods of time. To avoid getting
	# low space alert messages, I wait some time to see if files are deleted before sending any messages.
	# Feel free to customize the number of cycles to meet your needs.
	##
		
		
##########################################################################################################
###################################### 		Integrity Checks		######################################
##########################################################################################################

##
# Paths of files I want to make a checksum, there is a specific mail format for this kind of alers, you can
# check that section around the beggining of the file
##

# ================= 	System Files Integrity Check 	=============== #

	check file Crontab with path /etc/crontab
	if failed checksum then alert

	check file Monit-Config with path /etc/monitrc
	if failed checksum then alert

	##
	# This is a very nice feature of Monit. It can execute a program or script upon meeting the criteria, like 
	# checksum changed. When I detect that my .forward file in the root directory is messed with I execute a litte
	# Bash script I wrote that resets the file to the original values, besides doing this it send an alert.
	# this give you assurance that you will stil receive some mails but that inmediate action is needed. The nice
	# part is that you can do pretty much everything cause you write the script/program ;P
	##
	check file Root-Forward-File with path /root/.forward
	if failed checksum for 1 cycles then exec '/usr/local/serverScripts/forwardGuard'
	if failed checksum for 1 cycles then alert
	

	check file Apache-Config with path /etc/httpd/conf/httpd.conf
	if failed checksum then alert
	
	check file SSH-Config with path /etc/ssh/sshd_config
	if failed checksum then alert

	check file LogWatch-Cron-Script with path /etc/cron.daily/0logwatch
	if failed checksum then alert

	check file LogWatch-Config with path /usr/share/logwatch/default.conf/logwatch.conf
	if failed checksum then alert

	check file Sendmail-Config with path /etc/mail/sendmail.mc
	if failed checksum then alert

	check file Samba-Config with path /etc/samba/smb.conf
	if failed checksum then alert

	check file Sudoers-Config with path /etc/sudoers
	if failed checksum then alert

	check file Init-Tab with path /etc/inittab
	if failed checksum then alert

	check file Init-SysConfig with path /etc/sysconfig/init
	if failed checksum then alert

	check file Modules-Config with path /etc/modprobe.conf
	if failed checksum then alert

	check file System-Config with path /etc/sysctl.conf
	if failed checksum then alert


###############################################################################
## Includes
###############################################################################
##
## It is possible to include additional configuration parts from other files or
## directories.
#
# Service Specific configuration on this dir
#include /etc/monit.d/*
#
#
