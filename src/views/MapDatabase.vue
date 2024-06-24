<template>
  <div id="app">
    <header>
      <h1>Ecos del Agua : Mapa Interactivo</h1>
    </header>
    <main>
      <div class="responsive-frame">
        <div id="left-column">
          <div id="map" ref="mapContainer"></div>
        </div>
        <div id="right-column">
          <div id="polygon-menu">
            <ul>
              <li v-for="(polygon, index) in polygons" :key="index" @click="showPolygonContent(polygon, index)">
                {{ polygon.nombre }}
              </li>
            </ul>
          </div>
          <div id="polygon-content">
            <h2>{{ selectedPolygon.nombre }}</h2>
            <div v-html="selectedPolygon.descripcion"></div>
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
        name: '',
        content: ''
      },
      hoveredPolygonId: null,
      selectedPolygonId: null,
      map: null
    };
  },
  mounted() {
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
      this.fetchPolygons(); // Mover la llamada a fetchPolygons dentro del evento 'load'
    });
  },
  methods: {
    async fetchPolygons() {
      try {
        const response = await axios.get('https://ecosbackend-production.up.railway.app/api/poligonos');
        this.polygons = response.data;

        // Agrega los polígonos al mapa
        this.polygons.forEach((polygon, index) => {
          const polygonId = `${polygon.nombre} ${index + 1}`;
          const coordinates = JSON.parse(polygon.coordenadas).map(coord => [coord.lng, coord.lat]);

          const polygonFeature = {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates: [coordinates] // Coordenadas en formato adecuado para Mapbox
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

          // Cambiar el color del polígono al pasar el mouse por encima
          this.map.on('mouseenter', polygonId, () => {
            this.hoveredPolygonId = polygonId;
            this.updatePolygonOpacity();
          });

          this.map.on('mouseleave', polygonId, () => {
            this.hoveredPolygonId = null;
            this.updatePolygonOpacity();
          });

          // Mostrar contenido del polígono al hacer clic en él
          this.map.on('click', polygonId, () => {
            this.showPolygonContent(polygon, index);
          });
        });
      } catch (error) {
        console.error('Error fetching polygons:', error);
      }
    },
    showPolygonContent(polygon, index) {
      this.selectedPolygon = polygon;
      this.selectedPolygonId = `${polygon.nombre} ${index + 1}`;
      this.hoveredPolygonId = null;
      this.updatePolygonOpacity();
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
}

main {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.responsive-frame {
  display: flex;
  flex: 1;
  flex-direction: row;
  max-width: 100%;
  max-height: 100%;
  overflow: hidden;
}

#left-column, #right-column {
  display: flex;
  flex: 1;
  flex-direction: column;
  overflow: hidden;
}

#map {
  width: 100%;
  height: 100%;
}

#polygon-menu {
  height: 25%;
  overflow-y: auto;
  background-color: #f0f0f0;
  padding: 10px;
}

#polygon-menu ul {
  list-style-type: none;
  padding: 0;
}

#polygon-menu li {
  cursor: pointer;
  padding: 5px;
  border-bottom: 1px solid #ccc;
}

#polygon-menu li:hover {
  background-color: #ddd;
}

#polygon-content {
  height: 25%;
  padding: 20px;
}
</style>
