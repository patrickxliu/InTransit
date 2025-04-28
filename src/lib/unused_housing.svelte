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

        const routes = await fetch("/mbta_lines.geojson");
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
                "line-color": "#800080", // purple for commuter rail
                "line-width": 3,
                'line-opacity': .3,
            },
            filter: ["==", ["get", "network_id"], "commuter_rail"]
        });

        const stations = await fetch("/mbta_stops_with_buffer_collapsed.geojson");
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
                "fill-color": "#800080",  // Purple fill color
                "fill-opacity": 0.3, 
            },
        });

        // const parcels = await fetch("/parcel_zoning_compliance_municipal_Boston.geojson");
        // const parcelsData = await parcels.json();

        // const parcelsWithinBuffer = parcelsData.features.filter(parcel => {
        //     return commuterRailStopsWithBuffer.features.some(stop => {
        //         const stopBuffer = turf.polygon(stop.geometry.coordinates);
        //         const parcelGeometry = turf.polygon(parcel.geometry.coordinates);
        //         // Check if the parcel intersects with any of the buffers
        //         return turf.booleanIntersects(parcelGeometry, stopBuffer);
        //     });
        // });

        // const parcelsWithinBufferFeatureCollection = {
        //     type: "FeatureCollection",
        //     features: parcelsWithinBuffer,
        // };

        // // Add the filtered parcels within buffer zones to the map
        // map.addSource('parcels_within_buffer', {
        //     type: 'geojson',
        //     data: parcelsWithinBufferFeatureCollection,
        // });

        // // Add the parcels layer to the map
        // map.addLayer({
        //     id: "parcels_within_buffer_layer",
        //     type: "fill",  // You can use "fill" to highlight the areas of the parcels
        //     source: "parcels_within_buffer",
        //     paint: {
        //         "fill-color": "#00FF00",  // Green color for parcels within the buffer zones
        //         "fill-opacity": 0.5,
        //     },
        // });
    }

    onMount(async() => {
        await Mount();
    })
</script>

<div id="map">
</div>

<style>
    #map {
        width: 80%;
        height: 600px;
    }
</style>