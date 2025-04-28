<script>
    import * as d3 from "d3";
    import {onMount} from "svelte";
    import bars from "$lib/Bars.svelte";
    import Bars from "../lib/Bars.svelte";

    let data = [];
    let width = 400;

    onMount(async () =>
        data = await d3.csv("/density.csv", row => ({
            ...row,
            // station: row.Station,
            // order: row.order,
            // lines: row.lines,
            // munis:row.Munis,
            // area: Number(row.Area),
            // units: row.Units,
            // density: row.Density
        })))
</script>

<img src ="../../static/images/Title.png">
<h1 class="question">HOW IS ZONING CURRENTLY LIMITING THE SUPPLY OF TRANSIT-ACCESSIBLE HOUSING?</h1>
<h1>Introduction</h1>
<p>The housing crisis in Massachusetts is at an all-time high, and as demand for housing continues to grow, 
    the government is taking steps to spur more construction.
    This project focuses on one such step: the MBTA Communities act. The Communities act focuses on increasing density in the areas within .5 miles of 
    commuter rail, subway, ferry, and bus stations. We wanted to get a better understanding of what those areas are like. The following visualizations 
    will give insight into the current makeup of those areas.
 </p>
<h1>TOD Scores of MBTA Communities</h1>
<h2>Transit</h2>
<iframe title="Hello" src="transit.html" width="100%" height="600px" />
<h2>Orientation</h2>
<iframe title="Hello" src="orientation.html" width="100%" height="600px" />
<h2>Development</h2>
<iframe title="Hello" src="development.html" width="100%" height="600px" />

<h1>Density</h1>

<style>
    h1{
        font-family: helvetica;
    }
    h2{
        font-family: helvetica;
    }
</style>