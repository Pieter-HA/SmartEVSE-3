![image](https://user-images.githubusercontent.com/36994651/162763664-8f976840-70aa-4684-bcc6-c815cc23e701.png)

# Note
<span style="color:red">
This fork is exploring the capabilities in modifying the Smart-EVSEv3 firmware.<br/>
Feel free to use this repository to build it yourself or to use the latest on from the *releases* folder <b>but this is on your own risk</b>.
</span>
<br />
<br />

# Changes in regards with the original firmware
* New Status page using the Rest API
* Disabled WebSockets
* Reduced max backlight brightness
* Home battery integration
* Endpoint to send L1/2/3 data, this removed the need for a SensorBox
  * Note: Set MainsMeter to the new 'API' option in the config menu when sending L1/2/3
* Callable API endpoints for easy integration (e.g. Home Assistant) - (See [API Overview](#API-Overview))
  * Change charging mode
  * Override charge current
  * Pass in current measurements (p1, battery, ...) - this eliminates having to use additionalhard
  * Switch between single- and three phase power (requires extra 2P relais on the 2nd output)
* Added "Inverted Eastron" kWh, so that polarity is reversed when power is supplied to meter from below (like in most Dutch power panels)
* Added current-limiting functionality if a subpanel is used, example:

                             mains
                               |
                        [main breaker 25A]
                               |
                        [kWh meter "Mains"]
                               |
            -----------------------------------
            |            |                    |
                [group breaker 16A]   [subpanel breaker 16A]
                                              |
                                       [kWh meter "EV"]
                                              |
                                        ----------------
                                        |              |
                            [washer breaker 16A]  [smartevse breaker 16A]

   In this example you configure Mains to 25A, MaxCircuit to 16A; the charger will limit itself so that neither the 25A mains nor the 16A from the subpanel will be
   exceeded...
   Note that for this functionality you will need to be in Smart or Solar mode; it is no longer necessary to enable Load Balancing for this function.

* Added wifi-debugging: if compiled in, you can debug SmartEVSE device by telnetting to it over your wifi connection
* Small code optimisations, fixed some small bugs



# New Status Page
![image](https://user-images.githubusercontent.com/36994651/160653707-121dd618-ee0d-4cb3-bc39-82fde1a1a653.png)


# Home Battery Integration
In a normal EVSE setup a sensorbox is used to read the P1 information to deduce if there is sufficient solar energy available. This however can give unwanted results when also using a home battery as this will result in one battery charging the other one. <br/>

For this purpose the settings endpoint allows you to pass through the battery current information:
* A positive current means the battery is charging
* A negative current means the battery is discharging

The EVSE will use the battery current to neutralize the impact of a home battery on the P1 information.<br>
**Regular updates from the consumer are required to keep this working as values cannot be older than 60 seconds.**

### Example
* Home battery is charging at 2300W -> 10A
* P1 has an export value of 230W -> -1A
* EVSE will neutralize the battery and P1 will be "exporting" -11A

The sender has several options when sending the home battery current:
* Send the current AS-IS -> EVSE current will be maximized
* Only send when battery is discharging -> AS-IS operation but EVSE will not discharge the home battery
* Reserve an amount of current for the home battery (e.g. 10A) -> Prioritize the home battery up to a specific limit

# API Overview
View API <a href="https://swagger-ui.serkri.be/" target="_blank">https://swagger-ui.serkri.be/</a>


Have an idea for the API? Edit it here <a href="https://swagger-editor.serkri.be/" target="_blank">https://swagger-editor.serkri.be/</a> and copy/paste it in a new issue with your request (https://github.com/serkri/SmartEVSE-3/issues)

# Building the firmware

* Install platformio-core https://docs.platformio.org/en/latest/core/installation/methods/index.html
* Clone this github project, cd to the smartevse directory where platformio.ini is located
* Compile firmware.bin: platformio run
* Compile spiffs.bin: platformio run -t buildfs

If you are not using the webserver /update endpoint:
* Upload via USB configured in platformio.ini: platformio run --target upload
