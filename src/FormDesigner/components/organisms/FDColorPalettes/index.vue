<template>
  <div>
    <div class="buttonIn grid-design">
      <input
        type="text"
        :name="name"
        class="color-input"
        :value="value"
        disabled= "true"
      />
      <button
        id="clear"
        :style="colorPalettesStyleObj"
        @mousedown.prevent="openColorPalette"
      >
        _
      </button>
    </div>
    <div class="grid-design1">
      <div></div>
      <div class="outer-border">
        <div
          class="tabs"
          ref="colorPalleteRef"
          tabindex="1"
          v-if="isVisible"
          @blur="hideColorPallete"
        >
          <div class="tab">
            <input
              class="tabinput"
              type="radio"
              id="color-1"
              name="tab-color-1"
              checked
            />
            <label for="color-1">Palette</label>
            <div class="content">
              <div class="grid-container">
                <div
                  class="grid-item1"
                  v-for="(color, i) in colors"
                  :key="i"
                  @click="selectColor(color)"
                  :style="{ backgroundColor: color.value }"
                ></div>
              </div>
            </div>
          </div>
          <div class="tab">
            <input
              class="tabinput"
              type="radio"
              id="color-2"
              name="tab-color-1"
            />
            <label for="color-2">System</label>
            <div class="content">
              <div class="gridcontainer">
                <div
                  class="gridcontainerStyle"
                  v-for="(color, i) in colorSystem"
                  :key="i"
                  @click="selectColor(color)"
                >
                  <span
                    class="griditem1"
                    :style="{ backgroundColor: color.value }"
                  ></span
                  >{{ color.displayName }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { colors } from '../../../models/colors'
import { colorSystem } from '../../../models/colorSystem'
import { Component, Prop, Vue, Emit, Ref } from 'vue-property-decorator'
interface IColors {
  name: string;
  value: string;
}
interface ISelectedValue {
  name: string;
  value: string;
  displayName: string;
}
@Component({
  name: 'FDColorPalettes'
})
export default class FDColorPalettes extends Vue {
  @Prop() value: string;
  @Prop() name: string;
  @Ref('colorPalleteRef') readonly colorPalleteRef!: HTMLDivElement;
  private isVisible = false;
  colors: Array<IColors> = colors;
  colorSystem: Array<ISelectedValue> = colorSystem;
  selectedValue: ISelectedValue

  openColorPalette () {
    if (this.isVisible === true) {
      this.isVisible = false
    } else if (this.isVisible === false) {
      this.isVisible = true
    }
    if (this.isVisible) {
      this.$nextTick(function () {
        this.colorPalleteRef.focus()
      })
    }
  }

  selectColor (data: ISelectedValue) {
    this.selectedValue = data
    this.emitColorPalette(this.selectedValue.value, this.name)
    this.isVisible = !this.isVisible
  }

  hideColorPallete () {
    this.isVisible = false
  }

  @Emit('colorPalette')
  emitColorPalette (selectedValue: string, name: string) {
    return { propertyName: name, propertyValue: selectedValue }
  }
  get colorPalettesStyleObj () {
    return {
      height: '17px',
      width: '17px',
      padding: '0px',
      paddingLeft: '1px',
      paddingRight: '1px',
      border: '1px solid white',
      borderRightColor: 'gray',
      borderBottomColor: 'gray',
      borderLeftColor: 'lightgray',
      borderTopColor: 'lightgray',
      outline: 'none',
      marginLeft: '-2px'
    }
  }
}
</script>

<style scoped>
.grid-design {
  overflow: hidden;
  float: right;
  width: 100%;
  height: 17px;
}
grid-design1 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  background-color: rgb(238, 238, 238);
}
.color-input {
  border: none;
  outline: none;
  height: 14px;
  width: calc(100% - 20px);
}
.tabs {
  position: absolute;
  min-height: 174px;
  width: 204px;
  top: 18px;
  border: 1px solid gray;
  background-color: rgb(240, 240, 240);
  clear: both;
  align-self: right;
  z-index: 400;
  right: 0px;
  margin: 0px 0px;
  float: right;
  margin-right: 0px;
  outline: none;
}
.tab {
  float: left;
}
.tab label {
  background: rgb(238, 238, 238);
  height: 12px;
  border: 1px solid lightgray;
  margin-left: -1px;
  position: relative;
  left: 1px;
  padding: 3px;
}
.tab [type="radio"] {
  display: none;
}
.content {
  position: absolute;
  top: 18px;
  left: 0;
  background: rgb(238, 238, 238);
  width: 200px;
  height: 152px;
  border: 1px solid rgb(238, 238, 238);
  box-shadow: 1px 1px gray;
  margin-top: 1px;
}
[type="radio"]:checked ~ label {
  background: white;
  border: 2px solid lightgray;
  z-index: 2;
}
[type="radio"]:unchecked ~ label {
  border: 0.5px solid white;
}
[type="radio"]:checked ~ label ~ .content {
  z-index: 1;
}
.grid-container {
  display: grid;
  grid-template-columns: auto auto auto auto auto auto auto auto;
  width: 10px;
  padding: 10px;
}
.grid-item1 {
  border: 1px solid rgba(0, 0, 0, 0.8);
  width: 14px;
  height: 14px;
}

.gridcontainer {
  height: 150px;
  border: 1px solid gray;
  overflow-y: scroll;
  font-size: 12px;
}
.griditem1 {
  border: 1px solid rgba(0, 0, 0, 0.8);
  width: 14px;
  height: 14px;
  text-align: center;
  margin: 1px;
  display: inline-block;
  cursor: none;
}
.gridcontainerStyle {
  cursor: "context-menu";
}
</style>
