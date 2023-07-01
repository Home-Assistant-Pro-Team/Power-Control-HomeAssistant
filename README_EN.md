# Power-Control

`- Version: 1.3 -`

The package was written in Italian, but can be easily translated. As for the translation of the README, we relied on automatic translators and apologize for any translation errors.


Power control is a system for managing the power supply efficiently, preventing overloads and power outages. Its main objective is to regulate power consumption by turning off devices according to the required power, thus preventing exceeding the set limits and ensuring the stability of the electrical system. In addition, the system is able to turn previously turned off devices back on when consumption returns to normal.


https://github.com/Home-Assistant-Pro-Team/Power-Control/assets/62516592/9c10e918-f4c3-4458-83e7-35cc3fe0f49f

It is important to have a clear idea of how the electricity supply works in most cases. Take, for example, a 3 kW contract: the actual power available will be 3.3 kW, which includes a 10% reserve to handle any peak consumption. However, higher power can be used temporarily with a 33% tolerance, which equals 3,990 W, for up to 180 minutes. During this period, three checks are made to verify compliance with the power limits.

| Potenza  | Effettiva 	| Tolleranza 33% 	|
| -------- | --- 	 	| ----------- 		|
| 3.0 kW   | 3.3 kW  	| 3990 W   			|
| 4.5 kW   | 4.9 kW  	| 5980 W  			|
| 6.0 kW   | 6.6 kW  	| 7980 W  			|


Now, with this clear information in mind, we can properly set up the power control package. What makes this package special is its versatility: the devices that can be included in the control list can belong to different domains, such as switch, light, fan, climate, media_player, etc. There is no limit to the number of devices that can be included in the list. During the device checklist process, specific checks are performed to determine whether a device is actually in use. A check is made to see whether the device has a power consumption sensor (device_class power) and, if so, whether the consumption exceeds the 15 W threshold. This check identifies whether the device is actually drawing power. In the case where a device does not have a power consumption sensor, only the state of the device itself is considered to determine whether it is on or off.

The operation of the system is as follows: two minutes before reaching the set threshold for power disconnection, a high consumption warning notification is sent to allow manual management by the user. After the two minutes have elapsed and if consumption still remains high, the devices are turned off in sequential order from the list, with a 20-second interval between each. During each shutdown phase, a notification is sent to stay informed about the status of the process. In the event that consumption remains high but there are no devices turned on in the list, a notification is sent to alert the user of the event.

If the emergency threshold is exceeded, the system acts immediately without considering the elapsed time. Initially, it is checked whether, in the last 30 seconds, a device on the list has been turned on, also taking into account its power consumption. If an active device is detected, it is immediately turned off and temporarily excluded from load restoration. Conversely, if no active devices have been detected in the last 30 seconds, devices are turned off in sequential order as in the case of exceeding the normal threshold.

Once power consumption falls below the detachment threshold for the preset time, the system starts the restoration process. Devices are reactivated in reverse order of the order in which they were turned off, with notifications sent to inform the user of the restoration status. Specific devices can be excluded from restoration by selecting them through a dedicated selection menu. When the restore process is complete, a final notification is sent to confirm that all devices have been successfully reactivated and power consumption is back to normal.

NB: Push notifications will be sent only to people in the home.


