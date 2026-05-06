<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const devices = {
  iphone: {
    label: 'iPhone',
    previewWidth: 320,
    ratioWidth: 9,
    ratioHeight: 19.5,
    padding: 18,
    frameRadius: 52,
    screenRadius: 38,
    notch: true,
    cameraHole: false,
  },
  android: {
    label: 'Téléphone Android',
    previewWidth: 320,
    ratioWidth: 9,
    ratioHeight: 20,
    padding: 14,
    frameRadius: 38,
    screenRadius: 28,
    notch: false,
    cameraHole: true,
  },
}

const fallbackAssetScreen = {
  x: 0.05,
  y: 0.022,
  width: 0.9,
  height: 0.956,
}

const selectedDeviceKey = ref('iphone')
const frameColor = ref('#1f2937')
const fitMode = ref('contain')
const imageUrl = ref('')
const imageName = ref('')
const imageFile = ref(null)

const frameImageUrl = ref('')
const frameImageName = ref('')
const frameImageObjectUrl = ref('')
const frameImageMeta = ref(null)

const currentDevice = computed(() => devices[selectedDeviceKey.value])
const useIphoneAssetFrame = computed(
  () => selectedDeviceKey.value === 'iphone' && Boolean(frameImageUrl.value && frameImageMeta.value),
)
const isOpaqueAssetFrame = computed(
  () => useIphoneAssetFrame.value && frameImageMeta.value?.screenMode === 'dark',
)

const phoneStyle = computed(() => ({
  width: `${currentDevice.value.previewWidth}px`,
  aspectRatio: `${currentDevice.value.ratioWidth} / ${currentDevice.value.ratioHeight}`,
  '--frame-color': frameColor.value,
  '--frame-padding': `${currentDevice.value.padding}px`,
  '--frame-radius': `${currentDevice.value.frameRadius}px`,
}))

const screenStyle = computed(() => ({
  borderRadius: `${currentDevice.value.screenRadius}px`,
}))

const assetPhoneStyle = computed(() => {
  if (!frameImageMeta.value) return {}
  return {
    width: `${currentDevice.value.previewWidth}px`,
    aspectRatio: `${frameImageMeta.value.width} / ${frameImageMeta.value.height}`,
  }
})

const assetScreenStyle = computed(() => {
  const screen = frameImageMeta.value?.screen ?? fallbackAssetScreen
  return {
    left: `${screen.x * 100}%`,
    top: `${screen.y * 100}%`,
    width: `${screen.width * 100}%`,
    height: `${screen.height * 100}%`,
    borderRadius: '5%',
  }
})

function revokeImageUrl() {
  if (!imageUrl.value) return
  URL.revokeObjectURL(imageUrl.value)
}

function revokeFrameObjectUrl() {
  if (!frameImageObjectUrl.value) return
  URL.revokeObjectURL(frameImageObjectUrl.value)
  frameImageObjectUrl.value = ''
}

function getBaseFileName(fileName) {
  return fileName.replace(/\.[^.]+$/, '') || 'image'
}

function drawRoundedRect(ctx, x, y, width, height, radius) {
  const r = Math.min(radius, width / 2, height / 2)

  ctx.beginPath()
  ctx.moveTo(x + r, y)
  ctx.lineTo(x + width - r, y)
  ctx.quadraticCurveTo(x + width, y, x + width, y + r)
  ctx.lineTo(x + width, y + height - r)
  ctx.quadraticCurveTo(x + width, y + height, x + width - r, y + height)
  ctx.lineTo(x + r, y + height)
  ctx.quadraticCurveTo(x, y + height, x, y + height - r)
  ctx.lineTo(x, y + r)
  ctx.quadraticCurveTo(x, y, x + r, y)
  ctx.closePath()
}

function getImagePlacement(imageWidth, imageHeight, boxWidth, boxHeight, mode) {
  const scale =
    mode === 'cover'
      ? Math.max(boxWidth / imageWidth, boxHeight / imageHeight)
      : Math.min(boxWidth / imageWidth, boxHeight / imageHeight)

  const width = imageWidth * scale
  const height = imageHeight * scale

  return {
    x: (boxWidth - width) / 2,
    y: (boxHeight - height) / 2,
    width,
    height,
  }
}

function loadImage(url) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = () => reject(new Error("L'image n'a pas pu être chargée."))
    img.src = url
  })
}

function downloadFromHref(href, fileName) {
  const link = document.createElement('a')
  link.href = href
  link.download = fileName
  link.click()
}

