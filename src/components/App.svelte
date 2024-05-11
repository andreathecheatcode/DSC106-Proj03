<script>
  import mapboxgl from 'mapbox-gl';
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let csvData;
  let map;
  let selectedYear = 2000;

  async function fetchData() {
    try {
      let csv_data = await d3.csv('internet_data.csv');
      return csv_data;
    } catch (error) {
      console.error('Error loading CSV:', error);
    }
  }

  async function updateMap() {
    try {
      mapboxgl.accessToken = 'pk.eyJ1IjoiaGlsbGFyeWNoYW5nIiwiYSI6ImNsdzB0bTUwcTA0bDcyaXFjaDEzZGduMXUifQ.PwESlw8F78LN1gE7GjuyMg';
      
      map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/light-v10',
        center: [-71.0942, 42.3601],
        zoom: 3,
        minZoom: 1,
        maxZoom: 15,
      });

      map.on('load', async () => {
        const response = await fetch('https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json');
        const countriesGeoJSON = await response.json();

        // filter CSV data based on year
        const filteredData = csvData.filter(d => +d.Year === selectedYear);

        // merge CSV with GeoJSON features
        countriesGeoJSON.features.forEach(feature => {
          const countryData = filteredData.find(data => data['Region/Country/Area'] === feature.properties.name);
          if (countryData) {
            const percentage = +countryData.Value;
            const hue = (100 - percentage) / 100 * 240; 
            feature.properties['fill-color'] = `hsl(${hue}, 100%, 50%)`;
          } else {
            feature.properties['fill-color'] = '#ccc';
          }
        });

        map.addSource('countries', {
          type: 'geojson',
          data: countriesGeoJSON
        });

        map.addLayer({
          'id': 'country-layer',
          'type': 'fill',
          'source': 'countries',
          'paint': {
            'fill-color': ['get', 'fill-color'],
            'fill-opacity': 0.7
          }
        });
      });
    } catch (error) {
      console.error('Error updating map:', error);
    }
  }

  onMount(async () => {
    try {
      csvData = await fetchData();
      updateMap();
    } catch (error) {
      console.error('Error loading CSV data:', error);
    }

    return () => {
      if (map) {
        map.remove();
      }
    };
  });

  function updateYear(event) {
    selectedYear = event.target.value;
    map.getSource('countries').setData(getFilteredGeoJSON());
  }

  function getFilteredGeoJSON() {
    const filteredData = csvData.filter(d => +d.Year === selectedYear);
    const filteredGeoJSON = JSON.parse(JSON.stringify(map.getSource('countries')._data));
    filteredGeoJSON.features.forEach(feature => {
      const countryData = filteredData.find(data => data['Region/Country/Area'] === feature.properties.name);
      if (countryData) {
        const percentage = +countryData.Value;
        const hue = (100 - percentage) / 100 * 240;
        feature.properties['fill-color'] = `hsl(${hue}, 100%, 50%)`;
      } else {
        feature.properties['fill-color'] = '#ccc';
      }
    });
    return filteredGeoJSON;
  }
</script>

<main>
  <div id="map"></div>
  <input type="range" min="2000" max="2015" bind:value={selectedYear} on:input={updateYear}>
  <p>Year: {selectedYear}</p>
</main>

<style>
  #map {
    width: 100%;
    height: 600px;
  }
</style>
