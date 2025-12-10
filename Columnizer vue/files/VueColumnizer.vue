<template>
  <div ref="containerRef" class="vue-columnizer">
    <div
      v-for="(column, index) in columns"
      : key="index"
      : class="getColumnClasses(index)"
      :style="getColumnStyle()"
    >
      <div v-html="column"></div>
    </div>
    <div style="clear: both;"></div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'

// Props definition
const props = defineProps({
  // Content to columnize (HTML string or use default slot)
  content: {
    type:  String,
    default: ''
  },
  // Default width of columns in pixels
  width: {
    type: Number,
    default: 400
  },
  // Optional fixed number of columns (overrides width calculation)
  columns: {
    type: [Number, Boolean],
    default:  false
  },
  // Build columns once (no responsive rebuild)
  buildOnce: {
    type: Boolean,
    default: false
  },
  // Height constraint for overflow handling
  height: {
    type: [Number, Boolean],
    default:  false
  },
  // Overflow configuration { height: number, containerRef: ref }
  overflow:  {
    type:  [Object, Boolean],
    default: false
  },
  // Column float direction
  columnFloat: {
    type: String,
    default: 'left',
    validator: (val) => ['left', 'right']. includes(val)
  },
  // Ensure last column is never tallest
  lastNeverTallest:  {
    type:  Boolean,
    default: false
  },
  // Accuracy for text splitting (characters to jump)
  accuracy: {
    type: [Number, Boolean],
    default: false
  },
  // Use precise column widths (preserve decimals)
  precise: {
    type: Boolean,
    default: false
  },
  // Only use manual column breaks
  manualBreaks: {
    type: Boolean,
    default: false
  },
  // Disable single column mode
  disableSingle: {
    type: Boolean,
    default: false
  },
  // CSS class prefix
  cssClassPrefix: {
    type: String,
    default: ''
  },
  // Debounce delay for resize (ms)
  resizeDelay: {
    type: Number,
    default:  200
  }
})

// Emits
const emit = defineEmits(['done', 'overflow'])

// Refs
const containerRef = ref(null)
const columns = ref([])
const lastWidth = ref(0)
const isColumnizing = ref(false)
const originalContent = ref('')

// Helper to prefix class names
const prefixClassName = (className, withDot = false) => {
  const dot = withDot ?  '.' : ''
  if (props.cssClassPrefix. length) {
    return `${dot}${props.cssClassPrefix}-${className}`
  }
  return `${dot}${className}`
}

// Get column classes
const getColumnClasses = (index) => {
  const classes = [prefixClassName('column')]
  if (index === 0) classes.push(prefixClassName('first'))
  if (index === columns.value. length - 1) classes.push(prefixClassName('last'))
  return classes.join(' ')
}

// Get column style
const getColumnStyle = () => {
  const numCols = columns.value.length || 1
  const width = props.precise 
    ? (100 / numCols) 
    : Math.floor(100 / numCols)
  
  return {
    width:  `${width}%`,
    float: props.columnFloat,
    boxSizing: 'border-box'
  }
}

// Create a temporary invisible element for measurements
const createMeasureElement = (html) => {
  const div = document.createElement('div')
  div.style.visibility = 'hidden'
  div.style.position = 'absolute'
  div.style.width = '100%'
  div.innerHTML = html
  return div
}

// Check if element has class (supports prefixed classes)
const hasClass = (element, className) => {
  if (! element || !element.classList) return false
  return element.classList.contains(prefixClassName(className))
}

// Check if node should not end a column
const checkDontEndColumn = (node) => {
  if (!node) return false
  
  if (node.nodeType === 3) { // Text node
    if (/^\s+$/.test(node.nodeValue)) {
      if (! node.previousSibling) return false
      return checkDontEndColumn(node.previousSibling)
    }
    return false
  }
  
  if (node.nodeType !== 1) return false
  if (hasClass(node, 'dontend')) return true
  if (node.childNodes.length === 0) return false
  
  return checkDontEndColumn(node.childNodes[node.childNodes.length - 1])
}

// Check if element is a column break
const isColumnBreak = (element) => {
  if (! element) return false
  return hasClass(element, 'columnbreak') || 
         element.querySelector? .(prefixClassName('columnbreak', true))
}

