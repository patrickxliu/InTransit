<script>
    import * as d3 from "d3";
    import {onMount} from "svelte";
    import bars from "$lib/Bars.svelte";
    import UnusedHousing from "$lib/unused_housing.svelte";
    import Timeline from "$lib/timeline2.svelte";
    export const ssr = false;

    
    // import '../../static/style.css'

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


<img src ="images/header.svg" alt="heading" width="100%">
<!-- <h1 class="question">HOW IS ZONING CURRENTLY LIMITING THE SUPPLY OF TRANSIT-ACCESSIBLE HOUSING?</h1> -->
<h1>Introduction</h1>
<p>The housing crisis in Massachusetts is at an all-time high, and as demand for housing continues to grow, 
    the government is taking steps to spur more construction.
    This project focuses on one such step: the MBTA Communities act. The Communities act focuses on increasing density in the areas within .5 miles of 
    commuter rail, subway, ferry, and bus stations, called the buffer zone. We wanted to get a better understanding of what those areas are like. The following visualizations 
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
<div class='score-grid'>
    <div class='score-cat'>
        <h2>Transit</h2>
        <!-- <iframe title="Hello" src="transit.html" width="100%" height="600px" /> -->
        <p>
            The Transit score is the sum of three factors: 
        </p>
        <ul>
            <li>Land area accessibility: acreage accessible via transit within 30 minutes including transfers, scaled by frequency of transit service</li>
            <li>Frequency and utilization of transit: number of transit trips scheduled per week at a station, weighted by transit commute share and vehicles per household</li>
            <li>Percent of commuters using a mode other than driving alone: high ranking station areas get quality and frequent transit service</li>
        </ul>
    </div>
    <div class='score-cat'>
        <h2>Orientation</h2>
        <!-- <iframe title="Hello" src="orientation.html" width="100%" height="600px" /> -->
        <p>
            The Orientation score is the sum of four factors: 
        </p>
        <ul>
            <li>Percent of households with 0 vehicles available: high ranking station areas are likely to have higher transit ridership and lower VMT per household</li>
            <li>Percent of households earning under 25k per year: high ranking station areas are likely to depend on transit for mobility</li>
            <li>Renters as a share of all households: high ranking stations are more likely to use public transport than homeowners and less likely to own cars, more likely to use transit</li>
            <li>Estimated household transportation costs as a percent of regional median income: high ranking stations have viable alternatives to costly driving</li>
        </ul>
    </div>
    <div class='score-cat'>
        <h2>Development</h2>
        <!-- <iframe title="Hello" src="development.html" width="100%" height="600px" /> -->
        <p>
            The Development score is the sum of three factors: 
        </p>
        <ul>
            <li>Walk score: amenities within 5 min walk (0.25 mi) of station, then decaying with increasing distance, distances greater than 30 min walk are not considered</li>
            <li>Residential density in station area: high ranking stations indicate more households are within walking distance</li>
            <li>Employment proximity to station area: high ranking stations indicate jobs close/accessible via transit</li>
        </ul>
    </div>
    <div class='score-cat'>
        <h2>Zone Potential</h2>
        <p>
            While TOD metrics do a great job of encapsulating the current infrastructure and demographics around MBTA stations, it does not 
            show much about how zoning policies are enabling or restricting the further devlopment in those areas.
        </p>
        <p>
            Thus, we introduce our own metric, Zone Potential,
            which is the quintile ranking of housing units allowed by current zoning policies. A higher score indicates that current zoning policies enable the 
            addiition of more housing units in the future. This score is scaled to have equal weight as each of the other scores.
        </p>
    </div>
</div>

<h2>Summary</h2>
    <p>
        Recommendations to policymakers are focused on expanding areas which already fall into a "transit-oriented development" category. A key feature of the housing crisis is the lack of, not necessarily the quality of secured housing. We should be focused on areas which score low on transit and development, and high on orientation.
    </p>
    <p>
        The intractive visualization below allows you to explore TOD and housing statistics in the buffer zone around MBTA commuter rail and rapid-transit stations.
         TOD scores have been normalized to be out of 100 with the addition of our Zone Potential metric. Toggling to "housing" mode allows you to explore how different
         municipalties are or are not taking advantage of the zoning laws around MBTA stations, and how the average home price might play a role.
    </p>
<h1>MBTA TOD & Housing by the Numbers</h1>
<UnusedHousing/>

<h1>Conclusions and Future Directions</h1>
    <p>
        Including our Zone Potential subscore, top scoring MBTA communities include Massachusetts Avenue on the Orange Line,
        Northeastern University and Symphony on the Green Line,
        South Station on the Red Line, and other downtown areas whose buffer zones are already highly developed with non-residential units. To maximize the usability of this subscore, it would be necessary to factor in <i>developable</i> zoned units.

        The overall trend is that zoning utilization is quite low; just because housing units <i>could</i> be built doesn't mean anyone has incentive to actually do it. But the primary opposition appears to come from residents via arguments against school overcrowding and lowering home values. If the level of development cannot go beyond the level of the communities closest to Boston, and maybe it would fundamentally change the culture of Boston, it seems that the best route for the Massachusetts legislature to support <i>all</i> residents and would-be residents is through public transit. For example, if the current Greater Boston Metropolitan Area is not amenable to increased development, the most viable option for prospective residents would be to live further from Boston, or their work place, and commute in and as the radius expands, the barrier becomes the purchasing price of a vehicle.
    </p>

<style>
    @import '../../static/style.css';
    :root {
        --blue: #2F5DA6;
        --green: #008150;
        --orange: #FD8A03;
        --red: #FA2D27;
        --silver: #9A9C9D;
        --purple: #7B388C;
    }
/* 
    body {
        background-color: rgb(255, 255, 255);
        color: white;
        margin: 3%;
    } */

    h1 {
        font-family: helvetica;
        text-align: center;
        background-color: var(--green);
        border-radius: 4px;
    }

    h2 {
        font-family: helvetica;
        text-align: center;
        background-color: var(--blue);
        border-radius: 4px;
    }

    .score-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 1.5rem;
    }

    .score-cat {
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
</style>