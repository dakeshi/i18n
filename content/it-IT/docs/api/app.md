# app

> Controlla il ciclo di vita degli eventi della tua applicazione.

Processo: [Main](../glossary.md#main-process)

I seguenti esempi mostrano come uscire dall'applicazione quando l'ultima finestra è chiusa:

```javascript
const {app} = require('electron')
app.on('window-all-closed', () => {
  app.quit()
})
```

## Eventi

L'oggetto `app` emette i seguenti eventi:

### Evento: 'will-finish-launching'

Emesso quando l'app ha finito l'avvio di base. Su Windows e Linux, l'evento `will-finish-launching` equivale all'evento `ready`; su macOS questo evento rappresenta la notifica `applicationWillFinishLaunching` di `NSApplication`. Potresti necessitare spesso di definire ascoltatori (listener) per gli eventi `open-file` e `open-url` ed avviare il reporter dei crash e l'aggiornamento automatico.

In gran parte dei casi, dovresti solo fare tutto nel gestore dell'evento `ready`.

### Evento: 'ready'

Restituisce:

* `launchInfo` Object *macOS*

Emesso quando Electron ha concluso l'inizializzazione. Su macOS `launchInfo` detiene le `userInfo` della `NSUserNotification` usata per aprire l'applicazione, se lanciata dal Centro Notifiche. Puoi chiamare `app.isReady()` per controllare se gli eventi sono già stati generati.

### Evento: 'window-all-closed'

Emesso quando tutte le finestre sono state chiuse.

Se l'evento non viene gestito e tutte le finestre vengono chiuse, il comportamento predefinito è l'uscita dall'applicazione; però, se gestito, è possibile controllare il caso in cui l'applicazione deve uscire o meno. Se l'utente ha premuto `Cmd + Q` o lo sviluppatore ha invocato `app.quit()`, Electron proverà prima a chiudere tutte le finestre e poi emetterà l'evento `will-quit` e in questo caso l'evento `window-all-closed` non sarà emesso.

### Evento: 'before-quit'

Restituisce:

* `event` Event

Emesso prima che l'applicazione inizi a chiudere le sue finestre. Chiamare `event.preventDefault()` impedirà il comportamento predefinito, ovvero la terminazione l'applicazione.

**Note:** Se l'uscita dall'applicazione è avviata da `autoUpdater.quitAndInstall()` allora `before-quit` è emesso *dopo* aver emesso l'evento `close` su tutte le finestre e chiudendole.

### Evento: 'will-quit'

Restituisce:

* `event` Event

Emesso quando tutte le finestre sono state chiuse e l'applicazione uscirà. Chiamare `event.preventDefault()` impedirà il comportamento predefinito, ovvero la terminazione l'applicazione.

Vedi la descrizione dell'evento `window-all-closed` per le differenze tra gli eventi `will-quit` e `window-all-closed`.

### Evento: 'quit'

Restituisce:

* `event` Event
* `codiceUscita` Integer

Emesso quando l'applicazione è in uscita.

### Evento: 'open-file' *macOS*

Restituisce:

* `event` Event
* `path` String

Emesso quando l'utente vuole aprire un file con l'app. L'evento `open-file` è in genere emesso quando l'applicazione è già aperta e l'S.O. vuole riutilizzarla per aprire il file. `open-file` è anche emesso quando un file è rilasciato nel dock e l'applicazione non è ancora in esecuzione. E' necessario assicurarsi di ascoltare l'evento `open-file` molto presto all'avvio della tua app per gestire questo caso (anche prima dell'emissione dell'evento `ready`).

Dovresti chiamare `event.preventDefault()` se vuoi gestire questo evento.

Su Windows, devi analizzare `process.argv` (nel processo principale) per ottenere il percorso del file.

### Evento: 'open-url' *macOS*

Restituisce:

* `event` Event
* `url` Stringa

Emesso quando l'utente vuole aprire un URL con l'l'applicazione. Il file `Info.plist` della tua applicazione deve definire lo schema URL compreso della chiave `CFBundleURLTypes` ed impostare `NSPrincipalClass` ad `AtomApplication`.

Dovresti chiamare `event.preventDefault()` se vuoi gestire questo evento.

### Evento: 'activate' *macOS*

Restituisce:

* `event` Event
* `hasVisibleWindows` Boolean

Emesso quando l'applicazione è attivata. Varie azioni possono generare questo evento, come il lancio dell'applicazione per la prima volta, provare a rilanciarla quando è già aperta o cliccare sul dock dell'applicazione o sull'icona della taskbar.

### Evento: 'continue-activity' *macOS*

Restituisce:

* `event` Event
* `type` String - Una stringa che identifica l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `userInfo` Object - Contiene stati specifici dell'applicazione immagazzinati per attività su un altro dispositivo.

Emesso durante [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) quando un'attività da un altro dispositivo vuole essere ripristinata. Dovresti chiamare `event.preventDefault()` se vuoi gestire questo evento.

Un'attività dell'utente può essere continuata solo in un app con lo stesso developer Team ID come l'attività dell'app di riferimento e che supporti il tipo di attività. I tipi di attività supportati sono specificati nell'`Info.plist` dell'applicazione sotto la chiave `NSUserActivityTypes`.

### Evento: 'will-continue-activity' *macOS*

Restituisce:

* `event` Event
* `type` String - Una stringa che identifica l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).

