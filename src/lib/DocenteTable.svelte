<script>
  export let data = [];

  $: visible = data.slice(0, 15);
  $: maxCount = visible.length ? visible[0].count : 1;

  function capitalize(str) {
    return str.split(' ').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ');
  }
</script>

<div class="docente-list">
  {#each visible as item, i}
    <div class="docente-row" style="animation-delay: {i * 30}ms">
      <span class="rank">#{i+1}</span>
      <div class="docente-info">
        <span class="name">{capitalize(item.name)}</span>
        <div class="mini-bar-track">
          <div
            class="mini-bar"
            style="width: {(item.count / maxCount) * 100}%"
          ></div>
        </div>
      </div>
      <span class="count">{item.count}</span>
    </div>
  {/each}
</div>

<style>
  .docente-list {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
    max-height: 400px;
    overflow-y: auto;
  }

  .docente-list::-webkit-scrollbar {
    width: 4px;
  }
  .docente-list::-webkit-scrollbar-track {
    background: transparent;
  }
  .docente-list::-webkit-scrollbar-thumb {
    background: var(--border);
    border-radius: 4px;
  }

  .docente-row {
    display: grid;
    grid-template-columns: 28px 1fr 36px;
    align-items: center;
    gap: 0.4rem;
    padding: 0.35rem 0.5rem;
    border-radius: 8px;
    transition: background 0.15s;
    animation: fadeSlide 0.35s ease both;
  }

  .docente-row:hover {
    background: var(--surface-hover);
  }

  @keyframes fadeSlide {
    from { opacity: 0; transform: translateY(4px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .rank {
    font-size: 0.65rem;
    color: var(--text-dim);
    text-align: center;
    font-variant-numeric: tabular-nums;
  }

  .docente-info {
    display: flex;
    flex-direction: column;
    gap: 0.2rem;
    min-width: 0;
  }

  .name {
    font-size: 0.72rem;
    color: var(--text);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .mini-bar-track {
    height: 3px;
    background: rgba(255,255,255,0.05);
    border-radius: 2px;
  }

  .mini-bar {
    height: 100%;
    background: var(--accent);
    border-radius: 2px;
    transition: width 0.5s ease;
  }

  .count {
    font-size: 0.72rem;
    font-weight: 600;
    color: var(--accent);
    text-align: right;
    font-variant-numeric: tabular-nums;
  }
</style>
