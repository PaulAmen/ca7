<script>
  export let data = [];

  $: maxVal = data.length ? data[0].count : 1;
</script>

<div class="lugar-chart">
  {#each data as item, i}
    <div class="lugar-row" style="animation-delay: {i * 35}ms">
      <span class="lugar-name" title={item.name}>{item.name}</span>
      <div class="lugar-bar-track">
        <div
          class="lugar-bar"
          style="width: {(item.count / maxVal) * 100}%"
        ></div>
      </div>
      <span class="lugar-count">{item.count}</span>
    </div>
  {/each}
</div>

<style>
  .lugar-chart {
    display: flex;
    flex-direction: column;
    gap: 0.45rem;
  }

  .lugar-row {
    display: grid;
    grid-template-columns: 120px 1fr 40px;
    align-items: center;
    gap: 0.6rem;
    animation: fadeIn 0.4s ease both;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  .lugar-name {
    font-size: 0.7rem;
    color: var(--text-dim);
    text-align: right;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    text-transform: capitalize;
  }

  .lugar-bar-track {
    height: 18px;
    background: rgba(255,255,255,0.03);
    border-radius: 5px;
    overflow: hidden;
  }

  .lugar-bar {
    height: 100%;
    background: linear-gradient(90deg, var(--warm), #ffc078);
    border-radius: 5px;
    transition: width 0.6s cubic-bezier(0.22, 1, 0.36, 1);
  }

  .lugar-count {
    font-size: 0.72rem;
    font-weight: 600;
    color: var(--warm);
    text-align: right;
    font-variant-numeric: tabular-nums;
  }
</style>