function findMatchingPixel(width, height, predicate) {
  const centerX = Math.floor(width / 2)
  const centerY = Math.floor(height / 2)

  if (predicate(centerX, centerY)) {
    return { x: centerX, y: centerY }
  }

  const maxRadius = Math.min(Math.floor(Math.min(width, height) / 3), 500)
  for (let radius = 1; radius <= maxRadius; radius += 1) {
    const minX = Math.max(0, centerX - radius)
    const maxX = Math.min(width - 1, centerX + radius)
    const minY = Math.max(0, centerY - radius)
    const maxY = Math.min(height - 1, centerY + radius)

    for (let x = minX; x <= maxX; x += 1) {
      if (predicate(x, minY)) return { x, y: minY }
      if (predicate(x, maxY)) return { x, y: maxY }
    }

    for (let y = minY + 1; y < maxY; y += 1) {
      if (predicate(minX, y)) return { x: minX, y }
      if (predicate(maxX, y)) return { x: maxX, y }
    }
  }

  return null
}

function floodFillBounds(width, height, startX, startY, predicate) {
  const stack = [startY * width + startX]
  const visited = new Uint8Array(width * height)

  let minX = startX
  let maxX = startX
  let minY = startY
  let maxY = startY
  let pixels = 0

  while (stack.length) {
    const index = stack.pop()
    if (visited[index]) continue
    visited[index] = 1

    const x = index % width
    const y = (index - x) / width
    if (!predicate(x, y)) continue

    pixels += 1
    if (x < minX) minX = x
    if (x > maxX) maxX = x
    if (y < minY) minY = y
    if (y > maxY) maxY = y

    if (x > 0) stack.push(index - 1)
    if (x < width - 1) stack.push(index + 1)
    if (y > 0) stack.push(index - width)
    if (y < height - 1) stack.push(index + width)
  }

  return {
    minX,
    minY,
    maxX,
    maxY,
    width: maxX - minX + 1,
    height: maxY - minY + 1,
    pixels,
  }
}

function detectFrameScreen(frameImage) {
  const canvas = document.createElement('canvas')
  const width = frameImage.naturalWidth || frameImage.width
  const height = frameImage.naturalHeight || frameImage.height

  canvas.width = width
  canvas.height = height
  const ctx = canvas.getContext('2d')

  if (!ctx) {
    return { screen: fallbackAssetScreen, screenMode: 'transparent' }
  }

  ctx.drawImage(frameImage, 0, 0, width, height)
  const imageData = ctx.getImageData(0, 0, width, height).data

  const alphaAt = (x, y) => imageData[(y * width + x) * 4 + 3]
  const luminanceAt = (x, y) => {
    const offset = (y * width + x) * 4
    const red = imageData[offset]
    const green = imageData[offset + 1]
    const blue = imageData[offset + 2]
    return 0.2126 * red + 0.7152 * green + 0.0722 * blue
  }

  const centerX = Math.floor(width / 2)
  const centerY = Math.floor(height / 2)
  const screenMode = alphaAt(centerX, centerY) < 20 ? 'transparent' : 'dark'

  const isScreenPixel =
    screenMode === 'transparent'
      ? (x, y) => alphaAt(x, y) < 20
      : (x, y) => alphaAt(x, y) > 200 && luminanceAt(x, y) < 24

  const start = findMatchingPixel(width, height, isScreenPixel)
  if (!start) {
    return { screen: fallbackAssetScreen, screenMode }
  }

  const bounds = floodFillBounds(width, height, start.x, start.y, isScreenPixel)
  const isValidBounds =
    bounds.pixels > 0 &&
    bounds.width >= Math.round(width * 0.3) &&
    bounds.height >= Math.round(height * 0.5)

  if (!isValidBounds) {
    return { screen: fallbackAssetScreen, screenMode }
  }

  return {
    screen: {
      x: bounds.minX / width,
      y: bounds.minY / height,
      width: bounds.width / width,
      height: bounds.height / height,
    },
    screenMode,
  }
}

async function setIphoneFrame(url, fileName) {
  const frameImage = await loadImage(url)
  const width = frameImage.naturalWidth || frameImage.width
  const height = frameImage.naturalHeight || frameImage.height
  const { screen, screenMode } = detectFrameScreen(frameImage)

  frameImageUrl.value = url
  frameImageName.value = fileName
  frameImageMeta.value = {
    width,
    height,
    screen,
    screenMode,
  }
}

async function loadDefaultFrameAsset() {
  try {
    await setIphoneFrame('/assets/iphone-15-frame.png', 'iphone-15-frame.png')
  } catch (error) {
    console.info("Asset iPhone par défaut introuvable dans /public/assets.", error)
  }
}

