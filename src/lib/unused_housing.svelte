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
            style: "mapbox://styles/mapbox/light-v11",
            zoom: 8,
            center: [-71.09,42.36],
        });
        await new Promise(resolve => map.on("load", resolve));

        const routes = await fetch("mbta_lines.geojson");
        const routesData = await routes.json();

        const stations = await fetch("station_info.geojson");
        const stationsData = await stations.json();


        // COMMUTER RAIL LAYERS
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
                "fill-opacity": 0, 
            },
        });

        const commuterCentroids = {
            type: 'FeatureCollection',
            features: commuterRailStops.features.map(f => {
                const centroid = turf.centroid(f);
                // Add the original polygon's ID to the centroid for reference
                centroid.properties.stop_name= f.properties.stop_name; 
                centroid.properties.zoned=f.properties.zoned;
                centroid.properties.actual=f.properties.actual;
                centroid.properties.avg_value=f.properties.avg_value;
                return centroid;
            }),
        };

        map.addSource('commuter_centroids', {
            type: 'geojson',
            data: commuterCentroids,
        });

        map.addLayer({
            id: "commuter_dots",
            type: "circle",
            source: "commuter_centroids",
            paint: {
                "circle-radius": 3,
                "circle-color": "#7B388C",
            },
        });

        let clickedPopup = null;
        const popup = new mapboxgl.Popup({
            closeButton: false,
            closeOnClick: false
        });

        // Add hover event listener
        map.on('mousemove', 'commuter_dots', (e) => {
            // Get properties from the first feature under the mouse
            if (clickedPopup) return;
            const feature = e.features[0];

            // Set the popup content and location
            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)  // Adjust field name as needed
                .addTo(map);
        });

        map.on('mouseleave', 'commuter_dots', () => {
                popup.remove();
            });

        let selectedStation=null;
        map.on('click', 'commuter_dots', (e) => {
            if (clickedPopup) {
                clickedPopup.remove();
                clickedPopup = null;
            }
        
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('commuter_rail_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.4,  // Highlight selected buffer
                0     // Hide others
            ]);

            // Build your detailed HTML content
            const popupContent = `
                    <strong>${feature.properties.stop_name}</strong><br>
                    <em>Zoned: ${feature.properties.zoned}</em><br>
                    <em>Actual: ${feature.properties.actual}</em><br>
                    <em>Avg. Home Price: $${feature.properties.avg_value}</em><br>
                `;

            // Create and show the popup
            clickedPopup = new mapboxgl.Popup()
                .setLngLat(e.lngLat)
                .setHTML(popupContent)
                .addTo(map);

            clickedPopup.on('close', () => {
                clickedPopup = null;
            });

            const coordinates = feature.geometry.coordinates;

            // Zoom and center the map on the clicked centroid
            map.flyTo({
                center: coordinates,
                zoom: 13,  // Adjust zoom level as desired
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });
        });

        map.on('click', (e) => {
            // If clicking on an empty space (not a station)
            const features = map.queryRenderedFeatures(e.point, {
                layers: ['commuter_dots']
            });

            if (!features.length && clickedPopup) {
                clickedPopup.remove();
                clickedPopup = null;
            }
            if (!features.length) {
                selectedStation = null;
                map.setPaintProperty('commuter_rail_stations', 'fill-opacity', 0);
            }
        });

        //GREEN LINE LAYERS

        map.addLayer({
            id: "green_line_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#00843d", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["in", ["get", "route_id"], ['literal',["Green-D", 'Green-E', 'Green-C', "Green-B"]]]
        });

        map.addLayer({
            id: "red_line_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#da291c", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["==", ["get", "route_id"], "Red"]
        });

        map.addLayer({
            id: "mattapan_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#da291c", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["==", ["get", "route_id"], "Mattapan"]
        });

        map.addLayer({
            id: "blue_line_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#003da5", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["==", ["get", "route_id"], "Blue"]
        });

        map.addLayer({
            id: "orange_line_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#ed8b00", // purple for commuter rail
                "line-width": 1,
                'line-opacity': 1,
            },
            filter: ["==", ["get", "route_id"], "Orange"]
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
                "fill-opacity": 0.2, 
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
                "fill-opacity": 0.2, 
            },
        });
        
        
    }

    onMount(async() => {
        await Mount();
    })

    const layerGroups={
            'commuter':['commuter_rail_stations', 'commuter_rail_layer'],
            'green':['green_line_stations', 'green_line_layer'],
            'red':["red_line_stations", "red_line_layer", "mattapan_layer"],
            "blue": ["blue_line_layer"],
            'orange': ['orange_line_layer'],
        }

    const allLayers=['commuter_rail_stations', 'commuter_rail_layer', 'green_line_stations', "green_line_layer", "red_line_stations", "red_line_layer", "mattapan_layer", 'blue_line_layer', 'orange_line_layer']

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
    if (typeof window !== 'undefined') {
        window.showLayerGroup = showLayerGroup;
    }

    // window.showLayerGroup('commuter');
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