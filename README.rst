
===============
sudo for cygwin
===============

What is this?
-------------

Emurates Unix sudo in cygwin.

You can use this like::

    $ sudo vim /etc/hosts
    $ sudo cp foo.txt /cygdrive/c/Program Files/
    $ suco cygstart cmd # open elevated standard command prompt
    $ sudo cygstart regedit
    $ sudo # just invoke elevated shell

This might be handy if you are running cygwin on Vista or above with UAC. With this program, you can run processes as an administator, from normal, non-elevated cygwin shell.


Caution
-------------------------

UAC Elevation is usually done through UI prompt for good reasons.
This program can run elevated process without a UI prompt that does not
go along well with Cygwin shell environment.
However, it also means that you are weakening the system in terms of security.

How it works
------------

This is in fact a client/server application.

It looks as if the child process is running in the current terminal.
However, in fact, it's invoked by the server, and running remotely
(though "remote" is in the same PC).

You must launch a python script named ``sudoserver.py`` beforehand,
in desired privileges. If you want this to function like "Run as administrator",
just run ``sudoserver`` as administrator.
For this purpose, Windows' built-in Task Scheduler is handy.

``sudoserver.py`` listens on ``127.0.0.1:7070`` by default, 
then sits and wait for connections from ``sudo``.

``sudo``, when invoked, connects to the ``sudoserver``.
Then it sends it's command line arguments, environment variables,
current working directory, and terminal window size, to the ``sudoserver``.

When ``sudoserver`` accepts connection from ``sudo``, ``sudoserver`` forks a child process with pty, set up environments, current working directory or something, then execute the process.

The child process is spawned by the ``sudoserver``, therefore it runs in the privileges same as the server.

And, as the child process runs in a pty, it *acts* as if running in ordinary terminals. Therefore you can run cygwin's interactive console-based program like vim or less.

After execution, ``sudo`` and ``sudoserver`` bridges user's tty and the process I/O.

Requirements
------------

Both ``sudo`` and ``sudoserver.py`` is written in python2, therefore you need to install Python2.

Also, you need Python modules ``greenlet``, and ``eventlet``. These are not packaged in cygwin, therefore you must manually install them. ``python-devel`` is also required for building ``greenlet``.

How to setup
------------

#. Install ``python2``, ``python2-devel``, ``python2-pip`` with cygwin installer.

#. To install dependencies, type the following in the cygwin shell:

    $ pip2 install setuptools eventlet greenlet

#. Place ``sudo`` and ``sudoserver.py`` somewhere in your PATH.


Testing
------------

#. From cygwin shell, invoke ``sudoserver.py`` with::

    $ /path/to/sudoserver.py

#. To see all available arguments to ``sudoserver.py``, run::
    
    $ /path/to/sudoserver.py -h

#. Test sudo command with::

    $ sudo ls -l

Start sudo server at log in
---------------------------

#. If everything seems to work, you can register sudoserver.py to the Windows task scheduler:

   - Action: "Start a program"
   - Triggers: "At log on"
   - "Run with highest privileges": checked.
   - "Run only when user is logged on": checked.
   - "Program/script": output of ``cygpath -w "$(realpath which python2)`` 
   - "Add arguments(optional)": output of ``realpath /path/to/sudoserver.py`` and any additional arguments ( ``-nw`` is suggested )

Changing the TCP port
---------------------

#. Run sudoserver.py with argument ``-p PORT``
#. Set environment variable ``CYGWIN_SUDO_SERVER_PORT`` to the port for ``sudo``

Notes
-----

The ``sudoserver`` logs all connections/commands. However, ith argument "-nw" is specified, ``sudoserver`` hides it's console window.

``sudoserver`` sets an aditional environment variable "ELEVATED_SHELL" when spawing child processes. You can use this variable for changing your elevated shell prompt (PS1), to see which environment you are in. For example, you can put the following in your .bashrc::

    case $ELEVATED_SHELL in
    1) PS1='\[\033[31m\][\u@\h]#\[\033[0m\] ';;   # elevated
    *) PS1='\[\033[32m\][\u@\h]$\[\033[0m\] ';;
    esac

