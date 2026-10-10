<template>
  <div class="container" :class="{ 'dark-mode': darkMode }">
    <header class="top-bar">
      <div class="app-title">
        <span class="app-icon" aria-hidden="true">🧾</span>
        <h1>Kalkulator PPh 21</h1>
      </div>
      <button
        class="theme-toggle"
        @click="toggleDarkMode"
        :aria-label="darkMode ? 'Aktifkan mode terang' : 'Aktifkan mode gelap'"
        :title="darkMode ? 'Mode Terang' : 'Mode Gelap'"
      >
        <span v-if="darkMode" aria-hidden="true">☀️</span>
        <span v-else aria-hidden="true">🌙</span>
      </button>
    </header>

    <div class="form-group">
      <label for="income">Penghasilan Tahunan Setelah Dipotong PTKP</label>
      <div class="input-wrap">
        <span class="input-prefix">Rp</span>
        <input
          type="text"
          id="income"
          inputmode="numeric"
          autocomplete="off"
          v-model="formattedIncome"
          @input="formatInput"
          placeholder="0"
        />
      </div>
    </div>

    <button class="calc-btn" @click="calculateTax">
      <span aria-hidden="true">🧮</span>
      <span>Hitung Pajak</span>
    </button>

    <p v-if="taxResults.length === 0" class="empty-hint">
      Masukkan penghasilan tahunan, lalu tekan tombol di atas untuk melihat
      rincian pajak, perbandingan PT, dan penghasilan bersih Anda.
    </p>

    <div v-if="taxResults.length > 0" class="results">
      <!-- Rincian per lapisan -->
      <div class="section">
        <span class="eyebrow">Rincian per Lapisan Tarif</span>

        <div class="bracket-bar">
          <div
            v-for="(result, index) in taxResults"
            :key="'bar-' + index"
            class="bracket-segment"
            :style="{
              width: (result.amount / totalTax) * 100 + '%',
              backgroundColor: result.color,
            }"
          ></div>
        </div>

        <ul class="bracket-list">
          <li v-for="(result, index) in taxResults" :key="index">
            <span class="dot" :style="{ backgroundColor: result.color }"></span>
            <span class="bracket-text">
              <span class="bracket-name"
                >Lapisan {{ index + 1 }} · {{ result.rate }}%</span
              >
              <span class="bracket-range" v-if="result.max === Infinity"
                >di atas {{ formatCurrency(result.min) }}</span
              >
              <span class="bracket-range" v-else
                >hingga {{ formatCurrency(result.max) }}</span
              >
            </span>
            <span class="bracket-amount">{{
              formatCurrency(result.amount)
            }}</span>
          </li>
        </ul>
      </div>

      <!-- Perbandingan PPh 21 vs PT -->
      <div class="compare-grid">
        <div class="compare-card individu">
          <span class="compare-title">👤 PPh 21 (Orang Pribadi)</span>
          <div class="compare-row">
            <span>Total Pajak</span>
            <strong>{{ formatCurrency(totalTax) }}</strong>
          </div>
          <div class="compare-row">
            <span>Penghasilan Bersih</span>
            <strong>{{ formatCurrency(netIncome) }}</strong>
          </div>
          <span class="compare-sub"
            >{{ totalTaxPercentage }}% dari penghasilan tahunan</span
          >
        </div>

        <div class="compare-card badan">
          <span class="compare-title">🏢 Badan / PT (Tarif 22%)</span>
          <div class="compare-row">
            <span>Pajak PT</span>
            <strong>{{ formatCurrency(badanTax) }}</strong>
          </div>
          <div class="compare-row">
            <span>Penghasilan Bersih PT</span>
            <strong>{{ formatCurrency(badanNetIncome) }}</strong>
          </div>
        </div>
      </div>
    </div>

    <p class="design">
      Website design by. <a href="https://kokofibo.com">Kokofibo.com</a> 0877
      265 888 36
    </p>
  </div>
</template>

