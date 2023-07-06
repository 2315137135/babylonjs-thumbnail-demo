<template>
  <canvas ref="renderCanvas" class="hidden"></canvas>
  <div class="grid grid-cols-4 gap-8 p-4">
    <div v-for="data in viewData" class="w-32 h-32 bg-amber-300 ">
      <canvas @pointerenter="handleCanvasEnter"
              @pointerleave="handleCanvasLeave"
              ref="viewCanvases" class="h-full w-full block"></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import {onMounted, onUnmounted, ref} from "vue";
import {ArcRotateCamera, Color3, Engine, MeshBuilder, Scene, SceneLoader, Vector3} from "@babylonjs/core";


const renderCanvas = ref()
const viewCanvases = ref([])
const sceneMap = new WeakMap<HTMLElement, Scene>()
const viewData = [
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
  "",
]


function handleCanvasEnter(e: PointerEvent) {
  let view = engine.views.find(view => view.target === e.target)
  let scene = sceneMap.get(view.target)
  if (!view) return

  engine.views.forEach(e => {
    if (e !== view) {
      e.enabled = false
      sceneMap.get(e.target)?.detachControl()
    }
  })

  engine.inputElement = view.target
  scene.attachControl()
  view.enabled = true
}

function handleCanvasLeave(e: PointerEvent) {
  let view = engine.views.find(view => view.target === e.target)
  let scene = sceneMap.get(view.target)
  if (!view) return
  view.enabled = false
}

let engine: Engine
onMounted(() => {
  engine = new Engine(renderCanvas.value)
  const scene = new Scene(engine)
  scene.createDefaultCameraOrLight(true, true, true)

  // create views
  viewCanvases.value.forEach((canvas, index) => {
    let viewScene = new Scene(engine)
    let debugBox = MeshBuilder.CreateBox("debugBox", {}, viewScene)
    viewScene.clearColor = Color3.Random().toColor4()
    viewScene.createDefaultLight()

    let camera = new ArcRotateCamera("camera", 0, 1, 5, Vector3.Zero(), viewScene)
    camera.attachControl()
    sceneMap.set(canvas, viewScene)
    let view = engine.registerView(canvas, camera)

    let data = viewData[index]
    SceneLoader.ShowLoadingScreen = false
    SceneLoader.Append("", data)

    viewScene.executeWhenReady(() => {
      setTimeout(e => {
        view.enabled = false
      }, 300)
    })
  })

  const renderLoop = () => {
    if (engine.activeView) {
      let viewScene = sceneMap.get(engine.activeView.target)
      viewScene?.render()
    } else {
      scene.render()
    }
  }
  engine.runRenderLoop(renderLoop)

  onUnmounted(() => {
    engine.dispose()
  })
})
</script>
