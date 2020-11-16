<template>
  <script :id="id" type="text/plain"></script>
</template>

<script>
import globalMixin from '../mixins/globalMixin';

// 一个简单的事件订阅发布的实现,取代原始Event对象,提升IE下的兼容性
class LoadEvent {
  constructor() {
    this.listeners = {};
  }
  on(eventName, callback) {
    if (this.listeners[eventName] === undefined) {
      this.listeners[eventName] = [];
    } else {
      this.listeners[eventName] = '';
    }
    this.listeners[eventName].push(callback);
  }
  emit(eventName) {
    if (this.listeners[eventName]) {
      this.listeners[eventName].forEach(callback => callback());
    }
  }
}

export default {
  mixins: [globalMixin],
  data() {
    return {
      id: `editor${Math.random().toString().slice(-10)}`,
      editor: null,
      destroy: false,
      defaultConfig: {
        serverUrl: `${process.env.VUE_APP_UEDITOR}/ueditorServlet`,
        UEDITOR_HOME_URL: './UEditor/',
        autoHeightEnabled: false,
        initialFrameHeight: 240,
        initialFrameWidth: '100%',
        enableAutoSave: false,
        toolbars: [
          [
            'fullscreen', 'source', '|', 'undo', 'redo', '|',
            'bold', 'italic', 'underline', 'fontborder', 'strikethrough', 'superscript',
            'subscript', 'removeformat', 'formatmatch', 'autotypeset', 'blockquote', 'pasteplain', '|',
            'forecolor', 'backcolor', 'insertorderedlist', 'insertunorderedlist',
            'selectall', 'cleardoc', '|',
            'rowspacingtop', 'rowspacingbottom', 'lineheight', '|',
            'customstyle', 'paragraph', 'fontsize', '|',
            'directionalityltr', 'directionalityrtl', 'indent', '|',
            'justifyleft', 'justifycenter', 'justifyright', 'justifyjustify', '|',
            'touppercase', 'tolowercase', '|',
            'insertimage', 'emotion', 'scrawl', 'insertcode', 'pagebreak', 'template', 'background', '|',
            'horizontal', 'date', 'time', 'spechars', '|',
            'inserttable', 'deletetable', 'insertparagraphbeforetable', 'insertrow',
            'deleterow', 'insertcol', 'deletecol', 'mergecells', 'mergeright', 'mergedown',
            'splittocells', 'splittorows', 'splittocols', 'charts', '|',
          ],
        ],
      },
      mixedConfig: {},
    };
  },
  props: {
    value: {
      required: true,
      type: String,
    },
    config: {
      type: Object,
      default() {
        return {};
      },
    },
    init: {
      type: Function,
      default() {
        return () => {};
      },
    },
    needDestroy: {
      type: Boolean,
      default() {
        return true;
      },
    },
  },
  watch: {
    value(val) {
      if (this.editor) {
        this.setContent(val);
      }
    },
    config: {
      handler(val) {
        const newVal = this.methodCloneDeep(val);
        const defaultConfig = this.methodCloneDeep(this.defaultConfig);
        const mergeSet = new Set(defaultConfig.toolbars[0]);
        if (this.methodHas(newVal, 'toolbars') && newVal.toolbars.length !== 0) {
          newVal.toolbars[0].forEach(v => mergeSet.add(v));
        }
        defaultConfig.toolbars[0] = Array.from(mergeSet);
        this.mixedConfig = defaultConfig;
        this.$nextTick(() => this.renewEditor());
      },
      immediate: true,
      deep: true,
    },
    destroy() {
      if (this.editor) {
        this.editor.destroy();
      }
      this.beforeInitEditor(this.value);
    },
  },
  methods: {
    renewEditor() {
      this.destroy = !this.destroy;
    },
    registerButton: ({
      name, icon, tip, handler, UE = window.UE,
    }) => {
      UE.registerUI(name, (editor) => {
        editor.registerCommand(name, {
          execCommand: () => {
            handler(editor, name);
          },
        });
        const btn = new UE.ui.Button({
          name,
          title: tip,
          cssRules: `background-image: url(${icon}) !important;background-size: cover;`,
          onclick() {
            editor.execCommand(name);
          },
        });
        editor.addListener('selectionchange', () => {
          const state = editor.queryCommandState(name);
          if (state === -1) {
            btn.setDisabled(true);
            btn.setChecked(false);
          } else {
            btn.setDisabled(false);
            btn.setChecked(state);
          }
        });
        return btn;
      });
    },
    // 实例化编辑器之前-JS依赖检测
    beforeInitEditor(value) {
      // 准确判断ueditor.config.js和ueditor.all.js均已加载
      // 仅加载完ueditor.config.js时UE对象和UEDITOR_CONFIG对象也存在,
      // 仅加载完ueditor.all.js时UEDITOR_CONFIG对象也存在,但为空对象
      if (!!window.UE && !!window.UEDITOR_CONFIG &&
        Object.keys(window.UEDITOR_CONFIG).length !== 0 &&
        !!window.UE.getEditor) {
        this.initEditor(value);
      } else {
        this.loadScripts().then(() => this.initEditor(value));
      }
    },
    // 实例化编辑器
    initEditor(value) {
      this.$nextTick(() => {
        this.init();
        // 没有按官网示例那样链式调用ready方法的原因在于需要拿到getEditor返回的实例
        this.editor = window.UE.getEditor(this.id, this.mixedConfig);
        this.editor.addListener('ready', () => {
          this.$emit('ready', this.editor);
          this.editor.setContent(value);
          this.editor.addListener('contentChange', () => {
            this.$emit('input', this.editor.getContent());
          });
        });
      });
    },
    // 动态添加JS依赖
    loadScripts() {
      // 确保多个实例同时渲染时不会重复创建SCRIPT标签
      if (window.loadEnv) {
        return new Promise((resolve, reject) => {
          window.loadEnv.on('scriptsLoaded', () => {
            resolve();
          });
        });
      }
      window.loadEnv = new LoadEvent();
      return new Promise((resolve, reject) => {
        // 如果在其他地方只引用ueditor.all.min.js，
        // 在加载ueditor.config.js之后仍需要重新加载ueditor.all.min.js，
        // 所以必须确保ueditor.config.js已加载
        this.loadConfig().then(() => this.loadCore()).then(() => {
          window.loadEnv.emit('scriptsLoaded');
          resolve();
        });
      });
    },
    loadConfig() {
      return new Promise((resolve, reject) => {
        if (!!window.UE && !!window.UEDITOR_CONFIG &&
          Object.keys(window.UEDITOR_CONFIG).length !== 0) {
          resolve();
          return;
        }
        const configScript = document.createElement('script');
        configScript.type = 'text/javascript';
        configScript.src = `${this.mixedConfig.UEDITOR_HOME_URL}ueditor.config.js`;
        document.getElementsByTagName('head')[0].appendChild(configScript);
        configScript.onload = () => {
          if (!!window.UE && !!window.UEDITOR_CONFIG &&
            Object.keys(window.UEDITOR_CONFIG).length !== 0) {
            resolve();
          } else {
            console.error('加载ueditor.config.js失败,请检查您的配置地址UEDITOR_HOME_URL填写是否正确!');
          }
        };
      });
    },
    loadCore() {
      return new Promise((resolve, reject) => {
        if (!!window.UE && !!window.UE.getEditor) {
          resolve();
          return;
        }
        const coreScript = document.createElement('script');
        coreScript.type = 'text/javascript';
        coreScript.src = `${this.mixedConfig.UEDITOR_HOME_URL}ueditor.all.min.js`;
        document.getElementsByTagName('head')[0].appendChild(coreScript);
        coreScript.onload = () => {
          if (!!window.UE && !!window.UE.getEditor) {
            resolve();
          } else {
            console.error('加载ueditor.all.min.js失败,请检查您的配置地址UEDITOR_HOME_URL填写是否正确!');
          }
        };
      });
    },
    // 设置内容
    setContent(value) {
      if (value !== this.editor.getContent()) {
        this.editor.setContent(value);
      }
    },
  },
  beforeDestroy() {
    if (this.needDestroy && this.editor && this.editor.destroy) this.editor.destroy();
  },
};
</script>
