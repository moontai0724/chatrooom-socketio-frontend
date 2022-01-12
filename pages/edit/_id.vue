<template>
  <div class="code">
    <div class="codemirror">
      <codemirror
        ref="cmEditor"
        :value="code"
        :options="cmOptions"
        @input="onCmCodeChange"
      />
    </div>
    <pre class="pre">{{ code }}</pre>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Manager, Socket } from "socket.io-client";
import { codemirror } from "vue-codemirror";

// base style
import "codemirror/lib/codemirror.css";

// theme css
import "codemirror/theme/base16-dark.css";

// language
import "codemirror/mode/markdown/markdown.js";

// active-line.js
import "codemirror/addon/selection/active-line.js";

// styleSelectedText
import "codemirror/addon/selection/mark-selection.js";
import "codemirror/addon/search/searchcursor.js";

// highlightSelectionMatches
import "codemirror/addon/scroll/annotatescrollbar.js";
import "codemirror/addon/search/matchesonscrollbar.js";
import "codemirror/addon/search/match-highlighter.js";

export default Vue.extend({
  name: "EditPage",
  components: {
    codemirror,
  },
  data() {
    return {
      id: this.$route.params.id,
      title: "",
      code: "",
      anyKeyPressed: false,
      manager: {} as Manager,
      currentSocket: {} as Socket,
      cmOptions: {
        mode: "markdown",
        theme: "base16-dark",
        extraKeys: {
          F11(cm) {
            cm.setOption("fullScreen", !cm.getOption("fullScreen"));
          },
          Esc(cm) {
            if (cm.getOption("fullScreen")) cm.setOption("fullScreen", false);
          },
        },
        line: true,
        lineWrapping: true,
        lineNumbers: true,
        readOnly: false,
        showCursorWhenSelecting: true,
        lineWiseCopyCut: true,
        autofocus: true,
        viewportMargin: 20,
      },
    };
  },
  mounted() {
    this.manager = new Manager("ws://" + this.$config.WS_HOST, {
      reconnectionDelayMax: 10000,
      query: {
        documentName: "網路程式設計",
        userName: "moontai0724",
      },
    });
    this.currentSocket = this.manager.socket("/");
    this.currentSocket.emit("OpenFile", { id: this.id });
    this.currentSocket.on("RequestedFile", (file) => {
      if (!file) this.$router.push("/");
      this.code = file.content;
      this.title = file.title;
    });
    this.currentSocket.on("FileUpdated", (file) => {
      this.code = file.content;
      this.title = file.title;
    });
  },
  beforeDestroy() {
    this.currentSocket.disconnect();
  },
  methods: {
    onCmCodeChange(newCode) {
      if (this.code === newCode) return;
      this.code = newCode;
      this.currentSocket.emit("UpdateFile", {
        id: this.id,
        title: this.title,
        content: newCode,
        lastUpdate: Date.now(),
        updator: this.currentSocket.id,
      });
    },
  },
});
</script>

<style lang="scss" scoped>
.code {
  display: flex;
  height: 100%;

  .codemirror,
  .pre {
    width: 50%;
    height: 100%;
    margin: 0;
    overflow: auto;
  }

  .pre {
    display: block;
    height: 100%;
    overflow: auto;
    padding: 1rem;
    font-size: 12px;
    line-height: 1.6;
    word-break: break-all;
    word-wrap: break-word;
  }
}
</style>
