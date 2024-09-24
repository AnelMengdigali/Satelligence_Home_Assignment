<script>
	import { Map } from "@beyonk/svelte-mapbox";
	// import Instructions from './Instructions.svelte'
	
	let mapComponent;

	import eventsData from "./events.json";

	function onReady() {
		mapComponent.flyTo({ center: [117.85, 4.34] });
		console.log(mapComponent.getMap());

		eventsData.events.map((event) =>
			event.detections.map((detection) => {
				let id = (Math.random() + 1).toString(36).substring(7);
				mapComponent.getMap().addSource(id, {
					type: "geojson",
					data: {
						type: "Feature",
						geometry: {
							type: detection.geometry.type,
							coordinates: detection.geometry.coordinates,
						},
						properties: {
							date: detection.date,
							totalSize: detection.totalSize,
						},
					},
				});

				mapComponent.getMap().addLayer({
					id: id,
					type: "fill",
					source: id,
					layout: {},
					paint: {
						"fill-color": "#f08",
						"fill-opacity": 0.4,
					},
				});

				mapComponent.getMap().on("click", id, (e) => {
					const features = mapComponent
						.getMap()
						.queryRenderedFeatures(e.point, {
							layers: [id],
						});

					if (features.length > 0) {
						let feature = features[0]
						const date = feature.properties.date;
						const totalSize = feature.properties.totalSize;
						let popupContent = `<strong>Event Details</strong><br/>Time: ${date} <br/>Total Size: ${totalSize}<br/><br/>`;

						new mapboxgl.Popup({closeButton: false})
							.setLngLat(e.lngLat)
							.setHTML(popupContent)
							.addTo(mapComponent.getMap());
					}
				});
			}),
		);
	}
</script>

<!-- <Instructions/> -->

<Map
	accessToken="pk.eyJ1Ijoic2F0ZWxsaWdlbmNlLXN0YWdpbmciLCJhIjoiY2w2cWtxaGNtMGJlYjNkdGNsMXI4MnpiYSJ9.LEONl2580jXyWbjJE7iMaQ"
	style="mapbox://styles/mapbox/light-v11"
	zoom="6"
	version="v2.12.0"
  bind:this={mapComponent} 
  on:ready={onReady}
></Map>
