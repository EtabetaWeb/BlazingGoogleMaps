Non c'è sito o applicazione che non abbia una mappa integrata! Grazie alla libreria [BlazorGoogleMaps](https://www.nuget.org/packages/BlazorGoogleMaps/) andiamo a vedere come aggiungerla nelle nostre applicazioni.



## Introduzione ai tutorial "Blazing Code!"

Questo articolo fa parte della serie "**[Blazing Code!](https://etabetaweb.github.io/BlazingCode/)**", un progetto innovativo per imparare a sviluppare **applicazioni con Blazor**.  

Per seguire il **tutorial** occorre scaricare il codice dal repository di **GitHub**: [Blazing Google Maps](https://github.com/EtabetaWeb/BlazingGoogleMaps).

Il repository è composto da due cartelle:

- **AppCode**: contenente il codice completo e pronto per l'esecuzione;
- **TutorialCode**: contenente i files per seguire il tutorial; 

Nei files di "**TutorialCode**" sono presenti i "**CODEPOINTS**", questi sono i **segnaposto** che andremo a **sostituire con il codice** riportato in questo tutorial. 

Per ogni "CODEPOINT", nel tutorial, viene specificata la cartella, il file e la riga corrispondente dove inserire il codice. 



Ora che abbiamo introdotto come utilizzare il tutorial iniziamo a sviluppare la nostra applicazione!



## Richiediamo l'APIKey di Google Maps

Prima di sviluppare il codice occorre, se già non ne abbiamo una, l'APIKey per Google Maps. Per farlo dobbiamo andare sul sito [CloudGoogle](https://cloud.google.com/console/google/maps-apis/overview). 

**NOTA: nel codice completo (quello presente nella cartella "AppCode") è già presente una APIKey per i test locali ma vi suggerisco comunque di richiederne una nuova poiché quella fornita potrebbe avere delle limitazioni sull'utilizzo.**



## Aggiungiamo il riferimento a Google Maps

Ottenuta l'APIKey, apriamo il file "**_Host.cshtml**" che si trova nella cartella "**Pages**" ed andiamo a sostituire il "CODEPOINT: 01" con il seguente codice ricordandoci di sostituire il testo "**INSERIRE-GOOGLE-APIKEY**" con la chiave che avremo precedentemente richiesto.

##### CODEPOINT:01

- Pagina: Pages/_Host.cshhtml
- Riga: 18

```javascript
<script type="text/javascript" src="https://maps.googleapis.com/maps/api/js?key=**INSERIRE-GOOGLE-APIKEY**&v=3"></script>
```



Sempre in questo file andiamo ad inserire l'object manager della libreria.

##### CODEPOINT:02

- Pagina: Pages/_Host.cshhtml
- Riga: 39

```javascript
<script src="_content/BlazorGoogleMaps/objectManager.js"></script>
```



## Cerchiamo un indirizzo

La funzionalità che andremo ad inserire nella nostra applicazione è la ricerca di un indirizzo da visualizzare. 

Apriamo il file "**CercaIndirizzo.razor**" presente nella cartella "**Pages**" ed inseriamo per prima cosa le direttive "**using**".



##### CODEPOINT:03

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 3

```java
@using GoogleMapsComponents
@using GoogleMapsComponents.Maps
@using GoogleMapsComponents.Maps.Places
```



Aggiungiamo un **box di ricerca** con la visualizzazione del testo di ritorno.



##### CODEPOINT:04

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 7

```html
<div style="margin-bottom: 10px;">
    <input type="text" @ref="this.searchBox" id="searchBox" />
    <button @onclick="Search">Cerca</button>
</div>
<div>
    <p style="font-weight: bold; font-size: 1.2em">@this.message</p>
</div>
```



Aggiungiamo ora la mappa.



##### CODEPOINT:05

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 10

```html
<GoogleMap @ref="@(this.map1)" Id="map1" Options="@(this.mapOptions)" OnAfterInit="async () => await OnAfterMapInit()"></GoogleMap>
```



Ed inizializziamo le variabili.



##### CODEPOINT:06

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 15

```c#
    private readonly Stack<Marker> markers = new Stack<Marker>();
    private GoogleMap map1;
    private MapOptions mapOptions;
    private Autocomplete autocomplete;
    private string message;
    private ElementReference searchBox;
```



Aggiungiamo il metodo **OnInitialized** per impostare lo zoom ed il centro della mappa.



##### CODEPOINT:07

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 18

```c#
protected override void OnInitialized()
    {
        this.mapOptions = new MapOptions
        {
            Zoom = 5,
            Center = new LatLngLiteral
            {
                Lat = 43.034126,
                Lng = 12.037817
            },
            MapTypeId = MapTypeId.Roadmap
        };
    }
```



Aggiungiamo il metodo **OnAfterMapInit** che visualizza l'indirizzo sulla mappa con il relativo segnaposto.



##### CODEPOINT:08

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 21

```c#
private async Task OnAfterMapInit()
    {
        this.autocomplete = await Autocomplete.CreateAsync(this.map1.JsRuntime, this.searchBox, new AutocompleteOptions
        {
            StrictBounds = false
        });

        await this.autocomplete.SetFields(new[] { "address_components", "geometry", "name" });

        await this.autocomplete.AddListener("place_changed", async () =>
        {
            var place = await this.autocomplete.GetPlace();

            if (place?.Geometry == null)
            {
                this.message = "L'indirizzo " + place?.Name + " non è stato trovato!";
            }
            else if (place.Geometry.Location != null)
            {
                await this.map1.InteropObject.SetCenter(place.Geometry.Location);
                await this.map1.InteropObject.SetZoom(13);

                var marker = await Marker.CreateAsync(this.map1.JsRuntime, new MarkerOptions
                {
                    Position = place.Geometry.Location,
                    Map = this.map1.InteropObject,
                    Title = place.Name
                });

                this.markers.Push(marker);

                this.message = "Risultati trovati per " + place.Name;
            }
            else if (place.Geometry.Viewport != null)
            {
                await this.map1.InteropObject.FitBounds(place.Geometry.Viewport, 5);
                this.message = "Risultati trovati per " + place.Name;
            }

            this.StateHasChanged();
        });
    }
```



Per completare il codice della nostra pagina di ricerca, aggiungiamo un metodo vuoto **Search** che ci serve come workaround.



##### CODEPOINT:09

- Pagina: Pages/CercaIndirizzo.razor
- Riga: 24

```c#
    private void Search()
    {

    }
```



## Eseguiamo il codice

Ora che abbiamo inserito tutto il codice possiamo eseguire la nostra applicazione premendo [F5] ed ecco che dal menu laterale selezionando la voce "Cerca indirizzo" accederemo alla nostra pagina di ricerca.

Si conclude qui il tutorial per aggiungere una **Google Map** nelle nostre applicazioni **BLAZOR**.
