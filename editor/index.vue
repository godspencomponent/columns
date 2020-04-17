<template>
  <div class="componentEditor">
    <el-card class="box-card">
      <div slot="header">
        分栏 <small style="color: #999">基于12栏布局</small>
        <el-checkbox style="float:right" :value="!freeMode" @change="freeMode = !$event">{{ freeMode ? '已禁用' : '已启用' }}</el-checkbox>
      </div>
      <div class="columns-star-inserted">
        <el-row :gutter="12">
          <el-col :span="6" v-for="(col, i) in cols" :key="i">
            <el-row @click.native="pickCols(col)" :class="{active: isActive(col)}" class="el-row-preset" :gutter="3">
              <el-col v-for="(flex, i) in col" :key="i" :span="flex"><div class="grid-content"></div></el-col>
            </el-row>
          </el-col>
          <el-col :span="6">
            <el-row class="active el-row-preset" :gutter="3">
              <el-col :span="24"><div class="grid-content grid-content-input" @click="showCustomInput = true">输入</div></el-col>
            </el-row>
          </el-col>
        </el-row>
      </div>
      <el-input ref='input' v-show='showCustomInput' size='mini' :value="inputRaw" type="text" @keyup.enter.native="useCustom" @input="onInput" placeholder="分栏比例，总和12，如: 4+4+4+4">
        <el-button slot="append" icon="el-icon-plus" @click="useCustom"></el-button>
      </el-input>
      <el-row :gutter="1" style="margin-top: 10px;" class="cols-result">
        <el-col class="cols-result-item" @click.native="toogleOffsetOrNormal(i, col)" v-for="(col,i) in pickedCol" :title="isNaN(col) ? '留空' : ''" :class="{'empty-col': isNaN(col), ha: isNaN(col)}" :key="i+'_'+col" :span="parseInt(col)"><div class="grid-content">{{parseInt(col) / 2}}</div></el-col>
      </el-row>
    </el-card>
    <el-card class="box-card">
      <div slot="header">
        自由分配 <small style="color: #999">根据比例自由分配空间</small>
        <el-checkbox style="float:right" :value="freeMode" @change="freeMode = $event">{{ !freeMode ? '已禁用' : '已启用' }}</el-checkbox>
      </div>
      <div class="columns-star-inserted">
        <el-row :gutter="12">
          <el-col :span="6" v-for="(col, i) in opts" :key="i">
            <div @click="pickFreeCols(col)" :class="{active: isFreeActive(col)}" class="flex-preset">
              <div class="flex-preset-item" v-for="(flex, i) in col" :key="i" :style="{flex: flex}"><div class="grid-content"></div></div>
            </div>
          </el-col>
          <el-col :span="6">
            <div class="active flex-preset" :gutter="3">
              <div class="flex-preset-item" :style="{flex: 1}"><div class="grid-content grid-content-input" @click="showFreeCustomInput = true">输入</div></div>
            </div>
          </el-col>
        </el-row>
      </div>
      <el-input ref='input' v-show='showFreeCustomInput' size='mini' :value="inputFreeRaw" type="text" @keyup.enter.native="useFreeCustom" @input="onFreeInput" placeholder="分配比例，每一项为正整数，如: 1+2+3">
        <el-button slot="append" icon="el-icon-plus" @click="useFreeCustom"></el-button>
      </el-input>
      <el-row :gutter="1" style="margin-top: 10px;" class="cols-result free-mode">
        <div class="cols-result-item" @click="toogleFreeOffsetOrNormal(i, col)" v-for="(col,i) in pickedFreeCol" :title="isNaN(col) ? '留空' : ''" :class="{'empty-col': isNaN(col), ha: isNaN(col)}" :key="i+'_'+col" :style="{flex: parseFloat(col) || 0}">
          <div class="grid-content">{{parseFloat(col)}}</div>
        </div>
      </el-row>
    </el-card>
  </div>
</template>

