# Light lightness example

@tag52810nosupport

This example demonstrates how you can use mesh messages and events
from the [Light Lightness model](@ref LIGHT_LIGHTNESS_MODELS) API
to control the brightness of the LED on your board.

The example is composed of two minor examples that use the Light Lightness Client/Setup Server model: 
- Light Lightness Setup Server
- Light Lightness Client

For more information about the Light Lightness Client/Server model, see also the Bluetooth SIG's @link_ModelOverview.

For provisioning purposes, the example requires either the provisioner example that is provided in the @ref md_examples_provisioner_README
or the nRF Mesh mobile app.

Both the Light Lightness Setup Server and Light Lightness Client examples have the provisionee role
in the network.
They support provisioning over Advertising bearer (PB-ADV) and GATT bearer (PB-GATT)
and also support Mesh Proxy Service (Server).
Read more about the Proxy feature in @ref md_doc_user_guide_modules_provisioning_gatt_proxy.

**Table of contents**
- [Light Lightness Client](@ref light_lightness_example_light_lightness_client)
- [Light Lightness Setup Server](@ref light_lightness_example_light_lightness_server)
- [Light Lightness Client/Setup Server model](@ref light_lightness_example_light_lightness_model)
- [Hardware requirements](@ref light_lightness_example_hw_requirements)
- [Software requirements](@ref light_lightness_example_sw_requirements)
- [Setup](@ref light_lightness_example_setup)
    - [LED and button assignments](@ref light_lightness_example_setup_leds_buttons)
- [Testing the example](@ref light_lightness_example_testing)
    - [Evaluating using the static provisioner](@ref light_lightness_example_testing_dk)
    - [Evaluating using the nRF Mesh mobile app](@ref light_lightness_example_testing_app)
    - [Interacting with the boards](@ref light_lightness_example_testing_interacting)
        - [Controlling the lightness value](@ref light_lightness_example_controlling_lightness_value)
        - [Changing behavior on power-up](@ref light_lightness_example_changing_behavior_on_powerup)
        - [Restricting the range of the lightness value](@ref light_lightness_example_changing_lightness_range)
        - [Other factory default configuration](@ref light_lightness_example_other_factory_default_configuration)


![Light Lightness example structure](ll_example_structure.svg)

### Light Lightness Client example @anchor light_lightness_example_light_lightness_client

The Light Lightness Client example has a provisionee role in the network.
It implements two instances of the [Light Lightness Client model](@ref LIGHT_LIGHTNESS_CLIENT).
These instances are used to control the brightness of the LED 1 on the servers, 
the range of supported lightness levels, and the default lightness value after the servers' boot-up.

### Light Lightness Setup Server example @anchor light_lightness_example_light_lightness_server

The Light Lightness Setup Server example has a provisionee role in the network. 
It implements one instance of the [Light Lightness Setup Server model](@ref LIGHT_LIGHTNESS_SETUP_SERVER).

This model instance is used to receive the lightness level and change
the brightness of the LED 1 on the server board, whenever the Light Lightness Actual
or Light Lightness Linear state is changed. A change in the Light Lightness Actual state is 
reflected in the Light Lightness Linear state, and the other way around. 

The model instance uses the @link_APP_PWM library of the nRF5 SDK to control
the brightness of the LED. To map the lightness level to the allowed range of 
the PWM ticks, the value of the Light Lightness Actual state is converted to the value
of the Generic Level state.

### Light Lightness Client/Setup Server model @anchor light_lightness_example_light_lightness_model

The Light Lightness Client model is used for manipulating the following states
associated with the peer Light Lightness Setup Server model:
- Light Lightness Actual
- Light Lightness Linear
- Light Lightness Default
- Light Lightness Last
- Light Lightness Range

More information about the Light Lightness models can be found in the 
[Light Lightness model documentation](@ref LIGHT_LIGHTNESS_MODELS).


---


## Hardware requirements @anchor light_lightness_example_hw_requirements

You need at least two supported development kits for this example:

- One nRF52 development kit for the client.
- One or more nRF52 development kits for the servers.

