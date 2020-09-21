# Unpacking Chef Automate AIB's and Habitat .hart files. 
  
  
### Step 1. Extract the Chef Automate AIB
  
```bash
tail -c +8 automate-20200908235050.aib | tar -t
```
  
  
This will leave you with about 190 Habitat `.hart` files. Your output will look like this:  
```bash
$ tail -c +8 ../automate-20200908235050.aib | tar -t
bin
bin/hab
hab
hab/cache
hab/cache/artifacts
hab/cache/artifacts/chef-applications-service-1.0.0-20200908204810-x86_64-linux.hart
hab/cache/artifacts/chef-authn-service-0.1.0-20200908204817-x86_64-linux.hart
hab/cache/artifacts/chef-authz-service-0.1.0-20200908223357-x86_64-linux.hart
hab/cache/artifacts/chef-automate-builder-api-0.1.0-20200908204559-x86_64-linux.hart
hab/cache/artifacts/chef-automate-builder-api-proxy-0.1.0-20200908204559-x86_64-linux.hart
hab/cache/artifacts/chef-automate-builder-memcached-1.5.19-20200813174803-x86_64-linux.hart
hab/cache/artifacts/chef-automate-cds-0.1.0-20200908204559-x86_64-linux.hart
hab/cache/artifacts/chef-automate-cli-0.1.0-20200908223358-x86_64-linux.hart
hab/cache/artifacts/chef-automate-compliance-profiles-1.0.0-20200904151016-x86_64-linux.hart
hab/cache/artifacts/chef-automate-cs-bookshelf-14.0.25-20200908204717-x86_64-linux.hart
...
...
...
hab/cache/artifacts/core-tar-1.32-20200305233447-x86_64-linux.hart
hab/cache/artifacts/core-tzdata-2018g-20200403124218-x86_64-linux.hart
hab/cache/artifacts/core-util-linux-2.34-20200306003119-x86_64-linux.hart
hab/cache/artifacts/core-xlib-1.6.5-20200404200238-x86_64-linux.hart
hab/cache/artifacts/core-xz-5.2.4-20200306001321-x86_64-linux.hart
hab/cache/artifacts/core-zeromq-4.3.1-20200319192759-x86_64-linux.hart
hab/cache/artifacts/core-zlib-1.2.11-20190115003728-x86_64-linux.hart
hab/cache/artifacts/core-zlib-1.2.11-20200305174519-x86_64-linux.hart
hab/cache/artifacts/habitat-builder-api-9034-20200827185135-x86_64-linux.hart
hab/cache/artifacts/habitat-builder-api-proxy-8997-20200812161534-x86_64-linux.hart
hab/cache/keys
hab/cache/keys/chef-20160614114050.pub
hab/cache/keys/core-20180119235000.pub
hab/cache/keys/core-20200219002627.pub
hab/cache/keys/core-20200331193006.pub
hab/cache/keys/habitat-20180129222536.pub
manifest.json

```
   
  
### Step 2. Extract each of the Chef Habitat `.hart` files
  
```bash  
tail -n +6 /tmp/foo-bar.hart | xzcat | tar xf - -C .
```
   
You will need to loop through all the .hart files and they will be extracted into their respective folders.
  
  
