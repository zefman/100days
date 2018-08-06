<template>
  <day-wrapper :dayNum="2" title="Randomness">
    <template slot="description">
Day 2! Just an extension of what was learnt on day one really.

This time I have made the rectangles smaller, and you can regenate them randomly by clicking the canvas.

There are a lot more rectangles too!
    </template>

    <canvas ref="canvas" slot="experiment" @click="generate"></canvas>
  </day-wrapper>
</template>

<script>
import DayWrapper from '@/components/DayWrapper.vue'
import fragmentShaderSource from './fragmentShader.glsl'
import vertexShaderSource from './vertexShader.glsl'

let canvas
let gl
let vertexShader
let fragmentShader

function createShader (gl, type, source) {
  var shader = gl.createShader(type)
  gl.shaderSource(shader, source)
  gl.compileShader(shader)
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS)
  if (success) {
    return shader
  }
  console.log(gl.getShaderInfoLog(shader))
  gl.deleteShader(shader)
}

function createProgram (gl, vertexShader, fragmentShader) {
  var program = gl.createProgram()
  gl.attachShader(program, vertexShader)
  gl.attachShader(program, fragmentShader)
  gl.linkProgram(program)
  var success = gl.getProgramParameter(program, gl.LINK_STATUS)
  if (success) {
    return program
  }

  console.log(gl.getProgramInfoLog(program))
  gl.deleteProgram(program)
}

function resize (canvas) {
  // Lookup the size the browser is displaying the canvas.
  var displayWidth = canvas.clientWidth
  var displayHeight = canvas.clientHeight

  // Check if the canvas is not the same size.
  if (canvas.width !== displayWidth ||
      canvas.height !== displayHeight) {
    // Make the canvas the same size
    canvas.width = displayWidth
    canvas.height = displayHeight
  }
}

// Returns a random integer from 0 to range - 1.
function randomInt (range) {
  return Math.floor(Math.random() * range)
}

// Fills the buffer with the values that define a rectangle.

function setRectangle (gl, x, y, width, height) {
  var x1 = x
  var x2 = x + width
  var y1 = y
  var y2 = y + height

  // NOTE: gl.bufferData(gl.ARRAY_BUFFER, ...) will affect
  // whatever buffer is bound to the `ARRAY_BUFFER` bind point
  // but so far we only have one buffer. If we had more than one
  // buffer we'd want to bind that buffer to `ARRAY_BUFFER` first.

  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
    x1, y1,
    x2, y1,
    x1, y2,
    x1, y2,
    x2, y1,
    x2, y2]), gl.STATIC_DRAW)
}

export default {
  components: {
    DayWrapper
  },
  data () {
    return {
      colourLocation: null
    }
  },
  mounted () {
    // Code from https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html

    canvas = this.$refs['canvas']
    gl = canvas.getContext('webgl2')

    // Create both shaders
    vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource)
    fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource)

    // Create the program by adding in the pair of shaders
    const program = createProgram(gl, vertexShader, fragmentShader)

    // Find attribute location of our program
    const positionAttributeLocation = gl.getAttribLocation(program, 'a_position')
    const resolutionUniformLocation = gl.getUniformLocation(program, 'u_resolution')
    const colourLocation = gl.getUniformLocation(program, 'u_color')
    this.colourLocation = colourLocation

    // Attributes get their data from buffers so we need to create a buffer
    const positionBuffer = gl.createBuffer()

    // WebGL lets us manipulate many WebGL resources on global bind points. You can think of bind points as internal global variables inside WebGL. First you bind a resource to a bind point. Then, all other functions refer to the resource through the bind point. So, let's bind the position buffer.
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer)

    // Now we can put data in that buffer by referencing it through the bind point
    // three 2d points
    const positions = [
      10, 20,
      80, 20,
      10, 30,
      10, 30,
      80, 20,
      80, 30
    ]
    // gl.STATIC_DRAW tells WebGL we are not likely to change this data much.
    // Float32Array as needs to be strongly typed
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

    // Now that we've put data in a buffer we need to tell the attribute ('a_position') how to get data out of it. First we need to create a collection of attribute state called a Vertex Array Object.
    const vao = gl.createVertexArray()

    // And we need to make that the current vertex array so that all of our attribute settings will apply to that set of attribute state
    gl.bindVertexArray(vao)

    // Now we finally setup the attributes in the vertex array. First off we need to turn the attribute on. This tells WebGL we want to get data out of a buffer. If we don't turn on the attribute then the attribute will have a constant value.
    gl.enableVertexAttribArray(positionAttributeLocation)

    // Then we need to specify how to pull the data out
    const size = 2 // 2 components per iteration
    const type = gl.FLOAT // the data is 32bit floats
    const normalize = false // don't normalize the data
    const stride = 0 // 0 = move forward size * sizeof(type) each iteration to get the next position
    const offset = 0 // start at the beginning of the buffer
    gl.vertexAttribPointer(
      positionAttributeLocation, size, type, normalize, stride, offset)

    // Resize canvas
    resize(gl.canvas)

    // We need to tell WebGL how to convert from the clip space values we'll be setting gl_Position to back into pixels, often called screen space. To do this we call gl.viewport and pass it the current size of the canvas.
    gl.viewport(0, 0, gl.canvas.width, gl.canvas.height)

    // Tell it to use our program (pair of shaders)
    gl.useProgram(program)

    // Pass in the canvas resolution so we can convert from
    // pixels to clipspace in the shader
    gl.uniform2f(resolutionUniformLocation, gl.canvas.width, gl.canvas.height)

    // Set a random color.
    gl.uniform4f(colourLocation, Math.random(), Math.random(), Math.random(), 1)

    // Bind the attribute/buffer set we want.
    gl.bindVertexArray(vao)

    this.generate()
  },
  methods: {
    generate () {
      // Clear the canvas
      gl.clearColor(0, 0, 0, 0)
      gl.clear(gl.COLOR_BUFFER_BIT)

      // draw 50 random rectangles in random colors
      for (var ii = 0; ii < 1000; ++ii) {
        // Setup a random rectangle
        setRectangle(
          gl, randomInt(gl.canvas.width), randomInt(gl.canvas.height), 10, 10)

        // Set a random color.
        gl.uniform4f(this.colourLocation, Math.random(), Math.random(), Math.random(), 1)

        // Draw the rectangle.
        // execute our GLSL program.
        const primitiveType = gl.TRIANGLES
        const offset = 0
        const count = 6
        gl.drawArrays(primitiveType, offset, count)
      }
      // requestAnimationFrame(this.generate);
    }
  }
}
</script>

<style>
canvas {
  width: 100%;
  height: 80vh;
}
</style>
