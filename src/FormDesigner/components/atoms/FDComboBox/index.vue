<template>
  <div
    class="custom-select"
    :tabindex="tabindex"
    :style="customSelectObj"
    :title="properties.ControlTipText"
    @click="selectedItem"
    @keydown.enter="setContentEditable($event, true)"
    @keydown.esc="releaseEditMode"
    v-on="eventStoppers()"
  >
    <div
      class="combobox"
      :style="boxStyleObj"
      @mousedown="controlEditMode"
      tabindex="1"
      @click="toFocus"
    >
      <div
        :class="properties.SelectionMargin ? 'selectionDiv' : ''"
        :style="selectionStyle"
      >
        <span
          v-if="properties.SelectionMargin"
          class="selectionSpan"
          :style="selectionSpanObj"
          @click="setSelection"
        ></span>
        <textarea
          data-gramm="false"
          ref="textareaRef"
          :style="cssStyleProperty"
          wrap="off"
          :tabindex="properties.TabIndex"
          :readonly="getDisableValue || properties.Style === 1"
          :maxlength="properties.MaxLength !== 0 ? properties.MaxLength : ''"
          @blur="handleBlur($event, textareaRef, hideSelectionDiv)"
          @click="handleClick($event, textareaRef, hideSelectionDiv)"
          @input="handleTextInput($event)"
          class="text-box-design"
          :value="properties.Value"
          @dragstart="dragBehavior"
          @keydown.enter.prevent
        />
        <div
          ref="hideSelectionDiv"
          @click="divHide($event, textareaRef)"
          :style="divcssStyleProperty"
          :title="properties.ControlTipText"
          class="text-box-design"
        >
          {{ properties.Value }}
        </div>
        <label
          ref="autoSizeTextarea"
          class="labelStyle"
          :class="labelStyleObj"
        ></label>
      </div>
      <div
        class="selected"
        @click="enabledCheck($event)"
        :style="selectedStyleObj"
      >
        <div v-if="properties.DropButtonStyle === 1">
          <svg
            version="1.0"
            xmlns="http://www.w3.org/2000/svg"
            height="10"
            width="10"
            viewBox="0 0 810.000000 460.000000"
            preserveAspectRatio="xMidYMid meet"
          >
            <g
              :fill="
                properties.Enabled
                  ? '#000000'
                  : open
                  ? '#000000'
                  : 'rgb(190,190,190)'
              "
              transform="translate(0.000000,460.000000) scale(0.100000,-0.100000)"
              stroke="none"
            >
              <path
                d="M74 4585 c-4 -8 0 -23 7 -32 8 -9 897 -1036 1977 -2282 1612 -1862
          1967 -2266 1987 -2266 20 0 378 408 2005 2280 1089 1254 1979 2287 1977 2295
          -2 13 -453 15 -3975 18 -3540 2 -3973 0 -3978 -13z"
              />
            </g>
          </svg>
        </div>
        <div v-else-if="properties.DropButtonStyle === 2">
          <svg
            version="1.0"
            xmlns="http://www.w3.org/2000/svg"
            width="12"
            height="12"
            viewBox="0 0 1280.000000 1024.000000"
            preserveAspectRatio="xMidYMid meet"
          >
            <g
              transform="translate(0.000000,1024.000000) scale(0.100000,-0.100000)"
              :fill="
                properties.Enabled
                  ? '#000000'
                  : open
                  ? '#000000'
                  : 'rgb(180,180,180)'
              "
              stroke="none"
            >
              <path
                d="M3105 4084 c-295 -66 -529 -300 -590 -592 -19 -86 -19 -238 0 -324
32 -153 97 -270 217 -390 73 -73 101 -94 190 -137 58 -29 134 -58 173 -66 103
-22 275 -19 368 5 218 55 400 204 497 405 59 123 80 212 80 348 0 347 -231
651 -565 742 -87 24 -283 29 -370 9z"
              />
              <path
                d="M6236 4083 c-151 -32 -275 -101 -386 -213 -151 -153 -219 -324 -220
-546 0 -160 45 -304 138 -441 40 -58 130 -150 188 -190 59 -42 175 -94 251
-113 104 -27 281 -27 386 -1 259 67 466 267 549 531 20 65 23 95 23 225 -1
137 -3 157 -28 230 -78 223 -238 396 -444 480 -132 53 -316 69 -457 38z"
              />
              <path
                d="M9370 4085 c-364 -81 -610 -385 -610 -752 0 -136 21 -225 80 -348 97
