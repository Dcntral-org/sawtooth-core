******************************
Adding or Removing a PBFT Node
******************************

Membership of a Sawtooth PBFT network is controlled by the on-chain setting
``sawtooth.consensus.pbft.members``. An administrator must update this setting
when :ref:`adding a new node <adding-a-pbft-node-label>` or
:ref:`removing an existing node <removing-a-pbft-node-label>`.

All nodes in the network monitor this setting. When a node detects a change,
it updates its local list of PBFT members to match the new list.

.. _adding-a-pbft-node-label:

Adding a PBFT Node
==================

To add a new node to an existing PBFT network, you will install and configure
the node, start it and wait for it to catch up with the rest of the network,
then update ``sawtooth.consensus.pbft.members`` on an existing node.

You can add several nodes at the same time.

1. Install and configure Sawtooth on the new node, as described in
   :doc:`setting_up_sawtooth_poet-sim`. Note these important items:

   * Skip the procedure to create a genesis block.

   * In the :ref:`off-chain validator settings <sysadm-configure-validator-label>`,
     set ``peers`` to include all other nodes in the network. (A PBFT network
     must fully peered.) If you copy this setting from another node, be sure to
     include that node in the list of peers.

     You do not need to update the peer list on other nodes, because Sawtooth
     peering is bidirectional.

#. Start the new node. It will join the Sawtooth network and start the block
   catch-up procedure, where it receives and commits the blocks that are already
   on the blockchain. (For details, see the `PBFT node catch-up RFC
   <https://github.com/hyperledger/sawtooth-rfcs/blob/master/text/0031-pbft-node-catchup.md>`_.)

#. Wait for the node to catch up to the rest of the network (get within a few
   blocks of the chain head). If the new node becomes a PBFT member before it
   has caught up, the entire network could slow down or stall while the new
   member node catches up.

   This process can take a while, depending on the blockchain size, node's
   hardware, and network speed. You can look at the logs or run
   :ref:`sawtooth block list <sawtooth-block-list-label>` to check on the
   new node's progress.

#. Copy the new node's public validator key for use in the next step.

   .. code-block:: console

      $ cat /etc/sawtooth/keys/validator.pub

   This step assumes that the validator key is stored in the default location,
   ``/etc/sawtooth/keys``. If not, use the location specified by the ``key_dir``
   setting (see :doc:`configuring_sawtooth/path_configuration_file`).

#. On an existing member node, update ``sawtooth.consensus.pbft.members`` to
   include the public validator key of the new node.

   Run the following steps on a node that has permission to change on-chain
   settings; that is, a node whose validator key is listed in
   ``sawtooth.identity.allowed_keys``.

   .. Tip::

      Usually, the node that created the genesis block is listed in
      ``sawtooth.identity.allowed_keys``. Note that this list can include user
      keys as well as validator keys, so that changes can be made by an
      administrator from any node. For more information, see
      :ref:`config-onchain-txn-perm-label`.

   a. List the current PBFT member nodes:

      .. code-block:: console

         $ sawtooth settings list --filter sawtooth.consensus.pbft.members

   #. Submit a transaction that specifies the new list of all PBFT member nodes
      (the previous list plus the new node's key).

      .. Important::

         BE VERY CAREFUL! Make sure to specify the full list of keys.
         Double-check each key before you run this command, because a typo could
         stall the network.

      .. code-block:: console

         $ sawset proposal create \
           --key /etc/sawtooth/keys/validator.priv \
           sawtooth.consensus.pbft.members=[previous-list,NEW-KEY]

      If there are no errors, this change will be committed to the blockchain.

#. When all nodes have detected the change and updated their local copy of the
   member list, the new member node begins to participate in the PBFT network.

.. _removing-a-pbft-node-label:

Removing a PBFT Node
====================

To remove an existing node from a PBFT network, you will delete the node's
validator key from  the ``sawtooth.consensus.pbft.members`` setting, then shut
down the removed node. You can delete several nodes at the same time.

.. note::

   PBFT consensus requires a network with at least four nodes. A network with
   fewer than four nodes will fail.

1. Update ``sawtooth.consensus.pbft.members`` to no longer include the
   validator public key of the node you want to remove.

   Run the following steps on a node or as a user that has permission to change
   on-chain settings. For more information, see the tip in
   :ref:`adding-a-pbft-node-label`.

   a. List the current PBFT member nodes:

      .. code-block:: console

         $ sawtooth settings list --filter sawtooth.consensus.pbft.members

   #. Submit a transaction that specifies the new list of all PBFT member nodes
      (the previous list, minus the key of the node or nodes to be removed).

      .. Important::

         BE VERY CAREFUL! Make sure to specify the correct list of keys.
         Double-check each key before you run this command, because a typo could
         stall the network.

      .. code-block:: console

         $ sawset proposal create \
           --key /etc/sawtooth/keys/validator.priv \
           sawtooth.consensus.pbft.members=[UPDATED-LIST]

      If there are no errors, this change will be committed to the blockchain.

#. Make sure that change has been committed. You can check the setting from
   any node.

   .. code-block:: console

      $ sawtooth settings list --filter sawtooth.consensus.pbft.members

   .. note::

      Until the settings change is committed on all nodes, the removed node is
      considered part of the network. If the node is shut down too soon, it
      could be impossible to commit the settings change if there are too few
      working nodes. This is especially important on a small network or when
      removing several nodes at once.

   When all nodes have detected the change and updated their local copy of the
   member list, the node being removed stops participating in the PBFT network.

#. Shut down the old node.

   a. To stop the Sawtooth services, see
      :ref:`stop-restart-sawtooth-services-label`.

   #. To delete blockchain data, logs, and keys from this node, see
      :ref:`stop-sawtooth-ubuntu-label`.

   .. note::

      You do not need to remove this node from the ``peers`` list on the other
      nodes (in the :ref:`off-chain validator settings
      <sysadm-configure-validator-label>`). The network will operate correctly
      even if a removed node is still in this list.


.. Licensed under Creative Commons Attribution 4.0 International License
.. https://creativecommons.org/licenses/by/4.0/
