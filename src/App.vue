<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'

/* ---------------------------------- data ---------------------------------- */

const devices = {
  'iphone-pro': {
    label: 'iPhone 15 Pro',
    sub: 'Dynamic Island',
    kind: 'phone',
    previewWidth: 300,
    ratioWidth: 9,
    ratioHeight: 19.5,
    padding: 11,
    frameRadius: 54,
    screenRadius: 43,
    style: 'island',
  },
  'iphone-classic': {
    label: 'iPhone 14',
    sub: 'Encoche',
    kind: 'phone',
    previewWidth: 300,
    ratioWidth: 9,
    ratioHeight: 19.5,
    padding: 13,
    frameRadius: 50,
    screenRadius: 38,
    style: 'notch',
  },
  pixel: {
    label: 'Pixel 8',
    sub: 'Poinçon caméra',
    kind: 'phone',
    previewWidth: 300,
    ratioWidth: 9,
    ratioHeight: 20,
    padding: 12,
    frameRadius: 42,
    screenRadius: 32,
    style: 'hole',
  },
  tablet: {
    label: 'Tablette',
    sub: '10.9" · 3:4',
    kind: 'tablet',
    previewWidth: 330,
    ratioWidth: 3,
    ratioHeight: 4,
    padding: 18,
    frameRadius: 38,
    screenRadius: 22,
    style: 'none',
    bezelCam: true,
  },
  laptop: {
    label: 'MacBook',
    sub: '14" · 16:10',
    kind: 'laptop',
    previewWidth: 470,
    ratioWidth: 16,
    ratioHeight: 10,
    padding: 12,
    frameRadius: 18,
    screenRadius: 10,
    style: 'none',
    bezelCam: true,
  },
  monitor: {
    label: 'iMac',
    sub: '24" · 16:9',
    kind: 'monitor',
    previewWidth: 520,
    ratioWidth: 16,
    ratioHeight: 9,
    padding: 14,
    frameRadius: 20,
    screenRadius: 12,
    style: 'none',
    bezelCam: true,
  },
}

const finishes = [
  { id: 'noir', label: 'Titane noir', base: '#23262c', edge: '#6b7280', dark: '#08090b', btn: '#101216' },
  { id: 'naturel', label: 'Titane naturel', base: '#c9c9c6', edge: '#f4f4f2', dark: '#8f8f8c', btn: '#b5b5b2' },
  { id: 'blanc', label: 'Blanc étoilé', base: '#e9ebef', edge: '#ffffff', dark: '#a8adb7', btn: '#d4d8df' },
  { id: 'bleu', label: 'Bleu titane', base: '#3d4c68', edge: '#8fa3c7', dark: '#141c2e', btn: '#232f4a' },
  { id: 'rose', label: 'Rose', base: '#e8b4b8', edge: '#ffe3e6', dark: '#9c6b70', btn: '#d69aa0' },
]

const scenes = [
  {
    id: 'midnight', label: 'Studio nuit',
    bg: 'radial-gradient(120% 100% at 50% 0%, #1b2540 0%, #0a0f22 55%, #04060f 100%)',
    orb1: '#6366f1', orb2: '#06b6d4', dark: true,
    from: '#1b2540', to: '#04060f',
  },
  {
    id: 'light', label: 'Studio clair',
    bg: 'radial-gradient(120% 100% at 50% 0%, #ffffff 0%, #e2e8f0 55%, #cbd5e1 100%)',
    orb1: '#c7d2fe', orb2: '#bae6fd', dark: false,
    from: '#f8fafc', to: '#cbd5e1',
  },
  {
    id: 'sunset', label: 'Sunset',
    bg: 'linear-gradient(135deg, #ff9a8b 0%, #ff6a88 45%, #7b2ff7 100%)',
    orb1: '#ffd166', orb2: '#ff4d6d', dark: true,
    from: '#ff9a8b', to: '#7b2ff7',
  },
  {
    id: 'ocean', label: 'Océan',
    bg: 'linear-gradient(135deg, #43cea2 0%, #185a9d 100%)',
    orb1: '#5eead4', orb2: '#3b82f6', dark: true,
    from: '#43cea2', to: '#185a9d',
  },
  {
    id: 'grape', label: 'Raisin',
    bg: 'linear-gradient(135deg, #231942 0%, #5e548e 50%, #9f86c0 100%)',
    orb1: '#c084fc', orb2: '#f472b6', dark: true,
    from: '#231942', to: '#9f86c0',
  },
  {
    id: 'peach', label: 'Papier',
    bg: 'linear-gradient(135deg, #fdfbfb 0%, #ebedee 100%)',
    orb1: '#fed7aa', orb2: '#fecdd3', dark: false,
    from: '#fdfbfb', to: '#d7dde3',
  },
  {
    id: 'carbon', label: 'Carbone',
    bg: 'repeating-linear-gradient(0deg, #0b0e14 0 2px, #10141d 2px 4px), radial-gradient(80% 60% at 50% 20%, #1e293b 0%, transparent 70%)',
    orb1: '#334155', orb2: '#0ea5e9', dark: true,
    from: '#141a26', to: '#05070c',
  },
  {
    id: 'transparent', label: 'Transparent (PNG alpha)',
    bg: 'repeating-conic-gradient(#d7dee9 0% 25%, #f4f6fa 0% 50%) 0 0 / 26px 26px',
    orb1: null, orb2: null, dark: false,
    from: null, to: null,
  },
  {
    id: 'custom', label: 'Uni',
    bg: null, orb1: null, orb2: null, dark: true,
    from: '#0ea5e9', to: '#6366f1',
  },
]

const fallbackAssetScreen = { x: 0.05, y: 0.022, width: 0.9, height: 0.956 }

/* ---------------------------------- state --------------------------------- */

const selectedDeviceKey = ref('iphone-pro')
const selectedFinishId = ref('noir')
const customFrameColor = ref('#23262c')
const selectedSceneId = ref('midnight')
const customBg = ref('#4f46e5')

const fitMode = ref('cover')
const imageUrl = ref('')
const imageName = ref('')
const imageFile = ref(null)
const isDragging = ref(false)

const scale = ref(1)
const tiltX = ref(-8)
const tiltY = ref(15)
const rotation = ref(-3)
const zoom = ref(1)
const shiftY = ref(0)
const shadowStrength = ref(55)
const showGlare = ref(true)
const floatAnim = ref(true)
const mouseParallax = ref(true)
const parallax = ref({ x: 0, y: 0 })

const isExporting = ref(false)
const exportFormat = ref('png')
const webpQuality = ref(0.92)

const frameImageUrl = ref('')
const frameImageName = ref('')
const frameImageObjectUrl = ref('')
const frameImageMeta = ref(null)

/* -------------------------------- computed ------------------------------- */

const currentDevice = computed(() => devices[selectedDeviceKey.value])
const currentFinish = computed(
  () => finishes.find((f) => f.id === selectedFinishId.value) ?? finishes[0],
)
const currentScene = computed(
  () => scenes.find((s) => s.id === selectedSceneId.value) ?? scenes[0],
)
const isCustomFinish = computed(() => selectedFinishId.value === 'custom')
const frameBase = computed(() => (isCustomFinish.value ? customFrameColor.value : currentFinish.value.base))
const frameEdge = computed(() => (isCustomFinish.value ? lighten(customFrameColor.value, 45) : currentFinish.value.edge))
const frameDark = computed(() => (isCustomFinish.value ? darken(customFrameColor.value, 45) : currentFinish.value.dark))

const useIphoneAssetFrame = computed(
  () => selectedDeviceKey.value === 'iphone-pro' && Boolean(frameImageUrl.value && frameImageMeta.value),
)
const isOpaqueAssetFrame = computed(
  () => useIphoneAssetFrame.value && frameImageMeta.value?.screenMode === 'dark',
)

const previewBg = computed(() => {
  if (currentScene.value.id === 'custom') return customBg.value
  return currentScene.value.bg
})

const stageEl = ref(null)
const fitFactor = ref(1)

function displaySize() {
  const d = currentDevice.value
  const w = d.previewWidth * scale.value
  if (d.kind === 'laptop') return { w: w * 1.1, h: (w * d.ratioHeight) / d.ratioWidth + 72 }
  if (d.kind === 'monitor') return { w, h: (w * d.ratioHeight) / d.ratioWidth + 140 }
  return { w, h: (w * d.ratioHeight) / d.ratioWidth + 90 }
}

function updateFit() {
  const el = stageEl.value
  if (!el) return
  const size = displaySize()
  const availW = el.clientWidth - 70
  const availH = el.clientHeight - 120
  if (size.w <= 0 || size.h <= 0) return
  // marge pour le tilt 3D qui élargit l'appareil
  fitFactor.value = Math.min(1, Math.max(0.35, availW / (size.w * 1.15), availH / size.h))
}

const phoneStyle = computed(() => ({
  width: `${currentDevice.value.previewWidth * scale.value * fitFactor.value}px`,
  aspectRatio: `${currentDevice.value.ratioWidth} / ${currentDevice.value.ratioHeight}`,
  '--frame-base': frameBase.value,
  '--frame-edge': frameEdge.value,
  '--frame-dark': frameDark.value,
  '--frame-padding': `${currentDevice.value.padding}px`,
  '--frame-radius': `${currentDevice.value.frameRadius}px`,
  '--screen-radius': `${currentDevice.value.screenRadius}px`,
  '--shadow': `${shadowStrength.value}`,
}))

const wideStyle = computed(() => ({
  width: `${currentDevice.value.previewWidth * scale.value * fitFactor.value}px`,
  '--frame-base': frameBase.value,
  '--frame-edge': frameEdge.value,
  '--frame-dark': frameDark.value,
  '--shadow': `${shadowStrength.value}`,
}))

