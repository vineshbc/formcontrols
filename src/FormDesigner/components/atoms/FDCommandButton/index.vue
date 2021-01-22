<template>
  <button
    class="commandbutton"
    id="idBtn"
    :style="styleObj"
    :name="properties.Name"
    :tabindex="properties.TabIndex"
    :title="properties.ControlTipText"
    :runmode="getDisableValue"
    @blur="
      () => {
        isClicked = false;
      }
    "
    @mousedown="controlEditMode"
    @keydown.enter.prevent="setContentEditable($event, true)"
    @click.stop="commandButtonClick"
  >
  <div id="logo" :style="reverseStyle">
    <img v-if="properties.Picture" id="img" ref="imageProps" :src="properties.Picture" :style="imageProperty">
    <div v-if="!syncIsEditMode || isRunMode" :style="labelStyle">
      <span>{{ computedCaption.afterbeginCaption }}</span>
      <span class="spanClass">{{ computedCaption.acceleratorCaption }}</span>
      <span>{{ computedCaption.beforeendCaption }}</span>
    </div>
    <FDEditableText
      v-else
      :editable="isRunMode === false && syncIsEditMode"
      :style="labelStyle"
      :caption="properties.Caption"
      @updateCaption="updateCaption"
      @releaseEditMode="releaseEditMode"
    >
    </FDEditableText>
    </div>
  </button>
</template>

<script lang="ts">
/*eslint-disable */
import { Component, Emit, Mixins, Prop, Watch } from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import FDEditableText from '@/FormDesigner/components/atoms/FDEditableText/index.vue'
import Vue from 'vue'

@Component({
  name: 'FDCommandButton',
  components: {
    FDEditableText
  }
})
export default class FDCommandButton extends Mixins(FdControlVue) {
  $el!: HTMLButtonElement;
  $refs:{
    imageProps:HTMLImageElement
  }
  isClicked: boolean = false;
  isContentEditable: boolean = false;
  labelStyle = {}
  reverseStyle = {
   display : '',
   flexDirection : '',
   justifyItems : '',
   position:'',
   justifyContent:'center',
   alignItems: '',
   width:''
  }
  imageProperty = {
    height:'fit-content'
  }
  /**
   * @description getDisableValue checks for the RunMode of the control and then returns after checking for the Enabled
   * and the Locked property
   * @function getDisableValue
   */
  get getDisableValue () {
    if (this.isRunMode) {
      return (
        this.properties.Enabled === false || this.properties.Locked === true
      )
    } else {
      return true
    }
  }

  /**
   * @description commandButtonClick is a method to check the check the clicked functionality of the button tag.
   * Also It sets the variable isClicked based on the Locked property
   * @function commandButtonClick
   *
   */
  commandButtonClick (e: Event) {
    if (!this.isRunMode) {
      this.selectedItem(e)
    }
    if (this.isActivated) {
      if (this.properties.Locked) {
        this.isClicked = false
      } else {
        this.isClicked = true
      }
    }
    if (this.isEditMode) {
      (this.$el.children[0] as HTMLSpanElement).focus()
    }
  }

  /**
   * @description changes width and height when autoSize is true by getting content offsetWidth
   *  and offsetHeight with the help of Ref attribute
   * @function updateAutoSize
   * @override
   */
  updateAutoSize () {
    if (this.properties.AutoSize === true) {
      const btnId = document.getElementById('idBtn')
      btnId!.style.alignItems = 'baseline'
      const  imgStyle={
      width:'fit-content',
      height:'fit-content'
      }
      this.imageProperty = imgStyle
      if(this.properties.Picture){
      this.positionLogo(this.properties.PicturePosition)
      }
      this.$nextTick(() => {
        this.updateDataModel({
          propertyName: 'Height',
          value: (this.$el.childNodes[0] as HTMLSpanElement).offsetHeight + 5
        })
        this.updateDataModel({
          propertyName: 'Width',
          value: (this.$el.childNodes[0] as HTMLSpanElement).offsetWidth + 5
        })
      })
    }
  }

