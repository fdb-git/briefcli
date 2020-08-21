![BriefCLI logo](../assets/logo/briefcli.png?raw=true)

# Brief (Bash) CLI builder

__```briefcli```__ is a minimal framework to build your command line interfaces for whatever you want.

Example use cases:
- a sysadmin want to give its users a homogeneous CLI to multiple services in its network;
- a developer need to write a CLI tool without rewrite parameters and config file logic.

## Features
- Easy command definition interface with only 2 functions ```usage()``` that print usage info and ```call()``` that execute the real command:

```bash
function usage {
    echo 'Print your public ip address.'
}

function call {
    curl "ifconfig.io" # We are not related to ifconfig.io, it's only a simple example.
}
```

- Parameters can be defined as _key=value_ pairs through:
  - config file
  - environment variables (removing the initial string defaulting to "BC_") 
  - command line parameters
  - standard input

## Limitations
- No POSIX Argument Syntax Parsing, parameters are only _key=value_ pairs
- No nested subcommands
- No autocompletion

 If needed argument parsing or nested subcommand can be managed manually in ```call()``` function.

# Workflow to build a CLI with _briefcli_
1) download _briefcli_
1) create command files
1) set default values in config files
1) deploy _briefcli_ files and create a link with the new CLI name

## Download or update _briefcli_

```bash
INSTALL_DIR=/opt # or wherever you want
cd $INSTALL_DIR
git clone https://github.com/fabiodblab/briefcli.git
# alternatively you can update with:
git pull
```

Try _briefcli_:

    cd briefcli
    ./briefcli print-conf

## Configuration
_briefcli_ accept parameter from multiple sources and aggregates them into one  associative array that can be used later in command files.

The parameters are read in the following order, in case of homonymy the next one overwrites the previous one:
- config file
- environment variables (only those starting with ENVPREFIX config paramenter) 
- command line parameters
- standard input

As an example, a parameter _password_ defined in the _config_ file is not an excellent choice, so it's better to pass it as an environment variable:

    BC_user='john' BC_password='123456' ./briefcli print-conf

We can use command line parameters, but for a password also this is not a good choice because the command line is visible in process list:
 
    ./briefcli print-conf user='john' password='123456'

The last and more secure for pass secret parameters is via stdin (pay attention to insert _newline_ between parameters):

    echo -e "user='john'\npassword='123456'" | ./briefcli print-conf

Finally we can use mixed sources:

    echo 'user1=from_stdin' | BC_user2='from_env' ./briefcli print-conf user3='from_parameter'

## Create a command files

All commands resides in _commands_ directory and are defined by one file with 2 function inside: ```call()``` and ```help()```.

To start easily copy the __skel_ command as new file and start modifing it.

### The call() function
It is executed by _briefcli_ on every run and all configuration parameters can be found in the [associative array](https://www.gnu.org/software/bash/manual/html_node/Arrays.html) CONF:

```bash
function call {
    ${CONF[curl]} "${CONF[endpoint]}/" | ${CONF[w3m]}
}
```

### The help() function
It is used by _help_ command to print user documentation. The CONF array is also availabe here.
```bash
function help {
    echo 'Return a string with usage text'
}
```
# Deploy 
Create the link with your CLI name:

    ln -s $INSTALL_DIR/briefcli/briefcli /usr/local/bin/mycli

Now if the directory in which the link is contained is included in the path then you can run:

    mycli




