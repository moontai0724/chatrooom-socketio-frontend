<template>
  <div>
    <div>
      <h1>Intro</h1>
      <div>Hi！歡迎來到簡易線上共筆～</div>
      <div>以下是系統的簡易說明：</div>
      <ol>
        <li>左邊是目前現有檔案的列表，列表中是文件標題。</li>
        <li>右下角的綠色按鈕點下去後可以新增文件。</li>
        <li>本網站使用 Markdown 作為編輯語言。</li>
      </ol>
    </div>
    SERVER: {{ protocol === "https:" ? "wss://" : "ws://" }}{{ host }}
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "IndexPage",
  data() {
    return {
      host: this.$config.WS_HOST,
      protocol: "",
    };
  },
  mounted() {
    this.protocol = location.protocol;
  },
});
</script>
