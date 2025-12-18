---
toc: false
---

<div class="portaljs-banner">
  <div class="portaljs-banner-content">
    <div class="portaljs-banner-text">
      <p class="portaljs-banner-title">Create beautiful data portals with PortalJS 🌀</p>
    </div>
  </div>
  <a href="https://www.portaljs.com/" target="_blank" class="portaljs-banner-cta">
    Get started free
  </a>
</div>

```js
const releases = FileAttachment("data/nvidia_releases.csv").csv({typed: true});
const benchmarks = FileAttachment("data/gpu_benchmarks.csv").csv({typed: true});
```

```js
const releaseData = (await releases)
  .map(d => ({
    year: +d.release_year,
    count: +d.release
  }))
  .sort((a, b) => a.year - b.year);

const nvidiaPerf = (await benchmarks)
  .filter(d => d.brand === "NVIDIA" && d.testDate && d.G3Dmark)
  .map(d => ({
    gpuName: d.gpuName,
    year: +d.testDate,
    g3dmark: +d.G3Dmark,
    price: +d.price || 0,
    category: d.category || "Unknown"
  }))
  .filter(d => d.year >= 2012 && d.year <= 2024);

const perfByYear = d3.rollups(
  nvidiaPerf,
  v => ({
    avgScore: d3.mean(v, d => d.g3dmark),
    maxScore: d3.max(v, d => d.g3dmark),
    count: v.length,
    topGpu: v.sort((a, b) => b.g3dmark - a.g3dmark)[0]?.gpuName
  }),
  d => d.year
).map(([year, stats]) => ({year, ...stats})).sort((a, b) => a.year - b.year);

const cumulativeData = releaseData.map((d, i, arr) => ({
  year: d.year,
  count: d.count,
  cumulative: arr.slice(0, i + 1).reduce((sum, x) => sum + x.count, 0)
}));

const totalReleases = releaseData.reduce((sum, d) => sum + d.count, 0);
const peakYear = releaseData.reduce((max, d) => d.count > max.count ? d : max, {count: 0});
const years = releaseData.map(d => d.year);
const minYear = d3.min(years);
const maxYear = d3.max(years);
const avgPerYear = (totalReleases / releaseData.length).toFixed(1);

const topNvidiaGpus = nvidiaPerf.sort((a, b) => b.g3dmark - a.g3dmark).slice(0, 10);

const performanceGrowth = perfByYear.slice(1).map((d, i) => ({
  year: d.year,
  growth: ((d.avgScore - perfByYear[i].avgScore) / perfByYear[i].avgScore * 100),
  avgScore: d.avgScore
}));
```

<div class="hero">
  <h1>NVIDIA Historical Releases</h1>
  <p>GPU release history and performance evolution from ${minYear} to ${maxYear}</p>
</div>

