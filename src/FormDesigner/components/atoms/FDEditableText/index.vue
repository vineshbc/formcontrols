<template>
  <span
    class="editText"
    ref="labelArea"
    role="textbox"
    contenteditable="true"
    :readonly="editable === false"
    @input="onInput"
    v-on="eventStoppers()">
  </span>
</template>

<script lang="ts">
import { Component, Prop, Vue, Ref } from 'vue-property-decorator'

@Component({
  name: 'FDEditableText'
})
export default class FDEditableText extends Vue {
  @Prop() private editable! : boolean
  @Prop() private caption! : string

  @Ref('labelArea') readonly labelArea! : HTMLSpanElement

  mounted () {
    this.labelArea.innerText = this.caption
    this.labelArea.focus()
  }

  eventStoppers () {
    const eventStop = (event: Event) => event.stopPropagation()
    return this.editable === false ? null : {
      mousedown: eventStop,
      keydown: this.keydownHandle,
      click: eventStop
    }
  }

  keydownHandle (e: KeyboardEvent) {
    e.stopPropagation()
    if (e.key === 'Escape') {
      this.$emit('releaseEditMode')
    }
    if (e.key === 'Enter') {
      e.preventDefault()
    }
  }

  onInput (event: KeyboardEvent) {
    this.$emit('updateCaption', this.labelArea.innerText)
  }
}
</script>

<style scoped>
  .editText {
    display: block;
    text-align: inherit;
    background: inherit;
    font: inherit;
    border: none;
    outline: none;
    padding: 0;
    resize: none;
    overflow: hidden;
    text-decoration: inherit;
    color: inherit;
    white-space: inherit;
    word-break: inherit;
  }
</style>
