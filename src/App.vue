<template>
  <div class="flex flex-col p-4 gap-10">
    <div class="flex flex-col">
      <div class="flex">
        <h3 class="py-4 text-2xl text-orange-600 title">欢迎你, 召唤师</h3>
      </div>
      <div class="flex justify-start gap-4">
        <div>
          <div class="flex justify-start items-center gap-4">
            <div>时间</div>
            <div><input type="time" class="border rounded p-2 w-[200px]" v-model="date" @change="handleDateChange">
            </div>
          </div>
          <div class="flex justify-start items-center gap-4 pt-4">
            <div>名称</div>
            <div><input type="text" class="border rounded p-2 w-[200px]" v-model="name" @change="handleNameChange">
            </div>
          </div>
        </div>
        <div class="flex justify-start items-center gap-4">
          <div>头像</div>
          <img v-if="avatar" :src="avatar" class="w-[100px] h-[100px]"></img>
          <div>
            <div class="flex justify-start items-center">
              <label for="avatar">选择头像</label>
              <input type="file" id="avatar" name="file" accept="image/*" class="hidden" @change="handleAvatarChange"/>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="border-t-2 border-orange-600">
      <div class="flex justify-self-start items-center gap-4">
        <h3 class="py-4 text-2xl text-orange-600 title">素材</h3>
        <div class="flex justify-start items-center underline hover:text-orange-600">
          <label for="template" class="cursor-pointer">👉🏻选择图片👈🏻</label>
          <input
              type="file"
              id="template"
              name="template"
              accept="image/*"
              class="hidden"
              multiple
              @change="handleImageChange"
          />
        </div>
        <div>已选择：{{ images.length }}</div>
      </div>
      <div class="flex justify-start flex-wrap gap-2">
        <div v-for="(i, idx) in images" class="border border-orange-400">
          <p class="text-2xl text-red-600 text-center">{{ i.name }}</p>
          <img :src="i.url" alt="" class="w-[100px] h-auto">
        </div>
      </div>
    </div>

    <div class="border-t-2 border-orange-600">
      <div class="flex justify-self-start items-center gap-6">
        <h3 class="py-4 text-2xl text-orange-600 title">技能</h3>
        <div @click="handleStartHandle" class="cursor-pointer hover:text-orange-600">⚡️启动魔法</div>
        <div @click="handleDownload" class="cursor-pointer hover:text-orange-600">🤦‍打包下载</div>
        <div class="text-orange-600">进度：{{ results.length }}/{{ images.length }}</div>
      </div>

      <div class="flex justify-start flex-wrap gap-2">
        <div v-for="(i, idx) in results" class="border border-orange-400">
          <p class="text-2xl text-red-600 text-center">{{ i.name }}</p>
          <img :src="i.src" alt="" class="w-[100px] h-auto">
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import JSZip from 'jszip'

const config = {
  time: { x: 0, y: 0, size: 14, color: '#000000' },
  nickname: { x: 185, y: 303, size: 58, color: '#5b6a91' },
  avatar: { x: 36, y: 303, size: 120, radius: 6 },
}
const name = ref(null)
const avatar = ref(null)
const date = ref(null)
const images = ref([])
const results = ref([])

const handleNameChange = () => {
  if (name.value) {
    window.localStorage.setItem('name', name.value)
  }
}
const handleDateChange = () => {
  if (date.value) {
    window.localStorage.setItem('date', date.value)
  }
}
const handleAvatarChange = (evt) => {
  const files = evt.target.files || evt.dataTransfer.files
  if (!files.length) return

  const reader = new FileReader()

  reader.onload = (e) => {
    avatar.value = e.target.result
    window.localStorage.setItem('avatar', avatar.value)
  }

  reader.readAsDataURL(files[0])
}
const handleImageChange = (evt) => {
  const files = evt.target.files || evt.dataTransfer.files
  if (!files.length) return

  Array.from(files).forEach(file => {
    images.value.push({ name: file.name, url: URL.createObjectURL(file) })
  })
}
const handleStartHandle = () => {
  images.value.forEach(async item => {
    const blob = await generateImage(item.url)
    results.value.push({
      blob: blob, src: URL.createObjectURL(blob), name: item.name,
    })
  })
}
const handleDownload = () => {
  const zip = new JSZip()
  results.value.forEach(r => {
    zip.file(r.name, r.blob)
  })

  zip.generateAsync({ type: 'blob' }).then((content) => {
    const link = document.createElement('a')
    link.href = URL.createObjectURL(content)
    link.download = 'magic.zip'
    link.click()
  })
}
const generateImage = async (img) => {
  const basemap = new Image()

  await new Promise(resolve => {
    basemap.onload = resolve
    basemap.src = img
  })

  const canvas = new OffscreenCanvas(basemap.width, basemap.height)
  const ctx = canvas.getContext('2d')

  // 底图
  ctx.drawImage(basemap, 0, 0, basemap.width, basemap.height)

  // 擦除头像
  ctx.fillStyle = '#fefefe'
  ctx.fillRect(config.avatar.x, config.avatar.y, config.avatar.size, config.avatar.size)
  // 擦除名字
  ctx.fillStyle = '#fefefe'
  ctx.fillRect(config.nickname.x, config.nickname.y, 600, 80)
  // 擦除时间

  // 昵称
  ctx.fillStyle = config.nickname.color
  ctx.font = `bold ${ config.nickname.size }px PingFangSC`
  ctx.textBaseline = 'middle'
  ctx.textAlign = 'left'
  ctx.fillText(name.value, config.nickname.x, config.nickname.y + 28)

  // 头像
  const avatarImage = new Image()
  await new Promise(resolve => {
    avatarImage.onload = resolve
    avatarImage.src = avatar.value
  })
  const x = config.avatar.x
  const y = config.avatar.y
  const imgWidth = config.avatar.size
  const imgHeight = config.avatar.size
  const cornerRadius = config.avatar.radius
  // 圆角
  ctx.beginPath()
  ctx.moveTo(x + cornerRadius, y)
  ctx.lineTo(x + imgWidth - cornerRadius, y)
  ctx.arcTo(x + imgWidth, y, x + imgWidth, y + imgHeight, cornerRadius)
  ctx.lineTo(x + imgWidth, y + imgHeight - cornerRadius)
  ctx.arcTo(x + imgWidth, y + imgHeight, x + imgWidth - cornerRadius, y + imgHeight, cornerRadius)
  ctx.lineTo(x + cornerRadius, y + imgHeight)
  ctx.arcTo(x, y + imgHeight, x, y + imgHeight - cornerRadius, cornerRadius)
  ctx.lineTo(x, y + cornerRadius)
  ctx.arcTo(x, y, x + cornerRadius, y, cornerRadius)
  ctx.closePath()
  ctx.clip()
  ctx.drawImage(avatarImage, config.avatar.x, config.avatar.y, config.avatar.size, config.avatar.size)

  return await canvas.convertToBlob()
}

onMounted(() => {
  name.value = window.localStorage.getItem('name') || null
  avatar.value = window.localStorage.getItem('avatar') || null
  date.value = window.localStorage.getItem('date') || null

  const PingFangSC = new FontFace('PingFangSC', `url(/fonts/PingFangSC-Regular.woff2)`)
  const SfPro = new FontFace('SFPro', `url(/fonts/sf-pro-display-regular.woff2)`)

  Promise.all([PingFangSC.load(), SfPro.load()]).then((fonts) => {
    fonts.forEach(font => document.fonts.add(font))
    console.log('font success!')
  })
})
</script>

<style scoped lang="scss">
.title {
  background-image: linear-gradient(90deg, #116AF0, #EB0050);
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>
