<template>
  <div id="app">
    <header>
      <h1>Ecos del Agua : Mapa Interactivo</h1>
      <div id="polygon-dropdown">
        <select v-model="selectedPolygonId" @change="showPolygonContentByDropdown">
          <option v-for="(polygon, index) in polygons" :key="index" :value="`${polygon.nombre} ${index + 1}`">
            {{ polygon.nombre }}
          </option>
        </select>
      </div>
    </header>
    <main>
      <div id="map" ref="mapContainer"></div>
      <!-- The Modal -->
      <div id="myModal2" class="modal">

        <!-- Modal content -->
        <div class="modal-content">
          <div class="modal-header">
            <span class="close" @click="closeModal">&times;</span>
            <h2>{{ selectedPolygon.nombre }}</h2>
          </div>
          <div class="modal-body">
            <div v-html="selectedPolygon.descripcion"></div>
          </div>
          <div class="modal-footer">
            <p>{{ selectedPolygon.coordenadas }}</p>
          </div>
        </div>

      </div>
    </main>
  </div>



</template>
<script>
import axios from 'axios';
import mapboxgl from 'mapbox-gl';



export default {
  data() {
    return {
      polygons: [],
      selectedPolygon: {
        nombre: '',
        descripcion: ''
      },
      hoveredPolygonId: null,
      selectedPolygonId: null,
      map: null,
      modalVisible: false
    };
  },
  mounted() {

    var modal = document.getElementById("myModal2");

    // When the user clicks anywhere outside of the modal, close it
    window.onclick = function(event) {
      if (event.target == modal) {
        modal.style.display = "none";
      }
    }

    mapboxgl.accessToken = 'pk.eyJ1IjoicGF1bHRvb2wiLCJhIjoiY2tpbTNveHY5MHB1ODJ4bmFzeG5idzVwNyJ9.Hp_t8btogzDqOMFyOKjWyA';

    this.map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/satellite-v9',
      center: [-99.137539, 19.788065],
      zoom: 13,
      attributionControl: false
    });

    this.$nextTick(() => {
      this.map.resize();
    });

    this.map.on('load', () => {
      this.fetchPolygons();
    });
  },
  methods: {
    async fetchPolygons() {
      try {
        const response = await axios.get('https://ecosbackend-production.up.railway.app/api/poligonos');
        this.polygons = response.data;

        this.polygons.forEach((polygon, index) => {
          const polygonId = `${polygon.nombre} ${index + 1}`;
          const coordinates = JSON.parse(polygon.coordenadas).map(coord => [coord.lng, coord.lat]);

          const polygonFeature = {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates: [coordinates]
            },
            properties: {
              name: polygonId,
              content: polygon.descripcion
            }
          };

          this.map.addLayer({
            id: polygonId,
            type: 'fill',
            source: {
              type: 'geojson',
              data: {
                type: 'FeatureCollection',
                features: [polygonFeature]
              }
            },
            layout: {},
            paint: {
              'fill-color': '#088',
              'fill-opacity': [
                'case',
                ['==', ['get', 'name'], this.hoveredPolygonId], 0.7,
                ['==', ['get', 'name'], this.selectedPolygonId], 0.9,
                0.5
              ]
            }
          });

          this.map.on('mouseenter', polygonId, () => {
            this.hoveredPolygonId = polygonId;
            this.updatePolygonOpacity();
          });

          this.map.on('mouseleave', polygonId, () => {
            this.hoveredPolygonId = null;
            this.updatePolygonOpacity();
          });

          this.map.on('click', polygonId, () => {
            this.showPolygonContent(polygon, index);
          });
        });
      } catch (error) {
        console.error('Error fetching polygons:', error);
      }
    },
    showPolygonContent(polygon, index) {
      var modal = document.getElementById("myModal2");
      this.selectedPolygon = polygon;
      this.selectedPolygonId = `${polygon.nombre} ${index + 1}`;
      this.hoveredPolygonId = null;
      this.updatePolygonOpacity();
      modal.style.display = "block";
    },
    showPolygonContentByDropdown() {
      const selected = this.polygons.find((polygon, index) => `${polygon.nombre} ${index + 1}` === this.selectedPolygonId);
      if (selected) {
        const index = this.polygons.indexOf(selected);
        this.showPolygonContent(selected, index);
      }
    },
    closeModal() {
      var modal = document.getElementById("myModal2");
      this.modalVisible = false;
      modal.style.display = "none";
    },
    updatePolygonOpacity() {
      if (this.map && this.map.getStyle() && this.map.getStyle().layers) {
        this.polygons.forEach((polygon, index) => {
          const polygonId = `${polygon.nombre} ${index + 1}`;
          if (this.map.getLayer(polygonId)) {
            this.map.setPaintProperty(polygonId, 'fill-opacity', polygonId === this.selectedPolygonId ? 0.9 : (polygonId === this.hoveredPolygonId ? 0.7 : 0.5));
          }
        });
      }
    }
  }
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

header {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

#polygon-dropdown {
  margin-right: 20px;
}

main {
  display: flex;
  flex: 1;
  overflow: hidden;
}

#map {
  width: 100%;
  height: 100%;
}


/* Modal Header */
.modal-header {
  padding: 2px 16px;
  background-color: #5cb85c;
  color: white;
}

/* Modal Body */
.modal-body {padding: 2px 16px;}

/* Modal Footer */
.modal-footer {
  padding: 2px 16px;
  background-color: #5cb85c;
  color: Black;
}

/* Modal /* The Modal (background) */
.modal {
  display: none; /* Hidden by default */
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  padding-top: 100px; /* Location of the box */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
}

/* Modal Content */
.modal-content {
  position: relative;
  background-color: #fefefe;
  margin: auto;
  padding: 0;
  border: 1px solid #888;
  width: 80%;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2),0 6px 20px 0 rgba(0,0,0,0.19);
  -webkit-animation-name: animatetop;
  -webkit-animation-duration: 0.4s;
  animation-name: animatetop;
  animation-duration: 0.4s
}

/* Add Animation */
@-webkit-keyframes animatetop {
  from {top:-300px; opacity:0} 
  to {top:0; opacity:1}
}

/* The Close Button */
.close {
  color: white;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}


</style>