const tiltTransform = computed(() => {
  const px = mouseParallax.value ? parallax.value.x * 10 : 0
  const py = mouseParallax.value ? parallax.value.y * -10 : 0
  return `rotateX(${tiltX.value + py}deg) rotateY(${tiltY.value + px}deg) rotateZ(${rotation.value}deg)`
})

const imgStyle = computed(() => ({
  objectFit: fitMode.value,
  transform:
    zoom.value !== 1 || shiftY.value !== 0
      ? `scale(${zoom.value}) translateY(${shiftY.value}%)`
      : undefined,
}))

const assetPhoneStyle = computed(() => {
  if (!frameImageMeta.value) return {}
  return {
    width: `${currentDevice.value.previewWidth * scale.value * fitFactor.value}px`,
    aspectRatio: `${frameImageMeta.value.width} / ${frameImageMeta.value.height}`,
    transform: tiltTransform.value,
  }
})

const assetScreenStyle = computed(() => {
  const screen = frameImageMeta.value?.screen ?? fallbackAssetScreen
  return {
    left: `${screen.x * 100}%`,
    top: `${screen.y * 100}%`,
    width: `${screen.width * 100}%`,
    height: `${screen.height * 100}%`,
  }
})

const sceneIsDark = computed(() => {
  if (currentScene.value.id === 'transparent') return false
  if (currentScene.value.id === 'custom') return luminance(customBg.value) < 0.4
  return currentScene.value.dark
})

const showOrbs = computed(
  () => currentScene.value.id !== 'custom' && currentScene.value.id !== 'transparent',
)

const exportLabel = computed(() =>
  exportFormat.value === 'both' ? 'PNG + WebP' : exportFormat.value === 'webp' ? 'WebP' : 'PNG',
)

/* --------------------------------- helpers -------------------------------- */

function lighten(hex, amt) {
  const n = normalizeHex(hex)
  const r = Math.min(255, n.r + amt)
  const g = Math.min(255, n.g + amt)
  const b = Math.min(255, n.b + amt)
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`
}
function darken(hex, amt) {
  const n = normalizeHex(hex)
  const r = Math.max(0, n.r - amt)
  const g = Math.max(0, n.g - amt)
  const b = Math.max(0, n.b - amt)
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`
}
function normalizeHex(hex) {
  let h = hex.replace('#', '')
  if (h.length === 3) h = h.split('').map((c) => c + c).join('')
  return { r: parseInt(h.slice(0, 2), 16), g: parseInt(h.slice(2, 4), 16), b: parseInt(h.slice(4, 6), 16) }
}
function toHex(v) {
  return Math.round(v).toString(16).padStart(2, '0')
}
function luminance(hex) {
  try {
    const { r, g, b } = normalizeHex(hex)
    return (0.2126 * r + 0.7152 * g + 0.0722 * b) / 255
  } catch { return 0 }
}

function revokeImageUrl() {
  if (imageUrl.value.startsWith('blob:')) URL.revokeObjectURL(imageUrl.value)
}
function revokeFrameObjectUrl() {
  if (frameImageObjectUrl.value) {
    URL.revokeObjectURL(frameImageObjectUrl.value)
    frameImageObjectUrl.value = ''
  }
}
function getBaseFileName(fileName) {
  return (fileName || 'mockup').replace(/\.[^.]+$/, '') || 'mockup'
}

function onPreviewMouse(e) {
  if (!mouseParallax.value) return
  const el = e.currentTarget
  const r = el.getBoundingClientRect()
  parallax.value = {
    x: ((e.clientX - r.left) / r.width - 0.5) * 2,
    y: ((e.clientY - r.top) / r.height - 0.5) * 2,
  }
}
function resetParallax() {
  parallax.value = { x: 0, y: 0 }
}

function setPreset(name) {
  if (name === 'face') { tiltX.value = 0; tiltY.value = 0; rotation.value = 0 }
  if (name === 'hero') { tiltX.value = -8; tiltY.value = 15; rotation.value = -3 }
  if (name === 'dramatic') { tiltX.value = -14; tiltY.value = -24; rotation.value = -7 }
  if (name === 'top') { tiltX.value = -30; tiltY.value = 0; rotation.value = 0 }
}

/* ------------------------------ image handling ---------------------------- */

function acceptFile(file) {
  if (!file || !file.type.startsWith('image/')) return
  revokeImageUrl()
  imageFile.value = file
  imageUrl.value = URL.createObjectURL(file)
  imageName.value = file.name
}

function onFileChange(event) {
  const [file] = event.target.files ?? []
  acceptFile(file)
  event.target.value = ''
}
function onDrop(e) {
  e.preventDefault()
  isDragging.value = false
  const [file] = e.dataTransfer?.files ?? []
  acceptFile(file)
}
async function loadDemoImage() {
  try {
    const res = await fetch('https://picsum.photos/900/1900')
    const blob = await res.blob()
    acceptFile(new File([blob], 'demo-picsum.jpg', { type: blob.type || 'image/jpeg' }))
  } catch {
    // fallback: dégradé généré localement
    const c = document.createElement('canvas')
    c.width = 900; c.height = 1900
    const ctx = c.getContext('2d')
    const g = ctx.createLinearGradient(0, 0, 900, 1900)
    g.addColorStop(0, '#7c3aed'); g.addColorStop(0.5, '#db2777'); g.addColorStop(1, '#f59e0b')
    ctx.fillStyle = g; ctx.fillRect(0, 0, 900, 1900)
    ctx.fillStyle = 'rgba(255,255,255,.92)'
    ctx.font = 'bold 64px Inter, sans-serif'
    ctx.fillText('Votre app ici', 120, 950)
    c.toBlob((blob) => blob && acceptFile(new File([blob], 'demo.png', { type: 'image/png' })), 'image/png')
  }
}
function clearImage() {
  revokeImageUrl()
  imageUrl.value = ''
  imageName.value = ''
  imageFile.value = null
}

/* --------------------------- asset frame detection ------------------------ */

function drawRoundedRect(ctx, x, y, w, h, r) {
  const radius = Math.min(r, w / 2, h / 2)
  ctx.beginPath()
  ctx.moveTo(x + radius, y)
  ctx.lineTo(x + w - radius, y)
  ctx.quadraticCurveTo(x + w, y, x + w, y + radius)
  ctx.lineTo(x + w, y + h - radius)
  ctx.quadraticCurveTo(x + w, y + h, x + w - radius, y + h)
  ctx.lineTo(x + radius, y + h)
  ctx.quadraticCurveTo(x, y + h, x, y + h - radius)
  ctx.lineTo(x, y + radius)
  ctx.quadraticCurveTo(x, y, x + radius, y)
  ctx.closePath()
}
function getImagePlacement(iw, ih, bw, bh, mode) {
  const s = mode === 'cover' ? Math.max(bw / iw, bh / ih) : Math.min(bw / iw, bh / ih)
  const w = iw * s
  const h = ih * s
  return { x: (bw - w) / 2, y: (bh - h) / 2, w, h }
}
function loadImage(url) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = () => reject(new Error("L'image n'a pas pu être chargée."))
    img.src = url
  })
}
function findMatchingPixel(w, h, pred) {
  const cx = Math.floor(w / 2)
  const cy = Math.floor(h / 2)
  if (pred(cx, cy)) return { x: cx, y: cy }
  const maxR = Math.min(Math.floor(Math.min(w, h) / 3), 500)
  for (let r = 1; r <= maxR; r++) {
    const minX = Math.max(0, cx - r), maxX = Math.min(w - 1, cx + r)
    const minY = Math.max(0, cy - r), maxY = Math.min(h - 1, cy + r)
    for (let x = minX; x <= maxX; x++) {
      if (pred(x, minY)) return { x, y: minY }
      if (pred(x, maxY)) return { x, y: maxY }
    }
    for (let y = minY + 1; y < maxY; y++) {
      if (pred(minX, y)) return { x: minX, y }
      if (pred(maxX, y)) return { x: maxX, y }
    }
  }
  return null
}
function floodFillBounds(w, h, sx, sy, pred) {
  const stack = [sy * w + sx]
  const visited = new Uint8Array(w * h)
  let minX = sx, maxX = sx, minY = sy, maxY = sy, pixels = 0
  while (stack.length) {
    const i = stack.pop()
    if (visited[i]) continue
    visited[i] = 1
    const x = i % w, y = (i - x) / w
    if (!pred(x, y)) continue
    pixels++
    if (x < minX) minX = x
    if (x > maxX) maxX = x
    if (y < minY) minY = y
    if (y > maxY) maxY = y
    if (x > 0) stack.push(i - 1)
    if (x < w - 1) stack.push(i + 1)
    if (y > 0) stack.push(i - w)
    if (y < h - 1) stack.push(i + w)
  }
  return { minX, minY, maxX, maxY, width: maxX - minX + 1, height: maxY - minY + 1, pixels }
}
function detectFrameScreen(frameImage) {
  const width = frameImage.naturalWidth || frameImage.width
  const height = frameImage.naturalHeight || frameImage.height
  const canvas = document.createElement('canvas')
  canvas.width = width; canvas.height = height
  const ctx = canvas.getContext('2d')
  if (!ctx) return { screen: fallbackAssetScreen, screenMode: 'transparent' }
  ctx.drawImage(frameImage, 0, 0, width, height)
  const data = ctx.getImageData(0, 0, width, height).data
  const alphaAt = (x, y) => data[(y * width + x) * 4 + 3]
  const lumAt = (x, y) => {
    const o = (y * width + x) * 4
    return 0.2126 * data[o] + 0.7152 * data[o + 1] + 0.0722 * data[o + 2]
  }
  const cx = Math.floor(width / 2), cy = Math.floor(height / 2)
  const screenMode = alphaAt(cx, cy) < 20 ? 'transparent' : 'dark'
  const isScreen = screenMode === 'transparent'
    ? (x, y) => alphaAt(x, y) < 20
    : (x, y) => alphaAt(x, y) > 200 && lumAt(x, y) < 24
  const start = findMatchingPixel(width, height, isScreen)
  if (!start) return { screen: fallbackAssetScreen, screenMode }
  const b = floodFillBounds(width, height, start.x, start.y, isScreen)
  if (!(b.pixels > 0 && b.width >= width * 0.3 && b.height >= height * 0.5)) {
    return { screen: fallbackAssetScreen, screenMode }
  }
  return { screen: { x: b.minX / width, y: b.minY / height, width: b.width / width, height: b.height / height }, screenMode }
}
async function setIphoneFrame(url, fileName) {
  const img = await loadImage(url)
  const { screen, screenMode } = detectFrameScreen(img)
  frameImageUrl.value = url
  frameImageName.value = fileName
  frameImageMeta.value = { width: img.naturalWidth || img.width, height: img.naturalHeight || img.height, screen, screenMode }
}
async function loadDefaultFrameAsset() {
  try { await setIphoneFrame('/assets/iphone-15-frame.png', 'iphone-15-frame.png') }
  catch (e) { console.info('Asset iPhone par défaut introuvable.', e) }
}
async function onFrameFileChange(e) {
  const [file] = e.target.files ?? []
  if (!file || !file.type.startsWith('image/')) return
  revokeFrameObjectUrl()
  const url = URL.createObjectURL(file)
  frameImageObjectUrl.value = url
  await setIphoneFrame(url, file.name)
  e.target.value = ''
}
async function clearCustomFrameAsset() {
  revokeFrameObjectUrl()
  frameImageUrl.value = ''; frameImageName.value = ''; frameImageMeta.value = null
  await loadDefaultFrameAsset()
}

