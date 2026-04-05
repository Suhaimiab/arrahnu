# Ar-Rahnu Pro Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file web app for calculating, tracking, and simulating Bank Rakyat Ar-Rahnu pawn broking costs.

**Architecture:** Single `index.html` file using Vue.js 3 Options API via CDN for reactivity, Tailwind CSS via CDN for styling. All state in localStorage. Gold price fetched from CORS-friendly APIs with fallback to manual entry. Bilingual EN/BM via reactive i18n object.

**Tech Stack:** Vue.js 3 CDN, Tailwind CSS CDN, localStorage, vanilla JS fetch for gold price API.

**Spec:** `docs/superpowers/specs/2026-04-05-arrahnu-pro-design.md`

---

## File Structure

Single file — all code lives in `index.html` at project root:

```
arrahnu/
  index.html          # The entire app (HTML + CSS + JS)
  docs/               # Existing docs folder
```

The `index.html` is organized internally as:
1. `<head>` — CDN links (Vue 3, Tailwind), custom styles
2. `<body>` — Vue app template (header, tabs, 3 module sections)
3. `<script>` — Vue app definition with:
   - `i18n` translations object
   - `computed` for all calculations
   - `methods` for actions (save, export, import, fetch price)
   - `watch` for localStorage persistence
   - `mounted` for initialization (load data, fetch price)

---

## Task 1: Scaffold — Base HTML, Vue, Tailwind, Header, Tabs

**Files:**
- Create: `index.html`

**What this builds:** The app shell — dark themed page with header (title, gold price badge, language toggle), 3-tab navigation, and placeholder content for each tab. No functionality yet, just the skeleton.

- [ ] **Step 1: Create `index.html` with base structure**

Create the file with:
- `<!DOCTYPE html>` with lang attribute
- `<head>`: meta charset, viewport, title "Ar-Rahnu Pro", Tailwind CDN (`<script src="https://cdn.tailwindcss.com"></script>`), Vue 3 CDN (`<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>`), custom Tailwind config for gold color theme, custom CSS for dark background
- `<body>`: single `<div id="app">` containing:
  - Header bar: app title "Ar-Rahnu Pro", gold price badge (placeholder "RM ---/gm"), language toggle button (EN/BM)
  - Tab navigation: 3 buttons — Calculator/Kalkulator, Tracker/Penjejak, Simulator/Simulasi
  - 3 content sections (shown/hidden by `v-if` on `activeTab`), each with placeholder text
