<template>
  <main>
    <FileUpload @upload="onAdvancedUpload($event)" :auto="true" customUpload @uploader="loadFont">
      <template #empty>
        <p>Drag and drop font here to load.</p>
      </template>
    </FileUpload>
    <FieldSet legend="Parameters">
      <div class="flex flex-wrap gap-4">
        <div class="w-12rem flex flex-column gap-2">
          <label for="scale">Scale</label>
          <InputText id="scale" v-model.number="config.geometry.scale" class="w-full" />
          <Slider v-model="config.geometry.scale" :min="0.6" :max="1" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="sideMargin">Side margin</label>
          <InputText id="sideMargin" v-model.number="config.geometry.sideMargin" class="w-full" />
          <Slider v-model="config.geometry.sideMargin" :min="-0.2" :max="0.2" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="tracking">Tracking</label>
          <InputText id="tracking" v-model.number="config.geometry.tracking" class="w-full" />
          <Slider v-model="config.geometry.tracking" :min="-0.2" :max="0.2" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="yShiftMax">Y shift of first letter</label>
          <InputText id="yShiftMax" v-model.number="config.geometry.yShiftMax" class="w-full" />
          <Slider v-model="config.geometry.yShiftMax" :min="0" :max="0.4" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="yShiftMin">Y shift of last letter</label>
          <InputText id="yShiftMin" v-model.number="config.geometry.yShiftMin" class="w-full" />
          <Slider v-model="config.geometry.yShiftMin" :min="-0.2" :max="0" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="yTop">Top of the font</label>
          <InputText id="yTop" v-model.number="config.geometry.yTop" class="w-full" />
          <Slider v-model="config.geometry.yTop" :min="0.6" :max="1" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
        <div class="w-12rem flex flex-column gap-2">
          <label for="yBottom">Bottom of the font</label>
          <InputText id="yBottom" v-model.number="config.geometry.yBottom" class="w-full" />
          <Slider v-model="config.geometry.yBottom" :min="-0.4" :max="0" :step="0.01" class="w-full mt-1 mb-1" />
        </div>
      </div>
    </FieldSet>
    <FieldSet legend="Action">
      <Button @click="controlFont.download()">Save font</Button>
    </FieldSet>
    <FieldSet legend="Preview">
      <div class="flex flex-wrap gap-3">
        <canvas v-for="control in controlCharacters" :id="`preview-${control.codePoint}`"></canvas>
      </div>
    </FieldSet>
  </main>
</template>

<script>
import Button from 'primevue/button';
import FieldSet from 'primevue/fieldset';
import FileUpload from 'primevue/fileupload';
import InputText from 'primevue/inputtext';
import Slider from 'primevue/slider';

import opentype from 'opentype.js';

