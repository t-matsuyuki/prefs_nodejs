<template>
  <table class="population-table">
    <thead>
      <tr>
        <th>年</th>
        <th v-for="prefecture in prefectures" :key="prefecture.prefCode">{{ prefecture.prefName }}</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="yearData in sortedYearData" :key="yearData.year">
        <td>{{ yearData.year }}</td>
        <td v-for="prefecture in yearData.population" :key="prefecture.prefCode">
          {{ prefecture.value }}
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import styles from '../css/PopulationTable.css'
export default {
  props: {
    populationData: {
      type: Array,
      required: true
    },
    prefectures: {
      type: Array,
      required: true
    }
  },
  computed: {
    sortedYearData() {
      const sortedData = [...this.populationData] // populationDataをコピー
      sortedData.sort((a, b) => b.year - a.year) // 年の降順でソート
      return sortedData;
    },
    populationTable() {
      return styles['population-table']
    }
  }
}
</script>
<style>
.population-table {
  width: 100%;
  border-collapse: collapse;
}

.population-table th,
.population-table td {
  padding: 8px;
  text-align: center;
  border: 1px solid #ccc;
}

.population-table th {
  background-color: #f0f0f0;
  font-weight: bold;
}

.population-table td {
  background-color: #ffffff;
}

.population-table tbody tr:nth-child(even) td {
  background-color: #f8f8f8;
}

.population-table tbody tr:hover td {
  background-color: #eaf2f8;
}
</style>