```js
display(html`<div class="dashboard-layout">
  <div class="sidebar">
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-card-header">
          <span class="stat-card-icon"><svg viewBox="0 0 24 24"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg></span>
          <span class="stat-card-label">Total Releases</span>
        </div>
        <div class="stat-card-content">
          <div class="stat-card-value nvidia">${totalReleases}</div>
          <div class="stat-card-subvalue">${minYear}-${maxYear}</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-card-header">
          <span class="stat-card-icon"><svg viewBox="0 0 24 24"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"></polyline><polyline points="17 6 23 6 23 12"></polyline></svg></span>
          <span class="stat-card-label">Peak Year</span>
        </div>
        <div class="stat-card-content">
          <div class="stat-card-value">${peakYear.year}</div>
          <div class="stat-card-subvalue">${peakYear.count} GPUs released</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-card-header">
          <span class="stat-card-icon"><svg viewBox="0 0 24 24"><path d="M12 20V10"></path><path d="M18 20V4"></path><path d="M6 20v-4"></path></svg></span>
          <span class="stat-card-label">Avg Per Year</span>
        </div>
        <div class="stat-card-content">
          <div class="stat-card-value">${avgPerYear}</div>
          <div class="stat-card-subvalue">GPUs per year</div>
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-card-header">
          <span class="stat-card-icon"><svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg></span>
          <span class="stat-card-label">Years Span</span>
        </div>
        <div class="stat-card-content">
          <div class="stat-card-value">${maxYear - minYear}</div>
          <div class="stat-card-subvalue">Years of data</div>
        </div>
      </div>
    </div>

    <div class="insights">
      <div class="insights-header">
        <span class="insights-icon"><svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="16" x2="12" y2="12"></line><line x1="12" y1="8" x2="12.01" y2="8"></line></svg></span>
        <h4 class="insights-title">Key Insights</h4>
      </div>
      <ul class="insights-list">
        <li>2013 saw most releases (25 GPUs)</li>
        <li>Release frequency declined after 2016</li>
        <li>Focus shifted to fewer, powerful cards</li>
        <li>RTX series launched in 2018</li>
      </ul>
    </div>

    <div class="data-sources">
      <strong>Sources:</strong> <a href="https://www.nvidia.com/" target="_blank">NVIDIA</a>, <a href="https://www.techpowerup.com/gpu-specs/" target="_blank">TechPowerUp</a>
    </div>
  </div>

  <div class="main-content">
    <div class="chart-container chart-large">
      <h3>GPU Releases by Year</h3>
      ${resize((width) => {
        const isMobile = width < 640;
        return Plot.plot({
          width,
          height: isMobile ? 280 : 320,
          marginLeft: isMobile ? 45 : 50,
          marginRight: isMobile ? 20 : 30,
          marginBottom: isMobile ? 50 : 45,
          style: { background: "transparent" },
          x: { label: "Year →", tickFormat: d => String(d), ticks: isMobile ? 6 : 11 },
          y: { label: "↑ Number of Releases", grid: true },
          marks: [
            Plot.barY(releaseData, {
              x: "year", y: "count", fill: "#76b900", rx: 4, tip: true
            }),
            Plot.text(releaseData, {
              x: "year", y: "count",
              text: d => d.count,
              dy: -8, fontSize: 11, fontWeight: "600", fill: "#76b900"
            }),
            Plot.ruleY([0])
          ]
        });
      })}
    </div>

    <div class="chart-container chart-large">
      <h3>Cumulative GPU Releases Over Time</h3>
      ${resize((width) => {
        const isMobile = width < 640;
        return Plot.plot({
          width,
          height: isMobile ? 260 : 300,
          marginLeft: isMobile ? 50 : 55,
          marginRight: isMobile ? 20 : 30,
          marginBottom: isMobile ? 50 : 45,
          style: { background: "transparent" },
          x: { label: "Year →", tickFormat: d => String(d), ticks: isMobile ? 6 : 11 },
          y: { label: "↑ Total GPUs Released", grid: true },
          marks: [
            Plot.areaY(cumulativeData, {
              x: "year", y: "cumulative", fill: "#76b900", fillOpacity: 0.3, curve: "monotone-x"
            }),
            Plot.line(cumulativeData, {
              x: "year", y: "cumulative", stroke: "#76b900", strokeWidth: 3, curve: "monotone-x"
            }),
            Plot.dot(cumulativeData, {
              x: "year", y: "cumulative", fill: "#76b900", r: 5, stroke: "white", strokeWidth: 2,
              tip: true, title: d => String(d.year) + ": " + d.cumulative + " total\nThis year: " + d.count
            }),
            Plot.ruleY([0])
          ]
        });
      })}
    </div>

    <div class="chart-container chart-large">
      <h3>Average GPU Performance by Year (G3Dmark)</h3>
      ${resize((width) => {
        const isMobile = width < 640;
        return Plot.plot({
          width,
          height: isMobile ? 280 : 320,
          marginLeft: isMobile ? 60 : 70,
          marginRight: isMobile ? 20 : 30,
          marginBottom: isMobile ? 50 : 45,
          style: { background: "transparent" },
          x: { label: "Year →", tickFormat: d => String(d), ticks: isMobile ? 5 : 8 },
          y: { label: "↑ Avg G3Dmark Score", grid: true },
          marks: [
            Plot.areaY(perfByYear, {
              x: "year", y: "avgScore", fill: "#76b900", fillOpacity: 0.2, curve: "monotone-x"
            }),
            Plot.line(perfByYear, {
              x: "year", y: "avgScore", stroke: "#76b900", strokeWidth: 3, curve: "monotone-x"
            }),
            Plot.dot(perfByYear, {
              x: "year", y: "avgScore", fill: "#76b900", r: 6, stroke: "white", strokeWidth: 2,
              tip: true, title: d => String(d.year) + "\nAvg Score: " + Math.round(d.avgScore).toLocaleString() + "\nTop GPU: " + d.topGpu
            }),
            Plot.ruleY([0])
          ]
        });
      })}
    </div>

    <div class="chart-container chart-large">
      <h3>Year-over-Year Performance Growth</h3>
      ${resize((width) => {
        const isMobile = width < 640;
        const maxAbs = Math.max(Math.abs(d3.min(performanceGrowth, d => d.growth)), Math.abs(d3.max(performanceGrowth, d => d.growth)));
        const symmetricDomain = [-maxAbs * 1.2, maxAbs * 1.2];
        return Plot.plot({
          width,
          height: isMobile ? 260 : 300,
          marginLeft: isMobile ? 50 : 60,
          marginRight: isMobile ? 20 : 30,
          marginBottom: isMobile ? 50 : 45,
          style: { background: "transparent" },
          x: { label: "Year →", tickFormat: d => String(d), ticks: isMobile ? 5 : 8 },
          y: { label: "↑ Growth (%)", grid: true, domain: symmetricDomain },
          marks: [
            Plot.ruleY([0], { stroke: "#1C1917", strokeWidth: 2 }),
            Plot.barY(performanceGrowth, {
              x: "year", y: "growth",
              fill: d => d.growth >= 0 ? "#009966" : "#dc2626",
              rx: 4, tip: true
            }),
            Plot.text(performanceGrowth, {
              x: "year", y: "growth",
              text: d => d.growth.toFixed(0) + "%",
              dy: d => d.growth >= 0 ? -10 : 14,
              fontSize: 10,
              fill: d => d.growth >= 0 ? "#009966" : "#dc2626",
              fontWeight: "600"
            })
          ]
        });
      })}
      <div style="display: flex; gap: 1.5rem; justify-content: center; margin-top: 0.5rem; font-size: 11px;">
        <span><span style="display: inline-block; width: 12px; height: 12px; background: #009966; border-radius: 2px; margin-right: 4px;"></span>Growth</span>
        <span><span style="display: inline-block; width: 12px; height: 12px; background: #dc2626; border-radius: 2px; margin-right: 4px;"></span>Decline</span>
      </div>
    </div>

    <div class="chart-container chart-large">
      <h3>Top 10 NVIDIA GPUs by Performance</h3>
      ${resize((width) => {
        const isMobile = width < 640;
        return Plot.plot({
          width,
          height: isMobile ? 340 : 380,
          marginLeft: isMobile ? 150 : 180,
          marginRight: 80,
          style: { background: "transparent" },
          x: { label: "G3Dmark Score →", grid: true },
          y: { label: null },
          marks: [
            Plot.barX(topNvidiaGpus, {
              x: "g3dmark", y: "gpuName", fill: "#76b900", sort: {y: "-x"}, rx: 4, tip: true
            }),
            Plot.text(topNvidiaGpus, {
              x: "g3dmark", y: "gpuName",
              text: d => d.g3dmark.toLocaleString() + " (" + String(d.year) + ")",
              dx: 5, textAnchor: "start", fontSize: 10, fill: "#79716B"
            }),
            Plot.ruleX([0])
          ]
        });
      })}
    </div>

    <div class="chart-grid">
      <div class="chart-container">
        <h3>Release Rankings (Most to Least)</h3>
        ${resize((width) => {
          const isMobile = width < 640;
          const sorted = [...releaseData].sort((a, b) => b.count - a.count).map(d => ({...d, yearStr: String(d.year)}));
          return Plot.plot({
            width,
            height: isMobile ? 340 : 380,
            marginLeft: isMobile ? 50 : 60,
            marginRight: 50,
            style: { background: "transparent" },
            x: { label: "GPUs Released →", grid: true },
            y: { label: null, domain: sorted.map(d => d.yearStr) },
            marks: [
              Plot.barX(sorted, {
                x: "count", y: "yearStr", fill: "#76b900", rx: 4, tip: true
              }),
              Plot.text(sorted, {
                x: "count", y: "yearStr",
                text: d => d.count,
                dx: 5, textAnchor: "start", fontSize: 10, fill: "#79716B"
              }),
              Plot.ruleX([0])
            ]
          });
        })}
      </div>

      <div class="chart-container">
        <h3>Performance Milestones</h3>
        ${resize((width) => {
          const isMobile = width < 640;
          const milestones = perfByYear.filter(d => d.avgScore > 5000).sort((a, b) => b.avgScore - a.avgScore).map(d => ({...d, yearStr: String(d.year)}));
          return Plot.plot({
            width,
            height: isMobile ? 340 : 380,
            marginLeft: isMobile ? 50 : 60,
            marginRight: 70,
            style: { background: "transparent" },
            x: { label: "Avg G3Dmark Score →", grid: true },
            y: { label: null, domain: milestones.map(d => d.yearStr) },
            marks: [
              Plot.barX(milestones, {
                x: "avgScore", y: "yearStr", fill: "#76b900", rx: 4, tip: true
              }),
              Plot.text(milestones, {
                x: "avgScore", y: "yearStr",
                text: d => Math.round(d.avgScore).toLocaleString(),
                dx: 5, textAnchor: "start", fontSize: 10, fill: "#79716B"
              }),
              Plot.ruleX([0])
            ]
          });
        })}
      </div>
    </div>
  </div>
</div>

<div class="dashboard-footer">
  <span>Built with <a href="https://www.portaljs.com/">PortalJS</a> and Observable Framework</span>
  <span>Sources: <a href="https://www.nvidia.com/" target="_blank">NVIDIA</a>, <a href="https://www.techpowerup.com/gpu-specs/" target="_blank">TechPowerUp</a></span>
</div>`)
```
