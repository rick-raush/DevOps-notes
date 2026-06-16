sudo cat /etc/squid/squid.conf  
#  
\# Ansible managed  
#

http_port 3128

cache_effective_user squid  
cache_effective_group squid

#Log format  
logformat oauth-prod-format %6tr %>a %Ss/%03>Hs %<st %rm %ru %\[un %Sh/%<a %mt  
logformat combinedjson { "clientip": "%>a" , "timestamp": "%{%FT%T%z}tg" , "verb": "%rm" , "request": "%ru", "httpversion": "HTTP/%rv", "response": "%>Hs", "bytes": "%&lt;st" , "referer": "%{Referer}&gt;h", "user_agent": "%{User-Agent}>h" , "request_status": "%Ss", "hierarchy_status": "%Sh" }  
access_log /var/log/squid/access.log combinedjson

&nbsp;

pipeline_prefetch on  
shutdown_lifetime 1 second

cache_dir ufs /var/spool/squid 100 16 256

coredump_dir /var/spool/squid

#refresh_pattern ^ftp: 1440 20% 10080  
#refresh_pattern ^gopher: 1440 0% 1440  
#refresh_pattern -i (/cgi-bin/|\\?) 0 0% 0  
#refresh_pattern . 0 20% 4320

&nbsp;

httpd_suppress_version_string on

\# Access configuration  
acl localnet src 10.241.3.10/32 # RFC 1122 "this" network (LAN)  
#acl localnet src 10.0.0.0/8 # RFC 1918 local private network (LAN)  
#acl localnet src 100.64.0.0/10 # RFC 6598 shared address space (CGN)

#acl localnet src 169.254.0.0/16 # RFC 3927 link-local (directly plugged) machines  
#acl localnet src 172.16.0.0/12 # RFC 1918 local private network (LAN)  
#acl localnet src 192.168.0.0/16 # RFC 1918 local private network (LAN)  
acl oauth-prod src 10.241.0.0/18  
acl oauth-common src 10.178.0.0/16  
acl all src 0.0.0.0/0.0.0.0  
no_cache deny all

acl SSL_ports port 443  
acl Safe_ports port 80 # http  
#acl Safe_ports port 21 # ftp  
acl Safe_ports port 443 # https  
#acl Safe_ports port 70 # gopher  
#acl Safe_ports port 210 # wais  
#acl Safe_ports port 1025-65535 # unregistered ports  
#acl Safe_ports port 280 # http-mgmt  
#acl Safe_ports port 488 # gss-http  
#acl Safe_ports port 591 # filemaker  
#acl Safe_ports port 777 # multiling http  
acl CONNECT method CONNECT

http_access deny !Safe_ports  
http_access deny CONNECT !SSL_ports

http_access allow localhost manager  
http_access deny manager

http_access allow oauth-prod  
http_access allow oauth-common  
http_access deny all

memory_pools off

quick_abort_min 0 KB  
quick_abort_max 0 KB

client_db off  
log_icp_queries off  
buffered_logs on  
half_closed_clients off

\# Cache configuration  
maximum_object_size 1024 MB  
cache_replacement_policy heap LFUDA  
cache_replacement_policy heap GDSF

\# DNF repositories mirrors StoreIDs  
#store_id_program /usr/lib64/squid/storeid_file_rewrite /etc/squid/dnf_mirrors

\# Refresh patterns : RPM packages  
#refresh_pattern -i \\.(rpm|drpm|srpm)\$ 43200 80% 129600 override-expire override-lastmod ignore-reload ignore-no-store reload-into-ims  
#refresh_pattern /repodata/repomd\\.xml\$ 0 0% 0  
#refresh_pattern /repodata/\[^/\]+\$ 1440 80% 43200 override-expire override-lastmod ignore-reload ignore-no-store reload-into-ims

&nbsp;

\# Refresh patterns : Defaults  
#refresh_pattern ^ftp: 1440 20% 10080  
#refresh_pattern ^gopher: 1440 0% 1440  
#refresh_pattern -i (/cgi-bin/|\\?) 0 0% 0  
#refresh_pattern .  
\[raushan_T4468@ip-10-14-28-17 ~\]\$  
\[raushan_T4468@ip-10-14-28-17 ~\]\$