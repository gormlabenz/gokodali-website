<template>
  <section class="shapes"></section>
</template>

<script setup>
import MatterAttractors from 'matter-attractors'

import Matter, {
  Engine,
  Render,
  Bodies,
  World,
  MouseConstraint,
  Composites,
  Runner,
  Mouse,
  Events,
  Body,
} from 'matter-js'

import krümel_0 from '@/assets/krümel/0.png'
import krümel_1 from '@/assets/krümel/1.png'
import krümel_2 from '@/assets/krümel/2.png'
import krümel_3 from '@/assets/krümel/3.png'
import krümel_4 from '@/assets/krümel/4.png'
import krümel_5 from '@/assets/krümel/5.png'
import krümel_6 from '@/assets/krümel/6.png'
import krümel_7 from '@/assets/krümel/7.png'
import krümel_8 from '@/assets/krümel/8.png'
import krümel_9 from '@/assets/krümel/9.png'

import dino_1 from '@/assets/dinos/1.png'
import dino_2 from '@/assets/dinos/2.png'
import dino_3 from '@/assets/dinos/3.png'
import dino_4 from '@/assets/dinos/4.png'

Matter.use(MatterAttractors)

const textures = [
  krümel_0,
  krümel_1,
  krümel_2,
  krümel_3,
  krümel_4,
  krümel_5,
  krümel_6,
  krümel_7,
  krümel_8,
  krümel_9,
]

const dinos = [dino_1, dino_2, dino_3, dino_4]

function initMatter() {
  const sectionTag = document.querySelector('section.shapes')

  //width and height of the page
  let w = window.innerWidth
  let h = window.innerHeight

  const engine = Engine.create()

  const renderer = Render.create({
    element: sectionTag,
    engine: engine,
    options: {
      width: w,
      height: h,
      background: '#fff',
      wireframes: false,
      pixelRatio: window.devicePixelRatio,
    },
  })

  window.addEventListener('resize', () => {
    // renderer.bounds.max.x = window.innerWidth
    // renderer.bounds.max.y = window.innerHeight
    renderer.options.width = window.innerWidth
    renderer.options.height = window.innerHeight
    renderer.options.pixelRatio = window.devicePixelRatio
    renderer.canvas.width = window.innerWidth
    renderer.canvas.height = window.innerHeight
  })

  let count = 0
  const createShape = function (x, y) {
    count = count + 1

    let texture = textures[count % textures.length]

    return Bodies.circle(x, y, 3, {
      frictionAir: 0.3,
      render: {
        sprite: {
          texture: texture,
          yScale: 0.3,
          xScale: 0.3,
        },
      },
    })
  }

  dinos.forEach((dino, index) => {
    const dinoBody = Bodies.rectangle(100 + index * 100, 100, 200, 200, {
      frictionAir: 1,
      render: {
        sprite: {
          texture: dino,
          yScale: 0.3,
          xScale: 0.3,
        },
      },
    })

    World.add(engine.world, dinoBody)
  })

  //Create a Static Element
  const bigBall = Bodies.circle(w / 2, h / 2, Math.min(w / 4, h / 4), {
    isStatic: true,
    render: {
      fillStyle: 'white',
      visible: true,
    },
  })

  // Create a wall for the shapes to bounce off
  const wallOptions = {
    isStatic: true,
    render: {
      visible: true,
    },
  }

  const ground = Bodies.rectangle(w / 2, h + 50, w + 100, 100, wallOptions)
  const ceiling = Bodies.rectangle(w / 2, -50, w + 100, 100, wallOptions)
  const leftWall = Bodies.rectangle(-50, h / 2, 100, h + 100, wallOptions)
  const rightWall = Bodies.rectangle(w + 50, h / 2, 100, h + 100, wallOptions)

  const mouseControl = MouseConstraint.create(engine, {
    element: sectionTag,
    constraint: {
      render: {
        visible: false,
      },
    },
  })

  const intialShapes = Composites.stack(
    10,
    10,
    10,
    10,
    200,
    200,
    function (x, y) {
      return createShape(x, y)
    }
  )

  World.add(engine.world, [
    //bigBall,
    ground,
    ceiling,
    leftWall,
    rightWall,
    mouseControl,
    intialShapes,
  ])

  document.addEventListener('click', function (event) {
    const shape = createShape(event.pageX, event.pageY)
    intialShapes.bodies.push(shape)
    World.add(engine.world, shape)
  })

  Runner.run(engine)
  Render.run(renderer)

  window.addEventListener('resize', function () {})

  // Add Gravity

  let time = 0
  const changeGravity = function () {
    time = time + 0.008

    engine.world.gravity.x = Math.sin(time)
    engine.world.gravity.y = Math.cos(time)

    requestAnimationFrame(changeGravity)
  }

  changeGravity()

  var world = engine.world
  world.gravity.scale = 0

  var attractiveBody = Bodies.circle(
    renderer.options.width / 2,
    renderer.options.height / 2,
    50,
    {
      isStatic: true,

      // example of an attractor function that
      // returns a force vector that applies to bodyB
      plugin: {
        attractors: [
          function (bodyA, bodyB) {
            return {
              x: (bodyA.position.x - bodyB.position.x) * 1e-6,
              y: (bodyA.position.y - bodyB.position.y) * 1e-6,
            }
          },
        ],
      },
    }
  )

  World.add(world, attractiveBody)

  // add mouse control
  var mouse = Mouse.create(renderer.canvas)

  Events.on(engine, 'afterUpdate', function () {
    if (!mouse.position.x) {
      return
    }

    // smoothly move the attractor body towards the mouse
    Body.translate(attractiveBody, {
      x: (mouse.position.x - attractiveBody.position.x) * 0.25,
      y: (mouse.position.y - attractiveBody.position.y) * 0.25,
    })
  })
}

onMounted(() => initMatter())
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>
