## Ubuntu 14.04 amd64
### Core

    apt-get install python-software-properties software-properties-common
    add-apt-repository ppa:chris-lea/node.js
    apt-get update
    apt-get upgrade

    vi /etc/hostname
    vi /etc/hosts

    apt-get install mongodb=1:2.4.9-1ubuntu2
    apt-get install python g++ make nodejs git nginx redis-server ntp supervisor

    npm install coffee-script -g

    mongo

        use admin
        db.addUser({user: 'rpadmin', pwd: 'password', roles: ['readWriteAnyDatabase', 'userAdminAnyDatabase', 'dbAdminAnyDatabase', 'clusterAdmin']})
        use RootPanel
        db.addUser({user: 'rpadmin', pwd: 'password', roles: ['readWrite']})

    vi /etc/mongodb.conf

        auth = true
        noprealloc = true
        smallfiles = true
        
    vi /etc/redis/redis.conf
        
        requirepass password

    rm /etc/nginx/sites-enabled/default
    
    vi /etc/nginx/sites-enabled/rpadmin

        ssl_certificate /home/rpadmin/rpvhost.crt;
        ssl_certificate_key /home/rpadmin/keys/rpvhost.key;

        ssl_session_cache shared:SSL:10m;

        server {
            listen 80 default_server;
            listen 443 ssl default_server;
            listen [::]:80 default_server ipv6only=on;

            rewrite .* $scheme://rp.rpvhost.net/#redirect redirect;
        }

        server {
            listen 80;
            listen 443 ssl;

            server_name rp.rpvhost.net;

            location ~ /\.git {
                deny all;
            }

            location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass http://unix:/home/rpadmin/rootpanel.sock:/;
            }
        }

    useradd -m rpadmin
    usermod -G rpadmin -a www-data

    mkdir -m 750 /home/rpadmin/keys

    vi /etc/sudoers

        rpadmin ALL=(ALL) NOPASSWD: ALL

    vi /etc/rc.local

        ln -s /dev/xvda /dev/root
        iptables-restore < /etc/iptables.rules

    su rpadmin
    cd ~

    git clone -b stable https://github.com/jysperm/RootPanel.git
    cd RootPanel

    cp sample/core.config.coffee config.coffee

    npm install

    exit

    vi /etc/supervisor/conf.d/rpadmin.conf

        [program:RootPanel]
        command = coffee /home/rpadmin/RootPanel/app.coffee
        directory = /home/rpadmin/RootPanel
        autorestart = true
        redirect_stderr = true
        user = rpadmin

    service nginx restart
    service mongodb restart
    service redis-server restart
    service supervisor restart

### Plugins

    # Linux
    apt-get install quota quotatool

    vi /etc/fstab

        usrquota

    reboot

        quotacheck -am
        quotaon -au

    # Memcached

    apt-get install memcached

    # MySQL
    
    apt-get install mariadb-server
    
    mysql -u root -p
    
        GRANT ALL ON *.* TO 'rpadmin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
        
    # PHP-FPM
        
    apt-get install php5-fpm php-pear php5-readline php5-mysql php5-curl php5-gd php5-imap php5-mcrypt php5-memcache php5-tidy php5-xmlrpc php5-sqlite php5-mongo
    
    rm /etc/php5/fpm/pool.d/www.conf

    vi /etc/nginx/fastcgi_params
        
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    # ShadowSocks

    apt-get install python-pip python-m2crypto
    pip install shadowsocks

    mkdir /etc/shadowsocks

    vi /etc/default/supervisor

        ulimit -n 51200

    iptables -A OUTPUT -p tcp --dport 25 -d smtp.postmarkapp.com -j ACCEPT
    iptables -A OUTPUT -p tcp --dport 25 -j DROP
    iptables-save > /etc/iptables.rules

### Runtime

    # Shell
    apt-get install screen wget zip unzip iftop vim curl htop iptraf nethogs
    apt-get install libcurl4-openssl-dev axel unrar-free emacs subversion subversion-tools tmux mercurial postfix

    # Golang
    apt-get install golang golang-go.tools

    # Python
    apt-get install python python3 python-pip python3-pip python-dev python3-dev python-m2crypto
    pip install django tornado markdown python-memcached web.py mongo uwsgi virtualenv virtualenvwrapper flask gevent jinja2 requests MySQL-python

    # Node.js
    npm install forever gulp mocha harp bower -g
