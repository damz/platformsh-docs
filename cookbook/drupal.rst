Drupal
======

How to set up your local Drupal development
-------------------------------------------

This guide will teach you how to set up local Drupal development for use with Platform.sh.

Prerequisites
^^^^^^^^^^^^^

To succeed with Platform.sh you need the following installed on your local machine:

* An `id_rsa public/private keypair <https://help.github.com/articles/generating-ssh-keys/>`_
* `Git <http://git-scm.com/>`_
* `Composer <https://getcomposer.org/>`_
* `The Platform CLI tool <https://github.com/platformsh/platformsh-cli>`_
* `Drush <https://github.com/drush-ops/drush>`_
* `MariaDB (or MySQL) database <https://mariadb.org/>`_
* Optional: `Solr <https://lucene.apache.org/solr/>`_, `Redis <http://redis.io/>`_

You will also need to have signed up for a `Platform.sh <https://platform.sh>`_ project.

Goals
^^^^^

#. Authenticate locally using the `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_
#. Upload your SSH public key
#. Use the `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_ to obtain and build your project’s repository
#. Connect to your local database
#. Use your Drush aliases
#. Synchronize databases and files with Platform.sh
 
Authenticate locally using the `platform CLI tool`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_ will authenticate you with Platform.sh and show your projects.

.. code-block:: console

  # this command will authenticate you on Platform.sh
  $ platform
	
The credentials you enter are the same for your account on `Commerce Guys' Marketplace <https://marketplace.commerceguys.com/user>`_.

Upload your SSH public key
^^^^^^^^^^^^^^^^^^^^^^^^^^

You need an `id_rsa public/private keypair <https://help.github.com/articles/generating-ssh-keys/>`_ to use Platform.sh. 

To upload the public key in the browser go to `your user account <https://marketplace.commerceguys.com/user>`_ and click the `SSH Keys` tab. Name your key in the *Title* field, and paste the public key into the *Key* field. Your key will typically be found at ``~/.ssh/id_rsa.pub`` on Linux and Mac OS X machines.

.. image:: images/edit-ssh.png
   :alt: Screenshot of a public key field

Use the platform CLI to obtain and build your project’s repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``platform`` command will show you a list of your projects. 

.. code-block:: console

	$ platform
	Welcome to Platform.sh!
	Your projects are: 
	+---------------+-------------------------------------+-------------------------------------------------+
	| ID            | Name                                | URL                                             |
	+---------------+-------------------------------------+-------------------------------------------------+
	| ag5vkxcevi4su | My Drupal Site                      | https://eu.platform.sh/#/projects/ag5vkxcevi4su |
	| bdzgZfm4tg27x | A Symfony Project                   | https://eu.platform.sh/#/projects/bdzgZfm4tg27x |

I can obtain a local copy of the project using the ``platform get [ID]`` command:

.. code-block:: console

	# This command will get the `My Drupal Site` project
	$ platform get ag5vkxcevi4su

Now you can see the local directory structure that the Platform CLI provides for your local development:

.. code-block:: console

	$ ls -1
	# Contains all builds of your projects
	builds
	
	# Checkout of the Git repository
	repository
	
	# Your files directory, and your settings.local.php file
	shared
	
	# A symlink that always references the latest build
	www -> builds/2014-10-24--15-21-46--staging
	
The ``builds`` directory contains every build of your project. The ``repository`` directory is your local checkout of the Platform.sh Git repository. The ``shared`` directory is for your settings.local.php file which stores the connection details to your local database. 

See the section below about Settings.php for a full explanation of the settings.local.php file.

The builds directory contains built copies of the Drupal site. This is relevant when you use Drush Make files to assist in your site building.
The www symlink is created by the platform build command and will always reference the latest build in the builds directory. The www directory should become your DOCROOT for local development.
The .platform-project file is metadata for the platform tool.

Settings.php
^^^^^^^^^^^^

Drupal sites use a file called settings.php to store database connection details and other important configuration. Platform.sh has a specific concept for managing settings.php which is important to understand to succeed. 

Connect your local database
^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are building in vanilla mode (you commit all of Drupal's files directly into the Git repository and don't use Drush Make), you should add your own settings.local.php file with your local database credentials directly to sites/default. The following lines are present in your repository's .gitignore file, which will guarantee that a settings.local.php file won't get committed to Git:

