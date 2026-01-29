<template>
  <div class="app">
    <header class="header">
      <div class="header-content">
        <div>
          <h1>DiffJson</h1>
          <p>{{ $t('header.subtitle') }}</p>
        </div>
        <div class="language-switcher">
          <select 
            v-model="currentLanguage"
            @change="switchLanguage(currentLanguage)"
            class="language-select"
          >
            <option 
              v-for="option in languageOptions" 
              :key="option.value" 
              :value="option.value"
            >
              {{ option.label }}
            </option>
          </select>
        </div>
      </div>
    </header>
    
    <main class="main">
      <div class="control-panel">
        <div class="buttons">
          <button @click="loadExample1">{{ $t('control.loadExample1') }}</button>
          <button @click="loadExample2">{{ $t('control.loadExample2') }}</button>
        </div>
        <div class="array-mode">
          <div class="mode-toggle">
            <div class="mode-label-with-hint">
              <span class="mode-label">{{ $t('control.arrayMode') }}</span>
              <span class="hint-icon" :title="$t('control.arrayModeHint')">?</span>
            </div>
            <label class="toggle-switch">
              <input 
                type="checkbox" 
                v-model="strictArrayMode"
                @change="onJsonChange"
              />
              <span class="toggle-slider"></span>
            </label>
          </div>
          <span class="mode-status">{{ strictArrayMode ? $t('control.strictMode') : $t('control.looseMode') }}</span>
        </div>
      </div>
      
      <div class="diff-container">
        <div class="json-input left">
          <div class="input-header">
            <h2>{{ $t('jsonInput.leftTitle') }}</h2>
            <div class="input-buttons">
              <button @click="formatJson('left')">{{ $t('control.format') }}</button>
              <button @click="copyJson('left')">{{ $t('control.copy') }}</button>
            </div>
          </div>
          <div v-if="leftError" class="error">
            {{ leftError }}
          </div>
          <div class="code-container">
            <div 
              class="line-numbers"
              :style="{ scrollTop: leftScrollTop + 'px' }"
            >
              <div 
                v-for="n in leftLineNumbers" 
                :key="n" 
                class="line-number"
              >
                {{ n }}
              </div>
            </div>
            <textarea 
              v-model="leftJson" 
              @input="onJsonChange" 
              :placeholder="$t('jsonInput.placeholder')"
              @scroll="syncScroll('left')"
            ></textarea>
          </div>
        </div>
        
        <div class="diff-result">
          <h2>{{ $t('diffResult.title') }}</h2>
          <div v-if="leftError || rightError" class="error">
            {{ $t('diffResult.formatError') }}
          </div>
          <div v-else-if="diffError" class="error">
            {{ diffError }}
          </div>
          <div v-else-if="!diffResult" class="no-diff">
            {{ $t('diffResult.noDiff') }}
          </div>
          <div v-else-if="diffResult.length === 0" class="no-diff">
            {{ $t('diffResult.noChanges') }}
          </div>
          <div v-else class="diff-content">
            <div 
              v-for="(item, index) in diffResult" 
              :key="index"
              :class="['diff-item', item.type]"
            >
              <div v-if="item.type === 'equal'" class="equal">
                <span class="diff-symbol">=></span>
                <span class="diff-path">{{ item.path }}</span>
                <span class="diff-value">{{ JSON.stringify(item.value) }}</span>
              </div>
              <div v-else-if="item.type === 'added'" class="added">
                <span class="diff-symbol">+</span>
                <span class="diff-path">{{ item.path }}</span>
                <span class="diff-value">{{ JSON.stringify(item.value) }}</span>
              </div>
              <div v-else-if="item.type === 'removed'" class="removed">
                <span class="diff-symbol">-</span>
                <span class="diff-path">{{ item.path }}</span>
                <span class="diff-value">{{ JSON.stringify(item.value) }}</span>
              </div>
              <div v-else-if="item.type === 'modified'" class="modified">
                <span class="diff-symbol">~</span>
                <span class="diff-path">{{ item.path }}</span>
                <span class="diff-old-value">{{ JSON.stringify(item.oldValue) }}</span>
                <span class="diff-arrow">→</span>
                <span class="diff-new-value">{{ JSON.stringify(item.newValue) }}</span>
              </div>
            </div>
          </div>
        </div>
        
        <div class="json-input right">
          <div class="input-header">
            <h2>{{ $t('jsonInput.rightTitle') }}</h2>
            <div class="input-buttons">
              <button @click="formatJson('right')">{{ $t('control.format') }}</button>
              <button @click="copyJson('right')">{{ $t('control.copy') }}</button>
            </div>
          </div>
          <div v-if="rightError" class="error">
            {{ rightError }}
          </div>
          <div class="code-container">
            <div 
              class="line-numbers"
              :style="{ scrollTop: rightScrollTop + 'px' }"
            >
              <div 
                v-for="n in rightLineNumbers" 
                :key="n" 
                class="line-number"
              >
                {{ n }}
              </div>
            </div>
            <textarea 
              v-model="rightJson" 
              @input="onJsonChange" 
              :placeholder="$t('jsonInput.placeholder')"
              @scroll="syncScroll('right')"
            ></textarea>
          </div>
        </div>
      </div>
    </main>
    
    <footer class="footer">
      <p>{{ $t('footer.copyright') }}</p>
    </footer>
  </div>
