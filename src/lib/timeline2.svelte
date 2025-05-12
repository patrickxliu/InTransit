<script>
    import { onMount } from "svelte";
  
    let scrollProgress = 0;
    const trainMaxMove = 1200;
  
    const events = [
      { year: 1920, title: "Massachusetts Zoning Enabling Act", description: "This state act gave localities the power to use and create zoning ordinances." },
      { year: 1956, title: "Boston Zoning Charter Adopted", description: "This was Boston's first city-wide zoning code." },
      { year: 1957, title: "Boston Redevelopment Authority Founded", description: "Formed with the task of overseeing zoning and future planning through redevelopment, this group led some controversial clearance initiatives such as Boston’s West End." },
      { year: 1958, title: "Boston’s West End cleared", description: "With the goal of urban renewal, the city of Boston seized and demolished this neighborhood, leading to the displacement of its diverse, working-class community. Enabled by the federal Housing Act of 1949, this redevelopment impacted the zoning of the area, one of many examples of how rapid redevelopment of the city often shaped zoning - at the cost of some of its communities." },
      { year: 1960, title: "City Planning Board merges with BRA", description: "This strengthened and centralized power, highlighting Boston’s planning-oriented zoning approach." },
      { year: 1966, title: "Home Rule Amendment", description: "This state amendment increased city self-government, giving Boston more control over its zoning code." },
      { year: 1968, title: "Fair Housing Act", description: "This federal act aimed to promote equal rights and access to housing. However, zoning ordinances and local planning worked against this, making fair housing far from the reality." },
      { year: 1969, title: "Anti-Snob Zoning Law (Chapter 40B)", description: "Aimed at pushing back on exclusionary zoning, this state law allowed developers to bypass local zoning when building affordable housing." },
      { year: 1975, title: "Reforming State Zoning Act (Chapter 40A)", description: "Standardized the state-wide zoning processes for all localities except Boston." },
      { year: 1993, title: "Economic Development and Industrial Corporation of Boston and the BRA Merged", description: "While still legally acting as separate entities, in functionally working together and sharing resources, this centralized power created a singular city wide development agency." },
      { year: 2021, title: "Fair Housing Zoning Amendment", description: "An attempt to reverse the exclusionary zoning of the past, this made Boston the first major city in the US to include fair housing requirements directly in the zoning code." },
      { year: 2021, title: "MBTA Communities Act", description: "State-wide housing policy aimed at further housing development in transit-accessible areas." },
      { year: 2024, title: "Mayor Wu establishes Planning Dept.", description: "This brought planning under municipal control, allowing for more control over how planning initiatives impact and work alongside other citywide initiatives/goals." }
    ];
  
    const startYear = 1920;
    const endYear = 2025;
    const yearRange = endYear - startYear;
  
    const periods = [
      { label: "Start of zoning", start: 1920, end: 1930, color: "#2F5DA6" },
      { label: "Rapid Population Growth", start: 1945, end: 1968, color: "#FD8A03" },
      { label: "Exclusionary Zoning/Big Downzone", start: 1968, end: 1975, color: "#FA2D27" },
      { label: "Zoning Reform", start: 1975, end: 2025, color: "#008150" }
    ];
  
    function updateScrollProgress() {
      const container = document.getElementById("timeline-wrapper");
      const bounds = container.getBoundingClientRect();
      const scrollY = -bounds.top;
      const scrollRange = container.offsetHeight - window.innerHeight;
      scrollProgress = Math.min(Math.max(scrollY / scrollRange, 0), 1);
    }
  
    onMount(() => {
      window.addEventListener("scroll", updateScrollProgress);
      updateScrollProgress();
      return () => window.removeEventListener("scroll", updateScrollProgress);
    });
  
    // Compute position of each event based on its year
    $: eventPositions = events.map(e => ({
      ...e,
      position: ((e.year - startYear) / yearRange) * trainMaxMove
    }));
  
    // Train position in pixels
    $: trainPosition = scrollProgress * trainMaxMove;
  
    // Determine which event should be active
    $: activeIndex = eventPositions.reduce((lastIndex, event, index) => {
      return trainPosition >= event.position ? index : lastIndex;
    }, -1);
  </script>
  
  <style>
    body {
      margin: 0;
      background: transparent;
    }
  
    .sticky-title {
      position: sticky;
      top: 0;
      z-index: 100;
      padding: 1rem;
      background: white;
      animation: fadeIn 0.5s ease;
    }
  
    .sticky-title h1 {
      margin: 0;
      font-family: Helvetica, sans-serif;
      text-align: center;
      font-size: 2rem;
      border-radius: 4px;
    }
  
    #timeline-wrapper {
      height: 400vh;
      position: relative;
      background: transparent;
    }
  
    .timeline-sticky {
      position: sticky;
      top: 20vh;
      height: 300px;
      width: 100%;
      overflow: visible;
      background: transparent;
    }
  
    .track {
      position: absolute;
      bottom: 100px;
      width: 100%;
      height: 40px;
      z-index: 5;
    }
  
    .rail {
      position: absolute;
      height: 4px;
      background: #555;
      width: 100%;
      left: 0;
    }
  
    .rail.top {
      top: 0;
    }
  
    .rail.bottom {
      bottom: 0;
    }
  
    .period-bar {
      position: absolute;
      height: 6px;
      bottom: -12px;
      border-radius: 3px;
    }
  
    .cross-tie-container {
      position: absolute;
      top: 0;
      height: 40px;
      width: 100%;
      display: flex;
      justify-content: space-between;
      pointer-events: none;
    }
  
    .cross-tie {
      width: 4px;
      height: 100%;
      background: #333;
    }
  
    .period-labels {
      position: absolute;
      top: 60px;
      width: 100%;
      font-size: 0.85rem;
      color: #000;
      pointer-events: none;
    }
  
    .period-label {
      position: absolute;
      text-align: center;
      transform: translateX(-50%);
    }
  
    .train {
      position: absolute;
      bottom: 150px;
      width: 120px;
      z-index: 10;
      transform: translateX(var(--trainX)) scaleX(-1);
      filter: saturate(20);
    }
  
    .event-boxes {
      position: absolute;
      top: 280px;
      width: 100%;
      display: flex;
      justify-content: center;
      z-index: 1;
    }
  
    .event {
      position: absolute;
      opacity: 0;
      transition: opacity 0.3s ease;
      font-size: 1rem;
      background: lightgray;
      color: #111;
      padding: 1.5rem;
      border-radius: 8px;
      width: 700px;
      height: 220px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
      line-height: 1.5;
      text-align: center;
      word-break: break-word;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      justify-content: center;
      border: 6px solid transparent;
    }
  
    .event.active {
      opacity: 1;
    }
  
    .event[data-period="Start of zoning"] {
      border-color: #2F5DA6;
    }
  
    .event[data-period="Rapid Population Growth"] {
      border-color: #FD8A03;
    }
  
    .event[data-period="Exclusionary Zoning/Big Downzone"] {
      border-color: #FA2D27;
    }
  
    .event[data-period="Zoning Reform"] {
      border-color: #008150;
    }
  
    .event h2 {
      margin-bottom: 0.5rem;
      font-size: 1.5rem;
    }
  
    .event h3 {
      margin-top: 0;
      font-size: 1.2rem;
      font-weight: 500;
      color: #222;
    }
  
    .event p {
      margin-top: 0.75rem;
      font-size: 1rem;
      color: #333;
    }
  </style>
  
  <!-- Timeline Wrapper -->
  <div id="timeline-wrapper">
    <div class="timeline-sticky">
      <!-- Tracks -->
      <div class="track">
        <div class="rail top"></div>
        <div class="rail bottom"></div>
        <div class="cross-tie-container">
          {#each Array(30) as _, i}
            <div class="cross-tie" />
          {/each}
        </div>
        {#each periods as period}
          <div
            class="period-bar"
            style="left: {(period.start - startYear) / yearRange * 100}%; width: {(period.end - period.start) / yearRange * 100}%; background: {period.color}"
          ></div>
        {/each}
        <div class="period-labels">
          {#each periods as period}
            <div class="period-label" style="left: {(period.start - startYear + (period.end - period.start) / 2) / yearRange * 100}%">
              <span style="color: {period.color}">{period.label} <br />({period.start}–{period.end})</span>
            </div>
          {/each}
        </div>
      </div>
  
      <!-- Train -->
      <img
        class="train"
        src="images/train2.png"
        alt="Train"
        style="--trainX: {scrollProgress * trainMaxMove}px"
      />
  
      <!-- Events -->
      <div class="event-boxes">
        {#each eventPositions as event, i}
          <div
            class="event {i === activeIndex ? 'active' : ''}"
            data-period={periods.find(p => event.year >= p.start && event.year <= p.end)?.label || ''}
          >
            <h2>{event.year}</h2>
            <h3>{event.title}</h3>
            <p>{event.description}</p>
          </div>
        {/each}
      </div>
    </div>
  </div>
  <div style="height: 35vh;display: flex; align-items: center; justify-content: center;">
  </div>
  