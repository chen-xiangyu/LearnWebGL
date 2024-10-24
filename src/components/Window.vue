<template>
  <canvas ref="canvasRef" width="800" height="600" @mousedown="handleMouseDown"></canvas>
</template>

<script setup lang="ts" name="Window">
  import { ref, onMounted  } from "vue";
  import { mat4 } from "gl-matrix";

  const canvasRef = ref<HTMLCanvasElement | null>(null);
  const projectionMat = mat4.create();
  let webgl: WebGLRenderingContext | null;
  let program: WebGLProgram | null;
  
  const vertexString = `
    attribute vec3 aPosition;
    attribute vec3 aColor;

    varying vec3 vColor;

    uniform mat4 projectionMat;

    void main()
    {
      gl_Position = projectionMat * vec4(aPosition, 1.0);
      gl_PointSize = 10.0;
      vColor = aColor;
    }
  `;

  const fragmentString = `
    precision mediump float;
    varying vec3 vColor;

    void main() 
    {
      gl_FragColor = vec4(vColor, 1.0);
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
      canvasRef.value.width,
      canvasRef.value.height,
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

    if (!webgl.getShaderParameter(vertexShader, webgl.COMPILE_STATUS)) {
      let err = webgl.getShaderInfoLog(vertexShader);
      console.error(err);
      return ;
    }
    if (!webgl.getShaderParameter(fragmentShader, webgl.COMPILE_STATUS)) {
      let err = webgl.getShaderInfoLog(fragmentShader);
      console.error(err);
      return ;
    }

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

  let points = [];
  let colors = [];

  const handleMouseDown = (e: MouseEvent) => {
    const x = e.clientX;
    const y = e.clientY;
    const rect = (e.target as HTMLElement).getBoundingClientRect();
    const pointX = x - rect.left;
    const pointY = y - rect.top;
    points.push(pointX, pointY, 0);
    if (!canvasRef.value) return ;
    colors.push(pointX / canvasRef.value.width, pointY / canvasRef.value.height, 0.5);

    const pointPosition = new Float32Array(points);
    const pointBuffer = webgl?.createBuffer();
    if (!webgl || !pointBuffer || !program) return ;
    webgl.bindBuffer(webgl.ARRAY_BUFFER, pointBuffer);
    webgl.bufferData(webgl.ARRAY_BUFFER, pointPosition, webgl.STATIC_DRAW);
    
    const attributePosition = webgl.getAttribLocation(program, "aPosition");
    webgl.enableVertexAttribArray(attributePosition);
    webgl.vertexAttribPointer(attributePosition, 3, webgl.FLOAT, false, 0, 0);

    let pointColor = new Float32Array(colors);
    let pointColorBuffer = webgl.createBuffer();
    webgl.bindBuffer(webgl.ARRAY_BUFFER, pointColorBuffer);
    webgl.bufferData(webgl.ARRAY_BUFFER, pointColor, webgl.STATIC_DRAW);
    const attributeColor = webgl.getAttribLocation(program, "aColor");
    webgl.enableVertexAttribArray(attributeColor);
    webgl.vertexAttribPointer(attributeColor, 3, webgl.FLOAT, false, 0, 0);

    const uniformProjectionMat = webgl.getUniformLocation(program, "projectionMat");
    webgl.uniformMatrix4fv(uniformProjectionMat, false, projectionMat);

    webgl.clearColor(0.0, 0.0, 0.0, 1.0);
    webgl.clear(webgl.COLOR_BUFFER_BIT | webgl.DEPTH_BUFFER_BIT);
    webgl.drawArrays(webgl.POINTS, 0, points.length / 3);
  }

</script>

<style scoped lang="scss">

</style>