### **Requisiti packages**
- [HomeAssitant release 2023.4 ](https://www.home-assistant.io/blog/2023/04/05/release-20234/)
- [Cartella Package abilitata](https://www.home-assistant.io/docs/configuration/packages/)
- General home consumption control device es shelly em.

### **Installazione:**
For installation, you need to load the "custom_templates" folder in the "conf" directory of Home Assistant. If the folder already exists, you only need to copy the files into it.

- **personal.jinja**

    This file will be used for other projects within this GitHub repository. In the file, we set up personal data that will be used in all projects. Simply enter your entities respecting the JSON indentation.

	Let's see how to customize it. Although not all information is required for this package, it is advisable to fill in all fields to take full advantage of it in other projects.

	In this section, we define the entities and sensors for each person. If you do not want to associate a cell phone number or an alarm sensor with a specific person, simply assign the value "none" in the corresponding section of the file. To add or remove people from the dictionary list, you need to pay attention to the JSON syntax.

	```
	{% macro persons() %}
	[
		{
			"person": "person.marco",
			"battery": "sensor.cellulare_marco_battery_level",
			"notify": "mobile_app_cellulare_marco",
			"sveglia": "sensor.cellulare_marco_prossimo_allarme",
			"cellulare": "331000000"
		},
		{
			"person": "person.serena",
			"battery": "sensor.cellulare_serena_livello_della_batteria",
			"notify": "mobile_app_samsung_s21",
			"sveglia": "none",
			"cellulare": "335000000"
		}
	]
	{% endmacro%}
	```
    In this section, we will list our media players used for notifications. Be sure to correctly enter the selected media players for Alexa and TTS notifications (e.g., Google), carefully following the correct syntax.

	```
	{% macro media_players(type) %}
		{% set list_media = 
			[
				'media_player.camera',
				'media_player.studio',
				'media_player.googlehome_cameretta',
				'media_player.googlehome_bagno',
				'media_player.googlehome_cucina',
				'media_player.googlehome_salone'
			]
		%}
		{% for integrations in integration_entities(type) if integrations in list_media %}
			{{ integrations }}
		{% endfor %}
	{% endmacro %}
	```

- **power_control.jinja**

    In this file, simply enter your entities of the devices you wish to turn off, in order of priority sequence. The first device in the list will be the first to be turned off (if turned on), followed by the second device, and so on.

	```
	{% macro entities_control() %}
		{% set list_entities = 
			[
				'switch.idromassaggio',
				'climate.camera_ac',
				'climate.salotto',
				'climate.condizionatore_salone',
				'switch.lavatrice1pm',
				'switch.forno'
			]
		%}
		{% for d in list_entities %}
			{{ d }}
		{% endfor %}
	{% endmacro %}
	```

	In addition, it is necessary to insert the sensor that measures the instantaneous general household absorption, expressed in watts (W).


	```
	{% macro power_control() %}
		{
			"Sensore W": "sensor.shelly_em_assorbimento_casa"
		}
	{% endmacro %}
	```

At this point, you can load the power_control folder in the "packages" directory and restart Home Assistant.

### **Card:**

#### **Requisiti card:**

- Sensore power (W) incluso nel [recorder](https://www.home-assistant.io/integrations/recorder/)
- [Browser mode](https://github.com/thomasloven/hass-browser_mod) 
- [Button card](https://github.com/custom-cards/button-card)
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)
- [Apexcharts card](https://github.com/RomRider/apexcharts-card)
- [Mushroom](https://github.com/piitaya/lovelace-mushroom)


#### **Installazione card:**
To use the card, we need to follow some simple steps. We start by creating a new manual card and copy the contents of the file "card.txt" and enter the power sensor used previously in the power_meter variable.

![variables](example/variables.png)

#### **Spiegazione card:**
It is possible to turn load control on or off by pressing the button.

![power_on](example/power_on.png)	![power_off](example/power_off.png)


The card has 4 buttons at the bottom, each of which has a specific function. Below is the order and description of each button:

![button](example/button.png)


1) In the popup you can exclude devices from load restoration: This option allows you to specify one or more devices to be excluded from automatic load restoration. In other words, the selected devices will not be automatically turned back on after an interruption for load shedding. In addition, the devices entered in load control are displayed.


![list](example/list.png)

2) Questo popup viene utilizzato per configurare diverse impostazioni relative al sistema di gestione dei carichi. Ecco una spiegazione più dettagliata delle opzioni disponibili:

	- Ritardo Distacco Carico:  Load Detachment Delay: This option allows you to specify a waiting time, expressed in minutes, after which load detachment will be initiated. When the specified time elapses, the system will start disconnecting the connected devices.

	- Distacco carico: This setting requires a value, expressed in watts (W), which is the threshold above which the system will perform the disconnection of appliances. If the total power of the loads exceeds this value, the system will initiate the disconnection of the devices.

	- Ripristino Carico: This option allows you to set a value expressed in watts (W), which is the threshold below which devices will be automatically reactivated. The system will start the reactivation process of the previously deactivated devices with an interval of 20 seconds between each reactivation, provided that the instantaneous absorption is less than the set value.

	- Distacco Urgente:  This value should be higher than "Load Detachment" and allows the last appliance in the list to be turned off immediately if it is turned on within 30 seconds. On the other hand, if the last appliance is not turned on within this time interval, the order of the list for detaching the other loads will be respected.


![setting](example/setting.png)

3) This option enables push notifications that will be sent only to people in the home.

4) This function enables notifications for alexa media players and TTS (e.g., Google)

### **Contributi**
This project is open for contributions. If you would like to provide feedback, report a bug, or request a new feature, please create an issue on the repository.


### **Supportaci**
If you enjoyed this project, we would love to have your support. Even a simple coffee can make a difference. 
The funds raised will be used to purchase new equipment and carry out new projects. You can contribute by clicking on the button below. 
Thank you very much for your support!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M1MI00I)

### **Changelog**

#### **Version: 1.3:**

	- Removed two (non-blocking) errors from the log that were appearing when HomeAssistant was starting up
		```

			{% set a = states('sensor.check_ultimo_acceso').split()|default %}
			{% set b = state_attr('sensor.check_ultimo_acceso', 'old_state').split()|default %}
			
				.................


			{% if state_attr('timer.distacco_carico', 'finishes_at') is not none %}
				{{ now() > as_datetime(state_attr('timer.distacco_carico', 'finishes_at')) - timedelta(minutes=2) }}
			{% else %}
				False
			{% endif %}
		```
	- The logic of restarting loads has been modified. Now, instead of relying only on the disconnection threshold, a reactivation threshold has been added to avoid unwanted continuous cycling. In addition, a fixed time of 20 seconds has been set for power-up between loads.