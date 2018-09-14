# Rails-Delayed-Job-with-Monit-on-Linux

1.
- install monit:

```
apt install monit
```

2.
- since monit can't start daemon delayed job because unable to access ruby RVM environments, we need a scripts!
- first, write monitrc config on your application root directory. please note the "/etc/init.d/delayed_job"
- filename: delayed_job.monitrc

```
check process delayed_job
  with pidfile /usr/share/nginx/html/SaaS/auth_api/tmp/pids/delayed_job.pid every 1 cycles
  start program = "/bin/bash -c '/etc/init.d/delayed_job start'"
  stop program = "/bin/bash -c '/etc/init.d/delayed_job stop'"
  if mem is greater than 300.0 MB for 1 cycles then restart
  if cpu is greater than 80% for 3 cycles then restart
  group background
```

3.
- time for scripting!
- filename: delayed_job
- permissions: 755
- directory path: /etc/init.d/
- make sure other users can execute the script, to make sure: chmod o+x delayed_job
- necessary changes:
  a. change the set_path
  b. change the username in command "su - ubuntu -c"
  c. change the log path

```
  #! /bin/sh
  set_path="cd /usr/share/nginx/html/SaaS/auth_api"
  case "$1" in
    start)
      echo -n "Starting delayed_job: "
      su - ubuntu -c "$set_path; RAILS_ENV=production bin/delayed_job start" >> /usr/share/nginx/html/SaaS/auth_api/log/delayed_job.log 2>&1
      echo "done."     ;;
    stop)     echo -n "Stopping delayed_job: "
      su - ubuntu -c "$set_path; RAILS_ENV=production bin/delayed_job stop" >> /usr/share/nginx/html/SaaS/auth_api/log/delayed_job.log 2>&1
      echo "done."     ;;
    *)
      echo "Usage: $N {start|stop}" >&2
      exit 1
      ;;
  esac
  exit 0

```

- note the command "su - ubuntu -c"
- to allow switching to other user account without entering password, you'll need to add the following lines right below the pam_rootok.so line in your /etc/pam.d/su:

```
auth       [success=ignore default=1] pam_succeed_if.so user = ubuntu
auth       sufficient   pam_succeed_if.so use_uid user = ubuntu
```

4. 
- Done all the above?
- spawn the monit

```
sudo monit && sudo monit reload
```

- start delayed_job
```
sudo monit start delayed_job
```

############################################################################
TROUBLESHOOTING:
1. check monit log: "/var/log/monit.log"
2. check the script output log that you define in "/etc/init.d/delayed_job"
