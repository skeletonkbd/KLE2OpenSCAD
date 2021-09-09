<script>
import * as kle from '@ijprest/kle-serial'

export default {
  data: () => ({
    RAW: '',
    KLE_data: [],
    CODE: '',
    displayCode: false,
  }),
  methods: {
    getData() {
      let data = this.RAW
      if (data.length > 0) {
        data = data.replace(/([a-zA-Z]\d):/g, function () {
          return '"' + arguments[1] + '":'
        })
        data = data.replace(/([a-zA-Z][a-zA-Z]):/g, function () {
          return '"' + arguments[1] + '":'
        })
        data = data.replace(/([a-zA-Z]):/g, function () {
          return '"' + arguments[1] + '":'
        })
        data = '[' + data + ']'
        try {
          data = JSON.parse(data)
          //console.log(data);
        } catch (error) {
          alert('解析に失敗しました')
        }

        let keyboard = kle.Serial.deserialize(data)
        this.KLE_data = keyboard.keys
        this.displayCode = true
        console.log(keyboard.keys)
      }
    },
    copy() {
      const copyText = this.$el.querySelector('#code').textContent
      navigator.clipboard
        .writeText(copyText)
        .then(() => {
          alert('クリップボードにコピーしました')
        })
        .catch((e) => {
          console.error(e)
        })
    },
  },
  computed: {
    generatedCode() {
      console.log('コード生成開始')

      let pitch = 19.05
      let hole = 14
      let code = '// KLE_data2OpenSCAD\n\n'
      code += 'pitch = ' + pitch + '; // キーピッチ(mm)\n'
      code += 'hole = ' + hole + '; // キースイッチの穴のサイズ(mm)\n\n'
      code += '// 0: x, 1: y, 2:r, 3:rx, 4:ry, 5:w, 6:h\n'
      code += 'keys = ['
      this.KLE_data.forEach(function (key) {
        code +=
          '[' +
          key.x +
          ',' +
          key.y +
          ',' +
          key.rotation_angle +
          ',' +
          key.rotation_x +
          ',' +
          key.rotation_y +
          ',' +
          key.width +
          ',' +
          key.height +
          '],'
      })
      code += '];\n\n'
      code += `rotate([0,0,-90]) {
    for(k=keys){
        trans1_x = k[4] * pitch;
        trans1_y = k[3] * pitch;
        trans2_x = k[1] * pitch -trans1_x;
        trans2_y = k[0] * pitch -trans1_y;
        trans3_x = (k[6]-1) * pitch / 2;
        trans3_y = (k[5]-1) * pitch / 2;
        rotation = -k[2];
        ofs=-(pitch-hole)/2;

        translate([trans1_x,trans1_y,0])
        rotate([0,0,rotation])
        translate([trans2_x,trans2_y,0])
        translate([trans3_x,trans3_y,0])
        offset(ofs)
        square(pitch);
    }
}`
      console.log(code)
      //this.CODE = code
      return code
    },
  },
}
</script>

<template>
  <div class="container mx-auto py-10">
    <header class="mb-5">
      <h1 class="font-bold text-2xl">KLE_data2OpenSCAD</h1>
      <p>
        <a href="http://www.keyboard-layout-editor.com/" target="_blank"
          >Keyboard Layout Editor</a
        >で作成したレイアウトをもとに、OpenSCADのコードを自動生成するツールです。
      </p>
    </header>

    <div class="mb-5 p-3 bg-gray-100 border rounded">
      <h2 class="mb-2 font-bold">Keyboard Layout Editor Rawデータ</h2>
      <textarea
        v-model="RAW"
        class="resize-y border rounded-md w-full p-2"
        placeholder="ここに貼り付け…"
      ></textarea>
    </div>

    <div class="mb-5 text-center">
      <button
        class="
          bg-green-500
          px-10
          py-3
          rounded
          text-white
          font-bold
          text-xl
          shadow
          hover:bg-green-600
        "
        @click="getData"
      >
        Generate OpenSCAD code
      </button>
    </div>

    <div
      class="mb-5 border bg-green-100 text-green-800 rounded relative"
      v-if="displayCode"
    >
      <div class="absolute top-0 right-0 p-2">
        <span
          class="
            bg-green-200
            text-green-600
            p-1
            rounded
            hover:bg-green-400 hover:text-white
          "
          @click="copy"
          >Copy</span
        >
      </div>
      <pre><code id="code" class="block whitespace-pre overflow-x-scroll p-5">{{ generatedCode }}</code></pre>
    </div>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
