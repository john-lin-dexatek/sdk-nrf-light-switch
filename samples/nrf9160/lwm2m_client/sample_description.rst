.. _lwm2m_client_sample_desc:

Sample description
##################

.. contents::
   :local:
   :depth: 2

The LwM2M Client sample demonstrates the usage of the :term:`Lightweight Machine to Machine (LwM2M)` protocol to connect a Thingy:91 or an nRF9160 DK to an LwM2M server through LTE.
To achieve this, the sample uses the Zephyr's :ref:`lwm2m_interface` client and |NCS| :ref:`lib_lwm2m_client_utils` library.
The former provides a device vendor agnostic client implementation, whereas the latter includes all the Nordic specific bits and pieces.

The sample also supports a proprietary mechanism to fetch location assistance data from `nRF Cloud`_ by proxying it through the LwM2M server.
For this, the sample makes use of the :ref:`lib_lwm2m_location_assistance` library.

Requirements
************

.. _supported boards:

The sample supports the following development kits:

.. table-from-sample-yaml::

.. include:: /includes/tfm.txt

Additionally, the sample requires an activated SIM card, and an LwM2M server such as `Leshan Demo Server`_ or `Coiote Device Management`_ server.

Overview
********

LwM2M is an application layer protocol based on CoAP over UDP.
It is designed to expose various resources for reading, writing, and executing through an LwM2M server in a very lightweight environment.
The client sends data such as button and switch states, accelerometer data, temperature, and GNSS position to the LwM2M server.
It can also receive activation commands such as buzzer activation and light control.

.. note::
   The GNSS module interferes with the LTE connection.
   If GNSS is not needed or if the device is in an area with poor connection, disable the GNSS module.
   You can disable the GNSS module by setting the configuration option :ref:`CONFIG_APP_GNSS <CONFIG_APP_GNSS>` to ``n``.


The sample implements the following LwM2M objects:

.. list-table:: LwM2M objects
   :header-rows: 1

   *  - LwM2M objects
      - Object ID
      - Thingy:91
      - nRF9160 DK
   *  - LwM2M Server
      - 1
      - Yes
      - Yes
   *  - Device
      - 3
      - Yes
      - Yes
   *  - Connectivity Monitoring
      - 4
      - Yes
      - Yes
   *  - FOTA
      - 5
      - Yes
      - Yes
   *  - Location
      - 6
      - Yes
      - Yes
   *  - Accelerometer
      - 3313
      - Yes
      - Simulated
   *  - Color
      - 3335
      - Yes
      - Simulated
   *  - Temperature
      - 3303
      - Yes
      - Simulated
   *  - Pressure
      - 3323
      - Yes
      - Simulated
   *  - Humidity
      - 3304
      - Yes
      - Simulated
   *  - Generic Sensor
      - 3300
      - Yes
      - Simulated
   *  - Light Control
      - 3311
      - Yes
      - Yes
   *  - Push Button
      - 3347
      - Yes
      - Yes
   *  - Buzzer
      - 3338
      - Yes
      - No
   *  - On/Off Switch
      - 3342
      - No
      - Yes
   *  - GNSS Assistance
      - 33625
      - Yes
      - Yes
   *  - Ground Fix
      - 33626
      - Yes
      - Yes
   *  - Visible Wi-Fi Access Point
      - 33627
      - Yes
      - Yes

User interface
**************

Button 1:
   Triggers GNSS-based location service

Button 2:
   Triggers cell-based location service


.. _state_diagram:

State diagram
=============

The following diagram shows states and transitions for the LwM2M Client:

.. figure:: /images/lwm2m_client_state_diagram.svg
   :alt: LwM2M client state diagram

When the device boots up, the sample first connects to the LTE network and initiates the LwM2M connection.
If there are errors, in most error cases, the sample tries to reconnect the LwM2M client.
In the case of network errors, it tries to reconnect the LTE network.
When the number of retries to restore the network connection exceeds three times, the sample falls back to the bootstrap.
This enables the recovery in the cases where the LwM2M client credentials are outdated or removed from the server.

.. _dtls_support:

DTLS Support
============

