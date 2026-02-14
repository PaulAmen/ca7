# UNESUM — Dashboard de Asistencia Estudiantil

Dashboard interactivo construido con **Svelte 5 + Vite** para visualizar datos de asistencia estudiantil de la UNESUM.

## Características

- KPIs de resumen (registros, tiempo promedio, docentes, máx. permanencia)
- Gráficos por nivel, paralelo y distribución de tiempo
- Mapa de calor Nivel × Paralelo
- Ranking de docentes registradores
- Procedencia geográfica de estudiantes
- Tabla de detalle con búsqueda, ordenamiento y paginación
- Filtros por fecha y semestre
- Diseño responsive

## Desarrollo local

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

Los archivos de producción se generan en `dist/`.

## Deploy a GitHub Pages

### Opción 1: GitHub Actions (recomendado)

1. Crea un repositorio en GitHub
2. Sube el proyecto:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/unesum-dashboard.git
   git push -u origin main
   ```
3. En GitHub → Settings → Pages → Source: **GitHub Actions**
4. El workflow se ejecutará automáticamente y desplegará el sitio

### Opción 2: Manual con gh-pages

```bash
npm run deploy
```

Tu dashboard estará en: `https://TU_USUARIO.github.io/unesum-dashboard/`

## Datos

El archivo `static/data.json` contiene los datos de asistencia procesados. Para actualizar datos, reemplaza este archivo y haz push.

## Stack

- Svelte 5
- Vite 6
- CSS puro (sin frameworks de UI)
- GitHub Pages
