# webContents

> Web sayfalarını oluşturun ve kontrol edin.

İşlem: [Ana](../glossary.md#main-process)

`webContents` bir [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter) 'dır. Bir web sayfasını oluşturma ve denetlemekle sorumludur ve [`BrowserWindow`](browser-window.md) nesnesinin bir öğesidir. `webContents` nesnesine erişmenin bir örneği:

```javascript
const {BrowserWindow} = require('electron')

let win = new BrowserWindow({width: 800, height: 1500})
win.loadURL('http://github.com')

let contents = win.webContents
console.log(contents)
```

## Yöntemler

Bu yöntemlere `webContents` modülünden erişilebilir:

```javascript
const {webContents} = require('electron')
console.log(webContents)
```

### `webContents.getAllWebContents()`

`WebContents[]` 'ne döndürür - Tüm `WebContents` örneklerinin bir dizisi. Bu, tüm pencereler, web görüntüleri, açılan devtools eklentileri ve devtools uzantısı arka plan sayfaları için web içeriğini içerecektir.

### `webContents.getFocusedWebContents()`

`WebContents` 'ne dödürür - Bu uygulamaya odaklanmış web içeriği aksi takdirde `null` değerini döndürür.

### `webContents.fromId(id)`

* `id` tamsayı

`WebContents` 'ne döndürür - Belirli bir kimliği olan bir Web İçeriği örneği.

## Tür: Webİçerikleri

> Bir TarayıcıPenceresi örneğinin içeriğini oluşturun ve denetleyin.

İşlem: [Ana](../glossary.md#main-process)

### Örnek Events

#### Olay: 'did-finish-load'

Gezinme yapılırken, yani sekmenin döner kısmı dönmeyi durduğunda ortaya çıkar ve `onload` olayı gönderilir.

#### Olay: 'did-fail-load'

Dönüşler:

* `olay` Olay
* `errorCode` Tamsayı
* `errorDescription` Koşul
* `validatedURL` Koşul
* `isMainFrame` Boolean

Bu etkinlik, `did-finish-load` gibidir ancak yük başarısız olduğunda veya iptal edildiğinde yayınlanır, örneğin; `window.stop()` çağrılır. Hata kodlarının tam listesi ve anlamları [here](https://code.google.com/p/chromium/codesearch#chromium/src/net/base/net_error_list.h) mevcuttur.

#### Olay: 'did-frame-finish-load'

Dönüşler:

* `event` Event
* `isMainFrame` Boolean

Bir çerçeve aramayı bitirdiğinde ortaya çıkar.

#### Olay: 'did-start-loading'

Sekmenin döngüsü dönmeye başladığında puanlara karşılık gelir.

#### Olay: 'did-stop-loading'

Sekmenin döngüsü dönmeye başladığında puanlara karşılık gelir.

#### Olay: 'did-get-response-details'

Dönüşler:

* `event` Olay
* `status` Boolean
* `newURL` Dize
* `originalURL` Dize
* `httpResponseCode` Tamsayı
* `requestMethod` Dize
* `referrer` Dize
* `headers` Nesne
* `resourceType` String

İstenen bir kaynakla ilgili ayrıntılar mevcut olduğunda yayımlanır. `status` kaynağı indirmek için soket bağlantısını gösterir.

#### Olay: 'did-get-redirect-request'

Dönüşler:

* `event` Event
* `oldURL` Dize
* `newURL` Dize
* `isMainFrame` Boolean
* `httpResponseCode` Tamsayı
* `requestMethod` Dize
* `referrer` Dize
* `headers` Nesne

Bir kaynak talep ederken yönlendirme alındığında yayınlanır.

#### Olay: 'dom-ready'

Dönüşler:

* `event` Olay

Belirli bir çerçevedeki belge yüklendiğinde çıkar.

#### Olay: 'page-favicon-updated'

Dönüşler:

* `event` Event
* `favicons` String[] - URL'lerin dizilişleri

Sayfa sık kullanılan simge Url'lerini aldığında yayınlanır.

#### Olay: 'new-window'

Dönüşler:

* `event` Event
* `url` Dize
* `frameName` Dize
* `disposition` Dize - `default`, `foreground-tab`, `background-tab`, `new-window`, `ave-to-disk` ve `other` olabilir.
* `options` Nesne - Yeni `BrowserWindow` oluşturmak için kullanılacak seçenekler.
* `additionalFeatures` Dize[] - `window.open()` için verilen standart olmayan özellikler (Chromium veya Electron tarafından ele alınmayan özellikler).

Sayfa, bir `url` için yeni bir pencere açmayı istediğinde ortaya çıkar. `window.open` veya `<a target='_blank'>` gibi harici bir bağlantıyla istenebilir.

Varsayılan olarak `url` için yeni bir `BrowserWindow` oluşturulacaktır.

`event.preventDefault()` öğesinin çağrılması, Electron'un otomatik olarak yeni bir `BrowserWindow` oluşturmasını önleyecektir. `event.preventDefault()` öğesini çağırıp manuel olarak yeni bir `BrowserWindow` oluşturursanız, `event.newGuest` öğesini yeni `BrowserWindow` örneğine referans yapacak şekilde ayarlamanız gerekir; aksi halde beklenmeyen davranışlara neden olabilir. Örneğin:

```javascript
myBrowserWindow.webContents.on('new-window', (event, url) => {
  event.preventDefault()
  const win = new BrowserWindow({show: false})
  win.once('ready-to-show', () => win.show())
  win.loadURL(url)
  event.newGuest = win
})
```

#### Olay: 'will-navigate'

Dönüşler:

* `event` Event
* `url` Dize

Bir kullanıcı veya sayfa gezinme başlatmak istediğinde ortaya çıkar. `window.location` nesnesi değiştirildiğinde veya bir kullanıcı sayfadaki bir bağlantıyı tıklattığında olabilir.

Gezinme programlı olarak `webContents.loadURL` ve `webContents.back` gibi API'lerle başlatıldığında, bu olay yayınlanmaz.

Ayrıca, bağlı linkleri tıklama veya `window.location.hash` öğesini güncelleme gibi sayfa içi gezinmeler için de yayımlanmaz. Bu amaçla `did-navigate-in-page` etkinliğini kullanın.

`event.preventDefault()` öğesinin çağırılması gezinmeyi engeller.

#### Olay: 'did-navigate'

Dönüşler:

* `event` Event
* `url` Dize

Bir gezinme yapıldığında ortaya çıkar.

Ayrıca, bağlı linkleri tıklama veya `window.location.hash` öğesini güncelleme gibi sayfa içi gezinmeler için de yayımlanmaz. Bu amaçla `did-navigate-in-page` etkinliğini kullanın.

#### Olay: 'did-navigate-in-page'

Dönüşler:

* `event` Olay
* `url` Dize
* `isMainFrame` Boolean

Sayfa içi gezinme gerçekleştiğinde ortaya çıktı.

Sayfa içi gezinme gerçekleştiğinde, sayfa URL'si değişir, ancak sayfanın dışına çıkmasına neden olmaz. Bu gerçekleşen örnekler, bağlı link bağlantıları tıklandığında veya DOM `hashchange` olayı tetiklendiğinde görülür.

#### Olay: 'will-prevent-unload'

Dönüşler:

* `event` Olay

`beforeunload` olay işleyicisi, bir sayfayı kaldırmayı denediğinde yayımlanır.

`event.preventDefault()` öğesinin çağrılması, `beforeunload` olay işleyicisini yoksayar ve sayfanın boşaltılmasına izin verir.

```javascript
const {BrowserWindow, dialog} = require('electron')
const win = new BrowserWindow({width: 800, height: 600})
win.webContents.on('will-prevent-unload', (event) => {
  const choice = dialog.showMessageBox(win, {
    type: 'question',
    buttons: ['Çık', 'Kal'],
    title: 'Siteden çıkmak istediğinize emin misiniz?',
    message: 'Yaptığınız değişikliler kaydedilmeyecektir.',
    defaultId: 0,
    cancelId: 1
  })
  const leave = (choice === 0)
  if (leave) {
    event.preventDefault()
  }
})
```

#### Etkinlik: 'çöktü'

Dönüşler:

* `event` Etkinlik
* `killed` Boolean

Oluşturucu işlemi çöker veya yok olduğunda yayımlanır.

#### Event: 'plugin-crashed'

Dönüşler:

* `event` Olay
* `name` Dizi
* `versiyon` String

Bir eklenti işlemi çöktüğünde ortaya çıkar.

#### Etkinlik: 'yıkıldı'

`webContents` imha edildiğinde ortaya çıkar.

#### Olay: 'before-input-event'

Dönüşler:

* `event` Event
* `giriş` Nesne - Giriş özellikleri 
  * `type` Dize - `keyUp` veya `keyDown`
  * `key` Dize - Eşittir [KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `code` Dize - Eşittir [KeyboardEvent.code](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `isAutoRepeat` Boolean - Eşittir [KeyboardEvent.repeat](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `shift` Boolean - Eşittir [KeyboardEvent.shiftKey](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `control` Boolean - eşittir[KeyboardEvent.controlKey](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `alt` Boolean - Eşittir [KeyboardEvent.altKey](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
  * `meta` Boolean - Eşittir [KeyboardEvent.metaKey](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)

Sayfada `keydown` ve `keyup` olaylarını göndermeden önce yayınlanır. `event.preventDefault` öğesinin çağrılması, `keydown`/`keyup` etkinliklerini ve menü kısayollarını engeller.

Menü kısayollarını yalnızca engellemek için [`setIgnoreMenuShortcuts`](#contentssetignoremenushortcuts) kullanın:

```javascript
const {BrowserWindow} = require('electron')

let win = new BrowserWindow({width: 800, height: 600})

win.webContents.on('before-input-event', (event, input) => {
  // For example, only enable application menu keyboard shortcuts when
  // Ctrl/Cmd are down.
  win.webContents.setIgnoreMenuShortcuts(!input.control && !input.meta)
})
```

#### Olay: devtools açıldı

DevTools açıldığında yayınla.

#### Olay: devtools kapandı

DevTools kapandığında ortaya çıkar.

#### Olay: devtools odaklanıldı

DevTools odaklandığında / açıldığında ortaya çıkar.

#### Etkinlik: 'sertifika-hatası'

Dönüşler:

* `event` Olay
* `url` Dize
* `error` Dizi - Hata Kodu
* `certificate` [sertifika](structures/certificate.md)
* `geri aramak` Function 
  * `isTrusted` Boolean - Sertifikanın güvenilir olarak değerlendirilip değerlendirilemeyeceğini belirtir

Doğrulanamadığında ortaya çıkar `certificate` for `url`.

Kullanımı [the `certificate-error` olayı `app`](app.md#event-certificate-error) ile aynıdır.

#### Olay: 'select-client-certificate' 

Dönüşler:

* `event` Olay
* `url` URL
* `certificateList` [Sertifika[]](structures/certificate.md)
* `geri aramak` Function 
  * `certificate` [Certificate](structures/certificate.md) - Verilen listeden bir sertifika seçilmeli

Bir istemci sertifikası talep edildiğinde yayılır.

Kullanımı [the `select-client-certificate` olayı `app`](app.md#event-select-client-certificate) ile aynıdır.

#### Etkinlik: 'giriş'

Dönüşler:

* `event` Event
* `istek` Nesne 
  * `method` Dizi
  * `url` URL
  * `referrer` URL
* `authInfo` Nesne 
  * `isProxy` Boolean
  * `scheme` Dizi
  * `host` Dizi
  * `port` Tamsayı
  * `realm` Dizi
* `geri aramak` Function 
  * `username` Dizi
  * `password` Dizi

`webContents` temel doğrulama yapmak istediğinde çıkarılır.

Kullanımı [the `login` olayı `app`](app.md#event-login) ile aynıdır.

#### Etkinlik: 'sayfa içinde kurmak'

Dönüşler:

* `event` Event
* `sonuç` Nesne 
  * `requestId` Tamsayı
  * `activeMatchOrdinal` Integer - Etkin olan eşleşmenin konumu.
  * `matches` Integer - Eşleşmelerin sayısı.
  * `selectionArea` Object - İlk eşleşme alanının koordinatları.
  * `finalUpdate` Boolean

[`webContents.findInPage`] isteği için sonuç kullanılabilir olduğunda yayılıyor.

#### Olay: Medya oynamaya başladı

Medya oynamaya başladığında belirir.

#### Etkinlik: 'medya-duraklatıldı'

Medya duraklatıldığında veya oynatma süresi bittiğinde belirir.

#### Olay: tema rengi değiştirildi

Bir sayfanın tema rengi değiştiğinde ortaya çıkar. Bu genellikle karşılaşılanlardan kaynaklanmaktadır bir meta etiketi:

```html
<meta name='theme-color' content='#ff0000'>
```

Dönüşler:

* `event` Olay
* `color` (String | null) - Tema rengi '#rrggbb' biçiminde. Tema rengi ayarlanmadığında `null`'dir.

#### Etkinlik: 'update-target-url'

Dönüşler:

* `event` Olay
* `url` Dize

Fare bir bağlantı üzerinden geçtiğinde veya klavyenin bir bağlantıya odaklamasını sağladığı zaman yayımlanır.

#### Olay: 'cursor-changed'

Dönüşler:

* `event` Event
* `type` Dize
* `image` NativeImage (isteğe bağlı)
* `scale` Float (İsteğe Bağlı) Özel imleç için ölçekleme faktörü
* `size` [Size](structures/size.md) (isteğe bağlı) - `image` boyutu
* `hotspot` [Point](structures/point.md) (İsteğe bağlı) - Özel imlecin etkin noktasının koordinatları

İmlecin türü değiştiğinde çıkar. `type` parametresi bunlardan biri olabilir: `default`, `crosshair`, `pointer`, `text`, `wait`, `help`, `e-resize`, `n-resize`, `ne-resize`, `nw-resize`, `s-resize`, `se-resize`, `sw-resize`, `w-resize`, `ns-resize`, `ew-resize`, `nesw-resize`, `nwse-resize`, `col-resize`, `row-resize`, `m-panning`, `e-panning`, `n-panning`, `ne-panning`, `nw-panning`, `s-panning`, `se-panning`, `sw-panning`, `w-panning`, `move`, `vertical-text`, `cell`, `context-menu`, `alias`, `progress`, `nodrop`, `copy`, `none`, `not-allowed`, `zoom-in`, `zoom-out`, `grab`, `grabbing`, `custom`.

`type` parametre `custom` ise, `image` değişken özel imleç görüntüsünü `NativeImage` 'de ve `scale`, `size` ve `hotspot` özel imleç hakkında ek bilgi tutacaktır.

#### Olay:'bağlam-menüsü'

Dönüşler:

* `event` Olay
* `paramlar` Nesne 
  * `x` tamsayı - x koordinatı
  * `y` tamsayı - y koordinatı
  * `linkURL` Dize - Bağlam menüsünde çağrılan düğümü çevreleyen bağlantının URL' si.
  * `linkText` Dize - Bağlantıyla ilişkili metin. Bağlantının içeriği bir resim ise boş bir dize olabilir.
  * `pageURL` Dize - Bağlantı menüsünde çağırılan üst düzey sayfanın URL' si.
  * `frameURL` Dize - Bağlam menüsünün çağrıldığı alt çerçeveye ait URL.
  * `srcURL` Dize - İçerik menüsünde çağrıldığı öğenin kaynak URL' si. Görüntü, ses ve resimler kaynak URL' lerine sahiptirler.
  * `mediaType` Dize - Bağlam menüsünde çağırılan düğüm tipi. `none`, `image`, `audio`, `video`, `canvas`, `file` veya `plugin` olabilir.
  * `hasImageContents` Mantıksal - Bağlam menüsünün boş olmayan içeriğe sahip bir resim üzerinde çağrılmasına izin verilmez.
  * `isEditable` Mantıksal - Bağlamın düzenlenebilir olup olmadığı.
  * `selectionText` Dize - Bağlam menüsünün üzerinde çağırılan seçimin metni.
  * `titleText` Dize - Bağlam menüsü üzerinde çağırılan seçimin alt metni veya başlığı.
  * `misspelledWord` Metin - İmlecin altındaki yanlış yazılan sözcük.
  * `frameCharset` Dize - Menüden çağırılan çerçevenin karakter şifrelemesi.
  * `inputFieldType` Dizgi - Bağlam menüsü bir girdi alanından çağrıldığında, o alanın türü. Olası değerler: `none`, `plainText`, `password`, `other`.
  * `menuSourceType` Dizgi - Bağlam menüsünü çağıran giriş kaynağı.`none`, `mouse`, `keyboard`, `touch`, `touchMenu` olabilir.
  * `medya bayrakları` Obje - İçerik menüsünün medya elemanı için yapılmış bayraklar. 
    * `inError` Mantıksal - Ortam öğeside eğer çökme olursa.
    * `isPaused` Mantıksal - Ortam öğesinin duraklatılıp duraklatılmadığı.
    * `isMuted` Boolean - Ortam öğesinin sessiz olup olmadığı.
    * `hasAudio` Boolean - Ortam öğesinin sesli olup olmadığı.
    * `isLooping` Mantıksal - Ortam öğesi döngüsel olup olmadığında.
    * `isControlsVisible` Mantıksal - Ortam öğesinin kontrolleri olup olmadığını.
    * `canToggleControls` Boolean - Whether the media element's controls are toggleable.
    * `canRotate` Boolean - Whether the media element can be rotated.
  * `bayrakları editle` Object - These flags indicate whether the renderer believes it is able to perform the corresponding action. 
    * `canUndo` Boolean - Renderi alanın geri almasına inanması.
    * `canRedo` Boolean - Renderi alanın tekrar yapılmasına inanması.
    * `canCut` Boolean - Renderi alanın kesilmesine inanması.
    * `canCopy` Boolean - Renderi alanın kopyalanmasına inanması
    * `canPaste` Boolean - Renderi alanın yapıştırmaya inanması.
    * `canDelete` Boolean - Renderi alanın silinmesine inanması.
    * `canSelectAll` Boolean - Renderi alanın hepsinin seçilmesine inanması.

Emitted when there is a new context menu that needs to be handled.

#### Olay:'bluetooth-cihazı-seç'

Dönüşler:

* `event` Olay
* `devices` [BluetoothDevice[]](structures/bluetooth-device.md)
* `geri aramak` Function 
  * `deviceId` String

Bluetooth aygıtı `navigator.bluetooth.requestDevice` çağrı için seçilmesi gerektiğinde sinyal başlar. `navigator.bluetooth` api'sini kullanmak `webBluetooth`'u etkinleştirmelidir. Eğer `event.preventDefault` çağırılmazsa ilk bağlanılabilen alet seçilecektir. ` callback`, seçilecek `deviceId` ile çağırılmalıdır, `callback`'e boş string göndermek isteği iptal edecektir.

```javascript
const {app, webContents} = require('electron')
app.commandLine.appendSwitch('enable-web-bluetooth')

app.on('ready', () => {
  webContents.on('select-bluetooth-device', (event, deviceList, callback) => {
    event.preventDefault()
    let result = deviceList.find((device) => {
      return device.deviceName === 'test'
    })
    if (!result) {
      callback('')
    } else {
      callback(result.deviceId)
    }
  })
})
```

#### Etkinlik: 'boya'

Dönüşler:

* `event` Olay
* `dirtyRect` [Rectangle](structures/rectangle.md)
* `image` [NativeImage](native-image.md) - The image data of the whole frame.

Yeni kare oluşturulduğunda gönderilir. Yalnızca kirli alan arabellekten geçirilir.

```javascript
const {BrowserWindow} = require('electron')

let win = new BrowserWindow({webPreferences: {offscreen: true}})
win.webContents.on('paint', (event, dirty, image) => {
  // updateBitmap(dirty, image.getBitmap())
})
win.loadURL('http://github.com')
```

#### Olay:'devtools-reload-page'

Devtools penceresi webContents'ü yeniden yüklemeye yönlendirdiğinde çıkar

#### Event: 'will-attach-webview'

Dönüşler:

* `event` Olay
* `webPreferences` Nesne - Konuk sayfanın kullanacağı web tercihleri. Bu nesne konuk sayfası tercihlerini ayarlamak için değiştirilebilir.
* `params` Nesne - `src` URL gibi diğer `<webview>` parametreleri. Bu nesne konuk sayfası tercihlerini ayarlamak için değiştirilebilir.

`<webview>`'in web içerikleri bu web içeriklerine eklendiğinde gönderilir. `event.preventDefault()` çağırmak konuk sayfayı yok edecektir.

Bu event, `<webview>` yüklenmeden önce ` webContents`'inin `webPreferences<0>'ını ayarlamak için kullanılabilir ve <code><webview>` öznitelikleri aracılığıyla ayarlanamayacak ayarları değiştirme yetisi sağlar.

**Not:** Belirtilen `önyükleme` komut seçeneği `webPreferences` nesnesinin ` preloadURL`'u (`preload` değil) bu event'te gönderildikten sonra gözükecektir.

#### Event: 'did-attach-webview'

Dönüşler:

* `event` Olay
* `webContents` WebContents - The guest web contents that is used by the `<webview>`.

Emitted when a `<webview>` has been attached to this web contents.

#### Etkinlik: 'console-message'

Dönüşler:

* `level` Integer
* `message` String
* `line` Integer
* `sourceId` String

Emitted when the associated window logs a console message. Will not be emitted for windows with *offscreen rendering* enabled.

### Örnek Metodlar

#### `contents.loadURL(url[, options])`

* `url` Dize
* `seçenekler` Obje (opsiyonel) 
  * `httpReferrer` Dizgi (isteğe bağlı) - Bir HTTP başvuru bağlantısı.
  * `userAgent` Dizgi (isteğe bağlı) - İsteğin kaynağını oluşturan bir kullanıcı aracı.
  * `extraHeaders` Dizgi (isteğe bağlı) - "\n" ile ayrılan ek sayfa başlıkları
  * `postData` ([UploadRawData[]](structures/upload-raw-data.md) | [UploadFile[]](structures/upload-file.md) | [UploadFileSystem[]](structures/upload-file-system.md) | [UploadBlob[]](structures/upload-blob.md)) - (opsiyonel)
  * `baseURLForDataURL` Dizgi (isteğe bağlı) - Veri bağlantıları tarafından dosyaların yükleneceği (Dizin ayracına sahip) temel bağlantı. Buna, sadece belirtilen `url` bir veri bağlantısıysa ve başka dosyalar yüklemesi gerekiyorsa, gerek duyulur.

`url`'yi pencereye yükler. `url` bir protokol önadı içermek zorundadır, Örneğin `http://` veya `file://`. Eğer yüklemenin http önbelleğini atlaması gerekiyorsa, atlatmak için `pragma` başlığını kullanın.

```javascript
const {webContents} = require('electron')
const options = {extraHeaders: 'pragma: no-cache\n'}
webContents.loadURL('https://github.com', options)
```

#### `contents.downloadURL(url)`

* `url` Dize

Gezinme yapmadan `url` de bir kaynak indirmesi başlatır. `session`'a ait `will-download` olayı tetiklenir.

#### `contents.getURL()`

`String` olarak dönüt verir - Yürürlükteki web sayfasının bağlantısı.

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({width: 800, height: 600})
win.loadURL('http://github.com')

let currentURL = win.webContents.getURL()
console.log(currentURL)
```

#### `contents.getTitle()`

`String` olarak dönüt verir - Yürürlükteki web sayfasının başlığı.

#### `contents.isDestroyed()`

`Boolean` olarak dönüt verir - Web sayfasının yok edilip edilmediği.

#### `contents.focus()`

Web sayfasına odaklanır.

#### `contents.isFocused()`

`Boolean` olarak dönüt verir - Web sayfasına odaklanıp, odaklanılmadığı.

#### `contents.isLoading()`

`Boolean` olarak dönüt verir - Web sayfasının hala kaynak yüklemekte olup olmadığı.

#### `contents.isLoadingMainFrame()`

`Boolean` olarak dönüt verir - (Sadece bilgi iletim birimlerinin veya içindeki birimlerin değil) Ana bilgisayarın hala yüklemekte olup olmadığı.

#### `contents.isWaitingForResponse()`

`Boolean` olarak dönüt verir - Web sayfasının, sayfanın ana kaynağından gelecek bir ilk yanıt bekleyip beklemediği.

#### `contents.stop()`

Bekleyen gezinmeleri durdurur.

#### `contents.reload()`

Yürürlükteki web sayfasını yeniden yükler.

#### `contents.reloadIgnoringCache()`

Yürürlükteki sayfayı yeniden yükler ve önbelleği yoksayar.

#### `contents.canGoBack()`

`Boolean` olarak dönüt verir - Tarayıcının bir önceki web sayfasına geri gidip gidemeyeceği.

#### `contents.canGoForward()`

`Boolean` olarak dönüt verir - Tarayıcının bir sonraki web sayfasına gidip gidemeyeceği.

#### `contents.canGoToOffset(offset)`

* `offset` Tamsayı

`Boolean` olarak dönüt verir - Web sayfasının `offset`'e gidip gidemeyeceği.

#### `contents.clearHistory()`

Gezinme geçmişini temizler.

#### `contents.goBack()`

Tarayıcının bir sayfa geri gitmesini sağlar.

#### `contents.goForward()`

Tarayıcının bir sayfa ileri gitmesini sağlar.

#### `contents.goToIndex(index)`

* `index` Tamsayı

Tarayıcıyı belirtilmiş salt web sayfası dizinine (indeksine) yönlendirir.

#### `contents.goToOffset(offset)`

* `offset` Tamsayı

"Yürürlükteki girdi"den belirtilmiş göreli konuma (offsete) gider.

#### `contents.isCrashed()`

`Boolean` olarak dönüt verir - Görselleştirme işleminin arızalanıp arızalanmadığı.

#### `contents.setUserAgent(userAgent)`

* `userAgent` Dizgi

Bu sayfa için olan kullanıcı aracını geçersiz kılar.

#### `contents.getUserAgent()`

`String` olarak dönüt verir - Bu web sayfası için olan kullanıcı aracı.

#### `contents.insertCSS(css)`

* `css` Dizgi

Yürürlükteki web sayfasına CSS ekler.

#### `contents.executeJavaScript(code[, userGesture, callback])`

* `code` String
* `userGesture` Boolean (isteğe bağlı) - Varsayılan `false`'dır.
* `geri aramak` Function (isteğe bağlı) - Script çalıştıktan sonra çağırılır. 
  * `result` Any

`Promise` döndürür - çalışan kodun sonucuyla çözülen bir söz veya kodun sonucu reddedilen bir söz ise reddedilir.

Sayfadaki `code`'u ölçer.

Tarayıcı penceresinde, `requestFullScreen` gibi bazı HTML API'leri yalnızca kullanıcıdan gelen bir hareket ile çağrılmaktadır. `userGesture`'ü `true` olarak ayarlamak bu kısıtlamayı kaldırır.

Eğer çalıştırılan kodun sonucu bir promise ise, geri çağırma sonucu promise'un çözülen bir değeri olacaktır. Bir Promise ile sonuçlanan kodları işlemek için dönen Promise kullanmanızı tavsiye ederiz.

```js
contents.executeJavaScript('fetch("https://jsonplaceholder.typicode.com/users/1").then(resp => resp.json())', true)
  .then((result) => {
    console.log(result) // Will be the JSON object from the fetch call
  })
```

#### `contents.setIgnoreMenuShortcuts(ignore)` *Deneysel*

* `ignore` Boolean

Bu web içeriklerine odaklanılmışken uygulama menüsü kısayolları görmezden gelinir.

#### `contents.setAudioMuted(muted)`

* `muted` Boolean

Yürürlükteki web sayfasında bulunan sesi kapatır.

#### `contents.isAudioMuted()`

`Boolean` olarak dönüt verir - Sayfanın sesinin kapatılıp kapatılmadığı.

#### `contents.setZoomFactor(factor)`

* `factor` Number - Yakınlaştırma faktörü.

Yakınlaştırma faktörünü belirtilen faktöre değiştirir. Yakınlaştırma faktörü yakınlaştırma yüzdesinin 100'e bölünmüşüdür, böylece % 300 = 3.0 olur.

#### `contents.getZoomFactor(callback)`

* `geri aramak` Function 
  * `zoomFactor` Sayı

Yürürlükteki yakınlaştırma değerini almak için bir istek gönderir, `callback` , `callback(zoomFactor)` ile birlikte çağrılacaktır.

#### `contents.setZoomLevel(level)`

* `level` Number - Yakınlaştırma seviyesi

Yakınlaştırma düzeyini belirtilen seviyeye değiştirir. Orijinal boyut 0'dır ve her bir artım yukarıdaki veya aşağıdaki %20 daha büyük veya daha küçük, varsayılan %300 sınırına ve %50 orijinal boyutuna sırasıyla yakınlaştırma oranını temsil eder.

#### `contents.getZoomLevel(callback)`

* `geri aramak` Function 
  * `zoomLevel` Sayı

Yürürlükteki yakınlaştırma düzeyini almak için bir istek gönderir, `callback`, `callback(zoomLevel)` ile birlikte çağrılacaktır.

#### `contents.setZoomLevelLimits(minimumLevel, maximumLevel)`

* `minimumLevel` Number
* `maximumLevel` Number

**Kullanım dışı:** Bunun yerine, görsel yakınlaştırma seviye sınırlarını ayarlamak için `setVisualZoomLevelLimits` 'i çağırın. Bu yöntem Elektron 2.0'da kaldırılacaktır.

#### `contents.setVisualZoomLevelLimits(minimumLevel, maximumLevel)`

* `minimumLevel` Number
* `maximumLevel` Number

Maksimum ve minimum bas-yakınlaştır seviyesini ayarlar.

#### `contents.setLayoutZoomLevelLimits(minimumLevel, maximumLevel)`

* `minimumLevel` Number
* `maximumLevel` Number

Maksimum ve minimum layout-tabanlı (yani görsel olmayan) yakınlaştırma düzeyini ayarlar.

#### `contents.undo()`

`undo` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.redo()`

`redo` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.cut()`

`cut` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.copy()`

`copy` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.copyImageAt(x, y)`

* `x` Tamsayı
* `x` Integer

Verilen pozisyondaki görüntüyü panoya kopyalar.

#### `contents.paste()`

`paste` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.pasteAndMatchStyle()`

`pasteAndMatchStyle` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.delete()`

`delete` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.selectAll()`

`selectAll` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.unselect()`

`unselect` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.replace(text)`

* `text` String

`replace` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.replaceMisspelling(text)`

* `text` String

`replaceMisspelling` düzenleme komutunu web sayfasında çalıştırır.

#### `contents.insertText(text)`

* `text` Dizi

Odaklanmış öğeye `metin` ekler.

#### `contents.findInPage(text[, options])`

* `text` Dizi - Aranacak içerik; boş olmamalıdır.
* `seçenekler` Obje (opsiyonel) 
  * `forward` Boolean - (isteğe bağlı) İleriye veya geriye doğru arama yapılacağı, varsayılan olarak `true`'dur.
  * `findNext` Boolean - (İsteğe bağlı) İşlemin ilk istek veya takip isteği olduğu, varsayılan olarak `false`'tur.
  * `matchCase` Boolean - (İsteğe bağlı) Aramanın büyük-küçük harfe duyarlı olup olmayacağı, varsayılan olarak `false`'dur.
  * `wordStart` Boolean - (isteğe bağlı) Sadece kelime başlarına bakılıp bakılmayacağı, varsayılan olarak `false`'tur.
  * `medialCapitalAsWordStart` Boolean - (İsteğe bağlı) `wordStart` ile birleştirildiğinde, eğer eşleşme büyük harfle başlayıp küçük harf veya harf olmayan ifadeyle devam ediyorsa, eşleşmeyi kabul eder. Diğer çeşitli alt kelime (intra-word) eşleşmelerini kabul eder, varsayılan olarak `false`'tur.

Returns `Integer` - The request id used for the request.

Starts a request to find all matches for the `text` in the web page. The result of the request can be obtained by subscribing to [`found-in-page`](web-contents.md#event-found-in-page) event.

#### `contents.stopFindInPage(action)`

* `hareket` Dize - Bitişteki hareketi belirler`webContents.findInPage`] istek. 
  * `clearSelection` - Seçimi silin.
  * `keepSelection` - Seçimi normal bir seçime çevirir.
  * `activateSelection` - Odaklanır ve seçim ağına (node'a) tıklar.

Sunulan `action` ile birlikte, `webContents` için olan tüm `findInPage` isteklerini durdurur.

```javascript
const {webContents} = require('electron')
webContents.on('found-in-page', (event, result) => {
  if (result.finalUpdate) webContents.stopFindInPage('clearSelection')
})

const requestId = webContents.findInPage('api')
console.log(requestId)
```

#### `contents.capturePage([rect, ]callback)`

* `rect` [Rectangle](structures/rectangle.md) (isteğe bağlı) - Sayfanın yakalanılmak istenen alanı
* `geri aramak` Function 
  * `image` [NativeImage](native-image.md)

`rect` içerisinde kalan sayfanın anlık görüntüsünü yakalar. İşlemin tamamlanmasının ardından `callback`, `callback(İmage)` ile birlikte çağrılacaktır. `image`, anlık görüntünün verisini saklayan [NaviteImage](native-image.md)'in bir örneğidir. `rect` ifadesini çıkartmak görünebilen sayfanın tamamının yakalanmasını sağlar.

#### `contents.hasServiceWorker(callback)`

* `geri aramak` Function 
  * `hasWorker` Boolean

Herhangi bir ServiceWorker kaydı olup olmadığını kontrol eder ve `callback`'e yanıt olarak bir boolean dönütü verir.

#### `contents.unregisterServiceWorker(callback)`

* `geri aramak` Function 
  * `success` Boolean

Olan bütün ServiceWorker'ların kaydını siler ve JS promise çözüldüğünde veya reddedildiğinde, `callback`'e cevap olarak bir boolean döner.

#### `contents.getPrinters()`

Sistemdeki yazıcıların listesini alır.

[`PrinterInfo[]`](structures/printer-info.md) dönütünü verir

#### `contents.print([options], [callback])`

* `seçenekler` Obje (opsiyonel) 
  * `silent` Boolean (isteğe bağlı) - Kullanıcıya yazdırma seçeneklerini sormaz. Varsayılan olarak `false`'tur.
  * `printBackground` Boolean (isteğe bağlı) - Ek olarak arkaplan rengini ve web sayfasının görüntüsünü de yazdırır. Varsayılan olarak `false`'tur.
  * `deviceName` String (isteğe bağlı) - Kullanılacak yazıcının ismini ayarla. `''` varsayılandır.
* `geri aramak` Fonksiyon (isteğe bağlı) 
  * success` Boolean - Indicates success of the print call.

Penceredeki web sayfasını yazdırır. `silent`, `true` olarak ayarlandığında Electron, eğer `deviceName` boş bırakıldıysa, sistemin varsayılan yazıcısını ve varsayılan yazdırma ayarlarını seçecektir.

Web sayfasında `window.print()`'i çağırmak, `webContents.print({silent: false, printBackground: false, deviceName: ''})`'i çağırmaya denktir.

Yeni bir sayfa yazdırmaya zorlamak için `page-break-before: always;` CSS stilini kullanın.

#### `contents.printToPDF(options, callback)`

* `seçenekler` Nesne 
  * `marginsType` Integer - (isteğe bağlı) Kullanılacak kenar tipini belirler. Varsayılan kenar için 0, kenarsız olması için 1 ve en az kenar için 2'yi kullanır.
  * `pageSize` Dizgi - (İsteğe bağlı) üretilecek PDF'in sayfa boyutunu belirler. `A3`, `A4`, `A5`, `Legal`, `Letter`, `Tabloid` ya da micron olarak `height` ve `width` içeren bir nesne olabilir.
  * `printBackground` Boolean - (İsteğe bağlı) CSS arkaplanlarının yazdırılıp yazdırılmayacağı.
  * `printSelectionOnly` Boolean - (İsteğe bağlı) Sadece seçimin yazdırılıp yazdırılmayacağı.
  * `landscape` Boolean - (isteğe bağlı) manzara için `true`, portre için `false`.
* `geri aramak` Function 
  * `error` Error
  * `data` Buffer

Penceredeki web sayfasını Chromiumun özel yazdırma ayarları önizlemesiyle PDF olarak yazdırır.

İşlem tamamlandığında `callback`, `callback(error, data)` ile birlikte çağrılacaktır. `data` oluşturulan PDF'in verisini içeren bir `Buffer`'dır.

Eğer sayfada `@page` CSS kuralı (CSS at-rule) kullanıldıysa, `landscape` görmezden gelinecektir.

Varsayılan olarak, boş bir `options` şöyle kabul edilir:

```javascript
{
  marginsType: 0,
  printBackground: false,
  printSelectionOnly: false,
  landscape: false
}
```

Yeni bir sayfa yazdırmaya zorlamak için `page-break-before: always;` CSS stilini kullanın.

Bir `webContents.printToPDF` örneği:

```javascript
const {BrowserWindow} = require('electron')
const fs = require('fs')

let win = new BrowserWindow({width: 800, height: 600})
win.loadURL('http://github.com')

win.webContents.on('did-finish-load', () => {
  // Varsayılan yazdırma seçeneğini kullanıyoruz
  win.webContents.printToPDF({}, (error, data) => {
    if (error) throw error
    fs.writeFile('/tmp/print.pdf', data, (error) => {
      if (error) throw error
      console.log('PDF dosyası başarıyla yazıldı.')
    })
  })
})
```

#### `contents.addWorkSpace(path)`

* dizi `yolu`

Belirtilen yolu DevTools çalışma alanına ekler. DevTools yaratımından sonra kullanılması zorunludur:

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()
win.webContents.on('devtools-opened', () => {
  win.webContents.addWorkSpace(__dirname)
})
```

#### `contents.removeWorkSpace(path)`

* dizi `yolu`

Belirtilen yolu DevTools çalışma alanından kaldırır.

#### `contents.openDevTools([options])`

* `seçenekler` Obje (opsiyonel) 
  * `mode` Dizgi - Geliştirme araçlarını belirtilen yuvalama durumuyla açar, `right`, `bottom`, `undocked`, `detach` olabilir. Varsayılan olarak son kullanılan yuvalama durumunu kullanır. `undocked` moddayken, geri yuvalama (dock back) mümkündür. `detach` modda ise mümkün değildir.

Geliştirme araçlarını açar.

#### `contents.closeDevTools()`

Geliştirme araçlarını kapatır.

#### `contents.isDevToolsOpened()`

`Boolean` olarak dönüt verir - Geliştirme araçlarının açılıp açılmadığı.

#### `contents.isDevToolsFocused()`

`Boolean` olarak dönüt verir - Geliştirme araçları görünümüne odaklanıp odaklanılmadığı.

#### `contents.toggleDevTools()`

Geliştirme araçlarına geçiş yapar.

#### `contents.inspectElement(x, y)`

* `x` Tamsayı
* `x` Integer

(`x`,`y`) pozisyonundaki ögeyi incelemeye başlar.

#### `contents.inspectServiceWorker()`

Servis işçisisi bağlamı için geliştirici araçları açar.

#### `contents.send(channel[, arg1][, arg2][, ...])`

* `channel` Dizesi
* `...args` herhangi[]

İşleyiciye `channel` aracılığıyla bir asenkron mesaj yollayın, aynı zamanda rastgele argümanlar da yollayabilirsiniz. Bağımsız değişkenler dahili olarak JSON'da seri hale getirilecek ve dolayısıyla hiçbir işlev veya prototip zinciri dahil edilmeyecektir.

Render işlemi mesajı `channel` `ipcRenderer` modülü ile dinleyebilir.

Ana işlemden render işlemine gönderilen mesaj örneği:

```javascript
// Ana süreçte.
const {app, BrowserWindow} = require('electron')
let win = null

app.on('ready', () => {
  win = new BrowserWindow({width: 800, height: 600})
  win.loadURL(`file://${__dirname}/index.html`)
  win.webContents.on('did-finish-load', () => {
    win.webContents.send('ping', 'whoooooooh!')
  })
})
```

```html
<!-- index.html -->
<html>
<body>
  <script>
    require('electron').ipcRenderer.on('ping', (event, message) => {
      console.log(message)  // Prints 'whoooooooh!'
    })
  </script>
</body>
</html>
```

#### `contents.enableDeviceEmulation(parameters)`

* `parametreler` Nesne 
  * `ekranPozisyonu` Dize - Ekran tipini emulasyon için belirtiniz. (default: `masaüstü`) 
    * `desktop` - Masaüstü ekran tipi
    * `mobile` - Mobil ekran tipi
  * `screenSize`[Size](structures/size.md) - Emülasyon uygulanacak ekran genişliğini ayarlar (screenPosition == mobile)
  * `viewPosition` [Point](structures/point.md) - Position the view on the screen (screenPosition == mobile) (default: `{x: 0, y: 0}`)
  * `deviceScaleFactor` Integer - Set the device scale factor (if zero defaults to original device scale factor) (default: ``)
  * `viewSize` [Size](structures/size.md) -Benzetilmiş görüntü boyutunu ayarlar (boş demek üstüne yazma yok demek)
  * `fitToView` Boolean - Emulated görünümü gerekiyorsa varolan alana sığacak şekilde ölçeklendirilmelidir.( Varsayılan:`false`)
  * `offset` [Point](structures/point.md) - Emulated görüntünün kullanılabilir alan içerisindeki ofsetidir.(Görüntüleme moduna uygun değil) (varsayılan: `{x: 0, y: 0}`)
  * `scale` Float - Emulated görüntünün kullanılabilir alan içerisindeki ölçeğidir.( Görüntüleme moduna uygun değil) (varsayılan: `1`)

Verilen parametrelerle aygıt emülasyonuna izin verir.

#### `contents.disableDeviceEmulation()`

`webContents.enableDeviceEmulation` tarafından izin verilen araç taklitini devredışı bırakır.

#### `contents.sendInputEvent(event)`

* `event` Nesne 
  * `type` String (**required**) -Olabilir, olayın türü `mouseDown`, `mouseUp`, `mouseEnter`, `mouseLeave`, `contextMenu`, `mouseWheel`, `mouseMove`, `keyDown`, `keyUp`, `char`.
  * `modifiers` String[] - Bir olay düzenleyici dizisi şunları içerebilir `shift`, `control`, `alt`, `meta`, `isKeypad`, `isAutoRepeat`, `leftButtonDown`, `middleButtonDown`, `rightButtonDown`, `capsLock`, `numLock`, `left`, `right`.

Sayfaya bir `event` girdisi gönderir. **Note:** `sendInputEvent()`'in çalışması için içeriği içeren `BrowserWindow`'a odaklanılmış olması gereklidir.

Klavye olayları için `event` nesnesi aşağıdaki özellikleri de alacaktır:

* `keyCode` Dizgi (**gerekli**) - Klavye olayı olarak gönderilecek karakter. Sadece [Accelerator](accelerator.md)'daki geçerli anahtar kodları kullanılmalıdır.

Fare olayları için, `event` nesnesi aşağıdaki özellikleri de alacaktır:

* `x` Tamsayı (**gerekli**)
* `y` Tamsayı (**gerekli**)
* `button` Dizgi - Basılan düğme, `left`, `middle` veya `right` olabilir
* `globalX` Tamsayı
* `globalY` Tam sayı
* `movementX` Tamsayı
* `movementY` Tamsayı
* `clickCount` Tamsayı

`mouseWheel` olayı için, `event` nesnesi aşağıdaki özellikleri de alacaktır:

* `deltaX` Tamsayı
* `deltaY` Tamsayı
* `wheelTicksX` Tamsayı
* `whellTicksY` Tamsayı
* `accelerationRatioX` Tamsayı
* `accelerationRatioY` Tamsayı
* `hasPreciseScrollingDeltas` Boolean
* `canScroll` Boolean

#### `contents.beginFrameSubscription([onlyDirty ,]callback)`

* `onlyDirty` Boolean (İsteğe bağlı) - Varsayılan olarak `false`'tur
* `geri aramak` Function 
  * `frameBuffer` Buffer
  * `dirtyRect` [Rectangle](structures/rectangle.md)

Olayların ve yakalanan çerçevelerin sunulması için sürdürümcü olur; Bir sunum olayı olduğunda `callback` , `callback(frameBuffer,dirtyRect)` ile birlikte çağrılacaktır.

`frameBuffer` işlenmemiş piksel verilerini içeren bir `Buffer`'dır. Çoğu makine üzerinde piksel verileri etkili bir şekilde 32 bit BGRA formatında saklanır, ancak gerçek gösterim işlemcinin endianına bağlıdır (en modern işlemciler little-endian, big-endian işlemcili makinelerde veri 32 bit ARGB formatındadır).

`dirtyRect`, sayfanın hangi bölümlerinin yeniden boyandığını tanımlayan `x, y, width, height` özelliklerini barındıran bir nesnedir. Eğer `onlyDirty`, `true`'ya ayarlandıysa, `frameBuffer` sadece yeniden boyanan alanları içerecektir. `onlyDirty` varsayılanı `false`'tur.

#### `contents.endFrameSubscription()`

End subscribing for frame presentation events.

#### `contents.startDrag(item)`

* `öğe` Nesne 
  * `file` String or `files` Array - The path(s) to the file(s) being dragged.
  * `icon` [NativeImage](native-image.md) - The image must be non-empty on macOS.

Yürürlükteki sürükle-bırak işlemi içi `item`'i sürükleme elemanı olarak ayarlar; `file` dosyanın sürükleneceği değişmez dosya yoludur ve `icon` sürükleme sırasında imlecin altında gösterilecek olan görüntüdür.

#### `contents.savePage(fullPath, saveType, callback)`

* `fullPath` String - Tam dosya yolu.
* `saveType` String - Kayıt türünü belirtir. 
  * `HTMLOnly` - Yalnızca sayfanın HTML'ını kaydeder.
  * `HTMLComplete` - Save complete-html page.
  * `MHTML` - Save complete-html page as MHTML.
* `geri aramak` Function - `(error) => {}`. 
  * `error` Error

Eğer sayfayı kaydetme işlemi başarıyla gerçekleştirilirse `Boolean` - true döner.

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()

win.loadURL('https://github.com')

win.webContents.on('did-finish-load', () => {
  win.webContents.savePage('/tmp/test.html', 'HTMLComplete', (error) => {
    if (!error) console.log('Save page successfully')
  })
})
```

#### `contents.showDefinitionForSelection()` *macOS*

Sayfadan seçilen sözcüğü arayan bir pop-up sözlük gösterir.

#### `contents.setSize(options)`

Sayfanın boyutunu ayarlayın. Bu yalnızca `<webview>` konuk içerikler için desteklenmektedir.

* `seçenekler` Nesne 
  * `normal` Object (optional) - Normal size of the page. This can be used in combination with the [`disableguestresize`](web-view-tag.md#disableguestresize) webgörünümü misafir içeriğine verilecek özelliği belirle. 
    * `width` Tamsayı
    * `height` Integer

#### `contents.isOffscreen()`

`Boolean` döner- *offscreen rendering*'in etkinleştirilip etkinleştirilmediğini gösterir.

#### `contents.startPainting()`

Eğer *offscreen rendering* etkinleştirildiyse ve boyama yapılmıyorsa, boyamaya başla.

#### `contents.stopPainting()`

Eğer *offscreen rendering* etkinleştirildiyse ve boyama yapılıyorsa, boyamayı durdur.

#### `contents.isPainting()`

`Boolean` döner- Eğer *offscreen rendering* etkinleştirildiyse şu anda boyama yapılıp yapılmadığını döner.

#### `contents.setFrameRate(fps)`

* `fps` tamsayı

Eğer *offscreen rendering* etkinleştirildiyse kare hızını belirli bir sayıya ayarlar. Yalnızca 1 ve 60 arasındaki değerler kabul edilir.

#### `contents.getFrameRate()`

`Integer` döner - Eğer *offscreen rendering* etkinleştirildiyse şu anki kare hızını döner.

#### `contents.invalidate()`

Bu web içeriklerinin içinde olduğu pencereyi tamamen yeniden boyamak için zaman ayarlar.

Eğer *offscreen rendering* etkinleştirildiyse çerçeveyi geçersiz kılar ve `'paint'` olayı aracılığıyla yeni bir tane oluşturur.

#### `contents.getWebRTCIPHandlingPolicy()`

Returns `String` - Returns the WebRTC IP Handling Policy.

#### `contents.setWebRTCIPHandlingPolicy(policy)`

* `yönetmelik` String - WebRTC IP Yönetme İlkesini belirler. 
  * `default` - Kullanıcının açık ve yerel IP'lerini açığa çıkarır. Bu varsayılan davranıştır. Bu ilke kullanıldığında WebRTC bütün arayüzleri sıralama ve açık arayüzleri keşfetmek için onları bağlama hakkına sahip olur.
  * `default_public_interface_only` - Exposes user's public IP, but does not expose user's local IP. Bu ilke kullanıldığında WebRTC yalnızca http tarafından varsayılan yolu kullanmalıdır. Bu herhangi bir yerel adresi açığa çıkarmaz.
  * `default_public_and_private_interfaces` - Exposes user's public and local IPs. Bu ilke kullanıldığında WebRTC yalnızca http tarafından kullanılan varsayılan yolu kullanmalıdır. Bu ayrıca ilgili varsayılan özel adresleri de açığa çıkarır. Varsayılan yol, çok merkezli bir bitim noktasında İşletim Sistemi tarafından seçilen yoldur.
  * `disable_non_proxied_udp` - açık veya yerel IP'leri açığa çıkarmaz. Bu ilke kullanıldığında WebRTC, proxy sunucusu UDP'yi desteklemediği sürece eşlere veya servislere erişmek için yalnızca TCP kullanmalıdır.

WebRTC IP yönetme ilkesini ayarlamak size hangi IPlerin WebRTC tarafından gösterildiğini kontrol etme izni verir. Daha fazla detay için [BrowserLeaks](https://browserleaks.com/webrtc)'e bakın.

#### `contents.getOSProcessId()`

`Integer` döner- İlgili işleyici işleminin `pid`'si.

### Örnek Özellikleri

#### `contents.id`

A `Integer` representing the unique ID of this WebContents.

#### `contents.session`

WebContents tarafından kullanılan bir [`Integer`](session.md).

#### `contents.hostWebContents`

Bir [`WebContents`](web-contents.md) örneği `WebContents`'e sahip olabilir.

#### `contents.devToolsWebContents`

A `WebContents` of DevTools for this `WebContents`.

**Not:** Kullanıcılar asla bu nesneyi depolamamalıdırlar çünkü DevTools kapandığında nesne `null`'a dönebilir.

#### `contents.debugger`

Bu web içerikleri için bir [Hata ayıklayıcı](debugger.md) örneği.