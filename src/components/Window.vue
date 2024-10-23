<template>
  <canvas ref="canvasRef" width="400" height="300"></canvas>
</template>

<script setup lang="ts" name="Window">
  import { ref, onMounted  } from "vue";
  import { mat4 } from "gl-matrix";

  const canvasRef = ref<HTMLCanvasElement | null>(null);
  const projectionMat = mat4.create();
  let webgl: WebGLRenderingContext | null;
  let program: WebGLProgram | null;
  
  const vertexString = `
    attribute vec4 aPosition;
    uniform mat4 projectionMat;

    void main()
    {
      gl_Position = projectionMat * aPosition;
      gl_PointSize = 60.0;
    }
  `;

  const fragmentString = `
    void main() 
    {
      gl_FragColor = vec4(0, 0, 1.0, 1.0);
    }
  `;
  onMounted(() => {
    init();
  })

  function init() {
    initWebGL();
    initShader();
    initBuffer();
    draw();
  }

  function initWebGL() {
    if (!canvasRef.value) return ;
    webgl = canvasRef.value.getContext("webgl");
    if (!webgl) {
      console.error("WebGL not supported");
      return ;
    }

    webgl.viewport(0, 0, canvasRef.value.width, canvasRef.value.height);
    mat4.ortho(
      projectionMat,
      0,
      canvasRef.value.clientWidth,
      canvasRef.value.clientHeight,
      0,
      -1,
      1
    );
  }

  function initShader() {
    if (!webgl) return ;

    const vertexShader = webgl.createShader(webgl.VERTEX_SHADER);
    const fragmentShader = webgl.createShader(webgl.FRAGMENT_SHADER);

    if (!vertexShader || !fragmentShader) return ;

    webgl.shaderSource(vertexShader, vertexString);
    webgl.shaderSource(fragmentShader, fragmentString);

    webgl.compileShader(vertexShader);
    webgl.compileShader(fragmentShader);

    program = webgl.createProgram();
    if (!program) return ;
    webgl.attachShader(program, vertexShader);
    webgl.attachShader(program, fragmentShader);
    webgl.linkProgram(program);
    webgl.useProgram(program);
  }

  function initBuffer() {
    if (!webgl || !program) return ;

    const pointPosition = new Float32Array([100.0, 100.0, 0.0, 1.0]);
    const attributePosition = webgl.getAttribLocation(program, "aPosition");
    webgl.vertexAttrib4fv(attributePosition, pointPosition);

    const uniformProjectionMat = webgl.getUniformLocation(program, "projectionMat");
    webgl.uniformMatrix4fv(uniformProjectionMat, false, projectionMat);

  }

  function draw() {
    if (!webgl) return ;

    webgl.clearColor(0.0, 0.0, 0.0, 1.0);
    webgl.clear(webgl.COLOR_BUFFER_BIT);
    webgl.drawArrays(webgl.POINTS, 0, 1);
  }
</script>

<style scoped lang="scss">

</style>