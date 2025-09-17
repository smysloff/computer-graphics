
<!--- file: triangle rasterization --->

# Лекция №5:<br>Растеризация треугольников<br>Алгоритм scanline

## Почему треугольники?

Треугольник — минимальный многоугольник. Любую сложную модель можно разбить на треугольники. Это основа всего 3D!

## Основная идея алгоритма:

Мы не будем просто рисовать три линии. Мы найдем все пиксели внутри треугольника и закрасим их.

## Шаги алгоритма:

* Сортируем вершины по `Y`.

  * Пусть `v0.y <= v1.y <= v2.y`. Это разделит треугольник на два участка: верхний и нижний.

* Обрабатываем два участка:

  * Верхний участок (от `v0` до `v2` через `v1`): Здесь два ребра — левое и правое.

  * Нижний участок (от `v1` до `v2`): Здесь тоже два ребра — левое и правое.

* Интерполяция по ребрам:

  * Для каждого сканирующего `y`-уровня мы вычисляем `x`-координаты на левом и правом ребрах. Это даст нам начало и конец горизонтальной линии для закрашивания.

  * Закрашиваем сканирующие строки:
    * Для каждого `y` от `y_min` до `y_max` рисуем горизонтальную линию между `x_left` и `x_right`.

### Как вычислять `x` на ребрах?

Используем линейную интерполяцию!

Для ребра между `v0(x0,y0)` и `v1(x1,y1)` на текущем `y`:

`x = x0 + ( (y - y0) * (x1 - x0) ) / (y1 - y0)`

Важный нюанс:

Нужно аккуратно обрабатывать деление на ноль (горизонтальные ребра) и быть внимательным к точности.


## Задача для реализации:

* Реализуй метод `fillTriangle(x0, y0, x1, y1, x2, y2)` в нашем классе `GC`.

  * Вспомогательные функции: Тебе может понадобиться функция для сортировки вершин по `Y`.

  * Интерполяция: Напиши функцию, которая для данного ребра и `y` вернет `x`.

  * Сканирование: Пройди по всем `y` между верхней и нижней вершиной, вычисли `x` для левого и правого ребра, и используй `drawStraightLine`.


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

class Vec2 {

  constructor(x, y) {
    this.x = x
    this.y = y
  }
  
  asArray() {
    return [this.x, this.y]
  }
}

class Triangle {
  constructor(x0, y0, x1, y1, x2, y2) {
    const vertices = [
      new Vec2(x0, y0),
      new Vec2(x1, y1),
      new Vec2(x2, y2),
    ].sort((a, b) => b.y - a.y)
    this.v0 = vertices[0]
    this.v1 = vertices[1]
    this.v2 = vertices[2]
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
  
  drawLine(x0, y0, x1, y1) {

    let x = x0
    let y = y0

    if (y0 === y1) {
      while (x <= x1) {
        this.putPixel(x++, y)
      }
      return
    }
    
    if (x0 === x1) {
      while (y <= y1) {
        this.putPixel(x, y++)
      }
      return
    }
 
    const { abs, sign } = Math

    let sx = sign(x1 - x0)
    let sy = sign(y1 - y0)

    let dx = abs(x1 - x0)
    let dy = abs(y1 - y0)

    const dx2 = dx * 2
    const dy2 = dy * 2

    let error = 0

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
      this.drawLine(cx - x, cy + y, cx + x, cy + y)
      this.drawLine(cx - x, cy - y, cx + x, cy - y)
      this.drawLine(cx - y, cy + x, cx + y, cy + x)
      this.drawLine(cx - y, cy - x, cx + y, cy - x)
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

  strokeTriangle(x0, y0, x1, y1, x2, y2) {
    this.drawLine(x0, y0, x1, y1)
    this.drawLine(x1, y1, x2, y2)
    this.drawLine(x2, y2, x0, y0)
  }
  
  fillTriangle(x0, y0, x1, y1, x2, y2) {
    
    const { max, min, round } = Math

    const triangle = new Triangle(x0, y0, x1, y1, x2, y2)
    const { v0, v1, v2 } = triangle

    for (let y = v2.y; y <= v1.y; ++y) {
      const x = []
      x.push(round(v2.x + ((y - v2.y) * (v0.x - v2.x)) / (v0.y - v2.y)))
      x.push(round(v2.x + ((y - v2.y) * (v1.x - v2.x)) / (v1.y - v2.y)))
      this.drawLine(min(...x), y, max(...x), y)
    }
    
    for (let y = v1.y + 1; y <= v0.y; ++y) {
      const x = []
      x.push(round(v0.x + ((y - v0.y) * (v1.x - v0.x)) / (v1.y - v0.y)))
      x.push(round(v0.x + ((y - v0.y) * (v2.x - v0.x)) / (v2.y - v0.y)))
      this.drawLine(min(...x), y, max(...x), y)
    }
  }
  
}

const gc = new GC(640, 360)
gc.mount(document.body)

const cx = gc.element.width * .5
const cy = gc.element.height * .5

gc.setPixelStyle(0xff, 0xff, 0xff)
gc.strokeTriangle(50, 310, 270, 50, 590, 185)

gc.setPixelStyle(0x00, 0xff, 0x00)
gc.fillTriangle(120, 270, 280, 85, 500, 180)

gc.blit()
```
