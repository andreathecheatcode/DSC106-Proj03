<script>
  import mapboxgl from 'mapbox-gl';
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let csvData;
  let map;

  async function fetchData() {
    try {
      let csv_data = await d3.csv('internet_data.csv');
      return csv_data;
    } catch (error) {
      console.error('Error loading CSV:', error);
    }
  }

  async function updateMap() {
    mapboxgl.accessToken = 'pk.eyJ1IjoiaGlsbGFyeWNoYW5nIiwiYSI6ImNsdzB0bTUwcTA0bDcyaXFjaDEzZGduMXUifQ.PwESlw8F78LN1gE7GjuyMg';
    
    map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/outdoors-v11',
    center: [-81.224518, 40.3601],
    zoom: 2,
    minZoom: 1,
    maxZoom: 15,
    });

  }

  onMount(async () => {
    try {
      csvData = await fetchData();
      updateMap();
    } catch (error) {
      console.error('Error loading CSV data:', error);
    }
  });
</script>

<main>
  <div id="map"></div>
</main>

<style>
  #map {
    width: 100%;
    height: 600px; /* Adjust height as needed */
  }
</style>