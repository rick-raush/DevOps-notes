Path - /etc/logrotate.d  
<br/>

/var/log/nginx/\*.log {  
        rotate 20  
        maxsize 500M  
        daily  
        missingok  
        notifempty  
        dateext  
        dateformat -%Y%m%d-%s  
        compress  
        delaycompress  
        create 0644 nginx adm  
        sharedscripts  
        postrotate  
                \[ -s /run/nginx.pid \] && kill -USR1 \`cat /run/nginx.pid\`

&nbsp;       endscript  
}  
<br/><br/>

/app/batch10/webapps/logs/\*.log {  
    size 500M  
    rotate 10  
    compress  
    delaycompress  
    missingok  
    notifempty  
    copytruncate  
    dateext  
    dateformat -%Y%m%d-%s  
    olddir /app/batch10/webapps/logs/archived  
    create 0644 root root  
}