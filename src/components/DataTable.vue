<template>
  <div class="mt-4">
    <table v-if="table" class="text-sm text-left text-gray-500 dark:text-gray-400">
      <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
        <tr v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
          <th
            v-for="header in headerGroup.headers"
            :key="header.id"
            scope="col"
            class="px-6 py-3 text-left text-sm font-semibold"
            :class="{
              'cursor-pointer select-none': header.column.getCanSort(),
            }"
            @click="header.column.getToggleSortingHandler()?.($event)"
          >
            <FlexRender :render="header.column.columnDef.header" :props="header.getContext()" />
            {{ getSortArrow(header.column.getIsSorted()) }}
          </th>
        </tr>
      </thead>
      <tbody class="divide-y divide-gray-200">
        <tr
          v-for="row in table.getRowModel().rows"
          :key="row.id"
          class="bg-white border-b dark:bg-gray-800 dark:border-gray-700"
        >
          <td
            v-for="cell in row.getVisibleCells()"
            :key="cell.id"
            class="whitespace-nowrap text-sm px-6 py-4 cursor-pointer"
            @click="copyToClipboard(cell.getValue())"
          >
            <FlexRender :render="cell.column.columnDef.cell" :props="cell.getContext()" />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script lang="ts">
const arrowMap = { asc: ' ↑', desc: ' ↓' }

export default {
  name: 'DataTable',
}
</script>

<script setup lang="ts">
import { ref, toRefs, watch } from 'vue'
import type { PropType } from 'vue'
import {
  useVueTable,
  FlexRender,
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  SortingState,
  Table,
  SortDirection,
} from '@tanstack/vue-table'
import { TColumn, TData } from '../types'

const props = defineProps({
  rows: {
    type: Array as PropType<TData[]>,
    required: true,
  },
  columns: {
    type: Array as PropType<TColumn[]>,
    required: true,
  },
})

const { rows, columns } = toRefs(props)
const emits = defineEmits(['copy'])

const sorting = ref<SortingState>([])
const filter = ref('')

const table = ref<Table<TData>>()

const getSortArrow = (sorted: boolean | SortDirection) => {
  if (typeof sorted === 'boolean') return ''
  return arrowMap[sorted]
}

const copyToClipboard = (text: string | unknown) => {
  if (!navigator.clipboard) return
  if (!text || typeof text !== 'string') return
  navigator.clipboard.writeText(text)
  emits('copy')
}

watch(
  [rows, columns],
  () => {
    table.value = useVueTable({
      data: rows.value,
      columns: columns.value,
      getCoreRowModel: getCoreRowModel(),
      getSortedRowModel: getSortedRowModel(),
      getFilteredRowModel: getFilteredRowModel(),
      state: {
        get sorting() {
          return sorting.value
        },
        get globalFilter() {
          return filter.value
        },
      },
      onSortingChange: updaterOrValue => {
        sorting.value = typeof updaterOrValue === 'function' ? updaterOrValue(sorting.value) : updaterOrValue
      },
    })
  },
  { immediate: true },
)
</script>
