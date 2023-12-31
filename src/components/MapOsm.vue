


<script setup>

    // Import éléments de vue
    import { onMounted, ref, reactive } from 'vue'
    // Import leaflet
    import * as Leaflet from 'leaflet'
    // import geojson
    import { mapJSON } from '../data/map';
    import { markerJSON } from '../data/marker';
        
        // css leaflet
    import 'leaflet/dist/leaflet.css'
    

    // Canvas leaflet pour la carte
    let tileLayer = Leaflet.tileLayer
    // Initialisation de la carte sous forme de ref
let map = ref()

let userMarker = null; // Variable pour stocker le marqueur de l'utilisateur

function geoLocateUser(map) {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(function (position) {
      var userLat = position.coords.latitude;
      var userLng = position.coords.longitude;

      userMarker = L.marker([userLat, userLng]).addTo(map);
      map.setView([userLat, userLng], 15); // Ajustez le niveau de zoom ici

      function onLocationFound(e) {
        var radius = e.accuracy / 2;
        L.circle(e.latlng, radius).addTo(map);
      }

      function onLocationError(e) {
        alert(e.message);
      }

      map.on('locationfound', onLocationFound);
      map.on('locationerror', onLocationError);

      map.locate({ setView: true, maxZoom: 16 });

      // Recentrer la carte sur la position de l'utilisateur après 5 secondes d'inactivité
      let timeoutID = null;
      map.on('move', function () {
        clearTimeout(timeoutID);
        timeoutID = setTimeout(function () {
          map.setView([userLat, userLng], 15);
        }, 30000); // Temps d'attente en millisecondes (ici, 5 secondes)
      });
    });
  } else {
    alert("La géolocalisation n'est pas disponible sur votre appareil.");
  }
}






function openGoogleMapsDirections(buildingCoordinateY, buildingCoordinateX) {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(function (position) {
      var userLat = position.coords.latitude;
      var userLng = position.coords.longitude;

      var googleMapsURL = `https://www.google.com/maps/dir/?api=1&origin=${userLat},${userLng}&destination=${buildingCoordinateY},${buildingCoordinateX}`;
      
      // Ouvrir une nouvelle fenêtre ou un nouvel onglet avec l'URL pour les directions
      window.open(googleMapsURL, '_blank');
    });
  } else {
    alert("La géolocalisation n'est pas disponible sur votre appareil.");
  }
}
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
          minZoom: 5            // Zomm mini autorisé
        })
      
      // projection de la carte avec centrage aux coordonnées indiquées, avec facteur d'agrandissemet
      // .setView([40.6892, -74.0445], 17) 
      .setView([47.4952, 	6.8045], 18) 
  // .setView(statueCoordinates, 15)
  L.control.scale().addTo(map);

  geoLocateUser(map);

       
        
        // specify popup options 
        var customOptions =
            {
          'maxWidth': '400',
          'width': '200',
          'className' : 'custom'
          }
    

      // GEOJSON || AFFICHAGE DE TOUT LES BATIMENTS 
      // Votre code existant pour la création des bâtiments avec L.geoJSON
