## 1. Menu
1. Menu
2. Requirements
3. Installation process
4. Useful commands
5. System Setup
6. Internal global DB structure
7. CMS Usage (Not applicable for StudieTimer)

## 2. Requirements

The Ikibox Content Management System has the following, recommended, requirements to insure a stable and functioning service.
1. Ubuntu (or any other Linux distribution) though it SHOULD be 64-bit!
2. Ruby 1.9.3p125
3. MonogDB v1.8.2
4. Nginx or Apache (no particular version needed, we recommend Nginx for production)
5. Redis server v2.2.11

## 3. Installation
This guide will ensure a running environment. However it won't get into 
deployment strategies (like Capistrano and GIT setup). The following commands
can be extremely dangerous, Website-development and Briljandt are not responsible
for any damage the following commands could do to your system!

### 3.1 Update Ubuntu
Update and upgrade Ubuntu (probably with sudo, depending on settings and Linux distribution).
    
    apt-get update
    apt-get -y upgrade

Install the build-essentials (probably not installed yet)
    
    apt-get -y install build-essential

Install Git-core.
    
    apt-get -y install git-core

### 3.2 Rbenv
Website-development has decided to switch from RVM to Rbenv for 
management of different Ruby versions and gems. The following installation
commands are for a system wide installation of Rbenv. 

Download the Rbenv from Github.
    
    git clone git://github.com/sstephenson/rbenv.git /usr/local/rbenv

Now you need to add Rbenv to the path.
First create a profile and add RBenv.
    
    echo '# rbenv setup' > /etc/profile.d/rbenv.sh
    echo 'export RBENV_ROOT=/usr/local/rbenv' >> /etc/profile.d/rbenv.sh
    echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> /etc/profile.d/rbenv.sh
    echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh

Chmod the just created file.
    
    chmod +x /etc/profile.d/rbenv.sh

And reload.
    
    source /etc/profile.d/rbenv.sh

For easy installation en compilation of different Ruby languages instal Ruby-build:
    
    pushd /tmp
      git clone git://github.com/sstephenson/ruby-build.git
      cd ruby-build
      ./install.sh
    popd
    
Check if Rbenv is installed:
    
    rbenv --version

### 3.3 Ruby
To install Ruby use the following commands.
    
    rbenv install 1.9.3-p125

