Pasos para scrappear las oficinas de empleo
-------------------------------------------

Fuente:

http://www.trabajo.gov.ar/redempleo/directorio.asp

http://www.trabajo.gov.ar/redempleo/directorio_res.asp?txtProv=1
http://www.trabajo.gov.ar/redempleo/directorio_res.asp?txtProv=3
etc..

1. Copiamos las páginas que queremos scrappear
sacamos provincia y provincia_id con sublime text

Creamos un csv de la forma:

provincia_id,provincia
1,Buenos Aires
3,Catamarca
4,Chaco
[...]

2. Abrimos open refine
Nuevo proyecto desde el clipboard
pegamos el csv

add column based on url

'http://www.trabajo.gov.ar/redempleo/directorio_res.asp?txtProv=' + value

3. extraemos los municipios del html

value.parseHtml().select('.municipioCont').join('|||')

4. Edit cells, split multi-valued cells


5. leemos el municipio, tipo y la informacion de la oficina

municipio
htmlText(value.parseHtml().select('[id^=oficinaMas] div')[0])

tipo
htmlText(value.parseHtml().select('table[width=410] td')[0])

oficina_text
htmlText(value.parseHtml().select('table[width=410] td')[1])

direccion
match(value, /Dirección: (.*) Teléfono.*/)[0]

telefono
match(value, /.*Teléfono: (.*) E-mail.*/)[0]

6. sacamos el (OE) y (UE)

value.replace('(UE)', '').replace('(OE)', '')

---

geocodificacion

1.creamos columna direccion_geo

cells.direccion.value + ', ' +
cells.municipio.value + ', Provincia de ' +
cells.provincia.value + ', Argentina'

2. sacamos PB(xxxx)

partition(value, /, PB\(\d\d\d\d\)/)[0] +
partition(value, /, PB\(\d\d\d\d\)/)[2]

3. consumimos web service de google - creamos columna direccion_json

'http://maps.googleapis.com/maps/api/geocode/json?sensor=false&address=' + value.escape('url')

4. eliminamos las que fallaron

value.parseJson().lenght()

4. parseamos lat y lon
value.parseJson().results[0].geometry.location.lat.toNumber()
value.parseJson().results[0].geometry.location.lon.toNumber()

exportamos a cartodb

hacemos alguna visualizacion

http://opensas.cartodb.com/api/v2/sql?q=select * from oficinas_empleo limit 10






