<script setup lang="ts">
import rawData from './data.json'
interface Plant {
  名称: string
  春季: boolean
  夏季: boolean
  秋季: boolean
  冬季: boolean
  催长剂: number
  堆肥: number
  粪肥: number
}
const plants = rawData as Plant[]

import { reactive, computed } from 'vue'

const selected = reactive({
  春季: false,
  夏季: false,
  秋季: false,
  冬季: false,
})

const selectedPlants = computed((): Plant[] =>
  plants.filter((plant: Plant): boolean => {
    if (selected.春季 && !plant.春季) return false
    if (selected.夏季 && !plant.夏季) return false
    if (selected.秋季 && !plant.秋季) return false
    if (selected.冬季 && !plant.冬季) return false
    return true
  }),
)

const possiblePlan2 = computed((): string[] => {
  let result = []
  const possibleRatio = [
    [1, 1],
    [2, 1],
    [1, 2],
  ]
  for (const [ia, a] of selectedPlants.value.entries()) {
    for (const [ib, b] of selectedPlants.value.entries()) {
      if (existence[a.名称] === false || existence[b.名称] === false) continue
      if (ia >= ib) continue
      const vectorA = [a.催长剂, a.堆肥, a.粪肥]
      const vectorB = [b.催长剂, b.堆肥, b.粪肥]
      for (const ratio of possibleRatio) {
        const total = Array.from(
          { length: 3 },
          (_, i) => vectorA[i] * ratio[0] + vectorB[i] * ratio[1],
        )
        if (total.every((v) => v == 0)) {
          result.push(`${ratio[0]}${a.名称} + ${ratio[1]}${b.名称}`)
        }
      }
    }
  }
  return result
})

const possiblePlan3 = computed((): string[] => {
  let result = []
  const possibleRatio = [
    [1, 1, 1],
    [2, 1, 1],
    [1, 2, 1],
    [1, 1, 2],
    [1, 2, 2],
    [2, 1, 2],
    [2, 2, 1],
  ]
  for (const [ia, a] of selectedPlants.value.entries()) {
    for (const [ib, b] of selectedPlants.value.entries()) {
      for (const [ic, c] of selectedPlants.value.entries()) {
        if (
          existence[a.名称] === false ||
          existence[b.名称] === false ||
          existence[c.名称] === false
        )
          continue
        if (ia >= ib || ib >= ic) continue
        const vectorA = [a.催长剂, a.堆肥, a.粪肥]
        const vectorB = [b.催长剂, b.堆肥, b.粪肥]
        const vectorC = [c.催长剂, c.堆肥, c.粪肥]
        for (const ratio of possibleRatio) {
          const total = Array.from(
            { length: 3 },
            (_, i) => vectorA[i] * ratio[0] + vectorB[i] * ratio[1] + vectorC[i] * ratio[2],
          )
          if (total.every((v) => v == 0)) {
            result.push(`${ratio[0]}${a.名称} + ${ratio[1]}${b.名称} + ${ratio[2]}${c.名称}`)
          }
        }
      }
    }
  }
  return result
})

const possiblePlan = computed((): string[] => [...possiblePlan2.value, ...possiblePlan3.value])

function easySelect(spring: boolean, summer: boolean, autumn: boolean, winter: boolean): void {
  selected.春季 = spring
  selected.夏季 = summer
  selected.秋季 = autumn
  selected.冬季 = winter
}

const existence = reactive(
  plants.reduce(
    (result, plant) => {
      result[plant.名称] = true
      return result
    },
    {} as Record<string, boolean>,
  ),
)

function formatWithSign(num: number): string {
  return `${num >= 0 ? '+' : ''}${num}`
}
</script>

