<script>
    import * as d3 from "d3";
    import {onMount} from "svelte";
    import bars from "$lib/Bars.svelte";
    import Bars from "../lib/Bars.svelte";
    import UnusedHousing from "$lib/unused_housing.svelte";
    import Timeline from "$lib/timeline.svelte";

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

<img src ="/images/Title.png">
<h1 class="question">HOW IS ZONING CURRENTLY LIMITING THE SUPPLY OF TRANSIT-ACCESSIBLE HOUSING?</h1>
<h1>Introduction</h1>
<p>The housing crisis in Massachusetts is at an all-time high, and as demand for housing continues to grow, 
    the government is taking steps to spur more construction.
    This project focuses on one such step: the MBTA Communities act. The Communities act focuses on increasing density in the areas within .5 miles of 
    commuter rail, subway, ferry, and bus stations. We wanted to get a better understanding of what those areas are like. The following visualizations 
    will give insight into the current makeup of those areas.
 </p>

<h1>Boston Zoning History</h1>
<Timeline>timeline</Timeline>

<h1>TOD Scores of MBTA Communities</h1>
    <p>
        Information Station, a joint project between the Metropolitan Area Planning Council and Northeastern's Dukakis Center for Urban and Regional Policy, lends good insight into how legislators, policymakers, and city planners are looking at the housing problem and how the MBTA Communities Act might play a role. This project scores each MBTA community with a metric called the eTOD Score, a composite score summing three category scores: 
    </p>
    <ul>
        <li>Transit: how accessible, convenient, and used public transit in the area is</li>
        <li>Orientation: how favorable and likely to benefit the demographics in the area are to TOD</li>
        <li>Development: how primed the land development is, commercially, residentially, employability</li>
    </ul>
    <p>
        Each category score is the sum of several subscores, each of which represent the quintile ranking of an MBTA community against all MBTA communities.
    </p>
<h2>Transit</h2>
    <iframe title="Hello" src="transit.html" width="100%" height="600px" />
    <p>
        The Transit score is the sum of three factors: 
    </p>
    <ul>
        <li>Land area accessibility: acreage accessible via transit within 30 minutes including transfers, scaled by frequency of transit service</li>
        <li>Frequency and utilization of transit: number of transit trips scheduled per week at a station, weighted by transit commute share and vehicles per household</li>
        <li>Percent of commuters using a mode other than driving alone: high ranking station areas get quality and frequent transit service</li>
    </ul>
<h2>Orientation</h2>
    <iframe title="Hello" src="orientation.html" width="100%" height="600px" />
    <p>
        The Orientation score is the sum of four factors: 
    </p>
    <ul>
        <li>Percent of households with 0 vehicles available: high ranking station areas are likely to have higher transit ridership and lower VMT per household</li>
        <li>Percent of households earning under 25k per year: high ranking station areas are likely to depend on transit for mobility</li>
        <li>Renters as a share of all hosueholds: high ranking stations are more likely to use public transport than homeowners and less likely to own cars, more likely to use transit</li>
        <li>Estimated household transportation costs as a percent of regional median income: high ranking stations have viable alternatives to costly driving</li>
    </ul>
<h2>Development</h2>
    <iframe title="Hello" src="development.html" width="100%" height="600px" />
    <p>
        The Development score is the sum of three factors: 
    </p>
    <ul>
        <li>Walk score: amenities within 5 min walk=0.25 miles of station, then decaying with increasing distance, distances greater than 30 min walk are not considered</li>
        <li>Residential density in station area: high ranking stations indicate more households are within walking distance</li>
        <li>Employment proximity to station area: high ranking stations indicate jobs close/accessible via transit</li>
    </ul>

<h2>Takeaways</h2>
<p>
    Recommendations to policymakers are focused on expanding areas which already fall into a "transit-oriented development" category. A key feature of the housing crisis is the lack of, not necessarily the quality of secured housing. We should be focused on areas which score low on transit and development, and high on orientation.
</p>
<h1>Unbuilt Housing Units in MBTA Buffer Zones</h1>
<UnusedHousing/>
<style>
    h1{
        font-family: helvetica;
    }
    h2{
        font-family: helvetica;
    }
</style>