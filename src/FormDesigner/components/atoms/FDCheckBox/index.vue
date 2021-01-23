<template>
  <div
    :title="properties.ControlTipText"
    class="outer-check"
    :style="cssStyleProperty"
    @click.stop="checkBoxClick"
    @keydown.enter.prevent="setContentEditable($event, true)"
    :tabindex="properties.TabIndex"
  >
    <label class="control" :style="controlStyleObj">
      <input
        @change="handleChange($event, checkboxInput)"
        ref="checkboxInput"
        :name="properties.Name"
        :tabindex="properties.TabIndex"
        :disabled="getDisableValue"
        type="checkbox"
        class="control-input visually-hidden" />
      <span
        class="control-indicator"
        :style="controlIndicatorStyleObj"
        ref="spanRef"
      ></span
    ></label>
      <div  ref="logoProps" id="logo" :style="reverseStyle">
      <img id="img" v-if="properties.Picture" ref="imageProps" :src="properties.Picture" :style="[imageProperty,imagePos]">
        <div ref="labelProps" v-if="!syncIsEditMode || isRunMode"
          @click="isRunMode && makeChecked($event)"
          :style="labelStyle"
        >
          <span>{{ computedCaption.afterbeginCaption }}</span>
          <span class="spanStyle">{{
            computedCaption.acceleratorCaption
          }}</span>
          <span>{{ computedCaption.beforeendCaption }}</span>
        </div>
        <FDEditableText
          v-else
          ref="labelProps"
          :editable="isRunMode === false && syncIsEditMode"
          :caption="properties.Caption"
          :style="labelStyle"
          @updateCaption="updateCaption"
          @releaseEditMode="releaseEditMode"
        >
        </FDEditableText>
      </div>
    </div>
</template>

<script lang="ts">
/*eslint-disable */
import { Component, Ref, Mixins, Watch, Vue } from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import FDEditableText from '@/FormDesigner/components/atoms/FDEditableText/index.vue'