-201 279 -350 497 -405 93 -24 265 -27 368 -5 39 8 115 37 173 66 89 43 117
64 190 137 121 121 185 237 219 395 18 89 13 275 -11 362 -71 255 -285 469
-541 540 -84 23 -282 29 -365 10z"
              />
            </g>
          </svg>
        </div>
        <div v-else-if="properties.DropButtonStyle === 3">
          <svg
            version="1.0"
            xmlns="http://www.w3.org/2000/svg"
            width="12"
            height="12"
            viewBox="0 0 1200.000000 1200.000000"
            preserveAspectRatio="xMidYMid meet"
          >
            <g
              transform="translate(0.000000,1200.000000) scale(0.100000,-0.100000)"
              :fill="
                properties.Enabled
                  ? '#000000'
                  : open
                  ? '#000000'
                  : 'rgb(180,180,180)'
              "
              stroke="none"
            >
              <path
                d="M0 6000 l0 -460 6000 0 6000 0 0 460 0 460 -6000 0 -6000 0 0 -460z"
              />
            </g>
          </svg>
        </div>
      </div>
      <div class="items" :class="{ selectHide: !open }" :style="itemsStyleObj">
        <div
          class="listStyle"
          :title="properties.ControlTipText"
          :style="listStyleObj"
          @mousedown="scrollingOnListItems"
        >
          <div
            :style="tableStyleObj"
            class="table-style"
            @click="tableClick"
            ref="comboRef"
            tabindex="1"
          >
            <div class="tHeadStyle" v-if="properties.ColumnHeads === true">
              <div class="thClass" :style="colHeadsStyle">
                <template
                  :style="tdStyleObj"
                  v-if="properties.ListStyle === 1"
                  class="tdClass"
                ></template>
                <template
                  v-for="(a, columnIndex) in extraDatas.ColumnHeadsValues"
                >
                  <div
                    v-if="
                      (properties.RowSource !== '' &&
                        columnIndex < properties.ColumnCount) ||
                      properties.ColumnCount === -1
                    "
                    :key="columnIndex"
                    :style="updateColHeads(columnIndex)"
                    class="colHeadsClass"
                  >
                    {{ a }}
                  </div>
                </template>
                <div
                  v-if="properties.RowSource === ''"
                  :style="emptyColHeads"
                ></div>
                <hr v-if="properties.ColumnHeads" class="hrStyle" />
              </div>
            </div>
            <div v-else></div>
            <div
              class="tBodyStyle"
              @click="properties.Enabled ? (open = false) : (open = true)"
              v-if="properties.RowSource !== ''"
            >
              <div
                :tabindex="index"
                class="tr"
                :disabled="!properties.Enabled"
                v-for="(item, index) of tempArray"
                :key="index"
                @mouseenter="handleDrag"
                @keydown="handleExtendArrowKeySelect"
                @blur="clearMatchEntry"
                @mousedown="
                  isRunMode || isEditMode
                    ? properties.Enabled
                      ? handleMultiSelect($event)
                      : makeOpen()
                    : ''
                "
              >
                <div
                  :style="tdStyleObj"
                  class="tdClassIn"
                  v-if="
                    (properties.ListStyle === 1 &&
                      properties.ColumnCount > 0) ||
                    (properties.ListStyle === 1 &&
                      properties.ColumnCount === -1)
                  "
                >
                  <input name="radio" type="radio" class="inputClass" />
                </div>
                <div
                  class="column-item"
                  v-for="(i, index) in item"
                  :key="index"
                  :style="updateColumnValue(index)"
                >
                  <template
                    v-if="
                      index < properties.ColumnCount ||
                      properties.ColumnCount === -1
                    "
                    >{{ i }}</template
                  >
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
import {
  Component,
  Vue,
  Prop,
  Mixins,
  Watch,
  Ref
} from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import { Mutation, Action, Getter } from 'vuex-class'
@Component({
  name: 'FDComboBox'
})
export default class FDComboBox extends Mixins(FdControlVue) {
  $el!: HTMLDivElement;
  @Ref('textareaRef') textareaRef: HTMLTextAreaElement;
  @Ref('autoSizeTextarea') readonly autoSizeTextarea!: HTMLLabelElement;
  @Ref('hideSelectionDiv') readonly hideSelectionDiv!: HTMLDivElement;
  @Ref('comboRef') comboRef!: HTMLDivElement;

