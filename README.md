FTS Xapian plugin for Dovecot
=============================

What is this?
-------------

This project intends to provide a straightforward, simple and maintenance free, way to configure FTS plugin for [Dovecot](https://github.com/dovecot/), leveraging the efforts by the [Xapian.org](https://xapian.org/) team.

This effort came after Dovecot team decided to deprecate "fts_squat" included in the dovecot core, and due to the complexity of the Solr plugin capabilitles, un-needed for most users.



Prerequisites
-------------

You are going to need the following things to get this going:

```
* Dovecot 2.2.x (or above)
* Xapian 1.4.x (or above)
* ICU 60.x (or above)
```

You will need to configure properly [Users Home Directories](https://wiki.dovecot.org/VirtualUsers/Home) in dovecot configuration


Installing the Dovecot plugin
-----------------------------

First install the following packages, or equivalent for your operating system. 

```
Ubuntu:
apt-get build-dep dovecot-core
apt-get install dovecot-dev
apt-get install git xapian-core libicu-dev

Archlinux:
pacman -S dovecot
pacman -S xapian-core icu

FreeBSD:
pkg install xapian-core
pkg install xapian-bindings
pkg install icu
```

Clone this project:

```
git clone https://github.com/grosjo/fts-xapian
cd fts-xapian
```

Compile and install the plugin. 

```
autoreconf -vi
./configure --with-dovecot=/path/to/dovecot
make
sudo make install
```

Replace /path/to/dovecot by the actual path to 'dovecot-config'.
Type 'locate dovecot-config' in a shell to figure this out. On ArchLinux , it is /usr/lib/dovecot. 

For specific configuration, you may have to 'export PKG_CONFIG_PATH=...'. To check that, type 'pkg-config --cflags-only-I icu-uc icu-io icu-i18n', it shall return no error.

The module will be placed into the module directory of your dovecot configuration

Update your dovecot.conf file with something similar to:

```
mail_plugins = fts fts_xapian (...)

(...)

plugin {
	plugin = fts fts_xapian (...)

	fts = xapian
	fts_xapian = partial=3 full=20 attachments=0 verbose=0

	fts_autoindex = yes
	fts_enforced = yes
	
	fts_autoindex_exclude = \Trash
(...)
}

(...)
service indexer-worker {
	vsz_limit = 2G // or above (or 0 if you have rather large memory usable on your server, which is preferred for performance) 
}
(...)

```
Partial & full parameters : 3 and 20 are the NGram values for header fields, which means the keywords created for fields (To, Cc, ...) are between 3 and 20 chars long.
Full words are also added by default (if not longer than 245 chars, which is the limit of Xapian capability).

Example: "<john@doe>" will create joh, ohn, hn@, ..., john@d, ohn@do, ..., and finally john@doe as searchable keywords.

Set "verbose=1" to see verbose messages in the log, "verbose=2" for debug
Set "attachments=1" if you want to index attachments (this works only for text attachments)

Set "base_path=/path/..." to specify a base path where the Xapian index
files are stored. Under this directory a directory for each user is created.
The base path directory must be writable for all users (such as 1777 mode).
If a base path is not given, the index files will be stored in the users
Mail directory.

Restart Dovecot:

```
sudo servicectl restart dovecot
```


If this is not a fresh install of dovecot, you need to re-index your mailboxes

```
doveadm index -A -q \*
```

*The first index will re-index all emails, therefore may take a while.*



You shall put in a cron the following command (for daily run for instance) :
```
doveadm fts optimize -A
```


Debugging/Support
-----------------

Please submit requests/bugs via the [GitHub issue tracker](https://github.com/grosjo/fts-xapian/issues).

Thanks to Aki Tuomi <aki.tuomi@open-xchange.com>, Stephan Bosch <stephan@rename-it.nl>, Paul Hecker <paul@iwascoding.com>