Additionally, you need one development kit for the provisioner
if you decide to use the [static provisioner example](@ref md_examples_provisioner_README).
For details, see [software requirements](@ref light_lightness_example_sw_requirements).

See @ref md_doc_user_guide_mesh_compatibility for the supported development kits.

@note This example uses the PWM peripheral to control the brightness of the LED.
For this reason, it cannot be run on nRF51 devices.


---

## Software requirements @anchor light_lightness_example_sw_requirements

Depending on your choice of the provisioning method:
- If you decide to use the static provisioner example, you need the provisioner example:
`<InstallFolder>/examples/provisioner`
    - See the [Provisioner example](@ref md_examples_provisioner_README) page for more information
    about the provisioner example.
- If you decide to provision using the mobile application, you need to download and install
@link_nrf_mesh_app (available for @link_nrf_mesh_app_ios and @link_nrf_mesh_app_android).


---

## Setup @anchor light_lightness_example_setup

You can find the source code of this example in the following folder:
`<InstallFolder>/examples/light_lightness`


### LED and button assignments @anchor light_lightness_example_setup_leds_buttons

- Server:
  - LED 1: Reflects the value of the Light Lightness Actual state on the server.
  - When interacting with the boards:
    - You cannot use buttons on the server boards, because the Light Lightness Setup Server
    does not use the `simple_hal` module.
    - Instead of the buttons on the server boards, use the following RTT input:
      | RTT input     | DK Button     |   Effect                                                             |
      |---------------|---------------|----------------------------------------------------------------------|
      | `1`           | -             | The brightness of the LED 1 is _decreased_ in large step.            |
      | `2`           | -             | The brightness of the LED 1 is _increased_ in large step.            |
      | `4`           | -             | All mesh data is erased and the device is reset.                     |

- Client:
    - When interacting with the boards, you can use one of the following options:
        - RTT input (recommended): Due to a limited number of buttons on the DK board,
        use the following RTT input when evaluating this example:
            | RTT input     | DK Button     | Effect                                                                                                                                  |
            |---------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------|
            | `1`           | Button 1      | The actual lightness value is _increased_ in large steps and the Light Lightness Set Unacknowledged message is sent.                      |
            | `2`           | Button 2      | The actual lightness value is _decreased_ in large steps and the Light Lightness Set Unacknowledged message is sent.                      |
            | `3`           | Button 3      | The actual lightness value in a linear scale is _increased_ in large steps and Light Lightness Linear Set Unacknowledged message is sent. |
            | `4`           | Button 4      | The actual lightness value in a linear scale is _decreased_ in large steps and Light Lightness Linear Set Unacknowledged message is sent. |
            | `5`           | -             | The Light Lightness Last Get message is sent to request the last lightness value.                                                         |
            | `6`           | -             | The Light Lightness Default Get message is sent to request the default lightness value.                                                   |
            | `7`           | -             | The Light Lightness Range Get message is sent to request the range of supported lightness levels.                                         |
            | `8`           | -             | The Light Lightness Get message is sent to request the actual lightness value.                                                            |
            | `9`           | -             | The Light Lightness Linear Get message is sent to request the actual lightness value in linear scale.                                     |
            | `a`           | -             | The default light lightness value is _increased_ and the Light Lightness Default Set Unacknowledged message is sent.                      |
            | `b`           | -             | The internal default light lightness value is _decreased_ and the Light Lightness Default Set Unacknowledged message is sent.             |
            | `c`           | -             | The internal minimum value of lightness levels range is _increased_ and the Light Lightness Range Set Unacknowledged message is sent.     |
            | `d`           | -             | The internal minimum value of lightness levels range is _decreased_ and the Light Lightness Range Set Unacknowledged message is sent.     |
            | `e`           | -             | The internal maximum value of lightness levels range is _increased_ and the Light Lightness Range Set Unacknowledged message is sent.     |
            | `f`           | -             | The internal maximum value of lightness levels range is _decreased_ and the Light Lightness Range Set Unacknowledged message is sent.     |
            | `g`           | -             | The internal actual lightness value in a linear scale is set to 0 and the Light Lightness Linear Set message is sent.                     |
            | `h`           | -             | Switches the client instance to be used for sending messages.                                                                             |
        - Buttons: If you decide to use the buttons on the DK instead of the RTT input, you can only
        change the Light Lightness Actual state by sending Light Lightness Set Unacknowledged messages.