  private options = ['hello'];
  private tabindex = 0;
  eTargetValue: string = '';
  tempArray: Array<Array<string>> = [];
  open: boolean = false;
  multiselect = [];
  selectionStart: number = 0;
  selectionEnd: number = 0;
  tempInputValue: string = '';
  tempWidth: string = '0px';
  isScrolling: boolean = false;
  tempHeight: number;
  inBlur: boolean = false;
  makeOpen () {
    this.open = true
  }
  get getDisableValue () {
    if (this.isRunMode || this.isEditMode) {
      return this.properties.Enabled === false || this.properties.Locked
    } else {
      return true
    }
  }

  get emptyColHeads () {
    return {
      height: '15px'
    }
  }
  updateColumnValue (index: number) {
    return this.updateColumnWidths(index, this.tempHeight)
  }

  get selectionStyle () {
    const controlProp = this.properties
    return {
      borderLeft:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 2
            ? '2px solid gray'
            : controlProp.SpecialEffect === 3
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 4
                ? '0.5px solid gray'
                : '',
      borderTop:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 2
            ? '2px solid gray'
            : controlProp.SpecialEffect === 3
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 4
                ? '0.5px solid gray'
                : ''
    }
  }
  @Watch('open')
  updateTd () {
    if (!this.open && this.comboRef) {
      this.tempHeight = this.comboRef.children[1].children[0].clientHeight
    }
  }

  toFocus () {
    if (this.isEditMode) {
      this.isVisible = true
    } else {
      this.isVisible = false
    }
  }
  tableClick (e: Event) {
    this.tempListBoxComboBoxEvent = e
  }

  @Watch('properties.Font.FontSize', { deep: true })
  autoSizeValidateOnFontChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  @Watch('properties.RowSource')
  rowSourceValidate () {
    if (this.properties.RowSource !== '') {
      const initialRowSourceData = this.extraDatas.RowSourceData!
      if (initialRowSourceData) {
        this.tempArray = initialRowSourceData
      }
      this.updateDataModel({ propertyName: 'ControlSource', value: '' })
      if (initialRowSourceData && initialRowSourceData.length === 0) {
        this.updateDataModel({ propertyName: 'TopIndex', value: -1 })
      } else {
        this.updateDataModel({ propertyName: 'TopIndex', value: 0 })
      }
    }
  }

  handleTextInput (e: Event) {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
    if (e.target instanceof HTMLTextAreaElement) {
      const tempEvent = e.target
      this.eTargetValue = e.target.value
      this.updateDataModel({ propertyName: 'Value', value: this.eTargetValue })
      this.updateDataModel({ propertyName: 'Text', value: this.eTargetValue })
      if (this.properties.RowSource !== '') {
        if (this.properties.MatchEntry !== 0) {
          this.textareaRef.focus()
        }
        if (this.properties.MatchEntry === 0) {
          for (let i = 0; i < this.tempArray.length; i++) {
            if (this.tempArray[i][0][0] === this.textareaRef.value[0]) {
              this.textareaRef.value = this.tempArray[i][0]
              this.updateDataModel({
                propertyName: 'Text',
                value: this.tempArray[i][0]
              })
              break
            }
          }
          this.textareaRef.setSelectionRange(
            0,
            this.textareaRef.value.length,
            'forward'
          )
        }
      } else {
        this.tempArray = []
      }
    } else {
      throw new Error('target is not instance of div element')
    }
  }

  setSelection () {
    const setSelectionRef = this.textareaRef
    setSelectionRef.focus()
    setSelectionRef.select()
  }
  clearMatchEntry () {
    this.updateDataModelExtraData({ propertyName: 'MatchData', value: '' })
  }