<template>
  <div class="container">
    <h1>饥荒耕种配比表</h1>
    <h2>快速选择</h2>
    <div class="button-group">
      <button @click="easySelect(false, false, false, false)" class="btn btn-clear">
        清除筛选
      </button>
      <button @click="easySelect(false, false, true, false)" class="btn btn-season">
        切换秋季
      </button>
      <button @click="easySelect(false, false, true, true)" class="btn btn-two-season">
        切换秋冬
      </button>
      <button @click="easySelect(false, false, false, true)" class="btn btn-season">
        切换冬季
      </button>
      <button @click="easySelect(true, false, false, true)" class="btn btn-two-season">
        切换冬春
      </button>
      <button @click="easySelect(true, false, false, false)" class="btn btn-season">
        切换春季
      </button>
      <button @click="easySelect(true, true, false, false)" class="btn btn-two-season">
        切换春夏
      </button>
      <button @click="easySelect(false, true, false, false)" class="btn btn-season">
        切换夏季
      </button>
      <button @click="easySelect(false, true, true, false)" class="btn btn-two-season">
        切换夏秋
      </button>
    </div>

    <h2>季节选择</h2>
    <div class="checkbox-group">
      <label><input type="checkbox" v-model="selected.春季" /> 春季</label>
      <label><input type="checkbox" v-model="selected.夏季" /> 夏季</label>
      <label><input type="checkbox" v-model="selected.秋季" /> 秋季</label>
      <label><input type="checkbox" v-model="selected.冬季" /> 冬季</label>
    </div>

    <h2>有效植物</h2>
    <table class="plants-table">
      <thead>
        <tr>
          <th>存在</th>
          <th>名称</th>
          <th>春季</th>
          <th>夏季</th>
          <th>秋季</th>
          <th>冬季</th>
          <th>催长剂</th>
          <th>堆肥</th>
          <th>粪肥</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="plant in selectedPlants" :key="plant.名称">
          <td><input type="checkbox" v-model="existence[plant.名称]" /></td>
          <td>{{ plant.名称 }}</td>
          <td>{{ plant.春季 ? '是' : '否' }}</td>
          <td>{{ plant.夏季 ? '是' : '否' }}</td>
          <td>{{ plant.秋季 ? '是' : '否' }}</td>
          <td>{{ plant.冬季 ? '是' : '否' }}</td>
          <td>{{ formatWithSign(plant.催长剂) }}</td>
          <td>{{ formatWithSign(plant.堆肥) }}</td>
          <td>{{ formatWithSign(plant.粪肥) }}</td>
        </tr>
      </tbody>
    </table>

    <h2>有效方案</h2>
    <ul>
      <li v-for="plan in possiblePlan" :key="plan">{{ plan }}</li>
    </ul>
  </div>
</template>

<style scoped>
/* 基本样式 */
.container {
  font-family: 'Arial', sans-serif;
  background-color: #f4f7fa;
  color: #333;
  padding: 20px;
  margin: 0 auto;
  max-width: 1200px;
}

/* 标题样式 */
h1 {
  text-align: center;
  color: #5a9bcf;
}

h2 {
  color: #2c3e50;
  border-bottom: 2px solid #ecf0f1;
  padding-bottom: 5px;
  margin-bottom: 15px;
}

/* 按钮样式 */
.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.btn {
  background: linear-gradient(145deg, #6a99d6, #4a83c8);
  border: none;
  border-radius: 25px;
  padding: 10px 20px;
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.btn:hover {
  background: linear-gradient(145deg, #4a83c8, #6a99d6);
}

.btn-clear {
  background: #ff6f61;
}

.btn-season {
  background: #4bcf8f;
}

.btn-two-season {
  background: linear-gradient(145deg, #f39c12, #e67e22); /* 金黄色渐变 */
  box-shadow: 0 4px 8px rgba(255, 159, 0, 0.3); /* 光晕效果 */
}

/* 复选框样式 */
.checkbox-group {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.checkbox-group label {
  font-size: 16px;
}

/* 表格样式 */
.plants-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.plants-table th,
.plants-table td {
  padding: 10px;
  text-align: center;
  border: 1px solid #ecf0f1;
}

.plants-table th {
  background-color: #3498db;
  color: white;
}

.plants-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.plants-table tr:hover {
  background-color: #f1f1f1;
}

/* 列表样式 */
ul {
  padding-left: 20px;
}

ul li {
  background: #ecf0f1;
  padding: 10px;
  margin: 5px 0;
  border-radius: 5px;
}

ul li:hover {
  background: #dfe6e9;
}
</style>