export default {
  data() {
    return {
      font: null,
      dpr: window.devicePixelRatio,
      config: {
        geometry: {
          scale: 0.8,
          sideMargin: 0,
          tracking: 0,
          yShiftMax: 0.2,
          yShiftMin: -0.1,
          yTop: 0.8,
          yBottom: -0.2,
        },
      },
      controlCharacters: [
        { codePoint: 0x0, display: 'NUL' },
        { codePoint: 0x1, display: 'SOH' },
        { codePoint: 0x2, display: 'STX' },
        { codePoint: 0x3, display: 'ETX' },
        { codePoint: 0x4, display: 'EOT' },
        { codePoint: 0x5, display: 'ENQ' },
        { codePoint: 0x6, display: 'ACK' },
        { codePoint: 0x7, display: 'BEL' },
        { codePoint: 0x8, display: 'BS' },
        { codePoint: 0x9, display: 'HT' },
        { codePoint: 0xA, display: 'LF' },
        { codePoint: 0xB, display: 'VT' },
        { codePoint: 0xC, display: 'FF' },
        { codePoint: 0xD, display: 'CR' },
        { codePoint: 0xE, display: 'SO' },
        { codePoint: 0xF, display: 'SI' },
        { codePoint: 0x10, display: 'DLE' },
        { codePoint: 0x11, display: 'DC1' },
        { codePoint: 0x12, display: 'DC2' },
        { codePoint: 0x13, display: 'DC3' },
        { codePoint: 0x14, display: 'DC4' },
        { codePoint: 0x15, display: 'NAK' },
        { codePoint: 0x16, display: 'SYN' },
        { codePoint: 0x17, display: 'ETB' },
        { codePoint: 0x18, display: 'CAN' },
        { codePoint: 0x19, display: 'EM' },
        { codePoint: 0x1A, display: 'SUB' },
        { codePoint: 0x1B, display: 'ESC' },
        { codePoint: 0x1C, display: 'FS' },
        { codePoint: 0x1D, display: 'GS' },
        { codePoint: 0x1E, display: 'RS' },
        { codePoint: 0x1F, display: 'US' },
        { codePoint: 0x7F, display: 'DEL' },
      ],
    }
  },
  computed: {
    referencedAlnums() {
      const alnums = new Set();
      for (const cChar of this.controlCharacters) {
        for (const ch of cChar.display) {
          alnums.add(ch);
        }
      }
      return Array.from(alnums).sort();
    },
    originalGlyphs() {
      if (!this.font) {
        return {};
      }
      const glyphs = {};
      for (const ch of this.referencedAlnums) {
        glyphs[ch] = this.font.charToGlyph(ch);
      }
      return glyphs;
    },
    scaledGlyphs() {
      const glyphs = {};
      const orig = this.originalGlyphs;
      for (const ch in orig) {
        glyphs[ch] = new opentype.Glyph({
          name: ch,
          unicode: ch.codePointAt(0),
          advanceWidth: orig[ch].advanceWidth * this.config.geometry.scale,
          path: this.scalePath(orig[ch].path, this.config.geometry.scale),
        });
      }
      return glyphs;
    },
    composedGlyphs() {
      if (!this.font) {
        return [];
      }
      const upm = this.font.unitsPerEm;
      const geom = this.config.geometry;
      const result = [];
      for (const control of this.controlCharacters) {
        const length = control.display.length;
        let width = upm * geom.sideMargin - upm * geom.tracking;
        const path = new opentype.Path();
        for (let i = 0; i < length; i++) {
          const ch = control.display[i];
          const scaledGlyph = this.scaledGlyphs[ch];
          width += upm * geom.tracking;
          const yOffset = upm * (geom.yShiftMax * (length - 1 - i) + geom.yShiftMin * i) / (length - 1);
          path.extend(this.shiftPath(scaledGlyph.path, width, yOffset));
          width += scaledGlyph.advanceWidth;
        }
        width += upm * geom.sideMargin;
        path.extend(this.reversedBox(width));
        const glyph = new opentype.Glyph({
          name: control.display,
          unicode: control.codePoint,
          advanceWidth: width,
          path: path,
        });
        result.push(glyph);
      }
      return result;
    },
    controlFont() {
      const geom = this.config.geometry;
      if (!this.font) {
        return new opentype.Font({
          familyName: 'Red Panda Control',
          styleName: 'Regular',
          unitsPerEm: 1000,
          ascender: 1000 * geom.yTop,
          descender: 1000 * geom.yBottom,
          glyphs: [],
        });
      }
      const glyphs = this.composedGlyphs;
      const upm = this.font.unitsPerEm;
      const font = new opentype.Font({
        familyName: 'Red Panda Control',
        styleName: 'Regular',
        postScriptName: 'RedPandaControl-Regular',
        copyright: this.font.names.copyright?.en,
        license: this.font.names.license?.en,
        licenseURL: this.font.names.licenseURL?.en,
        manufacturer: 'Red Panda Control Font Generator',
        manufacturerURL: 'https://github.com/redpanda-cpp/control-font',
        unitsPerEm: upm,
        ascender: upm * geom.yTop,
        descender: upm * geom.yBottom,
        glyphs: glyphs,
      });
      return font;
    },
  },
  watch: {
    controlFont(newFont, oldFont) {
      const geom = this.config.geometry;
      const upm = newFont.unitsPerEm;
      const height = 100 * (geom.yTop - geom.yBottom);
      for (const control of this.controlCharacters) {
        const glyph = newFont.charToGlyph(String.fromCodePoint(control.codePoint));
        const width = glyph.advanceWidth / upm * 100;
        const canvas = document.getElementById(`preview-${control.codePoint}`);
        canvas.style.width = `${width}px`;
        canvas.style.height = `${height}px`;
        canvas.width = width * this.dpr;
        canvas.height = height * this.dpr;
        const ctx = canvas.getContext('2d');
        ctx.clearRect(0, 0, width * this.dpr, height * this.dpr);
        newFont.draw(ctx, String.fromCodePoint(control.codePoint), 0, 100 * this.dpr * geom.yTop, 100 * this.dpr);
      }
    }
  },
  methods: {
    loadFont(event) {
      const file = event.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        const font = opentype.parse(e.target.result);
        this.font = font;
        console.log('Font loaded:', font);
      };
      reader.readAsArrayBuffer(file);
    },
    transformPath(path, xx, xy, dx, yx, yy, dy) {
      const newPath = new opentype.Path();
      for (const cmd of path.commands) {
        switch (cmd.type) {
          case 'M':
            newPath.moveTo(
              cmd.x * xx + cmd.y * xy + dx,
              cmd.x * yx + cmd.y * yy + dy,
            );
            break;
          case 'L':
            newPath.lineTo(
              cmd.x * xx + cmd.y * xy + dx,
              cmd.x * yx + cmd.y * yy + dy,
            );
            break;
          case 'C':
            newPath.curveTo(
              cmd.x1 * xx + cmd.y1 * xy + dx,
              cmd.x1 * yx + cmd.y1 * yy + dy,
              cmd.x2 * xx + cmd.y2 * xy + dx,
              cmd.x2 * yx + cmd.y2 * yy + dy,
              cmd.x * xx + cmd.y * xy + dx,
              cmd.x * yx + cmd.y * yy + dy,
            );
            break;
          case 'Q':
            newPath.quadraticCurveTo(
              cmd.x1 * xx + cmd.y1 * xy + dx,
              cmd.x1 * yx + cmd.y1 * yy + dy,
              cmd.x * xx + cmd.y * xy + dx,
              cmd.x * yx + cmd.y * yy + dy,
            );
            break;
          case 'Z':
            newPath.close();
            break;
        }
      }
      return newPath;
    },
    scalePath(path, factor) {
      return this.transformPath(path, factor, 0, 0, 0, factor, 0);
    },
    shiftPath(path, dx, dy) {
      return this.transformPath(path, 1, 0, dx, 0, 1, dy);
    },
    reversedBox(width) {
      const path = new opentype.Path();
      const geom = this.config.geometry;
      const upm = this.font.unitsPerEm;
      if (this.font.outlinesFormat == 'truetype') {
        path.moveTo(0, upm * geom.yTop);
        path.lineTo(0, upm * geom.yBottom);
        path.lineTo(width, upm * geom.yBottom);
        path.lineTo(width, upm * geom.yTop);
        path.close();
      } else {
        path.moveTo(0, upm * geom.yTop);
        path.lineTo(width, upm * geom.yTop);
        path.lineTo(width, upm * geom.yBottom);
        path.lineTo(0, upm * geom.yBottom);
        path.close();
      }
      return path;
    },
  },
  components: {
    Button,
    FieldSet,
    FileUpload,
    InputText,
    Slider,
  },
};
</script>

<style scoped>
</style>
