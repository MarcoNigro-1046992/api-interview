# Esercizio su Stanza del Cittadino

OpencityItalia è un [software opensource, rilasciato sul catalogo del software per le PA italiane](https://developers.italia.it/it/software/opencontent-stanza-del-cittadino-core-410a6e), usato da oltre 250 enti italiani per l'erogazione di servizi online. Il sistema è multi-tenant: una sola istanza dell'applicativo è utilizzata da molti enti, ognuno dei quali implementa in modo indipendente i proprio servizi, gestisce le proprie pratiche etc.

Il presente esercizio è utilizzato durante i colloqui per l'assunzione del personale, come spunto di discussione per un colloquio tecnico. **Non** è ovviamente richiesta la realizzazione di un software completamente funzionante, interamente implementato, pronto da mettere in produzione: si richiede solo di lavorare alcune ore su un problema reale, per poterci poi ragionare sopra durante un colloquio tecnico. Ad esempio è perfettamente accettabile realizzare solo un sistema di raccolta dei dati (back-end) e trascurare del tutto il frontend, oppure dare per scontato che esista un'API che fornisce quanto necessario a fare una pagina web che interroga un JSON statico o un'API costruita ad-hoc, trascurando come questi dati vengano raccolti.

## Obiettivo

Realizzare una pagina web separata dall'applicativo che elenca i servizi erogati dalla stanza del cittadino in modo trasversale sui suoi tenant. Si può assumere che i servizi che si chiamano nello stesso modo siano lo stesso servizio erogato da due enti distinti. 
E' disponibile anche una indicazione di quanto un servizio è utilizzato, per cui è possibile ordinare i servizi dando maggior evidenza a quelli più utilizzati.

## Tempo necessario

Un paio d'ore sono sicuramente necessarie per realizzare qualcosa di minimale. Se ci si vuole spingere oltre e fare qualcosa di più completo probabilmente 4-6h. Più di una giornata non è decisamente sensato lavorarci, perché il tempo necessario per discuterne successivamente (1h) non sarebbe sufficiente per valorizzare quanto fatto.

Il candidato è quindi **libero** di semplificare il proprio compito come preferisce, sarà utile magari dare evidenza delle proprie scelte in un file di testo inserito nel repository stesso. Per esempio si può decidere di lavorare sui dati, immaginando un sistema di aggregazione, scraping della sorgente e trascurare del tutto il frontend. Oppure ignorare del tutto il database e fare un frontend che attinge a una fonte di dati statica (un file json nel repo stesso) realizzata partendo dai dati disponibili nelle varie API. In sostanza è lecito immaginare di essere parte di un team multidisciplinare in cui ognuno da il suo contributo in base alle proprie competenze.

 
## Risultato atteso

Completare un progetto, anche se piccolo, richiede comunque la cura di molti aspetti: un ragionamento sull'[architettura](https://martinfowler.com/architecture/) più conveniente per risolvere il problema, la scelta degli strumenti (un db? una soluzione storage-less, serve un'API? delle cache?), va scritto del codice, va organizzato e versionato, va ipotizzato un deploy (docker o configuration management? CI/CD?), va deciso come verrà monitorato in produzione (healthcheck, [metriche](https://www.slideshare.net/roidelapluie/prometheus-from-technical-metrics-to-business-observability), [logging](https://brennonloveless.medium.com/logging-v-monitoring-5f234d4edbd7)), etc. Molti aspetti diversi che richiedono ognuno tempo e conoscenze. Nello svolgimento dell'esercizio il candidato è libero di **scegliere** su cosa concentrarsi: il codice, un front-end, un'API, una cosa minimale ma ben dockerizzata e monitorabile, ogni soluzione è accettabile, anche solo un `README.md`, o un qualunque mix di questi.

L'unico requisito è che si produca qualcosa che possa essere inserito in un repository Git.

## Modalità di svolgimento

Il candidato deve fare un _fork_ del presente repository e committare il suo codice nella propria copia del repository. In caso di domande è possibile aprire una issue sul presente repository.

## Informazioni utili

I servizi sono l'elemento principale di cui vogliamo dare evidenza in questa pagina oggetto dell'esercizio.

Un ente ipotetico - Comune di Bugliano (PI) - ha un servizio di notifica del contatore dell'acqua di ogni abitazione: https://www2.stanzadelcittadino.it/comune-di-bugliano/it/servizi/richiesta-tessera-elettorale

Se avete SPID potete anche inviare realmente dei dati, si tratta di un ente di test e vi può essere utile per capire meglio il funzionamento del sistema.

### Reperimento informazioni sui servizi (crawling)

I dati di questo servizio sono disponibili via API a questo indirizzo:

https://www2.stanzadelcittadino.it/comune-di-bugliano/api/services/de041f18-e6b4-4aa6-a116-2c597646cef7

La form che avete eventualmente compilato è definita da questi due parametri (non c'e' alcun motivo per cui non è una semplice URL, è semplicemente così :-) )

```
[...]
        "formio_id": "60d064b100ecb10010fec9d5",
        "url": "https://form.stanzadelcittadino.it/form/"
[...]
```

Più semplicemente: https://form.stanzadelcittadino.it/form/60d064b100ecb10010fec9d5

Altri dati che può essere utile conoscere da questi documenti JSON: 
 * se è attivo o meno (status = 1), cioè se i cittadini possono o meno inviare pratiche
 * se richiede un pagamento o meno (payment_required = true)
 * la categoria a cui appartiene (topics)

I servizi da considerare sono solo quelli di ultima generazione, vanno eslusi i servizi _legacy_ che si riconoscono facilmente perché il valore di `"flow_steps"` è nullo.

Un altro dato che può essere utile raccogliere è il numero di pratiche presentate per ogni servizio. Nel comune dell'esempio questa informazioni è presente all'indirizzo

https://www2.stanzadelcittadino.it/comune-di-bugliano/metrics

L'indirizzo è disponibile per ogni tenant: le pratiche attraversano molti stati, quelli finali che ci interessa considerare sono "Iter completato" (equivale a una accettazione) e "Rifiutata".

Come si sarà capito non esiste un'API comoda per realizzare questo compito perché le API sono solo a livello di tenant al momento, ma si vuole realizzare la pagina per verificare l'interesse che può suscitare, partendo dai dati disponibili ad oggi. Le api di ogni tenant sono pubblicate a questo indirizzo: 

https://www2.stanzadelcittadino.it/comune-di-bugliano/api/doc

E' possibile avere l'elenco dei tenant della stanza del cittadino a questo indirizzo:

https://www2.stanzadelcittadino.it/prometheus.json

ogni istanza ha un indirizzo di base dato da ```job + __metrics_path_``` togliendo `/metrics` alla fine della stringa.

L'elenco dei servizi pubblicati di ogni ente è disponibile via API:

```
http GET https://www2.stanzadelcittadino.it/$ENTE/api/services
```

### Come rappresentare le informazioni

La rappresentazione può essere fatta in modo molto semplice, prendendo spunto dalla [pagina attuale dei servizi di ogni singolo tenant](https://www2.stanzadelcittadino.it/comune-di-bugliano/servizi/) o da questi esempi che usano [bootstrap-italia](https://italia.github.io/bootstrap-italia/):

![image](https://gitlab.com/opencontent/stanzadelcittadino/-/wikis/uploads/4a764e51118e470265c029f68f5c2f7a/image.png)

![image](https://gitlab.com/opencontent/stanzadelcittadino/-/wikis/uploads/588ae8b3d6ebafbb619bdf9bf5bb0962/image.png)

Se si preferisce, sono disponibili anche [temi bootstrap-italia per React e altri framework](https://github.com/italia/), ma in realtà solo quello di react è completo, gli altri non lo sono.

