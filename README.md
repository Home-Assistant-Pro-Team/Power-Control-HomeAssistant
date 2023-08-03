# Power-Control

`- Version: 1.4 -`


Il power control è un sistema che permette di gestire l'alimentazione elettrica in modo efficiente, prevenendo sovraccarichi e interruzioni di corrente. Il suo obiettivo principale è regolare il consumo di energia spegnendo i dispositivi in base alla potenza richiesta, evitando così il superamento dei limiti imposti e garantendo la stabilità del sistema elettrico. Inoltre, il sistema è in grado di riaccendere i dispositivi precedentemente spenti quando il consumo torna nella norma.


https://github.com/Home-Assistant-Pro-Team/Power-Control/assets/62516592/9c10e918-f4c3-4458-83e7-35cc3fe0f49f

È importante avere un'idea chiara di come funziona la fornitura elettrica nella maggior parte dei casi. Prendiamo ad esempio un contratto di 3 kW: la potenza effettivamente disponibile sarà di 3,3 kW, che include una riserva del 10% per gestire eventuali picchi di consumo. Tuttavia, è possibile utilizzare temporaneamente una potenza superiore con una tolleranza del 33%, che equivale a 3.990 W, per un massimo di 180 minuti. Durante questo periodo, vengono effettuati tre controlli per verificare il rispetto dei limiti di potenza.

| Potenza  | Effettiva 	| Tolleranza 33% 	|
| -------- | --- 	 	| ----------- 		|
| 3.0 kW   | 3.3 kW  	| 3990 W   			|
| 4.5 kW   | 4.9 kW  	| 5980 W  			|
| 6.0 kW   | 6.6 kW  	| 7980 W  			|


Ora, con queste informazioni chiare in mente, possiamo impostare correttamente il pacchetto di power control. Ciò che rende questo pacchetto particolare è la sua versatilità: i dispositivi che possono essere inclusi nella lista di controllo possono appartenere a diversi domini, come switch, light, fan, climate, media_player, ecc. Non ci sono limiti al numero di dispositivi che possono essere inseriti nella lista. Durante il processo di controllo dei dispositivi, vengono eseguiti controlli specifici per determinare se un dispositivo è effettivamente in uso. Si verifica se il dispositivo dispone di un sensore di consumo energetico (device_class power) e, in tal caso, se il consumo supera la soglia di 15 W. Questo controllo consente di identificare se il dispositivo sta effettivamente assorbendo energia. Nel caso in cui un dispositivo non disponga di un sensore di consumo energetico, viene considerato solo lo stato del dispositivo stesso per determinare se è attivo o spento.

Il funzionamento del sistema è il seguente: due minuti prima di raggiungere la soglia impostata per il distacco di potenza, viene inviata una notifica di avviso di consumo elevato per permettere una gestione manuale da parte dell'utente. Trascorsi i due minuti e se il consumo rimane ancora elevato, i dispositivi vengono spenti in ordine sequenziale dalla lista, con un intervallo di 20 secondi tra uno e l'altro. Durante ciascuna fase di spegnimento, viene inviata una notifica per restare informato sullo stato del processo. Nel caso in cui il consumo rimanga elevato ma non risultino dispositivi accesi nella lista, viene inviata una notifica per avvisare l'utente dell'evento.

Se si supera la soglia di emergenza, il sistema agisce immediatamente senza considerare il tempo trascorso. Inizialmente, viene controllato se, negli ultimi 30 secondi, è stato acceso un dispositivo presente nella lista, tenendo in considerazione anche il suo assorbimento energetico. Se viene rilevato un dispositivo attivo, viene immediatamente spento e temporaneamente escluso dal ripristino del carico. Al contrario, se non sono stati rilevati dispositivi attivi negli ultimi 30 secondi, i dispositivi vengono spenti in ordine sequenziale come nel caso del superamento della soglia normale.

Una volta che il consumo energetico scende sotto la soglia di distacco per il tempo preimpostato, il sistema avvia il processo di ripristino. I dispositivi vengono riattivati in ordine inverso rispetto all'ordine in cui sono stati spenti, con notifiche inviate per informare l'utente sullo stato del ripristino. È possibile escludere specifici dispositivi dal ripristino selezionandoli attraverso un menu di selezione dedicato. Al termine del processo di ripristino, viene inviata un'ultima notifica per confermare che tutti i dispositivi sono stati correttamente riattivati e che il consumo energetico è tornato nella norma.

NB: Le notifiche push verranno inviate solo alle persone che si trovano in casa.


