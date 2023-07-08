<template>
  <div>
    <h1>人口データ表</h1>
    <PopulationTable :populationData="populationData" :prefectures="prefectures" v-if="populationData.length > 0 && prefectures.length > 0" />
    <div v-else>
      データを取得中...
    </div>
  </div>
</template>

<script>
import PopulationTable from './components/PopulationTable.vue'
const TARGET_PFEFCODE_MIN = 8
const TARGET_PREFCODE_MAX = 14
const API_URL1 = 'https://opendata.resas-portal.go.jp/api/v1/prefectures'
const API_URL2 = 'https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?prefCode=[prefCode]&cityCode=-&addArea='
const headers  = {'X-API-KEY' : 'qC6sF6F9ylXndYA1deenJIaXKE36W812ZEKii8yt'}

export default {
  components: {
    PopulationTable
  },
  data() {
    return {
      populationData: [],
      prefectures: []
    }
  },
  mounted() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      let prefData = []
      // 都道府県データを取得
      fetch(API_URL1, {
        headers: headers
      })
        .then(response => response.json())
        .then(data => {
          prefData = data.result.filter(e => {
            return parseInt(e['prefCode'],10) >= TARGET_PFEFCODE_MIN && parseInt(e['prefCode'], 10) <= TARGET_PREFCODE_MAX
          })
          this.prefectures = prefData
          // console.log(`prefectures = ${JSON.stringify(this.prefectures)}`)

          // 対象都道府県分処理を繰り返す
          const fetchPopulationData = prefData.map(pref => {
            // 人口データを取得
            return fetch(API_URL2.replace('[prefCode]', pref.prefCode), {
              headers: headers
            })
            .then(response => response.json())
            .then(data => {
              const population = data.result.data.find(
                d => {
                  if(d.label === '総人口') {
                    return d.data
                  }
                }
              )
              return {prefCode: pref.prefCode, population: population.data}
            })
            .catch(error => {
              console.error('Error fetching population data:', error)
              return null
            })
          })

          Promise.all(fetchPopulationData)
            .then(populationData => {
              this.groupingYear(populationData)
              this.populationData = this.convertData(populationData.filter(data => data !== null))
            })
            .catch(error => {
              console.error('Error fetching population data:', error)
            })
        })
        .catch(error => {
          console.error('Error fetching prefectures:', error)
        })
    },
    groupingYear(data){
      // console.log(`groupingYear data = ${JSON.stringify(data)}`)
      return data.map(d => {
        return d
      })
    },
    convertData(inputData) {
      const convertedData
       = []

      // 年ごとにデータを変換
      inputData.forEach(({ prefCode, population }) => {
        population.forEach(({ year, value }) => {
          const existingYearData = convertedData
          .find(data => data.year === year)

          if (existingYearData) {
            existingYearData.population.push({
              prefCode,
              value
            })
          } else {
            convertedData
            .push({
              year,
              population: [{ prefCode, value }] // 最初の都道府県データ
            })
          }
        })
      })

      return convertedData
      
    }
  }
}
</script>
