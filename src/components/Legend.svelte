<script>
  import BasinSvg from "./BasinSvg.svelte";
  import getCSSVar from "../utils/getCSSVar.js";
</script>

<div class="legend">
  <div class="items">
    <div class="legend-item">
      <div class="gauge" style="border-color: {getCSSVar('marker-color')}"></div>
      <span>Chemical plant exempted from the HON rule</span>
    </div>
    <div class="legend-item">
     <BasinSvg width=25 fill="#B0B3B3"/>
      <span>Mississippi/Atchafalaya River Basin</span>
    </div>
  </div>
</div>

<style>
  .legend {
    margin-bottom: 0.5rem;
  }

  .items {
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 0.15rem;
  }
  .legend-item span {
    font-size: 0.875rem;
    color: #656568;
  }

  .gauge {
    width: 12px;
    height: 12px;
    border: 2px solid;
    border-radius: 100%;
  }
</style>