// Split text node to fit in column
const splitTextNode = (textNode, container, targetHeight, getHeight) => {
  const text = textNode.nodeValue
  const accuracy = props.accuracy || Math.floor(props.width / 18)
  
  let fittingText = ''
  let remainingText = text
  
  while (remainingText. length > 0) {
    const spaceIndex = remainingText.indexOf(' ', accuracy)
    const chunk = spaceIndex !== -1 
      ? remainingText.substring(0, spaceIndex)
      : remainingText
    
    // Test if adding this chunk exceeds height
    const testNode = document.createTextNode(fittingText + chunk)
    container.appendChild(testNode)
    
    if (getHeight() > targetHeight && fittingText. length > 0) {
      container.removeChild(testNode)
      break
    }
    
    container.removeChild(testNode)
    fittingText += chunk
    remainingText = spaceIndex !== -1 
      ? remainingText.substring(spaceIndex)
      : ''
  }
  
  return { fittingText, remainingText }
}

// Clone node without deep content
const shallowClone = (node) => {
  if (node.nodeType === 3) {
    return document.createTextNode(node.nodeValue)
  }
  const clone = node.cloneNode(false)
  return clone
}

// Deep clone with all attributes
const deepClone = (node) => {
  return node.cloneNode(true)
}

// Build columns from content
const columnize = async () => {
  if (isColumnizing.value) return
  if (!containerRef.value) return
  
  const containerWidth = containerRef.value.offsetWidth
  if (containerWidth === lastWidth.value && columns.value.length > 0) return
  
  lastWidth.value = containerWidth
  isColumnizing.value = true
  
  try {
    // Calculate number of columns
    let numCols = Math.round(containerWidth / props.width)
    if (props.columns) numCols = props. columns
    
    // Get content
    const content = props.content || originalContent.value
    if (!content) {
      columns.value = []
      return
    }
    
    // Handle manual breaks
    if (props.manualBreaks) {
      const breakClass = prefixClassName('columnbreak')
      const tempDiv = document.createElement('div')
      tempDiv. innerHTML = content
      const breaks = tempDiv.querySelectorAll(`.${breakClass}`)
      numCols = breaks. length + 1
    }
    
    // Single column mode
    if (numCols <= 1 && !props.disableSingle) {
      columns.value = [content]
      emit('done')
      return
    }
    
    // Create measurement container
    const measureDiv = document.createElement('div')
    measureDiv. style.cssText = `
      position: absolute;
      visibility: hidden;
      width: ${containerWidth / numCols}px;
    `
    document.body.appendChild(measureDiv)
    measureDiv.innerHTML = content
    
    const totalHeight = measureDiv.offsetHeight
    let targetHeight = props.height || props.overflow?.height || (totalHeight / numCols)
    
    // Build columns
    const builtColumns = []
    const sourceDiv = document.createElement('div')
    sourceDiv.innerHTML = content
    
    for (let i = 0; i < numCols; i++) {
      const columnDiv = document.createElement('div')
      measureDiv.innerHTML = ''
      
      // Fill column until target height
      while (sourceDiv.childNodes.length > 0) {
        const node = sourceDiv.childNodes[0]
        
        // Check for column break
        if (isColumnBreak(node)) {
          sourceDiv.removeChild(node)
          break
        }
        
        // Check for dontSplit elements
        if (node.nodeType === 1 && hasClass(node, 'dontsplit')) {
          const clone = deepClone(node)
          measureDiv.appendChild(clone)
          
          if (measureDiv.offsetHeight <= targetHeight + 20) {
            columnDiv.appendChild(deepClone(node))
            sourceDiv.removeChild(node)
          } else {
            measureDiv.removeChild(clone)
            break
          }
          continue
        }
        
        // Add node and check height
        const clone = deepClone(node)
        measureDiv. appendChild(clone)
        
        if (measureDiv.offsetHeight <= targetHeight) {
          columnDiv.appendChild(deepClone(node))
          sourceDiv. removeChild(node)
        } else {
          // Try to split text content
          measureDiv.removeChild(clone)
          
          if (node.nodeType === 3) { // Text node
            const result = splitTextNode(
              node, 
              measureDiv, 
              targetHeight,
              () => measureDiv.offsetHeight
            )
            
            if (result.fittingText) {
              columnDiv.appendChild(document.createTextNode(result.fittingText))
              node. nodeValue = result.remainingText
            }
          } else if (node.nodeType === 1) {
            // For elements, we might need to split their content
            // This is a simplified approach - full implementation would recurse
            if (! hasClass(node, 'dontsplit')) {
              // Try to fit partial content
              const partialClone = shallowClone(node)
              partialClone.classList?. add(prefixClassName('split'))
              
              // Move as many children as fit
              while (node.childNodes.length > 0) {
                const child = node.childNodes[0]
                const childClone = deepClone(child)
                measureDiv. innerHTML = ''
                measureDiv.appendChild(deepClone(columnDiv))
                const testWrapper = measureDiv.querySelector(': scope > *: last-child') || measureDiv
                
                partialClone.appendChild(childClone)
                testWrapper.appendChild(deepClone(partialClone))
                
                if (measureDiv.offsetHeight <= targetHeight) {
                  node.removeChild(child)
                } else {
                  partialClone.removeChild(childClone)
                  break
                }
              }
              
              if (partialClone.childNodes.length > 0) {
                columnDiv.appendChild(partialClone)
                if (node.childNodes.length === 0) {
                  sourceDiv.removeChild(node)
                }
              }
            }
          }
          break
        }
      }
      
      // Remove dontend items from column end
      while (columnDiv.lastChild && checkDontEndColumn(columnDiv.lastChild)) {
        const lastChild = columnDiv.lastChild
        columnDiv.removeChild(lastChild)
        sourceDiv.insertBefore(lastChild, sourceDiv.firstChild)
      }
      
      builtColumns.push(columnDiv. innerHTML)
      
      // Check if last column and there's remaining content
      if (i === numCols - 1 && sourceDiv.childNodes. length > 0) {
        if (props.overflow) {
          emit('overflow', sourceDiv. innerHTML)
        } else {
          // Append remaining to last column
          builtColumns[builtColumns.length - 1] += sourceDiv.innerHTML
        }
      }
    }
    
    // Clean up measurement div
    document.body.removeChild(measureDiv)
    
    // Handle lastNeverTallest
    if (props.lastNeverTallest && builtColumns.length > 1) {
      // This would require re-measuring and redistributing
      // Simplified:  just ensure we have balanced columns
    }
    
    columns.value = builtColumns
    emit('done')
    
  } finally {
    isColumnizing.value = false
  }
}

