<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import * as d3 from "d3";
    import * as turf from '@turf/turf';
    // import routesData from "$lib/mbta_lines.geojson"

    mapboxgl.accessToken = "pk.eyJ1IjoicGF0cmlja3hsaXUiLCJhIjoiY205OTF3ZHlsMDl4eTJpb2hnazRsM2o3eiJ9.Gfabla7QnJ0-1W5qddqoRw";

    import { onMount } from "svelte";

    let map;
    let selectedStation=null;
    const barWidth = 400;
    const barHeight = 350;
    
    function updateBarChart(barData, titleText) {
        const svg = d3.select('#bar-chart svg');
        const barHeight = 400;
        const barWidth = 400;
        const barMargin = { top: 20, right: 20, bottom: 20, left: 40 };

        const x = d3.scaleBand()
            .domain(barData.map(d => d.key))
            .range([barMargin.left, barWidth - barMargin.right])
            .padding(0.1);

        const y = d3.scaleLinear()
            .domain([0, d3.max(barData, d => d.value)]).nice()
            .range([barHeight - barMargin.bottom, barMargin.top]);

        // Update title
        document.getElementById('bar-title').textContent = titleText;

        // Draw or update bars
        const bars = svg.select('.bars')
            .selectAll('rect')
            .data(barData, d => d.key);

        bars.join(
            enter => enter.append('rect')
                .attr('x', d => x(d.key))
                .attr('width', x.bandwidth())
                .attr('y', y(0))
                .attr('height', 0)
                .attr('fill', '#FFC72C')
                .transition()
                .duration(500)
                .attr('y', d => y(d.value))
                .attr('height', d => y(0) - y(d.value)),

            update => update.transition()
                .duration(500)
                .attr('x', d => x(d.key))
                .attr('width', x.bandwidth())
                .attr('y', d => y(d.value))
                .attr('height', d => y(0) - y(d.value)),

            exit => exit.transition()
                .duration(300)
                .attr('height', 0)
                .attr('y', y(0))
                .remove()
        );

        const labels = svg.select('.labels')
            .selectAll('text')
            .data(barData, d => d.key);

        labels.join(
            enter => enter.append('text')
                .attr('class', 'bar-label')
                .attr('fill', 'white') 
                .attr('x', d => x(d.key) + x.bandwidth() / 2)
                .attr('y', y(0))
                .attr('text-anchor', 'middle')
                .text(d => d.value === 0 ? 'No data' : d.value)
                .transition()
                .duration(500)
                .attr('y', d => y(d.value) - 5),

            update => update.transition()
                .duration(500)
                .attr('x', d => x(d.key) + x.bandwidth() / 2)
                .attr('y', d => y(d.value) - 5)
                .text(d => d.value === 0 ? 'No data' : d.value),

            exit => exit.transition()
                .duration(300)
                .attr('y', y(0))
                .remove()
        );

        svg.select('.x-axis')
            .attr('transform', `translate(0,${barHeight - barMargin.bottom})`)
            .transition()
            .duration(500)
            .call(d3.axisBottom(x));

        svg.select('.y-axis')
            .attr('transform', `translate(${barMargin.left},0)`)
            .transition()
            .duration(500)
            .call(d3.axisLeft(y));
    }
    async function Mount(){
        map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/mapbox/light-v11",
            zoom: 10,
            center: [-71.09,42.36],
        });
        await new Promise(resolve => map.on("load", resolve));

        const routes = await fetch("mbta_lines.geojson");
        const routesData = await routes.json();

        const stations = await fetch("station_info.geojson");
        const stationsData = await stations.json();

        // BAR GRAPH
        let svg = d3.select('#bar-chart svg');
        if (svg.empty()) {
            svg = d3.select('#bar-chart')
                .append('svg')
                .attr('width', 400)
                .attr('height', 400);

            svg.append('g').attr('class', 'bars');
            svg.append('g').attr('class', 'labels');
            svg.append('g').attr('class', 'x-axis');
            svg.append('g').attr('class', 'y-axis');
        }
        const initialBarData = [
            { key: 'Zoned Units', value: 0 },
            { key: 'Existing Units', value: 0 }
        ];

        updateBarChart(initialBarData, 'Select a station');

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
                "line-color": "#7B388C",
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
                "fill-color": "#7B388C",
                "fill-opacity": 0, 
            },
        });

        const commuterCentroids = {
            type: 'FeatureCollection',
            features: commuterRailStops.features.map(f => {
                const centroid = turf.centroid(f);
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
                "circle-radius": 4,
                "circle-color": "#7B388C",
            },
        });

        const popup = new mapboxgl.Popup({
            closeButton: false,
            closeOnClick: false
        });

        // hover event listener
        map.on('mousemove', 'commuter_rail_stations', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)  // Adjust field name as needed
                .addTo(map);

            map.setPaintProperty('commuter_dots', 'circle-radius', [
                'case',
                ['==', ['get', 'stop_name'], feature.properties.stop_name],
                8,  // radius when hovered
                4    // default radius
            ]);
        });

        map.on('mouseleave', 'commuter_rail_stations', () => {
                popup.remove();
                map.setPaintProperty('commuter_dots', 'circle-radius', 4);
            });

        map.on('click', 'commuter_rail_stations', (e) => {
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('commuter_rail_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.5,  // Highlight selected buffer
                0     // Hide others
            ]);

            const coordinates = feature.geometry.coordinates;
            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 13,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            const barData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];

            updateBarChart(barData,selectedStation);

            const avgPrice = feature.properties.avg_value;
            const formattedPrice = avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';
            document.getElementById('avg-price').textContent = `${formattedPrice}`;
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
                "line-color": "#00843d",
                "line-width": 1,
                'line-opacity': 1,
            },
            layout:{
                visibility:'none',
            },
            filter: ["in", ["get", "route_id"], ['literal',["Green-D", 'Green-E', 'Green-C', "Green-B"]]]
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
                "fill-color": "#008150",
                "fill-opacity": 0.2, 
            },
            layout:{
                visibility:'none',
            },
        });

        // RED LINE LAYERS

        map.addLayer({
            id: "red_line_layer",
            type: "line",
            source: "train_lines",
            layout: {
                "line-join": "round",
                "line-cap": "round",
            },
            paint: {
                "line-color": "#da291c",
                "line-width": 1,
                'line-opacity': 1,
            },
            layout:{
                visibility:'none',
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
                "line-color": "#da291c",
                "line-width": 1,
                'line-opacity': 1,
            },
            layout:{
                visibility:'none',
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
                "line-color": "#003da5",
                "line-width": 1,
                'line-opacity': 1,
            },
            layout:{
                visibility:'none',
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
                "line-color": "#ed8b00", 
                "line-width": 1,
                'line-opacity': 1,
            },
            layout:{
                visibility:'none',
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
                "fill-color": "#FA2D27",
                "fill-opacity": 0.2, 
            },
            layout:{
                visibility:'none',
            },
        });
        
        
    }

    onMount(async() => {
        await Mount();
    })

    const layerGroups={
            'commuter':['commuter_rail_stations', 'commuter_rail_layer', "commuter_dots"],
            'green':['green_line_stations', 'green_line_layer'],
            'red':["red_line_stations", "red_line_layer", "mattapan_layer"],
            "blue": ["blue_line_layer"],
            'orange': ['orange_line_layer'],
        }

    const allLayers=['commuter_rail_stations', 'commuter_rail_layer', "commuter_dots", 'green_line_stations', "green_line_layer", "red_line_stations", "red_line_layer", "mattapan_layer", 'blue_line_layer', 'orange_line_layer']

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