  positionLogo(value:any){
      let style = {
        order: Number(),
        alignItems: '',
        transform:'',
        top:'',
        left:'',
        position:'',
        display:'inline-flex',
        width: '',
        justifyContent:''
      }
      this.reverseStyle = {
      display : '',
      flexDirection : '',
      justifyItems : '',
      position:'',
      justifyContent:'center',
      alignItems: '',
      width:''
      }
      this.reverseStyle.display = 'flex'
      switch(value) {
        case 0: 
        break
        case 1:style.alignItems = 'center'
        this.reverseStyle.alignItems = 'center'
        break
        case 2:style.alignItems = 'flex-end'
        this.reverseStyle.alignItems = 'flex-end'
        break
        case 3: this.reverseStyle.flexDirection = 'row-reverse'
        break
        case 4:
        this.reverseStyle.flexDirection = 'row-reverse'
        style.alignItems = 'center'
        this.reverseStyle.alignItems = 'center'
        break
        case 5:
        this.reverseStyle.flexDirection = 'row-reverse'
        style.alignItems = 'flex-end'
        this.reverseStyle.alignItems = 'flex-end'
        break
        case 6:
        this.reverseStyle.display = 'grid'
        break
        case 7:  
        this.reverseStyle.display = 'grid'
        this.reverseStyle.justifyItems = 'center'
        break
        case 8:  
        this.reverseStyle.display = 'grid'
        this.reverseStyle.justifyItems = 'end'
        break
        case 9:  
        this.reverseStyle.display = 'grid'
        style.order= -1
        break
        case 10:  
        this.reverseStyle.display = 'grid'
        this.reverseStyle.justifyItems = 'center'
        style.order = -1
        break
        case 11:  
        this.reverseStyle.display = 'grid'
        this.reverseStyle.justifyItems = 'end'
        style.order= -1
        break
        case 12:  
        this.reverseStyle.position = 'relative'
        this.reverseStyle.width = '100%'
        style.position = 'absolute',
        style.top = '50%',
        style.left = '50%',
        style.transform ='translate(-50%, -50%)'
        style.justifyContent = 'center'
        style.width = '100%'
        break
        default:
          console.log('none')
      }
      // console.log("style||",style)
      this.labelStyle = style 
    }
  pictureSize(){
    const  imgStyle={
      width:'fit-content',
      height:'fit-content',
      maxWidth:'100%',
    }
    
    if (this.properties.Picture) {
        Vue.nextTick(() => {
           const imgProp = document.getElementById('img')
           console.log("image size||",this.properties.Width,imgProp!.clientWidth)
           imgStyle.width = this.properties.Width! <= imgProp!.clientWidth ? `${this.properties.Width}px` : 'fit-content'
           imgStyle.height = this.properties.Height! <= imgProp!.clientHeight ? `${this.properties.Height}px` : 'fit-content'    
           imgProp!.scrollIntoView(true)          
       })
    }
      this.imageProperty = imgStyle
  }