</template>

<script>
import { languages, languageOptions } from './i18n'

export default {
  name: 'App',
  data() {
    return {
      leftJson: '',
      rightJson: '',
      leftError: '',
      rightError: '',
      diffError: '',
      diffResult: null,
      strictArrayMode: false,
      debounceTimer: null,
      currentLanguage: 'en-US',
      leftScrollTop: 0,
      rightScrollTop: 0,
      diffScrollTop: 0,
      languageOptions
    }
  },
  created() {
    // 检测用户浏览器语言，设置默认语言
    this.detectUserLanguage()
  },
  computed: {
    $t() {
      const lang = this.currentLanguage
      return (key) => {
        const keys = key.split('.')
        let result = languages[lang]
        for (const k of keys) {
          result = result?.[k]
        }
        return result || key
      }
    },
    leftLineNumbers() {
      const lines = this.leftJson.split('\n').length
      return Array.from({ length: lines }, (_, i) => i + 1)
    },
    rightLineNumbers() {
      const lines = this.rightJson.split('\n').length
      return Array.from({ length: lines }, (_, i) => i + 1)
    }
  },
  methods: {
    switchLanguage(lang) {
      this.currentLanguage = lang
      this.onJsonChange()
    },
    detectUserLanguage() {
      // 获取用户浏览器语言
      const userLang = navigator.language || navigator.userLanguage
      // 检查是否在支持的语言列表中
      const supportedLangs = Object.keys(languages)
      
      // 优先匹配完整语言代码（如 'zh-CN'）
      if (supportedLangs.includes(userLang)) {
        this.currentLanguage = userLang
      } else {
        // 尝试匹配语言代码的前缀（如 'zh'）
        const langPrefix = userLang.split('-')[0]
        const matchedLang = supportedLangs.find(lang => lang.startsWith(langPrefix))
        if (matchedLang) {
          this.currentLanguage = matchedLang
        }
        // 如果没有匹配的语言，使用默认语言（中文）
      }
      
      // 触发一次比较，确保界面显示正确
      this.onJsonChange()
    },
    syncScroll(side) {
      if (side === 'left') {
        this.leftScrollTop = event.target.scrollTop
      } else if (side === 'right') {
        this.rightScrollTop = event.target.scrollTop
      } else if (side === 'diff') {
        this.diffScrollTop = event.target.scrollTop
      }
    },
    onJsonChange() {
      // 防抖处理，避免频繁比较
      clearTimeout(this.debounceTimer)
      this.debounceTimer = setTimeout(() => {
        this.validateJson()
        this.compareJson()
      }, 300)
    },
    validateJson() {
      this.leftError = ''
      this.rightError = ''
      
      if (this.leftJson) {
        try {
          JSON.parse(this.leftJson)
        } catch (e) {
          this.leftError = this.$t('error.leftFormat') + e.message
        }
      }
      
      if (this.rightJson) {
        try {
          JSON.parse(this.rightJson)
        } catch (e) {
          this.rightError = this.$t('error.rightFormat') + e.message
        }
      }
    },
    compareJson() {
      this.diffError = ''
      this.diffResult = null
      
      if (!this.leftJson || !this.rightJson) {
        return
      }
      
      if (this.leftError || this.rightError) {
        return
      }
      
      try {
        const leftObj = JSON.parse(this.leftJson)
        const rightObj = JSON.parse(this.rightJson)
        this.diffResult = this.diff(leftObj, rightObj)
      } catch (e) {
        this.diffError = this.$t('error.compare') + e.message
      }
    },
    diff(obj1, obj2, path = '') {
      const result = []
      
      // 检查类型是否相同
      if (typeof obj1 !== typeof obj2) {
        result.push({
          type: 'modified',
          path,
          oldValue: obj1,
          newValue: obj2
        })
        return result
      }
      
      // 处理基本类型
      if (typeof obj1 !== 'object' || obj1 === null || obj2 === null) {
        if (obj1 !== obj2) {
          result.push({
            type: 'modified',
            path,
            oldValue: obj1,
            newValue: obj2
          })
        }
        // 只添加不同的值，相同的值不添加
        return result
      }
      
      // 处理数组
      if (Array.isArray(obj1) && Array.isArray(obj2)) {
        if (this.strictArrayMode) {
          // 严格模式：按顺序比较
          const maxLength = Math.max(obj1.length, obj2.length)
          for (let i = 0; i < maxLength; i++) {
            const newPath = path ? `${path}[${i}]` : `[${i}]`
            if (i >= obj1.length) {
              result.push({
                type: 'added',
                path: newPath,
                value: obj2[i]
              })
            } else if (i >= obj2.length) {
              result.push({
                type: 'removed',
                path: newPath,
                value: obj1[i]
              })
            } else {
              result.push(...this.diff(obj1[i], obj2[i], newPath))
            }
          }
        } else {
          // 宽松模式：不按顺序比较
          const compared = new Set()
          
          // 检查obj1中的元素
          for (let i = 0; i < obj1.length; i++) {
            const found = obj2.some((item, j) => {
              if (compared.has(j)) return false
              const isEqual = JSON.stringify(this.sortObject(obj1[i])) === JSON.stringify(this.sortObject(item))
              if (isEqual) {
                compared.add(j)
                return true
              }
              return false
            })
            
            const newPath = path ? `${path}[${i}]` : `[${i}]`
            if (!found) {
              result.push({
                type: 'removed',
                path: newPath,
                value: obj1[i]
              })
            }
          }
          
          // 检查obj2中的新增元素
          for (let i = 0; i < obj2.length; i++) {
            if (!compared.has(i)) {
              const newPath = path ? `${path}[${i}]` : `[${i}]`
              result.push({
                type: 'added',
                path: newPath,
                value: obj2[i]
              })
            }
          }
        }
        return result
      }
      
      // 处理对象
      const keys1 = Object.keys(obj1)
      const keys2 = Object.keys(obj2)
      const allKeys = [...new Set([...keys1, ...keys2])]
      
      for (const key of allKeys) {
        const newPath = path ? `${path}.${key}` : key
        
        if (!(key in obj1)) {
          result.push({
            type: 'added',
            path: newPath,
            value: obj2[key]
          })
        } else if (!(key in obj2)) {
          result.push({
            type: 'removed',
            path: newPath,
            value: obj1[key]
          })
        } else {
          result.push(...this.diff(obj1[key], obj2[key], newPath))
        }
      }
      
      return result
    },
    sortObject(obj) {
      if (typeof obj !== 'object' || obj === null) {
        return obj
      }
      if (Array.isArray(obj)) {
        return obj.map(item => this.sortObject(item))
      }
      return Object.keys(obj).sort().reduce((result, key) => {
        result[key] = this.sortObject(obj[key])
        return result
      }, {})
    },
    formatJson(side) {
      try {
        if (side === 'left' && this.leftJson) {
          const obj = JSON.parse(this.leftJson)
          this.leftJson = JSON.stringify(obj, null, 2)
        } else if (side === 'right' && this.rightJson) {
          const obj = JSON.parse(this.rightJson)
          this.rightJson = JSON.stringify(obj, null, 2)
        }
        this.onJsonChange()
      } catch (e) {
        console.error('格式化错误:', e)
      }
    },
    async copyJson(side) {
      try {
        const text = side === 'left' ? this.leftJson : this.rightJson
        await navigator.clipboard.writeText(text)
        this.showToast(this.$t('success.copied'))
      } catch (e) {
        console.error('复制失败:', e)
        this.showToast('复制失败', 'error')
      }
    },
    showToast(message, type = 'success') {
      // 创建toast元素
      const toast = document.createElement('div')
      toast.className = `toast ${type}`
      toast.textContent = message
      
      // 设置toast样式
      toast.style.position = 'fixed'
      toast.style.top = '20px'
      toast.style.right = '20px'
      toast.style.padding = '1rem 1.5rem'
      toast.style.borderRadius = '6px'
      toast.style.color = 'white'
      toast.style.fontSize = '14px'
      toast.style.fontWeight = '500'
      toast.style.boxShadow = '0 4px 12px rgba(0, 0, 0, 0.15)'
      toast.style.opacity = '0'
      toast.style.transform = 'translateX(100%)'
      toast.style.transition = 'all 0.3s ease'
      toast.style.zIndex = '1000'
      toast.style.fontFamily = 'inherit'
      
      // 设置不同类型的背景色
      if (type === 'success') {
        toast.style.background = 'linear-gradient(135deg, #4caf50 0%, #45a049 100%)'
      } else if (type === 'error') {
        toast.style.background = 'linear-gradient(135deg, #f44336 0%, #da190b 100%)'
      }
      
      // 添加到页面
      document.body.appendChild(toast)
      
      // 强制重排
      toast.offsetHeight
      
      // 显示toast
      setTimeout(() => {
        toast.style.opacity = '1'
        toast.style.transform = 'translateX(0)'
      }, 10)
      
      // 3秒后隐藏并移除toast
      setTimeout(() => {
        toast.style.opacity = '0'
        toast.style.transform = 'translateX(100%)'
        setTimeout(() => {
          if (toast.parentNode) {
            toast.parentNode.removeChild(toast)
          }
        }, 300)
      }, 3000)
    },
    loadExample1() {
      // 短示例
      this.leftJson = JSON.stringify({
        name: 'DiffJson',
        version: '1.0.0',
        features: ['JSON比较', '格式化', '复制'],
        settings: {
          theme: 'light',
          strictArrayMode: false
        }
      }, null, 2)
      
      this.rightJson = JSON.stringify({
        name: 'DiffJson',
        version: '1.0.1',
        features: ['JSON比较', '格式化', '复制', '示例'],
        settings: {
          theme: 'light',
          strictArrayMode: true
        }
      }, null, 2)
      
      this.onJsonChange()
    },
    loadExample2() {
      // 长示例
      this.leftJson = JSON.stringify({
        name: 'DiffJson',
        version: '1.0.0',
        description: '简单易用的JSON比较工具',
        features: ['JSON比较', '格式化', '复制', '多语言支持'],
        settings: {
          theme: 'light',
          strictArrayMode: false,
          fontSize: 14,
          fontFamily: 'monospace',
          colors: {
            primary: '#e8f5e8',
            secondary: '#2e7d32',
            error: '#e74c3c'
          }
        },
        languages: [
          {
            code: 'zh-CN',
            name: '中文',
            enabled: true
          },
          {
            code: 'en-US',
            name: 'English',
            enabled: true
          }
        ],
        stats: {
          totalUsers: 10000,
          monthlyActive: 5000,
          averageTime: '2.5m',
          successRate: 99.9
        },
        recentUpdates: [
          {
            version: '1.0.0',
            date: '2026-01-01',
            changes: [
              '初始版本发布',
              '支持JSON对象比较',
              '支持JSON数组比较'
            ]
          }
        ]
      }, null, 2)
      
      this.rightJson = JSON.stringify({
        name: 'DiffJson',
        version: '1.0.1',
        description: '专业的JSON比较工具',
        features: ['JSON比较', '格式化', '复制', '多语言支持', '实时比较'],
        settings: {
          theme: 'light',
          strictArrayMode: true,
          fontSize: 14,
          fontFamily: 'monospace',
          colors: {
            primary: '#e8f5e8',
            secondary: '#2e7d32',
            error: '#e74c3c',
            warning: '#f39c12'
          }
        },
        languages: [
          {
            code: 'zh-CN',
            name: '中文',
            enabled: true
          },
          {
            code: 'en-US',
            name: 'English',
            enabled: true
          },
          {
            code: 'ja-JP',
            name: '日本語',
            enabled: true
          },
          {
            code: 'ko-KR',
            name: '한국어',
            enabled: true
          },
          {
            code: 'fr-FR',
            name: 'Français',
            enabled: true
          }
        ],
        stats: {
          totalUsers: 15000,
          monthlyActive: 8000,
          averageTime: '2.0m',
          successRate: 99.95
        },
        recentUpdates: [
          {
            version: '1.0.1',
            date: '2026-01-28',
            changes: [
              '添加多语言支持',
              '优化UI界面',
              '修复数组比较bug'
            ]
          },
          {
            version: '1.0.0',
            date: '2026-01-01',
            changes: [
              '初始版本发布',
              '支持JSON对象比较',
              '支持JSON数组比较'
            ]
          }
        ]
      }, null, 2)
      
      this.onJsonChange()
    }
  }
}
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}

