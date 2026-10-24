const stocks = [
  {
    symbol: 'AAPL',
    company: 'Apple Inc.',
    sector: 'Technology',
    price: 214.42,
    change: 2.35,
    marketCap: 3220,
    volume: 4820000,
    peRatio: 31.6,
    risk: 1.2,
    growth: 18.4,
    dividendYield: 0.5,
  },
  {
    symbol: 'MSFT',
    company: 'Microsoft Corp.',
    sector: 'Technology',
    price: 421.08,
    change: 1.82,
    marketCap: 3110,
    volume: 3280000,
    peRatio: 34.5,
    risk: 1.1,
    growth: 16.9,
    dividendYield: 0.8,
  },
  {
    symbol: 'NVDA',
    company: 'NVIDIA Corp.',
    sector: 'Semiconductors',
    price: 128.7,
    change: 4.18,
    marketCap: 3150,
    volume: 7640000,
    peRatio: 52.2,
    risk: 2.3,
    growth: 31.2,
    dividendYield: 0.1,
  },
  {
    symbol: 'AMZN',
    company: 'Amazon.com Inc.',
    sector: 'Consumer',
    price: 187.94,
    change: 2.68,
    marketCap: 1980,
    volume: 4050000,
    peRatio: 38.8,
    risk: 1.6,
    growth: 24.1,
    dividendYield: 0,
  },
  {
    symbol: 'GOOGL',
    company: 'Alphabet Inc.',
    sector: 'Communication',
    price: 176.16,
    change: 1.49,
    marketCap: 2165,
    volume: 2540000,
    peRatio: 26.8,
    risk: 1.3,
    growth: 14.3,
    dividendYield: 0.4,
  },
  {
    symbol: 'TSLA',
    company: 'Tesla Inc.',
    sector: 'Automotive',
    price: 212.4,
    change: 3.51,
    marketCap: 680,
    volume: 8940000,
    peRatio: 48.7,
    risk: 3.1,
    growth: 27.8,
    dividendYield: 0,
  },
  {
    symbol: 'XOM',
    company: 'Exxon Mobil Corp.',
    sector: 'Energy',
    price: 112.83,
    change: 0.97,
    marketCap: 493,
    volume: 2260000,
    peRatio: 13.5,
    risk: 0.9,
    growth: 9.4,
    dividendYield: 3.1,
  },
  {
    symbol: 'JPM',
    company: 'JPMorgan Chase & Co.',
    sector: 'Finance',
    price: 198.12,
    change: 1.23,
    marketCap: 724,
    volume: 1890000,
    peRatio: 12.9,
    risk: 0.8,
    growth: 8.7,
    dividendYield: 2.3,
  },
  {
    symbol: 'WMT',
    company: 'Walmart Inc.',
    sector: 'Consumer',
    price: 64.6,
    change: 0.76,
    marketCap: 546,
    volume: 3410000,
    peRatio: 35.4,
    risk: 0.7,
    growth: 7.2,
    dividendYield: 1.4,
  },
  {
    symbol: 'META',
    company: 'Meta Platforms Inc.',
    sector: 'Communication',
    price: 518.81,
    change: 2.92,
    marketCap: 1315,
    volume: 1770000,
    peRatio: 28.4,
    risk: 1.8,
    growth: 22.7,
    dividendYield: 0.3,
  },
];

const sortRequirementSelect = document.getElementById('sortRequirement');
const sortDirectionSelect = document.getElementById('sortDirection');
const searchInput = document.getElementById('searchInput');
const sectorFilter = document.getElementById('sectorFilter');
const summaryGrid = document.getElementById('summaryGrid');
const tableBody = document.getElementById('stockTableBody');
const resultCount = document.getElementById('resultCount');

function formatMarketCap(value) {
  return `$${value.toFixed(1)}B`;
}

function formatVolume(value) {
  if (value >= 1_000_000) {
    return `${(value / 1_000_000).toFixed(2)}M`;
  }
  return `${(value / 1_000).toFixed(2)}K`;
}

function formatCurrency(value) {
  return `$${value.toFixed(2)}`;
}

function calculatePerformance(stock) {
  return (
    stock.change * 1.8 +
    stock.growth * 1.4 -
    stock.risk * 9 +
    (stock.marketCap / 100) * 0.12 +
    stock.dividendYield * 4 -
    stock.peRatio * 0.08
  );
}