The sample has DTLS security enabled by default.
You need to provide the following information to the LwM2M server before you can make a successful connection:

* Client endpoint
* Identity
* `Pre-Shared Key (PSK)`_

See :ref:`server setup <server_setup_lwm2m_client>` for instructions on providing the information to the server.

Sensor simulation
=================

If a sensor simulator is defined in Devicetree with the ``sensor_sim`` node label, it will be used over real devices.
This is useful, for example, on the nRF9160 DK, where only simulated sensor data is available, as it does not have any of the external sensors needed for actual measurements.

.. _notifications_lwm2m:

Notifications
=============

LwM2M specifies the Notify operation, which can be used to notify the server of changes to the value of a resource field, for example, the measured value of a temperature sensor.
This allows active monitoring while using minimal bandwidth, as notifications can be sent with the updated values without the need for the server querying the client regularly.

To enable notifications, the server must initiate an observation request on one or more resources of interest.
For more information, see :ref:`notifications_setup_lwm2m`.

.. _sensor_module_lwm2m:

Sensor Module
=============

The sample has a sensor module which, if enabled, reads the selected sensors and updates the client's LwM2M resource values.
Each sensor can be enabled separately.

The sensor module is intended to be used together with notifications.
If notifications are enabled for a Sensor Value resource and the corresponding sensor is enabled in the sensor module, a notification will be sent when the value changes.
The frequency of notification packets is configured by LwM2M attributes set by the server.

See :ref:`sensor_module_options` for information on enabling and configuring the sensor module.

.. note::

   When you track several resources and enable sensor module for several sensors , socket errors such as ``net_lwm2m_engine: Poll reported a socket error, 08`` and ``net_lwm2m_rd_client: RD Client socket error: 5`` might occur.

Configuration
*************

|config|

Setup
=====

Before building and running the sample, complete the following steps:

1. Select the device you plan to test.
#. Select the LwM2M server for testing.
#. Setup the LwM2M server by completing the steps listed in :ref:`server_setup_lwm2m_client`.
   This step retrieves the server address and the security tag that will be needed during the next steps.
#. :ref:`server_addr_PSK`.

.. |dtls_support| replace:: The same credentials must be provided in the :guilabel:`Leshan Demo Server Security configuration` page (see :ref:`dtls_support` for instructions).

.. _server_setup_lwm2m_client:

.. include:: /includes/lwm2m_common_server_setup.txt


.. note::

   The **Client Configuration** page of the LwM2M Bootstrap server and the **Registered Clients** page of the LwM2M server display only a limited number of devices by default.
   You can increase the number of displayed devices from the drop-down menu associated with **Rows per page**.
   In both cases, the menu is displayed at the bottom-right corner of the **Client Configuration** pages.

.. _server_addr_PSK:

Set the server address and PSK
------------------------------

1. Open :file:`src/prj.conf`.
#. Set :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_SERVER` to the correct server URL:

   * For `Leshan Demo Server`_ - ``coaps://leshan.eclipseprojects.io:5684`` (`public Leshan Demo Server`_).
   * For `Coiote Device Management`_ - ``coaps://eu.iot.avsystem.cloud:5684`` (`Coiote Device Management server`_).
   * For `Leshan Bootstrap Demo Server <public Leshan Bootstrap Server Demo_>`_ - ``coaps://leshan.eclipseprojects.io:5784``
   * For Coiote bootstrap server - ``coaps://eu.iot.avsystem.cloud:5694``
#. Set :kconfig:option:`CONFIG_LWM2M_RD_CLIENT_SUPPORT_BOOTSTRAP` if bootstrap is used.
#. Set :ref:`CONFIG_APP_LWM2M_PSK <CONFIG_APP_LWM2M_PSK>` to the hexadecimal representation of the PSK used when registering the device with the server.

.. _notifications_setup_lwm2m:

Enabling notifications
----------------------

Enabling notifications for a resource varies slightly from server to server.
The client must be connected to the server to enable notifications.
Following are the instructions for enabling notifications in the Leshan Demo server and Coiote Device Management server:

.. tabs::

   .. tab:: Leshan Demo Server

      1. Open the `Leshan Demo Server web UI`_.
      #. Identify your device in the :guilabel:`Clients` tab and select it.
      #. Select the desired object in the menu on the left in the UI.
      #. Identify one or more resources you want to track and click :guilabel:`OBS` next to it.

         * You can track either a single resource or all the resources of an object. It is recommended to track only the resources that are expected to change.
         * If you want to use the :ref:`sensor_module_lwm2m`, at least the Sensor Value resource must be tracked for all sensors enabled in the Sensor Module.

   .. tab:: Coiote Device Management server

      1. Open `Coiote Device Management server`_.
      #. Click :guilabel:`Device inventory` tab in the top.
      #. Identify your device in the list and click on the anchor text corresponding to the device ID in the **Identity** column.
      #. Click the :guilabel:`Objects` tab in the new menu to the left, just below :guilabel:`Dashboard`.
      #. Identify one or more objects that you want to receive notifications from, and expand it by clicking on them.
      #. Identify one or more resources of the object that you want to track:

         * You can track either a single resource or all the resources of an object. It is recommended to track only the resources that are expected to change.
         * If you want to use the :ref:`sensor_module_lwm2m`, at least the Sensor Value resource must be tracked for all sensors enabled in the Sensor Module.

      #. Click the :guilabel:`Value Tracking` button of the selected resource.
      #. Select :guilabel:`Observe` or :guilabel:`Monitoring` from the dropdown menu.

         * Selecting :guilabel:`Observe` will only update the Value field of the resource when it receives a notification.
         * Selecting :guilabel:`Monitoring` will additionally create a graph of the logged datapoints.

      #. Click :guilabel:`Limit data usage` to configure how often notifications are sent.

Configuration options
=====================

Check and configure the following configuration options for the sample:

Server options
--------------

.. _CONFIG_APP_LWM2M_PSK:

CONFIG_APP_LWM2M_PSK - Configuration for Pre-Shared Key
   The sample configuration sets the hexadecimal representation of the PSK used when registering the device with the server.

.. _CONFIG_APP_ENDPOINT_PREFIX:

CONFIG_APP_ENDPOINT_PREFIX - Configuration for setting prefix for endpoint name
   This configuration option changes the prefix of the endpoint name.

LwM2M objects options
---------------------

.. _CONFIG_APP_TEMP_SENSOR:

CONFIG_APP_TEMP_SENSOR - Configuration for enabling an LwM2M Temperature sensor object
   The sample configuration enables an LwM2M Temperature sensor object.
   All compatible objects are enabled by default.
   Disabled objects will not be visible in the server.
   You can use this configuration option for other LwM2M objects also by modifying the option accordingly.

.. _CONFIG_APP_GNSS:

CONFIG_APP_GNSS - Configuration for enabling GNSS functionality
   This configuration might interfere with LTE if the GNSS conditions are not optimal.
   Disable this option if GNSS is not needed.

.. _CONFIG_GNSS_PRIORITY_ON_FIRST_FIX:

CONFIG_GNSS_PRIORITY_ON_FIRST_FIX - Configuration for prioritizing GNSS over LTE during the search for first fix
   Enabling this option makes it significantly easier for the GNSS module to find a position but will also affect performance for the rest of the application during the search for first fix.

.. _CONFIG_LWM2M_IPSO_APP_COLOUR_SENSOR_VERSION_1_0:

CONFIG_LWM2M_IPSO_APP_COLOUR_SENSOR_VERSION_1_0 - Configuration for selecting the IPSO Color sensor object version
   The configuration option sets the version of the OMA IPSO object specification to 1.0.
   The user-defined Color sensor IPSO object uses this version.

.. _CONFIG_LWM2M_IPSO_APP_COLOUR_SENSOR_VERSION_1_1:

CONFIG_LWM2M_IPSO_APP_COLOUR_SENSOR_VERSION_1_1 - Configuration for selecting the IPSO Color sensor object version
   The configuration option sets the version of the OMA IPSO object specification to 1.1.
   The user-defined Color sensor IPSO object uses this version.

.. _CONFIG_APP_CUSTOM_VERSION:

CONFIG_APP_CUSTOM_VERSION - Configuration to set custom application version reported in the Device object
   The configuration option allows to specify custom application version to be reported to the LwM2M server.
   The option has the current |NCS| version as the default value.

.. _sensor_module_options:

Sensor module options
---------------------

.. _CONFIG_SENSOR_MODULE:

CONFIG_SENSOR_MODULE - Configuration for periodic sensor reading
   This configuration option enables periodic reading of sensors and updating the resource values when the change is sufficiently large.
   If you enable the option, the sample notifies the server if a change in one or more resources is observed.

.. _CONFIG_SENSOR_MODULE_TEMP:

CONFIG_SENSOR_MODULE_TEMP - Configuration to enable Temperature sensor
   This configuration option enables the Temperature sensor in the Sensor Module.

.. note::

   You can use the configuration options for different sensor types by modifying the configuration options accordingly.

Additional configuration
========================

Check and configure the following LwM2M options that are used by the sample:

* :kconfig:option:`CONFIG_LWM2M_PEER_PORT` - LwM2M server port.
* :kconfig:option:`CONFIG_LWM2M_ENGINE_MAX_OBSERVER` - Maximum number of resources that can be tracked.
  You must increase this value if you want to observe more than 10 resources.
* :kconfig:option:`CONFIG_LWM2M_ENGINE_MAX_MESSAGES` - Maximum number of LwM2M message objects.
  You must increase this value if many notifications will be sent at once.
* :kconfig:option:`CONFIG_LWM2M_ENGINE_MAX_PENDING` - Maximum number of pending LwM2M message objects.
  You must increase this value if you want to observe more than 10 resources.
* :kconfig:option:`CONFIG_LWM2M_ENGINE_MAX_REPLIES` - Maximum number of LwM2M reply objects.
  You must increase this value if many notifications will be sent at once.
* :kconfig:option:`CONFIG_LWM2M_COAP_BLOCK_SIZE` - Increase if you need to add several new LwM2M objects to the sample, as the registration procedure contains information about all the LwM2M objects in one block.
* :kconfig:option:`CONFIG_LWM2M_ENGINE_DEFAULT_LIFETIME` - Set this option to configure how often the client sends ``I'm alive`` messages to the server.
* :kconfig:option:`CONFIG_LWM2M_IPSO_TEMP_SENSOR_VERSION_1_0` - Sets the IPSO Temperature sensor object version to 1.0.
  You can use this configuration option for other IPSO objects also by modifying the option accordingly.
  See the `LwM2M Object and Resource Registry`_ for a list of objects and their available versions.
* :kconfig:option:`CONFIG_LWM2M_IPSO_TEMP_SENSOR_VERSION_1_1` - Sets the IPSO Temperature sensor object version to 1.1.
  You can use this configuration option for other IPSO objects also by modifying the option accordingly. See the `LwM2M Object and Resource Registry`_ for a list of objects and their available versions.
* :kconfig:option:`CONFIG_LWM2M_SHELL` - Enables the Zephyr shell and all LwM2M specific commands.

.. note::
   Changing lifetime might not work correctly if you set it to a value beyond 60 seconds.
   It might cause resending message error and eventually timeout.
   This is an issue specific for IPV4 when NAT is in use.

For Thingy:91, configure the ADXL362 accelerometer sensor range by choosing one of the following options (default value is |plusminus| 2 g):

* :kconfig:option:`CONFIG_ADXL362_ACCEL_RANGE_2G` - Sensor range of |plusminus| 2 g.
* :kconfig:option:`CONFIG_ADXL362_ACCEL_RANGE_4G` - Sensor range of |plusminus| 4 g.
* :kconfig:option:`CONFIG_ADXL362_ACCEL_RANGE_8G` - Sensor range of |plusminus| 8 g.

Resolution depends on range: |plusminus| 2 g has higher resolution than |plusminus| 4 g, which again has higher resolution than |plusminus| 8 g.

If you use an external GNSS antenna, add the following configuration:

* :kconfig:option:`CONFIG_MODEM_ANTENNA_GNSS_EXTERNAL` - Selects an external GNSS antenna.

