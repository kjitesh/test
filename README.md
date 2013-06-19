Talent Web
================================

This is guide for setting up Talent Web and Web2py.
We recommend that you follow these steps in the following order.

NOTE: This guide was setup for Mac and Linux machines. If you're on windows, good luck...

Just install pythonbrew, pythonbrew's virtualenv (or you can install virtualenv seperately), setup a talent_web virtualenv,
and activate the talent_web virtualenv whenever you run web2py (or run python at all). The point of virtualenv is to load all
required modules for web2py and Talent Web.


### Command Cheatsheet

NOTE: THIS IS A CHEATSHEET. DO NOT USE THIS IF YOU HAVEN'T INSTALLED ANYTHING. THIS IS JUST A REFERENCE

Activate your talent_web python environment

    pythonbrew venv use talent_web

Install missing modules

    ./talent_web modules install

Sync your modules to Talent Web project

    #Remember to activate your talent_web python environment
    #before installing any python modules
    pythonbrew venv use talent_web

    #this is an example module that you would install
    pip install catflip

    #after you're done, run this
    ./talent_web modules update

Run web2py and Talent Web

    #Remember to cd to the web2py folder
    #example:
    cd ~/web2py
    ./talent_web run <password>

Push to dev (and sync modules)

    ./talent_web push dev


Installation
-------------------------

Reminder: If any of the following steps failed, please refer to the Troubleshooting section below.

### Installing pythonbrew

Run the following commands in your console:

    curl -kL http://xrl.us/pythonbrewinstall | bash

    #After the command above is successful, run this one below
    echo '[[ -s "$HOME/.pythonbrew/etc/bashrc" ]] && source "$HOME/.pythonbrew/etc/bashrc"' >> ~/.bashrc

    #If you haven't setup bashrc to autoload on Mac, run the command below:
    echo 'source ~/.bashrc' >> ~/.bash_profile


Note: be sure to have gcc (or a C/C++ compiler on your machine)

    gcc -v

If nothing shows up, install a gcc compiler (On Mac, you need Command Line Tools from the Mac Developer site)

Restart your console, then run the following:

    pythonbrew install 2.7.2
    pythonbrew switch 2.7.2

    pythonbrew venv init
    pythonbrew venv create talent_web -n
    pythonbrew venv use talent_web

    #You need to install the new version of pip
    cd /tmp
    curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
    python get-pip.py

Note: If you're going to install new python modules or do anything in python related to Talent Web,
remember to activate the talent_web python environment

    pythonbrew venv use talent_web


### Installing Web2py

Git Clone this Github repository in the folder "web2py"

    #This is an example directory
    cd ~/
    git clone git@github.com:Veechi/web2py.git web2py
    cd web2py

    #Use the local routes.py version
    mv routes.py.local_example routes.py


### Installing Talent Web

cd into the applications folder under the web2py directory (where you installed web2py in the previous step)

    #This is an example directory
    cd ~/web2py/applications/

Git Clone the Talent-Web repository in the folder "web"

    git clone git@github.com:Veechi/Talent-Web.git web

If you're using the github app, make sure that this project's contents is in a folder called "web",
and that it's inside the applications folder of web2py.

### Installing python modules required for Talent Web

cd back into the "web2py" folder, then run the following commands:

    #Remember to activate your talent_web python environment
    pythonbrew venv use talent_web

    #This is an example directory
    cd ~/web2py

    chmod a+x talent_web
    ./talent_web modules install

You should now have all the required python modules installed for Talent Web




Development environment and deploying
-------------------------

### Setting up your SSH tunnel

To connect to the dev DB, you need to first setup your SSH tunnel user.

First, create a seperate pair of RSA public and private keys. **Use NO passphrase.**

    #Generate your new public and private keys in your ssh folder
    cd ~/.ssh
    ssh-keygen -f tunnel_user

Then, copy over your new tunney_key.pub over to the tunneling server's authorized keys:

    cat ~/.ssh/tunnel_user.pub | ssh tunnel_user@184.72.48.47 "cat >> ~/.ssh/authorized_keys"
    #When prompted, the password is s!tunnel

SSH Password is **s!tunnel**

After you've run the command above, test your ssh connection:

    ssh -i ~/.ssh/tunnel_user tunnel_user@184.72.48.47

Once you confirm that it works, press CTRL+D
**Note:** It shouldn't ask for your password. If it does, your tunnel_user private key doesn't exist in your ~/.ssh folder