L.geoJSON(mapJSON, {
  onEachFeature: function (feature, layer) {
    var content = `
      <div class=" bg-white rounded-2xl flex flex-row items-center gap-5">
        <div>
          <img class="rounded-2xl min-w-60" src="../../public/assets/${feature.properties.image}" alt="image du bâtiment ${feature.properties.image}" />
        </div>
        <div class="flex flex-col gap-2">
          <h2 class="font-semibold text-base">${feature.properties.name}</h2>
          <p>${feature.properties.description}</p>
          <div class="flex flex-row items-center justify-end gap-3">
            <button id="directionsButton">
              <svg width="40" height="40" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M38.8154 17.17L22.8414 1.17C21.282 -0.39 18.743 -0.39 17.1836 1.17L1.16956 17.17C-0.389854 18.73 -0.389854 21.27 1.16956 22.83L17.1836 38.83C18.743 40.39 21.282 40.39 22.8414 38.83L38.8154 22.83C40.3949 21.25 40.3949 18.73 38.8154 17.17ZM23.0014 24.99V19.99H16.004V25.99H12.0055V17.99C12.0055 16.89 12.9052 15.99 14.0047 15.99H23.0014V10.99L29.9988 17.99L23.0014 24.99Z" fill="#151515"/>
              </svg>
            </button>
            <button class="flex flex-row bg-Bleu rounded-2xl text-white p-2 gap-1 items-center">
              <svg width="30" height="30" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M6.25 26.25V6.25C6.25 5.5625 6.495 4.97375 6.985 4.48375C7.475 3.99375 8.06333 3.74917 8.75 3.75H21.25C21.9375 3.75 22.5263 3.995 23.0163 4.485C23.5063 4.975 23.7508 5.56334 23.75 6.25V26.25L15 22.5L6.25 26.25Z" fill="white"/>
              </svg>
              Enregistrer
            </button>
          </div>
        </div>
      </div>
    `;
    layer.on({
      mouseover: function (e) {
        layer.setStyle({
          fillColor: feature.properties.fill,
          fillOpacity: 1,
          weight: 3 // Épaisseur de la bordure à 2
        });
      },
      mouseout: function (e) {
        layer.setStyle({
          fillColor: feature.properties.fill,
          fillOpacity: feature.properties['fill-opacity'] || 1,
          weight: 2 // Rétablir l'épaisseur de la bordure à 1 par défaut
        });
      },
      click: function (e) {
        // Ajoutez la logique ici pour ouvrir la popup avec le contenu 'content'
        layer.bindPopup(content, customOptions).openPopup();

        // Récupérer les coordonnées du bâtiment depuis feature.properties.coordinate
        var buildingCoordinates = feature.properties.coordinate;

        // Sélectionner le bouton existant par son ID
        var directionsButton = document.getElementById('directionsButton');

        // Ajouter un événement de clic à ce bouton pour ouvrir les directions
        directionsButton.addEventListener('click', function() {
          openGoogleMapsDirections(buildingCoordinates[0], buildingCoordinates[1]);
        });

      }
    });
  },
  style: function (feature) {
    return {
      color: feature.properties.stroke || 'black', // Couleur de la bordure par défaut
      fillColor: feature.properties.fill || 'gray', // Couleur de remplissage par défaut
      fillOpacity: feature.properties['fill-opacity'] || 1,
      weight: 3 // Épaisseur de la bordure par défaut
    };
  }
}).addTo(map);


      // Création d'un icone
      // let myIcon = Leaflet.icon({
      //   iconUrl: 'assets/marker-icon.png', // Image de l'icône
      //   shadowUrl: 'assets/marker-shadow.png', // Image de l'ombre0
      //     iconSize:     [25, 41], // taille de l'icône
      //     shadowSize:   [25, 41], // taille de l'ombre
      //     iconAnchor:   [0, 0], // point de position de l'icône
      //     shadowAnchor: [-10, -10],  // point de position de l'ombre
      //     popupAnchor:  [0, 0] // point de position popup si elle existe, relativement à l'icône
      // });

      let homeIcon = Leaflet.icon({
        iconUrl: 'assets/icons/home.png', // Image de l'icône
      })
      let bookIcon = Leaflet.icon({
        iconUrl: 'assets/icons/books.png', // Image de l'icône
      })
      let restaurantIcon = Leaflet.icon({
        iconUrl: 'assets/icons/restaurants.png', // Image de l'icône
      })
      let busIcon = Leaflet.icon({
        iconUrl: 'assets/icons/transport.png', // Image de l'icône
      })
  
      // Ajout d'un marqueur
  // let marker = Leaflet.marker([47.495328, 6.8044455], { icon: myIcon }).addTo(map)
  // // Ajout d'une infobulle
  // marker.bindPopup("Je suis un marker")


 

  
     

      
      // Ajout d'un marqueur
      // à la position de centrage
      // let marker2 = Leaflet.marker(
      //     [47.50133850064826, 6.807621746718467], 
      //     {icon: myIcon}
      //   ).addTo(map)
  
      // // Ajout d'une infobulle
      // marker2.bindPopup("Je suis la gendarmerie nationale")
  // Marqueurs Home
  let home = Leaflet.marker([47.497122525563675, 6.803086062142094], { icon: homeIcon }).addTo(map)
  let residence = Leaflet.marker([47.49410392250028, 6.8016228610181555], { icon: homeIcon }).addTo(map)
  let bu = Leaflet.marker([47.4955579890414, 6.803860303027683], { icon: bookIcon }).addTo(map)
  let ruclinique = Leaflet.marker([47.49729338311438, 6.804366571950241], { icon: restaurantIcon }).addTo(map)
  let ru = Leaflet.marker([47.496153716474424, 6.803533505852742], { icon: restaurantIcon }).addTo(map)
  let bus = Leaflet.marker([47.495794898290534, 6.804027032291379], { icon: busIcon }).addTo(map)

// Ajoutez un événement click à l'élément DOM en utilisant addEventListener
  // Sélectionnez l'élément avec querySelector ou une autre méthode
  document.addEventListener('click', function (event) {
  if (event.target.classList.contains('directions-button')) {
    var buildingName = event.target.getAttribute('data-building-name');
    console.log('Bouton cliqué pour :', buildingName);
  }
});
});


  




 
</script>

<template>
    <div>
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

    /* .custom-popup .leaflet-popup-content-wrapper {
      color: black;
      }
    .custom-popup .leaflet-popup-content-wrapper h2 {
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
      } */

</style>