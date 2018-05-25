<template>
  <v-app id="inspire" dark>
    <v-toolbar app fixed clipped-left>
      <v-toolbar-title>Android SVG to Drawable resources</v-toolbar-title>
    </v-toolbar>
    <v-content>
      <v-container fluid fill-height>
        <v-layout justify-center align-center>
          <v-flex shrink>
            <v-layout row>
              <img ref="preview" :height="size" :src="imageData">
            </v-layout>
            <v-layout row>
              <v-text-field v-model="size" label="Size mdpi"></v-text-field>
            </v-layout>
            <v-layout row>
              <v-select :items="folders" v-model="selectFolder" label="Resources folder"></v-select>
            </v-layout>
            <v-layout row>
              <v-btn v-if="imageData !== ''" color="info" @click="download()">Download zip</v-btn>
              <v-btn v-else color="info" @click="$refs.inputFile.click()">Upload svg</v-btn>
            </v-layout>
            <div style="display: none;">
              <input ref="inputFile" type="file" @change="previewImage" accept="image/svg+xml">
              <canvas ref="canvas" :height="size" :width="size" />
            </div>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
  import JSZip from 'jszip'
  import FileSaver from 'filesaver.js-npm'


  export default {
    data: () => ({
      imageData: '',
      folders: ['mipmap', 'drawable'],
      size: 48,
      selectFolder: "mipmap",
      name: ""
    }),
    methods: {
        previewImage: function(event) {
            var input = event.target;

            this.name = input.files[0].name.replace(/.svg/gi, '')

            if (input.files && input.files[0]) {
                var reader = new FileReader();
                reader.onload = (e) => {
                    this.imageData = e.target.result;
                }
                reader.readAsDataURL(input.files[0]);
            }
        },
        download: function () {
          let name = this.name

          var zip = new JSZip();
          var mdpi = zip.folder(this.selectFolder + '-mdpi');
          mdpi.file(name + ".png", this.previewToPng(1), {base64: true});

          var hdpi = zip.folder(this.selectFolder + '-hdpi');
          hdpi.file(name + ".png", this.previewToPng(1.5), {base64: true});

          var xhdpi = zip.folder(this.selectFolder + '-xhdpi');
          xhdpi.file(name + ".png", this.previewToPng(2), {base64: true});

          var xxhdpi = zip.folder(this.selectFolder + '-xxhdpi');
          xxhdpi.file(name + ".png", this.previewToPng(3), {base64: true});

          var xxxhdpi = zip.folder(this.selectFolder + '-xxxhdpi');
          xxxhdpi.file(name + ".png", this.previewToPng(4), {base64: true});

          zip.generateAsync({type:"blob"})
          .then(function(content) {
              FileSaver.saveAs(content, name + ".zip");
          });
        },
        previewToPng: function (size) {
          let def = this.size;
          this.size = def * size;

          let c = this.$refs.canvas
          var ctx = c.getContext("2d")
          var img = this.$refs.preview

          ctx.canvas.width  = this.size;
          ctx.canvas.height = this.size;

          ctx.clearRect(0, 0, c.width, c.height);
          ctx.drawImage(img, 0, 0, this.size, this.size)

          this.size = def * 1;

          return c.toDataURL().replace(/data:image\/png;base64,/gi, '')
        },
        resize: function () {
          let def = this.size;

          this.size = def * 1;
          previewToPng()
          this.size = def * 1.5;
          this.size = def * 2;
          this.size = def * 3;
          this.size = def * 4;

          this.size = def * 1;
        }
    }
  }
</script>
