
# REDAXO-AddOn: Plyr

Das AddOn stellt den Video-/Audio-Player [Plyr](https://plyr.io) zur Verfügung.

![Screenshot](https://raw.githubusercontent.com/FriendsOfREDAXO/video/assets/mediapool.jpg)


Es können lokale Audio-Dateien (mp3), Videos und Youtube- sowie Vimeo-Videos eingebunden werden.  
Wir haben uns bewusst gegen eine automatische Einbindung im Frontend entschieden um dem Entwickler alle Freiheiten zu lassen. 

## AddOn Features
- Statische PHP Methode zur Ausgabe des Videos
- REX_PLYR[] Variable zur schnellen Ausgabe in einem Modul 
- Einbindung des Players im Backend
- Plyr bindet sich in die Detailseite des Medienpools ein
- Methoden zur Ermittlung des Videotyps
- Methode zur Erstellung von Playlists 
- Controls können je Ausgabe definiert werden
- JQuery wird für Playlists benötigt

> Bei Medien aus dem Medienpool muss nur der Dateiname angegeben werden. Bei Youtube und Vimeo immer die vollständige URL. 
Diese Methode bietet sich an um evtl. mehrere Videos z.B. aus einer Datenbank oder Medialist zu verarbeiten.

## Ausgabe eines Mediums

Die Ausgabe erfolgt über die Methode outputMedia(). 

Einzelmedium:

`rex_plyr::outputMedia($file,$controls,$poster)`

oder über die REX_VALUE für Einzelmedien: 

`REX_PLYR[id=1 controls="play,progress"]`

PlayLists werden wie folgt ausgegeben: 

`rex_plyr::outputMediaPlaylist($media_filenames,$controls)`



## Standard-Player 

### Einbindung im Frontend

Die nötigen Dateien findet man im Assets-Ordner. 
Eigene CSS und JS sollten nach Möglichkeit an anderer Stelle abgelegt werden um Probleme nach einem Update zu vermeiden. 

Plyr benötigt 2 JS-Dateien und eine CSS. In der `plyr_video.js` wird der Player initialisiert. 

#### CSS für Plyr

```html
<link rel="stylesheet" href="<?= rex_url::base('assets/addons/plyr/vendor/plyr/dist/plyr.css') ?>">
```

#### JS für Plyr

```php
<script src="<?= rex_url::base('assets/addons/plyr/vendor/plyr/dist/plyr.min.js') ?>"></script>
<script src="<?= rex_url::base('assets/addons/plyr/plyr_init.js') ?>"></script>
```
Die `plyr_init.js` ist als Beispiel anzusehen und bietet "nur" basic Settings. Sollte die Webpräsenz in einem Unterordner angelegt sein, müssen die Pfade für `iconUrl` und `blankVideo` angepasst werden. 

#### Inhalt der `plyr_init.js`

```js
document.addEventListener("DOMContentLoaded", function(){
 const players = Plyr.setup('.rex-plyr',{
	  youtube: {
            noCookie: true
        },
        vimeo: {
            dnt: false
        },
        iconUrl: '/assets/addons/plyr/vendor/plyr/dist/plyr.svg',
        blankVideo: '/assets/addons/plyr/vendor/plyr/dist/blank.mp4'
    });
    if (document.querySelector('.rex-plyr')) {
        players.forEach(function (player) {
            player.on('play', function () {
                var others = players.filter(other => other != player)
                others.forEach(function (other) {
                    other.pause();
                })
            });
        });
});

```

>Alle weiteren Infos zur Konfiguration der Skripte oder der Controls der Ausgaben, finden sich auf der GitHub-Site von [Plyr](https://plyr.io). 


### Modul-Beispiel mit MFORM CustomLink

Das CustomLink-Widget bietet sich an, weil die Redaktion damit lokale und externe Medien verlinken kann. 

#### Eingabe

```php
$mform = new MForm();
$mform->addFieldset("Video");
$mform->addCustomLinkField("1", array('label'=>'Medium', 'data-tel'=>'disable', 'data-mailto'=>'disable', 'data-formlink'=>'disable', 'data-intern'=>'disable'));
$mform->addMediaField(1, array('label'=>'Image'));
echo $mform->show();
```

#### Ausgabe über `rex_plyr::outputMedia`

```php
echo rex_plyr::outputMedia('REX_VALUE[1]','play-large,play,mute,volume,progress,airplay,pip,autoplay,loop','/media/cover/REX_MEDIA[1]');
```
> Beispiel mit allen Parametern, die nicht gewünschten Parameter bitte entfernen

## Playlist-Player

![Screenshot](https://raw.githubusercontent.com/FriendsOfREDAXO/video/assets/playlist.jpg)

### CSS für Playlist

Falls nicht bereits schon eingebunden: 

```html
<link rel="stylesheet" href="<?= rex_url::base('assets/addons/plyr/vendor/plyr/dist/plyr.css') ?>">
```
und zusätzlich: 
```html
<link rel="stylesheet" href="<?= rex_url::base('assets/addons/plyr/plyr_playlist.css') ?>">
```

### JS für Playlists

JS für Plyr Playlist lautet anders
```php
<script src="<?= rex_url::base('assets/addons/plyr/vendor/plyr/dist/plyr.min.js') ?>"></script>
<script src="<?= rex_url::base('assets/addons/plyr/plyr_playlist.js') ?>"></script>
```

### Modul-Eingabe Playlist

```php
REX_MEDIALIST[id="1" type="mp3,mp4" widget="1"]
```

#### Modul-Ausgabe Playlist 

Die Ausgabe erfolgt über `rex_plyr::outputMediaPlaylist`

Modul-Ausgabe

```php
$media_filenames = preg_grep('/^\s*$/s', explode(",", REX_MEDIALIST[1]), PREG_GREP_INVERT);
echo rex_plyr::outputMediaPlaylist($media_filenames,'play-large,play,progress,airplay,pip');
```

## Plyr und Consent-Abfragen

Der Parameter $consent erlaubt es einen Platzhalter-Text / Bild etc. einzubinden, der z.B. nach Aktivierung im Consent-Manager ersetzt wird. 

> Das o.g. Intitialisierungsskript 'plyr_init.js' wird nicht benötigt und muss aus dem Template entfernt werden.  


```php
$consent = '    
<div class="aspect-ratio-16-9">
		<div class="container uk-background-secondary uk-light uk-padding">
			<h2 class="page-title">Externes Video</h2>
			<p class="page-description">Bitte aktivieren Sie die Optionen zur Darstellung externer Video in den Datenschutzeinstellungen</p>
            <p><a class="uk-button uk-button-primary consent_manager-show-box">Datenschutz-Einstellungen bearbeiten</a></p>
		</div>
	</div>'; 
echo rex_plyr::outputMedia('REX_VALUE[1]','play-large,play,progress,airplay,pip','/media/cover/REX_MEDIA[1]',$consent);

```
Videos, die einen Consent erfordern erhalten die CSS-Class rex-plyr_consent. 
Aktuell Youtube und Vimeo
Es bietet sich an den Consent-Text zentral abzulegen, z.B. als Property im Project-Addon. 

Im Consent-Manager muss beim Cookie folgendes Script eingesetzt werden: 

```js
<script>
document.addEventListener("DOMContentLoaded", function(){
const players = Plyr.setup('.rex-plyr_consent', {
        youtube: {
            noCookie: true
        },
        vimeo: {
            dnt: false
        },
        iconUrl: '/assets/addons/plyr/vendor/plyr/dist/plyr.svg',
        blankVideo: '/assets/addons/plyr/vendor/plyr/dist/blank.mp4'
    });
    if (document.querySelector('.rex-plyr')) {
        players.forEach(function (player) {
            player.on('play', function () {
                var others = players.filter(other => other != player)
                others.forEach(function (other) {
                    other.pause();
                })
            });
        });
    }
});
</script>
```


## `REX_PLYR`

Zur Ausgabe einzelner Medien steht auch eine REDAXO-Variable zur Verfügung. 


```php
REX_PLYR[1]
```

oder mit Konfiguration der Controls:

```php
REX_PLYR[id=1 controls="play,progress"]
```



## Alternative init.js

zur Änderung der Vollbildanzeige bei Orientierungsänderung des Geräts

```js
document.addEventListener("DOMContentLoaded", function(){
const players = Plyr.setup('.rex-plyr',{
youtube: {
noCookie: true
},
fullscreen: {
enabled: true,
fallback: true,
iosNative: false }
});
});

const players = new Plyr('.rex-plyr');
players.on('play', event => {
const instance = event.detail.plyr;

screen.orientation.addEventListener("change", function() {
if(screen.orientation.type = 'landscape-primary') {
players.fullscreen.enter();
}
if(screen.orientation.type = 'portrait-primary') {
players.fullscreen.exit();
}
}, false);

window.addEventListener('orientationchange', function() {
if (window.orientation & 2) {
players.fullscreen.enter();
} else {
players.fullscreen.exit();
}

});

});
```

## Methoden in der rex_plyr class

`rex_plyr::outputMedia($url,$controls,$poster,$consent)`
Erstellt einen Player annhand einer Mediendatei oder URL. 

`rex_plyr::outputMediaPlaylist($media_filenames,$controls)`
Erstellt eine Playlist anhand des übergebenen Arrays

`checkUrl($url)`
Gibt sofern es sich um eine Mediapool-Datei handelt die URL zum Medium zurück. 

`checkYoutube($url)` 
Prüft ob es sich um eine Youtube-URL handelt

`getYoutubeId($url)` 
Ermittelt die Youtube-ID eines Videos

`checkVimeo($url)` 
Prüft ob es sich um eine Vimeo-URL handelt

`getVimeoId($url)` 
Ermittelt die Vimeo-Id eines Videos

`checkMedia($url)` 
Überprüft ob es sich um ein MP4-Video aus dem Medienpool handelt

`checkExternalMp4($url)`
Überprüft ob ein externes MP4-Video verlinkt ist.

`checkVideo($url)`
Überprüft ob es sich um eine Video-Datei / eine Video-Url handelt die plyr abspielen kann.

`checkAudio($url)` 
Überprüft ob es sich um eine MP3-Audio-Datei aus dem Medienpool handelt

### Beispiel

```php
$plyr = new rex_plyr();
$id = $plyr->checkMedia($url);
```


## Bugtracker

Du hast einen Fehler gefunden oder ein nettes Feature parat? [Lege ein Issue an](https://github.com/FriendsOfREDAXO/video/issues). Bevor du ein neues Issue erstellst, suche bitte ob bereits eines mit deinem Anliegen existiert.

## Lizenz

siehe [LICENSE.md](https://github.com/FriendsOfREDAXO/video/blob/master/LICENSE.md)

Plyr und Afterglow stehen unter MIT-Lizenz. Die Player bedienen sich jedoch teils unterschiedlicher Quellen, deren Lizenzen sich unterscheiden können. 


## Autor

**Friends Of REDAXO**

* http://www.redaxo.org
* https://github.com/FriendsOfREDAXO

**Projekt-Lead**
[Thomas Skerbis](https://github.com/skerbis)


## Credits:

First Release: [Christian Gehrke](https://github.com/chrison94)
PlayLists: [Tobias Krais](https://github.com/tobiaskrais)