  /**
   * @description style object is passed to :style attribute in button tag
   * dynamically changing the styles of the component based on properties
   * @function styleObj
   */
  protected get styleObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    this.pictureSize()
    const font: font = controlProp.Font
      ? controlProp.Font
      : {
        FontName: 'Arial',
        FontSize: 10,
        FontItalic: true,
        FontBold: true,
        FontUnderline: true,
        FontStrikethrough: true
      }
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible ? 'inline-block' : 'none'
    } else {
      display = 'inline-block'
    }
    this.reverseStyle.justifyContent = 'center'
    let alignItems = 'inherit'
    if(controlProp.Picture){
      display = 'flex'
      this.positionLogo(this.properties.PicturePosition)
      // this.$nextTick(() => {
      // if(this.$refs.imageProps.clientHeight < this.properties.Height! )
      // {
      //  alignItems = 'center'
      // }
      // })
      // let labelStyle = document.getElementById('logo')
      //   if (this.properties.Height! > labelStyle!.clientHeight) {
      //   alignItems = 'center'
      // }
        if ( this.$refs.imageProps && this.$refs.imageProps.naturalHeight > this.properties.Height! ){
          alignItems = 'inherit'
        }
      else {
        let labelStyle = document.getElementById('logo')
        if (this.properties.Width! > labelStyle!.clientWidth && this.properties.Height! > labelStyle!.clientHeight) {
        alignItems = 'center'
        }
      }
    }
    return {
      ...(!controlProp.AutoSize && this.renderSize),
      ...this.baseStyle,
      left: `${controlProp.Left}px`,
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      top: `${controlProp.Top}px`,
      borderTopColor: controlProp.Default ? 'black' : controlProp.BackColor,
      borderBottomColor: controlProp.Default ? 'black' : controlProp.BackColor,
      borderLeftColor: controlProp.Default ? 'black' : controlProp.BackColor,
      borderRightColor: controlProp.Default ? 'black' : controlProp.BackColor,
      outline: controlProp.Enabled
        ? controlProp.TakeFocusOnClick && this.isClicked
          ? '1px dotted black'
          : 'none'
        : 'none',
      outlineOffset:
        controlProp.TakeFocusOnClick && this.isClicked ? '-5px' : '0px',
      display: display,
      backgroundColor: controlProp.BackStyle ? controlProp.BackColor : 'transparent',
      color:
        controlProp.Enabled === true ? controlProp.ForeColor : this.getEnabled,
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.getMouseCursorData
          : 'default',
      fontFamily: (font.FontStyle! !== '') ? this.setFontStyle : font.FontName!,
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
      fontWeight: font.FontBold ? 'bold' : (font.FontStyle !== '') ? this.tempWeight : '',
      fontStretch: (font.FontStyle !== '') ? this.tempStretch : '',
      whiteSpace: controlProp.WordWrap ? 'pre-wrap' : 'pre',
      wordBreak: controlProp.WordWrap ? 'break-all' : 'normal',
      paddingLeft: controlProp.AutoSize ? '0px' : '0px',
      paddingRight: controlProp.WordWrap ? '0px' : '6px',
      alignItems: alignItems
      // backgroundImage: `url(${controlProp.Picture})`,
      // backgroundRepeat: this.getRepeat,
      // backgroundPosition: controlProp.Picture ? this.getPosition : '',
      // backgroundPositionX: controlProp.Picture ? this.getPositionX : '',
      // backgroundPositionY: controlProp.Picture ? this.getPositionY : ''
    }
  }
  /**
   * @description style object is passed to :style attribute in tag
   * dynamically changing the styles of the component based on properties
   * @function editCssObj
   *
   */
  protected get editCssObj (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    return {
      backgroundImage: 'none'
    }
  }

  /**
   * @description watches changes in propControlData to set autoset when true
   * @function autoSize
   * @param oldVal previous propControlData data
   * @param newVal  new/changed propControlData data
   */
  @Watch('properties.AutoSize', { deep: true })
  autoSize (newVal: boolean, oldVal: boolean) {
    // if autoSize is true then height and width value will not get updated
    this.updateAutoSize()
  }

  @Watch('properties.Font.FontSize', { deep: true })
  autoSizeValidateOnFontChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  @Watch('properties.WordWrap', { deep: true })
  autoSizeValidateOnWordWrapChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  @Watch('properties.Caption', { deep: true })
  autoSizeValidateOnCaptionChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  @Watch('properties.Picture', {deep:true})
  pictureChange(){
    if(this.properties.Picture){
    const  imgStyle={
      width:'auto',
      height:'auto'
    }
    this.imageProperty = imgStyle
    //wait for 100ms to get image tag propertie
      setTimeout(() => {
        this.pictureSize()
        this.positionLogo(this.properties.PicturePosition)
      }, 100);
    }
  }
  /**
   * @description mounted initializes the values which are required for the component
   */
  mounted () {
    this.$el.focus()
    this.updateAutoSize()
  }
  releaseEditMode (event: KeyboardEvent) {
    this.$el.focus()
    this.setContentEditable(event, false)
  }
}
</script>

<style scoped>
.commandbutton {
  overflow: hidden;
  display: inline-block;
  box-sizing: border-box;
  width: 0px;
  height: 0px;
  left: 0px;
  top: 0px;
  align-items: center;
  justify-content: center;
}
.commandbutton[runmode]:active {
  border-style: outset !important;
}
#logo{
 display: inline-flex;
 justify-content: center;
}
</style>
