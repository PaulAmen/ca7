<script>
  import { onMount } from 'svelte';
  import KpiCard from './lib/KpiCard.svelte';
  import BarChart from './lib/BarChart.svelte';
  import VerticalBarChart from './lib/VerticalBarChart.svelte';
  import LugarChart from './lib/LugarChart.svelte';
  import HeatMap from './lib/HeatMap.svelte';
  import EcuadorMap from './lib/EcuadorMap.svelte';

  let data = [];
  let loading = true;
  let selectedFecha = 'all';
  let selectedSemestre = 'all';

  // Deduplicate data: Only one record per person (CED+Sem+Par+Lug) per Day
  $: uniqueData = (() => {
    const seen = new Set();
    return data.filter(d => {
      // Use composite key to avoid collisions from masked CEDs
      const key = `${d.fec}-${d.ced}-${d.sem}-${d.par}-${d.lug}`;
      if (seen.has(key)) return false;
      seen.add(key);
      return true;
    });
  })();

  // Derived
  $: fechas = [...new Set(uniqueData.map(d => d.fec))].sort();
  $: semestres = [...new Set(uniqueData.map(d => d.sem).filter(Boolean))].sort((a, b) => {
    const order = {'1ER NIVEL':1,'2DO NIVEL':2,'3ER NIVEL':3,'4TO NIVEL':4,'5TO NIVEL':5,'6TO NIVEL':6,'7MO NIVEL':7,'8VO NIVEL':8};
    return (order[a]||99) - (order[b]||99);
  });

  $: filtered = uniqueData.filter(d => {
    if (selectedFecha !== 'all' && d.fec !== selectedFecha) return false;
    if (selectedSemestre !== 'all' && d.sem !== selectedSemestre) return false;
    return true;
  });

  // HeatMap Data
  $: heatMapData = (() => {
    const paralelos = [...new Set(filtered.map(d => d.par).filter(Boolean))].sort();
    const map = {};
    
    // Deduplicate people in the current filtered set (across multiple days if 'All' is selected)
    const seenPeople = new Set();
    const uniquePeopleInSelection = filtered.filter(d => {
      const personKey = `${d.ced}-${d.sem}-${d.par}-${d.lug}`;
      if (seenPeople.has(personKey)) return false;
      seenPeople.add(personKey);
      return true;
    });

    uniquePeopleInSelection.forEach(d => {
      if (d.sem && d.par) {
        const key = `${d.sem}|${d.par}`;
        map[key] = (map[key] || 0) + 1;
      }
    });
    return {
      rows: semestres,
      cols: paralelos,
      map
    };
  })();

  // KPIs
  $: totalEstudiantes = new Set(filtered.map(d => `${d.ced}-${d.sem}-${d.par}-${d.lug}`)).size;
  $: estudiantesConTiempo = filtered.filter(d => d.min > 0);
  $: avgMinutos = estudiantesConTiempo.length ? Math.round(estudiantesConTiempo.reduce((s,d) => s + d.min, 0) / estudiantesConTiempo.length) : 0;
  $: maxTiempo = estudiantesConTiempo.length ? Math.max(...estudiantesConTiempo.map(d => d.min)) : 0;
  $: totalLugares = new Set(filtered.map(d => d.lug)).size;

  // Avg time per day
  $: avgTimeByDay = (() => {
    const dayMap = {};
    filtered.forEach(d => {
      if (d.min <= 0) return;
      if (!dayMap[d.fec]) dayMap[d.fec] = { sum: 0, count: 0 };
      dayMap[d.fec].sum += d.min;
      dayMap[d.fec].count++;
    });
    return Object.entries(dayMap)
      .sort(([a],[b]) => a.localeCompare(b))
      .map(([fec, v]) => {
        const avgMin = v.sum / v.count;
        const hours = Math.round(avgMin / 60 * 10) / 10;
        return { label: fec, value: hours, display: `${hours}h` };
      });
  })();

  // Students per day (already unique due to uniqueData)
  $: totalByDay = (() => {
    const dayMap = {};
    filtered.forEach(d => {
      dayMap[d.fec] = (dayMap[d.fec] || 0) + 1;
    });
    return Object.entries(dayMap)
      .sort(([a],[b]) => a.localeCompare(b))
      .map(([fec, count]) => ({ label: fec, value: count }));
  })();

  // By semestre
  $: bySemestre = semestres.map(s => ({
    label: (s || '').replace(' NIVEL',''),
    value: filtered.filter(d => d.sem === s).length
  }));

  // Average time by semestre
  $: avgTimeBySem = semestres.map(s => {
    const items = filtered.filter(d => d.sem === s && d.min > 0);
    const avgMin = items.length ? items.reduce((a,b) => a + b.min, 0) / items.length : 0;
    return {
      label: (s || '').replace(' NIVEL',''),
      value: Math.round(avgMin / 60 * 10) / 10
    };
  });

  // Lugares count
  $: allLugaresCount = (() => {
    const map = {};
    filtered.forEach(d => {
      const name = d.lug || 'N/D';
      map[name] = (map[name] || 0) + 1;
    });
    return Object.entries(map).map(([name, count]) => ({ name, count }));
  })();

  $: lugarCount = [...allLugaresCount]
    .sort((a,b) => b.count - a.count)
    .slice(0, 10);

  // Time distribution buckets
  $: timeDistribution = (() => {
    const buckets = [
      { label: '0 min', min: 0, max: 0, count: 0 },
      { label: '1-60', min: 1, max: 60, count: 0 },
      { label: '61-120', min: 61, max: 120, count: 0 },
      { label: '121-240', min: 121, max: 240, count: 0 },
      { label: '241-360', min: 241, max: 360, count: 0 },
      { label: '361+', min: 361, max: 9999, count: 0 },
    ];
    filtered.forEach(d => {
      for (const b of buckets) {
        if (d.min >= b.min && d.min <= b.max) { b.count++; break; }
      }
    });
    return buckets.map(b => ({ label: b.label, value: b.count }));
  })();

  // Individual records table
  let searchQuery = '';
  let detailPage = 0;
  const detailPerPage = 25;
  let detailSortKey = 'ced';
  let detailSortDir = 1;

  $: detailFiltered = filtered.filter(d => {
    if (!searchQuery) return true;
    const q = searchQuery.toLowerCase();
    const ced = String(d.ced || '').toLowerCase();
    const lug = String(d.lug || '').toLowerCase();
    const sem = String(d.sem || '').toLowerCase();
    return ced.includes(q) || lug.includes(q) || sem.includes(q);
  });

  $: detailSorted = [...detailFiltered].sort((a, b) => {
    let va = a[detailSortKey], vb = b[detailSortKey];
    if (detailSortKey === 'min') return (va - vb) * detailSortDir;
    return String(va).localeCompare(String(vb)) * detailSortDir;
  });

  $: detailTotalPages = Math.ceil(detailSorted.length / detailPerPage);
  $: detailPaged = detailSorted.slice(detailPage * detailPerPage, (detailPage + 1) * detailPerPage);

  function toggleDetailSort(key) {
    if (detailSortKey === key) detailSortDir *= -1;
    else { detailSortKey = key; detailSortDir = 1; }
    detailPage = 0;
  }

  function detailArrow(key) {
    if (detailSortKey !== key) return '';
    return detailSortDir === 1 ? ' ↑' : ' ↓';
  }

  $: if (searchQuery) detailPage = 0;

  onMount(async () => {
    try {
      const res = await fetch('./data.json');
      data = await res.json();
    } catch (e) {
      console.error('Error loading data:', e);
    }
    loading = false;
  });

  function formatMinutes(m) {
    const h = Math.floor(m / 60);
    const min = m % 60;
    return h > 0 ? `${h}h ${min}m` : `${min}m`;
  }
