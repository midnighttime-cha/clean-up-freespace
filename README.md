# Clean up Freespace

## Ubuntu
```sh
#!/bin/sh
#Check the Drive Space Used by Cached Files
du -sh /var/cache/apt/archives 

#Clean all the log file
#for logs in `find /var/log -type f`; do > $logs; done
logs=`find /var/log -type f`

for i in $logs
do
> $i
done

#Getting rid of partial packages
apt-get clean && apt-get autoclean 
apt-get remove --purge -y software-properties-common 


#Getting rid of no longer required packages
apt-get autoremove -y 

#Getting rid of orphaned packages
deborphan | xargs sudo apt-get -y remove --purge 

#Free up space by clean out the cached packages
apt-get clean 

# Remove the Trash
rm -rf /home/*/.local/share/Trash/*/**
rm -rf /root/.local/share/Trash/*/**

# Remove Man
rm -rf /usr/share/man/??
rm -rf /usr/share/man/??_*

#Delete all .gz and rotated file
find /var/log -type f -regex ".*\.gz$" | xargs rm -Rf 
find /var/log -type f -regex ".*\.[0-9]$" | xargs rm -Rf 

#Cleaning the old kernels
dpkg-query -l|grep linux-im*
#dpkg-query -l |grep linux-im*|awk '{print $2}'
apt-get purge $(dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | head -n -1) --assume-yes 
apt-get install linux-headers-`uname -r|cut -d'-' -f3`-`uname -r|cut -d'-' -f4`

#Cleaning is completed
echo "Cleaning is completed"
```

## Redhat 6,7,8
```sh
#!/bin/sh

echo "Trim log files"
find /var -name "*.log" \( \( -size +50M -mtime +7 \) -o -mtime +30 \) -exec truncate {} --size 0 \;
echo "--------------------------------------------------------"

echo "Cleanup YUM cache"
yum clean all
rm -rf /var/cache/yum
rm -rf /var/tmp/yum-*
echo "--------------------------------------------------------"

echo "Remove orphan packages"
package-cleanup --quiet --leaves | xargs yum remove -y
echo "--------------------------------------------------------"

echo "Remove WP CLI cached WordPress downloads"
rm -rf /root/.wp-cli/cache/*
rm -rf /home/*/.wp-cli/cache/*
echo "--------------------------------------------------------"

echo "Remove old kernels"
(( $(rpm -E %{rhel}) >= 8 )) && dnf remove $(dnf repoquery --installonly --latest-limit=-2 -q)
(( $(rpm -E %{rhel}) <= 7 )) && package-cleanup --oldkernels --count=2
echo "--------------------------------------------------------"

echo "Remove Composer cache"
rm -rf /root/.composer/cache
rm -rf /home/*/.composer/cache
echo "--------------------------------------------------------"

echo "Remove core dumps"
find -regex ".*/core\.[0-9]+$" -delete
echo "--------------------------------------------------------"

echo "Remove error_log files (cPanel)"
find /home/*/public_html/ -name error_log -delete
echo "--------------------------------------------------------"

echo "Remove Node.js caches"
rm -rf /root/.npm /home/*/.npm /root/.node-gyp /home/*/.node-gyp /tmp/npm-*
echo "--------------------------------------------------------"

echo "Remove Mock caches"
rm -rf /var/cache/mock/* /var/lib/mock/*
echo "--------------------------------------------------------"

echo "Clear generic program caches"
rm -rf /home/*/.cache/*/* /root/.cache/*/*
echo "--------------------------------------------------------"

echo "--------------- Clean UP Successfully ---------------------"
```
