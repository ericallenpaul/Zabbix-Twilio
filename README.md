# Zabbix-Twilio
A perl script for creating custom alerts in Zabbix that allows it to send text messages using the Twilio service. This script also supports a weekley on call schedule.
_(You will need a [Twilo](https://www.twilio.com/) account in order to use this script.)_

# Installation
Drop the script and the config file into [ZabbixAlertScriptsPath](https://www.zabbix.com/documentation/3.4/manual/appendix/config/zabbix_server) and configure it as shown in the [documentation](https://www.zabbix.com/documentation/3.4/manual/config/notifications/media/script).
The script relys on the following perl modules:

```perl
XML::Simple
WWW::Twilio::API
File::Log
DateTime
Data::Dumper
Date::Calc
```
You may need to download and install them with ppm.
You will also need to make sure this script has write access to its local directory.

# Configuration
The configuration file consists of properties and values separated by an equal sign. Hash tags can be used to place comments in this file. The configuration file supports the following options:

##### Account information
There are 2 settings here, accountSid and authToken. Both of these are supplied by Twilio and are essentially your login credentials.
```
accountSid = XXXXXXXXXXXXXXXXXXXXXXXXXX
authToken = XXXXXXXXXXXXXXXXXXXXXXXXXXX
```

##### Log File Level
The logfile level determines how much information is written to the log. 4 will write both successes and failures to the log while 5 will only write failures. The log files will be rotated every 7 days.
```
LogFileLevel = 4
```
##### UseWeeklySchedule
Enabled = 1, Disabled = 0
When enabled it looks at OncallNumbers and rotates the weeks between them. This also effects how the schedule is created. The schedule is nothing more than a directory with files (1-52). Each file holds the phone number for the person who is "on call" for that week.
```
UseWeeklySchedule = 0
```

##### OnCallNumbers and OnCallNames
On call numbers is a command delimited list of phone numbers that will be rotated through weekely. The names list should match the numbers. If you only have a single number that needs to receive alerts then use the singular version of these settings and set UseWeeklySchedule to zero.
```
# Multiple people designated to receive messages
# on a weekly rotating call schedule
OnCallNumbers = 5555021212,5555021213,5555021214
OnCallNames = John,Jane,Joey

# for a single number designated to receive messages
OnCallNumber = 5555021212
OnCallName = John
```

# Using the script
To send a text message you just need to call for the script with a message argument:
```
./Zabbix-Twilio.pl "This is a test alert message"
```
The program will figure out what week of the year it is (1-52), get the phone number from the file and then use the supplied Twilio credentials to send the message.

When the script runs for the first time it will create a "Schedule" directory. In that directory are 52 files. Each file contains the phone number of the person who is on call for that week.  If you wanted to manually change the schedule all you would need to do is edit the file and change the phone number stored in the file. If you change the config file and want to refresh the schedule all you need to do is execute the script with the "UpdateSchedule" command:

```
./Zabbix-Twilio.pl UpdateSchedule
```
This will rework the schedule according to the new entry in the config file.

If you want to notify the person on call, execute the script with "notify" as the message. It will check to see if tomorrow is the day that starts the next week and then send a notification to that person that they are about to go on call.

```
./Zabbix-Twilio.pl Notify
```

*Note:* Depending on your OS you may need to:
1. Change the first line of the script from "#!/usr/bin/env perl" to "#!/usr/bin/perl". (The path with env is for windows).
2. Download and compile the latest version of perl. The Twilio module is itself dependendent on other modules, most would be considered standard but it's poissible they may not be included with your distribution. (The module uses SSL to acces the API. This means it needs Net::SSLeay, which in turn needs OpenSSL, for example).
3. If you get this error: "/usr/bin/perl^M: bad interpreter: No such file or directory" it means you have windows carriage returns instead of Unix carriage returns.
You may need to run the following command in the same directory with the script:
```
perl -i -pe 'y|\r||d' Zabbix-Twilio.pl
```
