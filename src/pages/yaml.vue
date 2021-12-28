<script setup lang="ts">
import yaml from 'js-yaml'
import { flatten, unflatten } from 'flat'
import { stringify } from 'csv-stringify/sync'

const files = ref<File[]>([])
const parsedFiles = ref<string[]>([])
const flattenedYaml = ref<any[]>([])
const convertedCsv = ref<string[]>([])

const fileNames = computed(() => files.value.map(file => file.name))
const activeIndex = ref(0)
const activeYamlContent = computed(() => JSON.stringify(flattenedYaml.value[activeIndex.value], null, 2))
const activeCsvContent = computed(() => convertedCsv.value[activeIndex.value])

const onSelectFile = (e: any) => {
  const newFiles = Array.from(e.target.files) as File[]
  newFiles.forEach((file: any) => {
    const reader = new FileReader()
    reader.onload = () => {
      // Read as text
      const text = reader.result
      // Store to parsedFiles
      parsedFiles.value.push(text as string)
      // Push file to file list
      files.value.push(file)
      // Parse YAML to Object & flatten it
      const content = yaml.load(text as string)
      const flattenedContent = flatten(content) as object
      flattenedYaml.value.push(flattenedContent)
      // Convert to CSV
      const entries = Object.entries(flattenedContent)
      const csv = stringify(entries)
      convertedCsv.value.push(csv)
    }
    reader.readAsText(file)
  })
}

const downloadCsv = () => {
  const element = document.createElement('a')
  element.setAttribute('href', `data:text/plain;charset=utf-8,${encodeURIComponent(activeCsvContent.value)}`)
  element.setAttribute('download', `${fileNames.value[activeIndex.value].split('.')[0]}.csv`)

  element.style.display = 'none'
  document.body.appendChild(element)

  element.click()

  document.body.removeChild(element)
}

</script>

<template>
  <div class="container mx-auto p-8 h-full">
    <h1 class="font-bold text-xl text-gray-700">
      YAML TO CSV
    </h1>
    <div class="grid gap-4 grid-cols-2 p-8 bg-gray-300 rounded h-full">
      <div>
        <div class="h-150px">
          <input
            :key="files.length"
            accept=".yml"
            multiple
            type="file"
            @change="onSelectFile($event)"
          >

          <div class="py-8">
            <ul class="flex flex-wrap -mx-1 -my-1">
              <li
                v-for="(file, index) in fileNames"
                :key="file"
                class="cursor-pointer px-1 py-5"
                @click="activeIndex = index"
              >
                <span
                  :class="['rounded px-3 py-1', activeIndex === index ? 'bg-dark-300 text-white' : 'bg-light-400']"
                >{{ file }}</span>
              </li>
            </ul>
          </div>
        </div>

        <textarea class="w-full" :value="activeYamlContent" readonly rows="30" />
      </div>
      <div>
        <div class="h-150px flex items-end justify-end py-4">
          <button
            :disabled="!activeCsvContent"
            class="px-4 py-2 bg-light-400 disabled:bg-light-900 rounded"
            @click="downloadCsv"
          >
            Download CSV
          </button>
        </div>
        <textarea class="w-full" :value="activeCsvContent" readonly rows="30" />
      </div>
    </div>
  </div>
</template>
