<template>
  <div class="container" :style="canva_holder_style">
    <div class="canva_holder">
      <canvas ref="color_picker" class="color_picker"></canvas>
      <canvas ref="color_picker_slider" class="color_picker_slider"></canvas>
      <canvas ref="alpha_picker_slider" class="color_picker_slider"></canvas>
    </div>
    <div>
      <span>RGBA</span>
      {{ corEscolhidaRGB }}
    </div>
  </div>
</template>

<script>
export default {
  name: "ColorPicker",
  props: {
    msg: String,
    canva_len: String,
  },
  data() {
    return {
      provider1: {
        context: null,
      },
      provider2: {
        context: null,
      },
      provider3: {
        context: null,
      },
      color_picker_width: this.canva_len,
      color_picker_height: this.canva_len,
      color_picker_slider_width: (this.canva_len * 0.1618) / 3,
      color_picker_slider_height: this.canva_len,
      hue: Math.random(),
      sat: Math.random(),
      light: Math.random(),
      alpha: Math.random(),

      R: 0,
      G: 0,
      B: 0,
      canva_holder_style: {
        backgroundColor: "#fff",
        width: this.canva_len * 1.25 + "px",
        height: this.canva_len * 1.25 + "px",
      },

      corEscolhidaHSL: "",
      corEscolhidaRGB: "",

      pickedColor: false,
      pickedColorSlider: false,
      pickedAlphaSlider: false,
    };
  },
  provide() {
    return {
      provider: this.provider,
    };
  },

  methods: {
    hslToRgb(h, s, l) {
      let r, g, b;

      if (s === 0) {
        r = g = b = l; // achromatic
      } else {
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
        const p = 2 * l - q;

        r = this.hueToRgb(p, q, h + 1 / 3);
        g = this.hueToRgb(p, q, h);
        b = this.hueToRgb(p, q, h - 1 / 3);
      }

      return [Math.round(r * 255), Math.round(g * 255), Math.round(b * 255)];
    },

    hueToRgb(p, q, t) {
      if (t < 0) t += 1;
      if (t > 1) t -= 1;
      if (t < 1 / 6) return p + (q - p) * 6 * t;
      if (t < 1 / 2) return q;
      if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
      return p;
    },
    drawCircle(ctx, x, y, radius, fill, stroke, strokeWidth) {
      ctx.beginPath();
      ctx.arc(x, y, radius, 0, 2 * Math.PI, false);
      if (fill) {
        ctx.fillStyle = fill;
        ctx.fill();
      }
      if (stroke) {
        ctx.lineWidth = strokeWidth;
        ctx.strokeStyle = stroke;
        ctx.stroke();
      }
    },
    listenSlider(event) {
      const rect = this.$refs["color_picker_slider"].getBoundingClientRect();
      const hue = event.clientY - rect.top;
      this.hue = hue / this.color_picker_slider_height;
      const ctx = this.provider1.context;
      ctx.clearRect(
        0,
        0,
        this.color_picker_slider_width,
        this.color_picker_slider_height
      );
      this.reloadColorpickerSlider();
      ctx.restore();

      this.drawCircle(
        ctx,
        this.color_picker_slider_width / 2,
        hue,
        this.color_picker_slider_width / 2,
        "transparent",
        "white",

        2
      );

      this.reloadColorpicker();
      const ctx2 = this.provider2.context;
      this.drawCircle(
        ctx2,

        this.light * this.color_picker_width,
        this.sat * this.color_picker_height,
        this.color_picker_slider_width / 2,
        "transparent",
        "white",
        2
      );
      this.updateDisplay();
    },
    listenSlider2(event) {
      const rect = this.$refs["alpha_picker_slider"].getBoundingClientRect();
      const hue = event.clientY - rect.top;
      this.alpha = hue / this.color_picker_slider_height;
      const ctx = this.provider3.context;
      ctx.clearRect(
        0,
        0,
        this.color_picker_slider_width,
        this.color_picker_slider_height
      );
      this.reloadAlphapickerSlider();
      this.drawCircle(
        ctx,
        this.color_picker_slider_width / 2,
        hue,
        this.color_picker_slider_width / 2,
        "transparent",
        "white",

        2
      );
      this.updateDisplay();
    },
    listenpicker(event) {
      const rect = this.$refs["color_picker"].getBoundingClientRect();
      const light = (event.clientX - rect.left) / this.color_picker_width;
      const saturation = (event.clientY - rect.top) / this.color_picker_height;
      const ctx = this.provider2.context;
      this.reloadColorpicker();
      this.drawCircle(
        ctx,
        event.clientX - rect.left,
        event.clientY - rect.top,
        this.color_picker_slider_width / 2,
        "transparent",
        "white",
        2
      );
      this.sat = saturation;
      this.light = light;
      this.updateDisplay();
    },

    updateDisplay() {
      let corfinal = this.hslToRgb(this.hue, this.sat, this.light);

      this.R = corfinal[0];
      this.G = corfinal[1];
      this.B = corfinal[2];

      this.corEscolhidaHSL =
        this.hue.toFixed(2) +
        ";" +
        this.sat.toFixed(2) +
        ";" +
        this.light.toFixed(2);
      this.corEscolhidaRGB =
        this.R + "," + this.G + "," + this.B + "," + this.alpha.toFixed(2);

      this.canva_holder_style.backgroundColor =
        "rgba(" + this.R + "," + this.G + "," + this.B + "," + this.alpha + ")";
    },

    reloadColorpicker() {
      const ctx2 = this.provider2.context;
      let sub_divisao = 100;
      let tamn_max_w = this.color_picker_width;
      let tamn_max_h = this.color_picker_width;
      let corfinal;

      for (
        let tam_w = 0;
        tam_w < tamn_max_w;
        tam_w += tamn_max_w / sub_divisao
      ) {
        for (
          let tam_h = 0;
          tam_h < tamn_max_h;
          tam_h += tamn_max_h / sub_divisao
        ) {
          corfinal = this.hslToRgb(
            this.hue,
            tam_h / tamn_max_h,
            tam_w / tamn_max_w
          );
          ctx2.fillStyle =
            "rgb(" + corfinal[0] + "," + corfinal[1] + "," + corfinal[2] + ")";
          ctx2.fillRect(
            tam_w,
            tam_h,
            tamn_max_w / sub_divisao,
            tamn_max_h / sub_divisao
          );
        }
      }
    },

    reloadColorpickerSlider() {
      let sub_divisao = 100;
      let tamn_max = this.color_picker_slider_height;
      let corfinal;
      const ctx = this.provider1.context;

      for (
        let tamn_d = 0;
        tamn_d < tamn_max;
        tamn_d += tamn_max / sub_divisao
      ) {
        corfinal = this.hslToRgb(tamn_d / tamn_max, 1, 0.5);
        ctx.fillStyle =
          "rgb(" + corfinal[0] + "," + corfinal[1] + "," + corfinal[2] + ")";
        ctx.fillRect(
          0,
          tamn_d,
          this.color_picker_slider_width,
          tamn_max / sub_divisao
        );
      }
    },
    reloadAlphapickerSlider() {
      let sub_divisao = 100;
      let tamn_max = this.color_picker_slider_height;
      const ctx = this.provider3.context;

      for (
        let tamn_d = 0;
        tamn_d < tamn_max;
        tamn_d += tamn_max / sub_divisao
      ) {
        ctx.fillStyle = "rgba(0,0,0," + tamn_d / tamn_max + ")";
        ctx.fillRect(
          0,
          tamn_d,
          this.color_picker_slider_width,
          tamn_max / sub_divisao
        );
      }
    },
    grabItPicker(event) {
      const rect = this.$refs["color_picker"].getBoundingClientRect();

      const light = (event.clientX - rect.left) / this.color_picker_width;
      const saturation = (event.clientY - rect.top) / this.color_picker_height;
      const ctx2 = this.provider2.context;

      if (this.pickedColor) {
        this.reloadColorpicker();
        this.drawCircle(
          ctx2,
          this.light * this.color_picker_width,
          this.sat * this.color_picker_height,
          this.color_picker_slider_width / 2,
          "transparent",
          "white",
          2
        );

        this.sat = saturation;
        this.light = light;
        this.updateDisplay();
      }
    },
    clickedPicker(event) {
      const rect = this.$refs["color_picker"].getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      const minimo = 10;

      if (
        y > this.sat * this.color_picker_width - minimo &&
        y < this.sat * this.color_picker_width + minimo &&
        x > this.light * this.color_picker_height - minimo &&
        x < this.light * this.color_picker_height + minimo
      ) {
        this.pickedColor = true;
      }
    },
    unclickedPicker(event) {
      const rect = this.$refs["color_picker"].getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      const minimo = 10;

      if (
        y > this.sat * this.color_picker_width - minimo &&
        y < this.sat * this.color_picker_width + minimo &&
        x > this.light * this.color_picker_height - minimo &&
        x < this.light * this.color_picker_height + minimo
      ) {
        this.pickedColor = false;
      }
    },

    grabItColorPicker(event) {
      const rect = this.$refs["color_picker_slider"].getBoundingClientRect();

      const hue = event.clientY - rect.top;

      const ctx = this.provider1.context;

      if (this.pickedColorSlider) {
        this.hue = hue / this.color_picker_slider_height;

        ctx.clearRect(
          0,
          0,
          this.color_picker_slider_width,
          this.color_picker_slider_height
        );
        this.reloadColorpickerSlider();
        this.drawCircle(
          ctx,
          this.color_picker_slider_width / 2,
          hue,
          this.color_picker_slider_width / 2,
          "transparent",
          "white",

          2
        );
        this.reloadColorpicker();
        const ctx2 = this.provider2.context;
        this.drawCircle(
          ctx2,

          this.light * this.color_picker_width,
          this.sat * this.color_picker_height,
          this.color_picker_slider_width / 2,
          "transparent",
          "white",
          2
        );

        this.updateDisplay();
      }
    },
    clickedColorPicker(event) {
      const rect = this.$refs["color_picker_slider"].getBoundingClientRect();
      const x = event.clientY - rect.top;
      const minimo = 10;

      if (
        x > this.hue * this.color_picker_slider_height - minimo &&
        x < this.hue * this.color_picker_slider_height + minimo
      ) {
        this.pickedColorSlider = true;
      }
    },
    unclickedColorPicker(event) {
      const rect = this.$refs["color_picker_slider"].getBoundingClientRect();
      const x = event.clientY - rect.top;
      const minimo = 10;
      if (
        x > this.hue * this.color_picker_slider_height - minimo &&
        x < this.hue * this.color_picker_slider_height + minimo
      ) {
        this.pickedColorSlider = false;
      }
    },

    grabItAlphaPicker(event) {
      const rect = this.$refs["alpha_picker_slider"].getBoundingClientRect();
      const hue = event.clientY - rect.top;
      const ctx = this.provider3.context;

      if (this.pickedAlphaSlider) {
        this.alpha = hue / this.color_picker_slider_height;

        ctx.clearRect(
          0,
          0,
          this.color_picker_slider_width,
          this.color_picker_slider_height
        );
        this.reloadAlphapickerSlider();
        this.drawCircle(
          ctx,
          this.color_picker_slider_width / 2,
          hue,
          this.color_picker_slider_width / 2,
          "transparent",
          "white",
          2
        );

        this.updateDisplay();
      }
    },
    clickedAlphaPicker(event) {
      const rect = this.$refs["alpha_picker_slider"].getBoundingClientRect();
      const x = event.clientY - rect.top;
      const minimo = 10;

      if (
        x > this.alpha * this.color_picker_slider_height - minimo &&
        x < this.alpha * this.color_picker_slider_height + minimo
      ) {
        this.pickedAlphaSlider = true;
      }
    },
    unclickedAlphaPicker(event) {
      const rect = this.$refs["alpha_picker_slider"].getBoundingClientRect();
      const x = event.clientY - rect.top;
      const minimo = 10;
      if (
        x > this.alpha * this.color_picker_slider_height - minimo &&
        x < this.alpha * this.color_picker_slider_height + minimo
      ) {
        this.pickedAlphaSlider = false;
      }
    },
  },

  mounted() {
    this.provider1.context = this.$refs["color_picker_slider"].getContext("2d");
    this.provider2.context = this.$refs["color_picker"].getContext("2d");
    this.provider3.context = this.$refs["alpha_picker_slider"].getContext("2d");

    this.$refs["color_picker_slider"].width = this.color_picker_slider_width;
    this.$refs["color_picker_slider"].height = this.color_picker_slider_height;

    this.$refs["color_picker"].width = this.color_picker_width;
    this.$refs["color_picker"].height = this.color_picker_height;

    this.$refs["alpha_picker_slider"].width = this.color_picker_slider_width;
    this.$refs["alpha_picker_slider"].height = this.color_picker_slider_height;

    const ctx = this.provider1.context;
    const ctx2 = this.provider2.context;
    const ctx3 = this.provider3.context;

    this.reloadColorpickerSlider();
    this.reloadAlphapickerSlider();
    this.reloadColorpicker();

    this.drawCircle(
      ctx,
      this.color_picker_slider_width / 2,
      this.hue * this.color_picker_slider_height,
      this.color_picker_slider_width / 2,
      "transparent",
      "white",
      2
    );
    this.drawCircle(
      ctx3,
      this.color_picker_slider_width / 2,
      this.alpha * this.color_picker_slider_height,
      this.color_picker_slider_width / 2,
      "transparent",
      "white",
      2
    );

    this.drawCircle(
      ctx2,
      this.light * this.color_picker_width,
      this.sat * this.color_picker_height,
      this.color_picker_slider_width / 2,
      "transparent",
      "white",
      2
    );

    ///////////////////////////////

    this.$refs["color_picker"].addEventListener("click", this.listenpicker);
    this.$refs["color_picker"].addEventListener(
      "mousedown",
      this.clickedPicker
    );
    this.$refs["color_picker"].addEventListener(
      "mouseup",
      this.unclickedPicker
    );
    this.$refs["color_picker"].addEventListener("mousemove", this.grabItPicker);

    ///////////////////////////////
    this.$refs["color_picker_slider"].addEventListener(
      "click",
      this.listenSlider
    );
    this.$refs["color_picker_slider"].addEventListener(
      "mousedown",
      this.clickedColorPicker
    );
    this.$refs["color_picker_slider"].addEventListener(
      "mouseup",
      this.unclickedColorPicker
    );
    this.$refs["color_picker_slider"].addEventListener(
      "mousemove",
      this.grabItColorPicker
    );

    // ////////////////////////////////

    this.$refs["alpha_picker_slider"].addEventListener(
      "click",
      this.listenSlider2
    );
    this.$refs["alpha_picker_slider"].addEventListener(
      "mousedown",
      this.clickedAlphaPicker
    );
    this.$refs["alpha_picker_slider"].addEventListener(
      "mouseup",
      this.unclickedAlphaPicker
    );
    this.$refs["alpha_picker_slider"].addEventListener(
      "mousemove",
      this.grabItAlphaPicker
    );

    this.updateDisplay();

    ctx.save();
    ctx2.save();
    ctx3.save();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.canva_holder {
  display: flex;
  flex-direction: row;
}
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 3px solid #3e435700;
  border-radius: 10px;
}

.color_picker {
  margin-right: 10px;
}

.color_picker_slider {
  border: 3px solid #3e435700;
  border-radius: 10px;
  overflow: hidden;
}
</style>
