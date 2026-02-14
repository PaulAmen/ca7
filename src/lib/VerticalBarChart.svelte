<script>
  export let data = []; // [{ label: string, value: number, display?: string }]
  export let color = 'var(--accent)';
  export let height = 180;

  $: maxVal = Math.max(...data.map(d => d.value), 1);
</script>

<div class="vertical-chart-container" style="height: {height}px">
  <div class="bars-wrapper">
    {#each data as item, i}
      <div class="bar-column">
        <div class="bar-outer">
          <div 
            class="bar-inner" 
            style="height: {(item.value / maxVal) * 100}%; background: {color}; animation-delay: {i * 50}ms"
          >
            {#if item.value > 0}
              <span class="value-popup">{item.display || item.value}</span>
            {/if}
          </div>
        </div>
        <span class="label">{item.label}</span>
      </div>
    {/each}
  </div>
</div>

<style>
  .vertical-chart-container {
    width: 100%;
    display: flex;
    align-items: flex-end;
    padding-top: 2rem;
    padding-bottom: 1.5rem;
  }

  .bars-wrapper {
    display: flex;
    align-items: flex-end;
    justify-content: space-around;
    width: 100%;
    height: 100%;
    gap: 0.5rem;
  }

  .bar-column {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    min-width: 0;
  }

  .bar-outer {
    flex: 1;
    width: 100%;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    background: rgba(255, 255, 255, 0.03);
    border-radius: 6px 6px 0 0;
    position: relative;
  }

  .bar-inner {
    width: 70%;
    max-width: 40px;
    border-radius: 6px 6px 0 0;
    transition: height 0.8s cubic-bezier(0.17, 0.67, 0.83, 0.67);
    animation: growUp 0.8s ease-out both;
    position: relative;
  }

  .bar-inner:hover {
    filter: brightness(1.2);
    box-shadow: 0 0 15px var(--accent-glow);
  }

  .value-popup {
    position: absolute;
    top: -24px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 0.7rem;
    font-weight: 600;
    color: var(--text);
    white-space: nowrap;
    opacity: 0;
    transition: opacity 0.2s;
  }

  .bar-inner:hover .value-popup {
    opacity: 1;
  }

  .label {
    margin-top: 0.6rem;
    font-size: 0.65rem;
    color: var(--text-dim);
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 100%;
  }

  @keyframes growUp {
    from { height: 0; }
  }
</style>
