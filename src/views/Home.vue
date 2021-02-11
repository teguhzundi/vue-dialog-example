<template>
  <div>
    <button @click="dialog1 = true">Dialog</button>
    &nbsp;
    <button @click="dialog2 = true">Dialog No Resize + No Drag</button>

    <Dialog v-model="dialog1" title="Dialog" @close="dialog1 = false" :width="800" >
      <div style="padding: 20px;">
        Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.
      </div>
    </Dialog>
     <Dialog v-model="dialog2" title="Dialog" @close="dialog2 = false" :draggable="false" :resizable="false" overlay>
      <div style="padding: 20px;">
        <lorem/>
      </div>
    </Dialog>
  </div>
</template>

<script>
import Dialog from '@/components/Dialog'

export default {
  name: 'HelloWorld',
  components: {
    Dialog,
    Lorem: () => import('@/components/Lorem')
  },
  data: () => ({
    dialog1: false,
    dialog2: false
  }),
  watch: {
    dialog1 (show) {
      console.info(show ? 'Load Data' : 'Remove Data')
    }
  }
}
</script>