function onFileChange(event) {
  const [file] = event.target.files ?? []
  if (!file || !file.type.startsWith('image/')) return

  revokeImageUrl()
  imageFile.value = file
  imageUrl.value = URL.createObjectURL(file)
  imageName.value = file.name
}

async function onFrameFileChange(event) {
  const [file] = event.target.files ?? []
  if (!file || !file.type.startsWith('image/')) return

  revokeFrameObjectUrl()
  const objectUrl = URL.createObjectURL(file)
  frameImageObjectUrl.value = objectUrl
  await setIphoneFrame(objectUrl, file.name)
}

async function clearCustomFrameAsset() {
  revokeFrameObjectUrl()
  frameImageUrl.value = ''
  frameImageName.value = ''
  frameImageMeta.value = null
  await loadDefaultFrameAsset()
}

function clearImage() {
  revokeImageUrl()
  imageUrl.value = ''
  imageName.value = ''
  imageFile.value = null
}

function drawIphoneNotch(ctx, phoneWidth, scale, color) {
  const notchWidth = phoneWidth * 0.38
  const notchHeight = 22 * scale
  const notchX = (phoneWidth - notchWidth) / 2
  const notchY = 7 * scale
  const notchRadius = 14 * scale

  ctx.beginPath()
  ctx.moveTo(notchX, notchY)
  ctx.lineTo(notchX + notchWidth, notchY)
  ctx.lineTo(notchX + notchWidth, notchY + notchHeight - notchRadius)
  ctx.quadraticCurveTo(
    notchX + notchWidth,
    notchY + notchHeight,
    notchX + notchWidth - notchRadius,
    notchY + notchHeight,
  )
  ctx.lineTo(notchX + notchRadius, notchY + notchHeight)
  ctx.quadraticCurveTo(notchX, notchY + notchHeight, notchX, notchY + notchHeight - notchRadius)
  ctx.closePath()
  ctx.fillStyle = color
  ctx.fill()
}

function drawAndroidCameraHole(ctx, phoneWidth, scale) {
  const centerX = phoneWidth / 2
  const centerY = 15 * scale
  const radius = 5 * scale

  ctx.beginPath()
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
  ctx.fillStyle = '#0b1220'
  ctx.fill()
  ctx.lineWidth = Math.max(2 * scale, 2)
  ctx.strokeStyle = 'rgba(15, 23, 42, 0.8)'
  ctx.stroke()
}

async function downloadAssetBasedMockup() {
  if (!frameImageUrl.value || !frameImageMeta.value) return

  const [sourceImage, frameImage] = await Promise.all([loadImage(imageUrl.value), loadImage(frameImageUrl.value)])
  const canvas = document.createElement('canvas')
  canvas.width = frameImage.naturalWidth || frameImage.width
  canvas.height = frameImage.naturalHeight || frameImage.height

  const ctx = canvas.getContext('2d')
  if (!ctx) {
    throw new Error("Le contexte de rendu n'a pas pu être créé.")
  }

  const screen = frameImageMeta.value.screen ?? fallbackAssetScreen
  const screenX = Math.round(screen.x * canvas.width)
  const screenY = Math.round(screen.y * canvas.height)
  const screenWidth = Math.round(screen.width * canvas.width)
  const screenHeight = Math.round(screen.height * canvas.height)
  const screenRadius = screenWidth * 0.05

  if (isOpaqueAssetFrame.value) {
    ctx.drawImage(frameImage, 0, 0, canvas.width, canvas.height)
  }

  drawRoundedRect(ctx, screenX, screenY, screenWidth, screenHeight, screenRadius)
  ctx.save()
  ctx.clip()

  ctx.fillStyle = '#000'
  ctx.fillRect(screenX, screenY, screenWidth, screenHeight)

  const placement = getImagePlacement(
    sourceImage.width,
    sourceImage.height,
    screenWidth,
    screenHeight,
    fitMode.value,
  )
  ctx.drawImage(
    sourceImage,
    screenX + placement.x,
    screenY + placement.y,
    placement.width,
    placement.height,
  )
  ctx.restore()

  if (!isOpaqueAssetFrame.value) {
    ctx.drawImage(frameImage, 0, 0, canvas.width, canvas.height)
  }

  const fileName = `${getBaseFileName(imageName.value)}-iphone-mockup.png`
  downloadFromHref(canvas.toDataURL('image/png'), fileName)
}

