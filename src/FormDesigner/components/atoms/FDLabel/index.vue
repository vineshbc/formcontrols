<template>
  <label
    class="label"
    :style="cssStyleProperty"
    :name="properties.Name"
    :tabindex="properties.TabIndex"
    :title="properties.ControlTipText"
    @mousedown="addEventCustomCallback"
    @keydown.enter.prevent="setContentEditable($event, true)"
    @click.stop="labelClick"
  >
    <div id="logo" :style="reverseStyle">
    <img v-if="properties.Picture" id="img" :src="properties.Picture" :style="imageProperty">
    <div v-if="!syncIsEditMode" id="label" :style="labelStyle">
      <span>{{ computedCaption.afterbeginCaption }}</span>
      <span class="spanClass">{{ computedCaption.acceleratorCaption }}</span>
      <span>{{ computedCaption.beforeendCaption }}</span>
    </div>
    <FDEditableText
      v-else
      id="label"
      :editable="isRunMode === false && syncIsEditMode"
      :style="labelStyle"
      :caption="properties.Caption"
      @updateCaption="updateCaption"
      @releaseEditMode="releaseEditMode"
    >
    </FDEditableText>
    </div>
  </label>
</template>
<script lang="ts">
/*eslint-disable */
import { Component, Watch, Mixins, Emit } from 'vue-property-decorator'
import FDEditableText from '@/FormDesigner/components/atoms/FDEditableText/index.vue'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import Vue from 'vue'

