<script setup lang="ts">
import yaml from 'js-yaml'
import { flatten, unflatten } from 'flat'
import { stringify } from 'csv-stringify/sync'
import { parse } from 'csv-parse/sync'

const files = ref<File[]>([])
const parsedFiles = ref<string[]>([])
const yamlContent = ref<string[]>([])

const fileNames = computed(() => files.value.map(file => file.name))
const activeIndex = ref(0)
const activeParsedFile = computed(() => parsedFiles.value[activeIndex.value])
const activeYamlContent = computed(() => yamlContent.value[activeIndex.value])

const onSelectFile = (e: any) => {
  const newFiles = Array.from(e.target.files) as File[]
  newFiles.forEach((file: any) => {
    const reader = new FileReader()
    reader.onload = () => {
      // Read as text
      const text = reader.result as string
      // Store to parsedFiles
      parsedFiles.value.push(text)
      // Push file to file list
      files.value.push(file)
      // Parse YAML to Object & flatten it
      const content = parse(text as string, {
        columns: false,
        skipEmptyLines: true,
      })
      // eslint-disable-next-line no-sequences
      const flattenedContent = content.reduce((result: any, current: any) => (result[current[0]] = current[1], result), {})
      const unflattened = unflatten(flattenedContent)
      const parsedContent = yaml.dump(unflattened, {
        quotingType: '"',
        noCompatMode: true,
      })
      yamlContent.value.push(parsedContent)
    }
    reader.readAsText(file)
  })
}

const downloadYml = () => {
  const element = document.createElement('a')
  element.setAttribute('href', `data:text/plain;charset=utf-8,${encodeURIComponent(activeYamlContent.value)}`)
  element.setAttribute('download', `${fileNames.value[activeIndex.value].split('.')[0]}.yml`)

  element.style.display = 'none'
  document.body.appendChild(element)

  element.click()

  document.body.removeChild(element)
}

</script>

<template>
  <div class="container mx-auto p-8 h-full">
    <h1 class="font-bold text-xl text-gray-700">
      CSV TO YAML
    </h1>
    <div class="grid gap-4 grid-cols-2 p-8 bg-light-blue-300 rounded-xl h-full">
      <div>
        <div class="h-150px">
          <input
            :key="files.length"
            accept=".csv"
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

        <textarea class="w-full" :value="activeParsedFile" readonly rows="30" />
      </div>
      <div>
        <div class="h-150px flex items-end justify-end py-4">
          <button
            :disabled="!activeYamlContent"
            class="px-4 py-2 bg-light-400 disabled:bg-light-900 rounded"
            @click="downloadYml"
          >
            Download CSV
          </button>
        </div>
        <textarea class="w-full" :value="activeYamlContent" readonly rows="30" />
      </div>
    </div>
  </div>
</template>