- `<script>`: Vue app with:
  - `data()` returning: `activeTab: 'calculator'`, `lang: 'en'`, `goldPrice: null`
  - `i18n` object with `en` and `bm` keys for: app title, tab names, gold price label
  - Computed `t` that returns `i18n[this.lang]`
  - Method `toggleLang()` switching between 'en' and 'bm'
  - `mounted()`: load `arrahnu_language` and `arrahnu_activeTab` from localStorage
  - `watch` on `lang` and `activeTab` to persist to localStorage

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ar-Rahnu Pro</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            gold: { 50: '#fdf8e8', 100: '#faf0c8', 200: '#f5e08e', 300: '#eecb4d', 400: '#e4b422', 500: '#d4a843', 600: '#a67c1a', 700: '#7a5c15', 800: '#4e3b10', 900: '#2a200a' },
            dark: { 50: '#e8e8ee', 100: '#c4c4d4', 200: '#9d9db7', 300: '#76769a', 400: '#585885', 500: '#3a3a70', 600: '#2d2d5e', 700: '#22224a', 800: '#1a1a2e', 900: '#0f0f1a' }
          }
        }
      }
    }
  </script>
  <style>
    body { background-color: #1a1a2e; }
    [v-cloak] { display: none; }
  </style>
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body class="min-h-screen text-gray-100">
  <div id="app" v-cloak>
    <!-- App content will be built in subsequent tasks -->
  </div>
  <script>
    // Vue app will be built in subsequent tasks
  </script>
</body>
</html>
```

- [ ] **Step 2: Build the full header with gold price badge and language toggle**

Inside `<div id="app">`, add the header:

```html
<!-- Header -->
<header class="bg-dark-700 border-b border-gold-500/30 px-4 py-3">
  <div class="max-w-6xl mx-auto flex items-center justify-between flex-wrap gap-2">
    <h1 class="text-xl font-bold text-gold-400">Ar-Rahnu Pro</h1>
    <div class="flex items-center gap-4">
      <!-- Gold Price Badge -->
      <div class="flex items-center gap-2 bg-dark-800 rounded-lg px-3 py-1.5 text-sm">
        <span class="text-gold-300">{{ t.goldPrice }}:</span>
        <span v-if="goldPrice" class="font-semibold text-gold-400">RM {{ formatNumber(goldPrice) }}/gm</span>
        <span v-else class="text-gray-500">---</span>
        <span v-if="goldPriceUpdated" class="text-xs text-gray-500">({{ goldPriceUpdated }})</span>
        <span v-if="goldPriceStale" class="text-xs text-yellow-500">{{ t.stale }}</span>
      </div>
      <!-- Language Toggle -->
      <button @click="toggleLang" class="bg-dark-600 hover:bg-dark-500 text-gold-300 px-3 py-1.5 rounded-lg text-sm font-medium transition">
        {{ lang === 'en' ? 'BM' : 'EN' }}
      </button>
    </div>
  </div>
</header>
```

- [ ] **Step 3: Build tab navigation and content sections**

Below the header:

```html
<!-- Tab Navigation -->
<nav class="bg-dark-700/50 border-b border-dark-600">
  <div class="max-w-6xl mx-auto flex">
    <button v-for="tab in tabs" :key="tab.id"
      @click="activeTab = tab.id"
      :class="activeTab === tab.id ? 'border-gold-500 text-gold-400 bg-dark-800/50' : 'border-transparent text-gray-400 hover:text-gray-200'"
      class="px-6 py-3 text-sm font-medium border-b-2 transition">
      {{ t.tabs[tab.id] }}
    </button>
  </div>
</nav>

<!-- Content -->
<main class="max-w-6xl mx-auto p-4">
  <!-- Calculator -->
  <div v-if="activeTab === 'calculator'">
    <p class="text-gray-500">{{ t.tabs.calculator }} — {{ t.comingSoon }}</p>
  </div>
  <!-- Tracker -->
  <div v-if="activeTab === 'tracker'">
    <p class="text-gray-500">{{ t.tabs.tracker }} — {{ t.comingSoon }}</p>
  </div>
  <!-- Simulator -->
  <div v-if="activeTab === 'simulator'">
    <p class="text-gray-500">{{ t.tabs.simulator }} — {{ t.comingSoon }}</p>
  </div>
</main>
```

- [ ] **Step 4: Write the Vue app with i18n, data, and persistence**

```javascript
const { createApp } = Vue

const i18n = {
  en: {
    goldPrice: 'Gold Price',
    stale: 'stale',
    comingSoon: 'Coming soon',
    tabs: { calculator: 'Calculator', tracker: 'Tracker', simulator: 'Simulator' }
  },
  bm: {
    goldPrice: 'Harga Emas',
    stale: 'lapuk',
    comingSoon: 'Akan datang',
    tabs: { calculator: 'Kalkulator', tracker: 'Penjejak', simulator: 'Simulasi' }
  }
}

createApp({
  data() {
    return {
      activeTab: 'calculator',
      lang: 'en',
      goldPrice: null,
      goldPriceUpdated: null,
      goldPriceStale: false,
      tabs: [
        { id: 'calculator' },
        { id: 'tracker' },
        { id: 'simulator' }
      ]
    }
  },
  computed: {
    t() { return i18n[this.lang] }
  },
  methods: {
    toggleLang() {
      this.lang = this.lang === 'en' ? 'bm' : 'en'
    },
    formatNumber(num) {
      return Number(num).toLocaleString('en-MY', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
    }
  },
  watch: {
    lang(val) { localStorage.setItem('arrahnu_language', val) },
    activeTab(val) { localStorage.setItem('arrahnu_activeTab', val) }
  },
  mounted() {
    this.lang = localStorage.getItem('arrahnu_language') || 'en'
    this.activeTab = localStorage.getItem('arrahnu_activeTab') || 'calculator'
  }
}).mount('#app')
```

- [ ] **Step 5: Manual test**

Open `index.html` in browser. Verify:
- Dark background with gold-themed header
- Gold price shows "---"
- Language toggle switches all text between EN and BM
- Tab switching works and shows placeholder text
- Reload preserves language and active tab

- [ ] **Step 6: Commit**

```bash
git init
git add index.html
git commit -m "feat: scaffold Ar-Rahnu Pro with header, tabs, i18n, dark theme"
```

---

## Task 2: Gold Price API Integration

**Files:**
- Modify: `index.html` (add fetch logic to Vue app)

**What this builds:** Auto-fetch gold spot price in MYR/gm on app load, cache in localStorage, show stale warning if cached, fall back to manual entry if all fails.

- [ ] **Step 1: Add gold price fetch methods**

Add these methods to the Vue app:

```javascript
async fetchGoldPrice() {
  try {
    // Step 1: Get USD/MYR exchange rate from frankfurter.app (free, no key, CORS-enabled)
    const fxRes = await fetch('https://api.frankfurter.app/latest?from=USD&to=MYR')
    if (!fxRes.ok) throw new Error('FX API failed')
    const fxData = await fxRes.json()
    const usdToMyr = fxData.rates.MYR

    // Step 2: Get gold price in USD per troy ounce
    // Using a proxy-friendly approach with fallback values
    // Gold spot price ~USD 2300/oz as of 2026 (will be overridden by API if available)
    let goldUsdPerOz = null

    try {
      // Try fetching from a CORS-friendly gold price source
      const goldRes = await fetch('https://data-asg.goldprice.org/dbXRates/USD')
      if (goldRes.ok) {
        const goldData = await goldRes.json()
        if (goldData.items && goldData.items[0]) {
          goldUsdPerOz = goldData.items[0].xauPrice
        }
      }
    } catch (e) {
      console.warn('Gold spot API failed, trying alternative...', e)
    }

    if (!goldUsdPerOz) {
      // Alternative: try another free source
      try {
        const altRes = await fetch('https://api.nbp.pl/api/cenyzlota?format=json')
        if (altRes.ok) {
          const altData = await altRes.json()
          // NBP returns price in PLN per gram, convert via USD
          // This is a less accurate fallback
          console.warn('Using NBP fallback — accuracy may vary')
        }
      } catch (e) {
        console.warn('All gold APIs failed', e)
      }
    }

    if (goldUsdPerOz && usdToMyr) {
      // Convert: USD/troy oz -> MYR/gram (1 troy oz = 31.1035 gm)
      const pricePerGram = (goldUsdPerOz * usdToMyr) / 31.1035
      this.goldPrice = parseFloat(pricePerGram.toFixed(2))
      this.goldPriceUpdated = new Date().toLocaleString('en-MY')
      this.goldPriceStale = false
      localStorage.setItem('arrahnu_goldPrice', JSON.stringify({
        price: this.goldPrice,
        updated: this.goldPriceUpdated
      }))
      return
    }
  } catch (e) {
    console.warn('Gold price fetch failed:', e)
  }

  // Fallback: use cached price from localStorage
  this.loadCachedGoldPrice()
},

loadCachedGoldPrice() {
  const cached = localStorage.getItem('arrahnu_goldPrice')
  if (cached) {
    const data = JSON.parse(cached)
    this.goldPrice = parseFloat(data.price)
    this.goldPriceUpdated = data.updated
    this.goldPriceStale = true
  }
  // If no cache either, goldPrice stays null — manual entry mode activates
}
```

- [ ] **Step 2: Add manual gold price override in header**

Update the gold price badge in the header template to include an editable input:

```html
<!-- Gold Price Badge — replace the existing badge -->
<div class="flex items-center gap-2 bg-dark-800 rounded-lg px-3 py-1.5 text-sm">
  <span class="text-gold-300">{{ t.goldPrice }}:</span>
  <span class="text-gold-300">RM</span>
  <input type="number" v-model.number="goldPrice" step="0.01" min="0"
    class="w-24 bg-dark-900 border border-dark-600 rounded px-2 py-0.5 text-gold-400 font-semibold text-right focus:border-gold-500 focus:outline-none"
    :placeholder="t.enterPrice">
  <span class="text-gold-300">/gm</span>
  <span v-if="goldPriceUpdated" class="text-xs text-gray-500">({{ goldPriceUpdated }})</span>
  <span v-if="goldPriceStale" class="text-xs text-yellow-500">{{ t.stale }}</span>
</div>
```

- [ ] **Step 3: Add disclaimer text**

Add i18n entries:

```javascript
// Add to both en and bm i18n objects:
en: {
  enterPrice: 'Enter price',
  disclaimer: 'Note: Bank Rakyat\'s valuation rate may differ from spot price. Use the bank\'s quoted rate for accurate calculations.',
  // ...existing
}
bm: {
  enterPrice: 'Masukkan harga',
  disclaimer: 'Nota: Kadar penilaian Bank Rakyat mungkin berbeza daripada harga spot. Gunakan kadar bank untuk pengiraan tepat.',
  // ...existing
}
```

Add disclaimer below header:

```html
<div class="max-w-6xl mx-auto px-4 py-2">
  <p class="text-xs text-gray-500 italic">{{ t.disclaimer }}</p>
</div>
```

- [ ] **Step 4: Call fetchGoldPrice on mount**

In `mounted()`, add:

```javascript
this.fetchGoldPrice()
```

- [ ] **Step 5: Manual test**

Open `index.html` in browser. Verify:
- Gold price auto-populates (or shows cached/manual entry mode)
- Manual override works — type a value and it updates
- Stale badge appears when using cached price
- Disclaimer text shows below header

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add gold price API integration with cache and manual override"
```

---

## Task 3: Calculator Module

**Files:**
- Modify: `index.html` (replace calculator placeholder with full module)

**What this builds:** The Calculator tab with inputs for gold weight, karat, price, financing %; reactive output showing marhun value, financing, profit breakdown, and cost table.

- [ ] **Step 1: Add calculator data properties**

Add to `data()`:

```javascript
// Calculator
calc: {
  weight: null,
  karat: 24,
  financingPct: 80,
  tenure: 18
},
karats: [
  { value: 24, label: '24K', purity: 1.000 },
  { value: 22, label: '22K', purity: 0.916 },
  { value: 21, label: '21K', purity: 0.875 },
  { value: 20, label: '20K', purity: 0.833 },
  { value: 18, label: '18K', purity: 0.750 },
  { value: 16, label: '16K', purity: 0.667 }
],
tenureOptions: [6, 12, 18],
```

- [ ] **Step 2: Add calculator computed properties**

```javascript
calcPurity() {
  const k = this.karats.find(k => k.value === this.calc.karat)
  return k ? k.purity : 1
},
calcMarhunValue() {
  if (!this.calc.weight || !this.goldPrice) return 0
  return this.calc.weight * this.goldPrice * this.calcPurity
},
calcFinancing() {
  return this.calcMarhunValue * (this.calc.financingPct / 100)
},
calcProfitRate() {
  const v = this.calcMarhunValue
  if (v < 500) return 0.60
  if (v < 5000) return 0.75
  if (v < 10000) return 0.80
  return 0.85
},
calcProfitRateTier() {
  const v = this.calcMarhunValue
  if (v < 500) return '< RM500'
  if (v < 5000) return 'RM500 - RM5,000'
  if (v < 10000) return 'RM5,000 - RM10,000'
  return '> RM10,000'
},
calcMonthlyProfit() {
  return this.calcMarhunValue / 100 * this.calcProfitRate
},
calcTotalProfit() {
  return this.calcMonthlyProfit * this.calc.tenure
},
calcTotalRepayment() {
  return this.calcFinancing + this.calcTotalProfit
},
calcAnnualizedRate() {
  if (!this.calcFinancing) return 0
  return (this.calcTotalProfit / this.calcFinancing) * (12 / this.calc.tenure) * 100
},
```

- [ ] **Step 3: Build calculator template**

Replace the calculator placeholder `<div>` with:

```html
<div v-if="activeTab === 'calculator'" class="space-y-6">
  <!-- Input Card -->
  <div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
    <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.calcTitle }}</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      <!-- Weight -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.weight }} (gm)</label>
        <input type="number" v-model.number="calc.weight" min="0" step="0.01"
          class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none" placeholder="0.00">
      </div>
      <!-- Karat -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.karat }}</label>
        <select v-model.number="calc.karat"
          class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
          <option v-for="k in karats" :value="k.value">{{ k.label }} ({{ (k.purity * 100).toFixed(1) }}%)</option>
        </select>
      </div>
      <!-- Gold Price -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.goldPrice }} (RM/gm)</label>
        <input type="number" v-model.number="goldPrice" min="0" step="0.01"
          class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none" placeholder="0.00">
      </div>
      <!-- Financing % -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.financingPct }}: {{ calc.financingPct }}%</label>
        <input type="range" v-model.number="calc.financingPct" min="50" max="80" step="1"
          class="w-full accent-gold-500">
      </div>
    </div>
    <!-- Tenure Toggle -->
    <div class="mt-4">
      <label class="block text-sm text-gray-400 mb-2">{{ t.tenure }}</label>
      <div class="flex gap-2">
        <button v-for="m in tenureOptions" :key="m" @click="calc.tenure = m"
          :class="calc.tenure === m ? 'bg-gold-500 text-dark-900' : 'bg-dark-600 text-gray-300 hover:bg-dark-500'"
          class="px-4 py-1.5 rounded-lg text-sm font-medium transition">
          {{ m }} {{ t.months }}
        </button>
      </div>
    </div>
  </div>

  <!-- Results Card -->
  <div v-if="calc.weight && goldPrice" class="bg-dark-700 rounded-xl p-6 border border-dark-600">
    <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.results }}</h2>
    <!-- Summary Grid -->
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.marhunValue }}</p>
        <p class="text-lg font-bold text-gold-300">RM {{ formatNumber(calcMarhunValue) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.financing }} ({{ calc.financingPct }}%)</p>
        <p class="text-lg font-bold text-green-400">RM {{ formatNumber(calcFinancing) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.monthlyProfit }}</p>
        <p class="text-lg font-bold text-red-400">RM {{ formatNumber(calcMonthlyProfit) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.annualizedRate }}</p>
        <p class="text-lg font-bold text-yellow-400">{{ calcAnnualizedRate.toFixed(1) }}%</p>
      </div>
    </div>
    <!-- Profit Rate Tier -->
    <div class="mb-4 text-sm">
      <span class="text-gray-400">{{ t.profitRateTier }}:</span>
      <span class="text-gold-400 font-medium ml-2">{{ calcProfitRateTier }} — RM{{ calcProfitRate.toFixed(2) }}/RM100/{{ t.month }}</span>
    </div>
    <!-- Cost Breakdown Table -->
    <div class="overflow-x-auto">
      <table class="w-full text-sm">
        <thead>
          <tr class="text-gray-400 border-b border-dark-600">
            <th class="text-left py-2">{{ t.period }}</th>
            <th class="text-right py-2">{{ t.profitCost }}</th>
            <th class="text-right py-2">{{ t.totalRepayment }}</th>
          </tr>
        </thead>
        <tbody>
          <tr class="border-b border-dark-600/50">
            <td class="py-2">{{ t.monthly }}</td>
            <td class="text-right text-red-400">RM {{ formatNumber(calcMonthlyProfit) }}</td>
            <td class="text-right">-</td>
          </tr>
          <tr class="border-b border-dark-600/50">
            <td class="py-2">6 {{ t.months }}</td>
            <td class="text-right text-red-400">RM {{ formatNumber(calcMonthlyProfit * 6) }}</td>
            <td class="text-right text-gray-300">RM {{ formatNumber(calcFinancing + calcMonthlyProfit * 6) }}</td>
          </tr>
          <tr class="border-b border-dark-600/50">
            <td class="py-2">12 {{ t.months }}</td>
            <td class="text-right text-red-400">RM {{ formatNumber(calcMonthlyProfit * 12) }}</td>
            <td class="text-right text-gray-300">RM {{ formatNumber(calcFinancing + calcMonthlyProfit * 12) }}</td>
          </tr>
          <tr class="font-semibold" :class="calc.tenure === 18 ? 'text-gold-400' : ''">
            <td class="py-2">18 {{ t.months }}</td>
            <td class="text-right text-red-400">RM {{ formatNumber(calcMonthlyProfit * 18) }}</td>
            <td class="text-right">RM {{ formatNumber(calcFinancing + calcMonthlyProfit * 18) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- Selected Tenure Summary -->
    <div class="mt-4 bg-gold-500/10 border border-gold-500/30 rounded-lg p-4">
      <div class="flex justify-between items-center">
        <span class="text-gold-300 font-medium">{{ t.totalFor }} {{ calc.tenure }} {{ t.months }}</span>
        <span class="text-xl font-bold text-gold-400">RM {{ formatNumber(calcTotalRepayment) }}</span>
      </div>
      <p class="text-xs text-gray-400 mt-1">{{ t.financing }}: RM {{ formatNumber(calcFinancing) }} + {{ t.profit }}: RM {{ formatNumber(calcTotalProfit) }}</p>
    </div>
  </div>
</div>
```

- [ ] **Step 4: Add calculator i18n strings**

Add to both `en` and `bm` objects in the i18n:

```javascript
// EN
calcTitle: 'Ar-Rahnu Calculator',
weight: 'Gold Weight',
karat: 'Karat',
financingPct: 'Financing',
tenure: 'Tenure',
months: 'months',
month: 'month',
results: 'Results',
marhunValue: 'Marhun Value',
financing: 'Financing',
monthlyProfit: 'Monthly Profit',
annualizedRate: 'Annualized Rate',
profitRateTier: 'Profit Rate Tier',
period: 'Period',
profitCost: 'Profit Cost',
totalRepayment: 'Total Repayment',
monthly: 'Monthly',
totalFor: 'Total for',
profit: 'Profit',
enterPrice: 'Enter price',

// BM
calcTitle: 'Kalkulator Ar-Rahnu',
weight: 'Berat Emas',
karat: 'Karat',
financingPct: 'Pembiayaan',
tenure: 'Tempoh',
months: 'bulan',
month: 'bulan',
results: 'Keputusan',
marhunValue: 'Nilai Marhun',
financing: 'Pembiayaan',
monthlyProfit: 'Keuntungan Bulanan',
annualizedRate: 'Kadar Tahunan',
profitRateTier: 'Tier Kadar Keuntungan',
period: 'Tempoh',
profitCost: 'Kos Keuntungan',
totalRepayment: 'Jumlah Bayaran',
monthly: 'Bulanan',
totalFor: 'Jumlah untuk',
profit: 'Keuntungan',
enterPrice: 'Masukkan harga',
```

- [ ] **Step 5: Manual test**

Open `index.html`. In Calculator tab:
- Enter weight: 200, karat: 24K, verify marhun value calculates correctly
- Adjust financing slider, verify financing amount updates
- Toggle tenure between 6/12/18, verify totals change
- Switch language, verify all labels translate
- Verify profit rate tier shows correct tier based on marhun value
- Test with small value (e.g. 1gm 16K) to verify lower tier rates

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add calculator module with profit rate tiers and cost breakdown"
```

---

## Task 4: Tracker Module — Dashboard & Account List

**Files:**
- Modify: `index.html` (replace tracker placeholder, add account data management)

**What this builds:** The Tracker tab with dashboard summary cards and account list. Accounts are stored in localStorage and displayed as cards with status, marhun summary, financing, and payment countdown.

- [ ] **Step 1: Add tracker data properties**

Add to `data()`:

```javascript
// Tracker
accounts: [],
showAccountForm: false,
editingAccountId: null,
newAccount: {
  accountNo: '',
  financingAccountNo: '',
  branch: '',
  status: 'active',
  createdDate: '',
  maturityDate: '',
  items: [{ description: '', karat: 24, weightGm: null, valueMYR: null }],
  financingAmount: null,
  rolloverFrom: null,
  rolloverTo: null,
  notes: ''
},
```

- [ ] **Step 2: Add tracker computed properties**

```javascript
activeAccounts() {
  return this.accounts.filter(a => a.status === 'active')
},
dashboardTotals() {
  const active = this.activeAccounts
  const totalMarhun = active.reduce((sum, a) => sum + this.getAccountMarhunValue(a), 0)
  const totalFinancing = active.reduce((sum, a) => sum + (a.financingAmount || 0), 0)
  const totalMonthlyProfit = active.reduce((sum, a) => sum + this.getAccountMonthlyProfit(a), 0)
  const nextDue = active
    .flatMap(a => (a.paymentSchedule || []).filter(p => !p.paid))
    .map(p => p.dueDate)
    .filter(Boolean)
    .sort()
  return {
    totalMarhun,
    totalFinancing,
    totalMonthlyProfit,
    nextDue: nextDue[0] || null,
    count: active.length
  }
},
```

- [ ] **Step 3: Add tracker helper methods**

```javascript
getAccountMarhunValue(account) {
  return (account.items || []).reduce((sum, item) => sum + (item.valueMYR || 0), 0)
},
getAccountMonthlyProfit(account) {
  const marhunValue = this.getAccountMarhunValue(account)
  const rate = this.getProfitRate(marhunValue)
  return marhunValue / 100 * rate
},
getProfitRate(marhunValue) {
  if (marhunValue < 500) return 0.60
  if (marhunValue < 5000) return 0.75
  if (marhunValue < 10000) return 0.80
  return 0.85
},
getAccountTotalWeight(account) {
  return (account.items || []).reduce((sum, item) => sum + (item.weightGm || 0), 0)
},
getDaysRemaining(dateStr) {
  if (!dateStr) return null
  const diff = new Date(dateStr) - new Date()
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
},
getNextPaymentDue(account) {
  const unpaid = (account.paymentSchedule || []).filter(p => !p.paid).sort((a, b) => a.dueDate.localeCompare(b.dueDate))
  return unpaid[0] || null
},
formatDate(dateStr) {
  if (!dateStr) return '-'
  const d = new Date(dateStr)
  return d.toLocaleDateString('en-MY', { day: '2-digit', month: '2-digit', year: 'numeric' })
},
getStatusColor(status) {
  const colors = { active: 'text-green-400 bg-green-400/10', matured: 'text-yellow-400 bg-yellow-400/10', redeemed: 'text-blue-400 bg-blue-400/10', rolled_over: 'text-purple-400 bg-purple-400/10' }
  return colors[status] || 'text-gray-400 bg-gray-400/10'
},
getStatusLabel(status) {
  const labels = {
    en: { active: 'Active', matured: 'Matured', redeemed: 'Redeemed', rolled_over: 'Rolled Over' },
    bm: { active: 'Aktif', matured: 'Matang', redeemed: 'Ditebus', rolled_over: 'Digadai Semula' }
  }
  return labels[this.lang][status] || status
},
```

- [ ] **Step 4: Build tracker dashboard and account list template**

Replace the tracker placeholder:

```html
<div v-if="activeTab === 'tracker'" class="space-y-6">
  <!-- Dashboard -->
  <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
    <div class="bg-dark-700 rounded-xl p-4 border border-dark-600">
      <p class="text-xs text-gray-400">{{ t.totalMarhun }}</p>
      <p class="text-lg font-bold text-gold-300">RM {{ formatNumber(dashboardTotals.totalMarhun) }}</p>
    </div>
    <div class="bg-dark-700 rounded-xl p-4 border border-dark-600">
      <p class="text-xs text-gray-400">{{ t.totalFinancing }}</p>
      <p class="text-lg font-bold text-green-400">RM {{ formatNumber(dashboardTotals.totalFinancing) }}</p>
    </div>
    <div class="bg-dark-700 rounded-xl p-4 border border-dark-600">
      <p class="text-xs text-gray-400">{{ t.totalMonthlyProfit }}</p>
      <p class="text-lg font-bold text-red-400">RM {{ formatNumber(dashboardTotals.totalMonthlyProfit) }}</p>
    </div>
    <div class="bg-dark-700 rounded-xl p-4 border border-dark-600">
      <p class="text-xs text-gray-400">{{ t.nextPayment }}</p>
      <p class="text-lg font-bold" :class="dashboardTotals.nextDue && getDaysRemaining(dashboardTotals.nextDue) <= 30 ? 'text-red-400' : 'text-gray-300'">
        {{ dashboardTotals.nextDue ? formatDate(dashboardTotals.nextDue) : '-' }}
      </p>
      <p v-if="dashboardTotals.nextDue" class="text-xs text-gray-500">{{ getDaysRemaining(dashboardTotals.nextDue) }} {{ t.daysLeft }}</p>
    </div>
    <div class="bg-dark-700 rounded-xl p-4 border border-dark-600">
      <p class="text-xs text-gray-400">{{ t.activeAccounts }}</p>
      <p class="text-lg font-bold text-gold-400">{{ dashboardTotals.count }}</p>
    </div>
  </div>

  <!-- Add Account Button -->
  <div class="flex justify-end">
    <button @click="openNewAccountForm" class="bg-gold-500 hover:bg-gold-600 text-dark-900 px-4 py-2 rounded-lg text-sm font-medium transition">
      + {{ t.addAccount }}
    </button>
  </div>

  <!-- Account Cards -->
  <div v-if="accounts.length" class="space-y-4">
    <div v-for="account in accounts" :key="account.id"
      class="bg-dark-700 rounded-xl p-5 border border-dark-600 hover:border-gold-500/30 transition">
      <div class="flex items-start justify-between mb-3">
        <div>
          <div class="flex items-center gap-2">
            <h3 class="font-semibold text-gray-100">{{ account.accountNo || t.noAccountNo }}</h3>
            <span :class="getStatusColor(account.status)" class="text-xs px-2 py-0.5 rounded-full font-medium">{{ getStatusLabel(account.status) }}</span>
          </div>
          <p v-if="account.branch" class="text-xs text-gray-500 mt-0.5">{{ account.branch }}</p>
        </div>
        <div class="flex gap-2">
          <button @click="editAccount(account.id)" class="text-gray-400 hover:text-gold-400 text-sm transition">{{ t.edit }}</button>
          <button @click="deleteAccount(account.id)" class="text-gray-400 hover:text-red-400 text-sm transition">{{ t.delete }}</button>
        </div>
      </div>
      <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm">
        <div>
          <p class="text-gray-500">{{ t.marhunValue }}</p>
          <p class="text-gold-300 font-medium">RM {{ formatNumber(getAccountMarhunValue(account)) }}</p>
          <p class="text-xs text-gray-500">{{ getAccountTotalWeight(account).toFixed(2) }}gm</p>
        </div>
        <div>
          <p class="text-gray-500">{{ t.financing }}</p>
          <p class="text-green-400 font-medium">RM {{ formatNumber(account.financingAmount || 0) }}</p>
        </div>
        <div>
          <p class="text-gray-500">{{ t.monthlyProfit }}</p>
          <p class="text-red-400 font-medium">RM {{ formatNumber(getAccountMonthlyProfit(account)) }}</p>
        </div>
        <div>
          <p class="text-gray-500">{{ t.nextPayment }}</p>
          <template v-if="getNextPaymentDue(account)">
            <p class="font-medium" :class="getDaysRemaining(getNextPaymentDue(account).dueDate) <= 30 ? 'text-red-400' : 'text-gray-300'">
              {{ formatDate(getNextPaymentDue(account).dueDate) }}
            </p>
            <p class="text-xs text-gray-500">{{ getDaysRemaining(getNextPaymentDue(account).dueDate) }} {{ t.daysLeft }}</p>
          </template>
          <p v-else class="text-gray-500">-</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Empty State -->
  <div v-else class="text-center py-12 text-gray-500">
    <p class="text-lg">{{ t.noAccounts }}</p>
    <p class="text-sm mt-1">{{ t.addAccountHint }}</p>
  </div>
</div>
```

- [ ] **Step 5: Add tracker i18n strings**

```javascript
// EN additions
totalMarhun: 'Total Marhun',
totalFinancing: 'Total Financing',
totalMonthlyProfit: 'Monthly Profit',
nextPayment: 'Next Payment',
daysLeft: 'days left',
activeAccounts: 'Active Accounts',
addAccount: 'Add Account',
noAccountNo: 'No Account #',
edit: 'Edit',
delete: 'Delete',
noAccounts: 'No accounts yet',
addAccountHint: 'Add your first Ar-Rahnu account to start tracking',

// BM additions
totalMarhun: 'Jumlah Marhun',
totalFinancing: 'Jumlah Pembiayaan',
totalMonthlyProfit: 'Keuntungan Bulanan',
nextPayment: 'Bayaran Seterusnya',
daysLeft: 'hari lagi',
activeAccounts: 'Akaun Aktif',
addAccount: 'Tambah Akaun',
noAccountNo: 'Tiada No. Akaun',
edit: 'Edit',
delete: 'Padam',
noAccounts: 'Tiada akaun lagi',
addAccountHint: 'Tambah akaun Ar-Rahnu pertama anda',
```

- [ ] **Step 6: Add localStorage persistence for accounts**

In `mounted()`:

```javascript
const savedAccounts = localStorage.getItem('arrahnu_accounts')
if (savedAccounts) this.accounts = JSON.parse(savedAccounts)
```

Add watcher:

```javascript
accounts: {
  handler(val) { localStorage.setItem('arrahnu_accounts', JSON.stringify(val)) },
  deep: true
}
```

- [ ] **Step 7: Manual test**

Open `index.html`, go to Tracker tab:
- Dashboard shows all zeros and "No accounts yet"
- Language toggle translates all labels
- (Account form will be built in next task)

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add tracker dashboard and account list display"
```

---

## Task 5: Tracker Module — Add/Edit Account Form & Validation

**Files:**
- Modify: `index.html` (add modal form, validation, CRUD methods)

**What this builds:** Modal form to add/edit accounts with dynamic item list, auto-calculated fields, payment schedule generation, and validation per spec rules.

- [ ] **Step 1: Add form methods**

```javascript
openNewAccountForm() {
  this.editingAccountId = null
  this.newAccount = {
    accountNo: '', financingAccountNo: '', branch: '', status: 'active',
    createdDate: '', maturityDate: '',
    items: [{ description: '', karat: 24, weightGm: null, valueMYR: null }],
    financingAmount: null, rolloverFrom: null, rolloverTo: null, notes: ''
  }
  this.showAccountForm = true
},
editAccount(id) {
  const account = this.accounts.find(a => a.id === id)
  if (!account) return
  this.editingAccountId = id
  this.newAccount = JSON.parse(JSON.stringify(account))
  this.showAccountForm = true
},
addItem() {
  this.newAccount.items.push({ description: '', karat: 24, weightGm: null, valueMYR: null })
},
removeItem(index) {
  if (this.newAccount.items.length > 1) this.newAccount.items.splice(index, 1)
},
getFormMarhunValue() {
  return this.newAccount.items.reduce((sum, item) => sum + (item.valueMYR || 0), 0)
},
generatePaymentSchedule(createdDate, maturityDate, marhunValue) {
  if (!createdDate) return []
  const rate = this.getProfitRate(marhunValue)
  const monthlyProfit = marhunValue / 100 * rate
  const cycleProfit = monthlyProfit * 6
  const schedule = []
  const start = new Date(createdDate)
  for (let i = 1; i <= 3; i++) {
    const due = new Date(start)
    due.setMonth(due.getMonth() + (i * 6))
    schedule.push({
      cycle: i,
      dueDate: due.toISOString().split('T')[0],
      amount: parseFloat(cycleProfit.toFixed(2)),
      paid: false
    })
  }
  return schedule
},
validateAccount() {
  const a = this.newAccount
  const errors = []
  if (!a.accountNo.trim()) errors.push(this.t.errAccountNo)
  if (!a.createdDate) errors.push(this.t.errCreatedDate)
  if (!a.maturityDate) errors.push(this.t.errMaturityDate)
  if (a.createdDate && a.maturityDate && a.maturityDate <= a.createdDate) errors.push(this.t.errDateOrder)
  if (!a.items.length || a.items.every(i => !i.description)) errors.push(this.t.errItems)
  for (const item of a.items) {
    if (item.description && (!item.weightGm || item.weightGm <= 0)) errors.push(this.t.errItemWeight)
    if (item.description && (!item.valueMYR || item.valueMYR <= 0)) errors.push(this.t.errItemValue)
  }
  if (!a.financingAmount || a.financingAmount <= 0) errors.push(this.t.errFinancingRequired)
  const marhunValue = this.getFormMarhunValue()
  if (a.financingAmount && a.financingAmount > marhunValue * 0.8) errors.push(this.t.errFinancingMax)
  return [...new Set(errors)]
},
saveAccount() {
  const errors = this.validateAccount()
  if (errors.length) {
    alert(errors.join('\n'))
    return
  }
  const marhunValue = this.getFormMarhunValue()
  const accountData = {
    ...this.newAccount,
    id: this.editingAccountId || crypto.randomUUID(),
    paymentSchedule: this.generatePaymentSchedule(this.newAccount.createdDate, this.newAccount.maturityDate, marhunValue)
  }
  if (this.editingAccountId) {
    const idx = this.accounts.findIndex(a => a.id === this.editingAccountId)
    if (idx !== -1) this.accounts.splice(idx, 1, accountData)
  } else {
    this.accounts.push(accountData)
  }
  this.showAccountForm = false
},
deleteAccount(id) {
  if (confirm(this.t.confirmDelete)) {
    this.accounts = this.accounts.filter(a => a.id !== id)
  }
},
togglePayment(accountId, cycle) {
  const account = this.accounts.find(a => a.id === accountId)
  if (!account) return
  const payment = account.paymentSchedule.find(p => p.cycle === cycle)
  if (payment) payment.paid = !payment.paid
},
```

- [ ] **Step 2: Build the account form modal template**

Add after the account cards section, inside the tracker `<div>`:

```html
<!-- Account Form Modal -->
<div v-if="showAccountForm" class="fixed inset-0 bg-black/60 flex items-center justify-center z-50 p-4">
  <div class="bg-dark-700 rounded-xl border border-dark-600 w-full max-w-2xl max-h-[90vh] overflow-y-auto p-6">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-lg font-semibold text-gold-400">{{ editingAccountId ? t.editAccount : t.addAccount }}</h2>
      <button @click="showAccountForm = false" class="text-gray-400 hover:text-gray-200 text-xl">&times;</button>
    </div>

    <!-- Account Details -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.accountNo }} *</label>
        <input type="text" v-model="newAccount.accountNo" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.financingAccountNo }}</label>
        <input type="text" v-model="newAccount.financingAccountNo" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.branch }}</label>
        <input type="text" v-model="newAccount.branch" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.status }}</label>
        <select v-model="newAccount.status" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
          <option value="active">{{ getStatusLabel('active') }}</option>
          <option value="matured">{{ getStatusLabel('matured') }}</option>
          <option value="redeemed">{{ getStatusLabel('redeemed') }}</option>
          <option value="rolled_over">{{ getStatusLabel('rolled_over') }}</option>
        </select>
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.createdDate }} *</label>
        <input type="date" v-model="newAccount.createdDate" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.maturityDate }} *</label>
        <input type="date" v-model="newAccount.maturityDate" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.financingAmount }}</label>
        <input type="number" v-model.number="newAccount.financingAmount" min="0" step="0.01" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
    </div>

    <!-- Items (Marhun) -->
    <div class="mb-4">
      <div class="flex justify-between items-center mb-2">
        <h3 class="text-sm font-medium text-gold-300">{{ t.marhunItems }}</h3>
        <button @click="addItem" class="text-gold-400 hover:text-gold-300 text-sm">+ {{ t.addItem }}</button>
      </div>
      <div v-for="(item, idx) in newAccount.items" :key="idx" class="grid grid-cols-12 gap-2 mb-2 items-end">
        <div class="col-span-4">
          <label v-if="idx === 0" class="block text-xs text-gray-500 mb-1">{{ t.description }}</label>
          <input type="text" v-model="item.description" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-sm text-gray-100 focus:border-gold-500 focus:outline-none" :placeholder="t.itemDesc">
        </div>
        <div class="col-span-2">
          <label v-if="idx === 0" class="block text-xs text-gray-500 mb-1">{{ t.karat }}</label>
          <select v-model.number="item.karat" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-sm text-gray-100 focus:border-gold-500 focus:outline-none">
            <option v-for="k in karats" :value="k.value">{{ k.label }}</option>
          </select>
        </div>
        <div class="col-span-2">
          <label v-if="idx === 0" class="block text-xs text-gray-500 mb-1">{{ t.weight }} (gm)</label>
          <input type="number" v-model.number="item.weightGm" min="0" step="0.01" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-sm text-gray-100 focus:border-gold-500 focus:outline-none">
        </div>
        <div class="col-span-3">
          <label v-if="idx === 0" class="block text-xs text-gray-500 mb-1">{{ t.value }} (RM)</label>
          <input type="number" v-model.number="item.valueMYR" min="0" step="0.01" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-sm text-gray-100 focus:border-gold-500 focus:outline-none">
        </div>
        <div class="col-span-1">
          <button v-if="newAccount.items.length > 1" @click="removeItem(idx)" class="text-red-400 hover:text-red-300 text-sm p-1">&times;</button>
        </div>
      </div>
      <div class="text-sm text-gray-400 mt-2">
        {{ t.totalMarhun }}: <span class="text-gold-400 font-medium">RM {{ formatNumber(getFormMarhunValue()) }}</span>
      </div>
    </div>

    <!-- Notes -->
    <div class="mb-4">
      <label class="block text-sm text-gray-400 mb-1">{{ t.notes }}</label>
      <textarea v-model="newAccount.notes" rows="2" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none text-sm"></textarea>
    </div>

    <!-- Form Actions -->
    <div class="flex justify-end gap-3">
      <button @click="showAccountForm = false" class="px-4 py-2 text-sm text-gray-400 hover:text-gray-200 transition">{{ t.cancel }}</button>
      <button @click="saveAccount" class="bg-gold-500 hover:bg-gold-600 text-dark-900 px-6 py-2 rounded-lg text-sm font-medium transition">{{ t.save }}</button>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Add form i18n strings**

```javascript
// EN additions
editAccount: 'Edit Account',
accountNo: 'Account No',
financingAccountNo: 'Financing Account No',
branch: 'Branch',
status: 'Status',
createdDate: 'Start Date',
maturityDate: 'Maturity Date',
financingAmount: 'Financing Amount',
marhunItems: 'Marhun Items',
addItem: 'Add Item',
description: 'Description',
itemDesc: 'e.g. Gold wafer',
value: 'Value',
notes: 'Notes',
cancel: 'Cancel',
save: 'Save',
confirmDelete: 'Delete this account?',
errAccountNo: 'Account number is required',
errCreatedDate: 'Start date is required',
errMaturityDate: 'Maturity date is required',
errDateOrder: 'Maturity date must be after start date',
errItems: 'At least one marhun item is required',
errItemWeight: 'Item weight must be greater than 0',
errItemValue: 'Item value must be greater than 0',
errFinancingMax: 'Financing cannot exceed 80% of marhun value',
errFinancingRequired: 'Financing amount is required and must be greater than 0',

// BM additions
editAccount: 'Edit Akaun',
accountNo: 'No Akaun',
financingAccountNo: 'No Akaun Pembiayaan',
branch: 'Cawangan',
status: 'Status',
createdDate: 'Tarikh Mula',
maturityDate: 'Tarikh Matang',
financingAmount: 'Jumlah Pembiayaan',
marhunItems: 'Item Marhun',
addItem: 'Tambah Item',
description: 'Keterangan',
itemDesc: 'cth. Wafer emas',
value: 'Nilai',
notes: 'Nota',
cancel: 'Batal',
save: 'Simpan',
confirmDelete: 'Padam akaun ini?',
errAccountNo: 'No akaun diperlukan',
errCreatedDate: 'Tarikh mula diperlukan',
errMaturityDate: 'Tarikh matang diperlukan',
errDateOrder: 'Tarikh matang mesti selepas tarikh mula',
errItems: 'Sekurang-kurangnya satu item marhun diperlukan',
errItemWeight: 'Berat item mesti lebih daripada 0',
errItemValue: 'Nilai item mesti lebih daripada 0',
errFinancingMax: 'Pembiayaan tidak boleh melebihi 80% nilai marhun',
errFinancingRequired: 'Jumlah pembiayaan diperlukan dan mesti lebih daripada 0',
```

- [ ] **Step 4: Manual test**

Open `index.html`, Tracker tab:
- Click "Add Account" — modal opens
- Fill in account details, add multiple items
- Verify total marhun value auto-calculates
- Try saving without required fields — verify error messages
- Try financing > 80% of marhun — verify error
- Save a valid account — verify it appears in the list
- Edit the account — verify form pre-fills
- Delete — verify confirmation and removal
- Reload page — verify accounts persist

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add tracker account form with validation and CRUD"
```

---

## Task 6: Tracker Module — Rollover Chain & Export/Import

**Files:**
- Modify: `index.html` (add rollover visualization, export/import, data management)

**What this builds:** Rollover chain timeline for linked accounts, JSON export/import with merge/replace options, and clear data function.

- [ ] **Step 1: Add rollover chain template**

Add inside each account card, after the grid section:

```html
<!-- Rollover Chain (if applicable) -->
<div v-if="getRolloverChain(account.id).length > 1" class="mt-3 pt-3 border-t border-dark-600">
  <p class="text-xs text-gray-500 mb-2">{{ t.rolloverChain }}</p>
  <div class="flex items-center gap-1 flex-wrap text-xs">
    <template v-for="(node, idx) in getRolloverChain(account.id)" :key="node.id">
      <span :class="node.id === account.id ? 'bg-gold-500/20 text-gold-400 border-gold-500/50' : 'bg-dark-800 text-gray-400 border-dark-600'"
        class="border rounded px-2 py-1">
        {{ node.accountNo }} <span class="text-gray-500">{{ formatDate(node.createdDate) }}</span>
        <span class="ml-1">RM {{ formatNumber(node.financingAmount || 0) }}</span>
      </span>
      <span v-if="idx < getRolloverChain(account.id).length - 1" class="text-gray-600">&rarr;</span>
    </template>
  </div>
</div>
```

- [ ] **Step 2: Add rollover chain method**

```javascript
getRolloverChain(accountId) {
  const chain = []
  // Walk backward to find the start
  let current = this.accounts.find(a => a.id === accountId)
  while (current && current.rolloverFrom) {
    current = this.accounts.find(a => a.id === current.rolloverFrom)
  }
  // Walk forward from start
  while (current) {
    chain.push({ id: current.id, accountNo: current.accountNo, createdDate: current.createdDate, financingAmount: current.financingAmount })
    if (current.rolloverTo) {
      current = this.accounts.find(a => a.id === current.rolloverTo)
    } else {
      break
    }
  }
  return chain
},
```

- [ ] **Step 3: Add export/import methods**

```javascript
exportData() {
  const data = {
    version: 1,
    exportedAt: new Date().toISOString(),
    accounts: this.accounts
  }
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `arrahnu-backup-${new Date().toISOString().split('T')[0]}.json`
  a.click()
  URL.revokeObjectURL(url)
},
importData() {
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = '.json'
  input.onchange = (e) => {
    const file = e.target.files[0]
    if (!file) return
    const reader = new FileReader()
    reader.onload = (ev) => {
      try {
        const data = JSON.parse(ev.target.result)
        if (!data.accounts || !Array.isArray(data.accounts)) {
          alert(this.t.importInvalid)
          return
        }
        // Validate each account before importing
        const valid = []
        const invalid = []
        for (const acc of data.accounts) {
          if (acc.accountNo && acc.createdDate && acc.maturityDate && acc.items && acc.items.length > 0 && acc.financingAmount > 0) {
            valid.push(acc)
          } else {
            invalid.push(acc.accountNo || 'unknown')
          }
        }
        if (invalid.length) {
          alert(this.t.importSkipped + ': ' + invalid.join(', '))
        }
        if (!valid.length) {
          alert(this.t.importNoValid)
          return
        }
        const choice = confirm(this.t.importChoice)
        if (choice) {
          // Replace all
          this.accounts = valid
        } else {
          // Merge — skip duplicates
          const existingIds = new Set(this.accounts.map(a => a.id))
          const newAccounts = valid.filter(a => !existingIds.has(a.id))
          this.accounts.push(...newAccounts)
        }
      } catch {
        alert(this.t.importError)
      }
    }
    reader.readAsText(file)
  }
  input.click()
},
clearAllData() {
  if (confirm(this.t.confirmClearAll)) {
    this.accounts = []
    localStorage.removeItem('arrahnu_accounts')
  }
},
```

- [ ] **Step 4: Add data management buttons to tracker template**

Add below the "Add Account" button, in the same flex container:

```html
<div class="flex justify-between items-center flex-wrap gap-2">
  <div class="flex gap-2">
    <button @click="exportData" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-2 rounded-lg text-sm transition">{{ t.export }}</button>
    <button @click="importData" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-2 rounded-lg text-sm transition">{{ t.import }}</button>
    <button v-if="accounts.length" @click="clearAllData" class="bg-dark-600 hover:bg-red-900/50 text-gray-300 hover:text-red-400 px-3 py-2 rounded-lg text-sm transition">{{ t.clearAll }}</button>
  </div>
  <button @click="openNewAccountForm" class="bg-gold-500 hover:bg-gold-600 text-dark-900 px-4 py-2 rounded-lg text-sm font-medium transition">+ {{ t.addAccount }}</button>
</div>
```

- [ ] **Step 5: Add i18n strings**

```javascript
// EN
rolloverChain: 'Rollover Chain',
export: 'Export',
import: 'Import',
clearAll: 'Clear All',
confirmClearAll: 'Delete ALL accounts? This cannot be undone.',
importInvalid: 'Invalid file format. Expected Ar-Rahnu Pro JSON backup.',
importChoice: 'OK = Replace all data\nCancel = Merge (skip duplicates)',
importError: 'Failed to read file. Please check the format.',
importSkipped: 'Skipped invalid accounts',
importNoValid: 'No valid accounts found in file.',

// BM
rolloverChain: 'Rantaian Gadai Semula',
export: 'Eksport',
import: 'Import',
clearAll: 'Padam Semua',
confirmClearAll: 'Padam SEMUA akaun? Tindakan ini tidak boleh dibatalkan.',
importInvalid: 'Format fail tidak sah. Fail JSON Ar-Rahnu Pro diperlukan.',
importChoice: 'OK = Ganti semua data\nBatal = Gabung (langkau duplikat)',
importError: 'Gagal membaca fail. Sila semak format.',
importSkipped: 'Akaun tidak sah dilangkau',
importNoValid: 'Tiada akaun sah dalam fail.',
```

- [ ] **Step 6: Add rolloverFrom/rolloverTo selection to account form**

In the account form modal, add a rollover field after the financing amount:

```html
<div v-if="accounts.length">
  <label class="block text-sm text-gray-400 mb-1">{{ t.rolloverFrom }}</label>
  <select v-model="newAccount.rolloverFrom" class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
    <option :value="null">{{ t.none }}</option>
    <option v-for="a in accounts.filter(a => a.id !== editingAccountId)" :value="a.id">{{ a.accountNo }}</option>
  </select>
</div>
```

Add i18n: `rolloverFrom: 'Rolled Over From'` (EN), `rolloverFrom: 'Digadai Semula Dari'` (BM), `none: 'None'` (EN), `none: 'Tiada'` (BM).

Update `saveAccount` to also set `rolloverTo` on the linked account:

```javascript
// After saving, update the linked account's rolloverTo
if (accountData.rolloverFrom) {
  const prev = this.accounts.find(a => a.id === accountData.rolloverFrom)
  if (prev) prev.rolloverTo = accountData.id
}
```

- [ ] **Step 7: Manual test**

- Add 3 accounts, link them as a rollover chain (A → B → C)
- Verify rollover timeline renders with arrows
- Export data — verify JSON file downloads
- Clear all — verify all accounts removed
- Import the exported file — verify accounts restored
- Test merge vs replace on import

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add rollover chain, export/import, and data management"
```

---

## Task 7: Simulator Module — Scenario Builder & Live Results

**Files:**
- Modify: `index.html` (replace simulator placeholder with scenario builder)

**What this builds:** The Simulator tab with interactive inputs (sliders + fields), live results panel, and tenure toggle. Results update instantly as inputs change.

- [ ] **Step 1: Add simulator data properties**

Add to `data()`:

```javascript
// Simulator
sim: {
  goldPrice: null,
  weight: 100,
  karat: 24,
  financingPct: 80,
  tenure: 18
},
```

In `mounted()`, add:

```javascript
// Load saved simulator state
const savedSim = localStorage.getItem('arrahnu_simulatorState')
if (savedSim) this.sim = { ...this.sim, ...JSON.parse(savedSim) }
// Default sim gold price to current price
if (!this.sim.goldPrice && this.goldPrice) this.sim.goldPrice = this.goldPrice
```

Add watcher:

```javascript
sim: {
  handler(val) { localStorage.setItem('arrahnu_simulatorState', JSON.stringify(val)) },
  deep: true
},
```

- [ ] **Step 2: Add simulator computed properties**

```javascript
simPurity() {
  const k = this.karats.find(k => k.value === this.sim.karat)
  return k ? k.purity : 1
},
simMarhunValue() {
  if (!this.sim.weight || !this.sim.goldPrice) return 0
  return this.sim.weight * this.sim.goldPrice * this.simPurity
},
simFinancing() {
  return this.simMarhunValue * (this.sim.financingPct / 100)
},
simProfitRate() {
  return this.getProfitRate(this.simMarhunValue)
},
simMonthlyProfit() {
  return this.simMarhunValue / 100 * this.simProfitRate
},
simTotalProfit() {
  return this.simMonthlyProfit * this.sim.tenure
},
simTotalRepayment() {
  return this.simFinancing + this.simTotalProfit
},
simAnnualizedRate() {
  if (!this.simFinancing) return 0
  return (this.simTotalProfit / this.simFinancing) * (12 / this.sim.tenure) * 100
},
```

- [ ] **Step 3: Build simulator template**

Replace the simulator placeholder:

```html
<div v-if="activeTab === 'simulator'" class="space-y-6">
  <!-- Scenario Builder -->
  <div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
    <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.scenarioBuilder }}</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <!-- Gold Price Slider -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.goldPrice }} (RM/gm)</label>
        <div class="flex items-center gap-3">
          <input type="range" v-model.number="sim.goldPrice" :min="Math.max(100, (goldPrice || 300) * 0.5)" :max="(goldPrice || 500) * 1.5" step="0.50"
            class="flex-1 accent-gold-500">
          <input type="number" v-model.number="sim.goldPrice" min="0" step="0.01"
            class="w-28 bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-right text-gold-400 focus:border-gold-500 focus:outline-none">
        </div>
        <div v-if="goldPrice" class="text-xs text-gray-500 mt-1">
          {{ t.currentSpot }}: RM {{ formatNumber(goldPrice) }}
          <span v-if="sim.goldPrice && goldPrice" :class="sim.goldPrice > goldPrice ? 'text-green-400' : sim.goldPrice < goldPrice ? 'text-red-400' : 'text-gray-400'">
            ({{ sim.goldPrice >= goldPrice ? '+' : '' }}{{ ((sim.goldPrice - goldPrice) / goldPrice * 100).toFixed(1) }}%)
          </span>
        </div>
      </div>
      <!-- Weight -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.weight }} (gm)</label>
        <div class="flex items-center gap-3">
          <input type="range" v-model.number="sim.weight" min="1" max="500" step="1" class="flex-1 accent-gold-500">
          <input type="number" v-model.number="sim.weight" min="0" step="0.01"
            class="w-28 bg-dark-800 border border-dark-600 rounded px-2 py-1.5 text-right text-gray-100 focus:border-gold-500 focus:outline-none">
        </div>
      </div>
      <!-- Karat -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.karat }}</label>
        <select v-model.number="sim.karat"
          class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
          <option v-for="k in karats" :value="k.value">{{ k.label }} ({{ (k.purity * 100).toFixed(1) }}%)</option>
        </select>
      </div>
      <!-- Financing % -->
      <div>
        <label class="block text-sm text-gray-400 mb-1">{{ t.financingPct }}: {{ sim.financingPct }}%</label>
        <input type="range" v-model.number="sim.financingPct" min="50" max="80" step="1" class="w-full accent-gold-500">
      </div>
    </div>
    <!-- Tenure -->
    <div class="mt-4">
      <label class="block text-sm text-gray-400 mb-2">{{ t.tenure }}</label>
      <div class="flex gap-2">
        <button v-for="m in tenureOptions" :key="m" @click="sim.tenure = m"
          :class="sim.tenure === m ? 'bg-gold-500 text-dark-900' : 'bg-dark-600 text-gray-300 hover:bg-dark-500'"
          class="px-4 py-1.5 rounded-lg text-sm font-medium transition">
          {{ m }} {{ t.months }}
        </button>
      </div>
    </div>
  </div>

  <!-- Live Results -->
  <div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
    <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.simResults }}</h2>
    <div class="grid grid-cols-2 md:grid-cols-3 gap-4 mb-4">
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.marhunValue }}</p>
        <p class="text-lg font-bold text-gold-300">RM {{ formatNumber(simMarhunValue) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.financing }} ({{ sim.financingPct }}%)</p>
        <p class="text-lg font-bold text-green-400">RM {{ formatNumber(simFinancing) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.monthlyProfit }}</p>
        <p class="text-lg font-bold text-red-400">RM {{ formatNumber(simMonthlyProfit) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.totalProfit }} ({{ sim.tenure }} {{ t.months }})</p>
        <p class="text-lg font-bold text-red-400">RM {{ formatNumber(simTotalProfit) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.totalRepayment }}</p>
        <p class="text-lg font-bold text-gold-400">RM {{ formatNumber(simTotalRepayment) }}</p>
      </div>
      <div class="bg-dark-800 rounded-lg p-4">
        <p class="text-xs text-gray-400">{{ t.annualizedRate }}</p>
        <p class="text-lg font-bold text-yellow-400">{{ simAnnualizedRate.toFixed(1) }}%</p>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 4: Add simulator i18n strings**