---

## Testing the example @anchor light_lightness_example_testing

To test the light lightness example, build the examples by following the instructions in
[Building the mesh stack](@ref md_doc_getting_started_how_to_build).

@note
If you have more than 40 boards for the server and decided to use the static provisioner example,
set `MAX_PROVISIONEE_NUMBER` (in `example_network_config.h`) to the number of boards available
and rebuild the provisioner example.

After building is complete, use one of the following methods, depending on the preferred
provisioning approach:
- [Evaluating using the static provisioner](@ref light_lightness_example_testing_dk)
- [Evaluating using the nRF Mesh mobile app](@ref light_lightness_example_testing_app)


### Evaluating using the static provisioner @anchor light_lightness_example_testing_dk

Complete the following steps:
1. Flash the examples by following the instructions in @ref md_doc_getting_started_how_to_run_examples,
including:
    -# Erase the flash of your development boards and program the SoftDevice.
    -# Flash the provisioner and the client firmware on individual boards and the server firmware on other boards.
2. After the reset at the end of the flashing process, press Button 1 on the provisioner board
to start the provisioning process:
    -# The provisioner provisions and configures the client and assigns the address 0x100 to the client node.
    -# The two instances of the Light Lightness client models are instantiated on separate secondary elements.
    For this reason, they get consecutive addresses starting with 0x101.
    -# The provisioner also provisions and configures the servers at random. It assigns them consecutive
    addresses starting with 0x401, and adds them to odd and even groups.
@note - The sequence of provisioned devices depends on the sequence of received unprovisioned beacons.
@note - You can use [RTT viewer](@ref segger-rtt) to view the RTT output generated by the provisioner.
The provisioner prints details about the provisioning and the configuration process in the RTT log.
3. Observe that the LED 1 on the provisioner board is turned ON when provisioner is scanning and provisioning a device.
4. Observe that the LED 2 on the provisioner board is turned ON when configuration procedure is in progress.
5. Wait until LED 1 on the provisioner board remains lit steadily for a few seconds, which indicates that
all available boards have been provisioned and configured.

If the provisioner encounters an error during the provisioning or configuration process for a certain node,
you can reset the provisioner to restart this process for that node.

### Evaluating using the nRF Mesh mobile app @anchor light_lightness_example_testing_app

See @ref nrf-mesh-mobile-app "the information on the main Examples page" for detailed steps required
to provision and configure the boards using the nRF Mesh mobile app.

The following naming convention is used in the app:
- Each server board is `nRF5x Mesh Lightness Setup Server`.
- The client board is `nRF5x Mesh Lightness Client`.

The following model instances must be configured in the app for this example:
- For the `nRF5x Mesh Lightness Setup Server` server boards: Light Lightness Setup Server, Light Lightness Server.
- For the `nRF5x Mesh Lightness Client` client board: Light Lightness Client.

@note The Light Lightness Client example allows to control the Light Lightness states.
For this purpose, it is enough to configure only the Light Lightness Setup Server and Light Lightness Server model 
instances. If you want to see how the binding works between the Light Lightness states and 
the Generic states, configure the generic models instantiated in the Light Lightness Setup Server example
and use the appropriate clients to control the Generic states. 

Once the provisioning is complete, you can start [interacting with the boards](@ref light_lightness_example_testing_interacting).

@note
You can also configure the publish address of the second Light Lightness client model instance.
To do this, repeat [step 3 from binding nodes](@ref nrf-mesh-mobile-app-binding) and 
[all steps from setting publication](@ref nrf-mesh-mobile-app-publication).


### Interacting with the boards @anchor light_lightness_example_testing_interacting

Once the provisioning and the configuration of the client node and of at least one of 
the server nodes are complete, you can press buttons on the client or send command 
numbers using the RTT Viewer to observe the changes in the brightness of the LED 1
on the corresponding server boards.

