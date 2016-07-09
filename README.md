# Prepare Environment
## install redis: 
```sh
sudo apt-get install redis
```

## install mysql and init tables: 
```sh
sudo apt-get install mysql-server
git clone https://github.com/open-falcon/scripts.git
cd ./scripts/
mysql -u root --password="$password" < db_schema/graph-db-schema.sql
mysql -u root --password="$password" < db_schema/dashboard-db-schema.sql
mysql -u root --password="$password" < db_schema/portal-db-schema.sql
mysql -u root --password="$password" < db_schema/links-db-schema.sql
mysql -u root --password="$password" < db_schema/uic-db-schema.sql

## optional: create user and grant privilege
```

## download falcon binary package
```sh
VERSION="v0.1.0"
wget https://github.com/open-falcon/of-release/releases/download/$VERSION/open-falcon-$VERSION.tar.gz -O /tmp/falcon-latest.tar.gz
```

## download falcon initd scripts
```sh
wget https://github.com/hfdiao/falcon-initd/archive/master.zip -O /tmp/falcon-initd.zip
```

# Main

```sh
## modify this array to modules wanted
modules=("agent" "transfer")

sudo su

# add user, unarchive all falcon modules
useradd falcon
mkdir -p /home/falcon/tmp/
cp /tmp/falcon-latest.tar.gz /home/falcon/
chown -R falcon:falcon /home/falcon/

su falcon
cd /home/falcon/
tar -zxf falcon-latest.tar.gz -C ./tmp/
for x in `find ./tmp/ -name "*.tar.gz"`;do \
    app=`echo $x|cut -d '-' -f2`; \
    mkdir -p $app/var; \
    tar -zxf $x -C $app; \
done
rm -fr ./tmp && rm -fr falcon-latest.tar.gz

# TODO: modify module configuration files
# module written in go: 
#   1. cd $module
#   2. modify cfg.example.json
#   3. mv cfg.example.json cfg.json
# module written in python:
#   1. cd $module
#   2. modify */config.py

# install initd scripts, start modules when system boots
cd /tmp/ && unzip falcon-initd.zip
for module in "${modules[@]}"
do
    cp /tmp/falcon-initd-master/falcon-$module /etc/init.d/ && chmod +x /etc/init.d/falcon-$module
    update-rc.d falcon-$module defaults
done
```