<script>
export default {
  name: "Pph21Calculator",
  data() {
    return {
      annualIncome: 0,
      formattedIncome: "",
      darkMode: false,
      taxBrackets: [
        { max: 60000000, rate: 5, color: "#bfe8d2" },
        { max: 250000000, rate: 15, color: "#8fd1ab" },
        { max: 500000000, rate: 25, color: "#5fb98a" },
        { max: 5000000000, rate: 30, color: "#2f9966" },
        { max: Infinity, rate: 35, color: "#0d6e4f" },
      ],
      badanTaxRate: 0.22, // Tarif umum PPh Badan (Pasal 17 ayat 1b UU PPh)
      taxResults: [],
    };
  },
  computed: {
    totalTax() {
      return this.taxResults.reduce((sum, result) => sum + result.amount, 0);
    },
    totalTaxPercentage() {
      return ((this.totalTax / this.annualIncome) * 100).toFixed(2);
    },
    netIncome() {
      return this.annualIncome - this.totalTax;
    },
    badanTax() {
      return this.annualIncome * this.badanTaxRate;
    },
    badanNetIncome() {
      return this.annualIncome - this.badanTax;
    },
  },
  mounted() {
    // Muat preferensi dark mode dari localStorage, atau ikuti preferensi sistem
    const saved = localStorage.getItem("pph21-dark-mode");
    if (saved !== null) {
      this.darkMode = saved === "true";
    } else if (window.matchMedia) {
      this.darkMode = window.matchMedia("(prefers-color-scheme: dark)").matches;
    }
  },
  methods: {
    toggleDarkMode() {
      this.darkMode = !this.darkMode;
      localStorage.setItem("pph21-dark-mode", this.darkMode);
    },
    formatInput(event) {
      let value = event.target.value.replace(/[^0-9]/g, "");
      this.formattedIncome = new Intl.NumberFormat("id-ID").format(value);
      this.annualIncome = parseInt(value) || 0;
    },
    calculateTax() {
      let remainingIncome = this.annualIncome;
      let lastMax = 0;
      this.taxResults = [];

      for (let bracket of this.taxBrackets) {
        if (remainingIncome <= 0) break;

        let taxableIncome = Math.min(remainingIncome, bracket.max - lastMax);
        let tax = taxableIncome * (bracket.rate / 100);

        if (taxableIncome > 0) {
          this.taxResults.push({
            rate: bracket.rate,
            amount: tax,
            max: bracket.max,
            min: lastMax,
            color: bracket.color,
          });
        }

        remainingIncome -= taxableIncome;
        lastMax = bracket.max;
      }
    },
    formatCurrency(amount) {
      return new Intl.NumberFormat("id-ID", {
        style: "currency",
        currency: "IDR",
      }).format(amount);
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&family=JetBrains+Mono:wght@500;600;700&display=swap");

/* ===== Global / page background ===== */
body {
  font-family: "Plus Jakarta Sans", Arial, sans-serif;
  margin: 0;
  padding: 0;
  min-height: 100%;
  display: flex;
  justify-content: center;
  background-color: #eef1ef;
}

/* ===== Design tokens ===== */
.container {
  --bg: #ffffff;
  --surface: #f4f6f5;
  --text: #1c2521;
  --text-muted: #6b7570;
  --border: #e3e7e4;
  --primary: #0d6e4f;
  --primary-dark: #0a5640;
  --primary-light: #e7f6ee;
  --accent: #c8932a;
  --accent-dark: #946c1c;
  --accent-light: #fbf1de;
  --radius-lg: 18px;
  --radius-md: 12px;
  --shadow:
    0 10px 28px -12px rgba(13, 110, 79, 0.22), 0 2px 8px rgba(0, 0, 0, 0.05);

  font-family: "Plus Jakarta Sans", Arial, sans-serif;
  width: 100%;
  max-width: 460px;
  margin: 16px auto;
  padding: 18px 16px 22px;
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow);
  background-color: var(--bg);
  color: var(--text);
  transition:
    background-color 0.25s ease,
    color 0.25s ease,
    border-color 0.25s ease;
  box-sizing: border-box;
}

.container.dark-mode {
  --bg: #161b1d;
  --surface: #1d2326;
  --text: #eef1ee;
  --text-muted: #98a19c;
  --border: #2b3235;
  --primary: #34c281;
  --primary-dark: #279764;
  --primary-light: rgba(52, 194, 129, 0.12);
  --accent: #e3ad4a;
  --accent-dark: #f0c267;
  --accent-light: rgba(227, 173, 74, 0.12);
  --shadow: 0 10px 28px -10px rgba(0, 0, 0, 0.5);
}

/* ===== Top bar ===== */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
  gap: 10px;
}

.app-title {
  display: flex;
  align-items: center;
  gap: 8px;
}

.app-icon {
  font-size: 22px;
  line-height: 1;
}

h1 {
  margin: 0;
  font-size: 18px;
  font-weight: 800;
  letter-spacing: -0.01em;
  color: var(--text);
}

.theme-toggle {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 17px;
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 50%;
  cursor: pointer;
  transition:
    background-color 0.2s ease,
    transform 0.15s ease;
}

.theme-toggle:hover {
  background-color: var(--surface);
}

.theme-toggle:active {
  transform: scale(0.92);
}

.theme-toggle:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

/* ===== Form ===== */
.form-group {
  margin-bottom: 14px;
}

label {
  display: block;
  margin-bottom: 6px;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-muted);
}