.. code-block:: php

	# Ignore configuration files that may contain sensitive information.
	sites/*/settings*.php

If you are building with Drush Make, the proper place for your file is shared/settings.local.php. The `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_ will have created this file for you when you ran the platform get command.

.. note:: 
	If there is no shared/settings.local.php file, create one following the `example found here <https://github.com/platformsh/platformsh-cli/blob/master/resources/drupal/settings.local.php>`_, and re-run platform build.) 

When using Drush Make files, the platform build command will generate a sites/default/settings.php file with each build of your application (when using Drush Make files). The shared/settings.local.php file will also be symlinked into the www/sites/default directory, where the generated settings.php can include it.

Example code from the generated settings.php:

.. code-block:: php

	$local_settings = dirname(__FILE__) . '/settings.local.php';
	if (file_exists($local_settings)) {
	  include $local_settings;
	}

The above code (found in the generated settings.php) shows how the shared/settings.local.php gets included in the built application.

There is a folder that the Platform CLI will use for local builds that require database credentials: ``shared/settings.local.php``

.. code-block:: php

   <?php
   // Database configuration.
   $databases['default']['default'] = array(
     'driver' => 'mysql',
     'host' => 'localhost',
     'username' => '',
     'password' => '',
     'database' => '',
     'prefix' => '',
   );


Drush Aliases
^^^^^^^^^^^^^

The `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_ generates and maintains Drush Aliases that allow you to issue remote Drush commands on any environment (branch) that is running on Platform.sh. After you have run platform build, there is also a Drush Alias for your local site. 

For example, on my site, here are my Drush Aliases:


Change the Drush Alias Group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can set the Drush alias group name to something more convenient:

.. code-block:: console

	$ platform drush-aliases -g robshouse

Synchronize Databases and Files with the Platform CLI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Given the Drush aliases shown above, I can now use a normal Drush command to synchronize my local database with the data from my Master environment online:

.. code-block:: console

	$ drush sql-sync @robshouse.master @robshouse._local

In the same style, use Drush to grab the uploaded files from the files directory and pull them into your local environment:

.. code-block:: console

	$ drush rsync @robshouse.staging:%files @robshouse._local:%files

Use the Platform CLI to Build Your Project 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Drupal sites have two separate build modes. In the “Vanilla” build mode, you commit all of your Drupal files (sans settings.php, which you should let Platform generate) into your git repository (NOTE: Never commit the uploaded files in your files directory - these never go into Git). In this build mode, running platform build will take care of creating symlinks with the local shared directory but nothing more.

The other build mode is when you use a directory structure like this and a Drush Make file to build your project. This is the best way! In this build mode, platform build will first execute the Drush Make file, and then move the other assets (modules, themes, libraries) into the newly built www/sites/default/ directory. It will then take care of the symlinks with the local shared directory.

Step 3: Sync your Database and Files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We are going to use the drush command ``sql-sync`` which accepts drush aliases @from and @to. 

.. code-block:: console

   $ drush sql-sync @platform.master @platform.local
   You will destroy data in db and replace with data from ssh.eu.platform.sh/main.
   You might want to make a backup first, using the sql-dump command.
   Do you really want to continue? (y/n): y
   
.. note::
   
   If you get an error like this one: ``Error: no database record could be found for target @platform.local``, then you can add the ``db-url`` to your local alias by opening up the drush alias file (see step 1) and adding this line: ``'db-url' => 'mysql://un:pw@localhost:8889/dbname',`` to the ``$aliases['local']`` array.
   
Finally, we need to bring in an up-to-date copy of the files to your local site (``drush rsync @from @to``):

.. code-block:: console

   $ drush rsync @platform.master:%files @platform.local:%files
   
.. note::
 
   If you get an error like ``Could not evaluate source path @platform.master%file`` you'll want to make sure you have a colon between the drush-alias and the file folder declaration. Or if you get an error like ``rsync: mkdir "~/Sites/platform/www/sites/default/files" failed: No such file or directory (2)`` then you'll want to make sure your ``'root' =>`` in your drush alias is pointing to a non-relative directory.
   
Step 4: Access your newly local running site!
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Congratulations! You now have a local version of your platform all setup. To synchronize from your chosen branch to your local, run these four commands from your ``platform/repository`` folder:

.. code-block:: console

   $ git pull
   $ platform build
   $ drush sql-sync @platform.master @platform.local
   $ drush rsync @platform.master:%files @platform.local:%files

IDE Specific Tips
^^^^^^^^^^^^^^^^^

MAMP pro:

In order for MAMP to work well with the symlinks created by the `platform CLI tool <https://github.com/platformsh/platformsh-cli>`_, add the following to the section under Hosts > Advanced called “Customized virtual host general settings.”

.. code-block:: console

	<Directory />
	        Options FollowSymLinks
	        AllowOverride All
	</Directory>

.. seealso::
	`Laravel Forum Archives <http://forumsarchive.laravel.io/viewtopic.php?pid=11232#p11232>`_

last update: |today|