@Component({
  name: 'FDCheckBox',
  components: {
    FDEditableText
  }
})
export default class FDCheckBox extends Mixins(FdControlVue) {
  @Ref('checkboxInput') checkboxInput!: HTMLInputElement;
  @Ref('spanRef') spanRef!: HTMLSpanElement;
  @Ref('labelProps') labelProps!: FDEditableText
  $el: HTMLDivElement
  imagePos:any ={}
  $refs:{
    imageProps:HTMLImageElement,
    labelProps:HTMLStyleElement,
     logoProps:HTMLStyleElement
  }
  alignItem: boolean = false
  labelStyle = {}
  reverseStyle:any = {
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
  get controlStyleObj () {
    const controlProp = this.properties
    return {
      order : controlProp.Alignment === 1 ? '0' : '1',
      position: 'sticky',
      top: `${controlProp.Height! / 2 - 10}px`
    }
  }
  /**
   * @description  watches Enabled property and the sets the backgroundColor
   * @function checkEnabled
   */
  @Watch('properties.Enabled', {
    deep: true
  })
  checkEnabled (newVal: boolean, oldVal: boolean) {
    if (!this.properties.Enabled) {
      this.spanRef.style.backgroundColor = 'rgba(220, 220, 220, 1)'
    } else {
      this.spanRef.style.backgroundColor = 'white'
    }
  }

  /**
   * @description  watches Value property and the sets the checked
   * @function verifyValue
   */
  @Watch('properties.Value', {
    deep: true
  })
  verifyValue () {
    if (this.isRunMode) {
      if (this.properties.Enabled && !this.properties.Locked) {
        this.handleValue(this.properties.Value! as string)
      }
    } else {
      this.handleValue(this.properties.Value! as string)
    }
  }

  handleValue (newVal: string) {
    let tempValue = newVal.toLowerCase()
    const checkDiv = this.checkboxInput
    if (!isNaN(parseInt(newVal))) {
      if (parseInt(newVal) === 0) {
        this.spanRef.style.backgroundColor = 'white'
        checkDiv.checked = false
      } else {
        this.spanRef.style.backgroundColor = 'white'
        checkDiv.checked = true
      }
    } else if (tempValue === 'true') {
      this.spanRef.style.backgroundColor = 'white'
      checkDiv.checked = true
    } else if (tempValue === 'false') {
      this.spanRef.style.backgroundColor = 'white'
      checkDiv.checked = false
    } else {
      checkDiv.checked = true
      this.spanRef.style.backgroundColor = 'rgba(220, 220, 220, 1)'
    }
  }
  /**
   * @description  makeChecked controls the checked of the control in RunMode
   * @function makeChecked
   */
  makeChecked () {
    if (!this.getDisableValue) {
      this.tripleState += 1
      const checkDiv = this.checkboxInput
      if (!this.properties.TripleState) {
        checkDiv.checked = !checkDiv.checked
        const tempval = checkDiv.checked
        this.updateDataModel({ propertyName: 'Value', value: tempval })
      } else {
        if (this.tripleState % 3 === 0) {
          checkDiv.checked = true
          this.spanRef.style.backgroundColor = 'rgba(220, 220, 220, 1)'
        } else {
          this.spanRef.style.backgroundColor = 'white'
          checkDiv.checked = !checkDiv.checked
          this.updateDataModel({
            propertyName: 'Value',
            value: checkDiv.checked
          })
        }
      }
    }
  }

  /**
   * @description getDisableValue checks for the RunMode of the control and then returns after checking for the Enabled
   * and the Locked property
   * @function getDisableValue
   */
  get getDisableValue () {
    if (this.isRunMode) {
      return this.properties.Enabled === false || this.properties.Locked
    } else {
      return true
    }
  }

  /**
   * @description style object is passed to :style attribute in span tag
   * dynamically changing the styles of the component based on properties
   * @function controlIndicatorStyleObj
   *
   */
  get controlIndicatorStyleObj () {
    const controlProp = this.properties
    return {
      boxShadow:
        controlProp.SpecialEffect === 0 ? '0px 0px gray' : '-1px -1px gray'
    }
  }

  /**
   * @description style object is passed to :style attribute in label tag
   * dynamically changing the styles of the component based on properties
   * @function cssStyleProperty
   *
   */
  get cssStyleProperty () {
    const controlProp = this.properties
    this.pictureSize()
    this.reverseStyle.justifyContent = "center"
    if(!controlProp.Picture){
    this.reverseStyle.justifyContent = 
    controlProp.TextAlign === 0 ? 'flex-start': controlProp.TextAlign === 1 ? 'center' : 'flex-end'
    }else{
    this.positionLogo(controlProp.PicturePosition)
    }
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
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible ? 'grid' : 'none'
    } else {
      display = 'grid'
    }
    const labelAlignItems = controlProp.Picture? 'inherit':'center'
    if(controlProp.Picture){
      Vue.nextTick(()=>{
          this.labelAlignment()
      })
    }
    return {
      left: `${controlProp.Left}px`,
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      top: `${controlProp.Top}px`,
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent',
      borderColor: controlProp.BorderColor,
      textAlign:
        controlProp.TextAlign === 0
          ? 'left'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'right',
      border: this.getBorderStyle,
      whiteSpace: controlProp.WordWrap ? 'pre-wrap' : 'pre',
      wordBreak: controlProp.WordWrap ? 'break-all' : 'normal',
      color:
        controlProp.Enabled === true ? controlProp.ForeColor : this.getEnabled,
      cursor:
        controlProp.MousePointer !== 0 || controlProp.MouseIcon !== ''
          ? this.getMouseCursorData
          : 'default',
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

      display: display,
      overflow: 'hidden',
      gridTemplateColumns: controlProp.Alignment === 1 ? '12px auto' : 'auto 12px',
      gridTemplateRows: '100%',
      gap: '2px',
      // alignItems: font.FontSize! > 17 ? 'center' : '',
      alignContent: 'center',
      boxShadow: 'none',
      alignItems: labelAlignItems
    }
  }