```javascript
// EN
scenarioBuilder: 'Scenario Builder',
currentSpot: 'Current spot',
simResults: 'Simulation Results',
totalProfit: 'Total Profit',

// BM
scenarioBuilder: 'Pembina Senario',
currentSpot: 'Harga semasa',
simResults: 'Keputusan Simulasi',
totalProfit: 'Jumlah Keuntungan',
```

- [ ] **Step 5: Sync sim gold price with header price on load**

In `mounted()`, after `fetchGoldPrice()`, add a watcher or use `nextTick` so that once goldPrice loads, sim.goldPrice is set if it's null:

```javascript
this.$watch('goldPrice', (val) => {
  if (val && !this.sim.goldPrice) this.sim.goldPrice = val
}, { immediate: true })
```

- [ ] **Step 6: Manual test**

Open `index.html`, Simulator tab:
- Verify sliders and inputs work reactively
- Change gold price — results update instantly
- Change weight, karat — results update
- Toggle tenure — totals change
- Percentage change indicator shows relative to spot price
- Reload — simulator state persists

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add simulator with scenario builder and live results"
```

---

## Task 8: Simulator Module — Comparison Mode & Early Redemption

**Files:**
- Modify: `index.html` (add comparison mode, preset scenarios, early redemption calculator)

**What this builds:** Side-by-side scenario comparison, preset scenario buttons (gold price change, add gold), and early redemption calculator.

- [ ] **Step 1: Add comparison mode data**

Add to `data()`:

```javascript
showComparison: false,
simB: {
  goldPrice: null,
  weight: 100,
  karat: 24,
  financingPct: 80,
  tenure: 18
},
earlyRedemption: {
  monthsHeld: 6
},
addGold: {
  weight: 50,
  karat: 24
},
rolloverCalc: {
  monthsBeforeRedeem: 6
},
```

- [ ] **Step 2: Add simB computed properties**

Add a helper method instead of duplicating computed properties:

```javascript
calcScenario(s) {
  const purity = (this.karats.find(k => k.value === s.karat) || {}).purity || 1
  const marhunValue = (s.weight || 0) * (s.goldPrice || 0) * purity
  const financing = marhunValue * (s.financingPct / 100)
  const profitRate = this.getProfitRate(marhunValue)
  const monthlyProfit = marhunValue / 100 * profitRate
  const totalProfit = monthlyProfit * s.tenure
  const totalRepayment = financing + totalProfit
  const annualizedRate = financing ? (totalProfit / financing) * (12 / s.tenure) * 100 : 0
  return { marhunValue, financing, profitRate, monthlyProfit, totalProfit, totalRepayment, annualizedRate }
},
```

Add computed:

```javascript
simAResult() { return this.calcScenario(this.sim) },
simBResult() { return this.calcScenario(this.simB) },
simDiff() {
  const a = this.simAResult
  const b = this.simBResult
  return {
    marhunValue: b.marhunValue - a.marhunValue,
    financing: b.financing - a.financing,
    monthlyProfit: b.monthlyProfit - a.monthlyProfit,
    totalProfit: b.totalProfit - a.totalProfit,
    totalRepayment: b.totalRepayment - a.totalRepayment
  }
},
addGoldResult() {
  const purity = (this.karats.find(k => k.value === this.addGold.karat) || {}).purity || 1
  const extraMarhun = (this.addGold.weight || 0) * (this.sim.goldPrice || 0) * purity
  const extraFinancing = extraMarhun * (this.sim.financingPct / 100)
  const newTotalFinancing = this.simFinancing + extraFinancing
  return { extraMarhun, extraFinancing, newTotalFinancing }
},
rolloverCalcResult() {
  const m = this.rolloverCalc.monthsBeforeRedeem
  // Option 1: Just roll over (pay full tenure profit)
  const rolloverCost = this.simTotalProfit
  // Option 2: Redeem at month m, then re-pledge (pay partial + new full tenure)
  const profitBeforeRedeem = this.simMonthlyProfit * m
  const newPledgeProfit = this.simMonthlyProfit * this.sim.tenure // re-pledge at same rate
  const redeemRePledgeCost = profitBeforeRedeem + newPledgeProfit
  const savings = redeemRePledgeCost - rolloverCost
  return { totalCost: redeemRePledgeCost, savings }
},
earlyRedemptionResult() {
  const months = this.earlyRedemption.monthsHeld
  const profitPaid = this.simMonthlyProfit * months
  const profitFull = this.simMonthlyProfit * this.sim.tenure
  const saved = profitFull - profitPaid
  return { profitPaid, profitFull, saved, netCost: this.simFinancing + profitPaid }
},
```

- [ ] **Step 3: Add comparison mode template**

Add below the live results section in the simulator:

```html
<!-- Comparison Toggle -->
<div class="flex items-center gap-4">
  <button @click="showComparison = !showComparison"
    :class="showComparison ? 'bg-gold-500 text-dark-900' : 'bg-dark-600 text-gray-300'"
    class="px-4 py-2 rounded-lg text-sm font-medium transition">
    {{ t.compareMode }}
  </button>