### How to run Talent Web and Web2py

Inside the "web2py" folder, run:

    ./talent_web run <password>

To exit, press CTRL + C. This will close web2py and shutdown the SSH tunnel that opens for connections to the db.

The command above (./talent_web run) does the following for you:
* Activates your talent_web virtual env
* Installs any missing python modules in your talent_web virtual env
* Sets up an SSH tunnel for your DB connection
* Runs web2py with a specified password (123 by default)
* Closes your SSH tunnel if you quit web2py

To run Talent Web in console mode:

    ./talent_web run console

### Adding/Syncing new modules to the Talent Web project

Again, remember to activate the talent_web python environment before installing new modules with the pip command:

    pythonbrew venv use talent_web

Install your python modules...

    pip install catflip-py

When you're finished working and you're ready to sync up your new modules with the Talent Web project:

    ./talent_web modules update

This will sync up any new modules in your talent_web python environment, and add them into python-module-requirements.txt if they are missing



### Server-side deployment

#### Upgrading python on a server

    #We need the shared version of python in order to use with mod_wsgi
    pythonbrew install --configure="--enable-shared" 2.7.2

    #change your LD config to use the new python lib folder
    echo 'export LD_LIBRARY_PATH=~/.pythonbrew/pythons/Python-2.7.2/lib' >> ~/.bashrc
    echo 'export LD_RUN_PATH=~/.pythonbrew/pythons/Python-2.7.2/lib' >> ~/.bashrc
    source ~/.bashrc
    echo 'include /home/ubuntu/.pythonbrew/pythons/Python-2.7.2/lib' >> /etc/ld.so.conf
    sudo ldconfig

    #install apache tools for building mod_wsgi
    sudo apt-get install apache2-dev

    #install modules again
    pythonbrew venv init
    pythonbrew venv create talent_web -n
    pythonbrew venv use talent_web

    #You need to install the new version of pip
    cd /tmp
    curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
    python get-pip.py

    cd /home/www-data/
    git clone git@github.com:Veechi/web2py.git web2py
    cd web2py

    #Use the server routes.py version
    mv routes.py.server_example routes.py
    touch parameters_433.py

    #install talent web app
    cd applications/
    git clone git@github.com:Veechi/Talent-Web.git web

    #install required python modules
    cd ..
    chmod a+x talent_web
    pythonbrew venv use talent_web
    ./talent_web modules install

    #change permissions of entire web2py folder
    cd ..
    sudo chown -R www-data:www-data web2py
    sudo chmod 2775 web2py
    find web2py -type d -exec sudo chmod 2775 {} +
    find web2py -type f -exec sudo chmod 0664 {} +

    #change/set your WSGIPythonHome to use talent_web virtualenv
    #add or change the line WSGIPythonHome to:
    #WSGIPythonHome /home/ubuntu/.pythonbrew/venvs/Python-2.7.2/talent_web/

    sudo nano /etc/apache2/httpd.conf
    WSGIPythonHome /home/ubuntu/.pythonbrew/venvs/Python-2.7.2/talent_web/

    #Download the recommended version of mod_wsgi
    #3.4 was the recommended version when this document was created - it might be different today
    cd /tmp
    wget https://modwsgi.googlecode.com/files/mod_wsgi-3.4.tar.gz
    tar xvfz mod_wsgi-3.4.tar.gz
    cd mod_wsgi-3.4/
    sudo make distclean
    pythonbrew use 2.7.2

    ./configure
    #IMPORTANT: make sure that the output of ./configure has the following line:
    #checking for python... /home/ubuntu/.pythonbrew/venvs/Python-2.7.2/talent_web/bin/python
    #mod_wsgi needs to be compiled with pythonbrew's 2.7.2

    make
    sudo make install

    #start apache2, make sure that the app works
    sudo service apache2 start

#### Pushing a branch to a server

    ./talent_web push <server> <branch>

List of servers:
* dev
* prod
* staging

If branch is left blank, it'll use git fetch, which updates all the branches on the server, and doesn't change the branch that the server uses.

If branch is specified, it'll git fetch all the branches, and checkout the specified branch on the server.

The command above will also automatically sync any new modules added to the Talent Web project, and restarts apache2.

Note: only certain users may push to the server (@noobcola and @oamasood)


Workflow
-------------------------
* Master topic-branch
* Pull requests


