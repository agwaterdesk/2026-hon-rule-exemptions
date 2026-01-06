<script>
  import Window from "./components/Window.svelte";
  import Map from "./components/Map.svelte";
  import sitesData from "./data/sites.csv";
  import basin from "./data/mississippi-river-basin.json";
  import Legend from "./components/Legend.svelte";

  // Handle responsive iframes for embeds
  import pym from "pym.js";

  new pym.Child({
    polling: 500,
  });

  function getUrlParameter(name) {
    const params = new URLSearchParams(window.location.search);
    return params.get(name);
  }

  let includeCredit = getUrlParameter("credit") != "false";

  // Filter sites for Chemical Manufacturing and July 2025 Proclamation
  const filteredSites = sitesData.filter(
    (site) =>
      site.Type_of_Facility === "Chemical Manufacturing" &&
      site.Source_Name === "July 2025 Proclamation",
  );
</script>

<Window />
<!-- Outer div must have class 'chart-container' don't change -->
<div class="chart-container">
  <h1 class="headline">
    Most chemical plants exempted from the HON rule are in the Mississippi River
    Basin
  </h1>

  <p class="dek">
    A July 2025 proclamation from President Trump temporarily exempted dozens of
    chemical plants from complying with stricter pollution standards finalized
    under President Biden. Most of those facilities are in the Mississippi River
    Basin.
  </p>
  <p class="sr-only">
    A map showing chemical manufacturing facilities temporarily exempted from the HON rule
    within the Mississippi/Atchafalaya River Basin.
  </p>

  <Legend />
  <div id="g-viz">
    <Map sites={filteredSites} {basin} />
  </div>

  {#if includeCredit}
    <div class="credit">
      Data: <a
        href="https://www.edf.org/maps/epa-pollution-pass/"
        target="_blank">Environmental Defense Fund</a
      >; Graphic by Jared Whalen /
      <a target="_blank" href="https://agwaterdesk.org/">Ag & Water Desk</a>
    </div>
  {/if}
</div>

<style lang="scss">
  .chart-container {
    max-width: 800px;
    width: 100%;
    padding: 0.5rem;

    #g-viz {
      position: relative;
      width: 100%;
      height: 600px;
      @include mq(600px, "max-width") {
        height: 600px;
      }
    }
  }
</style>