</div>

<!-- Comparison Mode -->
<div v-if="showComparison" class="grid grid-cols-1 md:grid-cols-2 gap-4">
  <!-- Scenario A -->
  <div class="bg-dark-700 rounded-xl p-5 border border-gold-500/30">
    <h3 class="text-sm font-semibold text-gold-400 mb-3">{{ t.scenarioA }}</h3>
    <div class="space-y-2 text-sm">
      <div class="flex justify-between"><span class="text-gray-400">{{ t.marhunValue }}</span><span class="text-gold-300">RM {{ formatNumber(simAResult.marhunValue) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.financing }}</span><span class="text-green-400">RM {{ formatNumber(simAResult.financing) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.monthlyProfit }}</span><span class="text-red-400">RM {{ formatNumber(simAResult.monthlyProfit) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.totalProfit }}</span><span class="text-red-400">RM {{ formatNumber(simAResult.totalProfit) }}</span></div>
      <div class="flex justify-between font-semibold"><span class="text-gray-300">{{ t.totalRepayment }}</span><span class="text-gold-400">RM {{ formatNumber(simAResult.totalRepayment) }}</span></div>
    </div>
  </div>
  <!-- Scenario B -->
  <div class="bg-dark-700 rounded-xl p-5 border border-dark-600">
    <h3 class="text-sm font-semibold text-gold-400 mb-3">{{ t.scenarioB }}</h3>
    <div class="grid grid-cols-2 gap-2 mb-3 text-sm">
      <div>
        <label class="text-xs text-gray-500">{{ t.goldPrice }}</label>
        <input type="number" v-model.number="simB.goldPrice" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="text-xs text-gray-500">{{ t.weight }}</label>
        <input type="number" v-model.number="simB.weight" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
      <div>
        <label class="text-xs text-gray-500">{{ t.karat }}</label>
        <select v-model.number="simB.karat" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1 text-gray-100 text-sm focus:border-gold-500 focus:outline-none">
          <option v-for="k in karats" :value="k.value">{{ k.label }}</option>
        </select>
      </div>
      <div>
        <label class="text-xs text-gray-500">{{ t.financingPct }}</label>
        <input type="number" v-model.number="simB.financingPct" min="50" max="80" class="w-full bg-dark-800 border border-dark-600 rounded px-2 py-1 text-gray-100 focus:border-gold-500 focus:outline-none">
      </div>
    </div>
    <div class="mb-3">
      <div class="flex gap-1">
        <button v-for="m in tenureOptions" :key="m" @click="simB.tenure = m"
          :class="simB.tenure === m ? 'bg-gold-500 text-dark-900' : 'bg-dark-600 text-gray-300'"
          class="px-3 py-1 rounded text-xs font-medium transition">{{ m }}{{ t.monthsShort }}</button>
      </div>
    </div>
    <div class="space-y-2 text-sm">
      <div class="flex justify-between"><span class="text-gray-400">{{ t.marhunValue }}</span><span class="text-gold-300">RM {{ formatNumber(simBResult.marhunValue) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.financing }}</span><span class="text-green-400">RM {{ formatNumber(simBResult.financing) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.monthlyProfit }}</span><span class="text-red-400">RM {{ formatNumber(simBResult.monthlyProfit) }}</span></div>
      <div class="flex justify-between"><span class="text-gray-400">{{ t.totalProfit }}</span><span class="text-red-400">RM {{ formatNumber(simBResult.totalProfit) }}</span></div>
      <div class="flex justify-between font-semibold"><span class="text-gray-300">{{ t.totalRepayment }}</span><span class="text-gold-400">RM {{ formatNumber(simBResult.totalRepayment) }}</span></div>
    </div>
  </div>
  <!-- Difference Row -->
  <div class="md:col-span-2 bg-dark-800 rounded-xl p-4 border border-dark-600">
    <h3 class="text-sm font-semibold text-gray-400 mb-2">{{ t.difference }} (B - A)</h3>
    <div class="grid grid-cols-2 md:grid-cols-5 gap-3 text-sm">
      <div>
        <p class="text-xs text-gray-500">{{ t.marhunValue }}</p>
        <p :class="simDiff.marhunValue >= 0 ? 'text-green-400' : 'text-red-400'" class="font-medium">{{ simDiff.marhunValue >= 0 ? '+' : '' }}RM {{ formatNumber(simDiff.marhunValue) }}</p>
      </div>
      <div>
        <p class="text-xs text-gray-500">{{ t.financing }}</p>
        <p :class="simDiff.financing >= 0 ? 'text-green-400' : 'text-red-400'" class="font-medium">{{ simDiff.financing >= 0 ? '+' : '' }}RM {{ formatNumber(simDiff.financing) }}</p>
      </div>
      <div>
        <p class="text-xs text-gray-500">{{ t.monthlyProfit }}</p>
        <p :class="simDiff.monthlyProfit <= 0 ? 'text-green-400' : 'text-red-400'" class="font-medium">{{ simDiff.monthlyProfit >= 0 ? '+' : '' }}RM {{ formatNumber(simDiff.monthlyProfit) }}</p>
      </div>
      <div>
        <p class="text-xs text-gray-500">{{ t.totalProfit }}</p>
        <p :class="simDiff.totalProfit <= 0 ? 'text-green-400' : 'text-red-400'" class="font-medium">{{ simDiff.totalProfit >= 0 ? '+' : '' }}RM {{ formatNumber(simDiff.totalProfit) }}</p>
      </div>
      <div>
        <p class="text-xs text-gray-500">{{ t.totalRepayment }}</p>
        <p :class="simDiff.totalRepayment <= 0 ? 'text-green-400' : 'text-red-400'" class="font-medium">{{ simDiff.totalRepayment >= 0 ? '+' : '' }}RM {{ formatNumber(simDiff.totalRepayment) }}</p>
      </div>
    </div>
  </div>