Emesso durante [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html), prima che un'attività da un dispositivo differente richieda di essere ripristinata. Dovresti chiamare `event.preventDefault()` se vuoi gestire questo evento.

### Evento: 'continue-activity-error' *macOS*

Restituisce:

* `event` Event
* `type` String - Una stringa che identifica l'l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `error` String - Una stringa contenente la descrizione localizzata dell'errore.

Emesso durante [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) quando un'attività da un dispositivo diverso fallisce nel ripristino.

### Evento: 'activity-was-continued' *macOS*

Restituisce:

* `event` Event
* `type` String - Una stringa che identifica l'l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `Infoutente` Object - Contiene uno stato specifico per l'app archiviato dall'attività.

Emesso durante [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) dopo che un'attività da questo dispositivo è stata ripristinata con successo su un altro.

### Evento: 'update-activity-state' *macOS*

Restituisce:

* `event` Event
* `type` String - Una stringa che identifica l'l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `Infoutente` Object - Contiene uno stato specifico per l'app archiviato dall'attività.

Emesso quando [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) sta per essere ripristinato su un altro dispositivo. Se necessiti aggiornare lo stato da trasferire, potresti chiamare `evento.preventDefault()` immediatamente, costruirne un nuovo dizionario `userInfo` e chiamare `app.updateCurrentActiviy()` tempestivamente. Altrimenti l'operazione fallirà e sarà chiamato l'`continue-activity-error`.

### Evento: 'new-window-for-tab' *macOS*

Restituisce:

* `event` Event

Emesso quando l'utente clicca il pulsante macOS nativo nuova scheda. Il pulsante nuova scheda è visibile solo se l'attuale `BrowserWindow` ha un `tabbingIdentifier`

### Evento: 'browser-window-blur'

Restituisce:

* `event` Event
* `window` [BrowserWindow](browser-window.md)

Emesso quando una [browserWindow](browser-window.md) è sfocata.

### Evento: 'browser-window-focus'

Restituisce:

* `event` Event
* `window` [BrowserWindow](browser-window.md)

Emesso quando una [browserWindow](browser-window.md) è focalizzata.

### Evento: 'browser-window-created'

Restituisce:

* `event` Event
* `window` [BrowserWindow](browser-window.md)

Emesso quando una [browserWindow](browser-window.md) è creata.

### Evento: 'web-contents-created'

Restituisce:

* `event` Event
* `webContents` [WebContents](web-contents.md)

Emesso quando un nuovo [webContents](web-contents.md) è creato.

### Evento: 'certificate-error'

Restituisce:

* `event` Event
* `webContents` [WebContents](web-contents.md)
* `url` String
* `error` String - Il codice d'errore
* `certificate` [Certificato](structures/certificate.md)
* `callback` Function 
  * `isTrusted` Boolean - Se considerare il certificato come verificato

Emesso quando fallisce la verifica del`certificate` per`url`, per verificare il certificato puoi prevenire il comportamento predefinito con `evento.preventDefault()` e chiamare `callback(true)`.

```javascript
const {app} = richiedi('electron')


app.on ('certificate-error', (event, webContents, url, error, certificate, callback) => {
  se (url === 'https://github.com') {
      // Logica di verifica.
    event.preventDefault()
    callback(true)
  } else {
    callback(false)
  }
})
```

### Evento: 'select-client-certificate'

Restituisce:

* `event` Event
* `webContents` [WebContents](web-contents.md)
* `url` URL
* `certificateList` [Certificate[]](structures/certificate.md)
* `callback` Function 
  * `certificate` [Certificate](structures/certificate.md) (opzionale)

Emesso quando un certificato client è richiesto.

L'`url` corrisponde alla voce di navigazione richiedente il certificato client e `callback` può essere chiamato con una voce filtrata dalla lista. Usando `evento.preventDefault()` si previene che l'app usi il primo certificato dal magazzino.

```javascript
const {app} = require('electron')

app.on('select-client-certificate', (event, webContents, url, list, callback) => {
  event.preventDefault()
  callback(list[0])
})
```

### Evento: 'login'

Restituisce:

* `event` Event
* `webContents` [WebContents](web-contents.md)
* `richiesta` Object 
  * `method` String
  * `url` URL
  * `referrer` URL
* `authInfo` Object 
  * `isProxy` Boolean
  * `scheme` String
  * `host` String
  * `port` Integer
  * `realm` String
* `callback` Function 
  * `username` String
  * `password` String

Emesso quando i `webContents` vogliono fare un'autenticazione base.

Il comportamento predefinito è di cancellare tutte le autenticazioni, per evitare ciò puoi prevenire il comportamento predefinito con `event.preventDefault()` e chiamare `callback(username, password)` con le credenziali.

```javascript
const {app} = require('electron')

app.on('login', (event, webContents, request, authInfo, callback) => {
  event.preventDefault()
  callback('username', 'secret')
})
```

### Evento: 'gpu-process-crashed'

Restituisce:

* `event` Event
* `killed` Boolean