Make it the default Ruby version (just to make sure it's working in the future)
    
    rbenv global 1.9.3-p125

And rehash to make Ruby work in the current session.
    
    rbenv rehash

### 3.4 Install Nginx and Passenger
First make sure that Nginx is not installed (Passenger will install it when it is done). Note 
that Nginx does not support loadable modules, in contrast to most other web servers. This is why it is 
essential you know what you are doing!

Install Passenger.
    
    gem install passenger

Install the Nginx module for Passenger. And follow the installer (there are probably some 
things still missing)
    
    passenger-install-nginx-module

Edit the Nginx config file (if you don't know VIM, LEARN IT).
    
    vim /opt/nginx/conf/nginx.conf

Replace it with the following, of course edit the file to make it work (change the debug, access, error and root locations).
    
    #user  nobody; 
    worker_processes  1; 
     
    events { 
        worker_connections  1024; 
    } 
     
     
    http { 
        passenger_root /usr/local/rvm/gems/ruby-1.9.3-p125/gems/passenger-3.0.11; 
        passenger_ruby /usr/local/rvm/bin/passenger_ruby; 
     
        include       mime.types; 
        default_type  application/octet-stream; 
     
        passenger_max_pool_size 5; 
        passenger_min_instances 4; 
        passenger_pool_idle_time 0; 
        passenger_log_level 1; 
        passenger_debug_log_file [DEBUG LOG LOCATION]
     
        sendfile        on; 
        tcp_nopush      on; 
        tcp_nodelay     on; 
        keepalive_timeout 65; 
        client_max_body_size 1G; 
        #gzip  on; 
     
        server { 
            listen       *:80; 
            passenger_use_global_queue on; 
     
            #access_log [ACCES LOG LOCATION]
            #error_log [ERRORS LOG LOCATION]
     
            root   [ROOT OF THE APPLICATION]
     
            passenger_enabled on; 
     
        } 
        
start nginx
    
    service nginx start



### 3.5 require gems
Installing all the required gems is a breeze. 
First install Bundler
    
    gem install bundler

Second install the gems:
    
    bundle install


### 3.6 Mongodb
Installing mongoDB (probably sudo required).
    
    apt-get install mongodb

Now we need to restore the database.
First install tar.
    
    apt-get install tar

Untar db files.
    
    tar -xvjf [tar location] 

And restore the files:
    
    mongorestore -d ikibox [Location unzipped DB files]
    

Start the mongodb server:
    
    sudo mongod --fork
                --dbpath  [Location DB files]
                --logpath [Location DB log]

### 3.7 Redis
Redis is used to track background tasks that are issued by the system. 
Install.
    
    apt-get install redis-server

start.
    
    service start redis

### 3.9 ImageMagick

Install libmagick.
    
    apt-get install libmagick9-dev


### 3.10 Wkhtmltopdf
Although WkhtmltoPDF is, probably, not required. It can 
be a pain to install! 

Install lzma
    
    apt-get install lzma

cd to current ruby bin.
    
    cd /usr/local/rvm/gems/ruby-1.9.3-p125/bin/

Download wkhtmltopdf.
    
    wget http://wkhtmltopdf.googlecode.com/files/wkhtmltopdf-0.10.0_beta5-static-amd64.tar.lzma

Test it.
    
    which wkhtmltopdf


## 4. Useful commands

Starting a production console (from the location of the ikibox root):
    
    RACK_ENV=production bundle exec tux

## 5. System setup

The Ikibox CMS is based on the MVC pattern (like Rails for example), it is however
totally build with Sinatra. The system is build in four distinct parts:

### 5.1 Websites Controller
The websites controller is in charge of serving the actual webpages for the virtual stored 
websites. Virtual trees are created to calculate the most efficient way to different parts in the 
website (A LOT of tree traversing is done). 

### 5.2 CMS Controller (Not applicable for StudieTimer)
The CMS controller is in charge of any request related to CRUD (For
good reasons REST is not implemented).

### 5.3 Code Controller
Due to the implementation of a virtual filesystem and not completed FTP interface, the code controller was 
created as a temporary solution to make CRUD actions possible on files. This is the only way to edit the HTML 
and Ruby files.

### 5.4 Admin Controller
The admin controller is the interface to create website and manage the virtual Models, attributes, domains etc. 


## 6. Internal global DB structure
The following, flexible, database structure is created to be a dynamic environment which could (theoretically) be 
able to handle any type of data structure.

The System has the following models
1. Assets (every file uploaded in the system has gets an asset record assigned to them. When the record of the asset is
destroyed, so will the associated file).
2. Clients (Clients are models which are just for keeping the system organized (especially when there are 30 or more clients, of 
course Clients can be associated with other clients and Websites)
3. Websites (Websites are the entering point for requests. Websites can have multiple Domains and Slugs associated with them). 
4. Documents (Documents are "virtual" models. Documents should always be associated with clients or websites)
5. Records (Records are the actual instance of documents (or virtual models as you will). These are the most important pieces in the 
system.).
6. Groups and Users (The Ikibox system has a RBAC system for user authentication. Rights can be associated to groups, users, documents, 
websites and any type of "primary" model (listed above).)

To allow the system to be as flexible as possible some primary models have relations with non primary models. These are models 
who do not get a BSON::ObjectId associated with them and can't be linked to outside the scope of the primary instances of the models. 
Websites:
7. Slugs (Every website, as a default, doesn't allow any url to pass. Every url should be saved as a slug, the website in turn will allow request
those requests to pass).
8. Domains are associated with websites. (To allow traffic to specific websites, the system needs to know which websites are responsible for which 
domains)

Documents:
9. Keys (Are the keys for the virtual model, when keys are associated with a documents associations of that documents will inherit methods with the name
of the key)
10. Validations (Validations are the specific rules the value of keys should pass before they are saved to the database)

All Primary models (Inherited through the IKI model)
11. Rights (To allow the most flexible RBAC system every primary model has the ability to be associated with groups or users and assigned a 
specific permission)
12. Connections (Connections are the most important part of the system. Every primary model has the ability to be associate to other instance of 
primary models. With this solution very complex tree structures can be created.)
13. Filters (are not used)


## 7. CMS Usage (Not applicable for StudieTimer)

The CMS methods are designed to make a flexible CMS possible:

Add:
    
    add [[Model name], [Position in the tree], [Section name / and file name], [multiple files for rendering]]

Edit
    
    edit [record id]

To make records sortable, two settings are required. First in the wrapper.
    
    sortable=[section name]

Second, for the records the should be sortable.
    
    sort=[record id]

Sections are methods that render all child records in the tree with a specific section name
    
    Section [section name], [parent position]

Layouts can be used to render the current selected record in the tree (again for a specific section name)
    
    layout [section name], [current position]

Partials can be used to render specific files:
    
    partial  [file name], [optional document name]