.input-wrap {
  display: flex;
  align-items: center;
  border: 1.5px solid var(--border);
  border-radius: var(--radius-md);
  background-color: var(--bg);
  overflow: hidden;
  transition: border-color 0.2s ease;
}

.input-wrap:focus-within {
  border-color: var(--primary);
}

.input-prefix {
  padding: 0 2px 0 14px;
  font-family: "JetBrains Mono", monospace;
  font-weight: 600;
  font-size: 15px;
  color: var(--text-muted);
  user-select: none;
}

.input-wrap input {
  flex: 1;
  width: 100%;
  border: none;
  outline: none;
  background: transparent;
  padding: 12px 14px 12px 6px;
  font-family: "JetBrains Mono", monospace;
  font-size: 16px;
  color: var(--text);
}

/* ===== Button ===== */
.calc-btn {
  width: 100%;
  min-height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 16px;
  font-weight: 700;
  color: #ffffff;
  background-color: var(--primary);
  border: none;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition:
    background-color 0.2s ease,
    transform 0.12s ease;
}

.calc-btn:hover {
  background-color: var(--primary-dark);
}

.calc-btn:active {
  transform: scale(0.98);
}

.calc-btn:focus-visible {
  outline: 3px solid var(--accent);
  outline-offset: 2px;
}

/* ===== Design credit ===== */
.design {
  font-size: 11px;
  text-align: right;
  margin: 18px 0 0;
  color: var(--text-muted);
}

.design a {
  color: var(--primary);
  text-decoration: none;
  font-weight: 600;
}

/* ===== Empty state ===== */
.empty-hint {
  margin-top: 16px;
  font-size: 13px;
  line-height: 1.55;
  text-align: center;
  color: var(--text-muted);
  padding: 0 6px;
}

/* ===== Results ===== */
.results {
  margin-top: 18px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  animation: fadeInUp 0.35s ease both;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(6px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.section {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 14px;
}

.eyebrow {
  display: block;
  font-size: 11.5px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: var(--text-muted);
  margin-bottom: 12px;
}

.bracket-bar {
  display: flex;
  height: 10px;
  width: 100%;
  border-radius: 999px;
  overflow: hidden;
  background-color: var(--border);
  margin-bottom: 14px;
}

.bracket-segment {
  height: 100%;
  transition: width 0.4s ease;
}

.bracket-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.bracket-list li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 7px 0;
  border-bottom: 1px solid var(--border);
}

.bracket-list li:last-child {
  border-bottom: none;
  padding-bottom: 0;
}

.dot {
  flex-shrink: 0;
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.bracket-text {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.bracket-name {
  font-size: 13.5px;
  font-weight: 600;
  color: var(--text);
}

.bracket-range {
  font-size: 11.5px;
  color: var(--text-muted);
}

.bracket-amount {
  font-family: "JetBrains Mono", monospace;
  font-weight: 600;
  font-size: 13.5px;
  white-space: nowrap;
  color: var(--text);
}

/* ===== Perbandingan PPh 21 vs PT ===== */
.compare-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

.compare-card {
  display: flex;
  flex-direction: column;
  gap: 8px;
  border-radius: var(--radius-lg);
  padding: 16px;
  border: 1px solid var(--border);
}

.compare-card.individu {
  background-color: var(--primary-light);
  border-color: var(--primary);
}

.compare-card.badan {
  background-color: var(--accent-light);
  border-color: var(--accent);
}

.compare-title {
  font-size: 13px;
  font-weight: 700;
  margin-bottom: 2px;
}

.compare-card.individu .compare-title {
  color: var(--primary-dark);
}

.compare-card.badan .compare-title {
  color: var(--accent-dark);
}

.compare-row {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 8px;
  font-size: 13.5px;
  color: var(--text);
}

.compare-row strong {
  font-family: "JetBrains Mono", monospace;
  font-size: 15px;
  font-weight: 700;
  white-space: nowrap;
}

.compare-card.individu .compare-row strong {
  color: var(--primary-dark);
}

.compare-card.badan .compare-row strong {
  color: var(--accent-dark);
}

.compare-sub {
  margin-top: 2px;
  font-size: 11.5px;
  color: var(--text-muted);
}

/* ===== Mobile-first: refine for larger screens ===== */
@media (min-width: 480px) {
  .container {
    margin: 32px auto;
    padding: 24px 26px 28px;
  }

  h1 {
    font-size: 19px;
  }
}
</style>
