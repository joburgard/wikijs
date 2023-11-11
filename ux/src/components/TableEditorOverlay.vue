<template lang="pug">
q-layout(view='hHh lpR fFf', container)
  q-header.card-header.q-px-md.q-py-sm
    q-icon(name='img:/_assets/icons/color-data-grid.svg', left, size='md')
    span {{t(`editor.tableEditor.title`)}}
    q-space
    q-btn.q-mr-sm(
      flat
      rounded
      color='white'
      :aria-label='t(`common.actions.refresh`)'
      icon='las la-question-circle'
      :href='siteStore.docsBase + `/admin/editors/markdown`'
      target='_blank'
      type='a'
    )
    q-btn-group(push)
      q-btn(
        push
        color='white'
        text-color='grey-7'
        :label='t(`common.actions.cancel`)'
        :aria-label='t(`common.actions.cancel`)'
        icon='las la-times'
        @click='close'
      )
      q-btn(
        push
        color='positive'
        text-color='white'
        :label='t(`common.actions.save`)'
        :aria-label='t(`common.actions.save`)'
        icon='las la-check'
        :disabled='state.loading > 0'
        @click='save'
      )
  q-page-container
    q-page(padding)
      div(class='row nowrap justify-center')
        div
          div
            q-btn-group(push)
              q-btn(
                push
                color='white'
                text-color='grey-7'
                icon='mdi-table-column-plus-after'
                @click='addColumn'
              )
              q-btn(
                push
                color='white'
                text-color='grey-7'
                icon='mdi-table-row-plus-after'
                @click='addRow'
              )
              q-btn(
                push
                color='white'
                text-color='grey-7'
                icon='mdi-table-column-remove'
              )
              q-btn(
                push
                color='white'
                text-color='grey-7'
                icon='mdi-table-row-remove'
              )
          div.q-mt-md(ref='tblRef')

      q-inner-loading(:showing='state.loading > 0')
        q-spinner(color='accent', size='lg')
</template>

<script setup>
import { useQuasar } from 'quasar'
import { EditModule, FormatModule, FrozenColumnsModule, MoveColumnsModule, MoveRowsModule, Tabulator } from 'tabulator-tables'
import { onMounted, reactive, ref } from 'vue'
import { useI18n } from 'vue-i18n'

import { useSiteStore } from 'src/stores/site'

import 'tabulator-tables/dist/css/tabulator.css'

// QUASAR

const $q = useQuasar()

// STORES

const siteStore = useSiteStore()

// I18N

const { t } = useI18n()

// DATA

const state = reactive({
  loading: 0
})
const tblRef = ref(null)
let table

// METHODS

function addColumn () {
  table.addColumn({ title: 'New', field: 'new', editor: 'input', editableTitle: true })
}

function addRow () {
  table.addRow({})
}

function close () {
  siteStore.$patch({ overlay: '' })
}

function save () {
  const columns = table.getColumns()

  let columnTitles = ''
  let columnBars = ''
  const columnFields = []
  for (const columnProxy of columns) {
    const column = columnProxy.getDefinition()

    if (column.title) {
      columnTitles += `|${column.title}`
      columnBars += '|-'
      columnFields.push(column.field)
    }
  }

  const data = table.getData()
  let rows = ''
  for (const item of data) {
    const values = columnFields.map(field => item[field]).join('|')
    rows += `\n|${values}|`
  }

  const markdown = `${columnTitles}|\n${columnBars}|${rows}`

  EVENT_BUS.emit('insertTableValue', markdown)
  close()
}

onMounted(() => {
  const tabledata = [
    { id: 1, name: 'Oli Bob', progress: 12, gender: 'male', rating: 1, col: 'red', dob: '19/02/1984', car: 1 },
    { id: 2, name: 'Mary May', progress: 1, gender: 'female', rating: 2, col: 'blue', dob: '14/05/1982', car: true },
    { id: 3, name: 'Christine Lobowski', progress: 42, gender: 'female', rating: 0, col: 'green', dob: '22/05/1982', car: 'true' },
    { id: 4, name: 'Brendon Philips', progress: 100, gender: 'male', rating: 1, col: 'orange', dob: '01/08/1980' },
    { id: 5, name: 'Margret Marmajuke', progress: 16, gender: 'female', rating: 5, col: 'yellow', dob: '31/01/1999' },
    { id: 6, name: 'Frank Harbours', progress: 38, gender: 'male', rating: 4, col: 'red', dob: '12/05/1966', car: 1 }
  ]

  Tabulator.registerModule([EditModule, MoveRowsModule, MoveColumnsModule, FormatModule, FrozenColumnsModule])

  table = new Tabulator(tblRef.value, {
    data: tabledata,
    clipboard: true,
    height: '100%',
    autoResize: true,
    layout: 'fitDataTable',
    movableColumns: true,
    movableRows: true,
    columns: [
      { rowHandle: true, formatter: 'handle', headerSort: false, frozen: true, width: 30, minWidth: 30 },
      { title: 'id', field: 'id', editor: 'input', editableTitle: true },
      { title: 'Name', field: 'name', editor: 'input', validator: 'required', editableTitle: true }
    ]
  })
})
</script>
