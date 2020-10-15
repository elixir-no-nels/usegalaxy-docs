operations notes
==============



CVMFS add data to repo
----------------------

.. code-block:: bash

  login in to sysadmin@data.usegalaxy.no
  # be roor
  sudo -s
  
  # Make repo writeable
  cvmfs_server  transaction
  
  # Add data

  # update repo:
  cvmfs_server publish
  #Abort changes
  cvmfs_server abort


Create singularity container from conda packages
------------------------------------------------
  
.. code-block:: bash

  ssh sysadmin@usegalaxy.no
  sudo su -
  . .venv/bin/activate
  cd /data/part0/tmp/ # (or somewhere with enough free disk space)
  mulled-build build-and-test 'graphicsmagick=1.3.31' -c iuc,conda-forge,bioconda --test 'ls --help' --singularity
  cp singularity_import/graphicsmagick\:1.3.31 /srv/galaxy/containers/singularity/


Rebuild galaxy client
----------------------
  
.. code-block:: bash
  

  rebuild client:
  cd /srv/galaxy/
  source venv/bin/activate
  cd server
  make client-production-maps



cloudflare DNS 
----------------------

cloudflare email and api key are found in env/main/secret_group_vars/global.vault
.. code-block:: bash
  
   # list entries

   ./bin/cloudflare-cli  -a <API-KEY> -e <EMAIL> list

   # delete entry
   ./bin/cloudflare-cli  -a <API-KEY> -e <EMAIL> delete <UUID>

   #create entry

   ./bin/cloudflare-cli  -a <API-KEY> -e <EMAIL> add help
   add requires: type, name, value and ttl

   ./bin/cloudflare-cli  -a <API> -e <EMAIL> add A test.usegalaxy.no 158.39.201.243 1000

   # do a list to confirm 

firewalld
----------------------
   

.. code-block:: bash
  

   # list all rules
   firewall-cmd --list-all-zones
   # add IP to trusted zone
   firewall-cmd --zone=trusted --add-source=158.39.201.192



db connections IP filtering
----------------------
   

.. code-block:: bash
   
  vim /database/postgres/data/pg_hba.conf

  systemctl restart postgresql-10

  #test db connection.
  /srv/galaxy/server/scripts/manage_db.py -c /srv/galaxy/config/galaxy.yml db_version


Ensure no unencrypted vault-files are commited
----------------------------------------------



.. code-block:: bash

   Add to .git/hooks/pre-commit
   chmod 755 .git/hooks/pre-commit


.. code-block:: bash
    #!/usr/bin/env bash
    #
    # Called by "git commit" with no arguments.  The hook should
    # exit with non-zero status after issuing an appropriate message if
    # it wants to stop the commit.

    # Unset variables produce errors
    set -u

    if git rev-parse --verify HEAD >/dev/null 2>&1
    then
	against=HEAD
    else
	# Initial commit: diff against an empty tree object
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
    fi

    # Redirect output to stderr.
    exec 1>&2

    EXIT_STATUS=0

    # Check that all changed *.vault files are encrypted
    # read: -r do not allow backslashes to escape characters; -d delimiter
    while IFS= read -r -d $'\0' file; do
	[[ "$file" != *.vault && "$file" != *.vault.yml ]] && continue
	# cut gets symbols 1-2
	file_status=$(git status --porcelain -- "$file" 2>&1 | cut -c1-2)
	file_status_index=${file_status:0:1}
	file_status_worktree=${file_status:1:1}
	[[ "$file_status_worktree" != ' ' ]] && {
		echo "ERROR: *.vault file is modified in worktree but not added to the index: $file"
		echo "Can not check if it is properly encrypted. Use git add or git stash to fix this."
		EXIT_STATUS=1
	}
	# check is neither required nor possible for deleted files
	[[ "$file_status_index" = 'D' ]] && continue
	head -1 "$file" | grep --quiet '^\$ANSIBLE_VAULT;' || {
		echo "ERROR: non-encrypted *.vault file: $file"
		EXIT_STATUS=1
	}
	done < <(git diff --cached --name-only -z "$against")

	exit $EXIT_STATUS



