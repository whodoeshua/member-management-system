<template>
<el-table-column :prop="prop" v-bind="$attrs">
  <template slot-scope="scope">
      <span @click.prevent="toggleHandle(scope.$index, scope.row)" :style="childStyles(scope.row)">
        <i :class="iconClasses(scope.row)" :style="iconStyles(scope.row)"></i>
        {{ scope.row[prop] }}
      </span>
    </template>
</el-table-column>
</template>

<script>
import isArray from 'lodash/isArray'
export default {
  name: 'table-tree-column',
  props: {
    prop: {
      type: String
    },
    treeKey: {
      type: String,
      default: 'id'
    },
    parentKey: {
      type: String,
      default: 'parentId'
    },
    levelKey: {
      type: String,
      default: '_level'
    },
    childKey: {
      type: String,
      default: 'children'
    }
  },
  methods: {
    childStyles(row) {
      return {
        'padding-left': (row[this.levelKey] > 1 ? row[this.levelKey] * 15 : 0) + 'px'
      }
    },
    iconClasses(row) {
      return [!row._expanded ? 'el-icon-caret-right' : 'el-icon-caret-bottom']
    },
    iconStyles(row) {
      return [{
          'visibility': this.hasChild(row) || row.haveChildBom ? 'visible' : 'hidden'
        },
        {
          'margin-right': '8px'
        }
      ]
    },
    hasChild(row) {
      return (isArray(row[this.childKey]) && row[this.childKey].length >= 1) || false
    },
    // 切换处理
    toggleHandle(index, row) {
      if (this.hasChild(row)) {
        var data = this.$parent.store.states.data.slice(0)
        data[index]._expanded = !data[index]._expanded;
        if (data[index]._expanded) {
          data = data.splice(0, index + 1).concat(row[this.childKey]).concat(data)
        } else {
          data = this.removeChildNode(data, row[this.treeKey])
        }
        // this.$parent.store.commit('setData', data)
        this.$set(this.$parent.store.states, 'data', data)
        this.$nextTick(() => {
          this.$parent.doLayout()
        })
      } else if (row.haveChildBom) {
        var data = this.$parent.store.states.data.slice(0)
        if (!data[index]._expanded) {
          this.$emit('loadMoreData', index, row, data)
        }
      }
    },
    // 移除子节点
    removeChildNode(data, parentId) {
      var parentIds = isArray(parentId) ? parentId : [parentId]
      if (parentId.length <= 0) {
        return data
      }
      var ids = []
      for (var i = 0; i < data.length; i++) {
        if (parentIds.indexOf(data[i][this.parentKey]) !== -1 && parentIds.indexOf(data[i][this.treeKey]) === -1) {
          ids.push(data.splice(i, 1)[0][this.treeKey])
          i--
        }
      }
      return this.removeChildNode(data, ids)
    }
  }
}
</script>
