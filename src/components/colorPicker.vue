<template>
  <div class="container">
    <div class="canva_holder">
      <canvas ref="color_picker"></canvas>
      <br />
      <canvas ref="color_picker_slider"></canvas>
    </div>
    <div>
      <span>HSL</span>
      {{ corEscolhidaHSL }}
      <span>RGB</span>
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
      color_picker_width: this.canva_len * 0.8,
      color_picker_height: this.canva_len,
      color_picker_slider_width: this.canva_len * 0.2,
      color_picker_slider_height: this.canva_len,
      hue: Math.random(),
      sat: 1,
      light: 0.5,

      corEscolhidaHSL: "",
      corEscolhidaRGB: "",
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

    listenSlider(event) {
      const rect = this.$refs["color_picker_slider"].getBoundingClientRect();
      const hue = event.clientY - rect.top;
      this.hue = hue / this.color_picker_slider_height;
      this.reloadColorpicker();
      this.updateDisplay();
    },

    updateDisplay() {
      let corfinal = this.hslToRgb(this.hue, this.sat, this.light);

      this.corEscolhidaHSL =
        this.hue.toFixed(2) +
        ";" +
        this.sat.toFixed(2) +
        ";" +
        this.light.toFixed(2);
      this.corEscolhidaRGB =
        corfinal[0] + ";" + corfinal[1] + ";" + corfinal[2];
    },
    listenpicker(event) {
      const rect = this.$refs["color_picker"].getBoundingClientRect();
      const light = (event.clientX - rect.left) / this.canva_len;
      const saturation = (event.clientY - rect.top) / this.canva_len;

      this.sat = saturation;
      this.light = light;
      this.updateDisplay();
    },
    reloadColorpicker() {
      const ctx2 = this.provider2.context;
      let sub_divisao = 100;
      let tamn_max = this.canva_len;
      let corfinal;

      for (let tam_w = 0; tam_w < tamn_max; tam_w += tamn_max / sub_divisao) {
        for (let tam_h = 0; tam_h < tamn_max; tam_h += tamn_max / sub_divisao) {
          corfinal = this.hslToRgb(this.hue, tam_h / 255, tam_w / 255);
          ctx2.fillStyle =
            "rgb(" + corfinal[0] + "," + corfinal[1] + "," + corfinal[2] + ")";
          ctx2.fillRect(
            tam_w,
            tam_h,
            tamn_max / sub_divisao,
            tamn_max / sub_divisao
          );
        }
      }
    },
  },

  mounted() {
    this.provider1.context = this.$refs["color_picker_slider"].getContext("2d");

    this.provider2.context = this.$refs["color_picker"].getContext("2d");

    this.$refs["color_picker_slider"].width = this.color_picker_slider_width;
    this.$refs["color_picker_slider"].height = this.color_picker_slider_height;

    this.$refs["color_picker"].width = this.canva_len;
    this.$refs["color_picker"].height = this.canva_len;

    const ctx = this.provider1.context;
    //const ctx2 = this.provider2.context;

    let sub_divisao = 100;
    let tamn_max = this.color_picker_slider_height;
    let corfinal;

    for (let tamn_d = 0; tamn_d < 255; tamn_d += tamn_max / sub_divisao) {
      corfinal = this.hslToRgb(tamn_d / 255, 1, 0.5);
      ctx.fillStyle =
        "rgb(" + corfinal[0] + "," + corfinal[1] + "," + corfinal[2] + ")";
      ctx.fillRect(
        0,
        tamn_d,
        this.color_picker_slider_width,
        tamn_max / sub_divisao
      );
    }

    this.reloadColorpicker();

    this.$refs["color_picker_slider"].addEventListener(
      "click",
      this.listenSlider
    );

    this.$refs["color_picker"].addEventListener("click", this.listenpicker);
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
}
</style>