/* --------------------------------- export --------------------------------- */

function sceneGradient(ctx, W, H) {
  const s = currentScene.value
  if (s.id === 'transparent') return null
  let from = s.from, to = s.to
  if (s.id === 'custom') { from = lighten(customBg.value, 25); to = darken(customBg.value, 35) }
  const g = ctx.createLinearGradient(0, 0, W, H)
  g.addColorStop(0, from); g.addColorStop(1, to)
  return g
}

async function downloadPhoneMockup() {
  if (!imageUrl.value || isExporting.value) return
  isExporting.value = true
  try {
    await downloadStudioMockup()
  } finally { isExporting.value = false }
}

/* Asset PNG : rendu à plat puis projeté en 3D comme les autres appareils */
async function renderFlatAssetPhone(src) {
  const frame = await loadImage(frameImageUrl.value)
  const W = frame.naturalWidth || frame.width
  const H = frame.naturalHeight || frame.height
  const pc = document.createElement('canvas')
  pc.width = W
  pc.height = H
  const p = pc.getContext('2d')
  const screen = frameImageMeta.value.screen ?? fallbackAssetScreen
  const sx = Math.round(screen.x * W)
  const sy = Math.round(screen.y * H)
  const sw = Math.round(screen.width * W)
  const sh = Math.round(screen.height * H)
  if (isOpaqueAssetFrame.value) p.drawImage(frame, 0, 0, W, H)
  drawRoundedRect(p, sx, sy, sw, sh, sw * 0.06)
  p.save()
  p.clip()
  p.fillStyle = '#000'
  p.fillRect(sx, sy, sw, sh)
  const place = getImagePlacement(src.width, src.height, sw, sh, fitMode.value)
  const zw = place.w * zoom.value
  const zh = place.h * zoom.value
  p.imageSmoothingQuality = 'high'
  p.drawImage(src, sx + (sw - zw) / 2, sy + (sh - zh) / 2 + (shiftY.value / 100) * sh, zw, zh)
  if (showGlare.value) {
    const g = p.createLinearGradient(sx, sy, sx + sw * 0.9, sy + sh * 0.45)
    g.addColorStop(0, 'rgba(255,255,255,.10)')
    g.addColorStop(0.4, 'rgba(255,255,255,0)')
    p.fillStyle = g
    p.fillRect(sx, sy, sw, sh)
  }
  p.restore()
  if (!isOpaqueAssetFrame.value) p.drawImage(frame, 0, 0, W, H)
  // rogne l'ombre cuite dans le PNG pour éviter le contour fantôme après le warp
  if (frameImageMeta.value?.screenMode === 'transparent') return cropTransparentMargins(pc)
  return pc
}

/* Rogne les marges transparentes (ombres cuites dans l'asset) */
function cropTransparentMargins(canvas, threshold = 16, pad = 3) {
  const w = canvas.width
  const h = canvas.height
  const ctx = canvas.getContext('2d')
  const data = ctx.getImageData(0, 0, w, h).data
  let minX = w
  let minY = h
  let maxX = -1
  let maxY = -1
  for (let y = 0; y < h; y++) {
    for (let x = 0; x < w; x++) {
      if (data[(y * w + x) * 4 + 3] > threshold) {
        if (x < minX) minX = x
        if (x > maxX) maxX = x
        if (y < minY) minY = y
        if (y > maxY) maxY = y
      }
    }
  }
  if (maxX < 0) return canvas
  minX = Math.max(0, minX - pad)
  minY = Math.max(0, minY - pad)
  maxX = Math.min(w - 1, maxX + pad)
  maxY = Math.min(h - 1, maxY + pad)
  const out = document.createElement('canvas')
  out.width = maxX - minX + 1
  out.height = maxY - minY + 1
  out.getContext('2d').drawImage(canvas, minX, minY, out.width, out.height, 0, 0, out.width, out.height)
  return out
}

async function downloadStudioMockup() {
  const src = await loadImage(imageUrl.value)
  const W = 1600, H = 1200
  const c = document.createElement('canvas')
  c.width = W; c.height = H
  const ctx = c.getContext('2d')

  // fond studio (rien si Transparent → PNG alpha)
  const bgGrad = sceneGradient(ctx, W, H)
  if (bgGrad) {
    ctx.fillStyle = bgGrad
    ctx.fillRect(0, 0, W, H)
  }
  // orbes décoratifs
  const orb = (x, y, r, color) => {
    const g = ctx.createRadialGradient(x, y, 0, x, y, r)
    g.addColorStop(0, color + 'aa'); g.addColorStop(1, color + '00')
    ctx.fillStyle = g
    ctx.fillRect(x - r, y - r, r * 2, r * 2)
  }
  if (showOrbs.value) {
    orb(W * 0.2, H * 0.15, 420, currentScene.value.orb1 ?? '#ffffff')
    orb(W * 0.85, H * 0.85, 480, currentScene.value.orb2 ?? '#ffffff')
  }

  // ---- appareil rendu à plat, puis projeté en 3D (asset PNG inclus) ----
  const pc = useIphoneAssetFrame.value ? await renderFlatAssetPhone(src) : renderFlatDevice(src)
  const flatW = pc.width
  const flatH = pc.height

  // ---- projection 3D : mêmes angles que la prévisualisation ----
  const fitH = H * 0.78
  const dispScale = Math.min(1, fitH / flatH) * (0.8 + scale.value * 0.2)
  const w = flatW * dispScale, h = flatH * dispScale
  const quad = projectQuad(W / 2, H / 2 + 24, w, h, tiltX.value, tiltY.value, rotation.value, 1200)

  // ombre au sol sous le téléphone, décalée côté éloigné
  const bottomY = Math.max(quad[2].y, quad[3].y)
  ctx.save()
  ctx.translate(W / 2 + tiltY.value * 1.5, bottomY + 46)
  ctx.scale(1, 0.2)
  const shR = w * 0.72
  const shG = ctx.createRadialGradient(0, 0, 8, 0, 0, shR)
  shG.addColorStop(0, `rgba(0,0,0,${0.18 + (shadowStrength.value / 100) * 0.5})`)
  shG.addColorStop(1, 'rgba(0,0,0,0)')
  ctx.fillStyle = shG
  ctx.fillRect(-shR, -shR, shR * 2, shR * 2)
  ctx.restore()

  // épaisseur synthétique uniquement pour les rendus vectoriels ;
  // le PNG asset est déjà photoréaliste, les tranches créeraient un contour fantôme
  if (!useIphoneAssetFrame.value) {
    drawThickness(ctx, quad, tiltY.value >= 0 ? 'right' : 'left', dispScale)
  }
  drawWarped(ctx, pc, quad)
  if (!useIphoneAssetFrame.value) {
    applyVolumeLight(ctx, quad)
  }

  const baseName = `${getBaseFileName(imageName.value)}-${selectedDeviceKey.value}-studio-3d`
  const targets = exportFormat.value === 'both' ? ['png', 'webp'] : [exportFormat.value]
  for (const fmt of targets) {
    const a = document.createElement('a')
    if (fmt === 'webp') {
      a.href = c.toDataURL('image/webp', webpQuality.value)
      a.download = `${baseName}.webp`
    } else {
      a.href = c.toDataURL('image/png')
      a.download = `${baseName}.png`
    }
    a.click()
  }
}

function frameGradientFill(p, x0, y0, x1, y1) {
  const g = p.createLinearGradient(x0, y0, x1, y1)
  g.addColorStop(0, frameEdge.value)
  g.addColorStop(0.3, frameBase.value)
  g.addColorStop(0.7, frameDark.value)
  g.addColorStop(1, frameBase.value)
  return g
}

