<template>
  <el-tabs v-model="activeTab" @tab-click="handleTabClick">
    <el-tab-pane label="音视频切换" name="B"></el-tab-pane>
    <el-tab-pane label="视频切换" name="V"></el-tab-pane>
    <el-tab-pane label="音频切换" name="A"></el-tab-pane>
  </el-tabs>

  <div class="grid-container">
    <el-row class="header-row">
      <el-col :span="1.5" class="label-container header-col">
        <div class="label-input">输入</div>
        <div class="label-output">输出</div>
      </el-col>
      <el-col
        :span="1.5"
        v-for="colLabel in colLabels[activeTab]"
        :key="colLabel"
        class="header-row-col"
      >
        {{ colLabel }}
      </el-col>
    </el-row>
    <el-row
      v-for="(rowLabel, rowIndex) in rowLabels[activeTab]"
      :key="rowIndex"
    >
      <el-col :span="1.5" class="header-col">{{ rowLabel }}</el-col>
      <el-col
        v-for="(colLabel, colIndex) in colLabels[activeTab]"
        :key="colLabel"
        :span="1.5"
        class="data-col"
      >
        <el-button
          @click="sendRequest(rowIndex, colIndex)"
          class="data-col-btn"
          :type="data[activeTab][rowIndex][colIndex] ? 'danger' : 'primary'"
        >
        </el-button>
      </el-col>
    </el-row>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
let activeTab = ref("V");
const colLabels = ref({
  B: Array.from({ length: 8 }, (_, i) => i + 1),
  V: Array.from({ length: 8 }, (_, i) => i + 1),
  A: [
    "无线MIC",
    "CH1",
    "CH2",
    "CH3",
    "CH4",
    "CH5",
    "CH6",
    "CH7",
    "CH8",
    "视频1",
    "视频2",
    "视频3",
    "视频4",
  ],
});

const rowLabels = ref({
  B: Array.from({ length: 8 }, (_, i) => i + 1),
  V: Array.from({ length: 8 }, (_, i) => i + 1),
  A: [
    "功放",
    "视频1",
    "视频2",
    "视频3",
    "视频4",
    "CH1",
    "CH2",
    "CH3",
    "CH4",
    "CH5",
    "CH6",
    "CH7",
    "CH8",
  ],
});
let data = ref({
  B: [
    [true, false, false, false, false, false, false, false],
    [false, true, false, false, false, false, false, false],
    [false, false, true, false, false, false, false, false],
    [false, false, false, true, false, false, false, false],
    [false, false, false, false, true, false, false, false],
    [false, false, false, false, false, true, false, false],
    [false, false, false, false, false, false, true, false],
    [false, false, false, false, false, false, false, true],
  ],
  V: [
    [true, false, false, false, false, false, false, false],
    [false, true, false, false, false, false, false, false],
    [false, false, true, false, false, false, false, false],
    [false, false, false, true, false, false, false, false],
    [false, false, false, false, true, false, false, false],
    [false, false, false, false, false, true, false, false],
    [false, false, false, false, false, false, true, false],
    [false, false, false, false, false, false, false, true],
  ],
  A: Array.from({ length: 13 }, () => Array.from({ length: 13 }, () => false)),
});

const handleTabClick = (tab) => {
  activeTab.value = tab.name;
};
const sendRequest = async (output, input) => {
  if (activeTab.value === "V") {
    input += 1;
    output += 1;
  } else if (activeTab.value === "A") {
    if (input === 0) {
      input = 9;
    } else if (input >= 1 && input <= 8) {
      input += 10;
    } else if (input >= 9) {
      input -= 8;
    }
    if (output === 0) {
      output = 5;
    } else if (output >= 5) {
      output += 1;
    }
  }
  let type = activeTab.value;
  const url = new URL("http://192.168.0.152:8081/change_status");
  url.searchParams.append("input", input);
  url.searchParams.append("output", output);
  url.searchParams.append("type", type);
  fetch(url).then((res) => {
    res.text().then((text) => {
      if (text === "ChangeSuccess") {
        console.log("text is ChangeSuccess");
        fetch(`http://192.168.0.152:8081/status`);
      }
    });
  });
};
// 创建WebSocket连接
let webSocket = (ref < WebSocket) | (null > null); // 指定类型为WebSocket或null
const connectWebSocket = async () => {
  webSocket = new WebSocket("ws://192.168.0.152:8081/ws");
  webSocket.onopen = () => {
    // 处理连接建立
    console.log("WebSocket connection established");
  };
  webSocket.onmessage = (event) => {
    console.log(
      "%c [ event ]-150",
      "font-size:13px; background:pink; color:#bf2c9f;",
      event
    );
    data.value = {
      B: Array.from({ length: 8 }, () =>
        Array.from({ length: 8 }, () => false)
      ),
      V: Array.from({ length: 8 }, () =>
        Array.from({ length: 8 }, () => false)
      ),
      A: Array.from({ length: 13 }, () =>
        Array.from({ length: 13 }, () => false)
      ),
    };
    for (let i = 0; i < event.data.length; i += 10) {
      let type = event.data.substring(i, i + 1);
      let input, output;
      if (event.data.substring(i + 2, i + 5) === "OFF") break;
      if (type === "B" || type === "V") {
        input = parseInt(event.data.substring(i + 2, i + 5)) - 1;
        output = parseInt(event.data.substring(i + 7, i + 10)) - 1;
      } else if (type === "A") {
        input = parseInt(event.data.substring(i + 2, i + 5));
        output = parseInt(event.data.substring(i + 7, i + 10));
        if (output === 5) {
          output = 0;
        } else if (output > 5) {
          output -= 1;
        }
        if (input === 9) {
          input = 0;
        } else if (input >= 11) {
          input -= 10;
        } else if (input <= 4) {
          input += 8;
        }
      }
      data.value[type][output][input] = true;
    }
  };

  webSocket.onclose = (event) => {
    // 处理连接关闭
    console.log("WebSocket connection closed:", event);
  };

  webSocket.onerror = (error) => {
    // 处理错误
    console.error("WebSocket error:", error);
  };
};
onMounted(async () => {
  await connectWebSocket();
  fetch(`http://192.168.0.152:8081/status`);
});
</script>

<style>
.header-col,
.header-row-col {
  text-align: center;
  align-self: center;
  font-weight: bold;
}
.grid-container {
  margin-top: 20px;
}
.header-col {
  width: 70px;
  text-align: end;
  padding-right: 15px;
  background-color: #f0f0f0;
}
.data-col-btn {
  width: 40px;
}
.header-row {
  height: 50px;
  background-color: #f0f0f0;
}
.header-row-col {
  width: 50px;
}
.label-container {
  position: relative; /* 设置相对定位作为后代绝对定位的参照 */
  /* 示例高度，根据需要调整 */
  height: 44px;
  /* width: 70px; */
  background-color: #f0f0f0;
}

.label-input {
  position: absolute; /* 绝对定位 */
  top: 0; /* 置于顶部 */
  right: 0; /* 置于右侧 */
  /* 一些内边距 */
  /* padding: 5px; */
  /* background-color: #dff0d8; */
}

.label-output {
  position: absolute; /* 绝对定位 */
  bottom: 0; /* 置于底部 */
  left: 0; /* 置于左侧 */
  /* 一些内边距 */
  /* padding: 5px; */
  /* background-color: #d9edf7;  */
}
.data-col-btn {
  margin: 5px;
}
</style>