  protected get selectionSpanObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent'
    }
  }

  protected get colHeadsStyle (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      backgroundColor: controlProp.BackColor,
      width: '100%'
    }
  }

  scrollingOnListItems (e: Event) {
    this.isScrolling = true
  }
  /**
   * @description  show selection when TextBox loses focus
   * when HideSelection is false selection is show if user selects any text
   * @function handleBlur
   * @event blur
   *
   */
  handleBlur (
    event: TextEvent,
    textareaRef: HTMLTextAreaElement,
    hideSelectionDiv: HTMLDivElement
  ) {
    this.inBlur = false
    if (this.isScrolling) {
      this.open = true
      this.textareaRef.focus()
    } else if (this.open) {
      this.inBlur = true
      this.open = false
    }
    this.isScrolling = false
    if (this.properties.ShowDropButtonWhen === 1) {
      this.isVisible = false
    }
    if (
      this.properties.EnterFieldBehavior === 1 &&
      event.target instanceof HTMLTextAreaElement
    ) {
      const eventTarget = event.target
      let tempField = this.tempInputValue.slice(
        eventTarget.selectionStart - eventTarget.selectionEnd
      )
      this.selectionStart = eventTarget.selectionStart
      this.selectionEnd = eventTarget.selectionEnd
    }
    if (
      !this.properties.HideSelection &&
      textareaRef &&
      event.target instanceof HTMLTextAreaElement
    ) {
      const eventTarget = event.target

      hideSelectionDiv.style.display = 'block'
      hideSelectionDiv.style.height = this.properties.Height! + 'px'
      hideSelectionDiv.style.width = this.properties.Width! + 'px'
      textareaRef.style.display = 'none'
      let textarea = eventTarget.value
      let firstPart =
        textarea.slice(0, eventTarget.selectionEnd) +
        '</span>' +
        textarea.slice(eventTarget.selectionEnd + Math.abs(0))
      let text =
        firstPart.slice(0, eventTarget.selectionStart) +
        "<span style='background-color:lightblue'>" +
        firstPart.slice(eventTarget.selectionStart + Math.abs(0))
      hideSelectionDiv.innerHTML = text
    }
    if (this.properties.MatchRequired && textareaRef) {
      const arrayCheck = this.extraDatas.RowSourceData!.findIndex(
        (element) => element[0] === this.tempInputValue
      )
      if (arrayCheck === -1) {
        this.textareaRef.focus()
      } else {
        return undefined
      }
    }
  }
  /**
   *@description hides the div when focus comes to textarea when hideSelection
   * properties is false
   * @function handleClick
   * @param event its of FocusEvent
   * @event click
   */
  handleClick (
    event: TextEvent,
    textareaRef: HTMLTextAreaElement,
    hideSelectionDiv: HTMLDivElement
  ) {
    if (!this.properties.HideSelection) {
      hideSelectionDiv.style.display = 'none'
    } else {
      return undefined
    }
    if (this.properties.EnterFieldBehavior === 0) {
      this.textareaRef.focus()
      this.textareaRef.select()
    } else if (this.properties.EnterFieldBehavior === 1) {
    } else {
      return undefined
    }
  }
  /**
   * @description hides div instead of textarea when hideSelection is false
   * when hideSelection properties is true textarea is shown
   * when hideSelection properties is false div is shown
   * @function divHide
   * @param event its of type MouseEvent
   * @event click
   */
  divHide (event: MouseEvent, textareaRef: HTMLTextAreaElement) {
    if (
      event.target instanceof HTMLSpanElement ||
      event.target instanceof HTMLDivElement
    ) {
      event.target.style.display = 'none'
      textareaRef.style.display = 'block'
      if (
        event.target.tagName === 'SPAN' &&
        event.target.parentNode!.nodeName === 'DIV'
      ) {
        (event.target.parentNode as HTMLElement).style.display = 'none'
      }
      textareaRef.focus()
      textareaRef.selectionStart = textareaRef.selectionEnd
    } else {
      throw new Error('event.target is not an instance of Span or Div Element')
    }
  }
  /**
   * @description dragBehavior - if true when dragging
   *  if false the cursor remains in same place
   * @function dragBehavior
   * @param event its of type KeyboardEvent
   * @event dragStart
   */
  dragBehavior (e: Event) {
    if (this.properties.DragBehavior) {
      return true
    }
    e.preventDefault()
  }

  /**
   * @override
   */
  @Watch('properties.AutoSize', { deep: true })
  updateAutoSize () {
    if (this.properties.AutoSize === true) {
      this.updateDataModel({ propertyName: 'SelectionMargin', value: false })
      this.$nextTick(() => {
        const textareaRef: HTMLTextAreaElement = this.textareaRef
        // replication of stype attribute to Label tag for autoSize property to work
        let tempLabel: HTMLLabelElement = this.autoSizeTextarea
        tempLabel.style.display = 'inline'
        tempLabel.style.fontStyle = textareaRef.style.fontStyle
        tempLabel.style.fontSize = parseInt(textareaRef.style.fontSize) + 'px'
        tempLabel.style.whiteSpace = textareaRef.style.whiteSpace
        tempLabel.style.wordBreak = textareaRef.style.wordBreak
        tempLabel.style.fontWeight = textareaRef.style.fontWeight
        tempLabel.style.width =
          (this.textareaRef.value.length + 1) *
            parseInt(textareaRef.style.fontSize) +
          'px'
        tempLabel.style.height = textareaRef.style.height
        tempLabel.innerText = textareaRef.value
        this.updateDataModel({
          propertyName: 'Width',
          value:
            tempLabel.offsetWidth > 20
              ? tempLabel.offsetWidth + 21
              : tempLabel.offsetWidth + 25
        })
        this.updateDataModel({
          propertyName: 'Height',
          value: tempLabel.offsetHeight + 5
        })
        tempLabel.innerText = ''
        tempLabel.style.display = 'none'
        this.selectionData[0] = this.eTargetValue
      })
    } else {
      return undefined
    }
  }

  protected get listStyleObj () {
    const controlProp = this.properties
    if (this.properties.RowSource !== '') {
      return {
        height: !controlProp.ColumnHeads
          ? controlProp.ListRows! > 0 &&
            controlProp.ListRows! < this.extraDatas.RowSourceData!.length
            ? controlProp.ListRows! * (controlProp.Font!.FontSize! + 9) + 'px'
            : ''
          : controlProp.ListRows! > 0 &&
            controlProp.ListRows! < this.extraDatas.RowSourceData!.length
            ? (controlProp.ListRows! + 1) * (controlProp.Font!.FontSize! + 9) +
            'px'
            : '',
        backgroundColor: controlProp.BackColor
      }
    } else {
      return {
        backgroundColor: controlProp.BackColor,
        border: '1px solid black',
        width: 'calc(100% - 2px)',
        height: 'calc(100% - 2px)',
        minWidth: '100px'
      }
    }
  }

  protected get tableStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    const font: font = controlProp.Font
      ? controlProp.Font
      : {
        FontName: 'Arial',
        FontSize: 20,
        FontItalic: true,
        FontBold: true,
        FontUnderline: true,
        FontStrikethrough: true
      }
    return {
      borderCollapse: 'collapse',
      tableLayout: 'fixed',
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent',
      color: controlProp.ForeColor,
      fontFamily: font.FontStyle! !== '' ? this.setFontStyle : font.FontName!,
      fontSize: `${font.FontSize}px`,
      fontStyle: font.FontItalic || this.isItalic ? 'italic' : '',
      textDecoration:
        font.FontStrikethrough === true && font.FontUnderline === true
          ? 'underline line-through'
          : font.FontUnderline
            ? 'underline'
            : font.FontStrikethrough
              ? 'line-through'
              : '',
      textUnderlinePosition: 'under',
      fontWeight: font.FontBold
        ? 'bold'
        : font.FontStyle !== ''
          ? this.tempWeight
          : '',
      fontStretch: font.FontStyle !== '' ? this.tempStretch : '',
      width:
        controlProp.ColumnWidths === ''
          ? 'calc(100% - 2px)'
          : 'calc(100% - 2px)' + parseInt(controlProp.ColumnWidths!) + 'px',
      outline: 'none'
    }
  }

  protected get cssStyleProperty (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    const font: font = controlProp.Font
      ? controlProp.Font
      : {
        FontName: 'Arial',
        FontSize: 20,
        FontItalic: true,
        FontBold: true,
        FontUnderline: true,
        FontStrikethrough: true
      }
    return {
      left: `${controlProp.Left}px`,
      width:
        controlProp.ShowDropButtonWhen === 0
          ? `${controlProp.Width! - 4}px`
          : controlProp.ShowDropButtonWhen === 1 && this.isVisible === false
            ? `${controlProp.Width! - 4}px`
            : controlProp.SelectionMargin
              ? `${controlProp.Width! - 30}px`
              : `${controlProp.Width! - 25}px`,
      height: `${controlProp.Height! - 5}px`,
      top: `${controlProp.Top}px`,
      fontFamily: font.FontStyle! !== '' ? this.setFontStyle : font.FontName!,
      fontSize: `${font.FontSize}px`,
      fontStyle: font.FontItalic || this.isItalic ? 'italic' : '',
      textDecoration:
        font.FontStrikethrough === true && font.FontUnderline === true
          ? 'underline line-through'
          : font.FontUnderline
            ? 'underline'
            : font.FontStrikethrough
              ? 'line-through'
              : '',
      textUnderlinePosition: 'under',
      fontWeight: font.FontBold
        ? 'bold'
        : font.FontStyle !== ''
          ? this.tempWeight
          : '',
      fontStretch: font.FontStyle !== '' ? this.tempStretch : '',
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent',
      color: controlProp.ForeColor,
      textAlign:
        controlProp.TextAlign === 0
          ? 'left'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'right',
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.isEditMode || !this.isActivated
            ? this.getMouseCursorData
            : 'default'
          : 'default'
    }
  }
  /**
   * @description style object is passed to :style attribute in div tag
   * dynamically changing the styles of the component based on cssStyleProperty of
   * textarea
   * @function divcssStyleProperty
   *
   */
  get divcssStyleProperty () {
    const styleObject = this.cssStyleProperty
    return {
      ...styleObject,
      display: 'none',
      whiteSpace: 'break-spaces',
      width: `${this.properties.Width! - 10}px`,
      height: `${this.properties.Height!}px`
    }
  }

  protected get customSelectObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible ? 'inline-block' : 'none'
    } else {
      display = 'inline-block'
    }
    return {
      display: display
      // overflow: 'hidden'
    }
  }
  protected get tdStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      textAlign:
        controlProp.TextAlign === 0
          ? 'left'
          : controlProp.TextAlign === 2
            ? 'right'
            : 'center',
      display: 'inline-block',
      marginRight: controlProp.ListStyle === 1 ? '5px' : ''
    }
  }
  setWidth () {
    this.tempWidth = `${this.properties.Width! + 20}px`
    return this.tempWidth
  }
  @Watch('properties.Value', { deep: true })
  textAndValueUpdateProp (newVal: string, oldVal: string) {
    if (this.properties.RowSource !== '') {
      if (
        this.properties.BoundColumn! > 0 &&
        this.properties.BoundColumn! < this.extraDatas.RowSourceData!.length
      ) {
        let tempData = [...this.extraDatas.RowSourceData!]
        if (tempData![0][this.properties.BoundColumn! - 1] === newVal) {
          this.updateDataModel({ propertyName: 'Value', value: newVal })
        } else {
          if (newVal !== '' && this.properties.Value) {
            this.updateDataModel({ propertyName: 'Text', value: newVal })
          }
        }
      }
      if (newVal !== '' && this.properties.Value) {
        this.selectionData[0] = newVal
        this.updateDataModel({ propertyName: 'Text', value: newVal })
      }
    } else {
      if (newVal !== '') {
        this.updateDataModel({ propertyName: 'Text', value: newVal })
      }
    }
  }

  @Watch('properties.SelectionMargin', { deep: true })
  checkSelectionMargin (newVal: boolean, oldVal: boolean) {
    this.selectionData[0] = this.eTargetValue
  }
  releaseEditMode (event: KeyboardEvent) {
    this.$el.focus()
    this.setContentEditable(event, false)
  }

  mounted () {
    this.$el.focus()
    if (this.properties.RowSource !== '') {
      const initialRowSourceData = this.extraDatas.RowSourceData!
      if (initialRowSourceData) {
        this.tempArray = initialRowSourceData
      }
      this.updateDataModel({ propertyName: 'ControlSource', value: '' })
      if (initialRowSourceData && initialRowSourceData.length === 0) {
        this.updateDataModel({ propertyName: 'TopIndex', value: -1 })
      } else {
        this.updateDataModel({ propertyName: 'TopIndex', value: 0 })
      }
    }
  }

  @Watch('properties.ControlSource', { deep: true })
  updateControlSourceValue (newVal: string, oldVal: string) {
    const propData: controlData = this.data
    if (propData.properties.ControlSource !== '') {
      const controlSourceValue = propData.extraDatas!.ControlSourceValue
      this.updateDataModel({
        propertyName: 'Value',
        value: controlSourceValue
      })
      this.updateDataModel({ propertyName: 'Text', value: controlSourceValue })
    } else {
      this.updateDataModel({
        propertyName: 'Value',
        value: ''
      })
      this.updateDataModel({ propertyName: 'Text', value: '' })
    }
  }
  expandWidth () {
    if (this.properties.ShowDropButtonWhen === 0) {
      return 'hidden'
    } else if (this.properties.ShowDropButtonWhen === 1) {
      if (this.isVisible) {
        return 'visible'
      } else {
        return 'hidden'
      }
    }
  }

  protected get labelStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    const font: font = controlProp.Font
      ? controlProp.Font
      : {
        FontName: 'Arial',
        FontSize: 20,
        FontItalic: true,
        FontBold: true,
        FontUnderline: true,
        FontStrikethrough: true
      }
    return {
      fontFamily: font.FontStyle! !== '' ? this.setFontStyle : font.FontName!,
      fontSize: `${font.FontSize}px`,
      fontStyle: font.FontItalic || this.isItalic ? 'italic' : '',
      textDecoration:
        font.FontStrikethrough === true && font.FontUnderline === true
          ? 'underline line-through'
          : font.FontUnderline
            ? 'underline'
            : font.FontStrikethrough
              ? 'line-through'
              : '',
      fontWeight: font.FontBold
        ? 'bold'
        : font.FontStyle !== ''
          ? this.tempWeight
          : '',
      fontStretch: font.FontStyle !== '' ? this.tempStretch : ''
    }
  }

  protected get boxStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      borderColor: controlProp.BorderStyle === 1 ? controlProp.BorderColor : '',
      // borderLeft: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 2 ? '2px solid gray' : controlProp.SpecialEffect === 3 ? '1.5px solid gray' : controlProp.SpecialEffect === 4 ? '0.5px solid gray' : '',
      borderRight:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 1
            ? '2px solid gray'
            : controlProp.SpecialEffect === 4
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 3
                ? '0.5px solid gray'
                : '',
      // borderTop: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 2 ? '2px solid gray' : controlProp.SpecialEffect === 3 ? '1.5px solid gray' : controlProp.SpecialEffect === 4 ? '0.5px solid gray' : '',
      borderBottom:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 1
            ? '2px solid gray'
            : controlProp.SpecialEffect === 4
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 3
                ? '0.5px solid gray'
                : '',
      display: 'grid',
      gridTemplateColumns: `${controlProp.Width! - 20}px` + ' 21px',
      gridTemplateRows: `${controlProp.Height!}px`,
      outline: 'none'
    }
  }

  protected get itemsStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      width:
        parseInt(controlProp.ListWidth!) > 0
          ? parseInt(controlProp.ListWidth!) + 'px'
          : this.setdropDownWidth,
      height:
        controlProp.RowSource !== ''
          ? ''
          : controlProp.ColumnHeads
            ? '30px'
            : '15px',
      border: controlProp.RowSource !== '' ? '1px solid black' : 'none',
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.isEditMode || !this.isActivated
            ? this.getMouseCursorData
            : 'default'
          : 'default'
    }
  }

  get setdropDownWidth () {
    if (this.properties.Width! < 100) {
      return '100px'
    } else {
      return `${this.properties.Width!}px`
    }
  }

  protected get selectedStyleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      visibility:
        controlProp.ShowDropButtonWhen === 2 ? 'visible' : this.expandWidth(),
      backgroundPosition:
        controlProp.DropButtonStyle === 1 ? 'center' : 'bottom',
      backgroundSize:
        controlProp.DropButtonStyle === 1
          ? '12px 12px'
          : controlProp.DropButtonStyle === 2
            ? '8px 8px'
            : controlProp.DropButtonStyle === 3
              ? '9px 14px'
              : '',
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.isEditMode || !this.isActivated
            ? this.getMouseCursorData
            : 'default'
          : 'default',
      display: 'flex',
      justifyContent: 'center',
      alignItems: controlProp.DropButtonStyle === 1 ? 'center' : 'flex-end'
    }
  }
  enabledCheck (e: MouseEvent) {
    if (this.isRunMode || this.isActivated || this.isEditMode) {
      if (this.open) {
        this.textareaRef.focus()
        this.open = false
      } else if (this.inBlur) {
        this.open = false
      } else {
        this.open = true
        this.textareaRef.focus()
      }
    }
    this.inBlur = false
  }

  eventStoppers () {
    const eventStop = (event: Event) => event.stopPropagation()
    return this.isEditMode === false
      ? null
      : {
        keydown: eventStop
      }
  }
}
</script>