<script>
export default {
  name: "maliangeditor",
  props: {
    componentInfo: Object
  },
  data: function() {
    return {
      freeMode: true,
      cols: [
        [24],
        [12,12],
        [8, 16],
        [16, 8],
        [8, 8, 8],
        [6, 6, 6, 6],
        [4, 4, 4, 4, 4, 4]
      ],
      opts: [
        Array.apply(null, {length: 1}).map(v => 1),
        Array.apply(null, {length: 2}).map(v => 1),
        Array.apply(null, {length: 3}).map(v => 1),
        Array.apply(null, {length: 4}).map(v => 1),
        Array.apply(null, {length: 5}).map(v => 1),
        Array.apply(null, {length: 6}).map(v => 1),
        Array.apply(null, {length: 8}).map(v => 1),
      ],
      inputRaw: '',
      showCustomInput: false,
      inputFreeRaw: '',
      showFreeCustomInput: false,
    };
  },
  watch: {
    freeMode (val) {
      this.componentInfo.freeMode = val
    }
  },
  computed: {
    pickedCol () {
      return this.componentInfo && this.componentInfo.layout
    },
    pickedFreeCol () {
      return this.componentInfo && this.componentInfo.freeLayout
    },
  },
  mounted: function() {
    if (this.componentInfo && !(this.componentInfo.layout instanceof Array)) this.$set(this.componentInfo, 'layout', [24])
    if (this.componentInfo && !(this.componentInfo.freeLayout instanceof Array)) this.$set(this.componentInfo, 'freeLayout', [1])
    if (this.componentInfo && (this.componentInfo.freeMode === undefined || this.componentInfo.freeMode === null)) this.$set(this.componentInfo, 'freeMode', false)
    this.freeMode = this.componentInfo && this.componentInfo.freeMode
  },
  methods: {
    isActive (col) {
      return String(col) === String(this.pickedCol).replace(/o/g, '')
    },
    isFreeActive (col) {
      return String(col) === String(this.pickedFreeCol).replace(/o/g, '')
    },
    onInput (str) {
      this.inputRaw = '✾'
      this.$nextTick(() => {
        this.inputRaw = str.replace(/[^\d+]/g, '')
      })
    },
    onFreeInput (str) {
      this.inputFreeRaw = '✾'
      this.$nextTick(() => {
        this.inputFreeRaw = str.replace(/[^\d+]/g, '')
      })
    },
    pickCols (col) {
      this.componentInfo.layout = Array.from(col)
    },
    pickFreeCols (col) {
      this.componentInfo.freeLayout = Array.from(col)
    },
    useCustom () {
      const cols = this.inputRaw.split('+')
      const sum = cols.reduce((o, a) => (o += +a), 0)
      if (sum !== 12) return this.$alert('各项相加的和不为12，请修正')
      this.pickCols(cols.map(v => v * 2))
      this.showCustomInput = false
    },
    useFreeCustom () {
      const cols = this.inputFreeRaw.split('+')
      this.pickFreeCols(cols)
      this.showFreeCustomInput = false
    },
    toogleOffsetOrNormal (i, val) {
      if (this.pickedCol) {
        this.pickedCol.splice(i, 1, !/o$/.test(val) ? `${val}o` : parseInt(val))
      }
    },
    toogleFreeOffsetOrNormal (i, val) {
      if (this.pickedFreeCol) {
        this.pickedFreeCol.splice(i, 1, !/o$/.test(val) ? `${val}o` : parseInt(val))
      }
    }
  }
};
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus" scoped>
.componentEditor {
}
.grid-content {
  background: #faad16;
  border-radius: 2px;
  min-height: 20px;
  opacity 0.2;
}
.el-row-preset {
  margin-bottom: 10px;
  cursor pointer
  &.active .grid-content {
    opacity 1
  }
  .grid-content-input {
    font-size: 12px;
    text-align: center;
    color #eee;
    line-height: 20px;
  }
}
.flex-preset {
  display: flex;
  margin-bottom: 10px;
  cursor pointer
  &.active .grid-content {
    opacity 1
  }
  .grid-content-input {
    font-size: 12px;
    text-align: center;
    color #eee;
    line-height: 20px;
  }
  .flex-preset-item {
    margin-right: 3px;
  }
  .flex-preset-item:last-child {
    margin-right 0
  }
}
.cols-result {
  .grid-content {
    background-color transparent
    outline: solid #faad16 1px 
    text-align: center;
    font-size: 12px;
    line-height: 20px;
    opacity: 1;
  }
  &.free-mode {
    display flex
  }
}
.empty-col {
  background-image url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAAXNSR0IB2cksfwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAGBQTFRF2dnZzc3Nzs7OysrKycnJ2tra8vLy/////v7++vr67u7u09PTx8fHzMzMxsbGz8/P9vb2/Pz80dHR/f399fX1+/v7+Pj4y8vL0NDQ0tLSyMjI+fn59/f31tbW2NjY8/PzHh+WPAAAALpJREFUeJzt1LsSgjAQheGzENkoCRgRA3h7/7c0OKa02K23/FJkMjkzP0ANtY4OHTN7fzyJ3QcXYxzGc3E5S1KXCyNRTxc/8ZSu801qOBpzaENkn3h/ldQIQ3C0EPGXzFJjXSKtlLd7Yv+YE0uNZqPcDDkmnriceanxDDm6Nr/ePO2/2kkN9f4/Q7t/NbT7V0O7fzW0+1dDu381tPtXQ7t/NawH1gPrgfXAemA9sB5YD6wH1gPrgfXgjz/wKK8hjpBrgwAAAABJRU5ErkJggg==')
  background-size 40px auto
}
.cols-result-item {
  cursor pointer
}
</style>
