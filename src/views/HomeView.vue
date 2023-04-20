<template>
  <div class="home">
    <div class="content">
      <div class="action-box">
        <button class="btn" @click="save()">Save</button>
        <button>Undo</button>
        <button>Redo</button>
        <button>Export</button>
        <button>Import</button>
        <button>
          <RouterLink to="/about" class="nav-link">View</RouterLink>
        </button>
      </div>

      <div class="wrap-box">
        <div class="left from">
          <div class="list-item">
            <div class="drop-zone" @drop="onDrop($event, 1)" @dragover.prevent @dragenter.prevent>
              <div
                v-for="item in listOne"
                :key="item.id"
                class="item drag-el"
                draggable="true"
                @dragstart="startDrag($event, item)"
              >
                <div class="thumb">{{ item.image }}</div>
                <!-- <div :class="item.component" v-text="getText(item)"></div> -->
                <div :class="item.component" v-if="item.component == 'ElementButton'">Button</div>
                <div :class="item.component" v-else-if="item.component == 'ElementParagraph'">
                  Paragraph
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="right to" @mousemove="updateCoordinates">
          <div class="content-box drag-drop-box">
            <div class="view-info-box">
              <p>
                <span>Mouse:</span><span> {{ xCoordinates }} / {{ yCoordinates }}</span>
              </p>
              <p>
                <span>Dragging:</span><span>{{ elementOnStarts }}</span>
              </p>
              <p>
                <span>Instances:</span><span>{{ listTwo.length }}</span>
              </p>
              <p>
                <span>Config:</span><span>{{ configContent }}</span>
              </p>
            </div>

            <div class="drop-zone" @drop="onDrop($event, 2)" @dragover.prevent @dragenter.prevent>
              <div
                class="drag-el"
                v-for="item in listTwo"
                :key="item.id"
                draggable="true"
                @dragstart="startDrag($event, item)"
              >
                <!-- <div 
                  v-on:click="changeContent(item)"
                  :class="item.component" v-if="item.component == 'ElementButton'">Button</div>
                <div 
                  v-on:click="changeContent(item)"
                  :class="item.component" v-else-if="item.component == 'ElementParagraph'">Paragraph
                </div> -->
                <div
                  v-on:click="changeContent(item)"
                  v-text="getText(item)"
                  :class="item.component"
                ></div>
              </div>
            </div>
          </div>
          <div class="content-box">
            <div class="paragraph-input" v-if="isShowParagraphContent">
              <div class="text-input">
                <h3>Paragraph Text</h3>
                <input type="text" v-model="paragraphText" :id="paragraphId" />
              </div>
            </div>
          </div>
          <div class="content-box">
            <div class="button-input" v-if="isShowButtonContent">
              <div class="text-input">
                <h3>Button Text</h3>
                <input type="text" v-model="buttonText" :id="buttonId"/>
              </div>
              <div class="message-input">
                <h3>Alert Message</h3>
                <input type="text" v-model="messageText" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { Guid } from 'guid-typescript'

const listOne = ref<Array<any>>([
  {
    id: 0,
    image: null,
    //text: 'Paragraph',
    component: 'ElementParagraph',
    props: {},
    list: 1
  },
  {
    id: 1,
    image: null,
    //text: 'Button',
    component: 'ElementButton',
    props: {},
    list: 1
  }
])

const xCoordinates = ref(0)
const yCoordinates = ref(0)

const listTwo = ref<Array<any>>([])
const paragraphText = ref('')
const buttonText = ref('')
const messageText = ref('')

const isShowButtonContent = ref<boolean>(false)
const isShowParagraphContent = ref<boolean>(false)

const configContent = ref<string>('') as any

const updateCoordinates = (event: MouseEvent) => {
  xCoordinates.value = event.clientX
  yCoordinates.value = event.clientY
}
const paragraphId = ref('')
const buttonId = ref('')
const changeContent = (content: any) => {
  configContent.value = content
  paragraphId.value = content.id
  listTwo.value.forEach((element) => {
    if (paragraphId.value == element.id) {
      paragraphText.value = element.text
    }
    if (buttonId.value == element.id) {
      buttonId.value = element.text
    }
  })

  isShowButtonContent.value = content?.component === 'ElementButton'
  isShowParagraphContent.value = content?.component === 'ElementParagraph'
}
const getText = (item: any) => {
  if (item.props.text) {
    return item.props.text
  }
  else if(item?.component === 'ElementButton') {
    return 'Button'
  } else if(item?.component === 'ElementParagraph'){
    return 'Paragraph'
  }
}

const save = () => {
  // localStorage.dataTam = listTwo.value;
  localStorage.dataTam = JSON.stringify(listTwo.value)
}
watch(paragraphText, (value) => {
  configContent.value.props.text = value
  listTwo.value.forEach((element) => {
    if (element.component == 'ElementParagraph' && paragraphId.value == element.id) {
      element.text = value
    }
    if (element.component == 'ElementButton' && buttonId.value == element.id) {
      element.text = value
    }
  })
})
watch(buttonText, (value) => {
  configContent.value.props.text = value
  listTwo.value.forEach((element) => {
    if (element.component == 'ElementButton') {
      element.text = value
    }
  })
})
watch(messageText, (value) => {
  configContent.value.props.message = value
})

const elementOnStarts = ref<string>('')
const startDrag = (evt: any, item: any) => {
  evt.dataTransfer.dropEffect = 'copy'
  evt.dataTransfer.effectAllowed = 'copy'
  evt.dataTransfer.setData('itemID', item.id)
  evt.dataTransfer.setData('itemList', item.list)
  elementOnStarts.value = item.component
  configContent.value = ''
}

const onDrop = (evt: any, list: any) => {
  configContent.value = ''
  elementOnStarts.value = ''
  const itemID = evt.dataTransfer.getData('itemID')
  const item = listOne.value.find((item: any) => item.id == itemID)

  if (item?.list == list) {
    return
  }

  const newGuild = Guid.create()
  listTwo.value.push({
    id: newGuild.toString(),
    //text: item.text,
    component: item.component,
    props: {}
  })
}
</script>

<style scoped>
.drop-zone {
  margin-bottom: 10px;
  padding: 10px;
  text-align: center;
}

.drag-el {
  margin-bottom: 10px;
  padding: 5px;
}
</style>
