<!-- Load Plotly from CDN -->
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<h2>Interactive Sector Leadership Heatmap</h2>
<p>
  This heatmap shows, for each sector, how strongly each stock (rows) appears to <b>lead</b> others (columns)
  based on our cross-correlation + Granger causality analysis.
</p>

<label for="sectorSelect"><b>Select sector:</b></label>
<select id="sectorSelect"></select>

<div id="leadership-heatmap" style="width:100%;max-width:900px;height:600px;"></div>

<script>
// Adjust this URL if needed (depends on where you saved the JSON file)
const DATA_URL = "{{ '/assets/leadership_heatmap.json' | relative_url }}";

let heatmapData = null;

function populateSectorSelect(sectors) {
  const sel = document.getElementById('sectorSelect');
  sel.innerHTML = '';
  sectors.forEach(sec => {
    const opt = document.createElement('option');
    opt.value = sec;
    opt.textContent = sec;
    sel.appendChild(opt);
  });
}

function plotSector(sector) {
  if (!heatmapData || !heatmapData[sector]) return;

  const data = heatmapData[sector];
  const leaders = data.leaders;
  const followers = data.followers;
  const z = data.z;

  const trace = {
    z: z,
    x: followers,
    y: leaders,
    type: 'heatmap',
    hoverongaps: false,
    colorbar: { title: 'Correlation' }
  };

  const layout = {
    title: 'Leader → Follower Correlations in ' + sector,
    xaxis: {
      title: 'Follower',
      tickangle: -45,
      automargin: true
    },
    yaxis: {
      title: 'Leader',
      automargin: true
    },
    margin: { t: 50, l: 100, r: 20, b: 120 }
  };

  Plotly.newPlot('leadership-heatmap', [trace], layout);
}

// Load JSON, then initialize dropdown and default plot
fetch(DATA_URL)
  .then(resp => resp.json())
  .then(json => {
    heatmapData = json;
    const sectors = Object.keys(heatmapData).sort();
    if (sectors.length === 0) {
      console.warn("No sectors in leadership_heatmap.json");
      return;
    }
    populateSectorSelect(sectors);
    plotSector(sectors[0]);

    document.getElementById('sectorSelect').addEventListener('change', (e) => {
      plotSector(e.target.value);
    });
  })
  .catch(err => {
    console.error("Failed to load leadership heatmap data:", err);
  });
</script>