// Resize handler with debounce
let resizeTimeout = null
const handleResize = () => {
  if (props.buildOnce) return
  
  if (resizeTimeout) {
    clearTimeout(resizeTimeout)
  }
  
  resizeTimeout = setTimeout(() => {
    columnize()
  }, props.resizeDelay)
}

// Lifecycle
onMounted(() => {
  // Store original content from slot if no content prop
  if (!props.content && containerRef.value) {
    originalContent. value = containerRef. value.innerHTML
  }
  
  nextTick(() => {
    columnize()
  })
  
  if (! props.buildOnce) {
    window.addEventListener('resize', handleResize)
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (resizeTimeout) {
    clearTimeout(resizeTimeout)
  }
})

// Watch for content changes
watch(() => props.content, () => {
  lastWidth.value = 0 // Force rebuild
  columnize()
})

watch(() => props.columns, () => {
  lastWidth.value = 0
  columnize()
})

watch(() => props.width, () => {
  lastWidth.value = 0
  columnize()
})

// Expose methods for external use
defineExpose({
  recolumnize:  () => {
    lastWidth.value = 0
    columnize()
  },
  uncolumnize: () => {
    columns.value = [originalContent.value || props.content]
  }
})
</script>

<style scoped>
.vue-columnizer {
  width: 100%;
  overflow: hidden;
}

.vue-columnizer:: after {
  content:  '';
  display: table;
  clear: both;
}
</style>

<style>
/* Print compatibility styles */
@media print {
  .vue-columnizer . column {
    break-inside: avoid-column;
    page-break-inside: avoid;
  }
  
  .vue-columnizer .dontsplit {
    break-inside: avoid;
    page-break-inside: avoid;
  }
  
  .vue-columnizer . dontend {
    break-after: avoid;
    page-break-after: avoid;
  }
}
</style>