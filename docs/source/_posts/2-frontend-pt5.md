---
title: 前端 | 学习 WebGL
date: 2020-7-15
tags: 
- JavaScript
- Frontend
categories: practice
photos:
    - /blog/img/javascript.jpg
---
<br>
<!--more-->

# show

<canvas id="draw" width="500" height="500"></canvas>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const canvas = document.getElementById('draw');
    const gl = canvas.getContext('webgl');

    gl.clearColor(1, 1, 1, 1);

    const sourceV = `
        attribute vec3 position;
        varying vec4 color;

        void main() {
            gl_Position = vec4(position, 1);
            color = gl_Position * 0.5 + 0.5;
        }
    `;

    const shaderV = gl.createShader(gl.VERTEX_SHADER);
    gl.shaderSource(shaderV, sourceV);
    gl.compileShader(shaderV);

    if (!gl.getShaderParameter(shaderV, gl.COMPILE_STATUS)) {
        console.error(gl.getShaderInfoLog(shaderV));
        throw new Error('Failed to compile vertex shader');
    }

    const sourceF = `
        precision mediump float;
        varying vec4 color;

        void main() {
            gl_FragColor = color;
        }
    `;

    const shaderF = gl.createShader(gl.FRAGMENT_SHADER);
    gl.shaderSource(shaderF, sourceF);
    gl.compileShader(shaderF);

    if (!gl.getShaderParameter(shaderF, gl.COMPILE_STATUS)) {
        console.error(gl.getShaderInfoLog(shaderF));
        throw new Error('Failed to compile fragment shader');
    }

    const program = gl.createProgram();
    gl.attachShader(program, shaderV);
    gl.attachShader(program, shaderF);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        console.error(gl.getProgramInfoLog(program));
        throw new Error('Failed to link program');
    }

    gl.useProgram(program);

    const positionsData = new Float32Array([
    -0.75, -0.65, -1,
     0.75, -0.65, -1,
     0   ,  0.65, -1,
    ]);

    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, positionsData, gl.STATIC_DRAW);

    const attribute = gl.getAttribLocation(program, 'position');
    gl.enableVertexAttribArray(attribute);
    gl.vertexAttribPointer(attribute, 3, gl.FLOAT, false, 0, 0);

    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLES, 0, 3);
});
</script>

# code

```html
<canvas id="draw" width="500" height="500"></canvas>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const canvas = document.getElementById('draw');
    const gl = canvas.getContext('webgl');

    gl.clearColor(1, 1, 1, 1);

    const sourceV = `
        attribute vec3 position;
        varying vec4 color;

        void main() {
            gl_Position = vec4(position, 1);
            color = gl_Position * 0.5 + 0.5;
        }
    `;

    const shaderV = gl.createShader(gl.VERTEX_SHADER);
    gl.shaderSource(shaderV, sourceV);
    gl.compileShader(shaderV);

    if (!gl.getShaderParameter(shaderV, gl.COMPILE_STATUS)) {
        console.error(gl.getShaderInfoLog(shaderV));
        throw new Error('Failed to compile vertex shader');
    }

    const sourceF = `
        precision mediump float;
        varying vec4 color;

        void main() {
            gl_FragColor = color;
        }
    `;

    const shaderF = gl.createShader(gl.FRAGMENT_SHADER);
    gl.shaderSource(shaderF, sourceF);
    gl.compileShader(shaderF);

    if (!gl.getShaderParameter(shaderF, gl.COMPILE_STATUS)) {
        console.error(gl.getShaderInfoLog(shaderF));
        throw new Error('Failed to compile fragment shader');
    }

    const program = gl.createProgram();
    gl.attachShader(program, shaderV);
    gl.attachShader(program, shaderF);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        console.error(gl.getProgramInfoLog(program));
        throw new Error('Failed to link program');
    }

    gl.useProgram(program);

    const positionsData = new Float32Array([
    -0.75, -0.65, -1,
     0.75, -0.65, -1,
     0   ,  0.65, -1,
    ]);

    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, positionsData, gl.STATIC_DRAW);

    const attribute = gl.getAttribLocation(program, 'position');
    gl.enableVertexAttribArray(attribute);
    gl.vertexAttribPointer(attribute, 3, gl.FLOAT, false, 0, 0);

    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLES, 0, 3);
});
</script>

```

# link


- 翻译：[最简WebGL教程，仅需 75 行代码](https://mp.weixin.qq.com/s/dIBe9oaPQaTdSBFPXeD8jg)

- 原文：[Barebones WebGL in 75 lines of code](https://avikdas.com/2020/07/08/barebones-webgl-in-75-lines-of-code.html)