function paintScreen(p, src, sx, sy, sw, sh, sr) {
  drawRoundedRect(p, sx, sy, sw, sh, sr)
  p.save()
  p.clip()
  p.fillStyle = '#000'
  p.fillRect(sx, sy, sw, sh)
  const place = getImagePlacement(src.width, src.height, sw, sh, fitMode.value)
  const zw = place.w * zoom.value
  const zh = place.h * zoom.value
  const zx = sx + (sw - zw) / 2
  const zy = sy + (sh - zh) / 2 + (shiftY.value / 100) * sh
  p.imageSmoothingQuality = 'high'
  p.drawImage(src, zx, zy, zw, zh)
  if (showGlare.value) {
    const g = p.createLinearGradient(sx, sy, sx + sw * 0.9, sy + sh * 0.45)
    g.addColorStop(0, 'rgba(255,255,255,.13)')
    g.addColorStop(0.35, 'rgba(255,255,255,.03)')
    g.addColorStop(0.6, 'rgba(255,255,255,0)')
    p.fillStyle = g
    p.fillRect(sx, sy, sw, sh)
  }
  p.restore()
  // ombre interne du bezel : fusion écran/cadre
  drawRoundedRect(p, sx, sy, sw, sh, sr)
  p.strokeStyle = 'rgba(0,0,0,.65)'
  p.lineWidth = Math.max(4, sw * 0.008)
  p.stroke()
}

function paintBezelCam(p, cx, cy, r) {
  p.beginPath()
  p.arc(cx, cy, r, 0, Math.PI * 2)
  p.fillStyle = '#05070c'
  p.fill()
  p.beginPath()
  p.arc(cx - r * 0.25, cy - r * 0.25, r * 0.32, 0, Math.PI * 2)
  p.fillStyle = '#1d2b45'
  p.fill()
}

function renderFlatDevice(src) {
  const device = currentDevice.value
  if (device.kind === 'laptop') return renderFlatLaptop(src, device)
  if (device.kind === 'monitor') return renderFlatMonitor(src, device)
  return renderFlatSlab(src, device)
}

/* Téléphone & tablette : dalle arrondie */
function renderFlatSlab(src, device) {
  const flatW = device.kind === 'tablet' ? 820 : 760
  const flatH = Math.round((flatW * device.ratioHeight) / device.ratioWidth)
  const pc = document.createElement('canvas')
  pc.width = flatW
  pc.height = flatH
  const p = pc.getContext('2d')
  const k = flatW / device.previewWidth
  const pad = device.padding * k
  const fr = device.frameRadius * k
  const sr = device.screenRadius * k

  drawRoundedRect(p, 0, 0, flatW, flatH, fr)
  p.fillStyle = frameGradientFill(p, 0, 0, flatW, flatH)
  p.fill()

  drawRoundedRect(p, 2.5, 2.5, flatW - 5, flatH - 5, fr - 2)
  p.strokeStyle = 'rgba(255,255,255,.28)'
  p.lineWidth = 2.5
  p.stroke()

  const sx = pad, sy = pad, sw = flatW - pad * 2, sh = flatH - pad * 2
  paintScreen(p, src, sx, sy, sw, sh, sr)

  p.fillStyle = '#000'
  if (device.style === 'island') {
    const iw = flatW * 0.32, ih = 26 * k
    drawRoundedRect(p, (flatW - iw) / 2, sy + 10 * k, iw, ih, ih / 2)
    p.fill()
  } else if (device.style === 'notch') {
    const nw = flatW * 0.42, nh = 24 * k
    p.beginPath()
    const nx = (flatW - nw) / 2, ny = 6 * k
    p.moveTo(nx, ny)
    p.lineTo(nx + nw, ny)
    p.lineTo(nx + nw, ny + nh - 10 * k)
    p.quadraticCurveTo(nx + nw, ny + nh, nx + nw - 10 * k, ny + nh)
    p.lineTo(nx + 10 * k, ny + nh)
    p.quadraticCurveTo(nx, ny + nh, nx, ny + nh - 10 * k)
    p.closePath()
    p.fill()
  } else if (device.style === 'hole') {
    p.beginPath()
    p.arc(flatW / 2, sy + 14 * k, 7 * k, 0, Math.PI * 2)
    p.fill()
  }
  if (device.bezelCam) {
    paintBezelCam(p, flatW / 2, pad / 2, Math.max(4, 5 * k * 0.5))
  }
  return pc
}

/* MacBook : écran 16:10 + base clavier */
function renderFlatLaptop(src, device) {
  const LW = 1080
  const scrH = Math.round((LW * device.ratioHeight) / device.ratioWidth)
  const k = LW / device.previewWidth
  const bez = device.padding * k
  const scrH2 = scrH + bez * 2
  const baseH = 34
  const side = 90
  const W = LW + side * 2
  const H = scrH2 + 10 + baseH + 14
  const pc = document.createElement('canvas')
  pc.width = W
  pc.height = H
  const p = pc.getContext('2d')
  const ox = side

  // écran
  const fr = device.frameRadius * k
  const sr = device.screenRadius * k
  drawRoundedRect(p, ox, 0, LW, scrH2, fr)
  p.fillStyle = frameGradientFill(p, ox, 0, ox + LW, scrH2)
  p.fill()
  paintScreen(p, src, ox + bez, bez, LW - bez * 2, scrH, sr)
  if (device.bezelCam) paintBezelCam(p, ox + LW / 2, bez / 2, 4.5)

  // base : trapèze vu de face, plus large que l'écran
  const by = scrH2 + 10
  const topW = LW * 0.94
  const botW = LW + side * 1.5
  p.beginPath()
  p.moveTo(ox + (LW - topW) / 2, by)
  p.lineTo(ox + (LW + topW) / 2, by)
  p.lineTo(ox + (LW + botW) / 2, by + baseH)
  p.lineTo(ox + (LW - botW) / 2, by + baseH)
  p.closePath()
  const bg = p.createLinearGradient(0, by, 0, by + baseH)
  bg.addColorStop(0, frameEdge.value)
  bg.addColorStop(0.35, frameBase.value)
  bg.addColorStop(1, frameDark.value)
  p.fillStyle = bg
  p.fill()
  // arête avant claire + encoche trackpad
  p.fillStyle = 'rgba(255,255,255,.35)'
  p.fillRect(ox + (LW - topW) / 2, by, topW, 2)
  drawRoundedRect(p, ox + LW / 2 - 60, by + 1, 120, 9, 4.5)
  p.fillStyle = 'rgba(0,0,0,.45)'
  p.fill()
  return pc
}

/* iMac : écran 16:9 + pied */
function renderFlatMonitor(src, device) {
  const MW = 1080
  const scrH = Math.round((MW * device.ratioHeight) / device.ratioWidth)
  const k = MW / device.previewWidth
  const bez = device.padding * k
  const scrH2 = scrH + bez * 2
  const neckW = 74
  const neckH = 96
  const footW = 320
  const footH = 26
  const W = MW + 60
  const H = scrH2 + neckH + footH + 20
  const pc = document.createElement('canvas')
  pc.width = W
  pc.height = H
  const p = pc.getContext('2d')
  const ox = 30

  // pied (derrière l'écran)
  p.fillStyle = frameDark.value
  p.beginPath()
  p.ellipse(ox + MW / 2, scrH2 + neckH + footH / 2, footW / 2, footH / 2, 0, 0, Math.PI * 2)
  p.fill()
  p.fillStyle = 'rgba(255,255,255,.18)'
  p.beginPath()
  p.ellipse(ox + MW / 2, scrH2 + neckH + footH / 2 - 3, footW / 2 - 14, footH / 2 - 6, 0, 0, Math.PI * 2)
  p.fill()
  // cou
  const neckGrad = p.createLinearGradient(0, scrH2 - 20, 0, scrH2 + neckH)
  neckGrad.addColorStop(0, frameBase.value)
  neckGrad.addColorStop(1, frameDark.value)
  p.fillStyle = neckGrad
  p.fillRect(ox + MW / 2 - neckW / 2, scrH2 - 20, neckW, neckH + 20)

  // écran
  const fr = device.frameRadius * k
  const sr = device.screenRadius * k
  drawRoundedRect(p, ox, 0, MW, scrH2, fr)
  p.fillStyle = frameGradientFill(p, ox, 0, ox + MW, scrH2)
  p.fill()
  paintScreen(p, src, ox + bez, bez, MW - bez * 2, scrH, sr)
  if (device.bezelCam) paintBezelCam(p, ox + MW / 2, bez / 2, 4.5)
  return pc
}

/* Épaisseur : fines tranches plaquées sur la silhouette (pas de dos dédoublé) */
function drawThickness(ctx, quad, face, dispScale) {
  const t = Math.max(4, 8 * dispScale)
  const [tl, tr, br, bl] = quad
  const poly = (pts, fill) => {
    ctx.beginPath()
    ctx.moveTo(pts[0].x, pts[0].y)
    for (let i = 1; i < pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y)
    ctx.closePath()
    ctx.fillStyle = fill
    ctx.fill()
  }
  // tranche basse (toujours visible)
  const eb = { x: face === 'right' ? t * 0.3 : -t * 0.3, y: t }
  const botGrad = ctx.createLinearGradient(0, bl.y, 0, bl.y + t)
  botGrad.addColorStop(0, frameDark.value)
  botGrad.addColorStop(1, '#05070b')
  poly(
    [bl, br, { x: br.x + eb.x, y: br.y + eb.y }, { x: bl.x + eb.x, y: bl.y + eb.y }],
    botGrad,
  )
  // tranche latérale du côté éloigné, extrudée vers l'extérieur
  if (face === 'right') {
    const g = ctx.createLinearGradient(tr.x, 0, tr.x + t, 0)
    g.addColorStop(0, frameDark.value)
    g.addColorStop(1, '#0a0d13')
    poly(
      [tr, br, { x: br.x + t, y: br.y + t * 0.2 }, { x: tr.x + t, y: tr.y + t * 0.2 }],
      g,
    )
  } else {
    const g = ctx.createLinearGradient(tl.x, 0, tl.x - t, 0)
    g.addColorStop(0, frameDark.value)
    g.addColorStop(1, '#0a0d13')
    poly(
      [tl, bl, { x: bl.x - t, y: bl.y + t * 0.2 }, { x: tl.x - t, y: tl.y + t * 0.2 }],
      g,
    )
  }
}