function getSortedStocks() {
  const requirement = sortRequirementSelect.value;
  const direction = sortDirectionSelect.value;
  const searchTerm = searchInput.value.trim().toLowerCase();
  const selectedSector = sectorFilter.value;

  let filteredStocks = stocks.filter((stock) => {
    const matchesSearch =
      stock.symbol.toLowerCase().includes(searchTerm) ||
      stock.company.toLowerCase().includes(searchTerm);

    const matchesSector = selectedSector === 'all' || stock.sector === selectedSector;
    return matchesSearch && matchesSector;
  });

  filteredStocks = filteredStocks.map((stock) => ({
    ...stock,
    performanceScore: calculatePerformance(stock),
  }));

  const sorterMap = {
    performanceScore: (a, b) => b.performanceScore - a.performanceScore,
    priceChange: (a, b) => b.change - a.change,
    volume: (a, b) => b.volume - a.volume,
    marketCap: (a, b) => b.marketCap - a.marketCap,
    peRatio: (a, b) => a.peRatio - b.peRatio,
    growth: (a, b) => b.growth - a.growth,
    riskLevel: (a, b) => a.risk - b.risk,
  };

  filteredStocks.sort(sorterMap[requirement]);

  if (direction === 'asc') {
    filteredStocks.reverse();
  }

  return filteredStocks;
}

function renderSummary(items) {
  if (!items.length) {
    summaryGrid.innerHTML = '<div class="summary-card"><div class="label">No data</div><div class="value">No matching stocks</div></div>';
    return;
  }

  const bestStock = [...items].sort((a, b) => b.performanceScore - a.performanceScore)[0];
  const mostActive = [...items].sort((a, b) => b.volume - a.volume)[0];
  const averageChange = items.reduce((sum, item) => sum + item.change, 0) / items.length;
  const biggestCap = [...items].sort((a, b) => b.marketCap - a.marketCap)[0];

  summaryGrid.innerHTML = `
    <div class="summary-card">
      <div class="label">Best performer</div>
      <div class="value">${bestStock.symbol}</div>
    </div>
    <div class="summary-card">
      <div class="label">Most active</div>
      <div class="value">${mostActive.symbol}</div>
    </div>
    <div class="summary-card">
      <div class="label">Avg. change</div>
      <div class="value ${averageChange >= 0 ? 'positive' : 'negative'}">${averageChange.toFixed(2)}%</div>
    </div>
    <div class="summary-card">
      <div class="label">Largest cap</div>
      <div class="value">${biggestCap.symbol}</div>
    </div>
  `;
}

function renderTable() {
  const items = getSortedStocks();
  resultCount.textContent = `${items.length} stocks shown`;
  renderSummary(items);

  if (!items.length) {
    tableBody.innerHTML = `
      <tr>
        <td colspan="11">No stocks match the current filters.</td>
      </tr>
    `;
    return;
  }

  tableBody.innerHTML = items
    .map((stock) => {
      const changeClass = stock.change >= 0 ? 'positive' : 'negative';
      const performanceBar = Math.min(100, Math.max(0, (stock.performanceScore + 30) * 1.8));

      return `
        <tr>
          <td class="symbol">${stock.symbol}</td>
          <td class="company-name">${stock.company}</td>
          <td>${stock.sector}</td>
          <td>${formatCurrency(stock.price)}</td>
          <td class="${changeClass}">${stock.change.toFixed(2)}%</td>
          <td>${formatMarketCap(stock.marketCap)}</td>
          <td>${formatVolume(stock.volume)}</td>
          <td>${stock.peRatio.toFixed(1)}</td>
          <td>${stock.risk.toFixed(1)}</td>
          <td>${stock.growth.toFixed(1)}%</td>
          <td>
            <div class="performance-pill">${stock.performanceScore.toFixed(1)}</div>
            <div class="score-bar"><div class="score-fill" style="width:${performanceBar}%"></div></div>
          </td>
        </tr>
      `;
    })
    .join('');
}

function populateSectorFilter() {
  const sectors = [...new Set(stocks.map((stock) => stock.sector))].sort();
  sectorFilter.innerHTML = '<option value="all">All sectors</option>' +
    sectors.map((sector) => `<option value="${sector}">${sector}</option>`).join('');
}

sortRequirementSelect.addEventListener('change', renderTable);
sortDirectionSelect.addEventListener('change', renderTable);
searchInput.addEventListener('input', renderTable);
sectorFilter.addEventListener('change', renderTable);

populateSectorFilter();
renderTable();