async function downloadGeneratedMockup() {
  const sourceImage = await loadImage(imageUrl.value)
  const device = currentDevice.value
  const exportWidth = 1080
  const exportHeight = Math.round(exportWidth * (device.ratioHeight / device.ratioWidth))
  const scale = exportWidth / device.previewWidth
  const framePadding = device.padding * scale
  const frameRadius = device.frameRadius * scale
  const screenRadius = device.screenRadius * scale

  const canvas = document.createElement('canvas')
  canvas.width = exportWidth
  canvas.height = exportHeight
  const ctx = canvas.getContext('2d')

  if (!ctx) {
    throw new Error("Le contexte de rendu n'a pas pu être créé.")
  }

  drawRoundedRect(ctx, 0, 0, exportWidth, exportHeight, frameRadius)
  ctx.fillStyle = frameColor.value
  ctx.fill()

  const screenX = framePadding
  const screenY = framePadding
  const screenWidth = exportWidth - framePadding * 2
  const screenHeight = exportHeight - framePadding * 2

  drawRoundedRect(ctx, screenX, screenY, screenWidth, screenHeight, screenRadius)
  ctx.save()
  ctx.clip()

  ctx.fillStyle = '#000'
  ctx.fillRect(screenX, screenY, screenWidth, screenHeight)

  const placement = getImagePlacement(
    sourceImage.width,
    sourceImage.height,
    screenWidth,
    screenHeight,
    fitMode.value,
  )

  ctx.drawImage(
    sourceImage,
    screenX + placement.x,
    screenY + placement.y,
    placement.width,
    placement.height,
  )
  ctx.restore()

  if (device.notch) {
    drawIphoneNotch(ctx, exportWidth, scale, frameColor.value)
  }

  if (device.cameraHole) {
    drawAndroidCameraHole(ctx, exportWidth, scale)
  }

  const fileName = `${getBaseFileName(imageName.value)}-${selectedDeviceKey.value}-mockup.png`
  downloadFromHref(canvas.toDataURL('image/png'), fileName)
}

async function downloadPhoneMockup() {
  if (!imageUrl.value || !imageName.value) return

  if (useIphoneAssetFrame.value) {
    await downloadAssetBasedMockup()
    return
  }

  await downloadGeneratedMockup()
}

function downloadSourceImage() {
  if (!imageFile.value) return
  const objectUrl = URL.createObjectURL(imageFile.value)
  downloadFromHref(objectUrl, imageName.value)
  setTimeout(() => {
    URL.revokeObjectURL(objectUrl)
  }, 0)
}

onMounted(() => {
  loadDefaultFrameAsset()
})

onBeforeUnmount(() => {
  revokeImageUrl()
  revokeFrameObjectUrl()
})
</script>

<template>
  <main class="layout">
    <section class="controls">
      <h1>Générateur de mockup téléphone</h1>
      <p>Importez une image, choisissez le type de téléphone et obtenez un mockup instantané.</p>

      <label class="upload">
        <input type="file" accept="image/*" @change="onFileChange" />
        <span>{{ imageUrl ? imageName : 'Choisir une image' }}</span>
      </label>

      <label class="upload">
        <input type="file" accept="image/*" @change="onFrameFileChange" />
        <span>
          {{ frameImageUrl ? `Asset iPhone: ${frameImageName}` : 'Choisir un asset iPhone (PNG)' }}
        </span>
      </label>

      <p class="hint">
        Placez aussi votre fichier dans <code>public/assets/iphone-15-frame.png</code> pour le charger
        automatiquement.
      </p>

      <label class="field">
        <span>Appareil</span>
        <select v-model="selectedDeviceKey">
          <option v-for="(device, key) in devices" :key="key" :value="key">
            {{ device.label }}
          </option>
        </select>
      </label>

      <label class="field">
        <span>Couleur du cadre</span>
        <input v-model="frameColor" type="color" />
      </label>

      <label class="field">
        <span>Ajustement de l'image</span>
        <select v-model="fitMode">
          <option value="contain">Contenir (contain)</option>
          <option value="cover">Remplir (cover)</option>
        </select>
      </label>

      <div class="actions">
        <button class="action" type="button" @click="downloadPhoneMockup" :disabled="!imageUrl">
          Télécharger le mockup téléphone
        </button>
        <button class="action" type="button" @click="downloadSourceImage" :disabled="!imageUrl">
          Télécharger l'image source
        </button>
      </div>

      <button class="clear" type="button" @click="clearImage" :disabled="!imageUrl">
        Retirer l'image
      </button>

      <button class="clear" type="button" @click="clearCustomFrameAsset" :disabled="!frameImageObjectUrl">
        Retirer l'asset iPhone personnalisé
      </button>
    </section>

    <section class="preview">
      <div v-if="useIphoneAssetFrame" class="asset-phone" :style="assetPhoneStyle">
        <img v-if="isOpaqueAssetFrame" class="asset-frame underlay" :src="frameImageUrl" alt="Cadre iPhone" />

        <div class="asset-screen" :style="assetScreenStyle">
          <img v-if="imageUrl" :src="imageUrl" alt="Mockup" :style="{ objectFit: fitMode }" />
          <div v-else class="placeholder">Ajoutez une image pour générer votre mockup.</div>
        </div>

        <img v-if="!isOpaqueAssetFrame" class="asset-frame overlay" :src="frameImageUrl" alt="Cadre iPhone" />
      </div>

      <div v-else class="phone" :style="phoneStyle">
        <div v-if="currentDevice.notch" class="notch"></div>
        <div v-if="currentDevice.cameraHole" class="camera-hole"></div>

        <div class="screen" :style="screenStyle">
          <img v-if="imageUrl" :src="imageUrl" alt="Mockup" :style="{ objectFit: fitMode }" />
          <div v-else class="placeholder">Ajoutez une image pour générer votre mockup.</div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.layout {
  min-height: 100vh;
  display: grid;
  grid-template-columns: minmax(280px, 380px) 1fr;
  gap: 2rem;
  padding: 2rem;
}