### **Requisiti packages**
- [HomeAssitant release 2023.4 ](https://www.home-assistant.io/blog/2023/04/05/release-20234/)
- [Cartella Package abilitata](https://www.home-assistant.io/docs/configuration/packages/)
- Dispositivo per controllo consumo generale casa es shelly em. 

### **Installazione:**
Per l'installazione, è necessario caricare la cartella "custom_templates" nella directory "conf" di Home Assistant. Se la cartella esiste già, è sufficiente copiare al suo interno solo i file.

- **personal.jinja**

	Questo file sarà utilizzato per altri progetti all'interno di questo repository GitHub. Nel file, impostiamo dati personali che verranno utilizzati in tutti i progetti. È sufficiente inserire le proprie entità rispettando l'indentazione JSON.

	Vediamo come personalizzarlo. Anche se non tutte le informazioni sono necessarie per questo pacchetto, è consigliabile compilare tutti i campi per poter sfruttarlo appieno in altri progetti.

	In questa sezione, definiamo le entità e i sensori per ogni persona. Se non si desidera associare un numero di cellulare o un sensore di sveglia a una persona specifica, è sufficiente assegnare il valore "none" nella sezione corrispondente del file. Per aggiungere o rimuovere persone dalla lista di dizionari, è necessario prestare attenzione alla sintassi JSON.

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
	In questa sezione, elencheremo i nostri media player utilizzati per le notifiche. Assicurati di inserire correttamente i media player selezionati per le notifiche Alexa e TTS (ad esempio, Google), seguendo attentamente la sintassi corretta.

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

	In questo file, è sufficiente inserire le proprie entità dei dispositivi che si desidera spegnere, in ordine di sequenza di priorità. Il primo dispositivo nella lista sarà il primo ad essere spento (se acceso), seguito dal secondo dispositivo e così via.

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

	Inoltre, è necessario inserire il sensore che misura l'assorbimento istantaneo generale di casa, espresso in watt (W).


	```
	{% macro power_control() %}
		{
			"Sensore W": "sensor.shelly_em_assorbimento_casa"
		}
	{% endmacro %}
	```

A questo punto, puoi caricare la cartella power_control nella directory "packages" e riavviare Home Assistant.

### **Card:**

#### **Requisiti card:**

- Sensore power (W) incluso nel [recorder](https://www.home-assistant.io/integrations/recorder/)
- [Browser mode](https://github.com/thomasloven/hass-browser_mod) 
- [Button card](https://github.com/custom-cards/button-card)
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)
- [Apexcharts card](https://github.com/RomRider/apexcharts-card)
- [Mushroom](https://github.com/piitaya/lovelace-mushroom)


#### **Installazione card:**
Per utilizzare la card, è necessario seguire alcuni semplici passaggi. Iniziamo creando una nuova scheda manuale e copiamo il contenuto del file "card.txt" ed inseriamo il sensore power utilizzato in precedenza nella variabile power_meter.

![variables](example/variables.png)

#### **Spiegazione card:**
E' possibile attivare o disattivare il controllo carichi premendo il pulsante.

![power_on](example/power_on.png)	![power_off](example/power_off.png)


La card presenta 4 pulsanti nella parte bassa, ognuno dei quali ha una specifica funzione. Qui di seguito l'ordine e la descrizione di ciascun pulsante:

![button](example/button.png)


1) Nel popup è possibile escludere dispositivi dal ripristino carico: Questa opzione consente di specificare uno o più dispositivi da escludere dal ripristino automatico dei carichi. In altre parole, i dispositivi selezionati non verranno riaccensi automaticamente dopo un'interruzione per il distacco carichi. Inoltre vengo visualizzati i dispositivi inseriti nel controllo dei carichi. 


![list](example/list.png)

2) Questo popup viene utilizzato per configurare diverse impostazioni relative al sistema di gestione dei carichi. Ecco una spiegazione più dettagliata delle opzioni disponibili:

	- Ritardo Distacco Carico:  Ritardo Distacco Carico: Questa opzione consente di specificare un tempo di attesa, espresso in minuti, dopo il quale verrà avviato il distacco dei carichi. Quando il tempo specificato trascorre, il sistema inizierà a disattivare i dispositivi collegati.

	- Distacco carico: Questa impostazione richiede un valore, espresso in watt (W), che rappresenta la soglia oltre la quale il sistema eseguirà il distacco degli elettrodomestici. Se la potenza totale dei carichi supera questo valore, il sistema avvierà il distacco dei dispositivi.

	- Ripristino Carico: Questa opzione consente di impostare un valore espresso in watt (W), che rappresenta la soglia al di sotto della quale i dispositivi verranno riattivati automaticamente. Il sistema avvierà il processo di riaccensione dei dispositivi precedentemente disattivati con un intervallo di 20 secondi tra una riaccensione e l'altra, a condizione che l'assorbimento istantaneo sia inferiore al valore impostato.

	- Distacco Urgente:  Questo valore deve essere più alto rispetto al "Distacco Carico" e consente di spegnere immediatamente l'ultimo elettrodomestico presente nella lista se è acceso entro 30 secondi. Se invece l'ultimo dispositivo non viene acceso entro tale intervallo di tempo, verrà rispettato l'ordine della lista per il distacco degli altri carichi.


![setting](example/setting.png)

3) Questa opzione consente di attivare le notifiche push che verranno inviate solo alle persone che si trovano in casa.

4) Questa funzione permette di abilitare le notifiche per i media player alexa e TTS (es. Google)

### **Contributi**
Questo progetto è aperto ai contributi. Se vuoi fornire feedback, segnalare un bug o richiedere una nuova funzionalità, ti invitiamo a creare una issue sul repository.


### **Supportaci**
Se hai apprezzato questo progetto, ci piacerebbe avere il tuo supporto. Anche un semplice caffè può fare la differenza. 
I fondi raccolti saranno utilizzati per acquistare nuovo materiale e realizzare nuovi progetti. Puoi contribuire cliccando sul pulsante qui sotto. 
Grazie di cuore per il tuo sostegno!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M1MI00I)

### **Changelog**

#### **Version: 1.3:**

- Eliminati due errore (non bloccanti) dal log che apparivano nel momento dell'avvio di HomeAssistant
 
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
  
- La logica di riaccensione dei carichi è stata modificata. Ora, anziché basarsi solo sulla soglia di distacco, è stata aggiunta una soglia di riattivazione per evitare cicli continui indesiderati. Inoltre, è stato impostato un tempo fisso di 20 secondi per l'accensione tra un carico e l'altro.
	
#### **Version: 1.4:**
	
- Risolto limite dei 255 caratteri per la lista dispositivi da spengnere
