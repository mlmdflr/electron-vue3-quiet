<template>
  <IpcOnMounted />
  <div class="print column">
    <div class="row">
      <span class="tips">建议使用打印PDF测试（省纸），成功后再使用真实情况</span>
    </div>
    <div class="row">
      <select v-model="selName" class="grip-right">
        <option v-for="(item, index) in printers" :key="index" :value="item.name">
          {{ item.displayName }}
        </option>
      </select>
      <button type="button" @click="print">打印</button>
    </div>
    <div class="row">
      <button class="grip-right" type="button" @click="() => (silent = !silent)">
        {{ silent ? '' : '非' }}静默打印
      </button>
      <button class="grip-right" type="button" @click="() => (printBackground = !printBackground)">
        {{ printBackground ? '使用' : '不使用' }}背景色
      </button>
      <button type="button" @click="() => (color = !color)">
        {{ color ? '彩色' : '黑白' }}
      </button>
    </div>
    <div class="line" />
    <div class="row">
      <span class="grip-right">边距:</span>
      <select v-model="margins.marginType" class="grip-right">
        <option v-for="(item, index) in marginTypes" :key="index" :value="item">
          {{ item }}
        </option>
      </select>
    </div>
    <div v-show="margins.marginType === 'custom'" class="row">
      <label class="grip-right">上边距: <input v-model.number="margins.top" class="small" />px</label>
      <label class="grip-right">下边距: <input v-model.number="margins.bottom" class="small" />px</label>
      <label class="grip-right">左边距: <input v-model.number="margins.left" class="small" />px</label>
      <label class="grip-right">右边距: <input v-model.number="margins.right" class="small" />px</label>
    </div>
    <div class="line" />
    <div class="row">
      <span class="grip-right">尺寸:</span>
      <!-- <div class="fake-radio"  @click="() => selPageSizeType = 0">{{ selPageSizeType === 0 ? '√' : '' }}</div> -->
      <select v-model="pageSizeString" class="grip-right">
        <option v-for="(item, index) in pageSizeOptions" :key="index" :value="item">
          {{ item }}
        </option>
      </select>
    </div>
    <!-- 设置宽高变成A4 大概哪里不对 -->
    <!-- <div class="row">
      <span class="grip-right">&emsp;&emsp;&nbsp;</span>
      <div class="fake-radio" @click="() => selPageSizeType = 1">{{ selPageSizeType === 1 ? '√' : '' }}</div>
      <label class="grip-right">宽度: <input class="normal" v-model.number="pageSizeObject.width" />μm</label>
      <label class="grip-right">高度: <input class="normal" v-model.number="pageSizeObject.height" />μm</label>
      <span class="tips">1mm = 1000μm</span>
    </div> -->
    <div class="line" />
    <div class="row">
      <div style="width: 100px; height: 100px; background: red" class="grip-right" />
      <img
        style="width: 100px; height: 100px; object-fit: cover"
        src="https://img2.baidu.com/it/u=2173864545,554093748&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=882"
      />
    </div>
  </div>
</template>
<script lang="ts" setup>
import IpcOnMounted from '@/components/IpcOnMounted'

const selName = ref('')
const printers = ref<Electron.PrinterInfo[]>([])
const silent = ref(false)
const printBackground = ref(false)
const color = ref(true)
const marginTypes = ref(['default', 'none', 'printableArea', 'custom'])
const margins = ref({
  marginType: 'default',
  top: 0,
  bottom: 0,
  left: 0,
  right: 0
})
const pageSizeString = ref<string>('A4')
const pageSizeOptions = ref(['A3', 'A4', 'A5', 'Legal', 'Letter', 'Tabloid'])
const pageSizeObject = ref({ width: 210000, height: 297000 })
const selPageSizeType = ref(0) // 0 string  1 Size
onMounted(async () => {
  // 获取打印机列表
  printers.value = await window.$ipc.invoke('print-option', 'get-printers')
  if (printers.value.length) {
    const defaultItem = printers.value.find(v => v.isDefault)
    if (defaultItem) {
      selName.value = defaultItem.name
    } else {
      selName.value = printers.value[0].name
    }
  }
})

async function print() {
  if (selName.value) {
    const printRes = await window.$ipc.invoke('print-option', 'print', {
      silent: silent.value,
      deviceName: selName.value,
      printBackground: printBackground.value,
      color: color.value,
      margins: toRaw(margins.value),
      pageSize: selPageSizeType.value === 0 ? toRaw(pageSizeString.value) : toRaw(pageSizeObject.value)
    })
    console.info(printRes)
  }
}
</script>
<style scoped>
.print {
  overflow: auto;
  user-select: none;
}

.tips {
  color: red;
}

.print {
  height: 100vh;
  padding: 10px;
  border: 1px solid blue;
}

select {
  height: 40px;
  border: 1px solid #000;
  border-radius: 4px;
}

button {
  height: 40px;
  padding: 0 20px;
  border: 1px solid #000;
  border-radius: 4px;
}

input {
  height: 40px;
  padding: 0 10px;
  border: 1px solid #000;
  border-radius: 4px;
}

input.small {
  width: 50px;
}

input.normal {
  width: 100px;
}

.row {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.column {
  display: flex;
  flex-direction: column;
}

.grip-right {
  margin-right: 10px;
}

.fake-radio {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  font-size: 20px;
  border: 1px solid #000;
  border-radius: 4px;
  margin-right: 10px;
}

.line {
  min-height: 1px;
  background: rgba(0, 0, 0, 0.1);
  margin-bottom: 10px;
}
</style>