Emesso quando i processi gpu crashano o soni uccisi.

### Evento: 'accessibility-support-changed' *macOS* *Windows*

Restituisce:

* `event` Event
* `accessibilitySupportEnabled` Boolean - `true` quando il supporto all'accessibilità a Chrome è abilitato, `false` altrimenti.

Emesso quando cambia il supporto accessibilità di Chrome. Questo evento avviene quando le tecnologie d'assistenza, come lettore schermo, sono abilitate o disabilitate. Vedi https://www.chromium.org/developers/design-documents/accessibility per ulteriori dettagli.

## Metodi

L'oggetto `app` ha i seguenti metodi:

**Nota:** Alcuni metodi sono disponibili solo su sistemi operativi specifici e sono etichettati come tali.

### `app.quit()`

Prova a chiudere tutte le finestre. L'evento `before-quit` sarà emesso prima. Se tutte le finestre sono chiuse con successo, l'evento `will-quit` sarà emesso e di default l'app sarà terminata.

Questo metodo garantisce che tutti i `beforeunload` e `unload` eventi gestionali siano correttamente eseguiti. È possibile che una finestra annulli l'uscita tornando `false` nell'evento gestionale `beforeunload`.

### `app.exit([exitCode])`

* `exitCode` Integer (opzionale)

Esci immediatamente con `exitCode`. Il `exitCode` predefinito è 0.

Tutte le finestre saranno immediatamente chiuse senza richiesta all'utente e gli eventi `before-quit` e `will-quit` non saranno emessi.

### `app.relaunch([options])`

* `options` Object (opzionale) 
  * `args` String[] - (opzionale)
  * `execPath` String (opzionale)

Rilancia l'app quando esiste la corrente istanza.

Di default la nuova istanza userà la stessa directory di lavoro e argomenti della linea di comando con la corrente istanza. Quando l'`args` è specificato, l'`args` sarà invece passato come argomento di linea di comando. Quando `execPath` è specificato, `execPath` sarà eseguito per rilanciare, invece, l'app corrente.

Nota che questo metodo non termina l'app quando eseguito, devi chiamare `app.quit` o `app.exit` dopo aver chiamato `app.relaunch` per riavviare l'app.

Quando `app.relaunch` è chiamato ripetutamente, le istanze multiple sarannò avviate dopo l'istanza corrente sia uscita.

Un esempio di riavvio dell'istanza corrente immediato e aggiungendo un nuovo argomento della linea di comando alla nuova istanza:

```javascript
const {app} = require('electron')


app.relaunch({args: process.argv..slice(1).concat(['--relaunch'])})
app.exit(0)
```

### `app.isReady()`

Restituisce `Boolean` - `true` se Electron ha finito l'inizializzazione, `false` viceversa.

### `app.focus()`

Su Linux, focalizza sulla prima finestra visibile. Su macOS rende l'applicazione attiva. Su Windows, focalizza sulla prima finestra dell'applicazione.

### `app.hide()` *macOS*

Nasconde tutte le finestre dell'applicazione senza minimizzarle.

### `app.show()` *macOS*

Mostra le finestre dell'applicazione dopo che sono state nascoste. Non le focalizza automaticamente.

### `app.getAppPath()`

Restituisce `String` - La directory dell'app corrente.

### `app.getPath(name)`

* `name` String

Restituisce `String` - Un percorso ad una directory speciale o ai file associati con `name`. In caso di fallimento avviene lanciato un `Error`.

Puoi richiedere i seguenti percorsi dal nome:

* `home` Directory della home utente.
* `appData` Dati della directory dell'app utente, con punti predefiniti a: 
  * `%APPDATA%` su Windows
  * `$XDG_CONFIG_HOME` o `~/.config` su Linux
  * `~/Library/Application Support` su macOS
* `userData` La directory per ammagazzinare i file di configurazione della tua app, che per valore predefinito è la directory `appData` seguita dal nome della tua app.
* `temp` Directory temporanea.
* `exe` L'attuale file eseguibile.
* `module` La libreria `libchromiumcontent`.
* `desktop` L'attuale directory del desktop utente.
* `documents` La directory per l'utente "I miei Documenti".
* `downloads` La directory per i file scaricati dall'utente.
* `music` La directory per la musica dell'utente.
* `pictures` La directory per le immagini dell'utente.
* `videos` La directory per i video dell'utente.
* `logs` La directory per la cartella registro della tua app.
* `pepperFlashSystemPlugin` Percorso intero alla versione di sistema del plugin Pepper Flash.

### `app.getFileIcon(path[, options], callback)`

* `path` String
* `options` Object (opzionale) 
  * `size` String 
    * `small` - 16x16
    * `normal` - 32x32
    * `large` - 48x48 su *Linux*, 32x32 su *Windows*, non supportato su *macOS*.
* `callback` Function 
  * `error` Error
  * `icon` [NativeImage](native-image.md)

Recupera un'icona associata al percorso.

Su *Windows* esistono 2 tipi di icone:

* Icone associate con certe estensioni di file come `.mp3`, `.png`, etc.
* Icone interne allo stesso file come `.exe`, `.dll`, `.ico`.