</div>

<!-- Preset Scenarios -->
<div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
  <h2 class="text-lg font-semibold text-gold-400 mb-3">{{ t.presets }}</h2>
  <div class="flex flex-wrap gap-2">
    <button @click="sim.goldPrice = goldPrice * 0.9" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-1.5 rounded-lg text-sm transition">{{ t.presetDrop10 }}</button>
    <button @click="sim.goldPrice = goldPrice * 1.1" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-1.5 rounded-lg text-sm transition">{{ t.presetRise10 }}</button>
    <button @click="sim.goldPrice = goldPrice * 0.8" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-1.5 rounded-lg text-sm transition">{{ t.presetDrop20 }}</button>
    <button @click="sim.goldPrice = goldPrice * 1.2" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-1.5 rounded-lg text-sm transition">{{ t.presetRise20 }}</button>
    <button @click="sim.goldPrice = goldPrice" class="bg-dark-600 hover:bg-dark-500 text-gray-300 px-3 py-1.5 rounded-lg text-sm transition">{{ t.presetReset }}</button>
  </div>
</div>

<!-- Add More Gold Calculator -->
<div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
  <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.addMoreGold }}</h2>
  <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
    <div>
      <label class="block text-sm text-gray-400 mb-1">{{ t.additionalWeight }} (gm)</label>
      <input type="number" v-model.number="addGold.weight" min="0" step="0.01"
        class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
    </div>
    <div>
      <label class="block text-sm text-gray-400 mb-1">{{ t.karat }}</label>
      <select v-model.number="addGold.karat"
        class="w-full bg-dark-800 border border-dark-600 rounded-lg px-3 py-2 text-gray-100 focus:border-gold-500 focus:outline-none">
        <option v-for="k in karats" :value="k.value">{{ k.label }}</option>
      </select>
    </div>
    <div class="flex items-end">
      <div class="bg-dark-800 rounded-lg p-3 w-full">
        <p class="text-xs text-gray-400">{{ t.additionalFinancing }}</p>
        <p class="text-lg font-bold text-green-400">+ RM {{ formatNumber(addGoldResult.extraFinancing) }}</p>
        <p class="text-xs text-gray-500">{{ t.newTotal }}: RM {{ formatNumber(addGoldResult.newTotalFinancing) }}</p>
      </div>
    </div>
  </div>
