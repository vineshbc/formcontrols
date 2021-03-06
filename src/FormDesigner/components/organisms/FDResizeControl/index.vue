<template>
  <div>
    <div
      :class="
        mainSelected && isEditMode && !isRunMode
          ? 'controlEditStyle'
          : canDragMainDiv
          ? 'controlSelectStyle'
          : 'controlStyle'
      "
      :style="resizeControlStyle"
      :ref="'draRef'.concat(controlId)"
      @mousedown.stop="
        mainSelected && !isRunMode
          ? handleDrag($event)
          : !isRunMode && dragGroupControl($event)
      "
      @contextmenu.stop="displayContextMenu"
    >
      <ResizeHandler
        v-if="!isRunMode"
        ref="resize"
        @updateResize="updateResize"
        :controlId="controlId"
        :userFormId="userFormId"
        controlType="control"
        @createGroup="createGroup"
        @muldragControl="muldragControl"
        @updateIsMove="updateIsMove"
        :size="{
          width: propControlData.properties.Width,
          height: propControlData.properties.Height,
        }"
      />
      <component
        :is="propControlData.type"
        :controlId="propControlData.properties.ID"
        :userFormId="getUserFormId"
        :data="propControlData"
        :isActivated="
          this.selectedControls[this.userFormId].selected.includes(this.controlId) &&
           this.selectedControls[this.userFormId].selected.length === 1
        "
        :isRunMode="isRunMode"
        :isEditMode="isEditMode"
        :containerId="containerId"
        @setEditMode="setEditMode"
        @selectedItem="selectedItem"
        @deleteItem="deleteItem"
        @updateModel="updateModelHandle"
        @updateModelExtraData="updateModelExtraDataHandle"
        @controlEditMode="controlEditMode"
        @updateEditMode="updateEditMode"
      >
        {{ propControlData.properties.Caption }}
      </component>
    </div>
  </div>
</template>

<script lang="ts">
import {
  Component,
  Emit,
  Prop,
  Vue,
  Ref,
  PropSync,
  Watch
} from 'vue-property-decorator'
import { State, Action } from 'vuex-class'

import Label from '@/FormDesigner/components/atoms/FDLabel/index.vue'
import CommandButton from '@/FormDesigner/components/atoms/FDCommandButton/index.vue'
import ToggleButton from '@/FormDesigner/components/atoms/FDToggleButton/index.vue'
import TextBox from '@/FormDesigner/components/atoms/FDTextBox/index.vue'
import SpinButton from '@/FormDesigner/components/atoms/FDSpinButton/index.vue'
import ScrollBar from '@/FormDesigner/components/atoms/FDScrollBar/index.vue'
import OptionButton from '@/FormDesigner/components/atoms/FDOptionButton/index.vue'
import CheckBox from '@/FormDesigner/components/atoms/FDCheckBox/index.vue'
import ComboBox from '@/FormDesigner/components/atoms/FDComboBox/index.vue'
import ListBox from '@/FormDesigner/components/atoms/FDListBox/index.vue'
import FDImage from '@/FormDesigner/components/atoms/FDImage/index.vue'
import Frame from '@/FormDesigner/components/atoms/FDFrame/index.vue'
import TabStrip from '@/FormDesigner/components/atoms/FDTabStrip/index.vue'
import MultiPage from '@/FormDesigner/components/atoms/FDMultiPage/index.vue'
import ResizeHandler from '@/FormDesigner/components/molecules/FDResizeHandler/index.vue'
import FdSelectVue from '@/api/abstract/FormDesigner/FdSelectVue'
import Container from '../FDContainer/index.vue'
import { EventBus } from '@/FormDesigner/event-bus'
@Component({
  name: 'ResizeControl',
  components: {
    ResizeHandler,
    Label,
    CommandButton,
    ToggleButton,
    TextBox,
    SpinButton,
    ScrollBar,
    CheckBox,
    OptionButton,
    FDImage,
    Frame,
    ComboBox,
    ListBox,
    TabStrip,
    MultiPage
  }
})
export default class ResizeControl extends FdSelectVue {
  @PropSync('currentSelectedGroup') public syncCurrentSelectedGroup!: string;
  @Prop({ required: true, type: String }) public containerId!: string;
  @Ref('resize') readonly resize!: ResizeHandler;
  selMultipleCtrl: boolean = false