<style scoped>
.custom-select {
  position: relative;
  text-align: left;
  outline: none;
  box-sizing: border-box;
}
.selectionSpan {
  width: 5px;
}
.selectionDiv {
  display: grid;
  grid-template-columns: 5px auto;
}
.selected {
  background-color: rgb(238, 238, 238);
  border: 1px solid #858586;
  border-left: 0px;
  color: black;
  background-size: 40%;
  background-position: center;
  background-repeat: no-repeat;
  cursor: pointer;
  width: 20px;
  height: calc(100% - 2px);
}

.items {
  color: black;
  background-color: white;
  overflow: hidden;
  left: 0;
  right: 0;
  width: calc(100% + 20px);
  cursor: context-menu;
}
.item {
  color: black;
  cursor: pointer;
  border-left: 1px solid black;
  border-right: 1px solid black;
}
.item:hover {
  background-color: #0380fc;
  color: white;
  border: 1px dotted black;
}
.selectHide {
  display: none;
}
.combobox {
  display: grid;
  grid-template-columns: auto 20px;
}
.columnHeads {
  height: 19.2px;
  border: 1px solid black;
  border-top: 0px;
  display: grid;
  grid-template-columns: 100%;
}
.listStyle {
  width: 100%;
  height: 100%;
  background-color: lightgray;
  border: 1px solid gray;
  overflow: auto;
}
.list-outer {
  border: 0.1px solid lightgray;
  width: 164px;
  height: 130px;
  overflow-y: auto;
  box-shadow: -1px -1px grey;
}
.tr {
  outline: none;
  display: flex;
}
.tr:hover:not([disabled]) {
  background-color: rgb(59, 122, 231);
}
.ul {
  display: grid;
  grid-template-columns: 100%;
  list-style-type: none;
  margin-block-start: 0em;
  margin-block-end: 0em;
  margin-inline-start: 0px;
  margin-inline-end: 0px;
  padding-inline-start: 0px;
}
.li {
  margin: 1px;
  text-align: left;
  font-size: 14px;
  height: 16px;
  border: 0.5px solid white;
  cursor: pointer;
}
.li:hover {
  margin: 1px;
  border-color: rgb(52, 52, 255);
  border-radius: 2px;
  background-color: rgb(59, 122, 231);
}
.fa {
  margin-left: 4px;
  margin-top: 3px;
}
.span {
  margin-left: 7px;
  margin-top: 0px;
  font-family: Arial, Helvetica, sans-serif;
}
.labelStyle {
  white-space: nowrap;
}
.div {
  display: grid;
  grid-template-columns: 1fr;
}
.listbox {
  display: grid;
  grid-template-columns: 100%;
  height: 100px;
  width: 300px;
  background-color: lightgray;
  border: 1px solid gray;
}
.columnHeads {
  height: 19.2px;
  border: 1px solid black;
  display: grid;
  grid-template-columns: 100%;
}
.optionsItems {
  display: grid;
  grid-template-columns: 100%;
}
.text-box-design {
  resize: none;
  overflow: hidden;
  border: none;
  font-family: monospace;
}
.text-box-design:focus {
  outline: none;
}
.text-box-design::selection {
  background: lightblue;
}
.table-style {
  width: calc(100% - 2px);
}
.thClass {
  position: sticky;
  top: 0;
  overflow: hidden;
  text-decoration: underline;
  white-space: nowrap;
}
.tdClass {
  width: 15px;
  border-right: 1px solid;
}
.tdClassIn {
  width: 10px;
}
.inputClass {
  margin: 0;
}
.hrStyle {
  margin: 0px;
}
.forPlain {
  background-image: none;
}
.tHeadStyle {
  position: sticky;
  top: 0px;
}
.column-item {
  display: flex;
}
.colHeadsClass {
  display: inline-block;
}
</style>
