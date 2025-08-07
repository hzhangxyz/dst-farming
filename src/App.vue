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

const 筛选 = reactive({
  春季: false,
  夏季: false,
  秋季: false,
  冬季: false,
})

const selectedPlants = computed((): Plant[] =>
  plants.filter((plant: Plant): boolean => {
    if (筛选.春季 && !plant.春季) return false
    if (筛选.夏季 && !plant.夏季) return false
    if (筛选.秋季 && !plant.秋季) return false
    if (筛选.冬季 && !plant.冬季) return false
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
  筛选.春季 = spring
  筛选.夏季 = summer
  筛选.秋季 = autumn
  筛选.冬季 = winter
}
</script>

<template>
  <h1>饥荒耕种配比表</h1>
  <h2>快速选择</h2>
  <button @click="easySelect(false, false, false, false)">清除筛选</button>
  <button @click="easySelect(true, false, false, false)">切换春季</button>
  <button @click="easySelect(false, true, false, false)">切换夏季</button>
  <button @click="easySelect(false, false, true, false)">切换秋季</button>
  <button @click="easySelect(false, false, false, true)">切换冬季</button>
  <button @click="easySelect(true, true, false, false)">切换春夏</button>
  <button @click="easySelect(false, true, true, false)">切换夏秋</button>
  <button @click="easySelect(false, false, true, true)">切换秋冬</button>
  <button @click="easySelect(true, false, false, true)">切换冬春</button>
  <h2>季节选择</h2>
  <label> <input type="checkbox" v-model="筛选.春季" /> 春季 </label>
  <label> <input type="checkbox" v-model="筛选.夏季" /> 夏季 </label>
  <label> <input type="checkbox" v-model="筛选.秋季" /> 秋季 </label>
  <label> <input type="checkbox" v-model="筛选.冬季" /> 冬季 </label>
  <h2>有效植物</h2>
  <table>
    <thead>
      <tr>
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
        <td>{{ plant.名称 }}</td>
        <td>{{ plant.春季 ? '是' : '否' }}</td>
        <td>{{ plant.夏季 ? '是' : '否' }}</td>
        <td>{{ plant.秋季 ? '是' : '否' }}</td>
        <td>{{ plant.冬季 ? '是' : '否' }}</td>
        <td>{{ plant.催长剂 }}</td>
        <td>{{ plant.堆肥 }}</td>
        <td>{{ plant.粪肥 }}</td>
      </tr>
    </tbody>
  </table>
  <h2>有效方案</h2>
  <ul>
    <li v-for="plan in possiblePlan" :key="plan">{{ plan }}</li>
  </ul>
</template>

<style scoped></style>