  handleDrag (event: MouseEvent) {
    if (this.selectedControls[this.userFormId].selected.length > 1 && this.selMultipleCtrl === false) {
      if (event.which !== 3 && this.isMoveWhenMouseDown) {
        this.selectedItem(event)
      }
    }
    this.isMoveWhenMouseDown = false
    this.resize.handleMouseDown(event, 'drag', 'control', this.controlId)
  }
  @Emit('muldragControl')
  private muldragControl (val: IDragResizeGroup) {
    return val
  }

  @Emit('dragControl')
  dragControl (event: MouseEvent) {
    return event
  }
  selectMultipleCtrl (val: boolean) {
    this.selMultipleCtrl = val
  }

  dragGroupControl (event: MouseEvent) {
    if (this.selectedControls[this.userFormId].selected.length > 1 && this.selMultipleCtrl === false) {
      if (event.which !== 3 && this.isMoveWhenMouseDown) {
        this.selectedItem(event)
      }
    }
    this.propControlData.properties.GroupID && this.dragControl(event)
  }
  @Emit('openMenu')
  openMenu (e: MouseEvent, parentID: string, controlID: string) {
    return {
      e,
      parentID,
      controlID
    }
  }

  get canDragMainDiv (): boolean {
    return this.mainSelected && !this.isEditMode && !this.isRunMode
  }

