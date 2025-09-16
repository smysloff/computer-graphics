# Лекция №1: "Холст, пиксели и первый луч света"

Сегодня основа основ. Любое изображение на экране — это сетка пикселей (буфер кадра). Наша задача — научиться управлять каждым из них.

1. HTML Canvas — это наш холст, область, где мы рисуем с помощью JavaScript.

```html
<canvas id="myCanvas" width="800" height="600"></canvas>
```

2. Контекст рисования — кисти, которыми мы будем работать.

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
```

3. Координаты. Начало (0,0) — левый верхний угол. Ось X — вправо, ось Y — вниз.

4. Первый пиксель. В Canvas нет функции для рисования одного пикселя, но мы можем нарисовать линию длиной в 1 пиксель или использовать массив пикселей (ImageData).


## Задачи на первый урок:

1. Разминка. Создай страницу с холстом 800x600. Сделай фон темно-серым (#222).

2. Точка. Напиши функцию drawPixel(x, y, color), которая будет ставить одну точку на холст. Пусть она в середине экрана нарисует ярко-красную точку.

3. Узор. Используя цикл и твою функцию drawPixel, нарисуй сетку из белых точек с шагом 10 пикселей по всей ширине и высоте холста.

4. Необязательное, со звездочкой: Попробуй сделать так, чтобы при движении мыши за ней тянулся след из случайно окрашенных пикселей.


## Решение

```pug
canvas#myCanvas(width='800' height='600')
```

```javascript

// get HTMLCanvasElement and CanvasRenderingContext2D

const canvas =
  document.getElementById('myCanvas')

const ctx = canvas.getContext('2d')


// standart canvas API

ctx.fillStyle = '#222'
ctx.fillRect(0, 0, canvas.width, canvas.height)


// pixel manipulation with canvas
// https://developer.mozilla.org/ru/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas

function drawPixel(imageData, x, y, r, g, b, a) {
  const index = y * (imageData.width * 4) + (x * 4)
  imageData.data[index + 0] = r
  imageData.data[index + 1] = g
  imageData.data[index + 2] = b
  imageData.data[index + 3] = a
}

// ImageData
//   .data - Uint8ClampedArray (RGBA)

const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)


// 1st task:
// red dot in the middle of the canvas

drawPixel(
  imageData,          // imageData
  canvas.width * .5,  // x
  canvas.height * .5, // y
  0xff, 0, 0, 0xff,   // r, g, b, a
)


// 2nd task:
// draw grid with 10 pixels space between lines

for (let y = 0; y <= canvas.height; y += 10) {
  for (let x = 0; x <= canvas.width; x += 10) {
    drawPixel(imageData, x, y, 0xff, 0xff, 0xff, 0xff)
  }
}

for (let x = 0; x <= canvas.width; x += 10) {
  for (let y = 0; y <= canvas.height; y += 10) {
    drawPixel(imageData, x, y, 0xff, 0xff, 0xff, 0xff)
  }
}


// 3rd task:
// draw random pixels as mouse tail *

canvas.addEventListener('mousemove', (event) => {
  const { clientX, clientY } = event
  const rect = canvas.getBoundingClientRect()
  const x = clientX - rect.left
  const y = clientY - rect.top
  const r = Math.floor(Math.random() * 256)
  const g = Math.floor(Math.random() * 256)
  const b = Math.floor(Math.random() * 256)
  const a = 0xff
  drawPixel(imageData, x, y, r, g, b, a)
  ctx.putImageData(imageData, 0, 0)
})


// blit to render results

ctx.putImageData(imageData, 0, 0)

```
