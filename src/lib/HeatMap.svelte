<script>
  export let heatData = { rows: [], cols: [], map: {} };

  $: maxVal = Math.max(...Object.values(heatData.map), 1);

  function getColor(val) {
    if (!val) return 'rgba(255,255,255,0.02)';
    const intensity = val / maxVal;
    if (intensity < 0.25) return `rgba(79, 140, 255, ${0.15 + intensity * 0.5})`;
    if (intensity < 0.5) return `rgba(79, 140, 255, ${0.3 + intensity * 0.4})`;
    if (intensity < 0.75) return `rgba(54, 214, 181, ${0.4 + intensity * 0.3})`;
    return `rgba(54, 214, 181, ${0.6 + intensity * 0.35})`;
  }
</script>

<div class="heatmap-wrapper">
  <div class="heatmap" style="grid-template-columns: 90px repeat({heatData.cols.length}, 1fr);">
    <!-- Header row -->
    <div class="corner"></div>
    {#each heatData.cols as col}
      <div class="col-header">{col}</div>
    {/each}

    <!-- Data rows -->
    {#each heatData.rows as row}
      <div class="row-label">{row.replace(' NIVEL','')}</div>
      {#each heatData.cols as col}
        {@const val = heatData.map[`${row}|${col}`] || 0}
        <div
          class="cell"
          style="background: {getColor(val)};"
          title="{row} - Paralelo {col}: {val} estudiantes"
        >
          {#if val > 0}
            <span>{val}</span>
          {/if}
        </div>
      {/each}
    {/each}
  </div>
</div>

<style>
  .heatmap-wrapper {
    overflow-x: auto;
  }

  .heatmap {
    display: grid;
    gap: 3px;
    min-width: 500px;
  }

  .corner {
    /* empty top-left corner */
  }

  .col-header {
    text-align: center;
    font-size: 0.7rem;
    font-weight: 600;
    color: var(--text-dim);
    padding: 0.3rem 0;
    letter-spacing: 0.05em;
  }

  .row-label {
    font-size: 0.68rem;
    color: var(--text-dim);
    display: flex;
    align-items: center;
    justify-content: flex-end;
    padding-right: 0.5rem;
    white-space: nowrap;
  }

  .cell {
    border-radius: 6px;
    min-height: 36px;
    display: grid;
    place-items: center;
    transition: transform 0.15s, box-shadow 0.15s;
    cursor: default;
  }

  .cell:hover {
    transform: scale(1.08);
    box-shadow: 0 4px 12px rgba(0,0,0,0.3);
    z-index: 2;
  }

  .cell span {
    font-size: 0.7rem;
    font-weight: 600;
    color: white;
    text-shadow: 0 1px 2px rgba(0,0,0,0.3);
  }
</style>
