<template>
  <div class="app-container">
    <el-input v-model="filterText" placeholder="Filter keyword" style="margin-bottom:30px;" />

    <el-tree
      ref="tree2"
      :data="subjectList"
      :props="defaultProps"
      :filter-node-method="filterNode"
      class="filter-tree"
      default-expand-all
    />

  </div>
</template>

<script>
import subject from '@/api/dzd/subject'
export default {
  data() {
    return {
      filterText: '',
      subjectList: [],
      defaultProps: {
        children: 'children',
        label: 'title'
      }
    }
  },
  watch: {
    filterText(val) {
      this.$refs.tree2.filter(val)
    }
  },

    created(){
        this.fetchNodeList()
    },

  methods: {
      fetchNodeList(){
            subject.getNestedTreeList().then(response =>{
                
                    this.subjectList = response.data.items
                
            })

      },
        filterNode(value, data) {
            if (!value) return true
            return data.title.toLowerCase().indexOf(value.toLowerCase()) !== -1
        }
  }
}
</script>