@Component({
  name: 'FDLabel',
  components: {
    FDEditableText
  }
})
export default class FDLabel extends Mixins(FdControlVue) {
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
  imageProperty={
    height:'fit-content'
  }
  $el!: HTMLLabelElement;
  /**
   * @description style object is passed to :style attribute in label tag
   * dynamically changing the styles of the component based on propControlData
   * @function cssStyleProperty
   *
   */
  protected get cssStyleProperty (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    this.pictureSize()
    this.reverseStyle.justifyContent = "center"
    if(!controlProp.Picture){
    this.reverseStyle.justifyContent = 
    controlProp.TextAlign === 0 ? 'flex-start': controlProp.TextAlign === 1 ? 'center' : 'flex-end'
    }
    else{
    this.positionLogo(controlProp.PicturePosition)
    }
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
    let alignItems = 'normal'
    if(controlProp.Picture){
      display = 'flex'
      let labelStyle = document.getElementById('logo')
      if (this.properties.Height! > labelStyle!.clientHeight) {
         alignItems = 'center'
      }
    }
    return {
      ...(!controlProp.AutoSize && this.renderSize),
      ...this.baseStyle,
      left: `${controlProp.Left}px`,
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      top: `${controlProp.Top}px`,
      backgroundColor: controlProp.BackStyle ? controlProp.BackColor : 'transparent',
      borderColor: controlProp.BorderStyle === 1 ? controlProp.BorderColor : '',
      textAlign:
        controlProp.TextAlign === 0
          ? 'left'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'right',
      borderLeft: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 2 ? '2px solid gray' : controlProp.SpecialEffect === 3 ? '1.5px solid gray' : controlProp.SpecialEffect === 4 ? '0.5px solid gray' : '',
      borderRight: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 1 ? '2px solid gray' : controlProp.SpecialEffect === 4 ? '1.5px solid gray' : controlProp.SpecialEffect === 3 ? '0.5px solid gray' : '',
      borderTop: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 2 ? '2px solid gray' : controlProp.SpecialEffect === 3 ? '1.5px solid gray' : controlProp.SpecialEffect === 4 ? '0.5px solid gray' : '',
      borderBottom: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : controlProp.SpecialEffect === 1 ? '2px solid gray' : controlProp.SpecialEffect === 4 ? '1.5px solid gray' : controlProp.SpecialEffect === 3 ? '0.5px solid gray' : '',
      whiteSpace: controlProp.WordWrap ? 'pre-wrap' : 'pre',
      wordBreak: controlProp.WordWrap ? 'break-all' : 'normal',
      color:
        controlProp.Enabled === true ? controlProp.ForeColor : this.getEnabled,
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.getMouseCursorData
          : 'default',
      // Fix Font.FontSize, Font.FontItalic ...
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
      //  position: 'relative',
      // backgroundImage: `url(${controlProp.Picture})`,
      // backgroundRepeat: this.getRepeat,
      // backgroundPosition: this.getPosition,
      // backgroundPositionX: this.getPositionX,
      // backgroundPositionY: this.getPositionY,
      display: display,
      alignItems: alignItems
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
  // @Watch('properties.Height')
  // updateImageSizeHeight () {
  //   const  imgStyle={
  //     width:'auto',
  //     height:'fit-content'
  //   }
  //   if (this.properties.Picture) {
  //     Vue.nextTick(() => {
  //     const imgProp = document.getElementById('img')
  //     // console.log('imgStyle||', imgProp!.clientHeight,imgProp!.clientWidth)
  //     if (this.properties.Height! < imgProp!.clientHeight) {
  //         imgStyle.width = '100%'
  //         imgStyle.height = '100%'
  //         this.reverseStyle.display = 'contents'
  //     }
  //     })
  //   }
  //     this.imageProperty = imgStyle
  // }
  // @Watch('properties.Width')
  // updateImageSizeWidth () {
  //   const  imgStyle={
  //     width:'auto',
  //     height:'fit-content'
  //   }
  //   if (this.properties.Picture) {
  //     Vue.nextTick(() => {
  //     const imgProp = document.getElementById('img')
  //     if (this.properties.Width! < imgProp!.clientWidth) {
  //         imgStyle.width = '100%'
  //         imgStyle.height = '100%'
  //         this.reverseStyle.display = 'contents'
  //     }
  //     })
  //   }
  //     this.imageProperty = imgStyle
  // }

  /**
   * @description watches changes in propControlData to set autoset when true
   * @function autoSize
   * @param oldVal previous propControlData data
   * @param newVal  new/changed propControlData data
   */
  @Watch('properties.AutoSize', { deep: true })
  autoSize () {
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



  /**
   * @description updateAutoSize calls Vuex Actions to update object
   * @function updateAutoSize
   * @override
   */
  updateAutoSize () {
    if (this.properties.AutoSize === true) {
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
          value: (this.$el.childNodes[0] as HTMLSpanElement).offsetHeight + 10
        })
        this.updateDataModel({
          propertyName: 'Width',
          value: (this.$el.childNodes[0] as HTMLSpanElement).offsetWidth + 5
        })
      })
    } else {
      return undefined
    }
  }
  mounted () {
    this.$el.focus()
  }
  releaseEditMode (event: KeyboardEvent) {
    this.$el.focus()
    this.setContentEditable(event, false)
  }
  labelClick (event: MouseEvent) {
    this.selectedItem(event)
    if (this.isEditMode) {
      (this.$el.children[0] as HTMLSpanElement).focus()
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
      height:'fit-content'
    }
    if (this.properties.Picture) {
        Vue.nextTick(() => {
          const imgProp = document.getElementById('img')
           imgStyle.width = this.properties.Width! < imgProp!.clientWidth ? `${this.properties.Width}px` : 'fit-content'
           imgStyle.height = this.properties.Height! < imgProp!.clientHeight ? `${this.properties.Height}px` : 'fit-content'
        })
    }
      this.imageProperty = imgStyle
  }
}
</script>

<style scoped>
.label {
  float: left;
  padding-left: 2px;
  overflow: hidden;
  outline: none;
  box-sizing: border-box;
  width: 0px;
  height: 0px;
  left: 0px;
  top: 0px;
  align-items: center;
  justify-content: center;
}
.spanClass {
  text-decoration: underline;
  text-underline-position: under;
}
#logo{
 display: inline-flex;
 justify-content: center;
 /* width: 100%; */
}

/* #img{
  width: 100%;
} */
</style>