</div>

<!-- Rollover Cost Calculator -->
<div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
  <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.rolloverCost }}</h2>
  <p class="text-sm text-gray-400 mb-4">{{ t.rolloverCostDesc }}</p>
  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
    <div class="bg-dark-800 rounded-lg p-4">
      <h3 class="text-sm font-medium text-gold-300 mb-2">{{ t.rolloverOption }}</h3>
      <p class="text-xs text-gray-400">{{ t.profitPaid }} ({{ sim.tenure }} {{ t.months }})</p>
      <p class="text-lg font-bold text-red-400">RM {{ formatNumber(simTotalProfit) }}</p>
      <p class="text-xs text-gray-400 mt-2">{{ t.rolloverNote }}</p>
    </div>
    <div class="bg-dark-800 rounded-lg p-4">
      <h3 class="text-sm font-medium text-gold-300 mb-2">{{ t.redeemRePledge }}</h3>
      <p class="text-xs text-gray-400">{{ t.profitPaid }} ({{ rolloverCalc.monthsBeforeRedeem }} {{ t.months }}) + {{ t.newPledgeProfit }}</p>
      <p class="text-lg font-bold text-red-400">RM {{ formatNumber(rolloverCalcResult.totalCost) }}</p>
      <p class="text-xs mt-2" :class="rolloverCalcResult.savings >= 0 ? 'text-green-400' : 'text-red-400'">
        {{ rolloverCalcResult.savings >= 0 ? t.youSave : t.extraCost }}: RM {{ formatNumber(Math.abs(rolloverCalcResult.savings)) }}
      </p>
    </div>
  </div>
  <div class="mt-3">
    <label class="block text-sm text-gray-400 mb-1">{{ t.monthsBeforeRedeem }}: {{ rolloverCalc.monthsBeforeRedeem }}</label>
    <input type="range" v-model.number="rolloverCalc.monthsBeforeRedeem" min="1" :max="sim.tenure" step="1" class="w-full accent-gold-500">
  </div>
