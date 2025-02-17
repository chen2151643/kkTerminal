<template>
  <div ref="aceEditorRef" class="kk-code-editor"></div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue';
import ace from 'ace-builds/src-noconflict/ace';
import 'ace-builds/src-noconflict/ext-searchbox';
import 'ace-builds/src-noconflict/theme-chrome';
import 'ace-builds/src-noconflict/mode-text';

export default {
  name: 'AceEditor',
  setup(props, context) {
    const aceEditorRef = ref();
    const aceEditor = ref(null);

    // 赋值内容
    const setValue = (text) => {
      if(aceEditor.value) aceEditor.value.setValue(text);
      if(aceEditor.value) aceEditor.value.selection.clearSelection();
    };

    // 清空撤销历史
    const reset = () => {
      if(aceEditor.value) aceEditor.value.getSession().getUndoManager().reset();
    }

    const CtrlS = (event) => {
      if (event.ctrlKey || event.metaKey) {
        switch (String.fromCharCode(event.which).toLowerCase()) {
          case 's':
            event.preventDefault();
            context.emit('handleSave', aceEditor.value.getValue());
            break;
        }
      }
    };

    onMounted(() => {
      aceEditor.value = ace.edit(aceEditorRef.value, {
        autoScrollEditorIntoView: false,
        copyWithEmptySelection: false,
        showPrintMargin: false,
        highlightActiveLine: true,
        theme: 'ace/theme/chrome',                            // 主题
        mode: 'ace/mode/text',                                // 高亮
      });

      // 编辑 监听内容变化
      aceEditor.value.on('change', () => {
        context.emit('handleChange');
      });

      // 查找替换
      aceEditor.value.find('needle',{
        backwards: false,
        wrap: false,
        caseSensitive: false,
        wholeWord: false,
        regExp: false
      });
      aceEditor.value.findNext();
      aceEditor.value.findPrevious();

      // 监听保存键 ctrl+s
      aceEditorRef.value.addEventListener('keydown', CtrlS);
    });

    onUnmounted(() => {
      if(aceEditorRef.value) aceEditorRef.value.removeEventListener('keydown', CtrlS);
    });

    return {
      aceEditorRef,
      aceEditor,
      setValue,
      CtrlS,
      reset,
    }
  }
}
</script>

<style scoped>
.kk-code-editor {
  height: 100%; 
  width: 100%;
  border: 1px solid #ececec;
}
</style>