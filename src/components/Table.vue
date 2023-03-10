<template>
  <div>
       <div class="_container">
          <div class="table">
            <div class="table__header">
              <div class="table__header_column" :class="{table__header_active: sortedByColumn === 0}">Id тега</div>
              <div class="table__header_column">
                <input class="table__header_search-field" v-model="searchField" placeholder="Фильтр по имени">
                Имя тега
              </div>
              <div
                  class="table__header_column table__header_attribute"
                  :class="{table__header_active: sortedByColumn === i}"
                  v-for="i in numberOfAttributeColumns"
                  @click="sortByColumn(i)"
              >
                Атрибут {{i}}
                <div
                    class="_symbol"
                    :class="{
                  _minus: i !== sortedByColumn || sortOrder === 0  ,
                  _arrowDown: i === sortedByColumn && sortOrder === 1,
                  _arrowUp: i === sortedByColumn && sortOrder === -1
                 }">
                </div>
              </div>
            </div>
            <div
                class="table__row"
                v-for="data in filteredList"
            >
              <div class="table__column">{{data.tag_id}}</div>
              <div class="table__column">{{data.tag_name}}</div>
              <div class="table__column table__column_attribute" v-for="i in numberOfAttributeColumns">
                <p v-if="data.tag_attributes[i-1]">
                  {{ data.tag_attributes[i - 1].attribute_value }}
                </p>
              </div>
            </div>
          </div>
        </div>
  </div>
</template>

<script>
import dataInfo from "@/data/data.json"

export default {
  mounted() {
    this.sortByTagId()
  },

  data() {
    return {
      dataInfo: dataInfo.result,
      dataInfoFiltered: '',
      searchField: '',
      numberOfAttributeColumns: 3,
      sortedByColumn: 0, // 0 - по tag_id; 1, 2, 3... - колонки атрибута
      sortOrder: 0 // 1 - по возрастанию; -1 - по убыванию; 0 - по tag_id
    }
  },

  methods: {
    sortByTagId() {
      this.dataInfo.sort((a, b) => a.tag_id - b.tag_id)
    },

    sortByColumn(column) {
      if (this.sortedByColumn === column) {
      this.sortOrder = !this.sortOrder ? 1 : this.sortOrder === 1 ? -1 : 0
      } else {
        this.sortOrder = 1
        this.sortedByColumn = column
      }

      this.dataInfo.sort((first, second) => {
        const a = !!first.tag_attributes[column-1] ? first.tag_attributes[column-1].attribute_value : ''
        const b = !!second.tag_attributes[column-1] ? second.tag_attributes[column-1].attribute_value : ''
        return (a === b ? 0 : a > b ? 1 : -1) * this.sortOrder
    })
    }
  },

  watch: {
    sortOrder: function () {
      if (!this.sortOrder) {
        this.sortByTagId()
        this.sortedByColumn = 0
      }
    },

    dataInfo: function () {
      if (!!this.dataInfo.length) {
        this.numberOfAttributeColumns =
            this.dataInfo
                .map(i => i.tag_attributes.length)
                .reduce((prev, curr) => prev > curr ? prev : curr)
        }
      }
    },

  computed: {
    filteredList: function () {
      this.dataInfoFiltered = this.dataInfo.filter((tag) => {
          return (
              tag.tag_name
              .toLowerCase()
              .indexOf(this.searchField.toLowerCase()) > -1
          )
      })
      return this.dataInfoFiltered
    },
  }
}

</script>

<style scoped>

._symbol {
  position: absolute;
  right: 15px;
}

._arrowUp,
._arrowDown {
  border: 0 solid transparent;
}
._arrowUp {
  border-right-width: 5px;
  border-left-width: 5px;
  border-bottom: 10px solid black;
}

._arrowDown {
  border-left-width: 5px;
  border-right-width: 5px;
  border-top: 10px solid black;
}

._minus {
  border: 0 solid black;
  border-left-width: 5px;
  border-right-width: 5px;
  border-top: 2px solid black;
}

._container {
  display: flex;
  padding: 0 30px;
  margin: 0 auto;
  width: 1000px;
}

.table {
  display: inline-block;
  border: 1px solid black;
}

.table__header,
.table__row
{
  position: relative;
  display: flex;
}

.table__header {
  background-color: #a3be4d;
}

.table__header_attribute {
  cursor: pointer;
}

.table__header_active {
  background-color: #bae538;
}

.table__header_search-field {
  position: absolute;
  top: -30px;
}

.table__header_column,
.table__column {
  user-select: none;
  position: relative;
  display: flex;
  flex-grow: 0;
  justify-content: center;
  align-items: center;
  border: 1px solid black;
  width: 150px;
  height: 50px;
}
</style>