<script>
  export let data = [];
  export let formatMinutes = (m) => m + 'm';

  let search = '';
  let page = 0;
  const perPage = 20;
  let sortKey = 'ap';
  let sortDir = 1;

  $: searched = data.filter(d => {
    if (!search) return true;
    const q = search.toLowerCase();
    return (
      d.ap.toLowerCase().includes(q) ||
      d.nom.toLowerCase().includes(q) ||
      d.id.includes(q) ||
      d.lug.toLowerCase().includes(q)
    );
  });

  $: sorted = [...searched].sort((a, b) => {
    let va = a[sortKey], vb = b[sortKey];
    if (sortKey === 'min') return (va - vb) * sortDir;
    return String(va).localeCompare(String(vb)) * sortDir;
  });

  $: totalPages = Math.ceil(sorted.length / perPage);
  $: paged = sorted.slice(page * perPage, (page + 1) * perPage);

  function toggleSort(key) {
    if (sortKey === key) sortDir *= -1;
    else { sortKey = key; sortDir = 1; }
    page = 0;
  }

  function arrow(key) {
    if (sortKey !== key) return '';
    return sortDir === 1 ? ' ↑' : ' ↓';
  }

  $: if (search) page = 0;
</script>

<div class="table-controls">
  <input
    type="text"
    placeholder="Buscar por nombre, cédula o ciudad…"
    bind:value={search}
  />
  <span class="result-count">{searched.length} resultados</span>
</div>

<div class="table-wrapper">
  <table>
    <thead>
      <tr>
        <th on:click={() => toggleSort('ap')} class="sortable">
          Apellidos{arrow('ap')}
        </th>
        <th on:click={() => toggleSort('nom')} class="sortable">
          Nombres{arrow('nom')}
        </th>
        <th on:click={() => toggleSort('sem')} class="sortable">
          Nivel{arrow('sem')}
        </th>
        <th>Par.</th>
        <th on:click={() => toggleSort('lug')} class="sortable">
          Procedencia{arrow('lug')}
        </th>
        <th>Fecha</th>
        <th>Entrada</th>
        <th>Salida</th>
        <th on:click={() => toggleSort('min')} class="sortable num">
          Tiempo{arrow('min')}
        </th>
      </tr>
    </thead>
    <tbody>
      {#each paged as row}
        <tr>
          <td class="name-cell">{row.ap}</td>
          <td class="name-cell">{row.nom}</td>
          <td><span class="badge">{row.sem.replace(' NIVEL','')}</span></td>
          <td class="center">{row.par}</td>
          <td class="lugar">{row.lug}</td>
          <td class="mono">{row.fec}</td>
          <td class="mono">{row.ent}</td>
          <td class="mono">{row.sal}</td>
          <td class="mono num" class:zero={row.min === 0}>
            {row.min > 0 ? formatMinutes(row.min) : '—'}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

{#if totalPages > 1}
  <div class="pagination">
    <button disabled={page === 0} on:click={() => page--}>← Anterior</button>
    <span class="page-info">{page + 1} / {totalPages}</span>
    <button disabled={page >= totalPages - 1} on:click={() => page++}>Siguiente →</button>
  </div>
{/if}

<style>
  .table-controls {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 0.75rem;
  }

  input {
    flex: 1;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 0.5rem 0.85rem;
    border-radius: 8px;
    font-family: inherit;
    font-size: 0.8rem;
    transition: border-color 0.2s;
    max-width: 380px;
  }

  input::placeholder { color: var(--text-dim); }
  input:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 2px var(--accent-glow);
  }

  .result-count {
    font-size: 0.72rem;
    color: var(--text-dim);
    white-space: nowrap;
  }

  .table-wrapper {
    overflow-x: auto;
    border-radius: 10px;
    border: 1px solid var(--border);
  }

  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.75rem;
    min-width: 800px;
  }

  thead {
    background: rgba(255,255,255,0.03);
  }

  th {
    padding: 0.65rem 0.6rem;
    text-align: left;
    font-weight: 500;
    color: var(--text-dim);
    font-size: 0.68rem;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    border-bottom: 1px solid var(--border);
    white-space: nowrap;
    user-select: none;
  }

  th.sortable {
    cursor: pointer;
  }

  th.sortable:hover {
    color: var(--accent);
  }

  th.num, td.num {
    text-align: right;
  }

  td {
    padding: 0.5rem 0.6rem;
    border-bottom: 1px solid rgba(255,255,255,0.03);
    color: var(--text);
  }

  tr:hover td {
    background: rgba(79, 140, 255, 0.04);
  }

  .name-cell {
    max-width: 160px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .center { text-align: center; }

  .mono {
    font-variant-numeric: tabular-nums;
    font-size: 0.72rem;
  }

  .lugar {
    text-transform: capitalize;
    font-size: 0.72rem;
    color: var(--text-dim);
  }

  .badge {
    background: var(--accent-glow);
    color: var(--accent);
    padding: 0.15rem 0.45rem;
    border-radius: 5px;
    font-size: 0.65rem;
    font-weight: 600;
    white-space: nowrap;
  }

  .zero {
    color: var(--text-dim);
  }

  .pagination {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-top: 0.85rem;
    padding-top: 0.5rem;
  }

  .pagination button {
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 0.4rem 0.9rem;
    border-radius: 7px;
    font-family: inherit;
    font-size: 0.72rem;
    cursor: pointer;
    transition: all 0.15s;
  }

  .pagination button:hover:not(:disabled) {
    border-color: var(--accent);
    color: var(--accent);
  }

  .pagination button:disabled {
    opacity: 0.3;
    cursor: not-allowed;
  }

  .page-info {
    font-size: 0.72rem;
    color: var(--text-dim);
    font-variant-numeric: tabular-nums;
  }
</style>
