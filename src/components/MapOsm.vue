<script setup>

    // Import éléments de vue
    import { onMounted, ref, reactive } from 'vue'
    // Import leaflet
    import * as Leaflet from 'leaflet'
    // css leaflet
    import 'leaflet/dist/leaflet.css'

    // Canevas leaflet pour la carte
    let tileLayer = Leaflet.tileLayer
    // Initialisation de la carte sous forme de ref
    let map = ref()

    // Lorsque le composant est monté dans la vue
    // On affiche la carte
    onMounted( async () => {
      // Caractéristiques visuelle de la carte
      tileLayer = Leaflet.tileLayer('https://{s}.tile.osm.org/{z}/{x}/{y}.png',
        {
          attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
        }
      );
  
      // Création de la carte sur la div ayant : id='map'
      map = Leaflet.map('map',
      {
          zoomControl: true,    // Contrôle du Zoom
          layers: [tileLayer],  // Canevas pour dessiner la carte
          maxZoom: 18,          // Zoom maxi autorié
          minZoom: 6            // Zomm mini autorisé
      })
      // projection de la carte avec centrage aux coordonnées indiquées, avec facteur d'agrandissemet
      .setView([40.6892, -74.0445], 17) 
      // .setView(statueCoordinates, 15)

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

      // Après avoir initialisé la carte
      let statueCoordinates = [40.6892, -74.0445]; // Coordonnées de la Statue de la Liberté

      // Création d'un marqueur pour la Statue de la Liberté
      let statueMarker = Leaflet.marker(
        statueCoordinates,
        { icon: myIcon }
      ).addTo(map);

    // Ajout d'une infobulle au marqueur
    statueMarker.bindPopup("Statue de la Liberté");

    // Centrage de la carte sur la Statue de la Liberté
    // map.setView(statueCoordinates, 15); // Le nombre après les coordonnées représente le niveau de zoom

    // Après avoir initialisé la carte et les marqueurs de la Statue de la Liberté

let latitudeAdjustment = 100 / 111; // Approximation de la latitude en degrés pour 2 km
let longitudeAdjustment = 50 / (85 * Math.cos((statueCoordinates[0] * Math.PI) / 180)); // Approximation de la longitude en degrés pour 2 km (en tenant compte de la latitude)

// Coordonnées pour le carré autour de la Statue de la Liberté
let squareCoordinates = [
  [
    [statueCoordinates[1] - longitudeAdjustment, statueCoordinates[0] + latitudeAdjustment], // Coin supérieur gauche
    [statueCoordinates[1] + longitudeAdjustment, statueCoordinates[0] + latitudeAdjustment], // Coin supérieur droit
    [statueCoordinates[1] + longitudeAdjustment, statueCoordinates[0] - latitudeAdjustment], // Coin inférieur droit
    [statueCoordinates[1] - longitudeAdjustment, statueCoordinates[0] - latitudeAdjustment], // Coin inférieur gauche
    [statueCoordinates[1] - longitudeAdjustment, statueCoordinates[0] + latitudeAdjustment]  // Retour au coin supérieur gauche
  ]
];

// Créer une couche pour le carré à partir de GeoJSON
let squareGeoJSON = {
  type: 'FeatureCollection',
  features: [
    {
      type: 'Feature',
      properties: {
        name: 'Square around Statue of Liberty'
      },
      geometry: {
        type: 'Polygon',
        coordinates: squareCoordinates
      }
    }
  ]
};

// Assurez-vous que la carte est initialisée avant d'ajouter la couche du carré
if (map.value) {
  let squareLayer = Leaflet.geoJSON(squareGeoJSON).addTo(map.value);

  // Gestionnaire d'événements pour le survol du carré
  squareLayer.eachLayer(function (layer) {
    layer.on('mouseover', function () {
      // Modifier le style lors du survol
      layer.setStyle({
        weight: 3,
        color: 'red',
        fillOpacity: 0.7
        // Ajoutez d'autres styles de mise en évidence au survol
      });
    });

    layer.on('mouseout', function () {
      // Rétablir le style par défaut lorsque la souris quitte le carré
      squareLayer.resetStyle(layer);
    });
  });
}
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
        <button type="button" @click="locMe()"> Se localiser</button>
        <span v-if="coordMe">
          coordonnées : {{ coordMe.latitude }} - {{ coordMe.longitude}}
        </span>
        <span v-else>
            Pas de coordonnées
        </span>
        <hr/>
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

</style>