There is a set of message types available for this demonstration:
- Light Lightness Set Unacknowledged
- Light Lightness Get
- Light Lightness Linear Set
- Light Lightness Linear Set Unacknowledged
- Light Lightness Linear Get
- Light Lightness Default Set Unacknowledged
- Light Lightness Default Get
- Light Lightness Range Set Unacknowledged
- Light Lightness Range Get
- Light Lightness Last Get

See [LED and button assignments](@ref light_lightness_example_setup_leds_buttons) 
section for the full list of available commands.

If any of the devices is powered off and then back on, it will remember its flash configuration
and rejoin the network. It will also restore values of the Light Lightness states.
For more information about the flash manager, see @ref md_doc_user_guide_modules_flash_manager.

#### Controlling the lightness value @anchor light_lightness_example_controlling_lightness_value

You can control the lightness value of the LED 1 using the RTT commands `1` - `4` or the buttons 1 - 4 on the board.
Use the RTT commands `7` and `8` to retrieve the current lightness value in the perceived (Actual) lightness or 
the measured (Linear) lightness value accordingly. 

To set the lightness value to `0`, use the RTT command `g`.

For more information about the difference between the Actual and the Linear lightness values,
see @link_ModelSpec appendix A.2.

#### Changing behavior on power-up @anchor light_lightness_example_changing_behavior_on_powerup

You can change how the lightness value will be restored during a power-up sequence. 
This can be done by controlling the Generic OnPowerUp state instantiated by the Light Lightness Setup Server model.

The following table describes how the lightness value will be restored:
| On PowerUp value | Lightness value          |
|------------------|--------------------------|
| 0                | 0                        |
| 1                | The value of the Light Lightness Default state is used if it is not a zero. Otherwise, the Light Lightness Last state will be used. |
| 2                | Last known value for the Light Lightness Actual before power down.  |

Use the RTT commands `9` and `a` to change the Light Lightness Default state, and 
the RTT commands `4` and `5` to retrieve the current last and default values.
See [LED and button assignments](@ref light_lightness_example_setup_leds_buttons) for additional commands.

The factory default values for these states are controlled through the following defines:
- @ref LIGHT_LIGHTNESS_DEFAULT_ON_POWERUP
- @ref LIGHT_LIGHTNESS_DEFAULT_LIGHTNESS_DEFAULT
- @ref LIGHT_LIGHTNESS_DEFAULT_LIGHTNESS_LAST
- @ref LIGHT_LIGHTNESS_DEFAULT_LIGHTNESS_ACTUAL

If you want to edit the factory default values, do this in `nrf_mesh_config_app.h` of the Light Lightness Setup Server example.
Follow instructions in [Testing the example](@ref light_lightness_example_testing)
to re-build and re-provision the example.

#### Restricting the range of the lightness value @anchor light_lightness_example_changing_lightness_range

You can restrict the range of the lightness value by changing the Light Lightness Range state.
The new value of the Light Lightness Range state will be reflected in the Light 
Lightness Actual state at the next lightness value change.

Use RTT commands `c`, `d`, `e`, and `f` to change the Light Lightness Range state, and 
the RTT command `6` to retrieve the current range.
See [LED and button assignments](@ref light_lightness_example_setup_leds_buttons) for additional commands.

The factory default values for the minimum and maximum possible range values are controlled through
@ref LIGHT_LIGHTNESS_DEFAULT_RANGE_MIN and @ref LIGHT_LIGHTNESS_DEFAULT_RANGE_MAX
values in the `nrf_mesh_config_app.h` file of the Light Lightness Setup Server example. 

#### Other factory default configuration @anchor light_lightness_example_other_factory_default_configuration

In addition to the parameters described in the previous sections, you can also set the factory default 
transition time in milliseconds when changing the lightness levels. To do this, redefine the
@ref LIGHT_LIGHTNESS_DEFAULT_DTT value of the Generic Default Transition Time state
in the `nrf_mesh_config_app.h` file of the Light Lightness Setup Server example.