.main {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
}

.footer {
  background: linear-gradient(135deg, #e8f5e8 0%, #c8e6c9 100%);
  padding: 1.5rem;
  text-align: center;
  border-top: 1px solid #a5d6a7;
  color: #388e3c;
  position: sticky;
  bottom: 0;
  z-index: 10;
  box-shadow: 0 -2px 4px rgba(0, 0, 0, 0.1);
}

.footer p {
  margin: 0;
  font-size: 14px;
  font-weight: 500;
}

/* Toast提示样式 */
.toast {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 1rem 1.5rem;
  border-radius: 6px;
  color: white;
  font-size: 14px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  opacity: 0;
  transform: translateX(100%);
  transition: all 0.3s ease;
  z-index: 1000;
}

.toast.show {
  opacity: 1;
  transform: translateX(0);
}

.toast.success {
  background: linear-gradient(135deg, #4caf50 0%, #45a049 100%);
}

.toast.error {
  background: linear-gradient(135deg, #f44336 0%, #da190b 100%);
}

.header {
  background: linear-gradient(135deg, #e8f5e8 0%, #c8e6c9 100%);
  padding: 2rem;
  border-bottom: 1px solid #a5d6a7;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.header h1 {
  margin: 0;
  color: #2e7d32;
  font-size: 2.5rem;
  font-weight: 700;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}

.header p {
  margin: 0.5rem 0 0;
  color: #388e3c;
  font-size: 1.1rem;
}

.language-switcher {
  display: flex;
  gap: 0.5rem;
}

.language-select {
  padding: 0.75rem 1rem;
  border: 1px solid #a5d6a7;
  border-radius: 6px;
  background: #ffffff;
  color: #2e7d32;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.language-select:hover {
  border-color: #81c784;
  box-shadow: 0 2px 8px rgba(129, 199, 132, 0.3);
  transform: translateY(-1px);
}

.language-select:focus {
  outline: none;
  border-color: #66bb6a;
  box-shadow: 0 0 0 3px rgba(102, 187, 106, 0.2);
}

.main {
  flex: 1;
  padding: 2rem;
}

.control-panel {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  padding: 1rem;
  background: #f9f9f9;
  border-radius: 8px;
}

.buttons {
  display: flex;
  gap: 0.5rem;
}

.buttons button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background: #e8f5e8;
  color: #2e7d32;
  cursor: pointer;
  transition: all 0.3s ease;
}

.buttons button:hover {
  background: #c8e6c9;
  transform: translateY(-1px);
}

.array-mode {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.mode-toggle {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.mode-label-with-hint {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.mode-label {
  font-weight: 500;
  color: #2c3e50;
}

.hint-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  background: #e8f5e8;
  color: #2e7d32;
  border-radius: 50%;
  font-size: 12px;
  font-weight: bold;
  cursor: help;
  transition: all 0.3s ease;
}

.hint-icon:hover {
  background: #c8e6c9;
  transform: scale(1.1);
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
  cursor: pointer;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-slider {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #e0e0e0;
  transition: .4s;
  border-radius: 34px;
}

.toggle-slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .toggle-slider {
  background-color: #81c784;
}

input:checked + .toggle-slider:before {
  transform: translateX(26px);
}

.mode-status {
  font-size: 14px;
  font-weight: 500;
  color: #27ae60;
  padding: 0.25rem 0.5rem;
  background: #f0fff4;
  border-radius: 4px;
  border: 1px solid #c6f6d5;
}

.diff-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 1rem;
  height: calc(100vh - 300px);
}

.json-input {
  display: flex;
  flex-direction: column;
  background: #f9f9f9;
  border-radius: 8px;
  padding: 1rem;
}

.json-input h2 {
  margin: 0 0 1rem;
  font-size: 1.2rem;
  color: #2c3e50;
}

.code-container {
  flex: 1;
  display: flex;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.line-numbers {
  width: 50px;
  background: #f5f5f5;
  border-right: 1px solid #e0e0e0;
  padding: 1rem 0.5rem;
  text-align: right;
  font-family: monospace;
  font-size: 14px;
  line-height: 1.5;
  color: #7f8c8d;
  overflow: hidden;
}

.line-number {
  height: 1.5em;
  line-height: 1.5em;
}

.json-input textarea {
  flex: 1;
  padding: 1rem;
  border: none;
  resize: none;
  font-family: monospace;
  font-size: 14px;
  line-height: 1.5;
}

.json-input textarea:focus {
  outline: none;
  box-shadow: none;
}

.input-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.input-header h2 {
  margin: 0;
  font-size: 1.2rem;
  color: #2c3e50;
}

.input-buttons {
  display: flex;
  gap: 0.5rem;
}

.input-buttons button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background: #e8f5e8;
  color: #2e7d32;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
}

.input-buttons button:hover {
  background: #c8e6c9;
  transform: translateY(-1px);
}

.error {
  margin-bottom: 0.5rem;
  padding: 0.5rem;
  background: #ffebee;
  color: #c62828;
  border-radius: 4px;
  font-size: 12px;
}

.diff-result {
  display: flex;
  flex-direction: column;
  background: #f9f9f9;
  border-radius: 8px;
  padding: 1rem;
  overflow: hidden;
}

.diff-result h2 {
  margin: 0 0 1rem;
  font-size: 1.2rem;
  color: #2c3e50;
}

.no-diff {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #7f8c8d;
  font-style: italic;
}

.diff-content {
  flex: 1;
  overflow-y: auto;
  font-family: monospace;
  font-size: 14px;
  line-height: 1.5;
  padding: 1rem;
  background: #f9f9f9;
  border-radius: 4px;
}

.diff-item {
  padding: 0.5rem;
  margin-bottom: 0.25rem;
  border-radius: 4px;
  transition: all 0.2s ease;
}

.diff-item:hover {
  transform: translateX(5px);
}

.diff-symbol {
  display: inline-block;
  width: 30px;
  font-weight: bold;
}

.diff-path {
  display: inline-block;
  width: 200px;
  font-weight: bold;
  color: #2c3e50;
}

.diff-value {
  display: inline-block;
  color: #27ae60;
}

.diff-old-value {
  display: inline-block;
  color: #e74c3c;
  margin-right: 10px;
}

.diff-arrow {
  display: inline-block;
  margin: 0 10px;
  color: #7f8c8d;
}

.diff-new-value {
  display: inline-block;
  color: #27ae60;
}

.equal {
  color: #7f8c8d;
}

.added {
  color: #2e7d32;
  background: rgba(46, 125, 50, 0.1);
  padding: 0.25rem;
  border-radius: 2px;
}

.removed {
  color: #c62828;
  background: rgba(198, 40, 40, 0.1);
  padding: 0.25rem;
  border-radius: 2px;
}

.modified {
  color: #ef6c00;
  background: rgba(239, 108, 0, 0.1);
  padding: 0.25rem;
  border-radius: 2px;
}

.footer {
  background: linear-gradient(135deg, #f0f9ff 0%, #e6f7ff 100%);
  padding: 1rem;
  text-align: center;
  border-top: 1px solid #e0e0e0;
  color: #7f8c8d;
}

@media (max-width: 1200px) {
  .diff-container {
    grid-template-columns: 1fr 1fr;
    grid-template-rows: auto 1fr;
    height: auto;
  }
  
  .diff-result {
    grid-column: 1 / -1;
    min-height: 300px;
  }
}

@media (max-width: 768px) {
  .diff-container {
    grid-template-columns: 1fr;
  }
  
  .control-panel {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }
  
  .buttons {
    flex-wrap: wrap;
  }
}
</style>