</div>

<!-- Early Redemption Calculator -->
<div class="bg-dark-700 rounded-xl p-6 border border-dark-600">
  <h2 class="text-lg font-semibold text-gold-400 mb-4">{{ t.earlyRedemption }}</h2>
  <div class="mb-4">
    <label class="block text-sm text-gray-400 mb-2">{{ t.monthsHeld }}: {{ earlyRedemption.monthsHeld }}</label>
    <input type="range" v-model.number="earlyRedemption.monthsHeld" min="1" :max="sim.tenure" step="1" class="w-full accent-gold-500">
  </div>
  <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
    <div class="bg-dark-800 rounded-lg p-4">
      <p class="text-xs text-gray-400">{{ t.profitPaid }}</p>
      <p class="text-lg font-bold text-red-400">RM {{ formatNumber(earlyRedemptionResult.profitPaid) }}</p>
    </div>
    <div class="bg-dark-800 rounded-lg p-4">
      <p class="text-xs text-gray-400">{{ t.profitFullTenure }}</p>
      <p class="text-lg font-bold text-gray-400">RM {{ formatNumber(earlyRedemptionResult.profitFull) }}</p>
    </div>
    <div class="bg-dark-800 rounded-lg p-4">
      <p class="text-xs text-gray-400">{{ t.youSave }}</p>
      <p class="text-lg font-bold text-green-400">RM {{ formatNumber(earlyRedemptionResult.saved) }}</p>
    </div>
    <div class="bg-dark-800 rounded-lg p-4">
      <p class="text-xs text-gray-400">{{ t.netCost }}</p>
      <p class="text-lg font-bold text-gold-400">RM {{ formatNumber(earlyRedemptionResult.netCost) }}</p>
    </div>
  </div>
