<template>
  <div ref="container"></div>
</template>

<script setup>
import { onMounted,ref } from 'vue';
import * as THREE from 'three'
import {gsap} from 'gsap'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
const scene=new THREE.Scene()
const camera=new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000)
camera.position.set(0,0,0)
scene.add(camera)
const renderer=new THREE.WebGLRenderer({antialias:true})
renderer.setSize(window.innerWidth,window.innerHeight)

// const controls=new OrbitControls(camera,renderer.domElement)

const helper=new THREE.AxesHelper(5)
scene.add(helper)
const container=ref(null)



function animate(){
  renderer.render(scene,camera)
  requestAnimationFrame(animate)
}


onMounted(()=>{
  
 
  
  container.value.appendChild(renderer.domElement)
  animate()
  let isMouseDown=false
  container.value.addEventListener('mousedown',()=>{
    isMouseDown=true
  },false)
  container.value.addEventListener('mouseup',()=>{
    isMouseDown=false
  },false)
  container.value.addEventListener('mouseout',()=>{
    isMouseDown=false
  })
  //是否按下鼠标
  container.value.addEventListener('mousemove',(event)=>{
    if(isMouseDown){
      camera.rotation.y+=event.movementX*0.002
      camera.rotation.x+=event.movementY*0.002
      camera.rotation.order='YXZ'
    }
  })
  //创建客厅
  let liveroom=new Room('客厅',4,'./imgs/living/')

  
  //创建厨房
  let kitEuler=new THREE.Euler(0,-Math.PI/2,0)
  let kitchenPosition=new THREE.Vector3(-5,0,-10)
  let kitchen=new Room('厨房',0,'./imgs/kitchen/',kitchenPosition,kitEuler)
  //创建厨房精灵文字
  let kitchenTextPosition=new THREE.Vector3(0,0,-4)
  let kitchenText=new SpriteText('厨房',kitchenTextPosition)
  kitchenText.onClick(()=>{
    console.log('厨房')
    gsap.to(camera.position,{
      x:kitchenPosition.x,
      y:kitchenPosition.y,
      z:kitchenPosition.z,
      duration:1
    })
    //让相机移动到厨房
  })
  //创建厨房回客厅精灵文字
  let kitchenBackTextPosition=new THREE.Vector3(-8,0,-12)
  let kitchenBackText=new SpriteText('客厅',kitchenBackTextPosition)
  kitchenBackText.onClick(()=>{
    //让相机移动到客厅
    gsap.to(camera.position,{
      duration:1,
      x:0,
      y:0,
      z:0
    })
  })
  //创建阳台
  let restroomPosition=new THREE.Vector3(0,0,-20)
  let restroomEular=new THREE.Euler(0,-Math.PI/2,0)
  let restroom=new Room('厕所',7,'./imgs/restroom/',restroomPosition,restroomEular)
  //创建厕所精灵文字
  let restroomTextPosition=new THREE.Vector3(-2,0,-4)
  let restroomText=new SpriteText('厕所',restroomTextPosition)
  restroomText.onClick(()=>{
    //让相机移动到厕所
    gsap.to(camera.position,{
      duration:1,
      x:restroomPosition.x,
      y:restroomPosition.y,
      z:restroomPosition.z
    })
  })
})

class Room{
  constructor(name,roomIndex,textureUrl,position=new THREE.Vector3(0,0,0),euler=new THREE.Euler(0,0,0)){
     
    this.name=name
    //创建立方体
    const geometry=new THREE.BoxGeometry(10,10,10)
    geometry.scale(1,1,-1)

    var arr=[
      `${roomIndex}_l`,
      `${roomIndex}_r`,
      `${roomIndex}_u`,
      `${roomIndex}_d`,
      `${roomIndex}_b`,
      `${roomIndex}_f`
    ]
    let boxMaterials=[]
    arr.forEach((item)=>{
      //纹理加载
      const texture=new THREE.TextureLoader().load(textureUrl+item+'.jpg')
      if(item===`${roomIndex}_d`||item===`${roomIndex}_u`){
        texture.rotation=Math.PI
        texture.center=new THREE.Vector2(0.5,0.5)
      }
      boxMaterials.push(new THREE.MeshBasicMaterial({map:texture}))
    })
    const cube=new THREE.Mesh(geometry,boxMaterials)
    cube.position.copy(position)
    cube.rotation.copy(euler)
    scene.add(cube)
  }
}
class SpriteText{
  constructor(text,position){
    this.callbacks=[]
    const canvas=document.createElement('canvas')
    canvas.width=1024
    canvas.height=1024
    const context=canvas.getContext('2d')
    context.fillStyle='rgba(100,100,100,0.7)'
    context.fillRect(0,256,1024,512)
    context.textAlign='center'
    context.textBaseline='middle'
    context.font='bold 200px Arial'
    context.fillStyle='white'
    context.fillText(text,512,512)
    let texture=new THREE.CanvasTexture(canvas)
    const material=new THREE.SpriteMaterial({
      map:texture,
      transparent:true
    })
    const sprite=new THREE.Sprite(material)
    sprite.position.copy(position)
    sprite.scale.set(0.5,0.5,0.5)
    this.sprite=sprite
    scene.add(sprite)
    let mouse=new THREE.Vector2()
    let raycaster=new THREE.Raycaster()
    window.addEventListener('click',(event)=>{
     
      mouse.x=(event.clientX/window.innerWidth)*2-1
      mouse.y=1-(event.clientY/window.innerHeight)*2
      raycaster.setFromCamera(mouse,camera)
      let intersects=raycaster.intersectObject(sprite)
      if(intersects.length>0){
      this.callbacks.forEach((callback)=>{
        callback()
      })
      }
     
    })
  }
  onClick(callback){

    this.callbacks.push(callback)
  }
}
</script>

<style lang="scss">
*{
  margin:0;
  padding:0;
}
canvas{
  position:fixed;
  width:100vw;
  height: 100vh;
  top:0;
  left:0;
}
</style>
