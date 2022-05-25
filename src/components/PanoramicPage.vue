<template>
  <div class="vrDiv" :style="{zIndex: showZIndex}">
    <el-button style="position:absolute; right: 2px" type="success" plain size="mini" @click="returnMap">返回</el-button>
    <div class="main_vr">
      <div class="vr_container" id="vr_container"></div>
      <video id="video" :loop="false" crossOrigin="anonymous" :autoplay="false"
             style="display:none;width: 0px;height: 0px" :src="videoUrl">
      </video>
      <!--        src="http://1259692143.vod2.myqcloud.com/6fda56ccvodcq1259692143/6479003d5285890815766187729/GMSMgREou44A.mp4"-->
      <!--    <video  loop crossOrigin="anonymous" :autoplay="false" playsinline style="width: 100%;height: 100%;" src="https://1259692143.vod2.myqcloud.com/6fda56ccvodcq1259692143/e9c1fd208602268010532095477/j0GlZUmBL8MA.mp4?uuid=2e3vtuuzaw4">-->
      <!--    </video>-->
    </div>
    <SceneSwitcher></SceneSwitcher>
  </div>
</template>

<script>
import * as THREE from 'three'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'
import imgUtl from  '../assets/img/backGround2.jpg'
import Stats from 'three/examples/jsm/libs/stats.module.js';
import SceneSwitcher from "./SceneSwitcher.vue";
// import Stats from './stats.min.js'
// import { OrbitControls } from './OrbitControls.js'
// , controls, raycaster , light
var container
var camera, scene, renderer, texture,controls,stats,geometry

