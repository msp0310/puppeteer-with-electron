<template>
  <div>
    <h1>Screenshot</h1>
    <el-row>
      <el-alert type="error" :title="message" v-if="message"></el-alert>
    </el-row>
    <el-row>
      <el-col :span="18">
        <el-input v-model="url" placeholder="URL">
        </el-input>
      </el-col>
      <el-col :span="6">
        <el-button type="primary" @click="openFileDialog" icon="el-icon-download" :loading="isLoading">Download</el-button>
        <input type="file" ref="fileDialog" hidden webkitdirectory>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  const puppeteer = require('puppeteer')
  const remote = require('electron').remote
  const dialog = remote.dialog

  export default {
    data () {
      return {
        url: 'http://',
        isLoading: false,
        folderPath: '',
        message: ''
      }
    },
    methods: {
      openFileDialog () {
        // open file selection dialog.
        dialog.showOpenDialog(null, {
          properties: ['openDirectory']
        }, folderPath => {
          this.folderPath = folderPath
          this.screenshot()
        })
      },
      async screenshot () {
        try {
          // puppeteer api.
          this.isLoading = true
          const browser = await puppeteer.launch({
            headless: true,
            executablePath: puppeteer.executablePath().replace('app.asar', 'app.asar.unpacked'),
            args: [
              '--no-sandbox',
              '--disable-setuid-sandbox'
            ]
          })
          const page = await browser.newPage()
          await page.goto(this.url)
          await page.screenshot({path: this.folderPath + '/screenshot.png', fullPage: true})
          await browser.close()
        } catch (e) {
          this.message = e.toString()
        } finally {
          this.isLoading = false
        }
      }
    }
  }
</script>

<style scoped>
</style>
