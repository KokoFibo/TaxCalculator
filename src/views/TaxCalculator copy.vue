<template>
  <div class="container">
    <h1>Kalkulator Pajak PPh 21</h1>
    <div class="form-group">
      <label for="income"
        >Penghasilan Tahunan (Rp) Setelah di potong PTKP:</label
      >
      <input
        type="text"
        id="income"
        v-model="formattedIncome"
        @input="formatInput"
        placeholder="Masukkan penghasilan tahunan"
      />
    </div>

    <button @click="calculateTax">Hitung Pajak</button>

    <div v-if="taxResults.length > 0" class="results">
      <h2>Hasil Perhitungan:</h2>
      <ul>
        <li v-for="(result, index) in taxResults" :key="index">
          <strong
            >Lapisan {{ index + 1 }} (hingga
            {{ formatCurrency(result.max) }}):</strong
          >
          {{ formatCurrency(result.amount) }} ({{ result.rate }}%)
        </li>
      </ul>
      <h3>Total Pajak: {{ formatCurrency(totalTax) }}</h3>
      <h4>Persentase Pajak dari Penghasilan: {{ totalTaxPercentage }}%</h4>
    </div>
  </div>
</template>

<script>
export default {
  name: "Pph21Calculator",
  data() {
    return {
      annualIncome: 0,
      formattedIncome: "",
      taxBrackets: [
        { max: 60000000, rate: 5 }, // Tarif 5% untuk pendapatan <= 60 juta
        { max: 250000000, rate: 15 }, // Tarif 15% untuk pendapatan <= 250 juta
        { max: 500000000, rate: 25 }, // Tarif 25% untuk pendapatan <= 500 juta
        { max: 5000000000, rate: 30 }, // Tarif 30% untuk pendapatan <= 5 M
        { max: Infinity, rate: 35 }, // Tarif 35% untuk penghasilan > 5 M
      ],
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
  },
  methods: {
    formatInput(event) {
      // Menghapus semua karakter selain angka
      let value = event.target.value.replace(/[^0-9]/g, "");

      // Memperbarui formattedIncome dengan format comma
      this.formattedIncome = new Intl.NumberFormat("id-ID").format(value);

      // Mengubah string kembali ke angka untuk perhitungan
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
            max: bracket.max, // Tambahkan keterangan batas (bracket)
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
.container {
  max-width: 500px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
h1,
h2,
h3,
h4 {
  text-align: center;
}
.form-group {
  margin-bottom: 20px;
}
input {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  margin-top: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  width: 100%;
  padding: 10px;
  font-size: 18px;
  color: white;
  background-color: #28a745;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:hover {
  background-color: #218838;
}
.results {
  margin-top: 20px;
}
</style>
