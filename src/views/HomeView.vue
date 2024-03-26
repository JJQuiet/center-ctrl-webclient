<template>
  <div>
    <el-tabs v-model="activeTab" @tab-click="handleTabClick">
      <el-tab-pane label="音视频切换" name="B"></el-tab-pane>
      <el-tab-pane label="视频切换" name="V"></el-tab-pane>
      <el-tab-pane label="音频切换" name="A"></el-tab-pane>
    </el-tabs>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column fixed prop="input" label="输入/输出" width="100">
        <template v-slot="scope">
          <span>{{ scope.row.input }}</span>
        </template>
      </el-table-column>
      <el-table-column
        v-for="index in outputCols"
        :key="index"
        :label="`输出${index}`"
        width="100"
      >
        <template v-slot="{ row }">
          <el-button size="small" @click="sendRequest(row.input, index)">
            {{ row["output" + index] }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: "B",
      tableData: this.createTableData(8), // 8 x 8 grid for B and V, for A it will be 13 x 13
      outputCols: 8, // Default for B and V tabs
    };
  },
  methods: {
    createTableData(size) {
      let arr = Array.from({ length: size }, (_, i) => {
        const row = { input: i + 1 };
        for (let j = 1; j <= size; j++) {
          row["output" + j] = `${i + 1}/${j}`;
        }
        return row;
      });
      console.log("arr", arr);
      return arr;
    },
    sendRequest(input, output) {
      const params = {
        input: input,
        output: output,
        type: this.activeTab,
      };
      // Replace with actual request code
      console.log(`Sending request with params:`, params);
    },
    handleTabClick(tab) {
      this.activeTab = tab.name;
      const size = tab.name === "A" ? 13 : 8;
      this.tableData = this.createTableData(size);
      this.outputCols = size;
    },
  },
};
</script>
