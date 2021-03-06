# BlobImageSL

Visualization of Blob Image with SpatiaLite and QGIS / Visualizzazione di Blob Image con SpatiaLite e QGIS

<!-- TOC -->

- [BlobImageSL](#blobimagesl)
  - [Why this repository / Perché questo repository](#why-this-repository--perché-questo-repository)
  - [Script](#script)
    - [Script action Python / script azione Python](#script-action-python--script-azione-python)
    - [Custom function for field calc / Funzione personalizzata per field calc](#custom-function-for-field-calc--funzione-personalizzata-per-field-calc)
    - [Expression for widget HTML / Espressione per widget HTML](#expression-for-widget-html--espressione-per-widget-html)
    - [Frame HTML for Atlas / Cornice HTML per Atlas](#frame-html-for-atlas--cornice-html-per-atlas)
  - [References / Riferimenti](#references--riferimenti)
  - [data and QGIS project](#data-and-qgis-project)
  - [Video demo](#video-demo)
  - [Screenshot](#screenshot)
    - [Action / Azione](#action--azione)
    - [Map tips / Suggerimenti mappa](#map-tips--suggerimenti-mappa)
    - [Widget HTML from module](#widget-html-from-module)
    - [Atlas](#atlas)

<!-- /TOC -->

## Why this repository / Perché questo repository

[**EN**] The [repository](https://pigrecoinfinito.com/2017/06/18/qgis-visualizzare-blob-image-spatialite/) was born after three years from my blog post on Pigrecoinfinito in which I describe how to view photos, with geotags, starting from a SpatiaLite database: a database created using [ImportEXIFphotos](http://www.gaia-gis.it/gaia-sins/spatialite-exif-2.3.1.html) that transforms photos into `BLOB` format.

The repository wants to document all the steps and scripts necessary to be able to use the photos (with geotags) and metadata directly in QGIS, for example viewing photos via Python action, map suggestions, modules via widgets and print composer.

[**ITA**] Il repository nasce dopo tre anni da [questo mio](https://pigrecoinfinito.com/2017/06/18/qgis-visualizzare-blob-image-spatialite/) blog post su Pigrecoinfinito in cui descrivo come visualizzare delle foto, con geotag, a partire da un database SpatiaLite: database realizzato utilizzando [ImportEXIFphotos](http://www.gaia-gis.it/gaia-sins/spatialite-exif-2.3.1.html) che trasforma le foto in formato `BLOB`.

Il repository vuole documentare tutti i passaggi e gli script necessari per poter utilizzare le foto (con geotag) e i metadati direttamente in QGIS, per esempio visualizzazione delle foto tramite _azione Python_, _suggerimenti mappa_, _moduli tramite widget_ e _compositore di stampe_.

↑[come back](#blobimagesl)↑

## Script

### Script action Python / script azione Python

[**EN**] In line 7 replace the unique_field_name value with the name of a unique field (perhaps the primary key), line 8 must contain the name of the field where the photos were stored and finally in line 9 the same value must be put as in line 5 in this case leaving the string '[%%]'.

[**ITA**] Alla riga 7 sostituire il valore unique_field_name con il nome di un campo univoco (magari la chiave primaria), la riga 8 deve contenere il nome del campo dove sono state memorizzate le foto ed infine alla riga 9 va messo lo stesso valore come alla riga 5 lasciando in questo caso la stringa ‘[% %]’.

```python
from qgis.PyQt.QtCore import Qt
from qgis.PyQt.QtWidgets import QLabel
from qgis.PyQt.QtGui import QImage, QPixmap
from qgis.utils import iface
import sqlite3

UNIQUE_FIELD = "unique_field_name"
BLOB_FIELD = "picture_field_name"
FIELD_VALUE = '[%unique_field_name%]'

vl = iface.activeLayer()
pr = vl.dataProvider()
db = pr.dataSourceUri().split(' ')[0].replace('dbname=','').replace("'","")
conn = sqlite3.connect(db)
sql = "SELECT {0} FROM {1} WHERE {2}={3}".format(BLOB_FIELD, vl.name(), 
                                                 UNIQUE_FIELD, FIELD_VALUE)
c = conn.cursor()
c.execute(sql)
rows = c.fetchone()
c.close()
conn.close()

qimg = QImage.fromData(rows[0])
pixmap = QPixmap.fromImage(qimg)

label = QLabel()
h = label.height()
w = label.width()
titolo = "[%name%]"
label.setWindowTitle(titolo)
label.setPixmap(pixmap.scaled(w,h,Qt.KeepAspectRatio))
label.show()
```

- **Author**: Salvatore Larosa
- **Article**: https://slarosagis.wordpress.com/2017/06/18/mai-visto-una-blob-image-da-qgis/

↑[come back](#blobimagesl)↑

### Custom function for field calc / Funzione personalizzata per field calc

Utilizzato per i suggerimenti mappa (visualizzare la foto con il passaggio del mouse)

```python
from qgis.core import *
from qgis.gui import *

@qgsfunction(args='auto', group='Custom', handlesnull=True)
def blobjpg_to_html(blob,style,feature,parent):
    """
    Restituisce il blob (che deve essere un jpeg) convertito in HTML img data url per visualizzarlo
    <br> in un Widget HTML o in un suggerimento mappa.
    <br>E' obbligatorio il secondo parametro style. 
    <br>Il parametro style puo' assumere il valore '' per nessuno stile (dimensioni originali) 
    <br>o puo' essere una stringa CSS di stile per img HTML tag,
    <br>Se stile = 'Null' viene applicato di default 'style="max-width:100%; max-height:100%;'.
    <p style="color:Olive"><b>Sintassi</b></p>
    <p style="color:blue"><b>blobjpg_to_html</b><mark style="color:black">(</mark>
    <mark style="color:red">blob</mark><mark style="color:black">,</mark><mark style="color:red">style</mark><mark style="color:black">)</mark>
    <p style="color:Olive"><b>Argomenti</b></p>
    <p style="color:red"><b>blob  </b><mark style="color:black"> - campo contenente i dati blob</mark><br>
    <mark style="color:red"><b>style </b><mark style="color:black"> - campo contenente stringa CSS</mark>
    <p style="color:Olive"><b>Esempi</b></p>
        <ul>
            <li><mark><i> blobjpg_to_html("photo", '') -> tag img con immagine a risoluzione originale</mark></li>
            <li><mark><i> blobjpg_to_html("photo", Null) -> tag img con dimensioni massime </mark></li>
            <li><mark><i> blobjpg_to_html("photo", 'width="250" height="250"')   -> tag img dimensionato</mark></li>
        </ul>
    <p style="color:Olive"><b>Ringraziamenti</b></p>
    <p>Tratto da https://gis.stackexchange.com/questions/350541/display-photo-stored-as-blob-in-gpkg</p>
    """
    blob64 = blob.toBase64().data().decode()
    if style is None:
        stylestring = 'style="max-width:100%; max-height:100%;"'
    elif not(style):
        stylestring = 'style=""'
    else:
        stylestring = 'style="' + style + '"'
    fullstring = '<img src="data:image/jpeg;base64,' + blob64 + '" ' + stylestring + ' alt="Invalid jpeg">'
    return fullstring
```

- **Author**: Giulio Fattori

↑[come back](#blobimagesl)↑

### Expression for widget HTML / Espressione per widget HTML

```html
<script>document.write(expression.evaluate(" blobjpg_to_html( \"photo\",'width=\"300\" height=\"420\"')"));</script>
``` 

- **Author**: Giulio Fattori

### Frame HTML for Atlas / Cornice HTML per Atlas

```
[% blobjpg_to_html("Photo",'width="450" height="650"') %]
```

↑[come back](#blobimagesl)↑

- **Author**: Giulio Fattori

## References / Riferimenti

- **blog post by Totò Fiandaca** : <https://pigrecoinfinito.com/2017/06/18/qgis-visualizzare-blob-image-spatialite/>
- **blog post by Salvatore Larosa** : <https://slarosagis.wordpress.com/2017/06/18/mai-visto-una-blob-image-da-qgis/>
- **gis.stackexchange**: <https://gis.stackexchange.com/questions/350541/display-photo-stored-as-blob-in-gpkg>
- **SpatiaLite EXIF GPS** : <http://www.gaia-gis.it/gaia-sins/spatialite-exif-2.3.1.html>

## data and QGIS project

- Directory `risorse` : <https://github.com/pigreco/BlobImageSL/tree/master/risorse>
  - database sqlite;
  - qgz project file;
  - qtp print layout file;
  - qml theming file.

## Video demo

[![add_col_area_perimetro](https://img.youtube.com/vi/BwCSSMRsf3o/0.jpg)](https://youtu.be/BwCSSMRsf3o "BLOB IMAGE SPATIALITE")

↑[come back](#blobimagesl)↑

## Screenshot

### Action / Azione

![](imgs/action.png)

![](imgs/action_view.png)

↑[come back](#blobimagesl)↑

### Map tips / Suggerimenti mappa

![](imgs/field_calc1.png)

![](imgs/field_calc2.png)

![](imgs/field_calc3.png)

![](imgs/maptips1.png)

↑[come back](#blobimagesl)↑

### Widget HTML from module

![](imgs/widget1.png)

![](imgs/widget2.png)

↑[come back](#blobimagesl)↑

### Atlas

![](imgs/ATLAS.png)

↑[come back](#blobimagesl)↑
