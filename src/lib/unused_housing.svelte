<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import * as d3 from "d3";
    import * as turf from '@turf/turf';

    mapboxgl.accessToken = "pk.eyJ1IjoicGF0cmlja3hsaXUiLCJhIjoiY205OTF3ZHlsMDl4eTJpb2hnazRsM2o3eiJ9.Gfabla7QnJ0-1W5qddqoRw";

    import { onMount } from "svelte";

    let initialized=false;
    let map;
    let selectedStation=null;
    const barWidth = 400;
    const barHeight = 350;
    let housing=false;
    let housingData= [
                    { key: 'Zoned Units', value: 0 },
                    { key: 'Existing Units', value: 0 },
                ];;
    let todData =[
                    { key: 'Transit', value: 0 },
                    { key: 'Orientation', value: 0 },
                    { key: 'Development', value: 0 },
                    { key: 'Zone Potential', value: 0},
                ];
    let avg_val='N/A';
    let tod_score='N/A';
    
    function updateBarChart(barData, titleText) {
        const svg = d3.select('#bar-chart svg');
        const barHeight = 400;
        const barWidth = 400;
        const barMargin = { top: 20, right: 20, bottom: 20, left: 60 };


        const x = d3.scaleBand()
            .domain(barData.map(d => d.key))
            .range([barMargin.left, barWidth - barMargin.right])
            .padding(0.1);

        let yMax = housing && d3.max(barData, d => d.value)!==0
            ? d3.max(barData, d => d.value)
            : Math.max(25, d3.max(barData, d => d.value));

        let y = housing ? d3.scaleLinear()
            .domain([0, yMax]).nice()
            .range([barHeight - barMargin.bottom, barMargin.top]):
            d3.scaleLinear()
            .domain([0, yMax])
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

        if (housing){
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
        }

        else{
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
        }

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

        const stations = await fetch("final_station_info2.geojson");
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
                    { key: 'Transit', value: 0 },
                    { key: 'Orientation', value: 0 },
                    { key: 'Development', value: 0 },
                    { key: 'Zone Potential', value: 0},
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

        // COMMUTER LISTENERS
        map.on('mousemove', 'commuter_rail_stations', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)
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

            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 14,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            
            housingData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];
        
            todData = [
                { key: 'Transit', value: +feature.properties.etod_sub1t },
                { key: 'Orientation', value: +feature.properties.etod_sub2o },
                { key: 'Development', value: +feature.properties.etod_sub3d },
                { key: 'Zone Potential', value: +feature.properties.etod_sub4z },
            ];

            const avgPrice = feature.properties.avg_value;
            avg_val= avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';

            tod_score=(feature.properties.etod_sub1t+feature.properties.etod_sub2o+feature.properties.etod_sub3d+(feature.properties.etod_sub4z))
            
            if (housing){
                updateBarChart(housingData,selectedStation);
                document.getElementById('avg-price').textContent = `${avg_val}`;
            }
            else{
                updateBarChart(todData,selectedStation);
                document.getElementById('avg-price').textContent = `${tod_score}/80`;
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
                "fill-color": "#00843d",
                "fill-opacity": 0.0, 
            },
            layout:{
                visibility:'none',
            },
        });

        const greenCentroids = {
            type: 'FeatureCollection',
            features: greenLineStops.features.map(f => {
                const centroid = turf.centroid(f);
                centroid.properties.stop_name= f.properties.stop_name; 
                centroid.properties.zoned=f.properties.zoned;
                centroid.properties.actual=f.properties.actual;
                centroid.properties.avg_value=f.properties.avg_value;
                centroid.properties.etod_sub1t=f.properties.etod_sub1t;
                centroid.properties.etod_sub2o=f.properties.etod_sub2o;
                centroid.properties.etod_sub3d=f.properties.etod_sub3d;
                centroid.properties.etod_sub4z=f.properties.etod_sub4z;
                return centroid;
            }),
        };

        map.addSource('green_centroids', {
            type: 'geojson',
            data: greenCentroids,
        });

        map.addLayer({
            id: "green_dots",
            type: "circle",
            source: "green_centroids",
            paint: {
                "circle-radius": 4,
                "circle-color": "#00843d",
            },
            layout:{
                visibility:'none',
            },
        });

        // GREEN LISTENERS
        map.on('mousemove', 'green_dots', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)
                .addTo(map);

            map.setPaintProperty('green_dots', 'circle-radius', [
                'case',
                ['==', ['get', 'stop_name'], feature.properties.stop_name],
                8,  // radius when hovered
                4    // default radius
            ]);
        });

        map.on('mouseleave', 'green_dots', () => {
                popup.remove();
                map.setPaintProperty('green_dots', 'circle-radius', 4);
            });

        map.on('click', 'green_dots', (e) => {
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('green_line_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.5,  // Highlight selected buffer
                0     // Hide others
            ]);

            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 14,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            housingData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];
        
            todData = [
                { key: 'Transit', value: +feature.properties.etod_sub1t },
                { key: 'Orientation', value: +feature.properties.etod_sub2o },
                { key: 'Development', value: +feature.properties.etod_sub3d },
                { key: 'Zone Potential', value: +feature.properties.etod_sub4z },
            ];

            const avgPrice = feature.properties.avg_value;
            avg_val= avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';

            tod_score=(feature.properties.etod_sub1t+feature.properties.etod_sub2o+feature.properties.etod_sub3d+(feature.properties.etod_sub4z))        
            
            if (housing){
                updateBarChart(housingData,selectedStation);
                document.getElementById('avg-price').textContent = `${avg_val}`;
            }
            else{
                updateBarChart(todData,selectedStation);
                document.getElementById('avg-price').textContent = `${tod_score}/80`;
            }
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

        const redLineStops= {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.route_id === "Red" || route.route_id==='Mattapan');
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
                "fill-color": "#da291c",
                "fill-opacity": 0, 
            },
            layout:{
                visibility:'none',
            },
        });

        const redCentroids = {
            type: 'FeatureCollection',
            features: redLineStops.features.map(f => {
                const centroid = turf.centroid(f);
                centroid.properties.stop_name= f.properties.stop_name; 
                centroid.properties.zoned=f.properties.zoned;
                centroid.properties.actual=f.properties.actual;
                centroid.properties.avg_value=f.properties.avg_value;
                centroid.properties.etod_sub1t=f.properties.etod_sub1t;
                centroid.properties.etod_sub2o=f.properties.etod_sub2o;
                centroid.properties.etod_sub3d=f.properties.etod_sub3d;
                centroid.properties.etod_sub4z=f.properties.etod_sub4z;
                return centroid;
            }),
        };

        map.addSource('red_centroids', {
            type: 'geojson',
            data: redCentroids,
        });

        map.addLayer({
            id: "red_dots",
            type: "circle",
            source: "red_centroids",
            paint: {
                "circle-radius": 4,
                "circle-color": "#da291c",
            },
            layout:{
                visibility:'none',
            },
        });

        // RED LISTENERS
        map.on('mousemove', 'red_dots', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`) 
                .addTo(map);

            map.setPaintProperty('red_dots', 'circle-radius', [
                'case',
                ['==', ['get', 'stop_name'], feature.properties.stop_name],
                8,  // radius when hovered
                4    // default radius
            ]);
        });

        map.on('mouseleave', 'red_dots', () => {
                popup.remove();
                map.setPaintProperty('red_dots', 'circle-radius', 4);
            });

        map.on('click', 'red_dots', (e) => {
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('red_line_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.5,  // Highlight selected buffer
                0     // Hide others
            ]);

            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 14,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            housingData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];
        
            todData = [
                { key: 'Transit', value: +feature.properties.etod_sub1t },
                { key: 'Orientation', value: +feature.properties.etod_sub2o },
                { key: 'Development', value: +feature.properties.etod_sub3d },
                { key: 'Zone Potential', value: +feature.properties.etod_sub4z },
            ];
            
            const avgPrice = feature.properties.avg_value;
            avg_val= avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';

            tod_score=(feature.properties.etod_sub1t+feature.properties.etod_sub2o+feature.properties.etod_sub3d+(feature.properties.etod_sub4z))
            
            if (housing){
                updateBarChart(housingData,selectedStation);
                document.getElementById('avg-price').textContent = `${avg_val}`;
            }
            else{
                updateBarChart(todData,selectedStation);
                document.getElementById('avg-price').textContent = `${tod_score}/80`;
            }
        });
        
        //BLUE LINE LAYERS
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

        const blueLineStops= {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.route_id === "Blue");
            })
        };

        map.addSource('blue_line_stops',{
            type:'geojson',
            data: blueLineStops,
        });
        map.addLayer({
            id: "blue_line_stations",
            type: "fill",
            source: "blue_line_stops",
            paint: {
                "fill-color": "#003da5",
                "fill-opacity": 0, 
            },
            layout:{
                visibility:'none',
            },
        });

        const blueCentroids = {
            type: 'FeatureCollection',
            features: blueLineStops.features.map(f => {
                const centroid = turf.centroid(f);
                centroid.properties.stop_name= f.properties.stop_name; 
                centroid.properties.zoned=f.properties.zoned;
                centroid.properties.actual=f.properties.actual;
                centroid.properties.avg_value=f.properties.avg_value;
                centroid.properties.etod_sub1t=f.properties.etod_sub1t;
                centroid.properties.etod_sub2o=f.properties.etod_sub2o;
                centroid.properties.etod_sub3d=f.properties.etod_sub3d;
                centroid.properties.etod_sub4z=f.properties.etod_sub4z;
                return centroid;
            }),
        };

        map.addSource('blue_centroids', {
            type: 'geojson',
            data: blueCentroids,
        });

        map.addLayer({
            id: "blue_dots",
            type: "circle",
            source: "blue_centroids",
            paint: {
                "circle-radius": 4,
                "circle-color": "#003da5",
            },
            layout:{
                visibility:'none',
            },
        });

        // BLUE LISTENERS
        map.on('mousemove', 'blue_dots', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`) 
                .addTo(map);

            map.setPaintProperty('blue_dots', 'circle-radius', [
                'case',
                ['==', ['get', 'stop_name'], feature.properties.stop_name],
                8,  // radius when hovered
                4    // default radius
            ]);
        });

        map.on('mouseleave', 'blue_dots', () => {
                popup.remove();
                map.setPaintProperty('blue_dots', 'circle-radius', 4);
            });

        map.on('click', 'blue_dots', (e) => {
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('blue_line_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.5,  // Highlight selected buffer
                0     // Hide others
            ]);

            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 14,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            housingData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];
        
            todData = [
                { key: 'Transit', value: +feature.properties.etod_sub1t },
                { key: 'Orientation', value: +feature.properties.etod_sub2o },
                { key: 'Development', value: +feature.properties.etod_sub3d },
                { key: 'Zone Potential', value: +feature.properties.etod_sub4z },
            ];
            
            const avgPrice = feature.properties.avg_value;
            avg_val= avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';

            tod_score=(feature.properties.etod_sub1t+feature.properties.etod_sub2o+feature.properties.etod_sub3d+(feature.properties.etod_sub4z))
            
            if (housing){
                updateBarChart(housingData,selectedStation);
                document.getElementById('avg-price').textContent = `${avg_val}`;
            }
            else{
                updateBarChart(todData,selectedStation);
                document.getElementById('avg-price').textContent = `${tod_score}/80`;
            }
        });

        //ORANGE LINE LAYERS
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
        
        const orangeLineStops= {
            type: "FeatureCollection",
            features: stationsData.features.filter(feature => {
                return feature.properties.routes.some(route => route.route_id === "Orange");
            })
        };

        map.addSource('orange_line_stops',{
            type:'geojson',
            data: orangeLineStops,
        });
        map.addLayer({
            id: "orange_line_stations",
            type: "fill",
            source: "orange_line_stops",
            paint: {
                "fill-color": "#ed8b00",
                "fill-opacity": 0, 
            },
            layout:{
                visibility:'none',
            },
        });

        const orangeCentroids = {
            type: 'FeatureCollection',
            features: orangeLineStops.features.map(f => {
                const centroid = turf.centroid(f);
                centroid.properties.stop_name= f.properties.stop_name; 
                centroid.properties.zoned=f.properties.zoned;
                centroid.properties.actual=f.properties.actual;
                centroid.properties.avg_value=f.properties.avg_value;
                centroid.properties.etod_sub1t=f.properties.etod_sub1t;
                centroid.properties.etod_sub2o=f.properties.etod_sub2o;
                centroid.properties.etod_sub3d=f.properties.etod_sub3d;
                centroid.properties.etod_sub4z=f.properties.etod_sub4z;
                return centroid;
            }),
        };

        map.addSource('orange_centroids', {
            type: 'geojson',
            data: orangeCentroids,
        });

        map.addLayer({
            id: "orange_dots",
            type: "circle",
            source: "orange_centroids",
            paint: {
                "circle-radius": 4,
                "circle-color": "#ed8b00",
            },
            layout:{
                visibility:'none',
            },
        });

        // ORANGE LISTENERS
        map.on('mousemove', 'orange_dots', (e) => {
            const feature = e.features[0];

            popup.setLngLat(e.lngLat)
                .setHTML(`<strong>${feature.properties.stop_name}</strong>`)
                .addTo(map);

            map.setPaintProperty('orange_dots', 'circle-radius', [
                'case',
                ['==', ['get', 'stop_name'], feature.properties.stop_name],
                8,  // radius when hovered
                4    // default radius
            ]);
        });

        map.on('mouseleave', 'orange_dots', () => {
                popup.remove();
                map.setPaintProperty('orange_dots', 'circle-radius', 4);
            });

        map.on('click', 'orange_dots', (e) => {
            const feature = e.features[0];
            selectedStation = e.features[0].properties.stop_name;

            map.setPaintProperty('orange_line_stations', 'fill-opacity', [
                'case',
                ['==', ['get', 'stop_name'], selectedStation],
                0.5,  // Highlight selected buffer
                0     // Hide others
            ]);

            const center = turf.centroid(feature).geometry.coordinates;

            // Zoom and center the map on the clicked station
            map.flyTo({
                center: center,
                zoom: 14,
                speed: 1.2,
                curve: 1.42,
                easing: (t) => t
            });

            housingData = [
                { key: 'Zoned Units', value: +feature.properties.zoned },
                { key: 'Existing Units', value: +feature.properties.actual },
            ];
        
            todData = [
                { key: 'Transit', value: +feature.properties.etod_sub1t },
                { key: 'Orientation', value: +feature.properties.etod_sub2o },
                { key: 'Development', value: +feature.properties.etod_sub3d },
                { key: 'Zone Potential', value: +feature.properties.etod_sub4z },
            ];
            
            const avgPrice = feature.properties.avg_value;
            avg_val= avgPrice
                ? `$${Number(avgPrice).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
                : 'N/A';

            tod_score=(feature.properties.etod_sub1t+feature.properties.etod_sub2o+feature.properties.etod_sub3d+(feature.properties.etod_sub4z))
            
            if (housing){
                updateBarChart(housingData,selectedStation);
                document.getElementById('avg-price').textContent = `${avg_val}`;
            }
            else{
                updateBarChart(todData,selectedStation);
                document.getElementById('avg-price').textContent = `${tod_score}/80`;
            }
        });
    }

    onMount(async() => {
        await Mount();
        initialized=true;
    })

    const layerGroups={
            'commuter':['commuter_rail_stations', 'commuter_rail_layer', "commuter_dots"],
            'green':['green_line_stations', 'green_line_layer', 'green_dots'],
            'red':["red_line_stations", "red_line_layer", "mattapan_layer", 'red_dots'],
            "blue": ["blue_line_layer", 'blue_line_stations', 'blue_dots'],
            'orange': ['orange_line_layer', 'orange_line_stations', 'orange_dots'],
        }

    const allLayers=[
                    'commuter_rail_stations',
                    'commuter_rail_layer',
                    "commuter_dots",
                    'green_line_stations',
                    "green_line_layer",
                    'green_dots',
                    "red_line_stations",
                    "red_line_layer",
                    'red_dots',
                    "mattapan_layer",
                    'blue_line_layer',
                    'blue_line_stations',
                    'blue_dots',
                    'orange_line_layer',
                    'orange_line_stations',
                    'orange_dots',
                ]

    function showLayerGroup(groupName) {
        const layersToShow = layerGroups[groupName];

        allLayers.forEach(layerId => {
            if (map.getLayer(layerId)) {
                const visibility = layersToShow.includes(layerId) ? 'visible' : 'none';
                map.setLayoutProperty(layerId, 'visibility', visibility);
            }
        });
    }
    if (typeof window !== 'undefined') {
        window.showLayerGroup = showLayerGroup;
    }

    $: if (initialized){
            if (housing) {
                if (selectedStation===null){
                    updateBarChart([
                        { key: 'Zoned Units', value: 0 },
                        { key: 'Existing Units', value: 0 },
                    ], 'Select a station');
                }
                else{
                    updateBarChart(housingData, selectedStation)
                }
                document.getElementById('avg-price').textContent = `${avg_val}`;
            } else {
                if (selectedStation===null){
                    updateBarChart([
                        { key: 'Transit', value: 0 },
                        { key: 'Orientation', value: 0 },
                        { key: 'Development', value: 0 },
                        { key: 'Zone Potential', value: 0}
                    ], 'Select a station');
                }
                else{
                    updateBarChart(todData, selectedStation)
                }
                document.getElementById('avg-price').textContent = tod_score!=='N/A' ? `${tod_score} / 100`: 'N/A';
            }
        }
    
</script>

<div id="container">
    <div id="map"></div>
    <div id="side-panel">
        <div id="layer-controls">
            <button class='commuter-button' onclick="showLayerGroup('commuter')">Commuter Rail</button>
            <button class='green-button' onclick="showLayerGroup('green')">Green Line</button>
            <button class='red-button' onclick="showLayerGroup('red')">Red Line</button>
            <button class='blue-button' onclick="showLayerGroup('blue')">Blue Line</button>
            <button class='orange-button' onclick="showLayerGroup('orange')">Orange Line</button>
        </div>
        <div id="bar-chart">
            <div class="bar-title-container">
                <h3 id="bar-title" class="bar-title">Select a station</h3>
                <div class="switch-labeled">
                    <span class="switch-label">TOD</span>
                    <label class="switch">
                        <input type="checkbox" bind:checked={housing}>
                        <span class="slider"></span>
                    </label>
                    <span class="switch-label">Housing</span>
                </div>
            </div>
        </div>
        <div class='price-heading'>{housing ? 'Average Home Price:' : 'Total TOD Score:'}</div>
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

    .bar-title-container {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 1rem;
    }
    .bar-title{
        font-size: 1.5em;
    }

    .switch-labeled {
        display: flex;
        align-items: center;
        gap: 0.4rem;
    }

    .switch-label {
        color: white;
        font-size: 0.75rem;
    }

    .switch {
		position: relative;
		display: inline-block;
		width: 40px;
		height: 24px;
	}
	.switch input {
		opacity: 0;
		width: 0;
		height: 0;
	}
	.slider {
		position: absolute;
		cursor: pointer;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: #ccc;
		transition: 0.4s;
		border-radius: 24px;
	}
	.slider:before {
		position: absolute;
		content: "";
		height: 16px;
		width: 16px;
		left: 4px;
		bottom: 4px;
		background-color: #7c878e;
		transition: 0.4s;
		border-radius: 50%;
	}
	input:checked + .slider:before {
		transform: translateX(16px);
	}

    #bar-chart {
        flex: 1;
    }

    .price-heading{
        padding-top: 1em;
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
        color: white;
        border: none;
        padding: 0.6rem 1rem;
        border-radius: 6px;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s, transform 0.1s;
    }

    #layer-controls button:hover {
        transform: translateY(-1px);
    }

    #layer-controls button:active {
        background-color: #1abc9c;
        transform: translateY(1px);
    }

    .commuter-button{
        background-color: #7B388C;
    }

    .commuter-button:hover{
        background-color: #4a2254;
    }

    .green-button{
        background-color: #00843d;
    }

    .green-button:hover{
        background-color: #004f25;
    }

    .red-button{
        background-color: #da291c;
    }

    .red-button:hover{
        background-color: #831911;
    }

    .blue-button{
        background-color: #003da5;
    }

    .blue-button:hover{
        background-color: #002563;
    }

    .orange-button{
        background-color: #ed8b00;
    }

    .orange-button:hover{
        background-color: #8e5300;
    }
</style>