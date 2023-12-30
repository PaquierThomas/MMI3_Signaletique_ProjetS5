<script setup>

    // Import éléments de vue
    import { onMounted, ref, reactive } from 'vue'
    // Import leaflet
    import * as Leaflet from 'leaflet'
    // import geojson
    import { mapJSON } from '../data/map';
        
        // css leaflet
    import 'leaflet/dist/leaflet.css'
    

    // Canevas leaflet pour la carte
    let tileLayer = Leaflet.tileLayer
    // Initialisation de la carte sous forme de ref
    let map = ref()

    // Lorsque le composant est monté dans la vue
    // On affiche la carte
onMounted(async () => {

  // Caractéristiques visuelle de la carte
      // Avec Google Map Layout
    //   tileLayer = L.tileLayer('http://{s}.google.com/vt/lyrs=m&x={x}&y={y}&z={z}&',{
    //     maxZoom: 20,
    //     subdomains:['mt0','mt1','mt2','mt3']
    // });
      tileLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
        subdomains: 'abcd',
        maxZoom: 20,
      });
      
  
      // Création de la carte sur la div ayant : id='map'
      map = Leaflet.map('map',
      {
          zoomControl: true,    // Contrôle du Zoom
          layers: [tileLayer],  // Canvas pour dessiner la carte
          minZoom: 16            // Zomm mini autorisé
        })
      
      // projection de la carte avec centrage aux coordonnées indiquées, avec facteur d'agrandissemet
      // .setView([40.6892, -74.0445], 17) 
      .setView([47.4952, 	6.8045], 18) 
      // .setView(statueCoordinates, 15)

       
        
        // specify popup options 
        var customOptions =
            {
          'maxWidth': '400',
          'width': '200',
          'className' : 'custom'
          }
    

      // GEOJSON || AFFICHAGE DE TOUT LES BATIMENTS 
      L.geoJSON(mapJSON, {
        onEachFeature: function (feature, layer) {
            var content = `
              <div class="popup-container">
                <div class="left-section">
                  <img src="../../public/assets/bu.webp" alt="maptime logo gif" class="rounded-image" />
                </div>
                <div class="right-section">
                  <div class="info-column">
                    <h2>${feature.properties.name}</h2>
                    <h4>${feature.properties.description}</h4>
                  </div>
                  <div class="button-row">
                    <button class="round-button">But trajet</button>
                    <button class="round-button">But fav</button>
                  </div>
                </div>
              </div>
            `;
            layer.bindPopup(content, customOptions);
        },
        style: function (feature) {
          return {
                color: feature.properties.stroke,
                fillColor: feature.properties.fill,
                fillOpacity: feature.properties['fill-opacity'] || 1, 
            };
        }
    }).addTo(map);

      // Création d'un icone
      let myIcon = Leaflet.icon({
        iconUrl: 'assets/marker-icon.png', // Image de l'icône
        shadowUrl: 'assets/marker-shadow.png', // Image de l'ombre0
          iconSize:     [25, 41], // taille de l'icône
          shadowSize:   [25, 41], // taille de l'ombre
          iconAnchor:   [0, 0], // point de position de l'icône
          shadowAnchor: [-10, -10],  // point de position de l'ombre
          popupAnchor:  [0, 0] // point de position popup si elle existe, relativement à l'icône
        });
  
      // Ajout d'un marqueur
      let marker = Leaflet.marker([47.495328, 6.8044455], {icon: myIcon}).addTo(map)

      // Ajout d'une infobulle
      marker.bindPopup("Je suis un marker")
  
      // Recentrage de la carte au bout de 5 secondes
      // // à une autre position
      // setTimeout(function () { 
      //   map.panTo([47.50133850064826, 6.807621746718467])
      // }, 5000);

      
      // Ajout d'un marqueur
      // à la position de centrage
      let marker2 = Leaflet.marker(
          [47.50133850064826, 6.807621746718467], 
          {icon: myIcon}
        ).addTo(map)
  
      // Ajout d'une infobulle
      marker2.bindPopup("Je suis la gendarmerie nationale")
      // // Personnalisation du pop up leaflet



      });


    // Hors de onMounted
    // Coordonnées de l'utilisateur
    const coordMe = reactive({ latitude:0, longitude:0 })

    // Fonction de détection de la géolocalisation via navigateur
    const locMe = () => {
        let watcher = navigator.geolocation.watchPosition(
            // Fonction à appeler en cas de success
            showLocation
        )
    }
    
    // Fonction de sa localisation si elle réussi
    const showLocation = (position) => {
        // Récupération latitude et longitude
        coordMe.latitude = position.coords.latitude;
        coordMe.longitude = position.coords.longitude;
        // Recentrage de la carte sur la position utilisateur
        map.panTo([coordMe.latitude, coordMe.longitude])
        // Création d'un marqueur
        // L'icone ayant déja été instancié
        // On n'a pas à la préciser, on le reprend par défaut
        let markerMe = Leaflet.marker(
            [coordMe.latitude, coordMe.longitude],
        ).addTo(map)
        // Ajout d'une infobulle
        markerMe.bindPopup("Je suis là!!!!!!")
}




 
</script>

<template>
    <div>
        <!-- <button type="button" @click="locMe()"> Se localiser</button>
        <span v-if="coordMe">
          coordonnées : {{ coordMe.latitude }} - {{ coordMe.longitude}}
        </span>
        <span v-else>
            Pas de coordonnées
        </span>
        <hr/> -->
        <div class="container">
            <div class="custom-popup" id="map">
            </div>
        </div>
    </div>
    
</template>

<style>
    #map {
        width:100%;
        height: 100vh;
    } 

    .custom-popup .leaflet-popup-content-wrapper {
      color:rgb(0, 0, 0) ;
      font-size:16px;
      line-height:24px;
      border-radius: 15px;
      }
    .custom-popup .leaflet-popup-content-wrapper h2 {
        color:red;
      }
    .custom-popup .leaflet-popup-content-wrapper .rounded-image{
        border-radius: 5%;
      }
    .custom-popup .leaflet-popup-tip-container {
      width:30px;
      height:15px;
      }
    .custom-popup .leaflet-popup-tip {
        background: transparent;
        border: none;
        box-shadow: none;
      }

</style>