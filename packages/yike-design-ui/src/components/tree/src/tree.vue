<template>
  <div :class="bem()">
    <Node v-for="option in options" :key="option.key" :option="option" />
  </div>
</template>
<script setup lang="ts">
import { TreeInjectionKey, TreeOption, TreeProps } from './tree'
import { createCssScope } from '../../utils/bem'
import Node from './node.vue'
import { Key } from '../../utils'
import { _TreeNode, tree2list } from './internal'
import { watch, ref, onMounted, provide, shallowRef } from 'vue'
import { IconRightFill } from '../../svg-icon'
import { h } from 'vue'
import { computed } from 'vue'

const bem = createCssScope('tree')

defineOptions({
  name: 'YkTree',
})

const props = withDefaults(defineProps<TreeProps>(), {
  blockNode: false,
  multiple: false,
  expandIcon() {
    return () => h(IconRightFill)
  },
  defaultExpandedKeys() {
    return []
  },
  defaultSelectedKeys() {
    return []
  },
  fileTree: false,
})

const emits = defineEmits<{
  expand: [keys: Key[]]
  select: [keys: Key[]]
}>()

// 获取节点的 map 结构
const nodeMaps = shallowRef(tree2list(props.options))
watch(
  () => props.options,
  () => {
    nodeMaps.value = tree2list(props.options)
  },
)

// 处理节点的展开收缩逻辑
const expandedKeys = defineModel<Key[]>('expandedKeys', {
  local: true,
  default: [],
})
const handleExpandPKey = (key: Key) => {
  onExpand(key, false, true)
  const pKey = nodeMaps.value.get(key)?.pKey
  if (!pKey) return
  handleExpandPKey(pKey)
}
onMounted(() => {
  if (props.defaultExpandedKeys.length === 0) {
    return
  }
  for (const key of props.defaultExpandedKeys) {
    handleExpandPKey(key)
  }
})
const onExpand = (key: Key, close = false, first = false) => {
  const s = new Set([...expandedKeys.value])
  if (close) {
    s.delete(key)
  } else {
    s.add(key)
  }
  expandedKeys.value = [...s]
  if (first) return
  emits('expand', expandedKeys.value)
}

// 处理节点选中的逻辑
const selectedKeys = defineModel<Key[]>('selectedKeys', {
  local: true,
  default: [],
})
onMounted(() => {
  if (props.defaultSelectedKeys.length === 0) {
    return
  }
  selectedKeys.value = props.defaultSelectedKeys
  for (const key of props.defaultSelectedKeys) {
    const pKey = nodeMaps.value.get(key)?.pKey
    if (pKey) handleExpandPKey(pKey)
  }
})
const onSelect = (key: Key) => {
  if (props.multiple) {
    selectedKeys.value = [...new Set([...selectedKeys.value, key])]
  } else {
    selectedKeys.value = [key]
  }
  emits('select', selectedKeys.value)
}

provide(TreeInjectionKey, {
  blockNode: props.blockNode,
  expandedKeys: expandedKeys,
  selectedKeys: selectedKeys,
  fileTree: props.fileTree,
  fileIcons: props.fileIcons,
  expandIcon: props.expandIcon,
  onSelect,
  onExpand,
})
</script>
