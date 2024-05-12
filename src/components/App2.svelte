<script>
  import { onMount, onDestroy } from 'svelte';
  import { Map } from 'mapbox-gl';
  import '../node_modules/mapbox-gl/dist/mapbox-gl.css';

  let map;
  let mapContainer;
  let lng, lat, zoom;

  lng = -71.224518;
  lat = 42.213995;
  zoom = 9;

  function updateData() {
    zoom = map.getZoom();
    lng = map.getCenter().lng;
    lat = map.getCenter().lat;
  }

  onMount(() => {
    const initialState = { lng: lng, lat: lat, zoom: zoom };
    mapboxgl.accessToken = 'pk.eyJ1IjoiaGlsbGFyeWNoYW5nIiwiYSI6ImNsdzB0bTUwcTA0bDcyaXFjaDEzZGduMXUifQ.PwESlw8F78LN1gE7GjuyMg';

    map = new mapboxgl.Map({
      container: mapContainer,
      style: `mapbox://styles/mapbox/outdoors-v11`,
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom
    });

    map.on('move', () => {
      updateData();
    });
  });

  onDestroy(() => {
    map.remove();
  });
</script>

<div>
  <div class="sidebar">
    Longitude: {lng.toFixed(4)} | Latitude: {lat.toFixed(4)} | Zoom:
    {zoom.toFixed( 2 )}
  </div>
  <div class="map-wrap">
    <div class="map" bind:this="{mapContainer}" />
  </div>
</div>

<style>
  .map {
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .sidebar {
    background-color: rgb(35 55 75 / 90%);
    color: #fff;
    padding: 6px 12px;
    font-family: monospace;
    z-index: 1;
    position: absolute;
    top: 0;
    left: 0;
    margin: 12px;
    border-radius: 4px;
  }
</style>
