OpenStack LBaaS v2 Demo Plan
============================

This is a plan for doing a live demonstration of LBaaSv2 in Liberty.

Demo: Load balancer provisioning
--------------------------------

* Create a load balancer
* Set a security group for the load balancer network port
* Create a listener for the load balancer on port 80
* Create a pool for the listener (round robin)
* Add the two instances as members in the pool

.. code-block:: console

  neutron lbaas-loadbalancer-create --name demolb01 public
  neutron port-update --security-group open VIP_PORT_ID
  neutron lbaas-listener-create --name demolb01-listener-80 --loadbalancer demolb01 --protocol HTTP --protocol-port 80
  neutron lbaas-pool-create --lb-algorithm ROUND_ROBIN --protocol HTTP --name demolb01-pool --listener demolb01-listener-80
  neutron lbaas-member-create --subnet public --address 10.127.121.92 --protocol-port 80 demolb01-pool
  neutron lbaas-member-create --subnet public --address 10.127.121.93 --protocol-port 80 demolb01-pool
  curl VIP_ADDRESS

Demo: Add health checks to pool
-------------------------------

* Add a health check for port 80 on the pool
* Log into instance to verify health checks are working
* Disable web server on instance to test check (if time allows)

.. code-block:: console

  neutron lbaas-healthmonitor-create --delay 5 --max-retries 2 --timeout 10 --type HTTP --pool demolb01-pool
  neutron lbaas-healthmonitor-list
  neutron lbaas-healthmonitor-delete UUID

Demo: Two listeners on one load balancer
----------------------------------------

* Create a listener for the load balancer on port 80
* Create a pool for the listener
* Add the two instances as members in the pool

.. code-block:: console

  neutron lbaas-listener-create --name demolb01-listener-8080 --loadbalancer demolb01 --protocol HTTP --protocol-port 8080
  neutron lbaas-pool-create --lb-algorithm ROUND_ROBIN --protocol HTTP --name demolb01-pool-8080 --listener demolb01-listener-8080
  neutron lbaas-member-create --subnet public --address 10.127.121.92 --protocol-port 80 demolb01-pool-8080
  neutron lbaas-member-create --subnet public --address 10.127.121.93 --protocol-port 80 demolb01-pool-8080

Demo: Drain traffic before instance maintenance
-----------------------------------------------

* Set weight on a member to 0
* Check that requests are no longer sent to that member
* Set weight on a member back to 1

.. code-block:: console

  neutron lbaas-member-list demolb01-pool
  neutron lbaas-member-update --weight 0 MEMBER_UUID demolb01-pool
  curl LB
  neutron lbaas-member-update --weight 1 MEMBER_UUID demolb01-pool
  curl LB

Demo: Load balancer statistics
------------------------------

* Generate traffic to load balancer
* Retrieve statistics

.. code-block:: console

  neutron lbaas-loadbalancer-stats demolb01

Demo: Load balancer algorithm
-----------------------------

* Switch pool to use least connections algorithm
* Test connectivity
* Switch pool to use source ip (sticky/persistent) algorithm
* Test connectivity

.. code-block:: console

  neutron lbaas-pool-update demolb01-pool --lb-algorithm LEAST_CONNECTIONS
  neutron lbaas-pool-update demolb01-pool --lb-algorithm SOURCE_IP
  neutron lbaas-pool-update demolb01-pool --lb-algorithm ROUND_ROBIN

Cleaning up
-----------

.. code-block:: console

  neutron lbaas-pool-delete demolb01-pool
  neutron lbaas-pool-delete demolb01-pool-8080
  neutron lbaas-listener-delete demolb01-listener-8080
  neutron lbaas-listener-delete demolb01-listener-80
  neutron lbaas-loadbalancer-delete demolb01