function quadPath(ctx, quad) {
  ctx.beginPath()
  ctx.moveTo(quad[0].x, quad[0].y)
  for (let i = 1; i < 4; i++) ctx.lineTo(quad[i].x, quad[i].y)
  ctx.closePath()
}

/* Modelé 3D : lumière directionnelle + occlusion des bords + rim light */
function applyVolumeLight(ctx, quad) {
  const [tl, tr, br, bl] = quad
  const nearLeft = tiltY.value >= 0 // arête gauche plus proche de l'observateur
  const amt = Math.min(1, Math.abs(tiltY.value) / 12) // 0 de face → 1 dès ~12°
  const s = 0.35 + 0.65 * amt
  const lc = { x: (tl.x + bl.x) / 2, y: (tl.y + bl.y) / 2 }
  const rc = { x: (tr.x + br.x) / 2, y: (tr.y + br.y) / 2 }

  ctx.save()
  quadPath(ctx, quad)
  ctx.clip()
  // clair côté proche, sombre côté éloigné
  const g = nearLeft
    ? ctx.createLinearGradient(rc.x, rc.y, lc.x, lc.y)
    : ctx.createLinearGradient(lc.x, lc.y, rc.x, rc.y)
  g.addColorStop(0, `rgba(0,0,0,${0.26 * s})`)
  g.addColorStop(0.45, 'rgba(0,0,0,0)')
  g.addColorStop(0.75, `rgba(255,255,255,${0.1 * s})`)
  g.addColorStop(1, `rgba(255,255,255,${0.16 * s})`)
  ctx.fillStyle = g
  ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height)
  // occlusion : assombrit l'intérieur de la silhouette pour creuser les bords
  if (typeof ctx.filter === 'string') {
    ctx.filter = 'blur(10px)'
    quadPath(ctx, quad)
    ctx.lineWidth = 34
    ctx.strokeStyle = 'rgba(0,0,0,0.32)'
    ctx.stroke()
    ctx.filter = 'none'
  }
  ctx.restore()

  // filet de lumière sur l'arête proche (rendus vectoriels uniquement :
  // sur le PNG asset, ça ferait un liseré parasite sur le cadre photo)
  if (!useIphoneAssetFrame.value) {
    ctx.save()
    ctx.beginPath()
    if (nearLeft) {
      ctx.moveTo(tl.x, tl.y)
      ctx.lineTo(bl.x, bl.y)
    } else {
      ctx.moveTo(tr.x, tr.y)
      ctx.lineTo(br.x, br.y)
    }
    ctx.lineWidth = 2.5
    ctx.strokeStyle = 'rgba(255,255,255,0.35)'
    ctx.stroke()
    ctx.restore()
  }
}

function projectQuad(cx, cy, w, h, rotXdeg, rotYdeg, rotZdeg, persp) {
  const rx = (rotXdeg * Math.PI) / 180
  const ry = (rotYdeg * Math.PI) / 180
  const pts = [
    { x: -w / 2, y: -h / 2 },
    { x: w / 2, y: -h / 2 },
    { x: w / 2, y: h / 2 },
    { x: -w / 2, y: h / 2 },
  ].map((pt) => {
    const y1 = pt.y * Math.cos(rx)
    const z1 = -pt.y * Math.sin(rx)
    const x2 = pt.x * Math.cos(ry) + z1 * Math.sin(ry)
    const z2 = -pt.x * Math.sin(ry) + z1 * Math.cos(ry)
    const s = persp / (persp - z2)
    return { x: x2 * s, y: y1 * s }
  })
  const rz = (rotZdeg * Math.PI) / 180
  const cos = Math.cos(rz)
  const sin = Math.sin(rz)
  return pts.map((pt) => ({
    x: cx + pt.x * cos - pt.y * sin,
    y: cy + pt.x * sin + pt.y * cos,
  }))
}

function solveLinear8(A, b) {
  const n = 8
  const M = A.map((row, i) => [...row, b[i]])
  for (let col = 0; col < n; col++) {
    let piv = col
    for (let r = col + 1; r < n; r++) {
      if (Math.abs(M[r][col]) > Math.abs(M[piv][col])) piv = r
    }
    const tmp = M[col]
    M[col] = M[piv]
    M[piv] = tmp
    const d = M[col][col] || 1e-12
    for (let r = 0; r < n; r++) {
      if (r === col) continue
      const f = M[r][col] / d
      for (let k = col; k <= n; k++) M[r][k] -= f * M[col][k]
    }
  }
  return M.map((row, i) => row[n] / (row[i] || 1e-12))
}

function invert3(m) {
  const [a, b, c, d, e, f, g, h, i] = m
  const A = e * i - f * h
  const B = f * g - d * i
  const C = d * h - e * g
  const det = a * A + b * B + c * C || 1e-12
  return [
    A / det, (c * h - b * i) / det, (b * f - c * e) / det,
    B / det, (a * i - c * g) / det, (c * d - a * f) / det,
    C / det, (b * g - a * h) / det, (a * e - b * d) / det,
  ]
}

/* Warp projectif exact, pixel par pixel (homographie inversée + bilinéaire).
   Aucune tranche, donc aucune couture ni double échantillonnage des bords. */
function drawWarped(ctx, src, quad) {
  const sw = src.width
  const sh = src.height
  const sData = src.getContext('2d').getImageData(0, 0, sw, sh).data

  const [tl, tr, br, bl] = quad
  const xs = [0, sw, sw, 0]
  const ys = [0, 0, sh, sh]
  const xd = [tl.x, tr.x, br.x, bl.x]
  const yd = [tl.y, tr.y, br.y, bl.y]
  const A = []
  const bv = []
  for (let i = 0; i < 4; i++) {
    A.push([xs[i], ys[i], 1, 0, 0, 0, -xd[i] * xs[i], -xd[i] * ys[i]])
    bv.push(xd[i])
    A.push([0, 0, 0, xs[i], ys[i], 1, -yd[i] * xs[i], -yd[i] * ys[i]])
    bv.push(yd[i])
  }
  const h = solveLinear8(A, bv)
  const H = invert3([h[0], h[1], h[2], h[3], h[4], h[5], h[6], h[7], 1])

  const cw = ctx.canvas.width
  const ch = ctx.canvas.height
  const minX = Math.max(0, Math.floor(Math.min(tl.x, tr.x, br.x, bl.x)))
  const maxX = Math.min(cw - 1, Math.ceil(Math.max(tl.x, tr.x, br.x, bl.x)))
  const minY = Math.max(0, Math.floor(Math.min(tl.y, tr.y, br.y, bl.y)))
  const maxY = Math.min(ch - 1, Math.ceil(Math.max(tl.y, tr.y, br.y, bl.y)))
  const outW = maxX - minX + 1
  const outH = maxY - minY + 1
  if (outW <= 0 || outH <= 0) return
  const out = ctx.createImageData(outW, outH)
  const dData = out.data

  for (let y = 0; y < outH; y++) {
    const dy = y + minY + 0.5
    for (let x = 0; x < outW; x++) {
      const dx = x + minX + 0.5
      const w = H[6] * dx + H[7] * dy + H[8]
      const sx = (H[0] * dx + H[1] * dy + H[2]) / w
      const sy = (H[3] * dx + H[4] * dy + H[5]) / w
      if (sx < -0.5 || sy < -0.5 || sx > sw - 0.5 || sy > sh - 0.5) continue
      const cx = Math.min(Math.max(sx, 0), sw - 1.001)
      const cy = Math.min(Math.max(sy, 0), sh - 1.001)
      const x0 = Math.floor(cx)
      const y0 = Math.floor(cy)
      const x1 = Math.min(x0 + 1, sw - 1)
      const y1 = Math.min(y0 + 1, sh - 1)
      const fx = cx - x0
      const fy = cy - y0
      const i00 = (y0 * sw + x0) * 4
      const i10 = (y0 * sw + x1) * 4
      const i01 = (y1 * sw + x0) * 4
      const i11 = (y1 * sw + x1) * 4
      const o = (y * outW + x) * 4
      for (let k = 0; k < 4; k++) {
        const top = sData[i00 + k] + (sData[i10 + k] - sData[i00 + k]) * fx
        const bot = sData[i01 + k] + (sData[i11 + k] - sData[i01 + k]) * fx
        dData[o + k] = top + (bot - top) * fy
      }
    }
  }
  const warped = document.createElement('canvas')
  warped.width = outW
  warped.height = outH
  warped.getContext('2d').putImageData(out, 0, 0)
  ctx.drawImage(warped, minX, minY)
}

watch([scale, selectedDeviceKey], () => updateFit())

let fitRO = null
onMounted(() => {
  loadDefaultFrameAsset()
  updateFit()
  fitRO = new ResizeObserver(updateFit)
  if (stageEl.value) fitRO.observe(stageEl.value)
  window.addEventListener('resize', updateFit)
})
onBeforeUnmount(() => {
  revokeImageUrl()
  revokeFrameObjectUrl()
  if (fitRO) fitRO.disconnect()
  window.removeEventListener('resize', updateFit)
})
</script>

