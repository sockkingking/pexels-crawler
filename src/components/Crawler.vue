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
              :rules="rulesNumb"
          ></v-text-field>
          <v-text-field
              class="mt-5"
              label="Page"
              hide-details="auto"
              v-model="page"
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
              @click="downloadExcel"
              :disabled="!valid"
          >search
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
    fileName: "",
    textError: "Connect error @@ .  ID is incorrect !!",
    timeout: 3500,
    snackbar: false,
    disableDownload: true,
    valid: true,
    keyword: "",
    pageSize: 10,
    page: 1,
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

    async downloadExcel() {
      if (this.valid) {
        const client = createClient(this.tokenId);
        const query = this.keyword;
        const per_page = this.pageSize;
        const page = this.page;
        let res;
        try {
          res = await client.photos.search({query, 'photoModalImageAlt': 1, per_page: per_page, page: page});
        } catch (e) {
          this.snackbar = true;
        }
        const imagesInfo = res.photos;

        const data = imagesInfo.map((img) => {
          const title = this.extractTitleFromUrl(img);
          const url = img.url;

          return {Title: title, URL: url};
        });
        const fileName = this.fileName;
        const exportType = 'xls'
        exportFromJSON({data, fileName, exportType});
      }
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