</script>

<div class="dashboard" class:loading>
  <!-- Header -->
  <header>
    <div class="header-content">
      <div class="brand">
        <img src="/logo.png" alt="UNESUM Logo" class="logo large" />
        <h2>Asistentes Casa Abierta <br> Educación en Acción VII</h2>
      </div>
      <div class="filters">
        <div class="filter-group">
          <label for="fecha">Fecha</label>
          <select id="fecha" bind:value={selectedFecha}>
            <option value="all">Todas ({fechas.length})</option>
            {#each fechas as f}
              <option value={f}>{f}</option>
            {/each}
          </select>
        </div>
        <div class="filter-group">
          <label for="semestre">Semestre</label>
          <select id="semestre" bind:value={selectedSemestre}>
            <option value="all">Todos</option>
            {#each semestres as s}
              <option value={s}>{s}</option>
            {/each}
          </select>
        </div>
      </div>
    </div>
  </header>

  {#if loading}
    <div class="loader">
      <div class="spinner"></div>
      <p>Cargando datos…</p>
    </div>
  {:else}
    <!-- KPIs -->
    <section class="kpis">
      <KpiCard label="Asistentes Totales" value={totalEstudiantes} icon="👥" />
      <KpiCard label="Tiempo Promedio" value={formatMinutes(avgMinutos)} icon="⏱" />
      <KpiCard label="Máx. Permanencia" value={formatMinutes(maxTiempo)} icon="🏛" />
      <KpiCard label="Ciudades de Origen" value={totalLugares} icon="📍" />
    </section>

    <!-- Daily charts -->
    <section class="charts-row">
      <div class="chart-card wide">
        <h3>Tiempo Promedio en Universidad por Día <span class="unit">(horas)</span></h3>
        <VerticalBarChart data={avgTimeByDay} color="var(--teal)" />
      </div>
      <div class="chart-card">
        <h3>Asistentes por Día</h3>
        <VerticalBarChart data={totalByDay} color="var(--accent)" />
      </div>
    </section>

    <!-- Charts row 1 -->
    <section class="charts-row">
      <div class="chart-card wide">
        <h3>Estudiantes por Nivel</h3>
        <BarChart data={bySemestre} color="var(--accent)" />
      </div>
      <div class="chart-card">
        <h3>Distribución de Permanencia</h3>
        <BarChart data={timeDistribution} color="var(--purple)" />
      </div>
    </section>

    <!-- Charts row 2 -->
    <section class="charts-row">
      <div class="chart-card wide">
        <h3>Tiempo Promedio por Nivel <span class="unit">(horas)</span></h3>
        <BarChart data={avgTimeBySem} color="var(--warm)" showValue />
      </div>
      <div class="chart-card">
        <h3>Top Ciudades</h3>
        <LugarChart data={lugarCount} />
      </div>
    </section>

    <!-- Map and Lugares -->
    <section class="charts-row">
      <div class="chart-card full">
        <h3>Mapa de Procedencia por Provincia</h3>
        <EcuadorMap data={allLugaresCount} />
      </div>
    </section>

    <!-- Heatmap Distribution -->
    <section class="charts-row">
      <div class="chart-card full">
        <h3>Concentración de Asistentes por Nivel y Paralelo</h3>
        <HeatMap heatData={heatMapData} />
      </div>
    </section>

    <!-- Individual Records Table -->
    <section class="table-section">
      <div class="chart-card full">
        <h3>Registros Individuales <span class="unit">({detailFiltered.length} registros)</span></h3>
        <div class="table-controls">
          <input
            type="text"
            placeholder="Buscar por cédula, ciudad o nivel…"
            bind:value={searchQuery}
          />
          <span class="result-count">{detailFiltered.length} de {filtered.length}</span>
        </div>
        <div class="table-wrapper">
          <table>
            <thead>
              <tr>
                <th class="sortable" on:click={() => toggleDetailSort('ced')}>
                  Cédula{detailArrow('ced')}
                </th>
                <th class="sortable" on:click={() => toggleDetailSort('sem')}>
                  Nivel{detailArrow('sem')}
                </th>
                <th>Par.</th>
                <th class="sortable" on:click={() => toggleDetailSort('lug')}>
                  Procedencia{detailArrow('lug')}
                </th>
                <th class="sortable" on:click={() => toggleDetailSort('fec')}>
                  Fecha{detailArrow('fec')}
                </th>
                <th class="sortable num" on:click={() => toggleDetailSort('min')}>
                  Tiempo{detailArrow('min')}
                </th>
              </tr>
            </thead>
            <tbody>
              {#each detailPaged as row}
                <tr>
                  <td class="mono ced">{row.ced}</td>
                  <td><span class="badge">{(row.sem || '').replace(' NIVEL','')}</span></td>
                  <td class="center">{row.par}</td>
                  <td class="lugar">{row.lug}</td>
                  <td class="mono">{row.fec}</td>
                  <td class="mono num" class:zero={row.min === 0}>
                    {row.min > 0 ? formatMinutes(row.min) : '—'}
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
        {#if detailTotalPages > 1}
          <div class="pagination">
            <button disabled={detailPage === 0} on:click={() => detailPage--}>← Anterior</button>
            <span class="page-info">{detailPage + 1} / {detailTotalPages}</span>
            <button disabled={detailPage >= detailTotalPages - 1} on:click={() => detailPage++}>Siguiente →</button>
          </div>
        {/if}
      </div>
    </section>
  {/if}

  <footer>
    <p>by paul.amen@unesum.edu.ec</p>
  </footer>
</div>

<style>
  :root {
    --bg: #0f131a;
    --surface: #1e232c;
    --surface-hover: #282e38;
    --border: #333945;
    --text: #e4e6ef;
    --text-dim: #8b8fa3;
    --accent: #4f8cff;
    --accent-glow: rgba(79, 140, 255, 0.15);
    --teal: #36d6b5;
    --warm: #ff8a4c;
    --purple: #a78bfa;
    --red: #f87171;
  }

  :global(*) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  :global(body) {
    background: var(--bg);
    color: var(--text);
    font-family: 'DM Sans', system-ui, sans-serif;
    -webkit-font-smoothing: antialiased;
    line-height: 1.5;
  }

  .dashboard {
    min-height: 100vh;
    padding-bottom: 4rem; /* Increased padding to accommodate fixed footer */
  }

  /* Header */
  header {
    background: var(--surface); /* Use a solid surface color for simplicity */
    border-bottom: 1px solid var(--border);
    padding: 1rem 2rem; /* Slightly reduced vertical padding */
    position: sticky;
    top: 0;
    z-index: 100;
    backdrop-filter: blur(12px); /* Keep backdrop blur for frosted glass effect */
  }

  .header-content {
    max-width: 1400px;
    margin: 0 auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 1.5rem; /* Increased gap */
  }

  .brand {
    display: flex;
    align-items: center;
    gap: 0.85rem;
  }

  .logo {
    width: 42px;
    height: 42px;
    border-radius: 10px; /* Assuming the logo might benefit from rounded corners like the old logo-mark */
    object-fit: contain; /* Ensure the image scales nicely within the bounds */
    background-color: white;
    padding: 5px;
  }

  .logo.large {
    width: 80px;
    height: 80px;
  }



  h1 {
    font-family: 'DM Sans', sans-serif;
    font-size: 1.5rem;
    font-weight: 700; /* Increased weight for headings */
    letter-spacing: -0.02em;
    line-height: 1.1;
  }

  h2 { /* Styling for the new "Asistencias estudiantes casa abierta" */
    font-family: 'DM Sans', sans-serif;
    font-size: 1.3rem; /* Slightly smaller than original h1, but still prominent */
    font-weight: 700; /* Increased weight for headings */
    letter-spacing: -0.01em;
    color: var(--text); /* Use primary text color */
    line-height: 1.2;
    margin-top: 0.2rem; /* Small margin to separate from logo */
  }

  .subtitle {
    font-size: 0.78rem;
    color: var(--text-dim);
    letter-spacing: 0.02em;
  }

  .filters {
    display: flex;
    gap: 1rem;
  }

  .filter-group {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .filter-group label {
    font-size: 0.65rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: var(--text-dim);
  }

  select {
    background: var(--surface-hover); /* Use surface-hover for a subtle distinction */
    border: 1px solid var(--border);
    color: var(--text);
    padding: 0.5rem 0.85rem; /* Slightly more padding */
    border-radius: 10px; /* More rounded corners */
    font-family: inherit;
    font-size: 0.82rem;
    cursor: pointer;
    transition: border-color 0.2s, box-shadow 0.2s; /* Add box-shadow to transition */
    min-width: 140px;
    -webkit-appearance: none; /* Remove default arrow on WebKit browsers */
    -moz-appearance: none; /* Remove default arrow on Firefox */
    appearance: none; /* Remove default arrow */
    background-image: url('data:image/svg+xml;utf8,<svg fill="%23e4e6ef" height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M7 10l5 5 5-5z"/><path d="M0 0h24v24H0z" fill="none"/></svg>'); /* Custom arrow */
    background-repeat: no-repeat;
    background-position: right 0.5rem center;
    background-size: 1.2em;
  }

  select:hover {
    border-color: var(--accent);
  }

  select:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-glow); /* Slightly larger glow */
  }

  /* Loading */
  .loader {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 6rem 2rem;
    gap: 1rem;
    color: var(--text-dim);
  }

  .spinner {
    width: 36px;
    height: 36px;
    border: 3px solid var(--border);
    border-top-color: var(--accent);
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* Sections */
  section {
    max-width: 1400px;
    margin: 0 auto;
    padding: 0 2rem;
  }

  .kpis {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
    margin-top: 1.5rem;
  }

  .charts-row {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 1rem;
    margin-top: 1rem;
  }

  .chart-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px; /* Slightly reduced border-radius for a sleeker look */
    padding: 1.5rem; /* Increased padding for more breathing room */
    transition: border-color 0.3s, box-shadow 0.3s; /* Add box-shadow to transition */
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); /* Subtle shadow */
  }

  .chart-card:hover {
    border-color: rgba(79, 140, 255, 0.3);
  }

  .chart-card.full {
    grid-column: 1 / -1;
  }

  .chart-card h3 {
    font-size: 1rem; /* Increased font size for better prominence */
    font-weight: 600; /* Bolder font weight */
    color: var(--text); /* Use primary text color */
    margin-bottom: 0.75rem; /* Slightly reduced margin */
    letter-spacing: 0em; /* Reset letter spacing */
  }

  .chart-card h3 .unit {
    font-weight: 300;
    font-size: 0.75rem;
    opacity: 0.6;
  }

  .table-section {
    margin-top: 1rem;
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
  }

  thead {
    background: rgba(255,255,255,0.03);
  }

  th {
    padding: 0.65rem 0.8rem;
    text-align: left;
    font-weight: 500;
    color: var(--text-dim);
    font-size: 0.68rem;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    border-bottom: 1px solid var(--border);
    white-space: nowrap;
  }

  th.num, td.num {
    text-align: right;
  }

  td {
    padding: 0.5rem 0.8rem;
    border-bottom: 1px solid rgba(255,255,255,0.03);
    color: var(--text);
  }

  tr:hover td {
    background: rgba(79, 140, 255, 0.04);
  }

  .center { text-align: center; }

  .mono {
    font-variant-numeric: tabular-nums;
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

  .table-controls {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 0.75rem;
  }

  .table-controls input {
    flex: 1;
    background: var(--surface-hover); /* Consistent with select background */
    border: 1px solid var(--border);
    color: var(--text);
    padding: 0.5rem 0.85rem; /* Consistent padding */
    border-radius: 10px; /* Consistent rounded corners */
    font-family: inherit;
    font-size: 0.8rem;
    transition: border-color 0.2s, box-shadow 0.2s;
    max-width: 380px;
  }

  .table-controls input::placeholder { color: var(--text-dim); }
  .table-controls input:focus {
    outline: none;
    border-color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-glow);
  }

  .result-count {
    font-size: 0.72rem;
    color: var(--text-dim);
    white-space: nowrap;
  }

  .sortable {
    cursor: pointer;
  }

  .sortable:hover {
    color: var(--accent);
  }

  .ced {
    letter-spacing: 0.05em;
    color: var(--text-dim);
  }

  .lugar {
    text-transform: capitalize;
    font-size: 0.72rem;
    color: var(--text-dim);
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
    background: var(--surface); /* Consistent with other interactive elements */
    border: 1px solid var(--border);
    color: var(--text);
    padding: 0.5rem 1rem; /* Slightly more generous padding */
    border-radius: 10px; /* Consistent rounded corners */
    font-family: inherit;
    font-size: 0.78rem; /* Slightly larger font */
    cursor: pointer;
    transition: all 0.15s;
  }

  .pagination button:hover:not(:disabled) {
    border-color: var(--accent);
    color: var(--accent);
    box-shadow: 0 0 0 3px var(--accent-glow); /* Consistent hover effect */
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

  footer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(30, 35, 44, 0.8); /* Semi-transparent surface color */
    backdrop-filter: blur(10px);
    border-top: 1px solid var(--border);
    padding: 0.75rem 2rem;
    text-align: center;
    color: var(--text); /* More prominent color */
    font-size: 0.9rem; /* Increased font size */
    z-index: 100;
  }

  /* Responsive */
  @media (max-width: 900px) {
    .charts-row {
      grid-template-columns: 1fr;
    }
    .header-content {
      flex-direction: column;
      align-items: flex-start;
    }
    header {
      padding: 1rem;
    }
    section {
      padding: 0 1rem;
    }
  }

  @media (max-width: 500px) {
    .kpis {
      grid-template-columns: 1fr 1fr;
    }
  }
</style>
