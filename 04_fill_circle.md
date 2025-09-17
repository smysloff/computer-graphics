<!--- file: 04_fill_circle.md --->

# Лекция №4:<br>Закраска окружности (Fill Circle)

## Основная идея:

Мы не просто рисуем контур. Нам нужно заполнить все пиксели внутри него.

## Самый простой способ (на основе нашего алгоритма):

* Мы рисуем окружность по алгоритму Брезенхема, получая точки `(x, y)` на контуре.

* Для каждой такой точки мы рисуем не один пиксель, а горизонтальную линию между точками `(-x, y)` и `(x, y)` (и для всех симметричных октантов!).

## Почему это работает?

Окружность симметрична. Для каждой координаты `y` мы знаем минимальный и максимальный `x`, которые принадлежат окружности. Просто рисуем между ними линию!

## Модифицируем наш код:

В твоей функции `circle` уже есть процедура `draw`, которая рисует 8 точек по симметрии. Ее нужно заменить на процедуру, которая будет рисовать 8 горизонтальных линий!

```javascript

// Было:
const draw = (cx, cy, x, y) => {
  this.pixel(cx + x, cy + y)
  this.pixel(cx - x, cy + y)
  // ... и т.д. для 8 точек
}

// Станет:
const fill = (cx, cy, x, y) => {
  this.hline(cx - x, cy + y, cx + x) // Верхняя половина
  this.hline(cx - x, cy - y, cx + x) // Нижняя половина
  this.hline(cx - y, cy + x, cx + y) // Еще две линии для полного покрытия
  this.hline(cx - y, cy - x, cx + y)
}
```

Но для этого тебе понадобится функция `hline()`! Ее можно реализовать очень просто — циклом от `x0` до `x1` с вызовом `pixel()`.


## Задачи на сегодня:

* Реализуй функцию `hline(x0, y, x1)` для рисования горизонтальной линии. Она будет гораздо проще твоего `line()`, так как `y` не меняется.

* Создай функцию `fillCircle(cx, cy, r)` на основе кода `circle`, но с заменой процедуры `draw` на процедуру `fill`, которая использует `hline`.

* Протестируй. Нарисуй несколько закрашенных кругов разного радиуса и цвета. Убедись, что нет "дырок" и лишних пикселей.


## Решение

```javascript
class ColorRGBA {

  constructor(r, g, b, a = 0xff) {
    this.set(r, g, b, a)
  }

  set(r, g, b, a = 0xff) {
    this.r = r
    this.g = g
    this.b = b
    this.a = a
  }

}

class GC {

  constructor(width = 320, height = 180) {
    this.element = document.createElement('canvas')
    this.context = this.element.getContext('2d')
    this.element.width = width
    this.element.height = height
    this.clean()
    this.imageData = this.context.getImageData(0, 0, this.element.width, this.element.height)
    this.pixelStyle = new ColorRGBA(0xff, 0xff, 0xff)
  }

  mount(element) {
    element.append(this.element)
  }

  fill(bgcolor) {
    this.context.fillStyle = bgcolor
    this.clean()
  }

  clean() {
    this.context.fillRect(0, 0, this.element.width, this.element.height)
  }

  blit() {
    this.context.putImageData(this.imageData, 0, 0)    
  }

  setPixelStyle(r, g, b, a = 0xff) {
    this.pixelStyle.set(r, g, b, a)
  }

  putPixel(x, y) {
    if (x < 0 || x >= this.imageData.width) return
    if (y < 0 || y >= this.imageData.height) return
    const { r, g, b, a } = this.pixelStyle
    const index = (y * this.imageData.width + x) * 4
    this.imageData.data[index    ] = r
    this.imageData.data[index + 1] = g
    this.imageData.data[index + 2] = b
    this.imageData.data[index + 3] = a
  }

  drawLine(x0, y0, x1, y1) { // brasenham's line algorithm

    const { abs, sign } = Math

    let sx = sign(x1 - x0)
    let sy = sign(y1 - y0)

    let dx = abs(x1 - x0)
    let dy = abs(y1 - y0)

    const dx2 = dx * 2
    const dy2 = dy * 2

    let error = 0
    let x = x0
    let y = y0
    this.putPixel(x, y)
    if (dx >= dy) {
      while (x !== x1) {
        x += sx
        error += dy2
        if (error > dx) {
          y += sy
          error -= dx2
        }
        this.putPixel(x, y)
      }
    } else {
      while (y !== y1) {
        y += sy
        error += dx2
        if (error > dy) {
          x += sx
          error -= dy2
        }
        this.putPixel(x, y)
      }
    }
  }

  strokeCircle(cx, cy, r) {

    const stroke = (cx, cy, x, y) => {
      this.putPixel(cx + x, cy + y)
      this.putPixel(cx - x, cy + y)
      this.putPixel(cx + x, cy - y)
      this.putPixel(cx - x, cy - y)
      this.putPixel(cx + y, cy + x)
      this.putPixel(cx - y, cy + x)
      this.putPixel(cx + y, cy - x)
      this.putPixel(cx - y, cy - x)
    }

    let error = 1 - r

    let x = 0
    let y = r
    stroke(cx, cy, x, y)

    while (x <= y) {
      ++x
      if (error < 0) {
        error += 2 * x + 3
      } else {
        error += 2 * (x - y) + 5
        --y
      }
      stroke(cx, cy, x, y)
    }

  }

  fillCircle(cx, cy, r) {

    const fill = (cx, cy, x, y) => {
      this.drawStraightLine(cx - x, cy + y, cx + x, cy + y)
      this.drawStraightLine(cx - x, cy - y, cx + x, cy - y)
      this.drawStraightLine(cx - y, cy + x, cx + y, cy + x)
      this.drawStraightLine(cx - y, cy - x, cx + y, cy - x)
    }

    let error = 1 - r

    let x = 0
    let y = r
    fill(cx, cy, x, y)

    while (x <= y) {
      ++x
      if (error < 0) {
        error += 2 * x + 3
      } else {
        error += 2 * (x - y) + 5
        --y
      }
      fill(cx, cy, x, y)
    }

  }

  drawStraightLine(x0, y0, x1, y1) {
    let x = x0
    let y = y0
    if (y0 === y1) {
      while (x <= x1) {
        this.putPixel(x++, y)
      } 
    } else if (x0 === x1) {
      while (y <= y1) {
        this.putPixel(x, y++)
      }
    } else {
      throw new Error(
        `Can't draw straight line from (${x0},${y0}) to (${x1},${y1})`)
    }
  }
}

const gc = new GC(640, 360)
gc.mount(document.body)

const cx = gc.element.width * .5
const cy = gc.element.height * .5

gc.setPixelStyle(0xff, 0xff, 0xff)
gc.strokeCircle(cx, cy, 100)

gc.setPixelStyle(0x00, 0xff, 0x00)
gc.strokeCircle(cx, cy, 1)

gc.setPixelStyle(0x00, 0xff, 0xff)
gc.strokeCircle(196, 128, 64)

gc.setPixelStyle(0xff, 0x00, 0xff)
gc.drawStraightLine(25, 300, 500, 300)

gc.setPixelStyle(0xff, 0xff, 0x00)
gc.drawStraightLine(50, 50, 50, 325)

gc.setPixelStyle(0x23, 0xaa, 0x92)
gc.fillCircle(250, 250, 32)

gc.setPixelStyle(0xff, 0xff, 0xff)
gc.fillCircle(250, 250, 1)

gc.setPixelStyle(0xff, 0x00, 0x00)
gc.fillCircle(0, 0, 64)

gc.blit()
```
