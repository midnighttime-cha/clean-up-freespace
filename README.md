# Clean up Freespace

## Ubuntu

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