Su *Linux* e *macOS* le icone dipendono dall'app associata con il tipo di file mimo.

### `app.setPath(name, path)`

* `name` String
* `path` String

Sostituisce il `path` ad una directory speciale o ad un file associato con `name`. Se il percorso specifica una directory che non esiste, la directory sarà creata da questo metodo. In caso di fallimento viene generato un `Error`.

Si possono sostituire solo i percorsi di un `name` definiti in `app.getPath`.

Di default, i cookie e la cache delle pagine web saranno immagazzinate sotto la directory `userData`. Se vuoi cambiare questa posizione devi sostituire al percorso `userData` prima che l'evento `ready` del modulo `app` venga emesso.

### `app.getVersion()`

Restituisce `String` - La versione dell'app caricata. Se non viene trovata nessuna versione nel file dell'app `package.json`, la versione dell'attuale pacchetto o eseguibile è restituita.

### `app.getName()`

Restituisce `String`. Il nome attuale dell'app, che è il nome nel file dell'app `package.json`.

Spesso il campo `name` del `package.json` è un breve nome in minuscolo, seconda la specifica dei moduli npm. Di solito si dovrebbe anche specificare un campo `productName`, che è il nome in maiuscolo della tua applicazione, e che sarà preferito al `name` da Electron.

### `app.setName(name)`

* `name` String

Sostituisce l'attuale nome dell'app.

### `app.getLocale()`

Restituisce `String` - L'attuale locale dell'app. Possibili valori restituiti sono documentati [qui](locales.md).

**Note:** Quando distribuisci il tuo pacchetto app, devi anche navigare nelle cartelle `locali`.

**Note:** Su Windows devi chiamarlo dopo che l'evento `ready` è emesso.

### `app.addRecentDocument(path)` *macOS* *Windows*

* `path` String

Aggiungi `path` alla lista documenti recenti.

Questa lista è gestita dall'OS. Su Windows puoi visitare la lista dalla taskbar e su macOS la puoi visitare dal menu dock.

### `app,clearRecentDocuments()` *macOS* *Windows*

Pulisce la lista documenti recenti.

### `app.setAsDefaultProtocolClient(protocol[, path, args])`

* `protocol` String - Il nome del tuo protocollo, senza `://`. Se vuoi che la tua app gestisca i link `electron://` chiama questo metodo con `electron` come parametro.
* `path` String (opzionale) *Windows* - Di default a `process.execPath`
* `args` String[] (opzionale) *Windows* - Di default ad un array vuoto

Restituisce `Boolean` - Se la chiamata ha avuto successo.

Questo metodo imposta l'attuale eseguibile come handler di default per un protocollo (a. k. a. schema URI). Ti permette di integrare la tua app in profondità nel sistema operativo. Una volta registrati, tutti i link con `your-protocol://` saranno aperti con l'attuale eseguibile. L'intero link, incluso il protocollo, sarà passato all'app come parametro.

Su Windows puoi fornire parametri di percorso opzionali, il percorso al tuo eseguibile e gli argomenti, un array di argomenti da passare al tuo eseguibile quando si lancia.

**Nota:** Su macOS, puoi solo registrare protocolli aggiunti alla tua app `info.plist`, che non può essere modificato in esecuzione. Puoi comunque cambiare il file con un semplice editore di testo o script durante il momento di costruzione. Si prega di riferirsi alla [documentazione Apple](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/TP40009249-102207-TPXREF115) per i dettagli.

L'API usa il Registro Windows e LSImpostaGestionaleDefaultPerSchemaURL internamente.

### `app.removeAsDefaultProtocolClient(protocol[, path, args])` *macOS* *Windows*

* `protocol` String - Il nome del tuo protocollo, senza `://`.
* `path` String (opzionale) *Windows* - Di default a `process.execPath`
* `args` String[] (opzionale) *Windows* - Di default ad un array vuoto

Restituisce `Boolean` - Se la chiamata ha avuto successo.

Questo metodo controlla se l'eseguibile attuale è come un gestionale di default per un protocollo (o schema URI). Se sì, rimuoverà l'app come gestionale predefinito.

### `app.isDefaultProtocolClient(protocol[, path, args])` *macOS* *Windows*

* `protocol` String - Il nome del tuo protocollo, senza `://`.
* `path` String (opzionale) *Windows* - Di default a `process.execPath`
* `args` String[] (opzionale) *Windows* - Di default ad un array vuoto

Restituisci `Boolean`

Questo metodo controlla se l'eseguibile attuale è come un gestionale per un protocollo (o schema URI). Se sì, restituirà true. Altrimenti, restituirà false.

**Nota:** Su macOS puoi usare questo metodo per controllare se l'app è stata registrata come gestionale di protocolli di default per un protocollo. Puoi anche verificarlo controllando `~/Library/Preferences/com.apple.LaunchServices.plist` su computer macOS. Si prega di riferirsi alla [documentazione Apple](https://developer.apple.com/library/mac/documentation/Carbon/Reference/LaunchServicesReference/#//apple_ref/c/func/LSCopyDefaultHandlerForURLScheme) per i dettagli.

L'API usa il Registro Windows e LSCopiaGestionaleDefaultPerSchemaURL internamente.

