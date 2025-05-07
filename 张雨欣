<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>P5.js Effects Demo with Color Selector & Pie</title>
  <!-- 引入 p5.js + DOM 扩展 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/addons/p5.dom.min.js"></script>
  <style>
    body { margin:0; overflow:hidden; }
    #controls {
      position:absolute; top:10px; left:10px; z-index:10;
      background:#fff; padding:10px; border-radius:5px;
      font-family:sans-serif; line-height:1.5;
    }
    #color-pool { margin-top:5px; display:flex; flex-wrap:wrap; max-width:300px; }
    .color-swatch {
      width:24px; height:24px; margin:2px;
      border:2px solid #ccc; cursor:pointer;
    }
    .selected { border-color:#000; }
    #apply-btn, #clear-btn { margin-top:5px; display:block; }
  </style>
</head>
<body>
  <div id="controls">
    <label>Upload Image:
      <input type="file" id="file-input" accept="image/*">
    </label>
    <button id="clear-btn">Clear Effects</button>
    <div id="color-pool"></div>
    <button id="apply-btn">Apply 5 Colors</button>
  </div>

  <script>
    // 全局状态
    let effects = [];
    let colors = ['#eac435','#345995','#e40066','#fb4d3d','#000000']; // 当前生效的 5 色
    let extracted = [];       // 提取出的 10 色
    let colorCounts = [];     // 每个提取色的像素计数
    let selected = new Set(); // 用户选中的索引

    function setup() {
      createCanvas(900, 900);
      rectMode(CENTER);
      // 绑定 DOM 事件
      select('#file-input').input(handleFile);
      select('#clear-btn').mousePressed(() => effects = []);
      select('#apply-btn').mousePressed(applyColors);
    }

    function draw() {
      background(230);
      // 运行并清理所有效果组
      effects.forEach(e => e.run());
      for (let i = effects.length - 1; i >= 0; i--) {
        if (effects[i].isDead) effects.splice(i, 1);
      }
      // 每隔 10 帧，产生一个新的效果组
      if (frameCount % 10 === 0) {
        effects.push(new EffectGroup(random(width), random(height)));
      }
      // 绘制右下角颜色比例扇形图
      drawPieChart();
    }

    // 文件上传回调
    function handleFile() {
      const file = this.elt.files[0];
      if (file && file.type.startsWith('image')) {
        loadImage(URL.createObjectURL(file), img => {
          img.resize(100, 0);
          extractColors(img);
        });
      }
    }

    // 提取前 10 主色并记录每色出现次数
    function extractColors(img) {
      img.loadPixels();
      const mapCount = {};
      const step = 5;
      for (let y = 0; y < img.height; y += step) {
        for (let x = 0; x < img.width; x += step) {
          const i = 4 * (y * img.width + x);
          let r = Math.round(img.pixels[i]   / 32) * 32;
          let g = Math.round(img.pixels[i+1] / 32) * 32;
          let b = Math.round(img.pixels[i+2] / 32) * 32;
          r = constrain(r,0,255);
          g = constrain(g,0,255);
          b = constrain(b,0,255);
          const key = `${r},${g},${b}`;
          mapCount[key] = (mapCount[key] || 0) + 1;
        }
      }
      // 排序取前 10
      const sorted = Object.entries(mapCount)
        .sort((a,b) => b[1] - a[1])
        .slice(0,10);
      extracted = sorted.map(([k])=>{
        const [r,g,b] = k.split(',').map(Number);
        return '#' + hex(r,2) + hex(g,2) + hex(b,2);
      });
      colorCounts = sorted.map(([_,count])=> count );
      // 重置选择并渲染 Swatches
      selected.clear();
      renderSwatches();
    }

    // 渲染 10 个可点击色块
    function renderSwatches() {
      const pool = select('#color-pool');
      pool.html('');
      extracted.forEach((c,i) => {
        const sw = createDiv();
        sw.addClass('color-swatch');
        sw.style('background', c);
        sw.attribute('data-index', i);
        sw.mousePressed(() => toggleSelect(i, sw));
        pool.child(sw);
      });
    }

    // 切换选中状态（最多 5 个）
    function toggleSelect(i, el) {
      if (selected.has(i)) {
        selected.delete(i);
        el.removeClass('selected');
      } else if (selected.size < 5) {
        selected.add(i);
        el.addClass('selected');
      }
    }

    // 将选中的 5 色 应用为 主色数组
    function applyColors() {
      if (selected.size === 5) {
        colors = Array.from(selected).map(i => extracted[i]);
      } else {
        alert('Please select exactly 5 colors.');
      }
    }

    // 在画布右下角绘制扇形图
    function drawPieChart() {
      const cx = width - 100;
      const cy = height - 100;
      const r  = 80;
      // 计算总选中计数
      let total = 0;
      selected.forEach(i => total += colorCounts[i]);
      if (total === 0) return;  // 无选中则不画

      let start = 0;
      selected.forEach(i => {
        const count = colorCounts[i];
        const angle = (count / total) * TWO_PI;
        noStroke();
        fill(extracted[i]);
        arc(cx, cy, 2*r, 2*r, start, start + angle, PIE);
        start += angle;
      });
      // 外圈白边
      stroke(255);
      strokeWeight(2);
      noFill();
      ellipse(cx, cy, 2*r, 2*r);
    }

    // —— 以下为原效果代码，无改动 ——
    function easeOutCubic(x) { return 1 - pow(1 - x, 3); }
    function easeOutExpo(x) { return x === 1 ? 1 : 1 - pow(2, -10 * x); }
    function easeInQuart(x) { return x * x * x * x; }

    class EffectGroup {
      constructor(x,y) {
        this.x = x; this.y = y;
        this.phase = 0; this.isDead = false;
        this.agents = [
          new Raindrop(x,y, int(random(90,120)), random(colors))
        ];
      }
      run() {
        this.agents.forEach(a => a.run());
        this.agents = this.agents.filter(a => !a.isDead);
        if (this.phase === 0 && this.agents.length === 0) {
          this.phase = 1;
        }
        if (this.phase === 1) {
          this.agents.push(new Donut(this.x,this.y, int(random(40,50)), random(colors)));
          for (let i = 0; i < 50; i++) {
            this.agents.push(new Splash(this.x,this.y, int(random(90,120))));
          }
          this.phase = 2;
        }
        if (this.phase === 2 && this.agents.length === 0) {
          this.isDead = true;
        }
      }
    }

    class Agent {
      constructor(x,y,life) {
        this.x = x; this.y = y;
        this.life = life; this.maxLife = life;
        this.isDead = false; this.amount = 0;
      }
      update() { this.amount = map(this.life, 0, this.maxLife, 1, 0); }
      checkLife() { if (--this.life <= 0) this.isDead = true; }
      run() {
        this.show();
        this.move();
        this.checkLife();
        this.update();
      }
      show() {}
      move() {}
    }

    class Raindrop extends Agent {
      constructor(x,y,life,clr) {
        super(x,y,life);
        this.w = width * random(0.01,0.02);
        this.h = this.w * 2.5;
        this.shiftY = -height/2;
        this.clr = clr;
      }
      show() {
        push();
        translate(this.x, this.y);
        noStroke(); fill(this.clr);
        ellipse(
          0,
          this.shiftY * pow(1 - this.amount, 3),
          this.w * this.amount,
          this.h * this.amount
        );
        pop();
      }
    }

    class Splash extends Agent {
      constructor(x,y,life) {
        super(x,y,life);
        this.type = int(random(4));
        this.w = width * random(0.01,0.05);
        let r = this.w * random(1,4),
            a = random(TAU);
        this.xShift = r * cos(a);
        this.yShift = r * sin(a);
        this.clr = random(colors);
        this.rev = TAU * random(10) * random(-1,1);
      }
      show() {
        push();
        translate(this.x, this.y);
        noFill(); stroke(this.clr);
        let t = this.amount;
        switch (this.type) {
          case 0:
            circle(
              this.xShift * easeOutExpo(t),
              this.yShift * easeOutExpo(t),
              this.w * (1 - easeOutCubic(t))
            );
            break;
          case 1:
            strokeWeight(this.w * 0.1 * (1 - t));
            line(
              this.xShift * easeOutExpo(t),
              this.yShift * easeOutExpo(t),
              this.xShift * easeOutCubic(t),
              this.yShift * easeOutCubic(t)
            );
            break;
          case 2:
            if (random() < 0.5) {
              circle(
                this.xShift * easeOutExpo(t),
                this.yShift * easeOutExpo(t),
                width * 0.005 * (1 - t)
              );
            }
            break;
          case 3:
            let ang = this.rev * t,
                rad = this.w * (1 - easeOutCubic(t));
            point(
              this.xShift * easeOutCubic(t) + rad * cos(ang),
              this.yShift * easeOutCubic(t) + rad * sin(ang)
            );
            break;
        }
        pop();
      }
    }

    class Donut extends Agent {
      constructor(x,y,life,clr) {
        super(x,y,life);
        this.d = width * random(0.05,0.08);
        this.clr = clr;
      }
      show() {
        push();
        translate(this.x, this.y);
        noStroke(); fill(this.clr);
        beginShape();
          for (let a = 0; a < TAU; a += TAU/360) {
            let r = (this.d/2) * easeOutExpo(this.amount);
            vertex(r * cos(a), r * sin(a));
          }
        beginContour();
          for (let a = TAU; a > 0; a -= TAU/360) {
            let r = (this.d/2) * easeInQuart(this.amount);
            vertex(r * cos(a), r * sin(a));
          }
        endContour();
        endShape();
        pop();
      }
    }
  </script>
</body>
</html>
