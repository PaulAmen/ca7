<script>
  export let label = '';
  export let value = 0;
  export let icon = '';
</script>

<div class="kpi">
  <div class="kpi-icon">{icon}</div>
  <div class="kpi-body">
    <span class="kpi-value">{value}</span>
    <span class="kpi-label">{label}</span>
  </div>
</div>

<style>
  .kpi {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.1rem 1.25rem;
    display: flex;
    align-items: center;
    gap: 0.85rem;
    transition: transform 0.2s, border-color 0.3s, box-shadow 0.3s;
  }

  .kpi:hover {
    transform: translateY(-2px);
    border-color: rgba(79, 140, 255, 0.35);
    box-shadow: 0 8px 24px rgba(0,0,0,0.2);
  }

  .kpi-icon {
    font-size: 1.6rem;
    width: 44px;
    height: 44px;
    background: var(--accent-glow);
    border-radius: 10px;
    display: grid;
    place-items: center;
    flex-shrink: 0;
  }

  .kpi-body {
    display: flex;
    flex-direction: column;
  }

  .kpi-value {
    font-family: 'DM Sans', sans-serif;
    font-size: 1.55rem;
    font-weight: 700;
    letter-spacing: -0.02em;
    line-height: 1.15;
  }

  .kpi-label {
    font-size: 0.72rem;
    color: var(--text-dim);
    letter-spacing: 0.02em;
    margin-top: 0.1rem;
  }
</style>
