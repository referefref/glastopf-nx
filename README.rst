Adapted from https://github.com/mushorg/glastopf by Lukas Rist and the Glastopf project (The Honeynet Project, now MushMush).

glastopf-nx is Glastopf migrated to Python 3 (tested on Debian 12 with Python 3.11). The upstream project is Python 2 only and is no longer maintained; its authors recommend the successors SNARE and TANNER for new deployments. This copy keeps the upstream history, the GPL licence and the original behaviour: the same emulators, request patterns, dork pages, responses and logs. Only the changes needed to run on Python 3 were made.

Changes for Python 3
--------------------

- Python 3 syntax and standard library names (``configparser``, ``queue``, ``urllib.parse``, ``urllib.request``, ``http.server``, ``io.BytesIO``, octal literals, ``print()``).
- Request and response buffers are bytes in Python 3: ``HTTPHandler`` now encodes text written by the emulators and decodes the request body, so emulators keep working with text as before. Binary responses (favicon, downloaded files, sandbox output) are passed through unchanged.
- ``gevent.wsgi`` (removed in gevent 1.3) is replaced by ``gevent.pywsgi``; the local socket address is read from its Python 3 location.
- ``pylibinjection`` takes and returns bytes in Python 3; the SQL injection classifier converts at the boundary.
- ``cgi.escape`` (removed in Python 3.8) is replaced by the equivalent ``html.escape(quote=False)``; ``str.decode('string_escape')`` by ``codecs.escape_decode``.
- The LFI emulator's path split uses ``(\.\./)+`` because Python 3.7 and later also split on empty regular-expression matches; results are the same as under Python 2.
- ``__import__`` level -1 (Python 2 implicit relative import) is replaced by absolute imports.
- Files are read and written in binary mode where they hold binary data; ``file()`` is replaced by ``open()``.
- The PHP sandbox Makefile calls ``python3``. The sandbox needs the BFR (Better Function Replacer) Zend extension, which builds against PHP 8.2 from https://github.com/mushorg/BFR; without it the RFI and PHP-CGI emulators still record and store the payload but return an empty response.
- ``requirements.txt`` pins the versions tested on Python 3.11. SQLAlchemy stays on 1.4 and PyMongo on 3.x (the code uses APIs removed in SQLAlchemy 2.0 and PyMongo 4). ``MySQL-python`` (Python 2 only) and the unused ``beautifulsoup`` were dropped; use a ``mysql+pymysql://`` connection string with PyMySQL for MySQL.

Not changed: known upstream behaviours such as the TRACE emulator, unsupported methods answered with an internal server error, and raw emulator responses whose lines also appear among the response headers are left as they are, so that the honeypot behaves as the original does. The unit tests under ``glastopf/testing``, the documentation and the Ubuntu 14.04 Dockerfile are the upstream versions and have not been migrated.

Quick start (Debian 12)::

    apt-get install python3-venv python3-dev build-essential make php-cli
    python3 -m venv /opt/glastopf/venv
    /opt/glastopf/venv/bin/pip install -r requirements.txt .
    mkdir /var/lib/glastopf && cd /var/lib/glastopf
    /opt/glastopf/venv/bin/glastopf-runner

The original README follows.

----

We highly recommend to check out the Glastopf successor `SNARE <https://github.com/mushorg/snare>`_ and `TANNER <https://github.com/mushorg/tanner/>`_.

Glastopf |Build Status|
=======================

.. |Build Status| image:: https://travis-ci.org/mushorg/glastopf.svg?branch=master
                       :target: https://travis-ci.org/mushorg/glastopf

ABOUT
-----

Glastopf is a Python web application honeypot founded by Lukas Rist.

General approach:

- Vulnerability *type* emulation instead of vulnerability emulation. Once a vulnerability type is emulated, Glastopf can handle unknown attacks of the same type. While implementation may be slower and more complicated, we remain ahead of the attackers until they come up with a new method or discover a new flaw in our implementation.
- Modular design to add new logging capabilities or attack type handlers. Various database capabilities are already in place. HPFeeds logging is supported for centralized data collection.
- Popular attack type emulation is already in place: Remote File Inclusion via a build-in PHP sandbox, Local File Inclusion providing files from a virtual file system and HTML injection via POST requests.
- Adversaries usually use search engines and special crafted search requests to find their victims. In order to attract them, Glastopf provides those keywords (AKA "dork") and additionally extracts them from requests, extending its attack surface automatically. As a result, the honeypot gets more and more attractive with each new attack attempted on it.
- We will make the SQL injection emulator public, provide IP profiling for crawler recognition and intelligent dork selection.

INSTALL
-------
Installation instructions can be found `here <https://github.com/mushorg/glastopf/tree/master/docs/source/installation>`_.

It is highly recommended to customize the default attack surface to avoid trivial detection of the honeypot.
