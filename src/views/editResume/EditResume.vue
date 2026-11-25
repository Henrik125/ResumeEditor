<template>
  <div class="resume-editor-page" :class="{'is-resizing': resizing}">
    <header id="EditResume" class="editor-toolbar">
      <div class="editor-toolbar-inner">
        <el-button class="home-button" text @click="router.push({path: `/`})">
          <span class="back-mark" aria-hidden="true">‹</span>
          <span>首页</span>
        </el-button>

        <el-dropdown :hide-on-click="false">
          <button type="button" class="spacing-trigger">
            <align-text-both-one theme="two-tone" size="25" :fill="['#ffffff' ,'#93c5fd']"/>
            <span>间距设置</span>
          </button>
          <template #dropdown>
            <el-dropdown-menu style="padding: 20px 15px 0">
              <div class="public-col-center">
                <div class="public-row dropItem1">
                  <h3 class="SpaceItem">字体间距大小</h3>
                  <el-input-number v-model="CStyle.FontSpace" size="small" @change="GlobalSpace"
                                   :precision="2" :step="0.05" :min="0"/>
                </div>
                <div class="public-row dropItem1">
                  <h3 class="SpaceItem">行间距大小</h3>
                  <el-input-number v-model="CStyle.LineHeight" size="small" @change="GlobalSpace"
                                   :precision="2" :step="0.05" :min="-0.5"/>
                </div>
                <div class="public-row dropItem1">
                  <h3 class="SpaceItem">模块间距大小</h3>
                  <el-input-number v-model="CStyle.PaddingTB" size="small" @change="GlobalSpace"
                                   :min="-5"/>
                </div>
              </div>
            </el-dropdown-menu>
          </template>
        </el-dropdown>

        <el-button v-print="'#resume'" class="saveToPdf">保存为 PDF</el-button>
      </div>
    </header>

    <main class="editor-workspace" :style="{'--editor-panel-width': editorWidth + 'px'}">
      <div
        class="editor-resize-handle"
        role="separator"
        aria-orientation="vertical"
        aria-label="调整右侧操作区宽度"
        :aria-valuemin="minEditorWidth"
        :aria-valuemax="maxEditorWidth"
        :aria-valuenow="editorWidth"
        tabindex="0"
        @pointerdown="startEditorResize"
        @pointermove="resizeEditor"
        @pointerup="stopEditorResize"
        @pointercancel="stopEditorResize"
        @keydown="resizeEditorByKeyboard"
      />
      <section id="myResume-index" class="resume-preview">
        <router-view/>
      </section>
    </main>
  </div>

</template>

<script>
import {defineComponent, ref, reactive, onMounted, watch} from "vue";
import {useRouter} from "vue-router";
import {AlignTextBothOne} from '@icon-park/vue-next';

export default defineComponent({
  // eslint-disable-next-line vue/multi-word-component-names
  name: "1",
  components: {AlignTextBothOne},
  setup() {
    const router = useRouter()
    const data = ref({test1: "", test2: ""})
    const editorWidth = ref(506)
    const resizing = ref(false)
    const minEditorWidth = 320
    const maxEditorWidth = 640

    const startEditorResize = (event) => {
      event.preventDefault()
      resizing.value = true
      event.currentTarget.setPointerCapture(event.pointerId)
    }
    const resizeEditor = (event) => {
      if (resizing.value) {
        const workspaceRight = event.currentTarget.parentElement.getBoundingClientRect().right
        const width = Math.round(workspaceRight - event.clientX)
        editorWidth.value = Math.min(maxEditorWidth, Math.max(minEditorWidth, width))
      }
    }
    const stopEditorResize = () => {
      resizing.value = false
    }
    const resizeEditorByKeyboard = (event) => {
      if (event.key === 'ArrowLeft' || event.key === 'ArrowRight') {
        const change = event.key === 'ArrowLeft' ? 16 : -16
        editorWidth.value = Math.min(maxEditorWidth, Math.max(minEditorWidth, editorWidth.value + change))
        event.preventDefault()
      }
    }

    const YStyle = reactive({
      FontSize: 16,
      FontSpace: 0,
      LineHeight: 1.5,
      PaddingTB: 5,
    })
    const CStyle = reactive({
      FontSize: 0,
      FontSpace: 0,
      LineHeight: 0,
      PaddingTB: 0,
    })
    // 在组件挂载后，设置 CSS 变量的值
    onMounted(() => {
      document.documentElement.style.setProperty("--FontSize", YStyle.FontSize + "px");
      document.documentElement.style.setProperty("--FontSpace", YStyle.FontSpace + "rem");
      document.documentElement.style.setProperty("--LineHeight", YStyle.LineHeight + "rem");
      document.documentElement.style.setProperty("--PaddingTB", YStyle.PaddingTB + "px");
    });
    // 监听变量的方法
    ((color) => {
      watch(color, () => {
            document.documentElement.style.setProperty("--FontSpace", YStyle.FontSpace + CStyle.FontSpace + "rem");
            document.documentElement.style.setProperty("--LineHeight", YStyle.LineHeight + CStyle.LineHeight + "rem");
            document.documentElement.style.setProperty("--PaddingTB", YStyle.PaddingTB + CStyle.PaddingTB + "px");
          }
      )
    })(CStyle)

    return {
      router, data, CStyle,
      editorWidth, resizing, minEditorWidth, maxEditorWidth,
      startEditorResize, resizeEditor, stopEditorResize, resizeEditorByKeyboard,
    }
  },
})
</script>

