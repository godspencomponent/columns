<template>
  <div class="component columns" :class="{'free-mode': freeMode}">
    <div :key="i" class="columns-item" :style='genstyle(s)' :class="genClass(s)" v-for="(s,i) in normalizeLayout">
      <template v-if='!(freeMode && s.offset > 0)'>
        <slot :name="'slot'+ s.i" ><div v-show="inEditor" class="slot-placeholder"></div></slot>
      </template>
    </div>
  </div>
</template>

<script>
  import {VueExtend} from 'godspen-lib'

  export default {
    mixins: [VueExtend.mixin],
    name: 'columns',
    label: process.env.LABEL,
    style: process.env.STYLE,
    props: {
      layout: {
        type: Array,
        editor: {
          ignore: true
        },
        default () {
          return [24]
        }
      },
      freeLayout: {
        type: Array,
        editor: {
          ignore: true
        },
        default () {
          return [1]
        }
      },
      freeMode: {
        type: Boolean,
        editor: {
          ignore: true
        },
        default: false
      },
      fill: {
        type: Boolean,
        editor: {
          type: 'boolean',
          label: '填充',
          desc: '强制设定子组件的宽高为100%'
        },
        default: true
      }
    },
    data () {
      return {
        inEditor: /^edit[oe]r$/i.test(window.EDIT_TYPE)
      }
    },
    slots: (props = {}) => {
      if (!props.fill) return true
      return {style: { width: '100%', height: '100%' }}
    },
    computed: {
      normalizeLayout () {
        if (!this.freeMode) {
          return (this.layout || []).reduceRight((o, i) => {
            if (isNaN(i)) {
              if (o[0] && typeof o[0] === 'object') o[0].offset = parseInt(i) + (o[0].offset || 0)
            } else o.unshift({col: i})
            return o
          }, []).map((v, i) => {
            v.i = i
            return v
          })
        } else {
          let i = 0
          return (this.freeLayout || []).map((o) => {
            return isNaN(o) ? {offset: parseInt(o)} : {col: o, i: i++}
          }, [])
        }
      }
    },
    editorMethods: {
    },
    methods: {
      genClass (col) {
        let classList = []
        if (this.freeMode) classList.push('free-mode')
        else {
          classList.push(`ml-col-${col.col}`)
          if (col.offset > 0) classList.push(`ml-col-offset-${col.offset}`)
        }
        return classList
      },
      genstyle (col) {
        if (!this.freeMode) return {}
        return {
          flex: col.col || col.offset,
        }
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus" scoped>
.component.columns{
  width: 100%
  height: 100%
  &.free-mode{
    display flex
  }
}
[class*="ml-col-"] {
  float: left;
  box-sizing: border-box;
}

.ml-col-0 {
  display: none;
}

for i in range(0, 24, 1) {
  .ml-col-{i} {
    width: (1 / 24 * i * 100) * 1%;
  }
  .ml-col-offset-{i} {
    margin-left: (1 / 24 * i * 100) * 1%;
  }
}
.slot-placeholder {
  border: solid 1px #333
  box-sizing border-box
  min-height 100px
}
.component.columns::after {
  content: ''
  clear both
  display table
}
.empty-col {
  background-image url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAAXNSR0IB2cksfwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAGBQTFRF2dnZzc3Nzs7OysrKycnJ2tra8vLy/////v7++vr67u7u09PTx8fHzMzMxsbGz8/P9vb2/Pz80dHR/f399fX1+/v7+Pj4y8vL0NDQ0tLSyMjI+fn59/f31tbW2NjY8/PzHh+WPAAAALpJREFUeJzt1LsSgjAQheGzENkoCRgRA3h7/7c0OKa02K23/FJkMjkzP0ANtY4OHTN7fzyJ3QcXYxzGc3E5S1KXCyNRTxc/8ZSu801qOBpzaENkn3h/ldQIQ3C0EPGXzFJjXSKtlLd7Yv+YE0uNZqPcDDkmnriceanxDDm6Nr/ePO2/2kkN9f4/Q7t/NbT7V0O7fzW0+1dDu381tPtXQ7t/NawH1gPrgfXAemA9sB5YD6wH1gPrgfXgjz/wKK8hjpBrgwAAAABJRU5ErkJggg==')
  background-size 20px auto
}
.columns-item {
  height: 100%;
}
</style>
