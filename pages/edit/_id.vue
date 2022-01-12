<template>
  <div class="code">
    <div class="codemirror">
      <codemirror
        v-model="code"
        :options="cmOptions"
        @cursorActivity="onCmCursorActivity"
        @ready="onCmReady"
        @focus="onCmFocus"
        @blur="onCmBlur"
      />
    </div>
    <pre class="pre">{{ code }}</pre>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Manager, Socket } from "socket.io-client";
// import { codemirror } from "vue-codemirror";

// // base style
// import "codemirror/lib/codemirror.css";

// // theme css
// import "codemirror/theme/base16-dark.css";

// // language
// import "codemirror/mode/vue/vue.js";

// // active-line.js
// import "codemirror/addon/selection/active-line.js";

// // styleSelectedText
// import "codemirror/addon/selection/mark-selection.js";
// import "codemirror/addon/search/searchcursor.js";

// // highlightSelectionMatches
// import "codemirror/addon/scroll/annotatescrollbar.js";
// import "codemirror/addon/search/matchesonscrollbar.js";
// import "codemirror/addon/search/match-highlighter.js";

// // keyMap
// import "codemirror/mode/clike/clike.js";
// import "codemirror/addon/edit/matchbrackets.js";
// import "codemirror/addon/comment/comment.js";
// import "codemirror/addon/dialog/dialog.js";
// import "codemirror/addon/dialog/dialog.css";
// import "codemirror/addon/search/search.js";
// import "codemirror/keymap/sublime.js";

// // foldGutter
// import "codemirror/addon/fold/foldgutter.css";
// import "codemirror/addon/fold/brace-fold.js";
// import "codemirror/addon/fold/comment-fold.js";
// import "codemirror/addon/fold/foldcode.js";
// import "codemirror/addon/fold/foldgutter.js";
// import "codemirror/addon/fold/indent-fold.js";
// import "codemirror/addon/fold/markdown-fold.js";
// import "codemirror/addon/fold/xml-fold.js";

export default Vue.extend({
  name: "EditPage",
  data() {
    return {
      title: "",
      code: "",
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
        lineWrapping: true,
        lineNumbers: true,
        readOnly: false,
        showCursorWhenSelecting: true,
        lineWiseCopyCut: true,
        autofocus: true,
      },
    };
  },
  // components: {
  //   codemirror,
  // },
  mounted() {
    this.manager = new Manager("ws://localhost", {
      reconnectionDelayMax: 10000,
      query: {
        documentName: "網路程式設計",
        userName: "moontai0724",
      },
    });
    this.currentSocket = this.manager.socket("/");
    this.currentSocket.emit("OpenFile", { id: this.$route.params.id });
    this.currentSocket.on("RequestedFile", (file) => {
      if (!file) this.$router.push("/");
      this.code = file.content;
      this.title = file.title;
    });
  },
  beforeDestroy() {
    this.currentSocket.disconnect();
  },
  methods: {
    onCmCursorActivity(codemirror) {
      console.debug("onCmCursorActivity", codemirror);
    },
    onCmReady(codemirror) {
      console.debug("onCmReady", codemirror);
    },
    onCmFocus(codemirror) {
      console.debug("onCmFocus", codemirror);
    },
    onCmBlur(codemirror) {
      console.debug("onCmBlur", codemirror);
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
    padding: 1rem;
    font-size: 12px;
    line-height: 1.6;
    word-break: break-all;
    word-wrap: break-word;
  }
}
</style>
