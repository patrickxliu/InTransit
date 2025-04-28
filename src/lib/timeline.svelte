<script>
    let periods = [
      {
        id: 1,
        name: "Start of zoning",
        years: "1920–1930",
        color: "#f87171",
        events: [
          { id: 1, year: 1920, description: "Massachusetts Zoning Enabling Act" },
        ],
      },
      {
        id: 2,
        name: "Rapid Population Growth",
        years: "Post war–1968",
        color: "#60a5fa",
        events: [
          { id: 2, year: 1956, description: "Boston Zoning Charter Adopted" },
          { id: 3, year: 1957, description: "Boston Redevelopment Authority Founded" },
          { id: 4, year: 1958, description: "Boston’s West End cleared" },
          { id: 5, year: 1960, description: "City Planning Board merges with BRA" },
          { id: 6, year: 1966, description: "Home Rule Amendment" },
        ],
      },
      {
        id: 3,
        name: "Big Downzone/Exclusionary Zoning",
        years: "1968–1975",
        color: "#34d399",
        events: [
          { id: 7, year: 1968, description: "Fair Housing Act" },
          { id: 8, year: 1969, description: "Anti-Snob Zoning Law (Chapter 40B)" },
        ],
      },
      {
        id: 4,
        name: "Efforts for Zoning Reform",
        years: "1975–Present (2025)",
        color: "#fbbf24",
        events: [
          { id: 9, year: 1975, description: "Reforming State Zoning Act (Chapter 40A)" },
          { id: 10, year: 1993, description: "Economic Development and Industrial Corporation Act" },
          { id: 11, year: 2021, description: "Fair Housing Zoning Amendment, MBTA Communities Act" },
          { id: 12, year: 2024, description: "Mayor Wu establishes Planning Department" },
        ],
      },
    ];
  
    let openPeriodId = null;
  
    function togglePeriod(periodId) {
      openPeriodId = openPeriodId === periodId ? null : periodId;
    }
  </script>
  
  <style>
    .timeline-wrapper {
      overflow-x: auto;
      padding: 2rem;
      white-space: nowrap;
    }
  
    .train-tracks {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: space-around;
      min-width: 1200px;
      padding: 5rem 2rem;
    }
  
    /* Cross ties are now moved up and can be enlarged */
    .ties {
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      display: flex;
      justify-content: space-between;
      align-items: center;
      pointer-events: none;
      z-index: 2; /* over the tracks */
    }
  
    .tie {
      width: 2px;
      height: 25px; /* Shortened height */
      background: #333;
      margin-top: -17px; /* Move up by negative value */
    }
  
    /* Tracks (rails) below cross ties */
    .track {
      position: absolute;
      height: 4px;
      background-color: #444;
      width: 100%;
      left: 0;
      z-index: 1; /* below cross ties */
    }
  
    .track-top {
      top: 40%;
    }
  
    .track-bottom {
      top: 50%;
    }
  
    /* Stations */
    .station {
      display: flex;
      flex-direction: column;
      align-items: center;
      cursor: pointer;
      flex-shrink: 0;
      width: 160px;
      z-index: 3; /* above everything */
    }
  
    .station-dot {
      width: 2rem;
      height: 2rem;
      background-color: var(--period-color);
      border: 4px solid white;
      border-radius: 50%;
      box-shadow: 0 0 0 3px var(--period-color);
      margin-bottom: 0.5rem;
    }
  
    .station-label {
      font-weight: bold;
      font-size: 0.9rem;
      text-align: center;
    }
  
    .events-container {
      display: flex;
      justify-content: center;
      margin-top: 2rem;
    }
  
    .event-list {
      background: #f3f4f6;
      padding: 1rem;
      border-radius: 0.5rem;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      width: fit-content;
      min-width: 300px;
      text-align: left;
    }
  
    .event {
      margin: 0.5rem 0;
      font-size: 0.9rem;
    }
  
    /* Mini Timeline Styling */
    .mini-timeline-wrapper {
      overflow-x: auto;
      padding: 1rem 0;
      margin-top: 2rem;
      white-space: nowrap;
    }
  
    .mini-timeline {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: space-around;
    }
  
    .mini-station {
      display: flex;
      flex-direction: column;
      align-items: center;
      cursor: pointer;
      flex-shrink: 0;
      width: 100px;
    }
  
    .mini-station-dot {
      width: 1.5rem;
      height: 1.5rem;
      background-color: var(--period-color);
      border: 4px solid white;
      border-radius: 50%;
      box-shadow: 0 0 0 3px var(--period-color);
      margin-bottom: 0.5rem;
    }
  
    .mini-station-label {
      font-weight: bold;
      font-size: 0.8rem;
      text-align: center;
    }
  
    /* Enlarge event when clicked */
    .event-enlarged {
      font-size: 1.2rem;
      font-weight: bold;
      margin: 1rem 0;
      padding: 0.5rem;
      background: #fef7e7;
      border-left: 5px solid #f87171;
    }
  
  </style>
  
  <div class="timeline-wrapper">
    <div class="train-tracks">
      <!-- Cross ties above the tracks -->
      <div class="ties">
        {#each Array(40) as _, i}
          <div class="tie"></div>
        {/each}
      </div>
  
      <!-- Tracks -->
      <div class="track track-top"></div>
      <div class="track track-bottom"></div>
  
      <!-- Stations -->
      {#each periods as period (period.id)}
        <div 
          class="station"
          style="--period-color: {period.color};"
          on:click={() => togglePeriod(period.id)}
        >
          <div class="station-dot"></div>
          <div class="station-label">
            {period.years}<br />{period.name}
          </div>
        </div>
      {/each}
    </div>
  </div>
  
  {#if openPeriodId}
    {#each periods as period (period.id)}
      {#if period.id === openPeriodId}
        <div class="events-container">
          <div class="event-list">
            {#each period.events as event (event.id)}
              <div class="event {openPeriodId === period.id ? 'event-enlarged' : ''}">
                <strong>{event.year}</strong>: {event.description}
              </div>
            {/each}
          </div>
        </div>
      {/if}
    {/each}
  {/if}
  
  
  