</div>
```

- [ ] **Step 4: Add i18n strings for comparison and presets**

```javascript
// EN
compareMode: 'Compare Scenarios',
scenarioA: 'Scenario A (current)',
scenarioB: 'Scenario B',
difference: 'Difference',
presets: 'Quick Scenarios',
presetDrop10: 'Gold -10%',
presetRise10: 'Gold +10%',
presetDrop20: 'Gold -20%',
presetRise20: 'Gold +20%',
presetReset: 'Reset to Current',
earlyRedemption: 'Early Redemption Calculator',
monthsHeld: 'Months Held',
profitPaid: 'Profit Paid',
profitFullTenure: 'Full Tenure Profit',
youSave: 'You Save',
netCost: 'Net Cost (Financing + Profit)',
addMoreGold: 'Add More Gold',
additionalWeight: 'Additional Weight',
additionalFinancing: 'Extra Financing Available',
newTotal: 'New Total',
rolloverCost: 'Rollover vs Redeem & Re-Pledge',
rolloverCostDesc: 'Compare the cost of rolling over your current pledge vs redeeming early and re-pledging.',
rolloverOption: 'Rollover (keep current)',
rolloverNote: 'Continue paying profit on existing pledge',
redeemRePledge: 'Redeem & Re-Pledge',
newPledgeProfit: 'new pledge profit',
extraCost: 'Extra cost',
monthsBeforeRedeem: 'Months before redemption',
monthsShort: 'm',

// BM
compareMode: 'Bandingkan Senario',
scenarioA: 'Senario A (semasa)',
scenarioB: 'Senario B',
difference: 'Perbezaan',
presets: 'Senario Pantas',
presetDrop10: 'Emas -10%',
presetRise10: 'Emas +10%',
presetDrop20: 'Emas -20%',
presetRise20: 'Emas +20%',
presetReset: 'Set Semula',
earlyRedemption: 'Kalkulator Penebusan Awal',
monthsHeld: 'Bulan Dipegang',
profitPaid: 'Keuntungan Dibayar',
profitFullTenure: 'Keuntungan Penuh',
youSave: 'Anda Jimat',
netCost: 'Kos Bersih (Pembiayaan + Keuntungan)',
addMoreGold: 'Tambah Lagi Emas',
additionalWeight: 'Berat Tambahan',
additionalFinancing: 'Pembiayaan Tambahan',
newTotal: 'Jumlah Baru',
rolloverCost: 'Gadai Semula vs Tebus & Gadai Baru',
rolloverCostDesc: 'Bandingkan kos gadai semula berbanding tebus awal dan gadai baru.',
rolloverOption: 'Gadai Semula (kekalkan)',
rolloverNote: 'Terus bayar keuntungan gadaian sedia ada',
redeemRePledge: 'Tebus & Gadai Baru',
newPledgeProfit: 'keuntungan gadaian baru',
extraCost: 'Kos tambahan',
monthsBeforeRedeem: 'Bulan sebelum tebus',
monthsShort: 'b',
```

- [ ] **Step 5: Initialize simB gold price**

In the `goldPrice` watcher, also set simB:

```javascript
if (val && !this.simB.goldPrice) this.simB.goldPrice = val
```

- [ ] **Step 6: Manual test**

Open `index.html`, Simulator tab:
- Click "Compare Scenarios" — side-by-side appears
- Adjust Scenario B inputs — results update live
- Difference row shows correct deltas with green/red coloring
- Preset buttons change gold price — results update
- Early redemption slider works — shows savings vs full tenure
- On mobile (resize browser), comparison stacks vertically

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add comparison mode, preset scenarios, and early redemption calculator"
```

---

## Task 9: Final Polish — Responsive, Footer, Edge Cases

**Files:**
- Modify: `index.html` (responsive fixes, footer, edge case handling)

**What this builds:** Final polish — responsive tweaks, footer with app info, handle edge cases (zero values, missing gold price, empty states).

- [ ] **Step 1: Add footer**

Below `</main>`, add:

```html
<footer class="max-w-6xl mx-auto px-4 py-6 mt-8 border-t border-dark-600 text-center text-xs text-gray-600">
  <p>Ar-Rahnu Pro v1.0 — {{ t.footerDesc }}</p>
  <p class="mt-1">{{ t.footerDisclaimer }}</p>
</footer>
```

Add i18n:

```javascript
// EN
footerDesc: 'Personal Ar-Rahnu Calculator, Tracker & Simulator',
footerDisclaimer: 'This tool is for personal reference only. Always verify with Bank Rakyat for official rates and calculations.',

// BM
footerDesc: 'Kalkulator, Penjejak & Simulasi Ar-Rahnu Peribadi',
footerDisclaimer: 'Alat ini untuk rujukan peribadi sahaja. Sentiasa sahkan dengan Bank Rakyat untuk kadar dan pengiraan rasmi.',
```

- [ ] **Step 2: Add edge case handling for zero/missing gold price**

In the Calculator and Simulator, wrap results with a check:

```html
<!-- Add to Calculator results card -->
<div v-if="!goldPrice" class="bg-yellow-500/10 border border-yellow-500/30 rounded-lg p-4 text-sm text-yellow-400">
  {{ t.enterGoldPrice }}
</div>
```

Add i18n: `enterGoldPrice: 'Please enter or wait for gold price to load'` (EN), `enterGoldPrice: 'Sila masukkan atau tunggu harga emas dimuatkan'` (BM).

- [ ] **Step 3: Add responsive tweaks**

Add to `<style>`:

Note: Do NOT add global grid overrides — Tailwind's responsive prefixes (`grid-cols-1 md:grid-cols-2`) already handle mobile layouts. Only add number input spinner removal:

```css
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type="number"] { -moz-appearance: textfield; }
```

- [ ] **Step 4: Add payment toggle to account cards**

In the account card, add a payment schedule section:

```html
<!-- Payment Schedule -->
<div v-if="account.paymentSchedule && account.paymentSchedule.length" class="mt-3 pt-3 border-t border-dark-600">
  <p class="text-xs text-gray-500 mb-2">{{ t.paymentSchedule }}</p>
  <div class="flex gap-2 flex-wrap">
    <button v-for="p in account.paymentSchedule" :key="p.cycle" @click="togglePayment(account.id, p.cycle)"
      :class="p.paid ? 'bg-green-500/20 text-green-400 border-green-500/50 line-through' : 'bg-dark-800 text-gray-300 border-dark-600'"
      class="border rounded px-3 py-1.5 text-xs transition">
      {{ t.cycle }} {{ p.cycle }}: RM {{ formatNumber(p.amount) }}
      <span class="text-gray-500 ml-1">{{ formatDate(p.dueDate) }}</span>
    </button>
  </div>
</div>
```

Add i18n: `paymentSchedule: 'Payment Schedule'` (EN), `paymentSchedule: 'Jadual Pembayaran'` (BM), `cycle: 'Cycle'` (EN), `cycle: 'Kitaran'` (BM).

- [ ] **Step 5: Manual test — full app test**

Open `index.html` and test the complete app:
- **Calculator**: enter values, verify calculations, switch tenure, switch language
- **Tracker**: add account, edit, delete, toggle payments, verify dashboard totals
- **Simulator**: adjust sliders, compare scenarios, use presets, early redemption
- **Header**: gold price loads or shows manual entry, language toggle works everywhere
- **Responsive**: resize to mobile width, verify layout stacks properly
- **Persistence**: reload page, verify all data persists (accounts, language, tab, simulator state)
- **Export/Import**: export data, clear all, import back

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add final polish — footer, responsive, payment schedule, edge cases"
```

---

## Summary

| Task | Description | Est. Steps |
|------|-------------|------------|
| 1 | Scaffold — HTML, Vue, Tailwind, header, tabs, i18n | 6 |
| 2 | Gold price API integration | 6 |
| 3 | Calculator module | 6 |
| 4 | Tracker — dashboard & account list | 8 |
| 5 | Tracker — add/edit form & validation | 5 |
| 6 | Tracker — rollover chain & export/import | 8 |
| 7 | Simulator — scenario builder & live results | 7 |
| 8 | Simulator — comparison mode & early redemption | 7 |
| 9 | Final polish | 6 |
| **Total** | | **59 steps** |
