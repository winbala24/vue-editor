<template>
    <div class="editor-container">
      <div class="title-bar">
        <input 
          type="text" 
          v-model="documentTitle" 
          placeholder="Enter document title"
          class="title-input"
        >
      </div>
      <div class="toolbar">
        <button @click="handleBold" :class="{ active: isBold }">
          <strong>B</strong>
        </button>
        <button @click="handleItalic" :class="{ active: isItalic }">
          <em>I</em>
        </button>
        <button @click="handleUnderline" :class="{ active: isUnderline }">
          <u>U</u>
        </button>
        <select v-model="fontSize">
          <option v-for="size in [12, 14, 16, 18, 20, 24]" :key="size" :value="size">
            {{ size }}px
          </option>
        </select>
        <button @click="saveFile">Save</button>
        <input
          type="file"
          accept=".work"
          @change="openFile"
          ref="fileInput"
          style="display: none"
        >
        <button @click="$refs.fileInput.click()">Open</button>
      </div>
      
      <div
        class="editor"
        contenteditable="true"
        ref="editor"
        @input="handleInput"
        @keydown="handleKeyDown"
      ></div>
      
      <div class="status-bar">
        <span>Title: {{ documentTitle || 'Untitled' }}</span>
        <span>Characters: {{ charCount }}</span>
        <span>Words: {{ wordCount }}</span>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'RichTextEditor',
    data() {
      return {
        content: '',
        documentTitle: '',
        charCount: 0,
        wordCount: 0,
        fontSize: 16,
        isBold: false,
        isItalic: false,
        isUnderline: false
      }
    },
    methods: {
      handleInput(e) {
        this.content = e.target.innerHTML
        this.updateCounts()
      },
      
      handleKeyDown(e) {
        if (e.key === 'Tab') {
          e.preventDefault()
          document.execCommand('insertHTML', false, '&nbsp;&nbsp;&nbsp;&nbsp;')
        }
      },
      
      updateCounts() {
        const plainText = this.content.replace(/<[^>]*>/g, '')
        this.charCount = plainText.length
        this.wordCount = plainText.trim().split(/\s+/).filter(word => word.length > 0).length
      },
      
      handleBold() {
        document.execCommand('bold', false, null)
        this.isBold = !this.isBold
      },
      
      handleItalic() {
        document.execCommand('italic', false, null)
        this.isItalic = !this.isItalic
      },
      
      handleUnderline() {
        document.execCommand('underline', false, null)
        this.isUnderline = !this.isUnderline
      },
      
      saveFile() {
        const content = {
          title: this.documentTitle,
          content: this.$refs.editor.innerHTML
        }
        
        const blob = new Blob([JSON.stringify(content)], { type: 'application/json' })
        const url = window.URL.createObjectURL(blob)
        const a = document.createElement('a')
        
        const fileName = this.documentTitle ? 
          `${this.documentTitle}.work` : 
          'untitled.work'
        
        a.href = url
        a.download = fileName
        document.body.appendChild(a)
        a.click()
        
        // Cleanup
        window.URL.revokeObjectURL(url)
        document.body.removeChild(a)
      },
      
      openFile(event) {
        const file = event.target.files[0]
        if (!file) return
        
        const reader = new FileReader()
        reader.onload = (e) => {
          try {
            const data = JSON.parse(e.target.result)
            this.documentTitle = data.title || ''
            this.$refs.editor.innerHTML = data.content || ''
            this.updateCounts()
          } catch (error) {
            // Handle legacy files that weren't saved with title
            this.$refs.editor.innerHTML = e.target.result
            this.documentTitle = file.name.replace('.work', '')
            this.updateCounts()
          }
        }
        reader.readAsText(file)
      }
    },
    
    watch: {
      fontSize(newSize) {
        this.$refs.editor.style.fontSize = `${newSize}px`
      }
    }
  }
  </script>
  
  <style scoped>
  .editor-container {
    border: 1px solid #ccc;
    border-radius: 4px;
    margin: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  .title-bar {
    padding: 10px;
    border-bottom: 1px solid #eee;
    background-color: #fff;
  }
  
  .title-input {
    width: 100%;
    padding: 8px;
    font-size: 18px;
    border: 1px solid #ddd;
    border-radius: 4px;
    outline: none;
  }
  
  .title-input:focus {
    border-color: #4a90e2;
  }
  
  .toolbar {
    padding: 10px;
    border-bottom: 1px solid #eee;
    display: flex;
    gap: 10px;
    background-color: #f5f5f5;
  }
  
  .toolbar button {
    padding: 5px 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: white;
    cursor: pointer;
  }
  
  .toolbar button.active {
    background-color: #e6e6e6;
  }
  
  .toolbar select {
    padding: 5px;
    border-radius: 4px;
  }
  
  .editor {
    min-height: 300px;
    padding: 20px;
    outline: none;
    font-family: Arial, sans-serif;
  }
  
  .status-bar {
    padding: 10px;
    border-top: 1px solid #eee;
    display: flex;
    gap: 20px;
    background-color: #f5f5f5;
    font-size: 14px;
    color: #666;
  }
  </style>