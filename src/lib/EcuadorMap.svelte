<script>
  import { onMount } from 'svelte';
  export let data = []; // Array of { name, count } where name is the location (city)

  let geoData = null;
  let features = [];

  const locationToProvince = {
    '24 DE MAYO': 'MANABI', 'JIPIJAPA': 'MANABI', 'MANTA': 'MANABI', 'PORTOVIEJO': 'MANABI', 'CHONE': 'MANABI', 'MONTECRISTI': 'MANABI', 'BOLIVAR': 'MANABI', 'EL CARMEN': 'MANABI', 'FLAVIO ALFARO': 'MANABI', 'JARAMIJO': 'MANABI', 'JUNIN': 'MANABI', 'OLMEDO': 'MANABI', 'PAJAN': 'MANABI', 'PEDERNALES': 'MANABI', 'PUERTO LOPEZ': 'MANABI', 'ROCAFUERTE': 'MANABI', 'SAN VICENTE': 'MANABI', 'SANTA ANA': 'MANABI', 'SUCRE': 'MANABI', 'TOSAGUA': 'MANABI',
    'GUAYAQUIL': 'GUAYAS', 'DAULE': 'GUAYAS', 'DURAN': 'GUAYAS', 'BALZAR': 'GUAYAS', 'EL EMPALME': 'GUAYAS', 'ISIDRO AYORA': 'GUAYAS', 'LOMAS DE SARGENTILLO': 'GUAYAS', 'NOBOL': 'GUAYAS', 'PEDRO CARBO': 'GUAYAS', 'PLAYAS': 'GUAYAS', 'SALITRE': 'GUAYAS', 'SAN JACINTO DE YAGUACHI': 'GUAYAS', 'SANTA LUCIA': 'GUAYAS', 'SIMON BOLIVAR': 'GUAYAS',
    'ESMERALDAS': 'ESMERALDAS', 'ATACAMES': 'ESMERALDAS', 'ELOY ALFARO': 'ESMERALDAS', 'MUISNE': 'ESMERALDAS', 'QUININDE': 'ESMERALDAS', 'RIOVERDE': 'ESMERALDAS', 'SAN LORENZO': 'ESMERALDAS',
    'MACHALA': 'EL ORO', 'ARENILLAS': 'EL ORO', 'HUAQUILLAS': 'EL ORO', 'PASAJE': 'EL ORO', 'SANTA ROSA': 'EL ORO',
    'DISTRITO METROPOLITANO DE QUITO': 'PICHINCHA', 'PUERTO QUITO': 'PICHINCHA', 'RUMIÑAHUI': 'PICHINCHA',
    'SANTO DOMINGO': 'SANTO DOMINGO DE LOS TSACHILAS', 'LA CONCORDIA': 'SANTO DOMINGO DE LOS TSACHILAS',
    'SALINAS': 'SANTA ELENA', 'LA LIBERTAD': 'SANTA ELENA', 'SANTA ELENA': 'SANTA ELENA',
    'QUEVEDO': 'LOS RIOS', 'URDANETA': 'LOS RIOS', 'VALENCIA': 'LOS RIOS', 'VENTANAS': 'LOS RIOS',
    'CUENCA': 'AZUAY', 'LA TRONCAL': 'CAÑAR', 'IBARRA': 'IMBABURA', 'LATACUNGA': 'COTOPAXI',
    'SHUSHUFINDI': 'SUCUMBIOS', 'LA JOYA DE LOS SACHAS': 'ORELLANA', 'PUTUMAYO': 'SUCUMBIOS', 'TENA': 'NAPO', 'EL CHACO': 'NAPO',
    'PALTAS': 'LOJA', 'CELICA': 'LOJA', 'ZAPOTILLO': 'LOJA', 'TAISHA': 'MORONA SANTIAGO',
    'LAS NAVES': 'BOLIVAR', 'SANTA CRUZ': 'GALAPAGOS'
  };

  $: provinceStats = (() => {
    const stats = {};
    data.forEach(d => {
      const province = locationToProvince[d.name.toUpperCase()];
      if (province) {
        stats[province] = (stats[province] || 0) + d.count;
      }
    });
    return stats;
  })();

  $: maxCount = Math.max(...Object.values(provinceStats), 1);

  function getFillColor(count) {
    if (!count) return 'rgba(255,255,255,0.03)';
    const intensity = Math.min(count / maxCount, 1);
    return `rgba(79, 140, 255, ${0.2 + intensity * 0.8})`;
  }

  const width = 400;
  const height = 400;
  const lonMin = -81.5, lonMax = -75;
  const latMin = -5.2, latMax = 1.5;

  function project(lon, lat) {
    const x = ((lon - lonMin) / (lonMax - lonMin)) * width;
    const y = height - ((lat - latMin) / (latMax - latMin)) * height;
    return [x, y];
  }

  function genPath(geometry) {
    if (!geometry) return '';
    const type = geometry.type;
    const coords = geometry.coordinates;

    if (type === 'Polygon') {
      return coords.map(ring => {
        return 'M' + ring.map(p => project(p[0], p[1]).join(',')).join('L') + 'Z';
      }).join(' ');
    } else if (type === 'MultiPolygon') {
      return coords.map(poly => {
        return poly.map(ring => {
          return 'M' + ring.map(p => project(p[0], p[1]).join(',')).join('L') + 'Z';
        }).join(' ');
      }).join(' ');
    }
    return '';
  }

  onMount(async () => {
    try {
      const res = await fetch('./ecuador-provincias.json');
      geoData = await res.json();
      features = geoData.features.map(f => ({
        name: f.properties.nombre,
        path: genPath(f.geometry)
      }));
    } catch (e) {
      console.error('Error loading map data:', e);
    }
  });

  let hoveredProvince = null;
