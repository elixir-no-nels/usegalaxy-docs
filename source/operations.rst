


CVMFS add data to repo
----------------------

.. code-block:: bash

  login in to data.usegalaxy.no
  
  # Make repo writeable
  cvmfs_server  transaction
  
  # Add data

  # update repo:
  cvmfs_server publish
  #Abort changes
  cvmfs_server abort