  get propControlData (): controlData {
    return this.userformData[this.userFormId][this.controlId]
  }
  openContextMenu (e: MouseEvent, parentID: string, controlID: string, type: string, mode: boolean) {
    this.$emit('openMenu', e, parentID, controlID, type, mode)
  }
  get resizeControlStyle () {
    const userData = this.userformData[this.userFormId]
    const currentProperties = this.propControlData.properties
    const extraData = this.propControlData.extraDatas!
    const bs = currentProperties.BorderStyle!
    const isRotate = currentProperties.Width! > currentProperties.Height!
    const type = this.propControlData.type
    let highestZIndex = -1
    if (this.selectedControlArray.length === 1 && !this.selectedControlArray[0].startsWith('group')) {
      const selected = userData[this.selectedControlArray[0]].type === 'Page' ? this.selectedContainer[0] : this.selectedControlArray[0]
      if (selected === this.propControlData.properties.ID && (type === 'Frame' || type === 'MultiPage')) {
        const containerControls = [...userData[this.getContainerList(selected)[0]].controls]
        containerControls.sort((a, b) => {
          return userData[b].extraDatas!.zIndex! - userData[a].extraDatas!.zIndex!
        })
        highestZIndex = userData[containerControls[0]].extraDatas!.zIndex!
      }
    }
    return {
      left: `${currentProperties.Left}px`,
      top: `${currentProperties.Top}px`,
      /* border width(5) * 2 = 10 */
      width: `${currentProperties.Width! + 10}px`,
      height: `${currentProperties.Height! + 10}px`,
      display:
        this.isRunMode && currentProperties.Visible === false
          ? 'none'
          : 'block',
      cursor: !this.isRunMode ? 'move' : 'default',
      zIndex: (highestZIndex !== -1 && type !== 'Page' && this.isEditMode) ? highestZIndex + 1 : extraData.zIndex!
    }
  }
  get mainSelected () {
    return (
      this.selectedControls[this.userFormId].selected.includes(
        this.controlId
      ) ||
      (this.selectedControls[this.userFormId].container.includes(
        this.controlId
      ) &&
        this.selectedControls[this.userFormId].selected.includes(
          this.controlId
        ) !==
          this.selectedControls[this.userFormId].container.includes(
            this.controlId
          ))
    )
  }
  deleteItem (event: KeyboardEvent) {
    const userData = this.userformData[this.userFormId]
    const type = userData[this.controlId].type
    const controlId = type === 'Page' ? this.containerId : this.controlId
    const containerId = type === 'Page' ? this.getContainerList(controlId)[0] : this.containerId
    if (event.key !== 'Backspace') {
      this.deleteZIndex(controlId)
      this.deleteTabIndex(controlId)
      this.deleteControl({
        userFormId: this.userFormId,
        parentId: containerId,
        targetId: controlId
      })
    }
    this.selectControl({
      userFormId: this.userFormId,
      select: {
        container: this.getContainerList(containerId),
        selected: [containerId]
      }
    })
  }
  selectedItem (e: MouseEvent) {
    if (this.selMultipleCtrl === false) {
      const userData = this.userformData[this.userFormId]
      const groupId = this.propControlData.properties.GroupID
        ? this.propControlData.properties.GroupID
        : ''
      const currentSelect = this.selectedControls[this.userFormId].selected
      if (currentSelect.length === 1 && currentSelect[0] === this.controlId) {
        if (
          this.isMoveWhenMouseDown !== true &&
        this.propControlData.type !== 'FDImage' &&
        this.propControlData.type !== 'Frame' &&
        this.propControlData.type !== 'MultiPage'
        ) {
          this.isEditMode = true
          this.isMoveWhenMouseDown = false
        }
      } else {
        if (currentSelect.length > 1 && (currentSelect.includes(this.controlId) || currentSelect.includes(this.userformData[this.userFormId][this.controlId].properties.GroupID!))) {
          if (currentSelect.includes(this.controlId)) {
            this.exchangeSelect()
          } else {
            if (
              this.userformData[this.userFormId][this.controlId].properties.GroupID !== ''
            ) {
              const selGrpName = this.userformData[this.userFormId][this.controlId].properties.GroupID!
              this.groupExchange(selGrpName)
            }
          }
        } else {
          let selectTarget = null
          let currentGroup = ''
          if (groupId !== '') {
            if (
              this.syncCurrentSelectedGroup === groupId &&
            currentSelect[0] === groupId
            ) {
              selectTarget = this.controlId
            } else {
              selectTarget = groupId
            }
            currentGroup = groupId
          } else {
            selectTarget = this.controlId
          }

          this.selectControl({
            userFormId: this.userFormId,
            select: { container: this.getContainerList(selectTarget), selected: [selectTarget] }
          })
          this.syncCurrentSelectedGroup = currentGroup
        }
      }
      if ((this.propControlData.type === 'Frame' || this.propControlData.type === 'MultiPage' || this.propControlData.type === 'ListBox') && currentSelect.length === 1) {
        if (this.propControlData.type !== 'ListBox') {
          this.isMoving = true
          this.isEditMode = true
          e.stopPropagation()
        }
      }
    } else {
      const userData = this.userformData[this.userFormId]
      let mainSelected = this.selectedControls[this.userFormId].selected[0]
      const controlData: controlData = userData[this.containerId]
      let divstyle: Array<IGroupStyle> = []
      if (mainSelected.startsWith('group') || userData[this.controlId].properties.GroupID !== '') {
        EventBus.$emit('getGroupSize', (divstayleArray: Array<IGroupStyle>) => {
          divstyle = divstayleArray
        })
      }
      let mainCtrl: controlProperties = { ID: '' }
      if (mainSelected.startsWith('group')) {
        const getIndex = divstyle.findIndex(val => val.groupName === mainSelected)
        mainCtrl = {
          Left: parseInt(divstyle[getIndex].left!),
          Top: parseInt(divstyle[getIndex].top!),
          Width: parseInt(divstyle[getIndex].width!),
          Height: parseInt(divstyle[getIndex].height!),
          ID: ''
        }
      } else {
        mainCtrl = userData[mainSelected].properties
        if (mainSelected === this.containerId) {
          mainSelected = ''
          mainCtrl = userData[this.controlId].properties
        }
      }
      let sideCtrl = userData[this.controlId].properties
      if (userData[this.controlId].properties.GroupID !== '') {
        const getIndex = divstyle.findIndex(val => val.groupName === userData[this.controlId].properties.GroupID)
        sideCtrl = {
          Left: parseInt(divstyle[getIndex].left!),
          Top: parseInt(divstyle[getIndex].top!),
          Width: parseInt(divstyle[getIndex].width!),
          Height: parseInt(divstyle[getIndex].height!),
          ID: ''
        }
      }
      const selectedSize = {
        Left: mainCtrl.Left! < sideCtrl.Left! ? mainCtrl.Left! : sideCtrl.Left!,
        Top: mainCtrl.Top! < sideCtrl.Top! ? mainCtrl.Top! : sideCtrl.Top!,
        Width: (mainCtrl.Left! + mainCtrl.Width!) > (sideCtrl.Left! + sideCtrl.Width!) ? mainCtrl.Left! + mainCtrl.Width! : sideCtrl.Left! + sideCtrl.Width!,
        Height: (mainCtrl.Top! + mainCtrl.Height!) > (sideCtrl.Top! + sideCtrl.Height!) ? mainCtrl.Top! + mainCtrl.Height! : sideCtrl.Top! + sideCtrl.Height!
      }
      const left = selectedSize.Left
      const top = selectedSize.Top
      const right = selectedSize.Width
      const bottom = selectedSize.Height
      const multipleCtrl = []
      if (left !== right || top !== bottom) {
        for (let i in controlData.controls) {
          const key: string = controlData.controls[i]
          const controlProp: controlProperties = this.userformData[this.userFormId][key].properties
          if (
            left <= controlProp!.Left! + controlProp!.Width! &&
            right >= controlProp!.Left! &&
            top <= controlProp!.Top! + controlProp!.Height! &&
            bottom >= controlProp!.Top!
          ) {
            multipleCtrl.push(key)
          }
        }
        const selectedGroup: string[] = []
        for (const val of multipleCtrl) {
          const controlGroupId: string = this.userformData[this.userFormId][val]
            .properties.GroupID!
          if (controlGroupId && controlGroupId !== '') {
            !selectedGroup.includes(controlGroupId)! &&
              selectedGroup.push(controlGroupId)
          } else {
            selectedGroup.push(val)
          }
        }
        if (this.selectedControlArray.length !== 0) {
          this.selectControl({
            userFormId: this.userFormId,
            select: {
              container: this.getContainerList(selectedGroup[0]),
              selected: [...selectedGroup]
            }
          })
        }
      }
    }
  }