  /**
   * @description style object is passed to :style attribute in div tag
   * dynamically changing the styles of the component based on properties
   * @function divcssStyleProperty
   *
   */
  get divcssStyleProperty () {
    const controlProp = this.properties
    return {
      overflow: 'hidden',
      // height: !this.isEditMode ? '100%' : '',
      display: 'flex',
      justifyContent:
        controlProp.TextAlign === 0
          ? 'flex-start'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'flex-end',
      alignItems: 'center',
      textAlign:
        controlProp.TextAlign === 0
          ? 'start'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'end',
      // position: !this.isEditMode ? 'relative' : 'absolute'
    }
  }

  /**
   * @description style object is passed to :style attribute in div tag
   * dynamically changing the styles of the component based on properties
   * @function pictureDivStyle
   *
   */
  get pictureDivStyle () {
    const controlProp = this.properties
    return {
      height: '100%',
      display: !this.isEditMode ? 'table-cell' : 'flex',
      alignItems: this.alignItem ? 'baseline' : 'center',
      backgroundImage: `url(${controlProp.Picture})`,
      backgroundRepeat: this.getRepeat,
      backgroundPosition: this.getPosition,
      backgroundPositionX: this.getPositionX,
      backgroundPositionY: this.getPositionY,
      position: 'relative'
    }
  }

  get setAlignment () {
    return {
      editMode: this.isEditMode,
      caption: this.properties.Caption
    }
  }

  @Watch('setAlignment', { deep: true })
  editableTextVerify () {
    if (this.isEditMode) {
      Vue.nextTick(() => {
        if (this.isEditMode && this.labelProps.$el.clientHeight > this.properties.Height!) {
          this.alignItem = true
        } else {
          this.alignItem = false
        }
      })
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
  //         imgStyle.height = 'auto'
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
  //         imgStyle.height = 'auto'
  //         this.reverseStyle.display = 'contents'
  //     }
  //     })
  //   }
  //     this.imageProperty = imgStyle
  // }
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
   * @override
   */
  @Watch('properties.AutoSize', {
    deep: true
  })
  updateAutoSize () {
    if (this.properties.AutoSize) {
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
   * @description  sets controlSource if present and updates Value property
   * @function controlSource
   */
  mounted () {
    this.verifyValue()
    this.$el.focus()
    this.controlSource()
  }
  releaseEditMode (event: KeyboardEvent) {
    this.$el.focus()
    this.setContentEditable(event, false)
  }
  checkBoxClick (event: MouseEvent) {
    this.selectedItem(event)
    if (this.isEditMode) {
      (this.labelProps.$el as HTMLSpanElement).focus()
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
* {
  box-sizing: border-box;
}

.outer-check {
  height: 0px;
  width: 0px;
  left: 0px;
  top: 0px;
  min-width: 12px;
  min-height: 13px;
  background-color: rgb(238, 238, 238);
  box-shadow: -1px -1px gray;
  overflow: hidden;
  align-items: center;
  box-sizing: border-box;
}

.visually-hidden {
  border: 0;
  clip: rect(0, 0, 0, 0);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
}

.control {
  display: inline-flex;
  position: sticky;
  top: 47%;
  align-self: center;
}

.control-indicator {
  width: 10px;
  height: 10px;
  margin: 1px;
  background-color: white;
  border: 1px inset grey;
}

.control-input:checked ~ .control-indicator {
  background-image: url(../../../../assets/checkmark.png);
  background-size: 10px;
  background-position: center;
  background-repeat: no-repeat;
}

.spanStyle {
  text-decoration: underline;
  text-underline-position: under;
}

.menu {
  width: 10%;
  margin: 0 auto;
}

.main {
  width: 90%;
  margin: 0 auto;
}

#logo{
 display: inline-flex;
 justify-content: center;
}
#logo-main{
  display: flex;
  justify-content: center;
}

</style>
