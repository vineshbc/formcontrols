<template>
  <label
    class="label"
    id="idLabel"
    :style="cssStyleProperty"
    :name="properties.Name"
    :tabindex="properties.TabIndex"
    :title="properties.ControlTipText"
    @mousedown="addEventCustomCallback"
    @keydown.enter.prevent="setContentEditable($event, true)"
    @click.stop="labelClick"
  >
    <div id="logo" ref="logoProps" :style="reverseStyle">
    <img  v-if="properties.Picture" id="img" ref="imageProps" :src="properties.Picture" :style="[imageProperty,imagePos]">
    <div v-if="!syncIsEditMode" id="label" :style="labelStyle" ref="labelProps">
      <span>{{ computedCaption.afterbeginCaption }}</span>
      <span class="spanClass">{{ computedCaption.acceleratorCaption }}</span>
      <span>{{ computedCaption.beforeendCaption }}</span>
    </div>
    <FDEditableText
      ref="labelProps"
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
   width:'',
   alignSelf:'center'
  }
  imageProperty={
    height:'fit-content',
  }
  imagePos:any ={}
  $el!: HTMLLabelElement;
  $refs:{
    imageProps:HTMLImageElement,
    labelProps:HTMLStyleElement,
     logoProps:HTMLStyleElement
  }
  /**
   * @description style object is passed to :style attribute in label tag
   * dynamically changing the styles of the component based on propControlData
   * @function cssStyleProperty
   *
   */
  protected get cssStyleProperty (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    this.reverseStyle.justifyContent = "center"
    if(!controlProp.Picture){
    this.reverseStyle.justifyContent = 
    controlProp.TextAlign === 0 ? 'flex-start': controlProp.TextAlign === 1 ? 'center' : 'flex-end'
    }
    else{
    this.positionLogo(controlProp.PicturePosition)
    this.pictureSize()
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
    const labelAlignItems = 'inherit'
    if(controlProp.Picture){
      display = 'flex'
      Vue.nextTick(()=>{
          this.labelAlignment()
      })
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
      alignItems: labelAlignItems
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

  @Watch('properties.Picture', {deep:true})
  pictureChange(){
    if(this.properties.Picture){
    const  imgStyle={
      width:'auto',
      height:'auto',
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
   * @description updateAutoSize calls Vuex Actions to update object
   * @function updateAutoSize
   * @override
   */
  updateAutoSize () {
    if (this.properties.AutoSize === true) {
      const  imgStyle={
      width:'fit-content',
      height:'fit-content',
      }
      this.imageProperty = imgStyle
      if(this.properties.Picture){
      this.positionLogo(this.properties.PicturePosition)
      }
      this.$nextTick(() => {
        this.updateDataModel({
          propertyName: 'Height',
          value:  this.getWidthHeight().height
        })
        this.updateDataModel({
          propertyName: 'Width',
          value: this.getWidthHeight().width + 6
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
      width:'',
      alignSelf:'center'
      }
      this.imagePos.alignSelf = ''
      this.reverseStyle.display = 'flex'
      switch(value) {
        case 0: 
        break
        case 1:style.alignItems = 'center'
        // this.reverseStyle.alignItems = 'center'
        this.imagePos.alignSelf = 'center'
        break
        case 2:style.alignItems = 'flex-end'
        this.reverseStyle.alignItems = 'flex-end'
        break
        case 3: this.reverseStyle.flexDirection = 'row-reverse'
        break
        case 4:
        this.reverseStyle.flexDirection = 'row-reverse'
        style.alignItems = 'center'
        // this.reverseStyle.alignItems = 'center'
        this.imagePos.alignSelf = 'center'
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
        style.width = 'fit-content'
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
           const imgProp = this.$refs.imageProps
           console.log("image size||",this.properties.Width,imgProp!.clientWidth)
           imgStyle.width = this.properties.Width! <= imgProp!.clientWidth ? `${this.properties.Width}px` : 'fit-content'
           imgStyle.height = this.properties.Height! <= imgProp!.clientHeight ? `${this.properties.Height}px` : 'fit-content'    
           if(this.properties.PicturePosition === 9 || this.properties.PicturePosition === 10 || this.properties.PicturePosition === 11 ){
           imgProp!.scrollIntoView(true)          
           }
       })
    }
      this.imageProperty = imgStyle
  }
  labelAlignment(){
    this.reverseStyle.alignSelf = 'inherit'
    if (this.$refs.imageProps && this.$refs.imageProps.naturalHeight > this.properties.Height! ){
          this.reverseStyle.alignSelf = 'inherit'
        }
    else {
      let labelProp = this.$refs && this.$refs.logoProps? this.$refs.logoProps : this.$refs.logoProps.$el
        if (this.properties.Width! >= labelProp!.clientWidth && this.properties.Height! >= labelProp!.clientHeight) {
       this.reverseStyle.alignSelf = 'center'
        }
      }
  }
  getWidthHeight(){
    let picPosLeftRight=[0,1,2,3,4,5]
    let picPosTopBottom = [6,7,8,9,10,11]

    const imgHeight = this.$refs.imageProps && this.$refs.imageProps.naturalHeight
    const imgWidth = this.$refs.imageProps && this.$refs.imageProps.naturalWidth
    const labelHeight = this.$refs.labelProps && this.$refs.labelProps.offsetHeight? this.$refs.labelProps.offsetHeight : this.$refs.labelProps.$el.offsetHeight
    const labelWidth = this.$refs.labelProps && this.$refs.labelProps.offsetWidth ? this.$refs.labelProps.offsetWidth : this.$refs.labelProps.$el.offsetWidth
    let widthHeightData = {
      width: labelWidth,
      height: labelHeight
    }
    if(this.properties.Picture){
    if( picPosLeftRight.includes(this.properties.PicturePosition!)){
      if(imgHeight >= labelHeight) {
        widthHeightData.height = imgHeight
      } else {
        widthHeightData.height = labelHeight
      }
      widthHeightData.width = imgWidth + labelWidth
    } 
     else if(picPosTopBottom.includes(this.properties.PicturePosition!)){
       
       if(imgWidth >= labelWidth){
         widthHeightData.width = imgWidth
       }else {
         widthHeightData.width = labelWidth 
       }
        widthHeightData.height = imgHeight + labelHeight
     }
     else if (this.properties.PicturePosition! === 12) {
         widthHeightData.width = imgWidth >= labelWidth ? imgWidth : labelWidth
         widthHeightData.height = imgHeight >= labelHeight ? imgHeight : labelHeight

       }
     }
    return widthHeightData
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