</script>

<div id="container">
    <div id="map"></div>
    <div id="side-panel">
        <div id="layer-controls">
            <button onclick="showLayerGroup('commuter')">Commuter Rail</button>
            <button onclick="showLayerGroup('green')">Green Line</button>
            <button onclick="showLayerGroup('red')">Red Line</button>
            <button onclick="showLayerGroup('blue')">Blue Line</button>
            <button onclick="showLayerGroup('orange')">Orange Line</button>
        </div>
        <div id="bar-chart">
            <h3 id="bar-title" class='bar-title'>Select a station</h3>
        </div>
        <div class='price-heading'>Average Home Price:</div>
        <div id="avg-price" class="price-display">N/A</div>
    </div>
</div>


<style>
    :global(body) {
        font-family: Helvetica, Arial, sans-serif;
    }
    #container {
        display: flex;
        width: 100%;
    }

    #map {
        width: 66%;
        height: 700px;
        color: black;
    }

    #side-panel {
        width: 34%;
        height: 700px;
        display: flex;
        flex-direction: column;
        color: white;
        padding: 1rem;
        box-sizing: border-box;
    }
    .bar-title{
        font-size: 2em;
    }

    #bar-chart {
        flex: 1;
    }

    .price-heading{
        margin-top: 0rem;
        color: white;
        font-size: 1rem;
        text-align: center;
    }
    .price-display {
        margin-top: .5rem;
        color: white;
        font-size: 2rem;
        text-align: center;
    }

    #layer-controls {
        display: flex;
        flex-wrap: wrap;
        gap: 0.5rem;
        margin-bottom: 1rem;
        justify-content: center;
    }

    #layer-controls button {
        background-color: #2c3e50;
        color: white;
        border: none;
        padding: 0.6rem 1rem;
        border-radius: 6px;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s, transform 0.1s;
    }

    #layer-controls button:hover {
        background-color: #34495e;
        transform: translateY(-1px);
    }

    #layer-controls button:active {
        background-color: #1abc9c;
        transform: translateY(1px);
    }
</style>