<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import * as d3 from "d3";
    import * as turf from '@turf/turf';
    // import routesData from "$lib/mbta_lines.geojson"

    mapboxgl.accessToken = "pk.eyJ1IjoicGF0cmlja3hsaXUiLCJhIjoiY205OTF3ZHlsMDl4eTJpb2hnazRsM2o3eiJ9.Gfabla7QnJ0-1W5qddqoRw";

    import { onMount } from "svelte";

    let map;

    async function Mount(){
        map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/mapbox/streets-v12",
            zoom: 8,
            center: [-71.09,42.36],
        });
        await new Promise(resolve => map.on("load", resolve));

        const routes = await fetch("mbta_lines.geojson");
        const routesData = await routes.json();
        // console.log(routesData);
        map.addSource('train_lines',{
            type:'geojson',
            data: routesData,
        });
        map.addLayer({
            id: "commuter_rail_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#7B388C", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["==", ["get", "network_id"], "commuter_rail"]
        });

        const stations = await fetch("housingdiff.geojson");
        const stationsData = await stations.json();

        const commuterRailStops = {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.network_id === "commuter_rail");
            })
        };

        
        map.addSource('commuter_rail_stops',{
            type:'geojson',
            data: commuterRailStops,
        });
        map.addLayer({
            id: "commuter_rail_stations",
            type: "fill",
            source: "commuter_rail_stops",
            paint: {
                "fill-color": "#7B388C",  // Purple fill color
                "fill-opacity": 0.5, 
            },
        });

        let clickedPopup = null;
        const popup = new mapboxgl.Popup({
            closeButton: false,
            closeOnClick: false
        });

        // Add hover event listener
        map.on('mousemove', 'commuter_rail_stations', (e) => {
            // Get properties from the first feature under the mouse
            if (clickedPopup) return;
            const feature = e.features[0];

            // Set the popup content and location
            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)  // Adjust field name as needed
                .addTo(map);
        });

        // Remove the popup when the mouse leaves the layer
        map.on('mouseleave', 'commuter_rail_stations', () => {
                popup.remove();
            });

        map.on('click', 'commuter_rail_stations', (e) => {
            if (clickedPopup) {
                clickedPopup.remove();
                clickedPopup = null;
            }
        
            const feature = e.features[0];

            // Build your detailed HTML content
            const popupContent = `
                    <strong>${feature.properties.stop_name}</strong><br>
                    <em>Zoned: ${feature.properties.zoned}</em><br>
                    <em>Actual: ${feature.properties.actual}</em><br>
                `;

            // Create and show the popup
            clickedPopup = new mapboxgl.Popup()
                .setLngLat(e.lngLat)
                .setHTML(popupContent)
                .addTo(map);

            clickedPopup.on('close', () => {
                clickedPopup = null;
            });
        });

        map.on('click', (e) => {
            // If clicking on an empty space (not a station)
            const features = map.queryRenderedFeatures(e.point, {
                layers: ['commuter_rail_stations']
            });

            if (!features.length && clickedPopup) {
                clickedPopup.remove();
                clickedPopup = null;
            }
        });

        const redLineStops= {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.route_id === "Red");
            })
        };

        map.addSource('red_line_stops',{
            type:'geojson',
            data: redLineStops,
        });
        map.addLayer({
            id: "red_line_stations",
            type: "fill",
            source: "red_line_stops",
            paint: {
                "fill-color": "#FA2D27",  // Purple fill color
                "fill-opacity": 0.5, 
            },
        });

        const greenLineStops= {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.route_id.includes("Green-"));
            })
        };

        map.addSource('green_line_stops',{
            type:'geojson',
            data: greenLineStops,
        });
        map.addLayer({
            id: "green_line_stations",
            type: "fill",
            source: "green_line_stops",
            paint: {
                "fill-color": "#008150",  // Purple fill color
                "fill-opacity": 0.5, 
            },
        });
        
        
    }

    onMount(async() => {
        await Mount();
    })

    const layerGroups={
            'commuter':['commuter_rail_stations', 'commuter_rail_layer'],
            'green':['green_line_stations'],
            'red':["red_line_stations"],
        }

    const allLayers=['commuter_rail_stations', 'commuter_rail_layer', 'green_line_stations', "red_line_stations"]

    function showLayerGroup(groupName) {
        const layersToShow = layerGroups[groupName];

        allLayers.forEach(layerId => {
            if (map.getLayer(layerId)) {
                const visibility = layersToShow.includes(layerId) ? 'visible' : 'none';
                map.setLayoutProperty(layerId, 'visibility', visibility);
                console.log(visibility);
            }
        });
    }
    window.showLayerGroup = showLayerGroup;
</script>
<div id="layer-controls">
    <button onclick="showLayerGroup('commuter')">Commuter Rail</button>
    <button onclick="showLayerGroup('green')">Green Line</button>
    <button onclick="showLayerGroup('red')">Red Line</button>
</div>
<div id="map">
</div>

<style>
    #map {
        width: 80%;
        height: 600px;
    }
</style>