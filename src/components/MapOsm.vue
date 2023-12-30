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

    

      // GEOJSON || AFFICHAGE DE TOUT LES BATIMENTS 
      L.geoJSON(mapJSON, {
        onEachFeature: function (feature, layer) {
            layer.bindPopup(feature.properties.name);
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

// Assurez-vous que la carte est initialisée avant d'ajouter la couche du carré
// if (map.value) {
//   let squareLayer = Leaflet.geoJSON(squareGeoJSON).addTo(map.value);

//   // Gestionnaire d'événements pour le survol du carré
//   squareLayer.eachLayer(function (layer) {
//     layer.on('mouseover', function () {
//       // Modifier le style lors du survol
//       layer.setStyle({
//         weight: 3,
//         color: 'red',
//         fillOpacity: 1
//         // Ajoutez d'autres styles de mise en évidence au survol
//       });
//     });

//     layer.on('mouseout', function () {
//       // Rétablir le style par défaut lorsque la souris quitte le carré
//       squareLayer.resetStyle(layer);
//     });
//   });
// }
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



// // Personnalisation du pop up leaflet
// //  create map object, tell it to live in 'map' div and give initial latitude, longitude, zoom values 
//     // create custom icon
//     var firefoxIcon = Leaflet.icon({
//         iconUrl: 'http://joshuafrazier.info/images/firefox.svg',
//         iconSize: [38, 95], // size of the icon
//         popupAnchor: [0,-15]
//         });

//     // create popup contents
//     var customPopup = "Mozilla Toronto Offices<br/><img src='http://joshuafrazier.info/images/maptime.gif' alt='maptime logo gif' width='350px'/>";
    
//     // specify popup options 
//     var customOptions =
//         {
//         'maxWidth': '500',
//         'className' : 'custom'
//         }
    
//     // create marker object, pass custom icon as option, pass content and options to popup, add to map
//     Leaflet.marker([47.4952, 	6.8045], {icon: firefoxIcon}).bindPopup(customPopup,customOptions).addTo(map);


 
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
            <div id="map">
            </div>
        </div>
    </div>
    
</template>

<style scoped>
    #map {
        width:100%;
        height: 100vh;
    } 

    /* css to customize Leaflet default styles  */
  .custom .leaflet-popup-tip,
  .custom .leaflet-popup-content-wrapper {
      background: #e93434;
      color: #ffffff;
  }
</style>