Location assistance options
---------------------------

Check and configure the following library options that are used by the sample:

* :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_GROUND_FIX_OBJ_SUPPORT` - Uses Ground Fix Location object (ID 33626).
  Used with nRF Cloud to estimate the location of the device based on the cell neighborhood and Wi-Fi AP neighborhood.
* :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_GNSS_ASSIST_OBJ_SUPPORT` - Uses GNSS Assistance object (ID 33625).
  Used with nRF Cloud to request assistance data for the GNSS module.
* :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_LOCATION_ASSIST_AGPS` - nRF Cloud provides A-GPS assistance data and the GNSS-module in the device uses the data for obtaining a GNSS fix, which is reported back to the LwM2M server.
* :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_LOCATION_ASSIST_PGPS` - nRF Cloud provides P-GPS predictions and the GNSS-module in the device uses the data for obtaining a GNSS fix, which is reported back to the LwM2M server.
* :kconfig:option:`CONFIG_LWM2M_CLIENT_UTILS_NEIGHBOUR_CELL_LISTENER` - Disable this option if you provide your own method of populating the LwM2M objects (ID 10256) containing the cell neighborhood information.


Configuration files
===================

The sample provides predefined configuration files for typical use cases.

The following files are available:

* :file:`prj.conf` - Standard default configuration file.
* :file:`overlay-queue.conf` - Enables LwM2M Queue Mode support.
* :file:`overlay-leshan-bootstrap.conf` - Enables LwM2M bootstrap support with Leshan demo server.
* :file:`overlay-avsystem.conf` - Uses `Coiote Device Management`_ server.
* :file:`overlay-avsystem-bootstrap.conf` - Uses Coiote in bootstrap mode.
* :file:`overlay-nbiot.conf` - Enables the use of NB-IoT.
* :file:`overlay-assist-agps.conf` - Enables A-GPS assistance.
* :file:`overlay-assist-cell.conf` - Enables cell-based location assistance.
* :file:`overlay-assist-pgps.conf` - Enables P-GPS assistance in the sample.
* :file:`overlay-lowpower.conf` - Disables certain features to bring the power consumption down.
* :file:`overlay-adv-firmware.conf` - Enables experimental Advanced Firmware Update object.

Moreover, the sample also provides the following files for LwM2M 1.1 features:

* :file:`overlay-lwm2m-1.1.conf` - Enables v1.1 and running of Interoperability Test Cases [0-499].
* :file:`overlay-lwm2m-1.1-core-interop.conf` - Enables v.1.1 and running of Core Specific Objects Test cases [500-999].
* :file:`overlay-lwm2m-1.1-object-interop.conf` -  Enables v.1.1 and running of Additional Objects Test cases [1000-1999].

For further information about the test cases, see `Enabler Test Specification (Interoperability) for Lightweight M2M`_.

You can configure the sample either by editing the :file:`prj.conf` file and the relevant overlay files, or through menuconfig or guiconfig.

.. _build_lwm2m:

Building and running
********************

.. |sample path| replace:: :file:`samples/nrf9160/lwm2m_client`

.. include:: /includes/build_and_run_ns.txt

After building and running the sample, you can locate your device in the server:

* Leshan - Devices are listed under **Clients**.
* Coiote - Devices are listed under **Device inventory**.

You can also optionally enable notifications for the resources so that the server actively monitors the resources.

Queue Mode support
==================

To use the LwM2M Client with LwM2M Queue Mode support, build it with the ``-DOVERLAY_CONFIG=overlay-queue.conf`` option:

.. code-block:: console

   west build -b nrf9160dk_nrf9160_ns -- -DOVERLAY_CONFIG=overlay-queue.conf

Bootstrap support
=================

To successfully run the bootstrap procedure, you must first register the device in the LwM2M bootstrap server.

See :ref:`Registering your device to an LwM2M boot strap server <bootstrap_server_reg>` for instructions.

To build the LwM2M Client with LwM2M bootstrap support, build it with the ``-DOVERLAY_CONFIG=overlay-bootstrap.conf`` option:

.. code-block:: console

   west build -b nrf9160dk_nrf9160_ns -- -DOVERLAY_CONFIG=overlay-bootstrap.conf

See :ref:`cmake_options` for instructions on how to add this option.
Keep in mind that the used bootstrap port is set in the aforementioned configuration file.

Testing
=======

|test_sample|

   #. |connect_kit|
   #. |connect_terminal|
   #. Observe that the sample starts in the terminal window.
   #. Check that the device is connected to the chosen LwM2M server.
   #. Press **Button 1** on nRF9160 DK or **SW3** on Thingy:91 and confirm that the button event appears in the terminal.
   #. Check that the button press event has been registered on the LwM2M server by confirming that the press count has been updated.
   #. Retrieve sensor data from various sensors and check if values are reasonable.
   #. Test GNSS module:

      a. Ensure that :ref:`CONFIG_GNSS_PRIORITY_ON_FIRST_FIX <CONFIG_GNSS_PRIORITY_ON_FIRST_FIX>` is enabled.
      #. Ensure that you are in a location with good GNSS signal, preferably outside.
      #. Wait for the GNSS to receive a fix, which will be displayed in the terminal.
         It might take several minutes for the first fix.

   #. Try to enable or disable some sensors in menuconfig and check if the sensors
      appear or disappear correspondingly in the LwM2M server.

.. _lwmwm_client_testing_shell:

Testing with the LwM2M shell
----------------------------

To test the sample using :ref:`lwm2m_shell`, complete the following steps:

.. note::

   To enable any LwM2M v1.1 specific commands (for example, ``LwM2M SEND``), choose any of the v1.1 overlay files.

1. Open a terminal emulator and observe that the development kit produces an output similar to the above section.
#. Verify that the shell inputs are working correctly by running few commands.
   You can, for example, run commands to trigger a registration update, pause and start the client.

   a. Registration update:

      .. code-block:: console

         uart:~$ lwm2m update

   #. Pause the client:

      .. code-block:: console

         uart:~$ lwm2m pause

   #. Resume the client:

      .. code-block:: console

         uart:~$ lwm2m resume

Firmware Over-the-Air (FOTA)
============================

You can update the firmware of the device if you are using Coiote Device Management server or Leshan server.
Application firmware updates and modem firmware (both full and delta) updates are supported.

To update the firmware, complete the following steps:

   1. Identify the firmware image file to be uploaded to the device. See :ref:`lte_modem` and :ref:`nrf9160_fota` for more information.
   #. Open `Coiote Device Management server`_ and click :guilabel:`LwM2M firmware`.
   #. Click :guilabel:`Schedule new firmware upgrade`.
   #. Click :guilabel:`Upload file` in the bottom left corner and upload the firmware image file.
   #. Configure the necessary firmware update settings in the menu to the right.
   #. Click :guilabel:`Upgrade`.
   #. Observe in the terminal window that the image file is being downloaded.
      The download will take some time.
      If you do not increase the server lifetime, the Coiote server might drop the connection to the device. The device reconnects later.
   #. When the download is complete, the device restarts on its own after installing the firmware.
      Restart the device manually if it has not started automatically.
      The device runs the updated firmware and reconnects to Coiote Device Management server automatically.


Dependencies
************

This sample application uses the following |NCS| libraries and drivers:

* :ref:`lib_lwm2m_client_utils`
* :ref:`lib_lwm2m_location_assistance`
* :ref:`modem_info_readme`
* :ref:`at_cmd_parser_readme`
* :ref:`dk_buttons_and_leds_readme`
* :ref:`lte_lc_readme`
* :ref:`lib_date_time`
* :ref:`lib_dfu_target`
* :ref:`lib_fmfu_fdev`
* :ref:`lib_fota_download`
* :ref:`lib_download_client`

It uses the following `sdk-nrfxlib`_ library:

* :ref:`nrfxlib:nrf_modem`

It uses the following Zephyr libraries:

* :ref:`gpio_api`
* :ref:`lwm2m_interface`
* :ref:`pwm_api`
* :ref:`sensor_api`

In addition, it uses the following secure firmware component:

* :ref:`Trusted Firmware-M <ug_tfm>`
