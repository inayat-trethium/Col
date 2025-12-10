# Vue Columnizer

A Vue 3 component that replicates all jQuery Columnizer functionality without jQuery dependency.  This component splits content into responsive, dynamic columns with support for print compatibility, manual breaks, and intelligent content splitting.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Props Reference](#props-reference)
- [Events](#events)
- [Exposed Methods](#exposed-methods)
- [CSS Classes](#css-classes)
- [Migration from jQuery Columnizer](#migration-from-jquery-columnizer)
- [Examples](#examples)
- [Print Support](#print-support)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Features

- ✅ **Zero jQuery dependency** - Pure Vue 3 implementation
- ✅ **Fully reactive** - Automatically responds to content and prop changes
- ✅ **Responsive** - Rebuilds columns on window resize
- ✅ **Dynamic height adjustments** - Balances column heights automatically
- ✅ **Print compatibility** - Proper page break handling for printing
- ✅ **Manual column breaks** - Support for forced column breaks
- ✅ **Content splitting** - Intelligent text and element splitting
- ✅ **TypeScript support** - Full type definitions included
- ✅ **Composition API** - Modern Vue 3 patterns
- ✅ **Drop-in replacement** - Same options as jQuery Columnizer

---

## Installation

### Option 1: Copy Files Directly

1. Copy `VueColumnizer. vue` to your components directory
2. Copy `useColumnizer.ts` to your composables directory (optional)

### Option 2: Register Globally

```javascript
// main.js or main.ts
import { createApp } from 'vue'
import App from './App.vue'
import VueColumnizer from './components/VueColumnizer.vue'

const app = createApp(App)
app.component('VueColumnizer', VueColumnizer)
app.mount('#app')
```

### Option 3: Local Import

```vue
<script setup>
import VueColumnizer from '@/components/VueColumnizer.vue'
</script>
```

---

## Quick Start

### Basic Usage

```vue
<template>
  <VueColumnizer : content="articleContent" :width="300" />
</template>

<script setup>
import { ref } from 'vue'
import VueColumnizer from './components/VueColumnizer.vue'

const articleContent = ref(`
  <h1>Article Title</h1>
  <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
  Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. </p>
  <p>Ut enim ad minim veniam, quis nostrud exercitation ullamco 
  laboris nisi ut aliquip ex ea commodo consequat.</p>
`)
</script>
```

### Fixed Number of Columns

```vue
<template>
  <VueColumnizer 
    :content="content" 
    :columns="3" 
    @done="onComplete"
  />
</template>
```

### With Overflow Handling

```vue
<template>
  <div>
    <VueColumnizer 
      :content="longContent" 
      :overflow="{ height: 400 }"
      @overflow="handleOverflow"
    />
    <div v-if="overflowContent" class="read-more">
      <h3>Continued...</h3>
      <div v-html="overflowContent"></div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const overflowContent = ref('')

const handleOverflow = (content) => {
  overflowContent. value = content
}
</script>
```

---

## Props Reference

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `content` | `String` | `''` | HTML content to columnize |
| `width` | `Number` | `400` | Target column width in pixels.  Columns are calculated based on container width / this value |
| `columns` | `Number \| Boolean` | `false` | Fixed number of columns.  When set, overrides `width` calculation |
| `buildOnce` | `Boolean` | `false` | If `true`, columns are built once and don't respond to window resize |
| `height` | `Number \| Boolean` | `false` | Fixed height for columns.  Enables horizontal scrolling mode |
| `overflow` | `Object \| Boolean` | `false` | Overflow configuration:  `{ height: number }`. Content exceeding height triggers `@overflow` event |
| `columnFloat` | `String` | `'left'` | Float direction for columns: `'left'` or `'right'` |
| `lastNeverTallest` | `Boolean` | `false` | If `true`, ensures the last column is never the tallest |
| `accuracy` | `Number \| Boolean` | `false` | Minimum characters to jump when splitting text.  Lower = more accurate but slower |
| `precise` | `Boolean` | `false` | If `true`, uses precise decimal column widths instead of rounded values |
| `manualBreaks` | `Boolean` | `false` | If `true`, only breaks columns at `.columnbreak` elements |
| `disableSingle` | `Boolean` | `false` | If `true`, prevents falling back to single column when container is narrow |
| `cssClassPrefix` | `String` | `''` | Prefix for all CSS classes (e.g., `'my'` produces `'my-column'`) |
| `resizeDelay` | `Number` | `200` | Debounce delay in milliseconds for resize handling |

---

## Events

| Event | Payload | Description |
|-------|---------|-------------|
| `@done` | `void` | Emitted when columnization is complete |
| `@overflow` | `string` (HTML content) | Emitted when content overflows.  Contains the overflow HTML |

### Event Examples

```vue
<template>
  <VueColumnizer 
    : content="content"
    : overflow="{ height: 300 }"
    @done="onColumnizationComplete"
    @overflow="onContentOverflow"
  />
</template>

<script setup>
const onColumnizationComplete = () => {
  console.log('Columns have been built!')
}

const onContentOverflow = (overflowHtml) => {
  console.log('Overflow content:', overflowHtml)
  // Handle overflow - show "read more", paginate, etc. 
}
</script>
```

---

## Exposed Methods

Access these methods via template ref:

```vue
<template>
  <VueColumnizer ref="columnizerRef" : content="content" />
  <button @click="rebuild">Rebuild</button>
  <button @click="reset">Reset</button>
</template>

<script setup>
import { ref } from 'vue'

const columnizerRef = ref(null)

const rebuild = () => {
  columnizerRef.value?. recolumnize()
}

const reset = () => {
  columnizerRef. value?.uncolumnize()
}
</script>
```

| Method | Description |
|--------|-------------|
| `recolumnize()` | Force rebuild of columns |
| `uncolumnize()` | Revert to single column with original content |

---

## CSS Classes

The component uses the following CSS classes (all support `cssClassPrefix`):

### Column Classes

| Class | Applied To | Description |
|-------|-----------|-------------|
| `.column` | Each column | Base column class |
| `.first` | First column | Identifies the first column |
| `.last` | Last column | Identifies the last column |
| `.split` | Split elements | Applied to elements that span multiple columns |

### Content Control Classes

Add these classes to your content to control column behavior: 

| Class | Description |
|-------|-------------|
| `.dontsplit` | Prevents the element from being split across columns |
| `.dontend` | Prevents the element from being the last item in a column |
| `.columnbreak` | Forces a column break before this element |
| `.removeiffirst` | Removes element if it's first in a column |
| `.removeiflast` | Removes element if it's last in a column |

### Example:  Controlling Content Splitting

```vue
<template>
  <VueColumnizer : content="content" : columns="3" />
</template>

<script setup>
const content = `
  <h1 class="dontend">Important Heading</h1>
  <p>This paragraph can be split across columns. </p>
  
  <div class="dontsplit">
    <h2>Keep Together</h2>
    <p>This entire div will stay in one column.</p>
  </div>
  
  <p>More content here...</p>
  
  <div class="columnbreak"></div>
  
  <h2>New Section</h2>
  <p>This starts in a new column.</p>
`
</script>
```

---

## Migration from jQuery Columnizer

### Before:  jQuery Columnizer

```html
<! DOCTYPE html>
<html>
<head>
  <script src="https://code.jquery.com/jquery-3.6.0.min. js"></script>
  <script src="jquery. columnizer.js"></script>
  <style>
    .column { padding: 10px; }
    .first. column { padding-left: 0; }
    . last.column { padding-right: 0; }
  </style>
</head>
<body>
  <div id="content">
    <h1>Article Title</h1>
    <p>Lorem ipsum dolor sit amet... </p>
    <p>Consectetur adipiscing elit... </p>
  </div>

  <script>
    $(function() {
      $('h1, h2, h3').addClass('dontend');
      $('table, li').addClass('dontsplit');
      
      $('#content').columnize({
        width: 300,
        buildOnce: false,
        lastNeverTallest: true,
        doneFunc: function() {
          console. log('Done!');
        }
      });
    });
  </script>
</body>
</html>
```

### After: Vue Columnizer

```vue
<template>
  <VueColumnizer
    :content="preparedContent"
    :width="300"
    :build-once="false"
    : last-never-tallest="true"
    @done="onDone"
  />
</template>

<script setup>
import { ref, computed } from 'vue'
import VueColumnizer from './components/VueColumnizer.vue'

const rawContent = ref(`
  <h1>Article Title</h1>
  <p>Lorem ipsum dolor sit amet... </p>
  <p>Consectetur adipiscing elit...</p>
`)

// Prepare content with appropriate classes
const preparedContent = computed(() => {
  const div = document.createElement('div')
  div.innerHTML = rawContent. value
  
  // Add dontend to headings
  div. querySelectorAll('h1, h2, h3').forEach(el => {
    el.classList. add('dontend')
  })
  
  // Add dontsplit to tables and list items
  div. querySelectorAll('table, li').forEach(el => {
    el.classList. add('dontsplit')
  })
  
  return div.innerHTML
})

const onDone = () => {
  console.log('Done!')
}
</script>

<style>
. column { padding: 10px; }
. first.column { padding-left: 0; }
.last.column { padding-right: 0; }
</style>
```

### Option Mapping Table

| jQuery Columnizer | Vue Columnizer | Notes |
|-------------------|----------------|-------|
| `width: 400` | `:width="400"` | Identical |
| `columns: 3` | `:columns="3"` | Identical |
| `buildOnce: true` | `:build-once="true"` | kebab-case in template |
| `overflow: {height: 500, id: '#overflow'}` | `:overflow="{height: 500}"` | Use `@overflow` event |
| `columnFloat: 'right'` | `:column-float="'right'"` | kebab-case |
| `lastNeverTallest:  true` | `:last-never-tallest="true"` | kebab-case |
| `accuracy: 20` | `:accuracy="20"` | Identical |
| `precise: true` | `:precise="true"` | Identical |
| `manualBreaks: true` | `:manual-breaks="true"` | kebab-case |
| `disableSingle: true` | `:disable-single="true"` | kebab-case |
| `cssClassPrefix: 'my'` | `:css-class-prefix="'my'"` | kebab-case |
| `doneFunc: callback` | `@done="callback"` | Vue event |
| `ignoreImageLoading: true` | N/A | Handled automatically |
| `target: '#other'` | N/A | Use Vue refs instead |
| `.uncolumnize()` | `ref.uncolumnize()` | Exposed method |
| `.columnize()` (rebuild) | `ref.recolumnize()` | Exposed method |

---

## Examples

### Example 1: Newspaper Layout

```vue
<template>
  <div class="newspaper">
    <h1 class="headline">{{ headline }}</h1>
    <VueColumnizer
      :content="articleBody"
      :width="250"
      :last-never-tallest="true"
      css-class-prefix="news"
    />
  </div>
</template>

<script setup>
const headline = 'Breaking News: Vue Columnizer Released!'
const articleBody = `
  <p class="lead">A new Vue 3 component promises to revolutionize 
  multi-column layouts on the web. </p>
  <p>The component, released today, offers a complete replacement 
  for the aging jQuery Columnizer plugin... </p>
  <!-- more paragraphs -->
`
</script>

<style>
.newspaper {
  max-width: 1200px;
  margin: 0 auto;
  font-family: Georgia, serif;
}

.headline {
  text-align: center;
  border-bottom: 2px solid #333;
  padding-bottom: 10px;
}

.news-column {
  padding: 0 15px;
  text-align: justify;
}

.news-first {
  padding-left: 0;
}

. news-last {
  padding-right:  0;
}
</style>
```

### Example 2: Fixed Columns with Manual Breaks

```vue
<template>
  <VueColumnizer
    :content="content"
    :manual-breaks="true"
    :columns="3"
  />
</template>

<script setup>
const content = `
  <div>
    <h2>Section 1</h2>
    <p>Content for section 1...</p>
  </div>
  
  <div class="columnbreak"></div>
  
  <div>
    <h2>Section 2</h2>
    <p>Content for section 2... </p>
  </div>
  
  <div class="columnbreak"></div>
  
  <div>
    <h2>Section 3</h2>
    <p>Content for section 3...</p>
  </div>
`
</script>
```

### Example 3: Paginated Content with Overflow

```vue
<template>
  <div class="book-reader">
    <div class="page">
      <VueColumnizer
        :content="currentPageContent"
        : columns="2"
        :overflow="{ height: 600 }"
        @overflow="storeNextPage"
        @done="onPageReady"
      />
    </div>
    
    <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 0">
        Previous
      </button>
      <span>Page {{ currentPage + 1 }} of {{ pages.length }}</span>
      <button @click="nextPage" :disabled="currentPage >= pages.length - 1">
        Next
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const fullContent = ref(`<!-- Your long book content here -->`)
const pages = ref([])
const currentPage = ref(0)

// Initialize first page
pages.value. push(fullContent.value)

const currentPageContent = computed(() => pages.value[currentPage.value] || '')

const storeNextPage = (overflowContent) => {
  if (overflowContent && currentPage.value === pages.value.length - 1) {
    pages.value.push(overflowContent)
  }
}

const prevPage = () => {
  if (currentPage.value > 0) currentPage.value--
}

const nextPage = () => {
  if (currentPage.value < pages. value.length - 1) currentPage.value++
}

const onPageReady = () => {
  console.log(`Page ${currentPage. value + 1} rendered`)
}
</script>

<style>
. book-reader {
  max-width:  800px;
  margin: 0 auto;
}

.page {
  min-height: 600px;
  border: 1px solid #ccc;
  padding: 20px;
  background: #fff;
}

. pagination {
  display: flex;
  justify-content: center;
  gap:  20px;
  margin-top:  20px;
}
</style>
```

### Example 4: Using the Composable

```vue
<template>
  <div>
    <VueColumnizer
      ref="columnizerRef"
      : content="preparedContent"
      v-bind="defaultOptions"
      @done="onDone"
    />
    
    <div class="controls">
      <button @click="recolumnize">Rebuild Columns</button>
      <button @click="uncolumnize">Show as Single Column</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import VueColumnizer from './components/VueColumnizer.vue'
import { useColumnizer } from './composables/useColumnizer'

const rawContent = ref(`
  <h1>Dynamic Content</h1>
  <table>
    <tr><td>Data 1</td><td>Data 2</td></tr>
  </table>
  <p>Paragraph content...</p>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
`)

const {
  columnizerRef,
  recolumnize,
  uncolumnize,
  prepareContent,
  defaultOptions
} = useColumnizer({
  width: 350,
  columns:  2,
  lastNeverTallest: true
})

// Automatically add dontsplit/dontend classes
const preparedContent = computed(() => {
  const div = document.createElement('div')
  div.innerHTML = rawContent.value
  return prepareContent(div)
})

const onDone = () => {
  console. log('Columnization complete!')
}
</script>
```

---

## Print Support

The component includes built-in print styles for proper page break handling:

```css
@media print {
  . vue-columnizer . column {
    break-inside: avoid-column;
    page-break-inside: avoid;
  }
  
  .vue-columnizer .dontsplit {
    break-inside: avoid;
    page-break-inside: avoid;
  }
  
  .vue-columnizer .dontend {
    break-after: avoid;
    page-break-after: avoid;
  }
}
```

### Custom Print Styles

You can add additional print styles in your app: 

```css
@media print {
  .vue-columnizer {
    /* Force single column for print */
    . column {
      width: 100% !important;
      float: none !important;
    }
  }
  
  /* Or keep columns but adjust */
  .vue-columnizer .column {
    padding: 0 10px;
  }
}
```

---

## Troubleshooting

### Columns not building correctly

**Problem:** Content isn't splitting into columns properly. 

**Solutions:**
1. Ensure content is visible (`display: block`, not `display: none`)
2. Check that container has a defined width
3. Verify content prop contains valid HTML

```vue
<!-- Make sure container is visible -->
<div style="width: 100%;">
  <VueColumnizer : content="content" />
</div>
```

### Images causing layout issues

**Problem:** Images load after columnization, breaking layouts.

**Solution:** Wait for images to load or use fixed dimensions: 

```vue
<script setup>
import { ref, onMounted } from 'vue'

const content = ref('')
const isReady = ref(false)

onMounted(async () => {
  // Load content
  content.value = await fetchContent()
  
  // Wait for images
  await nextTick()
  const images = document.querySelectorAll('img')
  await Promise.all(
    Array.from(images).map(img => 
      img.complete ? Promise.resolve() : 
      new Promise(resolve => img.onload = resolve)
    )
  )
  
  isReady. value = true
})
</script>

<template>
  <VueColumnizer v-if="isReady" :content="content" />
</template>
```

### Performance with large content

**Problem:** Columnization is slow with very large content.

**Solutions:**
1. Increase `accuracy` value (fewer text split calculations)
2. Use `buildOnce: true` if responsiveness isn't needed
3. Increase `resizeDelay` to reduce rebuild frequency

```vue
<VueColumnizer
  :content="largeContent"
  :accuracy="50"
  :build-once="true"
  :resize-delay="500"
/>
```

### Content not reactive

**Problem:** Changing `content` prop doesn't update columns.

**Solution:** Ensure you're using reactive data:

```vue
<script setup>
import { ref } from 'vue'

// ✅ Correct - reactive
const content = ref('<p>Initial content</p>')

// ❌ Wrong - not reactive
let content = '<p>Initial content</p>'

// Update correctly
const updateContent = (newContent) => {
  content.value = newContent  // ✅ Triggers re-columnization
}
</script>
```

---

## Browser Support

- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

Note: IE11 is not supported (Vue 3 requirement).

---

## License

This Vue Columnizer component is released under the MIT License. 

Original jQuery Columnizer by Adam Wulf - MIT/GPLv2 License. 

---

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests. 

### Development

```bash
# Clone the repository
git clone https://github.com/inayat-trethium/Col. git

# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build
```

---

## Changelog

### v1.0.0
- Initial release
- Full jQuery Columnizer feature parity
- Vue 3 Composition API
- TypeScript support
- Print compatibility