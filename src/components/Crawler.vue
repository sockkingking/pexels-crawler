<template>
  <v-container>
    <v-row class="text-center" align="center" justify="center">
      <v-col cols="10">
        <v-snackbar
            v-model="snackbar"
            :timeout="timeout"
            color="error"
            outlined
            centered
            absolute
            top
            width="800"
        >
          {{ textError }}

          <template v-slot:action="{ attrs }">
            <v-btn
                color="error"
                text
                v-bind="attrs"
                @click="snackbar = false"
            >
              Close
            </v-btn>
          </template>
        </v-snackbar>
      </v-col>
    </v-row>
    <v-row class="text-center mt-10" align="center"
           justify="center">
      <v-col cols="6">
        <v-form
            ref="form"
            v-model="valid"
            lazy-validation
        >
          <v-text-field
              label="Search"
              hide-details="auto"
              v-model="keyword"
              class="mt-5"
              :rules="ruleText"
              pattern="[A-Za-z]"
          ></v-text-field>
          <v-text-field
              label="Page size : max 80"
              hide-details="auto"
              v-model="pageSize"
              class="mt-5"
              disabled
              :rules="rulesNumb"
          ></v-text-field>
          <v-text-field
              class="mt-5"
              label="Page Start"
              hide-details="auto"
              v-model="pageStart"
              :rules="rulesNumb"
          ></v-text-field>
          <v-text-field
              class="mt-5"
              label="Page End"
              hide-details="auto"
              v-model="pageEnd"
              :rules="rulesNumb"
          ></v-text-field>
          <v-text-field
              class="mt-5"
              label="File name"
              hide-details="auto"
              v-model="fileName"
              :rules="ruleText"
          ></v-text-field>
          <v-text-field
              class="mt-5"
              label="ID"
              hide-details="auto"
              v-model="tokenId"
              :rules="ruleText"
          ></v-text-field>
          <v-btn
              class="mt-6"
              depressed
              color="primary"
              @click="exportExcel"
              :disabled="!valid"
              :loading="loading"
          >Download
            <v-icon
                right
                dark
            >
              mdi-cloud-download
            </v-icon>
          </v-btn>
        </v-form>

      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import {createClient} from 'pexels'
import exportFromJSON from 'export-from-json'

export default {
  name: 'Crawler',

  data: () => ({
    loading: false,
    fileName: "",
    textError: "Connect error @@ .  ID is incorrect !!",
    timeout: 3500,
    snackbar: false,
    disableDownload: true,
    valid: true,
    keyword: "",
    pageSize: 80,
    pageStart: 1,
    pageEnd: 10,
    tokenId: "",
    rulesNumb: [
      value => !!value || 'Required.',
      value => (value > 0) || 'Should be large than 1',
    ],
    ruleText: [
      value => !!value || 'Required.',
      value => (value.length >= 2) || 'Min 2 characters',
    ]
  }),

  methods: {
    validate() {
      this.$refs.form.validate();
    },

    async exportExcel() {
      this.loading = true;
      await setTimeout(()=> {
        this.downloadExcel().then(data => {
          const fileName = this.fileName;
          const exportType = 'xls';
          exportFromJSON({data, fileName, exportType});
          this.loading = false;
        }).catch(() => {this.snackbar = true; this.loading = false;});
      }, 1000);


    },

    async downloadExcel() {
      if (this.valid) {

        let imagesInfo
        try {
          imagesInfo = await Promise.all(this.getDataByPages());
        } catch (e) {
          this.snackbar = true;
        }

        let data = [];
        imagesInfo.forEach(promise => {
          let subData = promise.photos.map((img) => {
            const title = this.extractTitleFromUrl(img);
            const url = img.src.large;
            return {Title: title, URL: url};
          });
          Array.prototype.unshift.apply(data, subData);
        });

        let uniqueValues = new Set();
        let final = [];
        data.forEach(function(el) {

          if (!uniqueValues.has(el.URL)) {
            uniqueValues.add(el.URL);
            final.push(el);
          }
        });
        return final;
      }
    },

    getDataByPages() {
      const client = createClient(this.tokenId);
      const query = this.keyword;
      const per_page = this.pageSize;
      const pageStart = this.pageStart;
      const pageEnd = this.pageEnd;
      let promises = [];
      if (pageEnd < pageStart) {
        this.snackbar = true;
        return null;
      }
      for (let i = pageStart; i <= pageEnd; i++) {
        let res = client.photos.search({query, per_page: per_page, page: i});
        promises.push(res);
      }
      return promises;
    },

    extractTitleFromUrl(img) {
      const SPACE = " ";
      const {url, id} = img;
      const arr = url.split("/");
      const titleContainsId = arr[arr.length - 2];
      return titleContainsId.replace(`-${id}`, "").replace(/[^a-zA-Z0-9]/g, SPACE);
    }
  },
  mounted() {
    this.validate();
  }
}
</script>