<template>
  <div class="app">
    <!-- ======================= SIDEBAR ======================= -->
    <aside class="sidebar">
      <div class="brand">
        <div class="logo">◈</div>
        <div>
          <h1>Mockup Studio</h1>
          <p>Téléphone · rendu pro en 1 clic</p>
        </div>
        <span class="badge">v2</span>
      </div>

      <!-- Image -->
      <div class="card">
        <div class="card-title">① Image source</div>
        <div
          class="dropzone"
          :class="{ dragging: isDragging, filled: imageUrl }"
          @dragover.prevent="isDragging = true"
          @dragleave="isDragging = false"
          @drop="onDrop"
          @click="$refs.fileInput.click()"
        >
          <input ref="fileInput" type="file" accept="image/*" hidden @change="onFileChange" />
          <template v-if="imageUrl">
            <img :src="imageUrl" alt="" class="dz-thumb" />
            <div class="dz-meta">
              <strong>{{ imageName }}</strong>
              <span>Cliquez ou glissez pour remplacer</span>
            </div>
            <button class="icon-btn" title="Retirer" @click.stop="clearImage">✕</button>
          </template>
          <template v-else>
            <div class="dz-icon">＋</div>
            <strong>Glissez votre capture ici</strong>
            <span>ou cliquez pour parcourir · PNG / JPG</span>
          </template>
        </div>
        <button class="ghost-btn" @click="loadDemoImage">✨ Tester avec une image démo</button>
        <div class="seg">
          <button :class="{ active: fitMode === 'cover' }" @click="fitMode = 'cover'">Remplir</button>
          <button :class="{ active: fitMode === 'contain' }" @click="fitMode = 'contain'">Contenir</button>
        </div>
        <label class="slider">Zoom écran <b>{{ Math.round(zoom * 100) }}%</b>
          <input v-model.number="zoom" type="range" min="1" max="1.3" step="0.01" />
        </label>
        <label class="slider">Décalage vertical <b>{{ shiftY }}%</b>
          <input v-model.number="shiftY" type="range" min="-20" max="20" step="1" />
        </label>
      </div>

      <!-- Appareil -->
      <div class="card">
        <div class="card-title">② Appareil</div>
        <div class="device-grid">
          <button
            v-for="(d, key) in devices"
            :key="key"
            class="device-btn"
            :class="{ active: selectedDeviceKey === key }"
            @click="selectedDeviceKey = key"
          >
            <span v-if="d.kind === 'laptop'" class="mini-laptop"><i></i></span>
            <span v-else-if="d.kind === 'monitor'" class="mini-monitor"><i></i></span>
            <span v-else class="mini-phone" :class="d.style"><i></i></span>
            <strong>{{ d.label }}</strong>
            <small>{{ d.sub }}</small>
          </button>
        </div>
        <div class="card-title sm">Finition cadre</div>
        <div class="swatches">
          <button
            v-for="f in finishes"
            :key="f.id"
            class="swatch"
            :class="{ active: selectedFinishId === f.id }"
            :style="{ background: `linear-gradient(135deg, ${f.edge}, ${f.base} 55%, ${f.dark})` }"
            :title="f.label"
            @click="selectedFinishId = f.id"
          />
          <label class="swatch custom" :class="{ active: selectedFinishId === 'custom' }" title="Couleur perso">
            <input type="color" v-model="customFrameColor" hidden @change="selectedFinishId = 'custom'" @click="selectedFinishId = 'custom'" />
            <span>🎨</span>
          </label>
        </div>
        <div class="finish-name">{{ isCustomFinish ? 'Couleur personnalisée' : currentFinish.label }}</div>
        <input v-if="isCustomFinish" v-model="customFrameColor" type="color" class="color-full" />
      </div>

      <!-- Scène -->
      <div class="card">
        <div class="card-title">③ Scène & lumière</div>
        <div class="scene-grid">
          <button
            v-for="s in scenes"
            :key="s.id"
            class="scene-btn"
            :class="{ active: selectedSceneId === s.id }"
            @click="selectedSceneId = s.id"
          >
            <span
              class="scene-thumb"
              :style="s.bg ? { background: s.bg } : { background: customBg }"
            />
            <small>{{ s.label }}</small>
          </button>
        </div>
        <label v-if="selectedSceneId === 'custom'" class="field">
          <span>Couleur de fond</span>
          <input v-model="customBg" type="color" class="color-full" />
        </label>
      </div>

      <!-- Mise en scène -->
      <div class="card">
        <div class="card-title">④ Mise en scène 3D</div>
        <div class="presets">
          <button @click="setPreset('face')">Face</button>
          <button @click="setPreset('hero')">Hero ⭐</button>
          <button @click="setPreset('dramatic')">Dramatique</button>
          <button @click="setPreset('top')">Plongée</button>
        </div>
        <label class="slider">Échelle <b>{{ Math.round(scale * 100) }}%</b>
          <input v-model.number="scale" type="range" min="0.7" max="1.3" step="0.01" />
        </label>
        <label class="slider">Inclinaison X <b>{{ tiltX }}°</b>
          <input v-model.number="tiltX" type="range" min="-35" max="35" step="1" />
        </label>
        <label class="slider">Inclinaison Y <b>{{ tiltY }}°</b>
          <input v-model.number="tiltY" type="range" min="-35" max="35" step="1" />
        </label>
        <label class="slider">Rotation <b>{{ rotation }}°</b>
          <input v-model.number="rotation" type="range" min="-20" max="20" step="1" />
        </label>
        <label class="slider">Ombre <b>{{ shadowStrength }}%</b>
          <input v-model.number="shadowStrength" type="range" min="0" max="100" step="1" />
        </label>
        <div class="toggles">
          <label><input v-model="showGlare" type="checkbox" /> Reflet vitre</label>
          <label><input v-model="floatAnim" type="checkbox" /> Flottement</label>
          <label><input v-model="mouseParallax" type="checkbox" /> Parallaxe souris</label>
        </div>
        <div class="card-title sm">Format d'export</div>
        <div class="seg seg3">
          <button :class="{ active: exportFormat === 'png' }" @click="exportFormat = 'png'">PNG</button>
          <button :class="{ active: exportFormat === 'webp' }" @click="exportFormat = 'webp'">WebP</button>
          <button :class="{ active: exportFormat === 'both' }" @click="exportFormat = 'both'">Les deux</button>
        </div>
        <label v-if="exportFormat !== 'png'" class="slider">Qualité WebP <b>{{ Math.round(webpQuality * 100) }}%</b>
          <input v-model.number="webpQuality" type="range" min="0.6" max="1" step="0.01" />
        </label>
      </div>

      <div class="card subtle">
        <div class="card-title">Asset iPhone PNG (optionnel)</div>
        <label class="upload-row">
          <input type="file" accept="image/*" hidden @change="onFrameFileChange" />
          <span>{{ frameImageName || 'Choisir un PNG de cadre' }}</span>
        </label>
        <button v-if="frameImageObjectUrl" class="ghost-btn" @click="clearCustomFrameAsset">Retirer l'asset perso</button>
      </div>
    </aside>

    <!-- ======================= STAGE ======================= -->
    <main
      class="stage-wrap"
      :class="{ 'light-scene': !sceneIsDark }"
      :style="{ background: previewBg }"
      @mousemove="onPreviewMouse"
      @mouseleave="resetParallax"
    >
      <div class="orb orb-a" v-if="showOrbs" :style="{ background: currentScene.orb1 }" />
      <div class="orb orb-b" v-if="showOrbs" :style="{ background: currentScene.orb2 }" />
      <div class="grain" />

      <header class="stage-top">
        <div class="stage-info">
          <strong>{{ currentDevice.label }}</strong>
          <span>{{ currentFinish.label }} · {{ fitMode === 'cover' ? 'Remplir' : 'Contenir' }} · {{ exportLabel }} · 1600×1200 · vue 3D</span>
        </div>
        <div class="stage-actions">
          <button class="primary" :disabled="!imageUrl || isExporting" @click="downloadPhoneMockup">
            {{ isExporting ? 'Export…' : '⬇ Télécharger le mockup' }}
          </button>
        </div>
      </header>

      <div ref="stageEl" class="stage" :class="{ floating: floatAnim }">
        <!-- Asset PNG mode -->
        <div v-if="useIphoneAssetFrame" class="asset-phone" :style="assetPhoneStyle">
          <img v-if="isOpaqueAssetFrame" class="asset-frame underlay" :src="frameImageUrl" alt="" />
          <div class="asset-screen" :style="assetScreenStyle">
            <img v-if="imageUrl" :src="imageUrl" alt="" :style="imgStyle" />
            <div v-else class="empty"><span>＋</span><p>Ajoutez une image dans le panneau de gauche</p></div>
          </div>
          <img v-if="!isOpaqueAssetFrame" class="asset-frame overlay" :src="frameImageUrl" alt="" />
        </div>

        <!-- MacBook -->
        <div v-else-if="currentDevice.kind === 'laptop'" class="tilt" :style="{ transform: tiltTransform }">
          <div class="laptop" :style="wideStyle">
            <div class="laptop-screen">
              <div class="laptop-display">
                <img v-if="imageUrl" :src="imageUrl" alt="" :style="imgStyle" />
                <div v-else class="empty"><span>＋</span><p>Ajoutez une image<br />pour révéler le mockup</p></div>
                <div v-if="showGlare" class="glare" />
                <div v-if="currentDevice.bezelCam" class="laptop-cam" />
              </div>
            </div>
            <div class="laptop-base"><i></i></div>
          </div>
          <div class="floor-shadow wide" :style="{ opacity: shadowStrength / 100 }" />
        </div>

        <!-- iMac -->
        <div v-else-if="currentDevice.kind === 'monitor'" class="tilt" :style="{ transform: tiltTransform }">
          <div class="monitor" :style="wideStyle">
            <div class="monitor-screen">
              <div class="monitor-display">
                <img v-if="imageUrl" :src="imageUrl" alt="" :style="imgStyle" />
                <div v-else class="empty"><span>＋</span><p>Ajoutez une image<br />pour révéler le mockup</p></div>
                <div v-if="showGlare" class="glare" />
                <div v-if="currentDevice.bezelCam" class="laptop-cam" />
              </div>
            </div>
            <div class="monitor-neck"></div>
            <div class="monitor-foot"></div>
          </div>
          <div class="floor-shadow wide" :style="{ opacity: shadowStrength / 100 }" />
        </div>

        <!-- CSS realistic phone / tablet -->
        <div v-else class="tilt" :style="{ transform: tiltTransform }">
          <div class="phone" :style="phoneStyle">
            <span v-if="currentDevice.kind === 'phone'" class="btn-side b1" />
            <span v-if="currentDevice.kind === 'phone'" class="btn-side b2" />
            <span v-if="currentDevice.kind === 'phone'" class="btn-side b3" />
            <span v-if="currentDevice.kind === 'phone'" class="btn-side b4" />
            <div v-if="currentDevice.bezelCam" class="tablet-cam" />
            <div class="screen">
              <img v-if="imageUrl" :src="imageUrl" alt="" :style="imgStyle" />
              <div v-else class="empty"><span>＋</span><p>Ajoutez une image<br />pour révéler le mockup</p></div>
              <div v-if="showGlare" class="glare" />
              <div v-if="currentDevice.style === 'island'" class="island" />
              <div v-if="currentDevice.style === 'notch'" class="notch" />
              <div v-if="currentDevice.style === 'hole'" class="hole" />
            </div>
          </div>
          <div class="floor-shadow" :style="{ opacity: shadowStrength / 100 }" />
        </div>
      </div>

      <footer class="stage-bottom">
        <span v-if="!imageUrl">💡 Astuce : cliquez « Tester avec une image démo » pour voir le rendu instantanément.</span>
        <span v-else>💡 Bougez la souris sur la scène · l'export PNG reprend fond + ombre + reflet.</span>
      </footer>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 380px 1fr;
  background: #070b14;
}