</script>

<div class="ecuador-map-container">
  <div class="map-wrapper">
    {#if features.length > 0}
      <svg viewBox="0 0 {width} {height}" preserveAspectRatio="xMidYMid meet">
        <g class="provinces">
          {#each features as province}
            <path
              d={province.path}
              fill={getFillColor(provinceStats[province.name])}
              stroke="var(--bg)"
              stroke-width="0.5"
              on:mouseenter={() => hoveredProvince = province.name}
              on:mouseleave={() => hoveredProvince = null}
              class:hovered={hoveredProvince === province.name}
            />
          {/each}
        </g>
      </svg>
    {:else}
      <div class="map-loader">Cargando mapa...</div>
    {/if}
    
    {#if hoveredProvince}
      <div class="map-tooltip">
        <strong>{hoveredProvince}</strong>
        <span>{provinceStats[hoveredProvince] || 0} asistentes</span>
      </div>
    {/if}
  </div>

  <div class="legend">
    <h4>Provincias</h4>
    <div class="legend-items scrollable">
      {#each Object.entries(provinceStats).sort((a,b) => b[1] - a[1]) as [name, count]}
        <div class="legend-item" on:mouseenter={() => hoveredProvince = name} on:mouseleave={() => hoveredProvince = null} class:active={hoveredProvince === name}>
          <span class="dot" style="background: {getFillColor(count)}"></span>
          <span class="name">{name.toLowerCase()}</span>
          <span class="val">{count}</span>
        </div>
      {/each}
    </div>
  </div>
</div>

<style>
  .ecuador-map-container {
    display: grid;
    grid-template-columns: 1fr 180px;
    gap: 1rem;
    align-items: center;
    min-height: 350px;
  }

  .map-wrapper {
    position: relative;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  svg {
    width: 100%;
    height: auto;
    max-height: 380px;
    filter: drop-shadow(0 10px 25px rgba(0,0,0,0.3));
  }

  path {
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    cursor: pointer;
  }

  path:hover, path.hovered {
    filter: brightness(1.3);
    stroke: var(--accent);
    stroke-width: 1.5;
    z-index: 10;
  }

  .map-loader {
    font-size: 0.8rem;
    color: var(--text-dim);
    animation: pulse 1.5s infinite;
  }

  @keyframes pulse {
    50% { opacity: 0.5; }
  }

  .map-tooltip {
    position: absolute;
    bottom: 20px;
    left: 20px;
    background: var(--surface);
    border: 1px solid var(--border);
    padding: 0.75rem 1rem;
    border-radius: 12px;
    box-shadow: 0 8px 24px rgba(0,0,0,0.4);
    pointer-events: none;
    display: flex;
    flex-direction: column;
    gap: 0.2rem;
    z-index: 100;
    animation: slideUp 0.2s ease-out;
  }

  .map-tooltip strong {
    font-size: 0.85rem;
    color: var(--text);
    text-transform: capitalize;
  }

  .map-tooltip span {
    font-size: 0.75rem;
    color: var(--accent);
    font-weight: 600;
  }

  .legend h4 {
    font-size: 0.7rem;
    color: var(--text-dim);
    margin-bottom: 1rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
  }

  .legend-items {
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  .legend-items.scrollable {
    max-height: 350px;
    overflow-y: auto;
    padding-right: 5px;
  }

  .legend-items.scrollable::-webkit-scrollbar {
    width: 4px;
  }

  .legend-items.scrollable::-webkit-scrollbar-thumb {
    background: var(--border);
    border-radius: 4px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    font-size: 0.7rem;
    padding: 0.4rem 0.6rem;
    border-radius: 8px;
    transition: all 0.2s;
    cursor: pointer;
    border: 1px solid transparent;
  }

  .legend-item:hover, .legend-item.active {
    background: var(--surface-hover);
    border-color: var(--border);
  }

  .dot {
    width: 10px;
    height: 10px;
    border-radius: 3px;
  }

  .name {
    flex: 1;
    color: var(--text-dim);
    text-transform: capitalize;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .val {
    font-weight: 600;
    color: var(--text);
    font-variant-numeric: tabular-nums;
  }

  @keyframes slideUp {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }

  @media (max-width: 768px) {
    .ecuador-map-container {
      grid-template-columns: 1fr;
    }
  }
</style>