### `app.setUserTasks(tasks)` *Windows*

* `tasks` [Task[]](structures/task.md) - Array di oggetti `Task`

Aggiungi `tasks` alla categoria [Task](http://msdn.microsoft.com/en-us/library/windows/desktop/dd378460(v=vs.85).aspx#tasks) della JumpList su Windows.

`tasks` è un array di oggetti [`Task`](structures/task.md).

Restituisce `Boolean` - Se la chiamata ha avuto successo.

**Nota:** Se ti piacerebbe modificare la Jump List ecco altri usi, invece, `app.setJumpList(categorie)`.

### `app.getJumpListSettings()` *Windows*

Restituisci `Object`:

* `minItems` Integer - Il minimo numero di elementi che saranno mostrati nella JumpList (per una più dettagliata descrizione di questo valore vedere [MSDN docs](https://msdn.microsoft.com/en-us/library/windows/desktop/dd378398(v=vs.85).aspx)).
* `removedItems` [JumpListItem[]](structures/jump-list-item.md) - Insieme degli oggetti `JumpListItem` che corrisponde agli elementi esplicitamente rimossi dall'utente dalle categorie modificate nella Jump List. Questi elementi non possono essere nuovamente aggiunti alla Jump List alla **next** chiamata a `app.setJumpList()`, Windows non mostrerà alcuna categoria personalizzata che contenga alcuni valori rimossi.

### `app.setJumpList(categories)` *Windows*

* `categories` [JumpListCategory[]](structures/jump-list-category.md) o `null` - Insieme di oggetti `JumpListCategory`.

Imposta o rimuovi una JumpList personalizzata per l'app, e restituisci una delle seguenti strimghe:

* `ok` - Nulla è andato storto.
* `errore` - Uno o più errori sono avvenuti, abilita il log di esecuzione per mostrare la possibile causa.
* `invalidSeparatorError` - È stato fatto un tentativo di aggiungere un separatore ad una categoria personalizzata nella Jump List. I separatori sono permessi solo nella categoria `Tasks` standard.
* `fileTypeRegistrationError` - È stato fatto un tentativo di aggiungere un link file alla Jump List per un tipo di file non gestibile dall'app.
* `customCategoryAccessDeniedError` - Le categorie personalizzate non possono essere aggiunte alla Jump List per motivi di privacy dell'utente o per le impostazioni di privacy di gruppo.

Se le `categories` sono `null` la precedentemente impostata Jump List (se esistente) sarà rimpiazzata dalla Jump List standard per l'app (gestita da Windows).

**Note:** Se un oggetto `SalaCategoriaLista` non ha nè `tipo` nè `nome` impostati il suo tipo diviene `predefinito`. Se la proprietà `nome` é impostata ma la proprietà `tipo` é omessa, il `tipo` sarà considerato`modificato`.

**Note:** Gli utenti possono rimuovere gli elementi dalle categorie personalizzate, e Windows non permetterà ad un elemento rimosso di essere ri-aggiunto in una categoria personalizzata fino a **dopo** la successiva chiamata di successo a `app.impostaJumpList(categorie)`. Qualsiasi tentativo di aggiunta di un elemento rimosso ad una categoria personalizzata prima che questo risulterà nell'intera categoria personalizzata sarà omesso dalla Jump List. La lista degli elementi rimossi può essere ottenuta usando `app.ottieniImpostazioniJumpList()`.

Questo è un esempio molto semplice di come creare una Jump List personalizzata:

```javascript
const {app} = require('electron')

app.setJumpList([
  {
    type: 'custom',
    name: 'Progetti recenti',
    items: [
      { type: 'file', path: 'C:\\Projects\\project1.proj' },
      { type: 'file', path: 'C:\\Projects\\project2.proj' }
    ]
  },
  { // Ha un nome, così `type` viene considerato come "custom"
    name: 'Strumenti',
    items: [
      {
        type: 'task',
        title: 'Strumento A',
        program: process.execPath,
        args: '--run-tool-a',
        icon: process.execPath,
        iconIndex: 0,
        description: 'Esegui Strumento A'
      },
      {
        type: 'task',
        title: 'Strumento B',
        program: process.execPath,
        args: '--run-tool-b',
        icon: process.execPath,
        iconIndex: 0,
        description: 'Esegui Strumento B'
      }
    ]
  },
  { type: 'frequent' },
  { // NON ha un nome, così `type` viene considerato come "tasks"
    items: [
      {
        type: 'task',
        title: 'Nuovo Progetto',
        program: process.execPath,
        args: '--new-project',
        description: 'Crea un nuovo progetto.'
      },
      { type: 'separator' },
      {
        type: 'task',
        title: 'Recupera Progetto',
        program: process.execPath,
        args: '--recover-project',
        description: 'Recupera Progetto'
      }
    ]
  }
])
```

### `app.makeSingleInstance(callback)`

* `callback` Function 
  * `argv` String[] - Un insieme della linea di comando d'argomento della seconda istanza
  * `workingDirectory` Stringa - La directory funzionante della seconda istanza

Restituisce `Boolean`.

Questo metodo rende la tua app una app a Singola Istanza - invece di permettere multiple istanze della tua app da eseguire, questo assicurerà che solo una singola istanza della tua app sia in esecuzione e che le altre istanze segnino questa ed escano.

`callback` sarà chiamato dalla prima istanza con `callback(argv, workingDirectory)` quando una seconda istanza è stata eseguita. `argv` è un insieme delle linee di comando degli argomenti della seconda istanza e la `workingDirectory` è la sua attuale Directory in uso. Di solito le app rispondono a questo focalizzando la loro finestra primaria e non minimizzata.

Il `callback` è garantito essere eseguito dopo che l'evento `ready` dell'`app` è stato emesso.

Questo metodo restituisce `false` se il tuo processo è l'istanza primaria dell'applicazione e la tua app potrebbe continuare a caricare. E restituisce `true` se il tuo processo ha inviato i suoi parametri ad un'altra istanza e dovresti immediatamente uscire.

Su macOS il sistema fa rispettare l'istanza singola automaticamente quando l'utente prova ad aprirne un'altra della vostra app su Finder e per questo sono emessi gli eventi `open-file` ed `open-url`. Comunque quando un utente avvia la tua app nella linea di comando il meccanismo della singola istanza del sistema sarà bypassato e devi usare questo metodo per assicurare la singola istanza.

Un esempio dell'attivazione drll'istanza primaria quando se ne avvia una seconda:

```javascript
const {app} = require('electron')
let myWindow = null

const isSecondInstance = app.makeSingleInstance((commandLine, workingDirectory) => {
  // Qualcuno ha provato ad avviare una seconda istanza, dovremmo focalizzare la nostra finestra.
  if (myWindow) {
    if (myWindow.isMinimized()) myWindow.restore()
    myWindow.focus()
  }
})

if (isSecondInstance) {
  app.quit()
}

// Crea myWindow, carica il resto dell'app, ecc...
app.on('ready', () => {
})
```

### `app.releaseSingleInstance()`

Rilascia tutti i blocchi creati da `makeSingleInstance`. Permetterà alle istanze multiple dell'app di essere eseguite di nuovo al contempo.

### `app.setUserActivity(type, userInfo[, webpageURL])` *macOS*

* `type` String - Unicamente identifica l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `userInfo` Object - Stato app specifico al magazzino per usare da altro dispositivo.
* `webpageURL` String (opzionale) - La pagina web da caricare nel browser se non sono installate app adatte nel dispositivo ripristinante. Lo schema deve essere `http` o `https`.

Crea un'`NSUserActivity` e la imposta come attività corrente. L'attività è eleggibile per [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) ad un altro dispositivo poi.

### `app.getCurrentActivityType()` *macOS*

Restituisce `String` - Il tipo di attività al momento in esecuzione.

### `app.invalidateCurrentActivity()` *macOS*

* `type` String - Unicamente identifica l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).

Invalida l'attività [Handoff](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html) corrente dell'utente.

### `app.updateCurrentActivity(type, userInfo)` *macOS*

* `type` String - Unicamente identifica l'attività. In riferimento a [`NSUserActivity.activityType`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSUserActivity_Class/index.html#//apple_ref/occ/instp/NSUserActivity/activityType).
* `userInfo` Object - Stato app specifico al magazzino per usare da altro dispositivo.

Aggiorna l'attività corrente se il suo tipo corrisponde al `type`, fondendo le voci da `userInfo` nel suo dizionario corrente `userInfo`.

### `app.setAppUserModelId(id)` *Windows*

* `id` String

Cambia il [Modello Id Applicazione Utente](https://msdn.microsoft.com/en-us/library/windows/desktop/dd378459(v=vs.85).aspx) ad `id`.

### `app.importCertificate(options, callback)` *LINUX*

* `options` Object 
  * `certificate` Stringa - Percorso per il file pkcs12.
  * `password` String - Frase d'accesso per il certificato.
* `callback` Function 
  * `result` Integer - Risultato dell'importo.

Importa il certificato in formato pkcs12 nel magazzino del certificato della piattaforma. `callback` è chiamato con il `result` dell'operazione di importazione, un valore di `` indica successo, mentre un altro valore indica un fallimento in base al chromium [net_error_list](https://code.google.com/p/chromium/codesearch#chromium/src/net/base/net_error_list.h).

### `app.disableHardwareAcceleration()`

Disabilita l'accelerazione hardware per l'app attuale.

Questo metodo può essere chiamato solo prima che l'app sia pronta.

### `app.disableDomainBlockingFor3DAPIs()`

Di default, Chromium disabilita le API 3D (come WebGL) fino al riavvio su una base per dominio se i processi GPU crashano troppo spesso. Questa funzione disabilita questo comportamento.

Questo metodo può essere chiamato solo prima che l'app sia pronta.

### `app.getAppMemoryInfo()` *Deprecato*

Restituisce [`ProcessMetric[]`](structures/process-metric.md): Insieme di oggetti `ProcessMetric` corrispondenti alle statistiche di utilizzo della memoria e della cpu di tutti i processi associati con l'app. **Nota:** Questo metodo è deprecato, usa invece `app.getAppMetrics()`.

### `app.getAppMetrics()`

Restituisce [`ProcessMetric[]`](structures/process-metric.md): Insieme di oggetti `ProcessMetric` corrispondenti alle statistiche di utilizzo della memoria e della cpu di tutti i processi associati con l'app.

### `app.getGPUFeatureStatus()`

Restituisce lo [`GPUFeatureStatus`](structures/gpu-feature-status.md) - Lo Stato Funzioni Grafiche da `chrome://gpu/`.

### `app.setBadgeCount(count)` *Linux* *macOS*

* `count` Integer

Restituisce `Boolean` - Se la chiamata ha avuto successo.

Imposta il contatore badge per l'app attuale. Impostare il conto a `` nasconderà il badge.

Su macOS esso è mostrato sull'icona del dock. Su Linux lavora sol9 con il Launcher Unity,

**Nota:** Il launcher Unity richiede l'esistenza di un file `.desktop` per funzionare, per ulteriori informazioni leggere [Desktop Integrazione Ambiente](../tutorial/desktop-environment-integration.md#unity-launcher-shortcuts-linux).

### `app.getBadgeCount()` *Linux* *macOS*

Restituisce `Integer` - Il valore attuale è mostrato nel contatore di badge.

### `app.isUnityRunning()` *Linux*

Restituisce `Boolean` - Se l'attuale ambiente desktop è il launcher Unity.

### `app.getLoginItemSettings([options])` *macOS* *Windows*

* `options` Object (opzionale) 
  * `path` String (opzionale) *Windows* - Il percorso eseguibile a comparazione. Di default è `process.execPath`.
  * `args` String[] (opzionale) *Windows* - La linea di comando degli argomenti comparata. Di default è un insieme vuoto.

Se hai fornito le opzioni di `path` e di `args` a `app.setLoginItemSettings` dovrai passare gli stessi argomenti qui per `openAtLogin` per impostarlo correttamente.

Restituisci `Object`:

* `openAtLogin` Boolean - `true` se l'app è impostata a aperta all'accesso.
* `openAsHidden` Boolean - `true` se l'app è impostata ad aprirsi come nascosta all'accesso. Impostazione supportata solo su macOS.
* `wasOpenedAtLogin` Boolean - `true` se l'app era aperta automaticamente all'all'accesso. Questa impostazione è supportata solo su macOS.
* `wasOpenedAsHidden` Boolean - `true` se l'app era aperta come un elemento di accesso nascosto. Questo indica che l'app potrebbe non aprire alcuna finestra all'avvio. Questa impostazione è supportata solo su macOS.
* `restoreState` Boolean - `true` se l'app era aperta come elemento d'accesso che potrebbe ripristinare lo stato dalla sessione precedente. Questo indica che l'app potrebbe ripristinare le finestre aperte l'ultima volta che l'app è stata chiusa. Questa impostazione è supportata solo su macOS.

**Nota:** Questa API non ha effetto sulle [Costruzioni MAS](../tutorial/mac-app-store-submission-guide.md).

### `app.setLoginItemSettings(settings)` *macOS* *Windows*

* `settings` Object 
  * `openAtLogin` Boolean (opzionale) - `true` per aprire l'app all'accesso, `false` per rimuovere l'app come elemento di accesso. Di default a `false`.
  * `openAsHidden` Boolean (opzionale) - `true` per aprire l'app come nascosta. Di default `false`. L'utente può editare questa impostazione dalle Preferenze di Sistema quindi `app.getLoginItemStatus().wasOpenedAsHidden` potrebbe essere controllato quando l'app è aperta per conoscere il valore attuale. Questa impostazione è supportata solo su macOS.
  * `path` String (opzionale) *Windows*. L'eseguibile al lancio all'accesso. Di default a `process.execPath`.
  * `args` String[] (opzionale) *Windows* - La linea di comando dell'argomento per passare all'eseguibile. Di default ad un insieme vuoto. Stai attento ad avvolgere i percorsi in quote.

Imposta le impostazioni dell'elemento d'accesso all'app.

Per lavorare con l'`autoUpdater` di Electron su Windows, che usa [Squirrel](https://github.com/Squirrel/Squirrel.Windows), vorrai impostare il percorso di lancio ad Update.exe e passare gli argomenti per specificare il nome della tua applicazione. Ad esempio:

```javascript
const cartellaApp = path.dirname(process.execPath)
const aggiornaExe = path.resolve(cartellaApp, '..', 'Aggiornamento.exe')
const nomeExe = path.basename(process.execPath)

app.setLoginItemSettings({
  openAtLogin: true,
  path: aggiornaExe,
  args: [
    '--processStart', `"${nomeExe}"`,
    '--process-start-args', `"--hidden"`
  ]
})
```

**Nota:** Questa API non ha effetto sulle [Costruzioni MAS](../tutorial/mac-app-store-submission-guide.md).

### `app.isAccessibilitySupportEnabled()` *macOS* *Windows*

Restituisci `Boolean` - `true` se il supporto d'accessibilità a Chrome è abilitato, `false` altrimenti. Questa API restituirà `true` se l'uso delle tecnologie d'assistenza, come il lettore schermo, sono state trovate. Vedi https://www.chromium.org/developers/design-documents/accessibility per altri dettagli.

### `app.setAccessibilitySupportEnabled(enabled)` *macOS* *Windows*

* `enabled` Boolean - Abilita o disabilita il rendering dell'[albero accessibilità](https://developers.google.com/web/fundamentals/accessibility/semantics-builtin/the-accessibility-tree)

Abilita manualmente il supporto accessibilità di Chrome permettendo di esporre gli scambi di accessibilità ad utenti nelle impostazioni applicazione. https://www.chromium.org/developers/design-documents/accessibility for more details. Disabilitato di default.

**Nota:** L'albero accessibilità del rendering può colpire significativamente la performance della tua app. Potrebbe non essere abilitato di default.

### `app.setAboutPanelOptions(options)` *macOS*

* `options` Object 
  * `applicationName` String (opzionale) - Il nome dell'app.
  * `applicationVersion` String (opzionale) - La versione dell'app.
  * `copyright` String (opzionale) - Informazioni di copyright.
  * `credits` String (opzionale) - Informazioni dei crediti.
  * `version` String (opzionale) - Il numero della versione build dell'app.

Vedi il pannello delle opzioni. Questo oltrepasserà i valori definiti nel file `.plist` del file. Vedi i [documenti Apple](https://developer.apple.com/reference/appkit/nsapplication/1428479-orderfrontstandardaboutpanelwith?language=objc) per altri dettagli.

### `app.commandLine.appendSwitch(switch[, value])`

* `switch` String - Un interruttore della linea di comando
* `value` String (opziomale) - Un valore per l'interruttore dato

Aggiungi un interruttore (con `value` opzionale) alla linea di comando di Chromium.

**Nota:** Non colpirà `process.argv` ed è principalmente usato dagli sviluppatori per controllare alcuni comportamenti di basso livello di Chromium.

### `app.commandLine.appendArgument(value)`

* `value` String - L'argomento da aggiungere alla linea di comando

Aggiungi un argomento alla linea di comando di Chromium. L'argomento sarà quotato correttamente.

**Nota:** Non colpirà `processo.argv`.

### `app.enableMixedSandbox()` *Sperimentale* *macOS* *Windows*

Abilita la modalità scatola dei giochi mischiata nell'app.

Questo metodo può essere chiamato solo prima che l'app sia pronta.

### `app.isInApplicationsFolder()` *macOS*

Restituisce `Boolean` - Se l'app è in esecuzione dalla cartella Applicazione. Usa in combinazione con `app.moveToApplicationsFolder()`

### `app.moveToApplicationsFolder()` *macOS*

Restituisce `Boolean` - Se la mossa ha avuto successo. Si prega di notare che se la mossa ha successo la tua applicazione si chiuderà e riavvierà.

Nessun dialogo di conferma sarà presentato di default, se vuoi permettere all'utente di confermare l'operazione potresti farlo usando l'API di [`dialog`](dialog.md).

**NOTA:** Questo metodo lancia errori se ogni altro dall'utente causa un fallimento della mossa. Per istanza se l'utente annulla il dialogo di autorizzazione questo metodo restituisce falso. Se falliamo nel performare la copia questo metodo lancerà un errore. Il messaggio nell'errore dovrebbe essere informativo e dirti esattamente cosa è andato storto

### `app.dock.bounce([type])` *macOS*

* `tipo` String (opzionale) - Può essere `critical` o `informational`. Di default è `informational`

Quando `critical` è passato, l'icona del dock rimbalza finché l'app diventa attiva o la richiesta viene annullata.

Quando `informational` è passato, l'icona del dock rimbalzerà per un secondo. Comunque la richiesta resterà attiva finché l'l'applicazione non diviene attiva o la richiesta viene annullata.

Restituisce `Integer` un ID rappresentante la richiesta.

### `app.dock.cancelBounce(id)` *macOS*

* `id` Integer

Annulla il rimbalzo dell'`id`.

### `app.dock.downloadFinished(filePath)` *macOS*

* `filePath` String

Rimbalza il download impilato se il Percorsofile è nella cartella dei file scaricati.

### `app.dock.setBadge(text)` *macOS*

* `text` String

Imposta la stringa da mostrare nell'area del dock di badging.

### `app.dock.getBadge()` *macOS*

Restituisce `String` - La stringa del badge del dock.

### `app.dock.hide()` *macOS*

Nasconde l'icona del dock.

### `app.dock.show()` *macOS*

Mostra l'icona del dock.

### `app.dock.isVisible()` *macOS*

Restituisce `Boolean` - Se l'icona del dock è visibile. L'`app.dock.show()` chiamato è asincrono quindi questo metodo potrebbe non restituire true immediatamente dopo questa chiamata.

### `app.dock.setMenu(menu)` *macOS*

* `menu` [Menu](menu.md)

Imposta il [dock menu](https://developer.apple.com/library/mac/documentation/Carbon/Conceptual/customizing_docktile/concepts/dockconcepts.html#//apple_ref/doc/uid/TP30000986-CH2-TPXREF103) dell'applicazione.

### `app.dock.setIcon(image)` *macOS*

* `image` ([NativeImage](native-image.md) | String)

Imposta l'`image` associata a questa icona del dock.