// var spriteTL, spriteTR, spriteBL, spriteBR, spriteC, sceneOrtho, cameraOrtho
export default {
  name: 'PanoramicPage',
  components: {SceneSwitcher},
  props: {
    showZIndex: {
      type: Number,
      default: -1
    },
    videoUrl: {
      type: String,
      default: '../../public/img/backGround2.jpg'
    },
    showType: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      video: null,
      lon: 0,
      lat: 0,
      phi: 0,
      theta: 0,
      onPointerDownPointerY: 0,
      onPointerDownLon: 0,
      onPointerDownPointerX: 0,
      onPointerDownLat: 0
    }
  },
  watch: {
    handlerVideoUrl(newVideoUrl, oldVideoUrl) {
      console.log(newVideoUrl, oldVideoUrl)
      this.videoUrl = newVideoUrl
    }
  },
  created() {
    setTimeout( () => {
      this.init()
    },1000)
  },
  methods: {
    returnMap() {
      this.$emit('closeVR')
    },
    // three.js方法
    init( ) {
      // const app = document.querySelector('#app')
      // app.style.overflow = 'hidden'
      // this.videoUrl =  '../../public/img/backGround2.jpg'
      // this.showType = showType
      this.video = document.querySelector('#video')
      container = document.querySelector('#vr_container')
      console.log(this.video, container,window.innerWidth, window.innerHeight)


      stats = new Stats();
      container.appendChild(stats.dom);

      // 创建相机

      camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 1000)
      camera.target = new THREE.Vector3( 0, 0, 0 );

      // 创建场景
      scene = new THREE.Scene()
      // this.scene.background = new THREE.Color(0x101010)

      // 创建几何体
      geometry = new THREE.SphereGeometry(500, 100, 100)
      // invert the geometry on the x-axis so that all of the faces point inward

      geometry.scale(-1, 1, 1)

      renderer = new THREE.WebGLRenderer({ antialias: true })
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      container.appendChild(renderer.domElement)

      this.initLoadMesh()

      let Ambient = new THREE.AmbientLight(0xFFFFFF, 0.5);
      scene.add(Ambient);

      // canvas事件绑定
      renderer.domElement.onmousemove = this.canvasMouseMove
      renderer.domElement.onmouseout = this.canvasMouseOut
      renderer.domElement.onmouseup = this.canvasMouseup
      renderer.domElement.onmousedown = this.canvasMouseDown
      renderer.domElement.ontouchstart = this.canvasTouchstart
      renderer.domElement.ontouchmove = this.canvasTouchmove
      renderer.domElement.ontouchend = this.canvasTouchend

      window.addEventListener('resize', this.onWindowResize)

      setTimeout(() => {
        if (this.showType === 1) {
          this.video.play()
        }
        this.animate()
      }, 50)
    },
    initLoadMesh() {
      // if (this.showType === 1) {
      //   console.log(this.video)
      //   texture = new THREE.VideoTexture(this.video)
      //   // 组成物体
      //   const material = new THREE.MeshBasicMaterial({map: texture})
      //   const mesh = new THREE.Mesh(this.geometry, material)
      //   mesh.position.set(0, 0, 0)
      //   scene.add(mesh)
      // } else {
      //
      //   let  textureLoader = new THREE.TextureLoader();
      //   textureLoader.load( '../../public/img//backGround2.jpg',  ( texture ) =>  {
      //     texture.mapping = THREE.UVMapping;
      //     const options = {
      //       generateMipmaps: true,
      //       minFilter: THREE.LinearMipmapLinearFilter,
      //       magFilter: THREE.LinearFilter
      //     };
      //     scene.background = new THREE.WebGLCubeRenderTarget( 1024, options ).fromEquirectangularTexture( renderer, texture );
      //   } );
      //
      //   controls = new OrbitControls(camera, renderer.domElement);
      //   controls.target.set(0, -5, 0);
      //   controls.minDistance = 35;
      //   controls.maxDistance = 200;
      //
      //   controls.rotateSpeed = 1.0;
      //   controls.zoomSpeed = 1.2;
      //   controls.panSpeed = 0.8;
      // }

      if (this.showType === 1) {
        texture = new THREE.VideoTexture(this.video)
      } else {
        texture = new THREE.TextureLoader().load(this.videoUrl)
      }
      // 组成物体
      const material = new THREE.MeshBasicMaterial({ map: texture })
      const mesh = new THREE.Mesh(geometry, material)
      scene.add(mesh)
      mesh.position.set(0, 0, 0)
    },
    // 重绘
    animate() {
      this.rAfID = requestAnimationFrame(() => {
        this.animate()
      })
      this.update()
    },
    // 更新
    update() {
      this.lat = Math.max(-85, Math.min(85, this.lat))
      this.phi = THREE.MathUtils.degToRad(90 - this.lat)
      this.theta = THREE.MathUtils.degToRad(this.lon)
      camera.position.x = 50 * Math.sin(this.phi) * Math.cos(this.theta)
      camera.position.y = 50 * Math.cos(this.phi)
      camera.position.z = 50 * Math.sin(this.phi) * Math.sin(this.theta)
      camera.lookAt(camera.target)
      camera.position.copy(camera.target).negate()

      stats.update();
      renderer.render(scene, camera)
    },
    chengeVrSource(videoUrl, showType) {
      this.videoUrl = videoUrl
      this.showType = showType
      this.initLoadMesh()
      setTimeout(() => {
        if (this.showType === 1) this.video.play()
        this.animate()
      }, 20)
    },
    closeUpdate() {
      cancelAnimationFrame(this.rAfID)
      this.video.pause()
    },
    canvasMouseMove(event) {
      if (this.isMove) {
        this.lon = (this.onPointerDownPointerX - event.clientX) * 0.1 + this.onPointerDownLon
        this.lat = (this.onPointerDownPointerY - event.clientY) * 0.1 + this.onPointerDownLat
      }
    },
    canvasMouseOut(event) {
      if (this.isMove) {
        this.isMove = false
      }
    },
    canvasMouseup(event) {
      if (this.isMove) {
        this.isMove = false
      }
    },
    canvasMouseDown(event) {
      // if (this.showType) {
        this.isMove = true
      // }
      this.onPointerDownPointerX = event.clientX
      this.onPointerDownPointerY = event.clientY

      this.onPointerDownLon = this.lon
      this.onPointerDownLat = this.lat
    },

    // 触摸事件
    canvasTouchstart(event) {
      // this.flag3 = true
      // if (this.showType) {
        this.isMove = true
      // }

      event.preventDefault();
      this.isUserInteracting = true;
      this.onPointerDownPointerX = event.touches[0].clientX;
      this.onPointerDownPointerY = event.touches[0].clientY;

      this.onPointerDownLon = this.lon;
      this.onPointerDownLat = this.lat;

    },
    canvasTouchmove(event) {

      if (this.isMove === true) {

        this.lon = (this.onPointerDownPointerX - event.touches[0].clientX) * 0.1 + this.onPointerDownLon;
        this.lat = (this.onPointerDownPointerY - event.touches[0].clientY) * 0.1 + this.onPointerDownLat;

      }

    },
    canvasTouchend() {
      this.isMove = false;
      // this.flag3 = false
    },
    // 拖动canvas，动态改变选项的位置
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
    }
  }
}
</script>

<style lang="scss">
.vrDiv {
  width: 100%;
  height: 100%;
  position: absolute;
  background: rosybrown;

  .main_vr {

  }
}

.vr_container {
  width: 99%;
  height: 99%;
  left: 0;
  top: 0;
}
</style>