/* ---------- sidebar ---------- */
.sidebar {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  background: #0b1120;
  border-right: 1px solid #1b2440;
  max-height: 100vh;
  overflow-y: auto;
}
.brand {
  display: flex;
  align-items: center;
  gap: 12px;
}
.logo {
  width: 42px; height: 42px;
  display: grid; place-items: center;
  border-radius: 13px;
  font-size: 22px;
  color: white;
  background: linear-gradient(135deg, #6366f1, #a855f7 55%, #ec4899);
  box-shadow: 0 8px 24px rgba(139, 92, 246, .45);
}
.brand h1 { margin: 0; font-size: 1.15rem; letter-spacing: -.02em; }
.brand p { margin: 0; font-size: .8rem; color: #8b94ad; }
.badge {
  margin-left: auto;
  font-size: .7rem; font-weight: 700;
  background: #1a2340; color: #a5b4fc;
  border: 1px solid #2b3a6b;
  padding: 2px 8px; border-radius: 20px;
}
.card {
  background: #111834;
  border: 1px solid #1f2a4d;
  border-radius: 16px;
  padding: 14px;
  display: grid;
  gap: 10px;
}
.card.subtle { background: #0e1428; }
.card-title { font-size: .78rem; font-weight: 800; letter-spacing: .08em; text-transform: uppercase; color: #a5b4fc; }
.card-title.sm { margin-top: 4px; }

/* dropzone */
.dropzone {
  border: 1.5px dashed #334155;
  border-radius: 12px;
  padding: 16px 12px;
  text-align: center;
  cursor: pointer;
  display: grid;
  gap: 4px;
  justify-items: center;
  background: #0b1226;
  transition: .2s;
  position: relative;
}
.dropzone:hover, .dropzone.dragging {
  border-color: #818cf8;
  background: #131b3a;
  transform: translateY(-1px);
}
.dropzone strong { font-size: .9rem; }
.dropzone span { font-size: .78rem; color: #8b94ad; }
.dz-icon { font-size: 26px; color: #818cf8; }
.dz-thumb { width: 52px; height: 92px; object-fit: cover; border-radius: 8px; border: 1px solid #334155; }
.dz-meta { display: grid; gap: 2px; }
.dz-meta strong { max-width: 240px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.icon-btn {
  position: absolute; top: 8px; right: 8px;
  width: 26px; height: 26px; border-radius: 50%;
  border: 1px solid #334155; background: #1e293b; color: #cbd5e1; cursor: pointer;
}
.ghost-btn {
  background: #1a2340;
  border: 1px solid #2b3a6b;
  color: #c7d2fe;
  border-radius: 10px;
  padding: 8px 10px;
  cursor: pointer;
  font-size: .85rem;
  transition: .15s;
}
.ghost-btn:hover { background: #232f56; }
.seg { display: grid; grid-template-columns: 1fr 1fr; gap: 6px; }
.seg button, .presets button {
  background: #0b1226; border: 1px solid #2b3a6b; color: #94a3b8;
  border-radius: 9px; padding: 7px; cursor: pointer; font-size: .82rem;
}
.seg button.active, .presets button:hover { background: #4f46e5; border-color: #4f46e5; color: white; }
.seg.seg3 { grid-template-columns: repeat(3, 1fr); }
.presets { display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px; }

/* devices */
.device-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; }
.device-btn {
  background: #0b1226; border: 1.5px solid #223052; color: #cbd5e1;
  border-radius: 12px; padding: 10px 6px; cursor: pointer;
  display: grid; justify-items: center; gap: 3px;
  transition: .15s;
}
.device-btn.active { border-color: #818cf8; background: #1a2145; box-shadow: 0 0 0 3px rgba(99,102,241,.2); }
.device-btn strong { font-size: .74rem; }
.device-btn small { font-size: .68rem; color: #8b94ad; }
.mini-phone {
  width: 22px; height: 44px; border-radius: 7px;
  background: #1f2937; border: 2px solid #475569;
  position: relative; display: block; margin-bottom: 4px;
}
.mini-phone i { position: absolute; background: #020617; display: block; }
.mini-phone.island i { top: 4px; left: 50%; transform: translateX(-50%); width: 10px; height: 4px; border-radius: 4px; }
.mini-phone.notch i { top: 0; left: 50%; transform: translateX(-50%); width: 12px; height: 5px; border-radius: 0 0 5px 5px; }
.mini-phone.hole i { top: 5px; left: 50%; transform: translateX(-50%); width: 4px; height: 4px; border-radius: 50%; }

/* swatches / scenes */
.swatches { display: flex; gap: 8px; }
.swatch {
  width: 34px; height: 34px; border-radius: 50%; cursor: pointer;
  border: 2px solid transparent;
  transition: .15s;
}
.swatch.active { border-color: #fff; box-shadow: 0 0 0 3px rgba(129,140,248,.5); transform: scale(1.08); }
.swatch.custom { background: conic-gradient(#f43f5e, #f59e0b, #10b981, #3b82f6, #8b5cf6, #f43f5e); display: grid; place-items: center; }
.finish-name { font-size: .8rem; color: #94a3b8; }
.color-full { width: 100%; height: 38px; border-radius: 9px; border: 1px solid #334155; background: #0b1226; padding: 3px; cursor: pointer; }
.scene-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; }
.scene-btn { background: transparent; border: none; cursor: pointer; display: grid; gap: 4px; justify-items: center; padding: 0; }
.scene-thumb {
  width: 100%; aspect-ratio: 1; border-radius: 11px; display: block;
  border: 2px solid transparent; transition: .15s;
}
.scene-btn.active .scene-thumb { border-color: #fff; box-shadow: 0 6px 18px rgba(0,0,0,.4); transform: translateY(-2px); }
.scene-btn small { font-size: .68rem; color: #94a3b8; }
.scene-btn.active small { color: #e0e7ff; }

/* sliders */
.slider { display: grid; gap: 4px; font-size: .8rem; color: #94a3b8; }
.slider b { color: #e2e8f0; float: right; }
.slider input[type='range'] { width: 100%; accent-color: #6366f1; cursor: pointer; }
.toggles { display: flex; flex-wrap: wrap; gap: 10px 16px; font-size: .82rem; color: #cbd5e1; }
.toggles input { accent-color: #6366f1; }
.field { display: grid; gap: 6px; font-size: .82rem; color: #94a3b8; }
.upload-row {
  background: #0b1226; border: 1px dashed #334155; border-radius: 10px;
  padding: 9px 10px; cursor: pointer; font-size: .8rem; color: #94a3b8;
  display: block; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}

/* ---------- stage ---------- */
.stage-wrap {
  position: relative;
  display: grid;
  grid-template-rows: auto minmax(0, 1fr) auto;
  min-height: 100vh;
  overflow: hidden;
  transition: background .3s;
}
.orb {
  position: absolute; border-radius: 50%; filter: blur(90px); opacity: .5; pointer-events: none;
}
.orb-a { width: 480px; height: 480px; top: -140px; left: -120px; }
.orb-b { width: 560px; height: 560px; bottom: -200px; right: -140px; }
.grain {
  position: absolute; inset: 0; pointer-events: none; opacity: .5; mix-blend-mode: overlay;
  background-image: radial-gradient(rgba(255,255,255,.14) 1px, transparent 1px);
  background-size: 22px 22px;
}
.stage-top {
  z-index: 2;
  display: flex; align-items: center; justify-content: space-between;
  gap: 12px; padding: 18px 26px;
}
.stage-info strong { display: block; font-size: 1.05rem; letter-spacing: -.01em; }
.stage-info span { font-size: .8rem; opacity: .65; }
.light-scene .stage-info { color: #0f172a; }
.stage-wrap:not(.light-scene) .stage-info { color: #f1f5f9; }
.primary {
  background: linear-gradient(135deg, #6366f1, #a855f7);
  border: none; color: white; font-weight: 700;
  padding: 12px 20px; border-radius: 12px; cursor: pointer;
  box-shadow: 0 12px 30px rgba(99,102,241,.45);
  transition: .15s;
  white-space: nowrap;
}
.primary:hover:not(:disabled) { transform: translateY(-2px); box-shadow: 0 16px 36px rgba(99,102,241,.55); }
.primary:disabled { opacity: .45; cursor: not-allowed; box-shadow: none; }

.stage {
  z-index: 1;
  display: grid; place-items: center;
  perspective: 1400px;
  padding: 30px 20px 8px;
  min-height: 0;
  overflow: hidden;
}
.stage.floating .tilt { animation: float 5s ease-in-out infinite; }
@keyframes float {
  0%, 100% { translate: 0 0; }
  50% { translate: 0 -10px; }
}
.tilt { transform-style: preserve-3d; transition: transform .12s ease-out; position: relative; }

/* realistic phone */
.phone {
  position: relative;
  padding: var(--frame-padding);
  border-radius: var(--frame-radius);
  background:
    linear-gradient(145deg, var(--frame-edge), var(--frame-base) 30%, var(--frame-dark) 70%, var(--frame-base));
  box-shadow:
    calc(var(--shadow) * 0.9px) calc(var(--shadow) * 0.7px) calc(var(--shadow) * 1.1px) rgba(0,0,0,.55),
    inset 0 1px 1px rgba(255,255,255,.55),
    inset 0 -2px 4px rgba(0,0,0,.5);
  border: 1px solid rgba(255,255,255,.18);
}
.phone::before {
  content: '';
  position: absolute; inset: 3px;
  border-radius: calc(var(--frame-radius) - 3px);
  border: 1px solid rgba(0,0,0,.55);
  box-shadow: inset 0 0 0 2px rgba(0,0,0,.85);
  pointer-events: none; z-index: 3;
}
.btn-side { position: absolute; background: var(--frame-dark); border-radius: 3px; box-shadow: inset 0 1px 1px rgba(255,255,255,.3); }
.b1 { left: -3px; top: 110px; width: 3px; height: 34px; }
.b2 { left: -3px; top: 152px; width: 3px; height: 56px; }
.b3 { left: -3px; top: 214px; width: 3px; height: 56px; }
.b4 { right: -3px; top: 170px; width: 3px; height: 78px; }
.screen {
  position: relative;
  width: 100%; height: 100%;
  border-radius: var(--screen-radius);
  overflow: hidden;
  background: #000;
  box-shadow: inset 0 0 0 2px #000;
}
.screen img { width: 100%; height: 100%; object-fit: cover; display: block; }
.glare {
  position: absolute; inset: 0; pointer-events: none; z-index: 2;
  background:
    linear-gradient(115deg, rgba(255,255,255,.14) 0%, rgba(255,255,255,.03) 18%, transparent 32%),
    radial-gradient(120% 60% at 80% 110%, transparent 62%, rgba(0,0,0,.28) 100%);
}
.screen::after {
  content: ''; position: absolute; inset: 0; z-index: 2; pointer-events: none;
  border-radius: inherit;
  box-shadow: inset 0 0 18px rgba(0,0,0,.55);
}
.island {
  position: absolute; top: 12px; left: 50%; transform: translateX(-50%);
  width: 32%; height: 24px; background: #000; border-radius: 14px; z-index: 2;
  box-shadow: inset 0 0 0 1px rgba(255,255,255,.06);
}
.island::after {
  content: ''; position: absolute; right: 10px; top: 50%; transform: translateY(-50%);
  width: 8px; height: 8px; border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #1e293b, #000 70%);
}
.notch {
  position: absolute; top: 0; left: 50%; transform: translateX(-50%);
  width: 44%; height: 24px; background: #000;
  border-bottom-left-radius: 15px; border-bottom-right-radius: 15px; z-index: 2;
}
.hole {
  position: absolute; top: 13px; left: 50%; transform: translateX(-50%);
  width: 11px; height: 11px; border-radius: 50%; background: #000; z-index: 2;
  box-shadow: 0 0 0 3px rgba(0,0,0,.6), inset 0 0 3px #1e293b;
}
.empty {
  width: 100%; height: 100%; min-height: 420px;
  display: grid; place-content: center; gap: 8px; text-align: center;
  background:
    radial-gradient(80% 50% at 50% 20%, rgba(99,102,241,.25), transparent 70%),
    linear-gradient(180deg, #0f172a, #020617);
  color: #64748b; padding: 24px;
}
.empty span {
  width: 54px; height: 54px; margin: 0 auto;
  display: grid; place-items: center;
  font-size: 26px; color: #a5b4fc;
  border: 1.5px dashed #475569; border-radius: 16px; background: rgba(99,102,241,.08);
}
.empty p { margin: 0; font-size: .86rem; line-height: 1.5; }
.floor-shadow {
  height: 34px; margin: 26px auto 0; width: 72%;
  background: radial-gradient(50% 50% at 50% 50%, rgba(0,0,0,.55), transparent 70%);
  filter: blur(10px);
}

/* asset mode */
.asset-phone { position: relative; filter: drop-shadow(0 30px 50px rgba(0,0,0,.5)); }
.asset-frame { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: contain; pointer-events: none; }
.asset-frame.underlay { z-index: 0; }
.asset-frame.overlay { z-index: 2; }
.asset-screen { position: absolute; overflow: hidden; background: #000; z-index: 1; border-radius: 6%; }
.asset-screen img { width: 100%; height: 100%; display: block; }

.stage-bottom {
  z-index: 2; padding: 14px 26px 18px;
  font-size: .8rem; opacity: .65;
}
.light-scene .stage-bottom { color: #334155; }
.stage-wrap:not(.light-scene) .stage-bottom { color: #cbd5e1; }

.mini-laptop {
  width: 34px; height: 24px; border-radius: 3px 3px 0 0;
  background: #1f2937; border: 2px solid #475569; border-bottom: none;
  position: relative; display: block; margin-bottom: 8px;
}
.mini-laptop::after {
  content: ''; position: absolute; left: -6px; right: -6px; bottom: -5px; height: 3px;
  background: #475569; border-radius: 0 0 4px 4px;
}
.mini-monitor {
  width: 32px; height: 20px; border-radius: 3px;
  background: #1f2937; border: 2px solid #475569;
  position: relative; display: block; margin-bottom: 10px;
}
.mini-monitor::after {
  content: ''; position: absolute; left: 50%; top: 100%; transform: translateX(-50%);
  width: 3px; height: 6px; background: #475569;
  box-shadow: 0 6px 0 -1px #475569;
}
.mini-phone.none i { display: none; }

/* ---------- laptop / monitor preview ---------- */
.laptop { position: relative; }
.laptop-screen {
  padding: 12px;
  border-radius: 18px;
  background: linear-gradient(145deg, var(--frame-edge), var(--frame-base) 30%, var(--frame-dark) 70%, var(--frame-base));
  border: 1px solid rgba(255,255,255,.18);
  box-shadow:
    calc(var(--shadow) * 0.9px) calc(var(--shadow) * 0.7px) calc(var(--shadow) * 1.1px) rgba(0,0,0,.55),
    inset 0 1px 1px rgba(255,255,255,.55);
}
.laptop-display {
  position: relative;
  aspect-ratio: 16 / 10;
  border-radius: 10px;
  overflow: hidden;
  background: #000;
}
.laptop-display img { width: 100%; height: 100%; object-fit: cover; display: block; }
.laptop-cam {
  position: absolute; top: 5px; left: 50%; transform: translateX(-50%);
  width: 6px; height: 6px; border-radius: 50%; background: #000; z-index: 2;
  box-shadow: 0 0 0 2px rgba(0,0,0,.5), inset 0 0 2px #1e293b;
}
.laptop-base {
  height: 13px; margin: 0 -5.5%;
  background: linear-gradient(180deg, var(--frame-edge), var(--frame-base) 45%, var(--frame-dark));
  clip-path: polygon(4% 0, 96% 0, 100% 100%, 0 100%);
  border-radius: 0 0 10px 10px;
  position: relative;
}
.laptop-base i {
  position: absolute; top: 0; left: 50%; transform: translateX(-50%);
  width: 12%; height: 5px; background: rgba(0,0,0,.4); border-radius: 0 0 6px 6px;
  display: block;
}
.monitor { position: relative; }
.monitor-screen {
  padding: 14px;
  border-radius: 20px;
  background: linear-gradient(145deg, var(--frame-edge), var(--frame-base) 30%, var(--frame-dark) 70%, var(--frame-base));
  border: 1px solid rgba(255,255,255,.18);
  box-shadow:
    calc(var(--shadow) * 0.9px) calc(var(--shadow) * 0.7px) calc(var(--shadow) * 1.1px) rgba(0,0,0,.55),
    inset 0 1px 1px rgba(255,255,255,.55);
  position: relative; z-index: 2;
}
.monitor-display {
  position: relative;
  aspect-ratio: 16 / 9;
  border-radius: 12px;
  overflow: hidden;
  background: #000;
}
.monitor-display img { width: 100%; height: 100%; object-fit: cover; display: block; }
.monitor-neck {
  width: 60px; height: 46px; margin: -4px auto 0;
  background: linear-gradient(180deg, var(--frame-base), var(--frame-dark));
  position: relative; z-index: 1;
}
.monitor-foot {
  width: 200px; height: 12px; margin: 0 auto;
  border-radius: 50%;
  background: radial-gradient(50% 120% at 50% 0%, var(--frame-edge), var(--frame-dark));
  position: relative; z-index: 1;
}
.tablet-cam {
  position: absolute;
  top: calc(var(--frame-padding) / 2 - 4px);
  left: 50%; transform: translateX(-50%);
  width: 8px; height: 8px; border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #1e293b, #000 70%);
  box-shadow: 0 0 0 2px rgba(0,0,0,.4);
  z-index: 4;
}
.floor-shadow.wide { width: 88%; }
.laptop-display .empty, .monitor-display .empty { min-height: 220px; }

@media (max-width: 1020px) {
  .app { grid-template-columns: 1fr; }
  .sidebar { max-height: none; order: 2; }
  .stage-wrap { min-height: 82vh; order: 1; }
  .stage-top { flex-direction: column; align-items: stretch; }
}
</style>