<style scoped>
.resume-editor-page {
  min-height: 100vh;
  color: #152033;
  background: linear-gradient(180deg, #f6f8ff 0%, #ffffff 48%, #f7f8fb 100%);
}

.resume-editor-page.is-resizing {
  cursor: col-resize;
  user-select: none;
}

.editor-toolbar {
  position: relative;
  z-index: 10;
  height: 60px;
  color: #ffffff;
  background: linear-gradient(135deg, #0f172a 0%, #1d4ed8 58%, #38bdf8 100%);
  box-shadow: 0 8px 24px rgba(15, 23, 42, 0.16);
}

.editor-toolbar-inner {
  display: flex;
  align-items: center;
  width: min(1500px, calc(100% - 48px));
  height: 100%;
  margin: 0 auto;
  gap: 18px;
}

.home-button {
  height: 40px;
  padding: 0 10px;
  color: #ffffff;
  font-size: 16px;
  font-weight: 700;
  border-radius: 12px;
}

.home-button:hover,
.home-button:focus-visible {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.14);
}

.back-mark {
  margin-right: 3px;
  font-size: 27px;
  font-weight: 400;
  line-height: 1;
}

.spacing-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 7px 12px;
  color: rgba(255, 255, 255, 0.9);
  font: inherit;
  font-size: 14px;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
}

.spacing-trigger:hover,
.spacing-trigger:focus-visible {
  background: rgba(255, 255, 255, 0.18);
  outline: none;
}

.saveToPdf {
  height: 40px;
  padding: 0 18px;
  margin-left: auto;
  color: #1d4ed8;
  font-size: 14px;
  font-weight: 700;
  background: #ffffff;
  border-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 24px rgba(15, 23, 42, 0.18);
}

.saveToPdf:hover,
.saveToPdf:focus-visible {
  color: #1e40af;
  background: #eff6ff;
  border-color: #eff6ff;
}

.editor-workspace {
  --editor-panel-width: 506px;
  position: relative;
  height: calc(100vh - 60px);
  overflow: hidden;
  background:
    radial-gradient(circle at 12% 10%, rgba(76, 201, 240, 0.14), transparent 30%),
    linear-gradient(180deg, #eef3fb 0%, #f7f8fb 100%);
}

.editor-resize-handle {
  position: absolute;
  z-index: 8;
  top: 0;
  bottom: 0;
  left: calc(100% - var(--editor-panel-width));
  width: 14px;
  height: 100%;
  padding: 0;
  cursor: col-resize;
  touch-action: none;
  background: transparent;
  border: 0;
  transform: translateX(-50%);
}

.editor-resize-handle::before,
.editor-resize-handle::after {
  position: absolute;
  left: 50%;
  content: "";
}

.editor-resize-handle::before {
  top: 0;
  bottom: 0;
  width: 1px;
  background: #dbe4f0;
  transform: translateX(-50%);
}

.editor-resize-handle::after {
  top: 50%;
  width: 5px;
  height: 44px;
  background: #cbd5e1;
  border: 2px solid #ffffff;
  border-radius: 999px;
  box-shadow: 0 0 0 1px #dbe4f0;
  transform: translate(-50%, -50%);
}

.editor-resize-handle:hover::before,
.editor-resize-handle:focus-visible::before {
  width: 2px;
  background: #2563eb;
}

.editor-resize-handle:hover::after,
.editor-resize-handle:focus-visible::after {
  background: #2563eb;
}

.editor-resize-handle:focus-visible {
  outline: 2px solid rgba(37, 99, 235, 0.35);
  outline-offset: -2px;
}

.resume-preview {
  box-sizing: border-box;
  width: calc(100% - var(--editor-panel-width));
  height: 100%;
  padding: 30px 24px;
  overflow: auto;
}

:deep( #ContentBody  .md-editor-preview p) {
  letter-spacing: var(--FontSpace);
  font-size: var(--FontSize);
  line-height: var(--LineHeight);
  padding: var(--PaddingTB) 0;
}

:deep(#ContentBody .control-Title) {
  display: flex;
  justify-content: space-between;
  width: 100%;
  font-size: var(--FontSize);
}

.SpaceItem {
  width: 100px;
  text-align-last: justify;
  margin-right: 10px;
  margin-top: 2px;
}

.dropItem1 {
  margin-bottom: 15px;
  height: 1.5rem;
}
</style>