  exchangeSelect () {
    const sel = [...this.selectedControls[this.userFormId].selected]
    const controlIndex = this.selectedControls[this.userFormId].selected.findIndex((val) => val === this.controlId)
    sel.splice(controlIndex, 1)
    sel.unshift(this.controlId)
    this.selectControl({
      userFormId: this.userFormId,
      select: { container: this.getContainerList(sel[0]), selected: [...sel] }
    })
  }
  groupExchange (groupName: string) {
    const sel = [...this.selectedControls[this.userFormId].selected]
    const controlIndex = this.selectedControls[this.userFormId].selected.findIndex((val) => val === groupName)
    sel.splice(controlIndex, 1)
    sel.unshift(groupName)
    this.selectControl({
      userFormId: this.userFormId,
      select: { container: this.getContainerList(sel[0]), selected: [...sel] }
    })
  }
  get getModeStyle () {
    return this.mainSelected ? 'controlSelectStyle' : 'controlStyle'
  }

  @Watch('selectedControls', { deep: true })
  updateSelectedControls () {
    const type = this.userformData[this.userFormId][this.selectedContainer[0]].type
    const controlType = this.userformData[this.userFormId][this.controlId].type
    if (type === 'Frame' || type === 'Page' || type === 'MultiPage') {
      if (controlType === 'Frame' || controlType === 'MultiPage') {
        this.isEditMode = true
      } else {
        if (this.isMoving === false) {
          this.isEditMode = false
        } else {
          this.isMoving = false
        }
      }
    } else {
      if (this.isMoving === false) {
        this.isEditMode = false
      } else {
        this.isMoving = false
      }
    }
  }

  get getUserFormId () {
    if (
      this.propControlData.type === 'Frame' ||
      this.propControlData.type === 'MultiPage' ||
      this.propControlData.type === 'TabStrip'
    ) {
      return this.userFormId
    } else {
      return null
    }
  }
  created () {
    EventBus.$on('actMultipleCtrl', (val: boolean) => {
      this.selMultipleCtrl = val
    })
  }
  displayContextMenu (event: MouseEvent) {
    this.openContextMenu(event, this.containerId, this.controlId, 'control', this.isEditMode)
  }
}
</script>

<style scoped>
.controlSelectStyle {
  box-sizing: border-box;
  position: absolute;
  --border-width: 5;
  --stripe-distance: 2px;
  border: calc(var(--border-width) * 1px) solid transparent;
  border-image: repeating-linear-gradient(
      -110deg,
      black,
      transparent 1px,
      transparent var(--stripe-distance),
      black calc(var(--stripe-distance) + 0.2px)
    )
    var(--border-width);
}
.controlEditStyle {
  box-sizing: border-box;
  position: absolute;
  --border-width: 5;
  --stripe-distance: 2px;
  border: calc(var(--border-width) * 1px) solid transparent;
  border-image: repeating-linear-gradient(
      -45deg,
      black,
      transparent 2px,
      transparent var(--stripe-distance),
      black calc(var(--stripe-distance) + 0.2px)
    )
    var(--border-width);
}
.controlStyle {
  box-sizing: border-box;
  position: absolute;
  cursor: default !important;
}
:focus {
  outline: none;
}
</style>
