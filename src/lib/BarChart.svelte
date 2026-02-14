<script>
  export let data = [];
  export let color = 'var(--accent)';
  export let showValue = false;

  $: maxVal = Math.max(...data.map(d => d.value), 1);
</script>

<div class="bar-chart">
  {#each data as item, i}
    <div class="bar-row" style="animation-delay: {i * 40}ms">
      <span class="bar-label">{item.label}</span>
      <div class="bar-track">
        <div
          class="bar-fill"
          style="width: {(item.value / maxVal) * 100}%; background: {color};"
        ></div>
      </div>
      <span class="bar-value">{showValue ? item.value : item.value}</span>
    </div>
  {/each}
</div>

<style>
  .bar-chart {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .bar-row {
    display: grid;
    grid-template-columns: 70px 1fr 45px;
    align-items: center;
    gap: 0.6rem;
    animation: fadeSlide 0.4s ease both;
  }

  @keyframes fadeSlide {
    from { opacity: 0; transform: translateX(-8px); }
    to { opacity: 1; transform: translateX(0); }
  }

  .bar-label {
    font-size: 0.72rem;
    color: var(--text-dim);
    text-align: right;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .bar-track {
    height: 22px;
    background: rgba(255,255,255,0.04);
    border-radius: 6px;
    overflow: hidden;
  }

  .bar-fill {
    height: 100%;
    border-radius: 6px;
    transition: width 0.6s cubic-bezier(0.22, 1, 0.36, 1);
    min-width: 2px;
  }

  .bar-value {
    font-size: 0.75rem;
    font-weight: 600;
    color: var(--text);
    text-align: right;
    font-variant-numeric: tabular-nums;
  }
</style>