.controls {
  background: #111827;
  border: 1px solid #1f2937;
  border-radius: 16px;
  padding: 1.25rem;
  display: grid;
  gap: 1rem;
  align-content: start;
}

h1 {
  margin: 0;
  font-size: 1.4rem;
}

p {
  margin: 0;
  color: #9ca3af;
  font-size: 0.95rem;
}

.hint {
  font-size: 0.8rem;
  color: #94a3b8;
}

.hint code {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 0.1rem 0.3rem;
}

.upload {
  background: #0b1220;
  border: 1px dashed #334155;
  border-radius: 10px;
  padding: 0.8rem;
  cursor: pointer;
}

.upload input {
  display: none;
}

.upload span {
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.field {
  display: grid;
  gap: 0.4rem;
}

.field span {
  font-size: 0.85rem;
  color: #cbd5e1;
}

select,
input[type='color'],
.clear,
.action {
  width: 100%;
  border: 1px solid #334155;
  background: #0b1220;
  color: #e5e7eb;
  border-radius: 8px;
  padding: 0.6rem 0.7rem;
  font: inherit;
}

input[type='color'] {
  min-height: 42px;
  padding: 0.2rem;
}

.clear {
  cursor: pointer;
}

.actions {
  display: grid;
  gap: 0.6rem;
}

.action {
  cursor: pointer;
}

.clear:disabled,
.action:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.preview {
  display: grid;
  place-items: center;
  min-height: 70vh;
  border-radius: 16px;
  border: 1px solid #1f2937;
  background: radial-gradient(circle at center, #111827 0, #020617 75%);
  padding: 1.5rem;
}

.phone {
  position: relative;
  padding: var(--frame-padding);
  border-radius: var(--frame-radius);
  background: var(--frame-color);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.45);
}

.asset-phone {
  position: relative;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.45);
}

.asset-frame {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
  pointer-events: none;
}

.asset-frame.underlay {
  z-index: 0;
}

.asset-frame.overlay {
  z-index: 2;
}

.asset-screen {
  position: absolute;
  overflow: hidden;
  background: #000;
  z-index: 1;
}

.screen {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #000;
}

.screen img,
.asset-screen img {
  width: 100%;
  height: 100%;
  display: block;
}

.placeholder {
  width: 100%;
  height: 100%;
  display: grid;
  place-items: center;
  text-align: center;
  color: #64748b;
  padding: 1.2rem;
  font-size: 0.92rem;
}

.notch {
  position: absolute;
  top: 7px;
  left: 50%;
  transform: translateX(-50%);
  width: 38%;
  height: 22px;
  background: var(--frame-color);
  border-bottom-left-radius: 14px;
  border-bottom-right-radius: 14px;
  z-index: 1;
}

.camera-hole {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 2px solid rgba(15, 23, 42, 0.8);
  background: #0b1220;
  z-index: 1;
}

@media (max-width: 980px) {
  .layout {
    grid-template-columns: 1fr;
  }

  .preview {
    min-height: auto;
    padding: 2rem 1rem;
  }
}
</style>
